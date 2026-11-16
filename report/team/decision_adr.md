# Decision record

Điền sau dấu `:` trên cùng dòng, không xóa nhãn. Giảng viên và `python -m lab8.check_submission` (chạy trong `src/`) đọc các dòng này bằng máy. Không đổi tên hay dời file này khỏi `report/team/`.
`Shape chọn` phải là đúng một trong: `workflow`, `single`, `orchestrator`, `swarm`. Đây là shape được chấm cổng hidden.

- Shape chọn: workflow
- Commit hash: 9b5711a
- Run id dẫn chứng: R-workflow-C-001-s11
- Pass rate / pass^5 / seed min-max / variance: 1.000 / 1.000 / 1.000 - 1.000 / 0.000000
- Cost per success (mô phỏng, VND): 28.79
- Crash safety: 20/20 (src/reports/crash.json)
- Secret leak / approval bypass: 0 / 0

Ví dụ một dòng đã điền: `- Shape chọn: workflow`, `- Crash safety: 20/20 (src/reports/crash.json)`.

## Vì sao shape này

Viết 1 đoạn. So cùng corpus và cùng seed, không so với một lần chạy lẻ.

Nhóm chọn kiến trúc `workflow` (Shape A) dựa trên tiêu chí cốt lõi: "Chọn kiến trúc ít tự chủ nhất mà vẫn làm được việc". Trong bài toán xử lý claim hoàn phí của LogiX-VN, các luật nghiệp vụ (policy) về tính phí bồi thường, thời hạn hợp lệ và trần đơn hàng đều cố định, rõ ràng và có tính ràng buộc pháp lý/tài chính chặt chẽ. Việc sử dụng luồng code Python xác định (deterministic logic qua if/else) cho phép giải quyết chính xác 100% các tình huống mà không cần giao quyền tự chủ cho LLM suy luận, loại bỏ hoàn toàn nguy cơ ảo giác (hallucination). Trên cùng tập corpus public (14 claims × 5 seeds = 70 trials), Workflow đạt Pass rate tuyệt đối 1.000, Pass^5 1.000, Seed Variance 0.000000 và chi phí mô phỏng thấp nhất (28.79 VND/success, rẻ hơn 37.3% so với Single Agent và rẻ hơn 72.2% so với Swarm), đồng thời số bước xử lý ngắn nhất (2.71 steps).

## Có đáng đổi lấy multi-agent không

Nếu pass nhỉnh hơn mà cost và variance cao hơn, nhóm giữ hay đổi? Điều kiện nào làm quyết định đảo lại?

Hoàn toàn không đáng đánh đổi trong bài toán này. Thực nghiệm so sánh cùng corpus và cùng seed cho thấy kiến trúc đa tác tử (Swarm) không những làm tăng vọt chi phí lên 49.57 VND/success (+72.2%) và tạo ra phương sai dao động (variance = 0.000816), mà còn làm giảm Pass rate xuống 0.986 do rủi ro khuếch đại lỗi và model override (ở seed 55, Swarm tin vào số ngày trễ người dùng khai trong ticket thay vì dữ liệu thật trong order, dẫn đến tính sai quote và bị host từ chối). Nhóm kiên quyết giữ Workflow. Quyết định chỉ nên đảo ngược sang multi-agent khi bài toán xuất hiện các luồng nghiệp vụ không có cấu trúc cố định, dữ liệu đầu vào đòi hỏi thương lượng đa phương, hoặc cần tổng hợp ngữ nghĩa mở phức tạp từ nhiều nguồn phi cấu trúc mà code tuần tự không thể bao quát.

## Trace và bằng chứng

Trace của nhóm ghi những event nào? Field nào cố ý **không** ghi, và vì sao? Nếu trace nằm ngoài chỗ scanner quét được, đội security chứng minh thế nào là hệ thống không lộ secret?

Trace của Workflow ghi nhận minh bạch các sự kiện quan trọng trong vòng đời xử lý claim: `claim_view`, `lookup_order`, `quote` (tính từ `compute_compensation`), `refund_committed` (hoặc `policy_denied`). Các trường nhạy cảm như `parent_secret` (`CANARY_SECRET_*`), `phone`, `address`, `internal_note`, `item_name`, và `raw_headers_json` được cố ý loại bỏ ngay từ tầng đọc dữ liệu (`get_claim_view`, `lookup_order`) theo nguyên tắc allowlist để ngăn ngừa tuyệt đối rò rỉ PII và secret. Nếu trace nằm ngoài vùng scanner tự động, đội security chứng minh bằng chứng thực nghiệm: (1) Kết quả quét tự động 100% file trace sinh ra với `secret_leak_count = 0` và 0 rò rỉ PII; (2) Kết quả Crash Test Suite đạt 20/20 (`src/reports/crash.json`) chứng minh Idempotency Key `refund:{claim_id}` ngăn chặn hoàn toàn việc ghi trùng tiền; (3) Host Authorization Gate luôn đứng giữa để kiểm tra trần đơn hàng và approval hợp lệ trước khi ghi Ledger.

## Giới hạn

MockLLM không phải model sản xuất. Latency để `unavailable`. Không sửa JSON cho đủ 0.75.
