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

Shape C ghi trace tại `RunResult.trace_path`. JSON có `run_id` khớp kết quả và danh sách `events`. `handoff` chỉ ghi payload đã qua `make_handoff()`, trong đó investigator được cấp `lookup_order`. `investigator` ghi tên tool và cờ `visible`. `quote`, `host` và `decision` ghi số tiền đề xuất, kết quả kiểm tra và mã lý do nghiệp vụ. Các nhánh kết thúc sớm dùng cùng `finish()` để ghi trace trong lần chạy hiện tại.

Shape C giữ ticket, claim/order thô, header, `parent_secret`, số điện thoại, địa chỉ, ghi chú nội bộ và tên hàng ngoài trace. Shape ghi một file trace trong thư mục `src/traces/` để scanner nhận diện qua `trace_path` và `run_id`. Test `test_orchestrator_handoff_is_read_only_and_trace_is_safe` đối chiếu payload bàn giao thực tế với event `handoff` và rà dữ liệu nhạy cảm trong trace, kết quả và dữ liệu in ra terminal.

## Giới hạn

MockLLM không phải model sản xuất. Latency để `unavailable`. Không sửa JSON cho đủ 0.75.
