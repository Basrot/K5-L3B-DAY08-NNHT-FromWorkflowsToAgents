# Báo cáo cá nhân — 2A202603732_TranTrongDucTai

## 1. Thông tin thành viên

- Họ tên: Trần Trọng Đức Tài
- MSSV: 2A202603732
- Email: 26ai.taittd@vinuni.edu.vn
- GitHub: DucTai2804

## 2. Phần việc phụ trách

- Phụ trách chính: Shape B (`single` ReAct agent) trong `src/lab8/shapes/single.py`.
- Thiết kế vòng lặp ReAct (`run_react`) có giới hạn `max_steps = 8`.
- Đảm bảo ranh giới bảo mật tài chính: agent chỉ được gọi read tools (`lookup_order`, `compute_compensation`), tiền do code của host (`durability.issue_refund`) kiểm duyệt và ghi sổ nợ (ledger).
- Xử lý độ bền giao dịch và sự cố crash (`InjectedCrash`, `crash_at`) theo đúng chuẩn hợp đồng.

## 3. Đóng góp cụ thể

- Cài đặt toàn bộ hàm `run_claim` cho Shape B (`single.py`):
  - Phân tích ticket và claim view ban đầu.
  - Xử lý thiếu `order_id` trả về `needs_info`.
  - Hàm `decide(seen)` tuần tự tra cứu đơn hàng và tính entitlement bồi hoàn dựa trên dữ liệu thật trong DB, miễn nhiễm với hallucination hoặc khai khống ngày trễ từ ticket khách hàng.
  - Phân quyền escalations khi đơn hàng không tồn tại hoặc sai phạm vi sở hữu khách hàng.
  - Gọi `issue_refund` với quote đầy đủ (full quote), để host tự xử lý trần theo đơn và kiểm tra approval.
- Viết bộ unit test riêng `src/tests/test_single_agent.py`:
  - `test_single_agent_ignores_ticket_hallucination`: Kiểm thử tính kháng lỗi của Single Agent trên toàn bộ 5 seeds (kể cả seed 55 nơi reference swarm bị đánh lừa).
  - `test_single_agent_trace_safety_and_contract`: Kiểm tra hợp đồng Trace Contract v1 và đảm bảo không có rò rỉ canary secret hay PII vào trace.

## 4. Bằng chứng

- File mã nguồn: `src/lab8/shapes/single.py`
- File test bổ sung: `src/tests/test_single_agent.py`
- Kết quả test kiểm thử:
  - `pytest -q tests/test_single_agent.py`: `2 passed`
  - `pytest -q -m student -o addopts= -k "single]"`: `62 passed` (vượt qua 100% test cases T01–T14 của Single Agent).
- Số liệu Public Benchmark (`src/reports/benchmark.json`):
  - Pass rate: **70/70 (1.000)** trên 14 claims x 5 seeds.
  - Pass^5: **14/14 (1.000)**; Seed min-max: 1.000–1.000; Seed variance: **0.000000** (hoàn toàn ổn định, không bị trôi dạt như Swarm).
  - Cost per success (mô phỏng): **30.36 VND** (thấp thứ hai trong 4 shape, chỉ sau workflow 27.86 VND và rẻ hơn nhiều so với orchestrator 47.90 VND và swarm 49.57 VND).
  - Số bước trung bình / p95: **2.79 / 3 bước** (nằm sâu trong giới hạn an toàn `max_steps = 8`).
  - Crash safety: Đạt `20/20` (`src/reports/crash.json`), bảo toàn tính idempotency và không có rò rỉ secret/PII.
- Tự kiểm tra với bộ chấm: `python -m lab8.check_submission` đạt **0 FAIL, 0 WARN**.

## 5. Công cụ AI đã dùng

- Công cụ: Antigravity Assistant (Gemini 3.8 Flash).
- Mục đích: Hỗ trợ rà soát luồng ReAct, gợi ý cấu trúc trace hợp lệ theo Trace Contract v1, hỗ trợ tạo template test cho `pytest`.
- Cách tự kiểm tra kết quả: Tự chạy toàn bộ test suite bằng `pytest`, soi kỹ nội dung file trace `.json` sinh ra trong thư mục `traces/` để đảm bảo sạch secret và PII trước khi xác nhận.

## 6. Tự đánh giá

- Hoàn thành đầy đủ các tiêu chuẩn của Shape B theo yêu cầu CP2, CP3, CP4.
- Điểm tự đánh giá: Hoàn thành tốt phần việc được phân công.
