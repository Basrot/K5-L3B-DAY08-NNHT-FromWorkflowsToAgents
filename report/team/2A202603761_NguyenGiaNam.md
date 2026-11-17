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
- Chạy crash suite, kiểm các shape sau khi nhóm ghép code và chạy benchmark đủ bốn shape.
- Làm ablation R2: so swarm gốc với bản chỉ dùng số ngày trễ từ order, rồi ghi kết quả vào research card.
- Từ kết quả benchmark public, đề xuất workflow vì cùng đạt 70/70 trial như single và orchestrator nhưng có mức tự chủ và chi phí mô phỏng thấp hơn.

## 4. Bằng chứng

- Commit test: `e67183f`.
- Test đã thêm: `test_approval_and_order_caps_apply_across_successive_claims` và `test_repeated_expired_approval_denial_never_writes_refund`.
- Em tự chạy hai test mới, cổng host và crash suite: 2/2 pass, 56 host tests pass, crash 20/20. Sau khi nhóm ghép đủ shape, Codex chạy lại: 81 host tests và 248 student tests pass.
- `src/reports/benchmark.json`: workflow, single, orchestrator đều pass 70/70 trial; swarm pass 69/70. Workflow có cost/success mô phỏng thấp nhất: 27,86 VND.
- `src/reports/r2_ablation.json`: swarm gốc pass 69/70, bản `swarm_order_only` pass 70/70. Test ablation: `src/tests/test_swarm_ticket_ablation.py`.

## 5. Công cụ AI đã dùng

- Em dùng Codex để đọc yêu cầu, hỗ trợ viết test, ablation và báo cáo. Em tự chạy lại hai test ledger, cổng host và crash suite; kết quả lần lượt là 2/2 test pass, 56 host tests pass và crash 20/20. Codex hỗ trợ chạy benchmark cuối và ablation; số liệu được lấy từ các file JSON do công cụ của lab tạo, không sửa tay.

## 6. Tự đánh giá

Em đã hoàn thành phần kiểm thử ledger, approval, crash và đánh giá bốn shape theo phân công. Qua hai test đã thêm, em kiểm tra được rằng shape phải truyền quote đầy đủ cho host; host sẽ giới hạn số tiền theo approval và tổng refund của đơn, đồng thời retry không được tạo thêm refund. Trên bộ public, workflow, single và orchestrator đều pass 70/70 trial, nhưng workflow có chi phí mô phỏng thấp nhất và mức tự chủ thấp nhất nên nhóm chọn workflow. Kết luận này chỉ áp dụng cho dữ liệu giả và MockLLM của lab; kết quả hidden có thể làm quyết định thay đổi.
