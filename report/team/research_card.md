# Research card

File này nằm cố định ở `report/team/research_card.md`; `python -m lab8.check_submission` đọc cột "Nhóm điền" bằng máy.

Phần này nối tiếp phần mini research ở buổi lý thuyết (slide 62–63). Chọn **một** câu R1–R4, điền 6 dòng, rồi biến nó thành **một ablation** chạy được trong lab: giữ nguyên model (MockLLM), tool, seed, corpus và giá, chỉ thay đổi một biến.

| Câu | Nguồn (thư viện tham khảo, slide 68–69) | Câu hỏi gợi ý cho lab |
| --- | --- | --- |
| R1 · Scaling | Google Research 2026, *Towards a Science of Scaling Agent Systems* | Task claims chủ yếu tuần tự hay song song được? Dự đoán thêm investigator sẽ làm pass rate và cost/success thay đổi thế nào. |
| R2 · Failure taxonomy | MAST, NeurIPS 2025, *Why Do Multi-Agent LLM Systems Fail?* | Các trial fail của swarm (seed 55) thuộc nhóm nào: lỗi giao việc, lệch mục tiêu, hay lỗi kiểm chứng? |
| R3 · Architecture choices | AgentArch, AACL 2026 | Nếu đổi topology mà giữ nguyên mọi thứ khác, biến nào phải cố định để kết luận còn đúng? |
| R4 · Evaluation confounders | EMNLP 2026, *Rethinking Evaluation of MAS Efficiency* | Làm sao chứng minh chênh lệch giữa A/B/C/D là do topology, không phải do prompt, tool hay seed? |

## Phiếu của nhóm

Điền vào cột cuối. Không xóa hai cột đầu.

| Field | Hướng dẫn | Nhóm điền |
| --- | --- | --- |
| CÂU | R1 / R2 / R3 / R4 | R2 — phân loại lỗi của swarm ở C-002, seed 55 |
| SOURCE | Paper/blog, năm, URL, loại bằng chứng (benchmark, production report, blog kinh nghiệm) | Cemri và cộng sự, *Why Do Multi-Agent LLM Systems Fail?*, NeurIPS 2025, nghiên cứu trên trace: https://proceedings.neurips.cc/paper_files/paper/2025/hash/b1041e52d3be19f0a9bc491657488e4a-Abstract-Datasets_and_Benchmarks_Track.html |
| CLAIM | Một kết luận gốc của nguồn. Không viết rộng hơn phạm vi paper | Paper đề xuất MAST với 14 kiểu lỗi thuộc 3 nhóm: thiết kế hệ thống, lệch giữa các agent và kiểm chứng kết quả. |
| BOUNDARY | Model / dataset / setup giới hạn kết luận đó | Nguồn phân tích hơn 1.600 trace của 7 framework multi-agent, chủ yếu bài coding, toán và tác vụ agent. Không lấy tỉ lệ lỗi của paper áp vào MockLLM và dữ liệu giả của lab. |
| HYPOTHESIS | Nếu đổi shape A→B (hoặc bật/tắt một thành phần), metric X sẽ đổi theo hướng nào, khoảng bao nhiêu | Nếu swarm chỉ dùng `delay_days` của order, lỗi C-002 ở seed 55 sẽ hết: pass rate dự kiến tăng từ 69/70 lên 70/70, pass^5 từ 13/14 lên 14/14. |
| METHOD | Biến giữ cố định: model, tools, seed, corpus, giá, evaluator. Biến duy nhất được thay đổi: … | Giữ MockLLM, bộ tool, 14 claim public, seed 11/22/33/44/55, bảng giá và `lab8.benchmark`. Chỉ tắt nhánh lấy số ngày trễ từ ticket để ghi đè order; model vẫn được gọi như cũ. So `swarm` với `swarm_order_only` trong `src/bonus/swarm_ticket_ablation.py`. |
| FAILURE / METRIC | Một lỗi dự kiến và cách đo nó: pass rate, pass^5, variance, cost/success, leak, bypass | Nhóm xếp lỗi này gần FM-3.2 (thiếu kiểm chứng dữ liệu đầu vào) của MAST; đây là cách nhóm diễn giải, paper không chấm claim của lab. Đo pass rate, pass^5, seed variance, cost/success và xem riêng C-002 seed 55. |

## Kết quả ablation

- Lệnh đã chạy (trong `src/`): `python -m lab8.benchmark --extra-shape swarm_order_only=bonus.swarm_ticket_ablation:run_claim --shapes swarm,swarm_order_only --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/r2_ablation.json`
- Run id / file report: `R-swarm-C-002-s55`, `R-swarm_order_only-C-002-s55`; `src/reports/r2_ablation.json`.
- Kết quả so với hypothesis: đúng trên bộ public. Swarm gốc pass 69/70, pass^5 13/14, variance 0,000816, cost/success 49,57 VND; bản chỉ dùng order pass 70/70, pass^5 14/14, variance 0, cost/success 48,72 VND. Ở C-002 seed 55, bản gốc trả `resolved`, bản ablation trả đúng `denied`.
- Giới hạn: Đây là 14 claim giả và một lỗi được cài sẵn trong swarm tham chiếu. Kết quả chưa chứng minh cách sửa này tốt hơn trên dữ liệu thật. MockLLM không phải model sản xuất; latency là `unavailable`.

Muốn đi xa hơn một ablation để lấy điểm cộng thì xem [docs/bonus_topics.md](../../docs/bonus_topics.md).
