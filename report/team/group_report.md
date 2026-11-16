# Báo cáo nhóm — Lab 8 LogiX-VN

Tối đa 5 trang ngoài phụ lục. Số liệu lấy từ file trong `src/reports/`, không chép tay từ terminal.

## 1. Thông tin nhóm

- Tên nhóm:
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

...

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
