# Báo cáo cá nhân — 2A202603761_NguyenGiaNam

## 1. Thông tin thành viên

- Họ tên: Nguyễn Gia Nam
- MSSV: 2A202603761
- Email: 26ai.namng@vinuni.edu.vn
- GitHub: Basrot

## 2. Phần việc phụ trách

- Kiểm thử ledger, approval, crash và đánh giá thực nghiệm cho nhóm.

## 3. Đóng góp cụ thể

- Commit kỹ thuật `e67183f`: thêm hai test trong `src/tests/test_ledger_order_budget.py`.
- Test `test_approval_and_order_caps_apply_across_successive_claims` kiểm approval thấp hơn quote, phần entitlement còn lại của đơn và retry không ghi thêm tiền.
- Test `test_repeated_expired_approval_denial_never_writes_refund` kiểm approval hết hạn không tạo refund hoặc audit committed khi gọi lại.
- Chạy crash suite chính thức và tạo `src/reports/crash.json` bằng công cụ của lab.

## 4. Bằng chứng

- Commit: https://github.com/Basrot/K5-L3B-DAY08-NNHT-FromWorkflowsToAgents/commit/e67183f (link hoạt động sau khi push).
- Lệnh test: `python -m pytest -q` trong `src/` → `56 passed, 248 deselected`.
- Lệnh crash: `python -m lab8.crash_test --count 20 --out reports/crash.json` trong `src/` → `crash_safety 20/20`.
- Benchmark bốn shape và quyết định kiến trúc sẽ bổ sung sau khi ba shape của nhóm hoàn thiện.

## 5. Công cụ AI đã dùng

- Codex hỗ trợ đọc contract, viết test và chạy kiểm chứng trên Python 3.11. Tôi cần tự đọc lại test và chạy lại các lệnh ở mục 4 trước khi nộp.

## 6. Tự đánh giá

- Đã hoàn thành phần kiểm thử host và bằng chứng crash độc lập. Cần tiếp tục kiểm các shape do nhóm viết và chạy benchmark cuối cùng khi tích hợp xong.
