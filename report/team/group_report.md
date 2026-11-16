# Báo cáo nhóm — Lab 8 LogiX-VN

Tối đa 5 trang ngoài phụ lục. Số liệu lấy từ file trong `src/reports/`, không chép tay từ terminal.

## 1. Thông tin nhóm

- Tên nhóm: Hallucination
- Thành viên: xem `TEAM.md`
- Commit chốt (trùng `report/team/decision_adr.md`):

## 2. Mục tiêu và cách tiếp cận

Nhóm hiểu bài toán thế nào, chia việc ra sao, thứ tự làm bốn shape.

...

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
| **Single Agent (Shape B)** | **1.000** | **1.000** | 1.000 – 1.000 | **0.000000** | **39.54 VND** | 8.21 / 9.0 | 5.64 | Hoàn thành, 100% pass |
| **Orchestrator (Shape C)** | *Đang cập nhật* | *Đang cập nhật* | *Đang cập nhật* | *Đang cập nhật* | *Đang cập nhật* | *Đang cập nhật* | *Đang cập nhật* | Đang phát triển |
| **Swarm (Shape D)** | **0.986** | **0.929** | 0.929 – 1.000 | **0.000816** | **49.57 VND** | 2.80 / 3.0 | 3.00 | Đối chứng, fail seed 55 |

- **Crash Safety**: Đạt **`20/20`** an toàn tuyệt đối trên cả 20 kịch bản tiêm lỗi crash (file minh chứng: `src/reports/crash.json`).

### Phân tích so sánh chi tiết giữa các kiến trúc:

1. **Workflow (Shape A)**:
   - **Ưu điểm vượt trội về chi phí và độ ổn định**: Đây là kiến trúc có chi phí thấp nhất (**28.79 VND/success**) và số bước thực thi ngắn nhất (2.71 steps). Độ ổn định đạt mức tuyệt đối (**Variance = 0.000000**, pass rate 100% trên cả 5 seeds).
   - **Bản chất**: Toàn bộ luồng nghiệp vụ được mã hóa cố định bằng code Python xác định (deterministic). Hệ thống không gọi LLM để suy luận hay quyết định logic bồi thường, loại bỏ hoàn toàn khả năng phát sinh ảo giác (hallucination). Rất phù hợp và hiệu quả nhất khi luật nghiệp vụ đã rõ ràng, khép kín.

2. **Single Agent (Shape B)**:
   - **Độ chính xác cao nhưng tốn kém tài nguyên**: Đạt pass rate 100% tương đương Workflow, tuy nhiên chi phí cao hơn **~37.3%** (**39.54 VND/success**) và số bước thực thi kéo dài (trung bình 8.21 steps).
   - **Bản chất**: Vận hành theo vòng lặp ReAct ($\le 8$ bước), tốn trung bình **5.64 model calls** cho mỗi claim để phân tích ngữ cảnh và quyết định gọi các tool đọc dữ liệu. Mặc dù tốn chi phí và thời gian hơn, Single Agent có tính linh hoạt cao hơn khi cần xử lý các yêu cầu thay đổi động hoặc cần diễn giải thông tin phức tạp.

3. **Swarm (Shape D - Tham chiếu đối chứng)**:
   - **Kém ổn định và chi phí cao nhất**: Chi phí lên tới **49.57 VND/success** (đắt hơn 72.2% so với Workflow) do tốn nhiều lượt token gọi model giữa các peer và parse ticket.
   - **Điểm yếu cốt tử**: Tồn tại phương sai giữa các seeds (`Variance = 0.000816`), Pass Rate chỉ đạt **0.986**. Ở seed 55 (claim C-002), Swarm bị ảo giác do tin vào số ngày trễ người dùng tự khai trong ticket thay vì đối chiếu order row từ database, dẫn đến việc tính sai quote và bị host từ chối (`status resolved != denied`).

Quyết định kiến trúc và lý do: xem `report/team/decision_adr.md`. Phiếu nghiên cứu và ablation: xem `report/team/research_card.md`.

## 5. Khó khăn và cách xử lý

Lỗi gặp phải (kèm lệnh, output hoặc ảnh chụp) và cách nhóm xử lý.

...

## 6. Tổng hợp đóng góp

Chi tiết đóng góp của từng thành viên nằm trong `TEAM.md` (bảng đóng góp) và báo cáo cá nhân `report/team/MSSV_HoTen.md`.

## 7. Hạn chế và hướng cải thiện

MockLLM không phải model sản xuất; latency là `unavailable`. Nêu thêm giới hạn của nhóm và điều kiện làm kết luận thay đổi.

...

## 8. Tài liệu tham khảo

...
