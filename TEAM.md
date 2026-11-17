# Team

## Thông tin nhóm

- Tên nhóm: NNHT
- Lớp: L3B
- Repo nhóm: `K5-L3B-DAY08-NNHT-FromWorkflowsToAgents` (xem [SUBMISSION.md](SUBMISSION.md))
- URL repo, nhánh nộp, commit chốt: https://github.com/Basrot/K5-L3B-DAY08-NNHT-FromWorkflowsToAgents (nhánh `main`, commit chốt cập nhật ở CP5)

## Thành viên

| STT | Họ tên             | MSSV        | Email                     | GitHub     | Vai trò                |
| --: | ------------------ | ----------- | ------------------------- | ---------- | ---------------------- |
|   1 | Thiều Thế Ngọc     | 2A202603788 | 26ai.ngoctt@vinuni.edu.vn | BFNGOC2    | workflow + integration |
|   2 | Trần Trọng Đức Tài | 2A202603732 | 26ai.taittd@vinuni.edu.vn | DucTai2804 | single                 |
|   3 | Đào Ngọc Hân       | 2A202603779 | 26ai.handn@vinuni.edu.vn  | daongochan | orchestrator + handoff |
|   4 | Nguyễn Gia Nam     | 2A202603761 | 26ai.namng@vinuni.edu.vn  | Basrot     | ledger + evaluation    |

Gợi ý chia vai:

- Nhóm 6 người: integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report.
- Nhóm 4 người: workflow + integration; single; orchestrator + handoff; ledger + evaluation.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người              | Vai                    | Commit    | Test đã thêm hoặc sửa                                                                                                                                                                               |
| ------------------ | ---------------------- | --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Thiều Thế Ngọc     | workflow + integration | `837c75f` | `test_workflow_full_quote_is_capped_once_and_retry_is_idempotent`                                                                                                                                   |
| Trần Trọng Đức Tài | single                 | 41669db   | `test_single_agent_ignores_ticket_hallucination`; `test_single_agent_trace_safety_and_contract`                                                                                                     |
| Đào Ngọc Hân       | orchestrator + handoff | fb10ddd   | `test_orchestrator_handoff_allowlist_and_no_secret_or_pii_leak`; `test_orchestrator_ignores_untrusted_ticket_claims_across_all_seeds`; `test_orchestrator_crash_and_resume_preserves_single_refund` |
| Nguyễn Gia Nam     | ledger + evaluation    | e67183f   | `test_approval_and_order_caps_apply_across_successive_claims`; `test_repeated_expired_approval_denial_never_writes_refund`                                                                          |

## Cá nhân

Mỗi thành viên đồng thời tạo một báo cáo riêng trong `report/` theo dạng:

```text
report/team/MSSV_HoTen.md
```

Ví dụ: `report/team/20241234_NguyenVanAn.md`.

Sao chép mục dưới đây cho từng thành viên.

## 2A202603788_ThieuTheNgoc

### Phần việc phụ trách

- Shape A (`workflow`) và integration; đã kiểm tra tích hợp cả bốn shape trên bộ public.

### Đóng góp cụ thể

- Commit `837c75f` (`feat: CP2-3`) triển khai Workflow A trong `src/lab8/shapes/workflow.py`.
- Rà soát contract các shape, chạy test toàn hệ thống, crash suite, benchmark public và quét contract/chuỗi cấm trên 280 trace. Không phát hiện lỗi tích hợp cần sửa, không sửa shape của thành viên khác.

### Bằng chứng

- Commit `837c75f` (`src/lab8/shapes/workflow.py`) và `ba4b326` (`src/tests/test_workflow.py`).
- Host tests: 82 passed, 248 deselected; student tests: 248 passed, 82 deselected.
- Crash suite: 20/20. Public benchmark: workflow 70/70, single 70/70, orchestrator 70/70, swarm 69/70.
- `python -m lab8.check_submission`: 0 FAIL, 0 WARN.

## 2A202603732_TranTrongDucTai

### Phần việc phụ trách

- Shape B (`single` ReAct agent).

### Đóng góp cụ thể

- Cài đặt vòng lặp ReAct (`run_react`) với chặn cứng `max_steps=8` trong `src/lab8/shapes/single.py`.
- Thiết kế hàm `decide(seen)` tra cứu đơn hàng và tính quyền lợi bám sát DB, chống hallucination ngày trễ từ ticket khách hàng.
- Xử lý các nhánh `needs_info`, `escalated`, `denied`, `resolved`, truyền `crash_at` vào `durability.issue_refund` và flush trace trước khi crash.

### Bằng chứng

- Commit `41669db` (`src/lab8/shapes/single.py`, `src/tests/test_single_agent.py`, `report/team/2A202603732_TranTrongDucTai.md`).
- Test: `test_single_agent_ignores_ticket_hallucination`, `test_single_agent_trace_safety_and_contract`.
- Kết quả: `pytest -k "single]"` đạt 62/62 passed.

## 2A202603779_DaoNgocHan

### Phần việc phụ trách

- Shape C (`orchestrator`) và cơ chế lọc dữ liệu chuyển giao (`handoff`).

### Đóng góp cụ thể

- Cài đặt Shape C (`src/lab8/shapes/orchestrator.py`) với `make_handoff` allowlist, Investigator chỉ có quyền đọc (`lookup_order`, `lookup_policy`), truyền quote đầy đủ và `crash_at` vào `durability.issue_refund`, ghi trace đúng Trace contract v1 không lộ secret/PII.
- Thêm bộ kiểm thử `src/tests/test_orchestrator_handoff.py` (21 test case) cho handoff allowlist, kháng ticket không đáng tin trên 5 seed, trần hoàn tiền theo đơn và khôi phục sau crash.

### Bằng chứng

- Commit kỹ thuật: `fb10ddd` (khởi tạo `TEAM.md`: `50b9db9`, `ded3408`).
- `src/lab8/shapes/orchestrator.py`; `src/tests/test_orchestrator_handoff.py` (`62/62` test student `orchestrator` pass).
- Báo cáo cá nhân: `report/team/2A202603779_DaoNgocHan.md`.

## 2A202603761_NguyenGiaNam

### Phần việc phụ trách

- Kiểm thử độ bền giao dịch / chính sách (`ledger`, `policy`, `crash`) và đánh giá thực nghiệm (`evaluation` / báo cáo).

### Đóng góp cụ thể

- Thêm hai test host cho hạn mức hoàn theo đơn, giới hạn approval và retry sau khi approval hết hạn.
- Chạy crash suite 20 điểm và lưu báo cáo do công cụ sinh ra.

### Bằng chứng

- Commit kỹ thuật: `e67183f`.
- `src/tests/test_ledger_order_budget.py`; `src/reports/crash.json` (`crash_safety: 20/20`).
