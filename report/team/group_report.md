# Báo cáo nhóm — Lab 8 LogiX-VN

Tối đa 5 trang ngoài phụ lục. Số liệu lấy từ file trong `src/reports/`, không chép tay từ terminal.

## 1. Thông tin nhóm

- Tên nhóm: NNHT (Lớp L3B)
- Thành viên: xem `TEAM.md`
- Commit chốt (trùng `report/team/decision_adr.md`): `a7ffc66`

## 2. Mục tiêu và cách tiếp cận

- **Cách hiểu bài toán:** Hệ thống xử lý claim hoàn phí vận chuyển / bồi thường hỏng hàng của LogiX-VN dưới cùng chữ ký `run_claim(claim_id, *, seed, store_path, crash_at=None) -> RunResult`. Nhóm giữ nguyên tắc xuyên suốt: mọi quyết định về phạm vi chủ sở hữu (`principal_id`), hiệu lực điều khoản (`is_current = 1`, `effective_date <= 2026-11-15`), công thức tính tiền (`compute_compensation`), kiểm duyệt approval theo đúng claim, trần hoàn tiền theo đơn và chống ghi trùng (`idempotency_key = refund:{claim_id}`) đều do **code và cổng host** quyết định; `MockLLM` chỉ hỗ trợ điều phối đọc trong giới hạn hoặc soạn câu chữ, còn nội dung `ticket` luôn được coi là dữ liệu người dùng không đáng tin.
- **Phân công trong nhóm 4 thành viên:**
  - **Thiều Thế Ngọc:** Cài đặt Shape A (`workflow`), viết test `test_workflow.py` và kiểm tra tích hợp 4 shape.
  - **Trần Trọng Đức Tài:** Cài đặt Shape B (`single` ReAct agent) và viết test `test_single_agent.py`.
  - **Đào Ngọc Hân:** Cài đặt Shape C (`orchestrator`), xử lý lọc dữ liệu chuyển giao `handoff.v1`, viết test `test_orchestrator_handoff.py` và rà soát bảo mật trace liên shape.
  - **Nguyễn Gia Nam:** Kiểm thử `ledger` / `approval` / `crash` (`test_ledger_order_budget.py`), chạy crash suite `20/20`, chạy benchmark 4 shape và thực hiện ablation R2 (`swarm_ticket_ablation.py`).
- **Thứ tự thực hiện:** Kiểm tra cổng host (`pytest -q`) và đọc `swarm_reference.py` (Shape D) ở CP1 → triển khai độc lập 3 shape A, B, C kèm `handoff` allowlist và truyền `crash_at` ở CP2–CP3 → chạy crash suite 20 điểm và quét chuỗi cấm trên trace ở CP4 → gộp cả 4 shape, đo trên cùng corpus/seed/bảng giá để chốt kiến trúc ít tự chủ nhất mà vẫn đạt yêu cầu ở CP5.

## 3. Giải pháp

Mô tả ba shape nhóm viết (workflow, single, orchestrator): mỗi shape đọc gì, quyết định gì, gọi host ra sao. Chỉ rõ ranh giới code / model (tham khảo [docs/architecture.md](../../docs/architecture.md)), cách handoff được sanitize và trace ghi những event nào.

### 3.1 Shape A — `workflow` (`src/lab8/shapes/workflow.py`)

- **Luồng thực thi và ranh giới Code / Model:** Luồng bước Python tuần tự cố định, không gọi `MockLLM` (`model_calls = 0`), toàn bộ logic nằm trong code.
  - Mở kết nối SQLite, đọc claim qua `get_claim_view` (không dùng `SELECT *` nên không chạm `parent_secret` và `raw_headers_json`). Nếu `claim_id` không tồn tại trả `failed`; nếu thiếu `order_id` trả `needs_info` (`proposal_amount = 0`, không ghi ledger).
  - Gọi `lookup_order(conn, order_id, principal_id)`. Nếu `visible = False` (không tìm thấy đơn hoặc đơn thuộc `principal` khác) trả `escalated` (`proposal_amount = 0`), không đọc các trường PII của đơn hàng.
  - Tính quote tất định từ `order_view["order"]` bằng `compute_compensation`, bỏ qua hoàn toàn con số ngày trễ hoặc điều khoản cũ ghi trong `ticket`.
- **Gọi cổng Host và ghi Trace:** Nếu `proposal <= 0` trả `denied` (`proposal_amount = 0`, không gọi `issue_refund`). Nếu `proposal > 0`, truyền **quote đầy đủ** và `crash_at` vào `durability.issue_refund` (nếu gặp `InjectedCrash` thì ghi event `crash`, flush trace rồi cho ngoại lệ lan truyền). Trace được xuất ở mọi nhánh qua `common.finish()`, gồm các event `tool_call` (chỉ ghi tên tool và trạng thái `found`/`visible`/`reason`), `quote`, `refund`, `decision` và `crash`.

### 3.2 Shape B — `single` (`src/lab8/shapes/single.py`)

- **Luồng thực thi và vòng lặp ReAct:** Đọc claim qua `get_claim_view`, kiểm tra sớm `unknown_claim` (`failed`) và thiếu `order_id` (`needs_info`). Gọi `MockLLM` (`purpose="parse"` và `"react"`) có tính vào `Budget` nhưng không dùng kết quả sinh chữ để ghi đè dữ liệu DB.
  - Chạy một vòng `run_react` với giới hạn cứng `max_steps = 8` (`worker = False`; `dispatch` tự từ chối mọi write tool nếu model cố gọi). Hàm `decide(seen)` điều phối tuần tự hai read tool: bước 1 gọi `lookup_order` theo đúng `principal_id` của claim (nếu `visible = False` thì kết thúc vòng lặp với `escalated`); bước 2 gọi `compute_compensation` trên bản ghi đơn có cấu trúc rồi trả kết quả `ready` kèm quote. Nếu chạm ngưỡng 8 bước (`step_cap`) trả `failed`.
- **Ranh giới Code / Model, cổng Host và Trace:** Model chỉ điều phối bước đọc và soạn nháp phản hồi cuối (`purpose="draft"`). Sau khi vòng ReAct kết thúc, code của shape kiểm tra `proposal`: bằng `0` trả `denied`; lớn hơn `0` mới gọi `durability.issue_refund` với **quote đầy đủ** và `crash_at` (flush trace trước khi re-raise `InjectedCrash`). Trace ghi các event `tool_call` (chỉ chứa các trường trong `ORDER_FIELDS`), `quote`, `decision`, `error` và `crash`.

### 3.3 Shape C — `orchestrator` & Cơ chế Handoff / Trace an toàn (`src/lab8/shapes/orchestrator.py`)

- **Luồng đọc và tách quyền Orchestrator / Investigator:**
  - Orchestrator đọc thông tin claim an toàn qua `get_claim_view` (chỉ lấy `claim_id`, `principal_id`, `order_id`, `ticket`; không đọc `parent_secret` hay `raw_headers_json`). Nếu `claim_id` không tồn tại trả `failed`; nếu thiếu `order_id` trả ngay `needs_info` (`proposal_amount = 0`, không trích mã đơn từ nội dung `ticket`).
  - Orchestrator gọi `MockLLM` (`purpose="plan"`) để lập kế hoạch điều phối, sau đó đóng gói yêu cầu bằng `lab8.handoff.make_handoff` theo chuẩn `handoff.v1`.
  - Investigator chạy qua `run_readonly_investigators` chỉ với các tool đọc `["lookup_order", "lookup_policy"]` gắn chặt theo `principal_id` của claim, hoàn toàn không được cấp quyền gọi tool ghi (`issue_refund`, `append_audit`). Nếu `lookup_order` báo `visible = False` (không tìm thấy đơn hoặc khác chủ sở hữu), Orchestrator trả `escalated` (`proposal_amount = 0`).
- **Ranh giới Code / Model và gọi cổng Host:**
  - `MockLLM` chỉ tham gia ở bước `plan` và soạn câu trả lời `draft`; tuyệt đối không lấy `delay_days` từ câu chữ trong `ticket` (tránh lỗi ở `seed = 55` của `swarm` trên T02) và không tự tính tiền.
  - Orchestrator lấy bản ghi đơn có cấu trúc từ kết quả của Investigator, gọi `compute_compensation` để tính quote tất định theo policy hiện hành (`is_current = 1`, `effective_date <= 2026-11-15`) và quy tắc không cộng dồn (`max`).
  - Nếu `proposal <= 0` → trả `denied` (`proposal_amount = 0`), không gọi `issue_refund`. Nếu `proposal > 0` → truyền **quote đầy đủ** và `crash_at` vào `durability.issue_refund` để host kiểm tra approval thuộc đúng claim, tự trừ phần tiền đơn hàng đã hoàn ở các claim trước (T13, T14) và ghi audit `policy_denied` nếu từ chối. Khi gặp `InjectedCrash`, shape ghi sự kiện `crash`, flush file trace rồi cho ngoại lệ lan truyền.
- **Cơ chế Sanitize Handoff và Trace Contract v1:**
  - Thay vì dùng LLM tóm tắt ngữ cảnh (không đảm bảo loại bỏ triệt để secret), `make_handoff` kết hợp `sanitize` lọc cứng theo allowlist `ALLOWED_FIELDS` (`contract_version`, `request_id`, `claim_id`, `principal_scope`, `task`, `facts` theo `FACT_KEYS`, `allowed_tools`, `max_output_tokens`) và chặn cứng nếu chuỗi JSON còn chứa `CANARY_SECRET_*`, `parent_secret`, `raw_headers` hoặc tool ghi.
  - Trace được ghi mới ở mọi nhánh trả về thông qua `common.finish()`, gồm các event có cấu trúc: `handoff` (gói `handoff.v1` đã lọc), `investigator` (`lookup_order` chỉ gồm `ORDER_FIELDS` và `policy_count`), `quote`, `decision`, `error` và `crash`. Các trường nhạy cảm (`parent_secret`, `raw_headers`, `ticket` thô, `phone`, `address`, `item_name`, `internal_note`) cố ý không được đưa vào trace hay `stdout`.

## 4. Kết quả

Nhóm chạy bốn shape trên cùng 14 claim public và seed `11,22,33,44,55`. Số liệu dưới đây lấy từ `src/reports/benchmark.json`; chi phí là VND mô phỏng.

| Shape | Pass rate | Pass^5 | Seed min–max | Variance | Cost/success | Steps mean/p95 |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Workflow | 70/70 | 14/14 | 1,000–1,000 | 0 | 27,86 | 4,43 / 5 |
| Single | 70/70 | 14/14 | 1,000–1,000 | 0 | 30,36 | 2,79 / 3 |
| Orchestrator | 70/70 | 14/14 | 1,000–1,000 | 0 | 47,90 | 3,71 / 4 |
| Swarm | 69/70 | 13/14 | 0,929–1,000 | 0,000816 | 49,57 | 2,80 / 3 |

`python -m pytest -q` đạt 82 test host pass; bộ student đạt 248/248. `src/reports/crash.json` ghi `crash_safety` 20/20. Swarm trượt C-002 ở seed 55 vì tin số ngày trễ trong ticket; latency là `unavailable`.

**Tự kiểm trên bộ hidden không nhãn (`student_drop/` → `src/reports/hidden_selfcheck.json`):** Cả ba shape `workflow`, `single` và `orchestrator` đều hoàn thành trọn vẹn 40 claim × 5 seed = 200 trial/shape với `error_trials = 0`, `leak_suspect_trials = 0`, 100% trace đạt chuẩn `Trace contract v1` và thống nhất tuyệt đối về phân bố trạng thái trên toàn bộ 200 cặp `(claim_id, seed)`: `denied: 95`, `resolved: 90`, `escalated: 10`, `needs_info: 5` (số bước trung bình/p95 lần lượt là `4,53 / 5`, `2,90 / 3`, `3,88 / 4`; tổng chi phí mô phỏng `5.800,0`, `6.295,6` và `9.982,6 VND`). Kết quả pass rate chính thức trên bộ hidden do grader riêng của giảng viên chấm.

Quyết định kiến trúc và lý do: xem `report/team/decision_adr.md`. Phiếu nghiên cứu và ablation: xem `report/team/research_card.md`.

## 5. Khó khăn và cách xử lý

- **Lỗi `403` khi `git push` trên Windows dù tài khoản đã có quyền Collaborator (`write`):** Khi chạy `git push origin main`, Git báo `remote: Permission to Basrot/K5-L3B-DAY08-NNHT-FromWorkflowsToAgents.git denied to daongochan (403)`. Nguyên nhân là cấu hình Git toàn cục gọi `gh auth git-credential`, ưu tiên đọc biến môi trường `GITHUB_TOKEN` đang chứa Fine-grained PAT (chỉ có hiệu lực trên repo cá nhân của `daongochan`, không ghi được vào repo của `Basrot`). Nhóm xử lý bằng cách cấu hình cục bộ `.git/config` dùng trực tiếp `helper = manager` với tài khoản OAuth trong Git Credential Manager và thêm cờ `-4` khi kết nối IPv6 tới `github.com:443` bị timeout.
- **Khác biệt phiên bản Python trên máy cá nhân:** Một số máy cài sẵn Python 3.15 làm mặc định hệ thống (vượt khoảng `3.11–3.13` của đề bài). Nhóm tạo môi trường ảo bằng `uv venv --python 3.13 .venv` trong thư mục `src/` để cố định Python `3.13.5`, chạy `python -m lab8.smoke` đạt ngay từ đầu.
- **Xung đột gộp `TEAM.md` khi nhiều thành viên push song song:** Trong quá trình gộp nhánh (`1e85ef7`), file `TEAM.md` bị dính dấu xung đột stash (`<<<<<<< Updated upstream`). Nhóm phát hiện khi rà soát tích hợp, gộp lại đầy đủ thông tin của cả 4 thành viên ở commit `99cafcd` và kiểm tra lại bằng `git diff --check` cùng `python -m lab8.check_submission`.

## 6. Tổng hợp đóng góp

Chi tiết đóng góp của từng thành viên nằm trong `TEAM.md` (bảng đóng góp) và các báo cáo cá nhân trong `report/team/` (`2A202603788_ThieuTheNgoc.md`, `2A202603732_TranTrongDucTai.md`, `2A202603779_DaoNgocHan.md`, `2A202603761_NguyenGiaNam.md`).

## 7. Hạn chế và hướng cải thiện

- **Giới hạn của môi trường mô phỏng:** `MockLLM` hoàn toàn tất định theo `seed`, không phản ánh sai số sinh văn bản tự nhiên, đứt gãy định dạng JSON/tool call hay giới hạn cửa sổ ngữ cảnh của mô hình thật; độ trễ thực thi (`latency`) là `unavailable` và chi phí VND chỉ là giá mô phỏng từ `src/config/pricing.yaml`.
- **Giới hạn của dữ liệu đánh giá:** Bộ public chỉ gồm 14 claim (70 trial/shape) và các trường nghiệp vụ cốt lõi (`delay_days`, `damage_verified`, `status`) đã được cấu trúc hóa sẵn trong bảng `orders`, nên `workflow` cố định đạt 70/70 mà chưa cần khả năng lập kế hoạch linh hoạt. Nhóm chưa có nhãn của bộ hidden 40 claim (do grader riêng của giảng viên chấm).
- **Điều kiện đảo quyết định và hướng cải thiện:** Nếu trong bài toán thực tế, bằng chứng giao trễ hoặc biên bản xác minh hỏng hàng nằm phân tán ở nhiều nguồn bán cấu trúc khiến luồng bước cứng không bao phủ hết, nhóm sẽ chuyển sang `single` hoặc `orchestrator` (vẫn giữ allowlist `handoff.v1` và cổng host kiểm tiền/approval bằng code). Hướng mở rộng tiếp theo là bổ sung cơ chế đối soát hai pha (`reserve -> provider -> confirm`) cho cổng thanh toán ngoài và đánh giá lại với LLM thật có đo latency p95.

## 8. Tài liệu tham khảo

1. Đề bài và tài liệu kỹ thuật Lab 8 LogiX-VN: `README.md`, `docs/business_rules.md`, `docs/contracts.md`, `docs/architecture.md`, `docs/metrics.md`.
2. Cemri, M. et al. (2025). *Why Do Multi-Agent LLM Systems Fail?* (MAST). NeurIPS 2025 Datasets and Benchmarks Track. https://proceedings.neurips.cc/paper_files/paper/2025/hash/b1041e52d3be19f0a9bc491657488e4a-Abstract-Datasets_and_Benchmarks_Track.html
3. Tài liệu bài giảng Ngày 8 AI20K K5 — *From Workflows to Agents* (nguyên tắc chọn mức tự chủ tối thiểu, durable ledger, typed handoff allowlist và kiểm soát biến nhiễu khi benchmark hệ đa tác tử).
