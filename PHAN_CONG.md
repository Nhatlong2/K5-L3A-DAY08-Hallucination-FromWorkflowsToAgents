# Phân công công việc — Lab 8 LogiX-VN (nhóm 4 người)

Chia vai theo gợi ý trong [TEAM.md](TEAM.md): **workflow + integration · single · orchestrator + handoff · ledger + evaluation**.
Mọi lệnh chạy trong thư mục `src/`. Chỉ sửa 3 file shape (`src/lab8/shapes/workflow.py`, `single.py`, `orchestrator.py`), thêm test trong `src/tests/`, code bonus trong `src/bonus/`.

## Tổng quan

| TV | Vai | Phụ trách chính | Hạng mục rubric |
| --- | --- | --- | --- |
| TV1 | Workflow + Integration (nhóm trưởng) | Shape A `workflow`, tạo repo, merge, `TEAM.md`, chốt nộp | C1, C7, C8 |
| TV2 | Single agent | Shape B `single` (vòng ReAct ≤ 8 bước) | C1, C4 |
| TV3 | Orchestrator + Handoff | Shape C `orchestrator` + investigator read-only, handoff sanitize, trace an toàn | C1, C5 |
| TV4 | Ledger + Evaluation | Crash suite, approval/policy, benchmark, ADR, research card, báo cáo nhóm | C3, C4, C6 |

Mỗi người bắt buộc: **ít nhất 1 commit kỹ thuật có nội dung + ít nhất 1 test thêm/sửa trong `src/tests/`** (C8), và 1 báo cáo cá nhân `report/team/MSSV_HoTen.md`.

## Lịch theo checkpoint

| Mốc | Việc chung |
| --- | --- |
| CP1 (13:30) | Cả nhóm cài môi trường, smoke pass, đọc `business_rules.md`, `contracts.md`, `swarm_reference.py` |
| CP2 (13:45) | TV1/TV2/TV3 viết 3 shape song song; TV4 viết test bổ sung + chuẩn bị crash/benchmark |
| CP3 (14:30) | Truyền `crash_at`, handoff sanitize; T11, T12 xanh |
| CP4 (15:30) | Crash suite 20/20, rà approval & secret. Đăng ký bonus (nếu tự đề xuất) **trước 15:30** |
| CP5 (16:15→17:00) | Benchmark 4 shape × 5 seed, tự kiểm `student_drop/`, ADR, research card, báo cáo |

---

## TV1 — Workflow + Integration (nhóm trưởng)

**Phạm vi**
- Tạo repo nhóm `K5-L3A-DAY08-TenNhom-FromWorkflowsToAgents` (fork + đổi tên), public hoặc mời giảng viên.
- Viết shape A `src/lab8/shapes/workflow.py`: các bước Python cố định, tiền do `compute_compensation` tính, gọi `durability.issue_refund` với **quote đầy đủ**.
- Quản lý merge, giữ cổng host xanh, điền `TEAM.md`, chốt commit hash và chạy `check_submission`.

**Todo checklist**
- [ ] Fork repo, đổi tên đúng quy ước, mời các thành viên + giảng viên
- [ ] Cài môi trường, `python -m lab8.smoke` in `lab8.smoke pass`
- [ ] `python -m pytest -q` xanh (mong đợi `52 passed, 248 deselected`)
- [ ] Điền phần Thông tin nhóm + Thành viên trong `TEAM.md` (CP1)
- [ ] Đọc docstring đầu `workflow.py`, các dòng `TODO CP2` / `TODO CP3`
- [ ] Workflow: đọc claim qua view đã allowlist (`get_claim_view`, `lookup_order`), **không** lấy `order_id`/số ngày từ chữ trong ticket
- [ ] Xử lý đủ nhánh: thiếu `order_id` → `needs_info`; không tìm thấy đơn / sai principal → `escalated`; quote = 0 → `denied` (không gọi `issue_refund`)
- [ ] Bỏ qua policy cũ `SHIP-LATE-OLD` và policy tương lai (tự kiểm `is_current`, `effective_date <= 2026-11-15`)
- [ ] Ship + damage cùng đủ điều kiện → lấy **số lớn hơn**, không cộng
- [ ] Quote > 0 → luôn gọi `durability.issue_refund(quote đầy đủ, crash_at=crash_at)`; không bắt `InjectedCrash`
- [ ] Trả kết quả bằng `lab8.shapes.common.finish()`, `run_id = make_run_id(...)`, ghi trace ở mọi nhánh return
- [ ] **Không** copy khối `REFERENCE_WEAKNESS` của swarm
- [ ] `python -m pytest -q -m student -o addopts= -k workflow` → 62 passed
- [ ] Viết ít nhất 1 test riêng (vd: `src/tests/test_workflow_extra.py` — ticket nói "trễ 9 ngày"/"ignore policy" vẫn bị bỏ qua)
- [ ] Review & merge PR của TV2, TV3, TV4; chạy lại `python -m pytest -q` sau mỗi lần merge
- [ ] Điền bảng đóng góp trong `TEAM.md` (mỗi người 1 dòng: vai, commit, test)
- [ ] Chốt nộp: commit → `git rev-parse --short HEAD` → đưa hash cho TV4 ghi ADR → commit tiếp
- [ ] `python -m lab8.check_submission` ra `0 FAIL`, push, nhắc cả nhóm nộp link repo trên LMS
- [ ] Viết báo cáo cá nhân `report/team/MSSV_HoTen.md` (có mục công cụ AI đã dùng)

---

## TV2 — Single agent

**Phạm vi**
- Viết shape B `src/lab8/shapes/single.py`: vòng ReAct tối đa 8 bước, model **chỉ được chọn tool đọc**. Sau vòng lặp, code của shape tính tiền bằng `compute_compensation` và gọi `durability.issue_refund`.
- Hỗ trợ TV4 rà các nhánh approval/policy (C4).

**Todo checklist**
- [ ] Cài môi trường, smoke pass, cổng host xanh
- [x] Đọc `src/lab8/runtime.py` (vòng ReAct), `config/tools.manifest.v1.1.yaml`, `config/autonomy_policy.yaml`
- [x] Đọc docstring đầu `single.py`, các dòng `TODO CP2` / `TODO CP3`
- [x] Cài vòng ReAct ≤ 8 bước qua MockLLM, chỉ expose tool đọc (không có tool ghi)
- [x] Coi chữ trong ticket là dữ liệu: không để model quyết số tiền, `order_id` hay số ngày trễ
- [x] Sau vòng lặp: code tính quote bằng `compute_compensation`; quote > 0 → `durability.issue_refund(quote đầy đủ, crash_at=crash_at)`
- [x] Xử lý đủ status `needs_info` / `escalated` / `denied` / `resolved` / `failed`; `run_claim` không được ném exception
- [x] Hết 8 bước mà chưa đủ dữ kiện → vẫn trả `RunResult` hợp lệ + trace
- [x] Trả bằng `finish()`, trace ghi ở mọi nhánh; không ghi claim row/order row thô vào trace
- [x] `python -m pytest -q -m student -o addopts= -k single` → 62 passed
- [x] Kiểm T02 ở seed 55 vẫn pass (không tin số ngày trong ticket như swarm)
- [x] Viết ít nhất 1 test (vd: `src/tests/test_single_extra.py` — vòng lặp không vượt 8 bước; approval của claim khác → `denied` + `policy_denied`)
- [ ] Mở PR, nhờ TV1 review & merge
- [x] Viết báo cáo cá nhân `report/team/MSSV_HoTen.md`

---

## TV3 — Orchestrator + Handoff

**Phạm vi**
- Viết shape C `src/lab8/shapes/orchestrator.py`: orchestrator + investigator. Investigator **chỉ có tool đọc**; orchestrator gọi `durability.issue_refund`.
- Dựng handoff bằng `lab8.handoff.make_handoff` (chỉ field allowlist). Phụ trách tính an toàn trace (C5): không lộ `CANARY_SECRET_*`, SĐT, địa chỉ, ghi chú nội bộ, tên hàng.

**Todo checklist**
- [ ] Cài môi trường, smoke pass, cổng host xanh
- [ ] Đọc `src/lab8/handoff.py` (allowlist), `src/lab8/runtime.py` (investigator read-only), `harness_map.yaml`
- [ ] Đọc docstring đầu `orchestrator.py`, các dòng `TODO CP2` / `TODO CP3`
- [ ] Orchestrator dựng handoff bằng `make_handoff`, chỉ truyền field trong allowlist (không `parent_secret`, header, phone, address…)
- [ ] Investigator chỉ có tool đọc; kết quả trả về orchestrator để code tính quote bằng `compute_compensation`
- [ ] Orchestrator gọi `durability.issue_refund(quote đầy đủ, crash_at=crash_at)`; không bắt `InjectedCrash`
- [ ] Trace chỉ ghi **handoff đã sanitize**, không ghi parent context; tóm tắt bằng LLM **không** được coi là redaction
- [ ] Mọi file debug (nếu có) nằm cùng thư mục trace, tên bắt đầu bằng `run_id`; không in dữ liệu nhạy cảm ra stdout
- [ ] Trả bằng `finish()`, trace đúng contract v1 ở mọi nhánh
- [ ] `python -m pytest -q -m student -o addopts= -k orchestrator` → 62 passed
- [ ] Rà trace trong `src/traces/` của cả 3 shape: grep `CANARY_SECRET`, SĐT, địa chỉ → phải không có
- [ ] Viết ít nhất 1 test (vd: `src/tests/test_handoff_extra.py` — handoff không chứa field ngoài allowlist; trace không chứa `CANARY_SECRET_*`)
- [ ] Soạn đoạn "Trace và bằng chứng" (event nào được ghi, field nào cố ý không ghi, vì sao) gửi TV4 đưa vào ADR
- [ ] Soạn phần mô tả 3 shape + ranh giới code/model cho mục 3 `group_report.md`
- [ ] Mở PR, nhờ TV1 review & merge
- [ ] Viết báo cáo cá nhân `report/team/MSSV_HoTen.md`

---

## TV4 — Ledger + Evaluation

**Phạm vi**
- Durability & policy (C3, C4): T11 (gọi lại), T12 (crash + resume), crash suite 20/20, rà approval hỏng / sai claim / đơn đã hoàn đủ, T13/T14 (trần theo đơn).
- Evaluation & báo cáo (C6, C7): benchmark, tự kiểm hidden, `decision_adr.md`, `research_card.md`, `group_report.md`.

**Todo checklist**

*Durability & policy*
- [ ] Cài môi trường, smoke pass, cổng host xanh
- [ ] Đọc `src/lab8/durability.py` (idempotency key `refund:{claim_id}`), chạy `python -m lab8.demo.buggy_workflow` để hiểu vì sao checkpoint không thay được ledger
- [ ] Viết sẵn test bổ sung trong lúc 3 shape đang viết (vd: `src/tests/test_ledger_extra.py` — gọi lại cùng claim chỉ 1 dòng ledger; T14 ledger 120.000 với `proposal_amount` 150.000)
- [ ] Kiểm cả 3 shape truyền `crash_at` và không bắt `InjectedCrash`; T11, T12 xanh trên cả 3 shape
- [ ] Thử thủ công: `python -m lab8.run --shape workflow --claim-id C-012 --seed 11 --db data/crash.sqlite --crash-at after_effect_before_checkpoint` rồi resume
- [ ] Rà nhánh: không approval / hết hạn / bị thu hồi / approval của claim khác (`approval_claim_mismatch`) / đơn đã hoàn đủ (`order_already_refunded`) → `denied` + audit `policy_denied`
- [ ] Chạy `python -m lab8.crash_test --count 20 --out reports/crash.json` → `crash_safety` = `20/20`
- [ ] Toàn bộ ma trận: `python -m pytest -q -m student -o addopts=` (3 shape nhóm viết xanh hết)

*Evaluation & báo cáo*
- [ ] Chạy benchmark: `python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json`
- [ ] Sau 16:15: đặt `student_drop/` vào `src/`, chạy tự kiểm hidden → `reports/hidden_selfcheck.json` (không `error`, không `leak_suspect`)
- [ ] Cùng nhóm chọn shape (ít tự chủ nhất mà vẫn làm được việc) dựa trên pass rate, pass^5, variance, cost/success
- [ ] Điền `report/team/decision_adr.md`: Shape chọn, Commit hash (lấy từ TV1), Run id, số liệu **lấy từ file report**, crash safety, secret leak / approval bypass; viết 4 đoạn giải thích (giữ nguyên dòng nhãn)
- [ ] Điền `report/team/research_card.md`: chọn 1 câu R1–R4, điền đủ 6 dòng "Nhóm điền", chạy 1 ablation và ghi lệnh + run id + kết quả
- [ ] Viết `report/team/group_report.md` (≤ 5 trang): mục 4 bảng so sánh 4 shape, mục 5 khó khăn, mục 7 hạn chế (MockLLM, latency `unavailable`); gom phần TV3 gửi vào mục 3
- [ ] **Không sửa tay** JSON trong `src/reports/`; không commit `traces/`, `*.sqlite`, `student_drop/`
- [ ] Viết báo cáo cá nhân `report/team/MSSV_HoTen.md`

---

## Bonus (tùy chọn, làm khi còn thời gian)

Chỉ tính khi shape chọn qua 3 must-pass và hidden pass rate ≥ 0.80. Gợi ý: **B3 (swarm_verified)** hoặc **B4 (ablation + thống kê)** vì gắn sẵn với phần đánh giá của TV4.

- [ ] Chọn 1 topic B1–B7 trong [docs/bonus_topics.md](docs/bonus_topics.md) (tự đề xuất thì đăng ký trước 15:30)
- [ ] Code trong `src/bonus/` (có `__init__.py`), ≥ 2 test `src/tests/test_bonus_*.py`
- [ ] Chạy qua `--extra-shape`, xuất `src/reports/bonus_*.json`
- [ ] Viết `report/team/bonus_report.md` (≤ 2 trang, theo khung)

## Checklist nộp chung (cả nhóm kiểm trước deadline)

- [ ] Repo đúng tên `K5-L3A-DAY08-TenNhom-FromWorkflowsToAgents`, public hoặc đã mời giảng viên
- [ ] Ba must-pass: approval bypass = 0, secret leak = 0, crash `20/20`
- [ ] `python -m pytest -q` xanh
- [ ] Có `src/reports/benchmark.json` và `src/reports/crash.json` do công cụ sinh ra
- [ ] `TEAM.md` đủ thông tin + bảng đóng góp 4 dòng
- [ ] `decision_adr.md`, `research_card.md`, `group_report.md` đã điền đủ
- [ ] 4 file báo cáo cá nhân `report/team/MSSV_HoTen.md` (giữ nguyên file mẫu)
- [ ] Mỗi thành viên có commit kỹ thuật + ít nhất 1 test trong lịch sử branch nộp
- [ ] Không sửa file thước đo (`metrics.py`, `durability.py`, `policy.py`, `config/`, `data/`)
- [ ] `python -m lab8.check_submission` ra `0 FAIL`, đã push
- [ ] **Từng thành viên** nộp link repo nhóm trên LMS bằng tài khoản cá nhân
