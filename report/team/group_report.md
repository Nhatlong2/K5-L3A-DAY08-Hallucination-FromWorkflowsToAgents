# Báo cáo nhóm — Lab 8 LogiX-VN

Tối đa 5 trang ngoài phụ lục. Số liệu lấy từ file trong `src/reports/`, không chép tay từ terminal.

## 1. Thông tin nhóm

- Tên nhóm: Hallucination
- Thành viên: xem `TEAM.md`
- Commit chốt (trùng `report/team/decision_adr.md`):

## 2. Mục tiêu và cách tiếp cận

- **Hiểu bài toán**: Xử lý claim hoàn phí vận chuyển và bảo hiểm hư hại của công ty giả lập LogiX-VN theo cùng chữ ký hàm `run_claim` trên bốn kiến trúc khác nhau (Workflow, Single Agent, Orchestrator, Swarm). Ranh giới sống còn: tiền bồi thường, phê duyệt (approval) và chống ghi trùng do Host và code quyết định; model LLM chỉ được đọc dữ liệu và soạn thảo văn bản.
- **Cách tiếp cận & Tiêu chí lựa chọn**: Thực nghiệm đo lường công bằng trên cùng tập dữ liệu, cùng 5 seed ngẫu nhiên và cùng bảng giá. Nhóm chọn kiến trúc "ít tự chủ nhất mà vẫn làm được việc" để giảm thiểu tối đa chi phí và loại bỏ hoàn toàn rủi ro ảo giác (hallucination).
- **Phân chia công việc**: Nhóm 4 người chia việc độc lập theo 4 luồng song song: TV1 (Workflow + Tích hợp), TV2 (Single Agent ReAct), TV3 (Orchestrator + Handoff allowlist), TV4 (Durability, Crash suite, Benchmark và Evaluation).
- **Thứ tự thực hiện**: Cài đặt cổng host sạch (CP1) → Triển khai 3 shape độc lập (CP2) → Rà soát Idempotency và Crash resilience (CP3) → Nghiệm thu Crash suite 20/20 và rà secret leak (CP4) → Benchmark 4 shape và ra quyết định kiến trúc (CP5).

## 3. Giải pháp

- **Workflow** đọc claim và đơn theo các bước Python cố định. Code dùng dữ liệu đơn và policy hiện hành để tính quote. MockLLM chỉ soạn phản hồi.
- **Single Agent** đọc claim, đơn và lịch sử hoàn tiền qua vòng ReAct giới hạn 8 bước. Agent chỉ dùng tool đọc. Sau vòng lặp, code tính lại quote từ dữ liệu đơn.
- **Orchestrator** lấy mã claim và phạm vi chủ đơn từ dữ liệu hệ thống. `make_handoff()` tạo `handoff.v1` theo allowlist, để trống `facts` và chỉ cấp `lookup_order` cho investigator. Investigator đọc đơn theo phạm vi chủ đơn. Orchestrator tính quote bằng `compute_compensation()` từ order view. MockLLM chỉ nhận trạng thái để soạn phản hồi.

Cả ba shape gửi quote dương đầy đủ cho `durability.issue_refund()`. Host kiểm tra phê duyệt, áp trần theo đơn và xử lý các lần gọi lại. Shape C truyền `crash_at` đến host để bộ test kiểm tra khôi phục sau lỗi.

Trace của Shape C ghi handoff đã lọc, tên tool và kết quả kiểm tra phạm vi truy cập, giá trị quote, kết quả từ host và quyết định cuối. Các nhánh kết thúc sớm cũng ghi trace qua `finish()`. Nội dung ticket, claim/order thô, header, secret và thông tin cá nhân được giữ ngoài trace.

## 4. Kết quả

Bảng so sánh 4 shape trên bộ public (từ `src/reports/benchmark.json`): pass rate, pass^5, seed min–max, variance, cost/success, mean/p95 steps. Kèm crash safety (`src/reports/crash.json`) và, nếu có, kết quả tự kiểm trên `student_drop/`.

### Bảng số liệu Benchmark trên bộ Public Claims (14 claims × 5 seeds)

| Kiến trúc (Shape) | Pass Rate | Pass^5 | Seed min–max | Seed Variance | Cost/Success (mô phỏng) | Mean/p95 Steps | Model Calls TB | Ghi chú trạng thái |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :--- |
| **Workflow (Shape A)** | **1.000** | **1.000** | 1.000 – 1.000 | **0.000000** | **28.79 VND** | 2.71 / 3.0 | 1.00 | Hoàn thành, 100% pass |
| **Orchestrator (Shape C)** | **1.000** | **1.000** | 1.000 – 1.000 | **0.000000** | **38.06 VND** | 4.36 / 5.0 | 2.00 | Hoàn thành, 100% pass |
| **Single Agent (Shape B)** | **1.000** | **1.000** | 1.000 – 1.000 | **0.000000** | **39.54 VND** | 8.21 / 9.0 | 5.64 | Hoàn thành, 100% pass |
| **Swarm (Shape D)** | **0.986** | **0.929** | 0.929 – 1.000 | **0.000816** | **49.57 VND** | 2.80 / 3.0 | 3.00 | Đối chứng, fail seed 55 |

- **Crash Safety**: Đạt **`20/20`** an toàn tuyệt đối trên cả 20 kịch bản tiêm lỗi crash (file minh chứng: `src/reports/crash.json`).

### Phân tích so sánh chi tiết giữa các kiến trúc:

1. **Workflow (Shape A)**:
   - **Ưu điểm vượt trội về chi phí và độ ổn định**: Đây là kiến trúc có chi phí thấp nhất (**28.79 VND/success**) và số bước thực thi ngắn nhất (2.71 steps). Độ ổn định đạt mức tuyệt đối (**Variance = 0.000000**, pass rate 100% trên cả 5 seeds).
   - **Bản chất**: Toàn bộ luồng nghiệp vụ được mã hóa cố định bằng code Python xác định (deterministic). Hệ thống không gọi LLM để suy luận hay quyết định logic bồi thường, loại bỏ hoàn toàn khả năng phát sinh ảo giác (hallucination). Rất phù hợp và hiệu quả nhất khi luật nghiệp vụ đã rõ ràng, khép kín.

2. **Single Agent (Shape B)**:
   - **Độ chính xác cao nhưng tốn kém tài nguyên**: Đạt pass rate 100% tương đương Workflow, tuy nhiên chi phí cao hơn **~37.3%** (**39.54 VND/success**) và số bước thực thi kéo dài (trung bình 8.21 steps).
   - **Bản chất**: Vận hành theo vòng lặp ReAct ($\le 8$ bước), tốn trung bình **5.64 model calls** cho mỗi claim để phân tích ngữ cảnh và quyết định gọi các tool đọc dữ liệu. Mặc dù tốn chi phí và thời gian hơn, Single Agent có tính linh hoạt cao hơn khi cần xử lý các yêu cầu thay đổi động hoặc cần diễn giải thông tin phức tạp.

3. **Orchestrator (Shape C)**:
   - **Cân bằng giữa cấu trúc và phân quyền**: Chi phí bồi thường mô phỏng đạt **38.06 VND/success** (thấp hơn Single Agent ~3.7%), số bước thực thi trung bình **4.36 steps** và chỉ mất trung bình **2.00 model calls**. Đạt độ ổn định tuyệt đối (**Pass rate 1.000**, **Variance 0.000000**).
   - **Bản chất**: Orchestrator đóng vai trò điều phối trung tâm, sử dụng `make_handoff` tuân thủ nghiêm ngặt Trace Contract v1 để tạo payload an toàn cho sub-agent (`investigator`). Sub-agent chỉ được cấp quyền đọc `lookup_order` với phạm vi giới hạn theo `customer_id` của chủ đơn, hoàn toàn không được cấp tool ghi tiền. Sau khi investigator thu thập order view, Orchestrator tự tính quote bằng code xác định và gửi quote đầy đủ đến Host. Nhờ không phải chạy vòng lặp ReAct dài như Single Agent, Orchestrator tiết kiệm token hơn trong khi vẫn duy trì sự cô lập bảo mật chặt chẽ.

4. **Swarm (Shape D - Tham chiếu đối chứng)**:
   - **Kém ổn định và chi phí cao nhất**: Chi phí lên tới **49.57 VND/success** (đắt hơn 72.2% so với Workflow) do tốn nhiều lượt token gọi model giữa các peer và parse ticket.
   - **Điểm yếu cốt tử**: Tồn tại phương sai giữa các seeds (`Variance = 0.000816`), Pass Rate chỉ đạt **0.986**. Ở seed 55 (claim C-002), Swarm bị ảo giác do tin vào số ngày trễ người dùng tự khai trong ticket thay vì đối chiếu order row từ database, dẫn đến việc tính sai quote và bị host từ chối (`status resolved != denied`).

Quyết định kiến trúc và lý do: xem `report/team/decision_adr.md`. Phiếu nghiên cứu và ablation: xem `report/team/research_card.md`.

## 5. Khó khăn và cách xử lý

Lỗi gặp phải (kèm lệnh, output hoặc ảnh chụp) và cách nhóm xử lý.

1. **Ranh giới an toàn thông tin & Trace Contract v1**:
   - *Vấn đề*: Trong các shape có sự tham gia của Agent (Single, Orchestrator), nếu để Agent tự do đọc dữ liệu thô từ claim hoặc order, nguy cơ rò rỉ các thông tin nhạy cảm như `parent_secret` (`CANARY_SECRET_*`), token `Bearer`, số điện thoại, địa chỉ khách hàng (`PII`) và ghi chú nội bộ vào file nhật ký `trace` là rất lớn. Bộ chấm thi của Ban tổ chức sẽ fail-closed ngay lập tức nếu phát hiện chuỗi cấm trong trace.
   - *Cách xử lý*: Cả nhóm thống nhất áp dụng nguyên tắc Allowlist nghiêm ngặt tại các cổng đọc dữ liệu (`get_claim_view`, `lookup_order`). Với Orchestrator, hàm `make_handoff()` chủ động bóc tách và loại bỏ hoàn toàn các trường cấm trước khi chuyển ngữ cảnh cho sub-agent. Toàn bộ 272 file trace sinh ra trong quá trình chạy thực tế đều được quét tự động bằng script bảo mật và đạt **0 secret leak**, **0 PII leak**.

2. **Tính bền vững (Durability) và Khôi phục sau Crash**:
   - *Vấn đề*: Khi hệ thống gặp sự cố mất điện/crash tại các thời điểm nhạy cảm (như ngay sau khi ghi tiền vào Ledger nhưng chưa kịp lưu checkpoint, hoặc trước khi ghi hiệu ứng), việc chạy lại (retry/resume) có nguy cơ gây ra lỗi ghi trùng tiền (double refund), vi phạm nghiêm trọng nguyên tắc kế toán tài chính.
   - *Cách xử lý*: Nhóm triển khai cơ chế Idempotency dựa trên khóa duy nhất `refund:{claim_id}` trong SQLite với transaction `BEGIN IMMEDIATE`. Khi phát hiện claim đã được xử lý thành công trước đó, host lập tức trả về kết quả đã ghi nhận thay vì thực thi lại. Kiểm thử toàn diện qua Crash Suite 20/20 (`python -m lab8.crash_test --count 20 --out reports/crash.json`) đã chứng minh hệ thống an toàn tuyệt đối, đạt `crash_safety = 20/20`, không có bất kỳ giao dịch nào bị nhân đôi.

3. **Bẫy nghiệp vụ trần hoàn tiền theo đơn hàng (Per-order Cap)**:
   - *Vấn đề*: Một số claim phức tạp (như ca T13, T14) có đơn hàng đã từng được hoàn tiền một phần trước đó. Nếu Agent hoặc Workflow tự ý tính toán và trừ bớt số tiền đã hoàn trước khi gửi sang Host, hoặc ngược lại bỏ qua lịch sử hoàn tiền, sẽ dẫn tới việc trừ hai lần (double deduction) hoặc vượt quá tổng giá trị đơn hàng.
   - *Cách xử lý*: Nhóm tuân thủ đúng hợp đồng kiến trúc: các shape luôn gửi **quote bồi thường đầy đủ** (tính theo công thức chuẩn của vi phạm) sang cho Host. Trách nhiệm kiểm tra lịch sử hoàn tiền, đối chiếu trần đơn hàng và quyết định số tiền thực chi cuối cùng thuộc về Host Authorization Gate. Nhờ đó, cả 4 shape đều xử lý chính xác tuyệt đối các ca biên này.

## 6. Tổng hợp đóng góp

Chi tiết đóng góp của từng thành viên nằm trong `TEAM.md` (bảng đóng góp) và báo cáo cá nhân `report/team/MSSV_HoTen.md`.

## 7. Hạn chế và hướng cải thiện

MockLLM không phải model sản xuất; latency là `unavailable`. Nêu thêm giới hạn của nhóm và điều kiện làm kết luận thay đổi.

1. **Hạn chế thực nghiệm hiện tại**:
   - **Mô hình MockLLM**: Trong phạm vi bài lab, MockLLM hoạt động dựa trên các bộ luật và quy tắc mô phỏng xác định (deterministic mock). Do đó, các độ đo về chi phí token và số bước gọi model mới chỉ phản ánh mức độ phức tạp về mặt cấu trúc luồng điều khiển, chưa phản ánh được sự biến động thực tế về mặt ngữ nghĩa (semantic drift) hay độ suy giảm hiệu năng do context window dài của các mô hình LLM sản xuất (production models như Gemini 1.5 Pro, Claude 3.5 Sonnet).
   - **Độ trễ (Latency)**: Trường `latency_ms` trong benchmark đang để giá trị `unavailable` do MockLLM phản hồi tức thì. Trong môi trường sản xuất thực tế, độ trễ mạng khi gọi API LLM từ xa sẽ là một nút thắt cổ chai lớn đối với Single Agent (5.64 calls) và Swarm (3.00 calls).
   - **Phạm vi dữ liệu**: Dữ liệu đầu vào trong bài lab chủ yếu là các trường có cấu trúc (bảng claim, bảng order).

2. **Hướng cải thiện & Điều kiện làm quyết định kiến trúc thay đổi**:
   - Khi bài toán mở rộng sang tiếp nhận các loại dữ liệu phi cấu trúc phức tạp (như hình ảnh gói hàng bị móp méo, tài liệu hóa đơn scan PDF mờ, hoặc biên bản đối soát viết tay từ bên chuyển phát), kiến trúc Workflow thuần túy bằng code if/else sẽ không thể tự trích xuất thông tin. Khi đó, nhóm sẽ nâng cấp lên kiến trúc **Orchestrator** (Shape C): dùng Agent thị giác (Vision LLM) để phân tích trích xuất dữ liệu thô vào các schema chuẩn, sau đó bàn giao (handoff) qua allowlist sang Workflow xác định để tính toán tài chính và ra quyết định bồi thường, kết hợp với Guardrails để chống Prompt Injection.

## 8. Tài liệu tham khảo

1. Tài liệu bài giảng và hướng dẫn thực hành AI20K K5 - L3A Day 08: *Hallucination — From Workflows to Agents*.
2. Anthropic (2024), *Building Effective Agents*, Anthropic Research Blog.
