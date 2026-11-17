# Team

## Thông tin nhóm

- Tên nhóm: NNHT
- Lớp: L3B
- Repo nhóm: `K5-L3B-DAY08-NNHT-FromWorkflowsToAgents` (xem [SUBMISSION.md](SUBMISSION.md))
- URL repo, nhánh nộp, commit chốt: https://github.com/Basrot/K5-L3B-DAY08-NNHT-FromWorkflowsToAgents (nhánh `main`, commit chốt cập nhật ở CP5)

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| --: | ------ | ---- | ----- | ------ | ------- |
|   1 | Thiều Thế Ngọc | 2A202603788 | 26ai.ngoctt@vinuni.edu.vn | BFNGOC2 | workflow + integration |
|   2 | Trần Trọng Đức Tài | 2A202603732 | 26ai.taittd@vinuni.edu.vn | DucTai2804 | single |
|   3 | Đào Ngọc Hân | 2A202603779 | daongochan1505@gmail.com | daongochan | orchestrator + handoff |
|   4 | Nguyễn Gia Nam | 2A202603761 | 26ai.namng@vinuni.edu.vn | Basrot | ledger + evaluation |

Gợi ý chia vai:

- Nhóm 6 người: integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report.
- Nhóm 4 người: workflow + integration; single; orchestrator + handoff; ledger + evaluation.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| --- | --- | --- | --- |
| Thiều Thế Ngọc | workflow + integration | | |
| Trần Trọng Đức Tài | single | | |
| Đào Ngọc Hân | orchestrator + handoff | | |
| Nguyễn Gia Nam | ledger + evaluation | e67183f | `test_approval_and_order_caps_apply_across_successive_claims`; `test_repeated_expired_approval_denial_never_writes_refund` |

## Cá nhân

Mỗi thành viên đồng thời tạo một báo cáo riêng trong `report/` theo dạng:

```text
report/team/MSSV_HoTen.md
```

Ví dụ: `report/team/20241234_NguyenVanAn.md`.

Sao chép mục dưới đây cho từng thành viên.

## 2A202603788_ThieuTheNgoc

### Phần việc phụ trách

- Shape A (`workflow`) và tích hợp chung (`integration`).

### Đóng góp cụ thể

-

### Bằng chứng

-

## 2A202603732_TranTrongDucTai

### Phần việc phụ trách

- Shape B (`single` ReAct agent).

### Đóng góp cụ thể

-

### Bằng chứng

-

## 2A202603779_DaoNgocHan

### Phần việc phụ trách

- Shape C (`orchestrator`) và cơ chế lọc dữ liệu chuyển giao (`handoff`).

### Đóng góp cụ thể

-

### Bằng chứng

-

## 2A202603761_NguyenGiaNam

### Phần việc phụ trách

- Kiểm thử độ bền giao dịch / chính sách (`ledger`, `policy`, `crash`) và đánh giá thực nghiệm (`evaluation` / báo cáo).

### Đóng góp cụ thể

- Thêm hai test host cho hạn mức hoàn theo đơn, giới hạn approval và retry sau khi approval hết hạn.
- Chạy crash suite 20 điểm và lưu báo cáo do công cụ sinh ra.

### Bằng chứng

- Commit kỹ thuật: `e67183f`.
- `src/tests/test_ledger_order_budget.py`; `src/reports/crash.json` (`crash_safety: 20/20`).
