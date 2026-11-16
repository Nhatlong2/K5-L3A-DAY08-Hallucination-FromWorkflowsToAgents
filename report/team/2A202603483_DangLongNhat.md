# Báo cáo cá nhân — 2A202603483_DangLongNhat

## 1. Thông tin thành viên

- Họ tên: Đặng Long Nhật
- MSSV: 2A202603483
- GitHub: [Nhatlong2](https://github.com/Nhatlong2)
- Email: xem `TEAM.md`

## 2. Phần việc phụ trách

TV3 phụ trách Shape C `orchestrator`, investigator chỉ đọc, handoff theo allowlist và trace theo hợp đồng v1.

## 3. Đóng góp cụ thể

- Commit kỹ thuật `a56d25e` gồm `src/lab8/shapes/orchestrator.py` và `src/tests/test_orchestrator_handoff.py`.
- Investigator được cấp duy nhất `lookup_order`. Orchestrator tính quote từ dữ liệu đơn rồi chuyển quote đầy đủ cùng `crash_at` cho host. Kết quả lưu `proposal_amount` tách biệt với số tiền thực tế trong ledger.
- Test bổ sung kiểm tra payload investigator nhận, quyền đọc, trace an toàn và trace khi claim không tồn tại.

## 4. Bằng chứng

- `python -m pytest -q` đạt 199 ca, gồm 2 ca bổ sung.
- `python -m pytest -q -m student -o addopts= -k orchestrator` đạt 62 ca Shape C. Toàn bộ ma trận student đạt 248 ca.
- Rà 210 trace public của Workflow, Single Agent và Orchestrator từ 14 claim × 5 seed. Không tìm thấy canary, giá trị header hoặc thông tin cá nhân trong fixture.
- Với run `R-orchestrator-C-012-s11`, thử cả ba điểm crash trên DB riêng. Mỗi lần chạy lại ghi đúng một refund 35.000 VND và một audit `refund_committed`.

## 5. Công cụ AI đã dùng

Codex hỗ trợ triển khai bản nháp, viết test và chạy các lệnh kiểm chứng trên. TV3 cần tự chạy lại test Shape C và đối chiếu trace với ledger trước khi chốt báo cáo.

## 6. Tự đánh giá

TV3 bổ sung phần tự đánh giá sau khi tự kiểm tra kết quả.
