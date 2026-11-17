# Báo cáo cá nhân — 2A202603788_ThieuTheNgoc

## 1. Thông tin thành viên

- Họ tên: Thiều Thế Ngọc
- MSSV: 2A202603788
- Email: 26ai.ngoctt@vinuni.edu.vn
- GitHub: BFNGOC2
- Nhóm: NNHT, lớp L3B

## 2. Phần việc phụ trách

- Workflow A và integration.
- Đã triển khai Workflow A và thực hiện kiểm tra tích hợp các shape. Không sửa code shape của thành viên khác; không phát hiện lỗi tích hợp cần sửa.

## 3. Đóng góp cụ thể

- Commit `837c75f` (`feat: CP2-3`) triển khai Workflow A trong `src/lab8/shapes/workflow.py`.
- Rà soát contract của workflow, single, orchestrator và swarm; chạy test toàn hệ thống, crash suite, benchmark public và kiểm tra trace.
- Commit trên chỉ thay đổi `workflow.py`. Tôi chưa thêm hoặc sửa test riêng; các bài kiểm tra dưới đây là test có sẵn trong repo.

## 4. Bằng chứng

Kiểm thử từ `src/`:

- `python -m pytest -q`: **79 passed, 248 deselected**.
- `python -m pytest -q -m student -o addopts=`: **248 passed, 79 deselected**.
- `python -m lab8.crash_test --count 20 --out reports/crash.json`: **crash_safety 20/20**.
- `python -m lab8.check_submission`: **8 FAIL, 1 WARN**; chi tiết thiếu sót được ghi ở mục 6. Không chạy hidden tests.

Benchmark public có 14 claim × 5 seed (70 trial mỗi shape); cost là VND mô phỏng:

| Shape | Pass rate | Pass^5 | Seed variance | Seed min–max | Cost/success |
|---|---:|---:|---:|---:|---:|
| workflow | 1.000 | 1.000 | 0.000000 | 1.000–1.000 | 27.86 |
| single | 1.000 | 1.000 | 0.000000 | 1.000–1.000 | 30.36 |
| orchestrator | 1.000 | 1.000 | 0.000000 | 1.000–1.000 | 47.90 |
| swarm | 0.986 | 0.929 | 0.000816 | 0.929–1.000 | 49.57 |

Swarm có một trial sai: C-002, seed 55, trả `resolved` thay vì `denied` do điểm yếu tham chiếu tin ngày trễ trong ticket. Workflow, single và orchestrator đều đạt 70/70 trên public benchmark. Đã kiểm tra contract và chuỗi cấm trên 280 trace benchmark: 0 lỗi.

Trên bộ public, workflow là lựa chọn ít tự chủ nhất và có cost/success thấp nhất trong bốn shape. Đây không phải kết luận cho bộ hidden. `git diff --check` sạch.

## 5. Công cụ AI đã dùng

- Dùng Copilot trong VS Code để hỗ trợ đọc source, rà soát contract và diễn giải kết quả kiểm thử/benchmark.
- Tự chạy các lệnh pytest, crash suite, benchmark và submission checker nêu trên. Không tự nhận đã chạy hidden tests.

## 6. Tự đánh giá

Phần Workflow A và kiểm tra tích hợp public đã hoàn thành; không phát hiện lỗi cần sửa và không thay đổi code của shape khác. `check_submission` còn **8 FAIL**: bảy trường bắt buộc trong `report/team/decision_adr.md` còn trống và bảy trường trong Research Card chưa điền. Checker có **1 WARN** vì `reports/benchmark.json` đang chưa được commit. Tôi không sửa ADR hay báo cáo nhóm vì đây là quyết định/thông tin dùng chung.

Tôi chưa có test tự thêm/sửa, nên chưa đáp ứng bằng chứng test cá nhân trong rubric. Hidden evaluation chưa chạy; các thành viên cần thống nhất và hoàn thiện ADR, Research Card, báo cáo nhóm, rồi bổ sung test cá nhân và commit/push các deliverable trước khi nộp.
