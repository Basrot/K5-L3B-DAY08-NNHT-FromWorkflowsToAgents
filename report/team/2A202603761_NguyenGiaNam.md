# Báo cáo cá nhân — 2A202603761_NguyenGiaNam

## 1. Thông tin thành viên

- Họ tên: Nguyễn Gia Nam
- MSSV: 2A202603761
- Email: 26ai.namng@vinuni.edu.vn
- GitHub: Basrot

## 2. Phần việc phụ trách

- Em phụ trách kiểm tra ledger, approval, crash và phần số liệu đánh giá.

## 3. Đóng góp cụ thể

- Thêm hai test về giới hạn hoàn tiền theo đơn và approval hết hạn trong `src/tests/test_ledger_order_budget.py`.
- Chạy crash suite, đo swarm và benchmark tạm ba shape đã có. Workflow chưa hoàn thiện nên chưa chạy benchmark cuối.

## 4. Bằng chứng

- Commit test: `e67183f`.
- Test đã thêm: `test_approval_and_order_caps_apply_across_successive_claims` và `test_repeated_expired_approval_denial_never_writes_refund`.
- `python -m pytest -q`: 56 passed, 248 deselected. `python -m lab8.crash_test --count 20 --out reports/crash.json`: 20/20.
- `src/reports/swarm_baseline.json`: swarm pass 69/70 trial; trial trượt là `R-swarm-C-002-s55`. Đây chỉ là baseline, chưa phải benchmark bốn shape.
- `src/reports/partial_benchmark.json`: cùng 14 claim và 5 seed, single và orchestrator đều pass 70/70, swarm pass 69/70. Chi phí mô phỏng mỗi lần thành công lần lượt là 30,36; 47,90; 49,57 VND. Đây là kết quả tạm, chưa có workflow.

## 5. Công cụ AI đã dùng

- Em dùng Codex để đọc yêu cầu, hỗ trợ viết test và báo cáo. Sau đó em tự chạy lại hai test mới, cổng host và crash suite trên máy: 2 test pass, 56 test host pass và crash 20/20. Codex chạy phần benchmark tạm; em cần tự xem lại số liệu trước khi nộp.

## 6. Tự đánh giá

Em đã hoàn thành hai test cho phần ledger và approval, sau đó tự chạy lại trên máy. Hai test mới đều pass, cổng host có 56 test pass và crash suite đạt 20/20 ở lần kiểm tra đó. Em đã có kết quả tạm của single, orchestrator và swarm; phần còn lại là kiểm workflow và chạy benchmark đủ bốn shape khi nhóm hoàn thiện.
