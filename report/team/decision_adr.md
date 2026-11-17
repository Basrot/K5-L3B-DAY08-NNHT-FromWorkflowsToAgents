# Decision record

Điền sau dấu `:` trên cùng dòng, không xóa nhãn. Giảng viên và `python -m lab8.check_submission` (chạy trong `src/`) đọc các dòng này bằng máy. Không đổi tên hay dời file này khỏi `report/team/`.
`Shape chọn` phải là đúng một trong: `workflow`, `single`, `orchestrator`, `swarm`. Đây là shape được chấm cổng hidden.

- Shape chọn: workflow
- Commit hash: 99cafcd
- Run id dẫn chứng: R-workflow-C-001-s11; R-swarm-C-002-s55 (src/reports/benchmark.json)
- Pass rate / pass^5 / seed min-max / variance: workflow 70/70 = 1,000 / 14/14 = 1,000 / 1,000–1,000 / 0,000000
- Cost per success (mô phỏng, VND): workflow 27,86; single 30,36; orchestrator 47,90; swarm 49,57
- Crash safety: 20/20 (src/reports/crash.json); T12 workflow pass trong student tests
- Secret leak / approval bypass: không phát hiện trong 70 trial public của workflow; hidden chưa có kết quả

Ví dụ một dòng đã điền: `- Shape chọn: workflow`, `- Crash safety: 20/20 (src/reports/crash.json)`.

## Vì sao shape này

Trên cùng 14 claim và 5 seed, workflow, single và orchestrator đều pass 70/70 trial; swarm pass 69/70. Workflow có chi phí mô phỏng thấp nhất (27,86 VND/lần pass) và variance bằng 0. Vì cả ba shape đầu cùng pass trên bộ public, nhóm chọn workflow theo mục tiêu dùng kiến trúc ít tự chủ nhất mà vẫn làm được việc. Đây là quyết định trên dữ liệu public, chưa thay cho kết quả hidden.

## Có đáng đổi lấy multi-agent không

Trên bộ public, multi-agent chưa cho thêm trial pass so với workflow nhưng tốn chi phí mô phỏng cao hơn. Nhóm sẽ cân nhắc đổi nếu bộ hidden cho thấy workflow bỏ sót trường hợp mà một shape khác xử lý đúng, trong khi vẫn đạt các điều kiện must-pass và mức tăng chi phí chấp nhận được.

## Trace và bằng chứng

- **Các event ghi trong trace:** Tùy theo shape, file `traces/R-<shape>-<claim_id>-s<seed>.json` ghi danh sách `events` có cấu trúc gồm: `handoff` (gói `handoff.v1` đã qua allowlist ở Shape C/D), `investigator` hoặc `tool_call` (kết quả đọc từ `lookup_order` chỉ gồm `ORDER_FIELDS`, số lượng policy hiện hành, kết quả `compute_compensation`), `quote` (chi tiết quote tất định), `decision` (trạng thái chốt và lý do), `error` (như `unknown_claim`, `step_cap`) và `crash` (điểm `crash_at` được flush trước khi `InjectedCrash` lan truyền).
- **Các field cố ý không ghi và lý do:** Nhóm tuyệt đối không ghi dòng `claims` thô (`parent_secret`, `raw_headers_json`), dòng `orders` thô (`phone`, `address`, `item_name`, `internal_note` chứa `CANARY_INTERNAL_*`) và nội dung câu chữ `ticket` thô của khách hàng. Lý do: trace là một phần của ranh giới bảo mật mà hệ thống chấm/giám sát đọc trực tiếp; ghi dữ liệu gốc chưa lọc sẽ làm rò rỉ bí mật định danh của tác tử cha, lộ thông tin cá nhân (PII) khi tra cứu chéo `principal` (như T05), hoặc lưu lại dữ liệu nhạy cảm khách tự gõ trong ticket.
- **Vì sao phải fail-closed nếu trace nằm ngoài vùng quét:** Nếu thiếu file trace (`missing_trace`), dùng lại trace cũ (`stale_trace`), sai định dạng/`run_id`, hoặc shape ghi debug log ra đường dẫn ngoài quy ước `traces/<run_id>*` hay `stdout`, bộ quét bảo mật (scanner) không thể đọc được toàn bộ dữ liệu xuất ra của lần chạy đó. Không quan sát được đồng nghĩa với không thể chứng minh lần chạy không làm lộ secret/PII ("absence of evidence is not evidence of absence"), nên đội security và grader bắt buộc phải đánh trượt (fail-closed) và kiểm soát chặt mọi kênh ghi file theo đúng hợp đồng `Trace contract v1`.

## Giới hạn

MockLLM không phải model sản xuất. Latency để `unavailable`. Không sửa JSON cho đủ 0.75.
