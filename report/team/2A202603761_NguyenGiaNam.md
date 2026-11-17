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
- Đo baseline riêng cho swarm tham chiếu trên 14 claim public × 5 seed, lưu `src/reports/swarm_baseline.json` do công cụ của lab tạo.

## 4. Bằng chứng

- Commit: https://github.com/Basrot/K5-L3B-DAY08-NNHT-FromWorkflowsToAgents/commit/e67183f (link hoạt động sau khi push).
- Lệnh test: `python -m pytest -q` trong `src/` → `56 passed, 248 deselected`.
- Lệnh crash: `python -m lab8.crash_test --count 20 --out reports/crash.json` trong `src/` → `crash_safety 20/20`.
- Lệnh baseline: `python -m lab8.benchmark --shapes swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/swarm_baseline.json` trong `src/` → pass rate `69/70 = 0.9857`, pass^5 `13/14 = 0.9286`, seed variance `0.000816`, cost/success mô phỏng `49.57 VND`.
- Trial fail của swarm: `R-swarm-C-002-s55`, `status resolved != denied`; seed 55 có pass rate `13/14`, bốn seed còn lại `14/14`.
- `python -m lab8.check_submission` hiện báo `12 FAIL` vì ba shape còn `not_implemented`, thiếu benchmark bốn shape và ADR/research card của nhóm chưa điền; đây chưa phải trạng thái nộp bài.
- Benchmark bốn shape và quyết định kiến trúc sẽ bổ sung sau khi ba shape của nhóm hoàn thiện.

## 5. Công cụ AI đã dùng

- Codex hỗ trợ đọc contract, viết test và chạy kiểm chứng trên Python 3.11. Tôi cần tự đọc lại test và chạy lại các lệnh ở mục 4 trước khi nộp.

## 6. Tự đánh giá

- Đã hoàn thành phần kiểm thử host và bằng chứng crash độc lập. Cần tiếp tục kiểm các shape do nhóm viết và chạy benchmark cuối cùng khi tích hợp xong.
