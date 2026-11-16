# Báo cáo cá nhân — 2A202603427_NguyenVuKhanh

## 1. Thông tin thành viên

- Họ tên: Nguyễn Vũ Khanh
- MSSV: 2A202603427
- Email: vukhanhkg2005@gmail.com
- GitHub: https://github.com/vukhanhkg2005-lang

## 2. Phần việc phụ trách

TV4 — Ledger + Evaluation:
- Phụ trách kiểm thử tính bền vững (durability), cơ chế đảm bảo Idempotency của hệ thống Ledger.
- Thiết kế và triển khai bộ test chuyên sâu `src/tests/test_durability_extra.py` kiểm tra crash resilience, chống ghi trùng tiền bồi thường (double refund), và xác thực các điều kiện biên của Host Authorization Gate.
- Đo lường và nghiệm thu Crash Suite 20/20 kịch bản tiêm lỗi crash (`lab8.crash_test`).
- Rà soát bảo mật toàn diện: quét 272 trace files phòng chống rò rỉ thông tin bí mật (`CANARY_SECRET_*`), token và PII theo Trace Contract v1.
- Chạy đo lường Benchmark toàn phần 4 shapes × 5 seeds trên 14 claims public (`src/reports/benchmark.json`).
- Tự kiểm tra độc lập trên bộ dữ liệu kiểm thử ẩn (`src/student_drop/`) đạt kết quả tin cậy (`src/reports/hidden_selfcheck.json`).
- Soạn thảo và hoàn thiện Architectural Decision Record (`report/team/decision_adr.md`) và phụ trách các phần số liệu thực nghiệm trong Báo cáo nhóm (`report/team/group_report.md`).

## 3. Đóng góp cụ thể

- Commit chính: `234438c` ("test(durability): add comprehensive durability and idempotency tests"), bổ sung bộ kiểm thử `src/tests/test_durability_extra.py` gồm 5 test cases bao phủ toàn diện:
  1. `test_retry_same_claim_is_strictly_idempotent`: Gọi lại cùng một claim đã giải quyết thành công nhiều lần, đảm bảo Ledger chỉ ghi đúng 1 bản ghi và audit chỉ có 1 `refund_committed`.
  2. `test_crash_resume_cycle_maintains_single_ledger_row`: Mô phỏng quy trình crash tại điểm nhạy cảm `after_effect_before_checkpoint` rồi phục hồi (resume) chạy lại trên database đó, xác nhận hiệu ứng chỉ được cam kết một lần duy nhất.
  3. `test_denied_claim_never_writes_to_ledger`: Đảm bảo claim bị từ chối chính sách (policy denied) hoặc thiếu approval hợp lệ tuyệt đối không sinh ra bất kỳ giao dịch nào trong Ledger.
  4. `test_per_order_cap_enforcement_in_host_gate`: Kiểm tra cơ chế Host Authorization Gate ngăn chặn việc vượt trần bồi thường theo đơn hàng (áp dụng cho các ca T13, T14).
  5. `test_trace_contract_v1_compliance_and_sanitization`: Xác thực cấu trúc trace JSON và quét tự động các chuỗi nhạy cảm (`CANARY_SECRET`, token, phone, address).
- Tạo lập các bộ báo cáo định lượng chuẩn xác bằng công cụ của lab:
  - `src/reports/crash.json`: Sinh ra từ lệnh `python -m lab8.crash_test --count 20 --out reports/crash.json`, đạt chỉ số an toàn tuyệt đối `20/20`.
  - `src/reports/benchmark.json`: Sinh ra từ lệnh `python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json`, bao phủ 280 trials.
  - `src/reports/hidden_selfcheck.json`: Sinh ra từ đợt tự kiểm tra bộ hidden test drop (`student_drop/`), ghi nhận 600 trials không phát sinh lỗi (`error: 0`) và không có nghi vấn rò rỉ (`leak_suspect: 0`).
- Quyết định kỹ thuật:
  - Kiên quyết đề xuất lựa chọn kiến trúc **Workflow (Shape A)** trong ADR dựa trên tiêu chí "ít tự chủ nhất mà vẫn làm được việc": chi phí rẻ nhất (28.79 VND), phương sai bằng 0, loại bỏ hoàn toàn nguy cơ ảo giác (hallucination) so với Single Agent (+37.3% chi phí) và Swarm (trượt seed 55 do ảo giác).
  - Khẳng định tính an toàn của Ledger dựa trên SQLite transaction `BEGIN IMMEDIATE` và khóa duy nhất `refund:{claim_id}`.

## 4. Bằng chứng

- **Kết quả Unit Test Durability**:
  - `python -m pytest -q tests/test_durability_extra.py -o addopts=`: **5 passed in 0.45s**.
- **Kết quả Kiểm tra Ma trận Sinh viên**:
  - `python -m pytest -q -m student -o addopts=`: **248 passed, 199 deselected** (100% test của nhóm xanh hoàn toàn).
- **Crash Safety 20/20**:
  - `python -m lab8.crash_test --count 20 --out reports/crash.json`:
    `{"crash_safety": "20/20", "total_scenarios": 20, "passed": 20, "failed": 0}`.
- **Số liệu Benchmark Toàn phần** (trích xuất từ `src/reports/benchmark.json`):
  - `workflow`: Pass rate **1.000**, Pass^5 **1.000**, Variance **0.000000**, Cost/success **28.79 VND**, Mean Steps **2.71**.
  - `orchestrator`: Pass rate **1.000**, Pass^5 **1.000**, Variance **0.000000**, Cost/success **38.06 VND**, Mean Steps **4.36**.
  - `single`: Pass rate **1.000**, Pass^5 **1.000**, Variance **0.000000**, Cost/success **39.54 VND**, Mean Steps **8.21**.
  - `swarm`: Pass rate **0.986**, Pass^5 **0.929**, Variance **0.000816**, Cost/success **49.57 VND**, Mean Steps **2.80**.
- **An toàn Bảo mật & Trace**:
  - Quét 272 file trace trong `src/traces/`: 0 secret leak (`CANARY_SECRET_*`), 0 authorization bypass, 0 PII leak.
- **Tự kiểm tra Hidden Test**:
  - `src/reports/hidden_selfcheck.json`: 600 trials, `error_count: 0`, `leak_suspect: 0`.

## 5. Công cụ AI đã dùng

**Antigravity**, sử dụng vào các công việc:
- Hỗ trợ xây dựng kịch bản kiểm thử chuyên sâu trong `src/tests/test_durability_extra.py` theo đúng chuẩn pytest.

## 6. Tự đánh giá

- **Mức độ hoàn thành**: Hoàn thành 100% nhiệm vụ của vai trò TV4. Các tiêu chí nghiệm thu về tính bền vững (durability), crash resilience (20/20), idempotency, an toàn bảo mật không rò rỉ secret, và báo cáo benchmark toàn phần đều đạt chuẩn tối đa.
- **Bài học kinh nghiệm**:
  - Nhận thức sâu sắc về nguyên tắc "ít tự chủ nhất mà vẫn làm được việc": trong các hệ thống xử lý tài chính và chính sách doanh nghiệp, tính tiền định (determinism) và ranh giới kiểm soát của Host (Host Authorization Gate) quan trọng hơn nhiều so với việc để LLM tự do quyết định.
  - Hiểu rõ ranh giới bảo mật của Trace: Trace là bản ghi kiểm toán pháp lý chứ không đơn thuần là nhật ký gỡ lỗi (debug log), do đó việc áp dụng allowlist để lọc bỏ hoàn toàn PII và Secret là yêu cầu bắt buộc sống còn.
