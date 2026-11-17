# Báo cáo cá nhân — 2A202603779_DaoNgocHan

## 1. Thông tin thành viên

- Họ tên: Đào Ngọc Hân
- MSSV: 2A202603779
- Email: daongochan1505@gmail.com
- GitHub: daongochan

## 2. Phần việc phụ trách

- Thiết kế và cài đặt **Shape C (`orchestrator`)** tại `src/lab8/shapes/orchestrator.py`, cơ chế lọc dữ liệu chuyển giao qua allowlist (`handoff.v1`) và đảm bảo an toàn nhật ký chạy (`Trace contract v1`).
- Tổng hợp thông tin thành viên và phân công vai trò ban đầu trong `TEAM.md` (CP1).

## 3. Đóng góp cụ thể

- Triển khai đầy đủ `run_claim` cho Shape C trong `src/lab8/shapes/orchestrator.py` (commit `fb10ddd`):
  - Đọc claim qua `get_claim_view` thay vì `SELECT *` để không chạm `parent_secret` và `raw_headers`; xử lý nhánh `unknown_claim` (`failed`) và thiếu `order_id` (`needs_info`, không trích mã đơn từ nội dung `ticket`).
  - Dựng gói dữ liệu sạch bằng `lab8.handoff.make_handoff` chỉ chứa các trường trong `ALLOWED_FIELDS` và chỉ cấp quyền hai tool đọc `["lookup_order", "lookup_policy"]` cho `run_readonly_investigators` theo đúng `principal_id` của claim (trả `escalated` khi đơn không tồn tại hoặc khác chủ sở hữu, không lộ PII).
  - Giữ chặt ranh giới code / model: `MockLLM` chỉ dùng cho bước lập kế hoạch (`purpose="plan"`) và soạn nháp phản hồi (`purpose="draft"`), tuyệt đối không lấy `delay_days` từ `ticket` hay tự tính tiền; Orchestrator tính quote tất định qua `compute_compensation` từ dữ liệu đơn có cấu trúc.
  - Truyền **quote đầy đủ** và tham số `crash_at` vào `durability.issue_refund` để host kiểm approval, trừ phần đơn đã hoàn trước đó (T13, T14) và ghi audit `policy_denied`; khi gặp `InjectedCrash` thì ghi sự kiện `crash`, flush file trace rồi cho exception lan truyền.
  - Ghi trace qua `common.finish()` trên mọi nhánh trả về, cố ý loại bỏ `parent_secret`, `raw_headers`, câu chữ `ticket` thô và các trường PII của đơn hàng.
- Viết mới bộ test `src/tests/test_orchestrator_handoff.py` (5 hàm kiểm thử / 21 test case tham số hóa) kiểm chứng allowlist `handoff.v1`, chống lộ secret/PII, kháng ticket khai khống / điều khoản hết hiệu lực / prompt injection trên cả 5 seed (`11, 22, 33, 44, 55`), truyền quote đầy đủ cho trần theo đơn và khôi phục sau 3 điểm crash.

## 4. Bằng chứng

- Commit cập nhật `TEAM.md` (CP1): `50b9db9`, `ded3408`.
- Commit kỹ thuật Shape C + test (CP2/CP3): `fb10ddd`.
- File mã nguồn và kiểm thử:
  - `src/lab8/shapes/orchestrator.py`
  - `src/tests/test_orchestrator_handoff.py`: `test_orchestrator_handoff_allowlist_and_no_secret_or_pii_leak`, `test_orchestrator_ignores_untrusted_ticket_claims_across_all_seeds`, `test_orchestrator_passes_full_quote_for_caps_and_denials`, `test_orchestrator_crash_and_resume_preserves_single_refund`, `test_orchestrator_unknown_claim_fails_with_valid_trace`.
- Kết quả chạy kiểm chứng trên máy:
  - Cổng host sau khi tích hợp với nhóm (`uv run python -m pytest -q`): `79 passed, 248 deselected` (`0 failed`).
  - Ma trận student cho `orchestrator` (`uv run python -m pytest -q -m student -o addopts= -k orchestrator`): `62 passed` (`0 failed`, đạt đủ T01–T14 trên 5 seed, gồm cả T11 retry và T12 crash-resume).
  - Rà soát an toàn trace trên các shape đã có (`single`, `orchestrator`, `swarm`) trên 14 claim public × 5 seed (`11,22,33,44,55`): `orchestrator` đạt `pass_rate = 1.000` (70/70 trial, gồm cả `R-orchestrator-C-002-s55` nơi `swarm` trượt), `pass_5 = 1.000`, `seed_variance = 0.0`, `cost_per_success_vnd = 47.90`; 0 rò rỉ `CANARY_SECRET_*` / PII ở cả `single` (`30.36 VND`) và `orchestrator` *(số liệu chính thức đủ 4 shape sẽ chốt từ `src/reports/benchmark.json` ở CP5)*.

## 5. Công cụ AI đã dùng

- Sử dụng Google Antigravity (Gemini) để tra cứu nhanh luật nghiệp vụ trong `docs/business_rules.md`, `docs/contracts.md`, hỗ trợ triển khai `src/lab8/shapes/orchestrator.py` và xây dựng các kịch bản test trong `src/tests/test_orchestrator_handoff.py`.
- Cách tự kiểm tra kết quả: tự đọc rà từng nhánh trả về trong `orchestrator.py` để chắc chắn Investigator không có tool ghi, trace không lưu `ticket` hay bản ghi DB thô, `proposal` truyền nguyên vẹn vào `issue_refund`; sau đó chạy trực tiếp `pytest -q`, `pytest -q -m student -o addopts= -k orchestrator` và `lab8.benchmark` trên môi trường `uv` (Python 3.13.5) để kiểm chứng đối chiếu với SQLite ledger.

## 6. Tự đánh giá

<!-- Bạn tự viết vài câu đánh giá ngắn tại đây sau khi tự kiểm tra (theo RULES.md không dùng AI viết thay mục Tự đánh giá). -->
