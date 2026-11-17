# Báo cáo cá nhân — 2A202603761_NguyenGiaNam

## 1. Thông tin thành viên

- Họ tên: Nguyễn Gia Nam
- MSSV: 2A202603761
- Email: 26ai.namng@vinuni.edu.vn
- GitHub: Basrot

## 2. Phần việc phụ trách

- Mình phụ trách kiểm tra ledger, approval, crash và phần số liệu đánh giá.

## 3. Đóng góp cụ thể

- Thêm hai test về giới hạn hoàn tiền theo đơn và approval hết hạn trong `src/tests/test_ledger_order_budget.py`.
- Chạy crash suite và benchmark riêng cho swarm để có số liệu ban đầu. Mình chưa đánh giá ba shape nhóm đang viết.

## 4. Bằng chứng

- Commit test: `e67183f`.
- Test đã thêm: `test_approval_and_order_caps_apply_across_successive_claims` và `test_repeated_expired_approval_denial_never_writes_refund`.
- `python -m pytest -q`: 56 passed, 248 deselected. `python -m lab8.crash_test --count 20 --out reports/crash.json`: 20/20.
- `src/reports/swarm_baseline.json`: swarm pass 69/70 trial; trial trượt là `R-swarm-C-002-s55`. Đây chỉ là baseline, chưa phải benchmark bốn shape.

## 5. Công cụ AI đã dùng

- Mình dùng Codex để đọc yêu cầu, hỗ trợ viết test và báo cáo. Sau đó mình tự chạy lại các lệnh ở mục 4 trên máy: 2 test mới pass, cổng host 56 test pass và crash suite đạt 20/20.

## 6. Tự đánh giá

Mình đã hoàn thành hai test cho phần ledger và approval, sau đó tự chạy lại trên máy. Hai test mới đều pass, cổng host có 56 test pass và crash suite đạt 20/20 ở lần kiểm tra đó. Phần đánh giá của mình vẫn chưa xong vì chưa kiểm thử tích hợp và benchmark đủ bốn shape; khi nhóm hoàn thiện các shape còn lại, mình sẽ làm tiếp.
