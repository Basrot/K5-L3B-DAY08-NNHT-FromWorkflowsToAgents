# Báo cáo nhóm — Lab 8 LogiX-VN

Tối đa 5 trang ngoài phụ lục. Số liệu lấy từ file trong `src/reports/`, không chép tay từ terminal.

## 1. Thông tin nhóm

- Tên nhóm: NNHT (Lớp L3B)
- Thành viên: xem `TEAM.md`
- Commit chốt (trùng `report/team/decision_adr.md`): `99cafcd`

## 2. Mục tiêu và cách tiếp cận

Nhóm hiểu bài toán thế nào, chia việc ra sao, thứ tự làm bốn shape.

...

## 3. Giải pháp

Mô tả ba shape nhóm viết (workflow, single, orchestrator): mỗi shape đọc gì, quyết định gì, gọi host ra sao. Chỉ rõ ranh giới code / model (tham khảo [docs/architecture.md](../../docs/architecture.md)), cách handoff được sanitize và trace ghi những event nào.

### 3.1 Shape A — `workflow` (`src/lab8/shapes/workflow.py`)

...

### 3.2 Shape B — `single` (`src/lab8/shapes/single.py`)

...

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

`python -m pytest -q` đạt 81 test host pass; bộ student đạt 248/248. `src/reports/crash.json` ghi `crash_safety` 20/20. Swarm trượt C-002 ở seed 55 vì tin số ngày trễ trong ticket. Nhóm chưa có kết quả hidden; latency là `unavailable`.

Quyết định kiến trúc và lý do: xem `report/team/decision_adr.md`. Phiếu nghiên cứu và ablation: xem `report/team/research_card.md`.

## 5. Khó khăn và cách xử lý

Lỗi gặp phải (kèm lệnh, output hoặc ảnh chụp) và cách nhóm xử lý.

...

## 6. Tổng hợp đóng góp

Chi tiết đóng góp của từng thành viên nằm trong `TEAM.md` (bảng đóng góp) và báo cáo cá nhân `report/team/MSSV_HoTen.md`.

## 7. Hạn chế và hướng cải thiện

MockLLM không phải model sản xuất; latency là `unavailable`. Nêu thêm giới hạn của nhóm và điều kiện làm kết luận thay đổi.

...

## 8. Tài liệu tham khảo

...
