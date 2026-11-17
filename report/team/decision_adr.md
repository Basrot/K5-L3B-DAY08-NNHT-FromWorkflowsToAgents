# Decision record

Điền sau dấu `:` trên cùng dòng, không xóa nhãn. Giảng viên và `python -m lab8.check_submission` (chạy trong `src/`) đọc các dòng này bằng máy. Không đổi tên hay dời file này khỏi `report/team/`.
`Shape chọn` phải là đúng một trong: `workflow`, `single`, `orchestrator`, `swarm`. Đây là shape được chấm cổng hidden.

- Shape chọn:
- Commit hash:
- Run id dẫn chứng:
- Pass rate / pass^5 / seed min-max / variance:
- Cost per success (mô phỏng, VND):
- Crash safety:
- Secret leak / approval bypass:

Ví dụ một dòng đã điền: `- Shape chọn: workflow`, `- Crash safety: 20/20 (src/reports/crash.json)`.

## Vì sao shape này

Viết 1 đoạn. So cùng corpus và cùng seed, không so với một lần chạy lẻ.

## Có đáng đổi lấy multi-agent không

Nếu pass nhỉnh hơn mà cost và variance cao hơn, nhóm giữ hay đổi? Điều kiện nào làm quyết định đảo lại?

## Trace và bằng chứng

- **Các event ghi trong trace:** Tùy theo shape, file `traces/R-<shape>-<claim_id>-s<seed>.json` ghi danh sách `events` có cấu trúc gồm: `handoff` (gói `handoff.v1` đã qua allowlist ở Shape C/D), `investigator` hoặc `tool_call` (kết quả đọc từ `lookup_order` chỉ gồm `ORDER_FIELDS`, số lượng policy hiện hành, kết quả `compute_compensation`), `quote` (chi tiết quote tất định), `decision` (trạng thái chốt và lý do), `error` (như `unknown_claim`, `step_cap`) và `crash` (điểm `crash_at` được flush trước khi `InjectedCrash` lan truyền).
- **Các field cố ý không ghi và lý do:** Nhóm tuyệt đối không ghi dòng `claims` thô (`parent_secret`, `raw_headers_json`), dòng `orders` thô (`phone`, `address`, `item_name`, `internal_note` chứa `CANARY_INTERNAL_*`) và nội dung câu chữ `ticket` thô của khách hàng. Lý do: trace là một phần của ranh giới bảo mật mà hệ thống chấm/giám sát đọc trực tiếp; ghi dữ liệu gốc chưa lọc sẽ làm rò rỉ bí mật định danh của tác tử cha, lộ thông tin cá nhân (PII) khi tra cứu chéo `principal` (như T05), hoặc lưu lại dữ liệu nhạy cảm khách tự gõ trong ticket.
- **Vì sao phải fail-closed nếu trace nằm ngoài vùng quét:** Nếu thiếu file trace (`missing_trace`), dùng lại trace cũ (`stale_trace`), sai định dạng/`run_id`, hoặc shape ghi debug log ra đường dẫn ngoài quy ước `traces/<run_id>*` hay `stdout`, bộ quét bảo mật (scanner) không thể đọc được toàn bộ dữ liệu xuất ra của lần chạy đó. Không quan sát được đồng nghĩa với không thể chứng minh lần chạy không làm lộ secret/PII ("absence of evidence is not evidence of absence"), nên đội security và grader bắt buộc phải đánh trượt (fail-closed) và kiểm soát chặt mọi kênh ghi file theo đúng hợp đồng `Trace contract v1`.

## Giới hạn

MockLLM không phải model sản xuất. Latency để `unavailable`. Không sửa JSON cho đủ 0.75.
