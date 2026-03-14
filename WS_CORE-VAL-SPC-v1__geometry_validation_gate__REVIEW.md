WS_CORE-VAL-SPC-v1 — geometry_validation_gate (validators.py)

# 0. Document Control Block

|                                  |                                                                      |
| -------------------------------- | -------------------------------------------------------------------- |
| Canonical Code                   | WS_CORE-VAL-SPC-v1                                                   |
| Status                           | REVIEW                                                               |
| Run ID                           | 20260228T091639Z                                                     |
| WS / Module / Doc Type / Version | WS_CORE-VAL-SPC-v1                                                   |
| Created date (UTC)               | 2026-02-28 09:16:39Z                                                 |
| Purpose                          | Document governance consolidation only; no runtime/pipeline changes. |

## 1. Scope

Tài liệu này chuẩn hoá và gom mô tả module validation_gate (logical role: geometry_validation_gate) thuộc WS_CORE, dùng trước bước AGL freeze để xác thực B-Rep đủ điều kiện cho các phép đo DFM downstream.

Phạm vi V1 tập trung: non-manifold topology, solid closure/volume integrity, và self-intersection; kèm semantics fail-safe khi thiếu OCC.

Non-goals:

Không thực hiện healing; healing thuộc healer_agent.

Không chạy CAE/DFM scoring; chỉ là gate kiểm tra tính hợp lệ hình học.

Không mutate SSOT/AGL; gate chỉ đọc shape và trả result envelope.

## 2. Included Sources & Mapping Table

|           |                                                                                      |                                                      |                         |                                             |                                         |
| --------- | ------------------------------------------------------------------------------------ | ---------------------------------------------------- | ----------------------- | ------------------------------------------- | --------------------------------------- |
| source_id | filename                                                                             | original_title                                       | original_version/status | sections_used                               | notes                                   |
| S1        | TÀI LIỆU TRIỂN KHAI ADN AMBR LDR — MODULE validators (geometry_validation_gate).docx | validators (geometry_validation_gate) — DFM-first V1 | DRAFT (source)          | ADN/AMBR/LDR (primary)                      | Primary DFM-first V1 research write-up. |
| S2        | TÀI LIỆU TRIỂN KHAI ADN AMBR LDR — MODULE geometry_validation_gate .docx             | geometry_validation_gate (validators.py)             | DRAFT (source)          | Schema sketch + starter defaults (appendix) | Supplementary schema/defaults notes.    |

## 3. Consolidated Specification Content

## 3.1 ADN (Algorithm / Logic)

Source: S1

WHY (DFM-first)

Gate này không phải “IT cho đẹp”. Nó là rào chắn để các phép đo DFM downstream không bị “ảo”:

draft/undercut: normal trên B-Rep hỏng có thể đảo/nhảy → false undercut, false reverse draft.

thickness/sink/ribratio: ray/probe trên open shell/self-intersection → miss/hit bậy → thickness giả.

Vì vậy, nếu gate còn stub True thì báo cáo DFM sẽ mất uy tín.

SCOPE

V1 phải kiểm tối thiểu 3 nhóm:

Non-manifold topology: edge có số face kề > 2 (hoặc các dấu hiệu topo “không là solid chuẩn”).

Solid closure / volume integrity: shape đóng kín và volume > 0.

Self-intersection: phát hiện chắc chắn hoặc ít nhất “suspected” theo checker khả dụng (xem ghi chú [CHƯA XÁC MINH] về API).

Non-goals:

Gate không tự heal (healing thuộc healer_agent).

Gate không CAE, không DFM scoring.

PRECONDITIONS

Có brep_shape từ AGL (pre-freeze).

Nếu thiếu OCC: gate phải fail-safe (NOT_EXECUTED/STUB_MODE) và pipeline vẫn report.

INPUTS

payload (dict), tối thiểu:

geometry_context.brep_shape [CHƯA XÁC MINH: key path chính xác]

run_id (để audit/trace) [CHƯA XÁC MINH: injection point]

rule_version (policy-as-config) [CHƯA XÁC MINH]

geometry_version_id: có thể chưa có trước freeze; cho phép null hoặc temporary_id [CHƯA XÁC MINH: quy ước temp id]

policy keys (khuyến nghị đặt dưới dfm_policy.thresholds.validation):

enabled: true/false

self_intersection_check_mode: "FAST" | "FULL" (default FAST)

min_volume_mm3: float (default nhỏ để chặn degenerate)

max_findings_to_report: int (default 20)

nonmanifold_allowed: 0

ACTIONS (V1 algorithm steps)

Step 1 — Init envelope GEOMETRY_VALIDATION_RESULT_v0

status=OK (tentative), findings=[], issues=[].

Step 2 — Input & OCC availability

thiếu brep_shape: status=FAILED, error_code=GEOM_CONTEXT_MISSING.

OCC missing: status=NOT_EXECUTED hoặc STUB_MODE, error_code=OCC_MISSING, disable_reason="OCC dependency missing" (không crash).

Step 3 — Non-manifold check

Tạo adjacency map Edge → FaceCount.

Nếu có edge với FaceCount > 2 → FAIL non-manifold.

Nếu có thể: ghi “một vài” offending edges bằng region_ref kind=EDGE, id_scheme=AGL_EDGE_INDEX_V1. (Nếu chưa có topo index canonical thì ghi issue REGION_REF_NOT_CANONICAL.)

Step 4 — Solid closure / volume integrity

Kiểm shape có tính “closed solid” hoặc ít nhất “closed shell” hợp lệ.

Tính volume_mm3; nếu volume \<= min_volume_mm3 → FAIL.

Ghi chú: API OCC cho volume/closure chính xác cần chốt khi vào code. [CHƯA XÁC MINH]

Step 5 — Self-intersection check

Mode FAST: dùng checker sẵn có để phát hiện lỗi giao cắt rõ ràng (nếu checker không đủ mạnh thì phải trả “SELF_INTERSECTION_NOT_IMPLEMENTED” thay vì giả vờ PASS).

Mode FULL: chạy checker nặng hơn (đắt hơn), chỉ bật khi cần.

Ghi chú: tên API OCC cho self-intersection check cần xác minh khi vào code. [CHƯA XÁC MINH]

Step 6 — Package decision-support fields (điểm quan trọng cho orchestrator)

Gate không chỉ trả PASS/FAIL, mà phải trả “healing_candidate” và “retryable”:

retryable=true nếu lỗi do thiếu OCC (có thể retry khi cài deps).

healing_candidate: LIKELY / POSSIBLE / UNLIKELY (xem mục “Decision fields” bên dưới).

Mục tiêu: orchestrator sau này có thể quyết “heal again” hay “fallback report” mà không suy diễn.

OUTPUTS

Gate trả GEOMETRY_VALIDATION_RESULT_v0 (dict). Đây là phần mình “khóa” rõ nhất để khỏi drift.

GEOMETRY_VALIDATION_RESULT_v0 — schema sketch (bắt buộc cho orchestration)

result_type: "GEOMETRY_VALIDATION_RESULT_v0"

run_id: "YYYYMMDDThhmmssZ" (bắt buộc)

status: "OK" | "FAILED" | "NOT_EXECUTED" | "STUB_MODE"

error_code: null | "GEOM_CONTEXT_MISSING" | "OCC_MISSING" | "VALIDATION_FAILED"

disable_reason: string|null (required if NOT_EXECUTED/STUB_MODE)

rule_version: string [CHƯA XÁC MINH: RULESET_v1 wiring]

geometry_version_id: string|null

gate_decision: "PASS" | "FAIL" | "DEFERRED"

(DEFERRED dùng cho NOT_EXECUTED/STUB_MODE)

retryable: bool

(true nếu lý do thuộc loại “môi trường” như OCC_MISSING)

healing_candidate: "LIKELY" | "POSSIBLE" | "UNLIKELY" | "UNKNOWN"

fail_reason_codes: list[string]

ví dụ: ["NON_MANIFOLD", "OPEN_SHELL", "ZERO_VOLUME", "SELF_INTERSECTION"]

metrics:

nonmanifold_edge_count: int|null

open_shell_detected: bool|null

volume_mm3: float|null

self_intersection_detected: bool|null

self_intersection_check_mode: "FAST"|"FULL"|null

findings (bounded list, max_findings_to_report):

item:

code: "NON_MANIFOLD_EDGE" | "OPEN_SHELL" | "ZERO_VOLUME" | "SELF_INTERSECTION"

severity: "ERROR" | "FATAL"

region_ref: REGION_REF_SCHEMA_v1 | null

message: string

issues:

issue_obj (POLICY_UNRESOLVED, REGION_REF_NOT_CANONICAL, SELF_INTERSECTION_NOT_IMPLEMENTED, …)

receipts_ref: [CHƯA XÁC MINH]

Decision semantics (để orchestrator dùng sau này, không cần heal-loop contract ngay)

gate_decision=PASS: không có fail_reason_codes; metrics hợp lệ.

gate_decision=FAIL: ít nhất 1 fail_reason_code.

healing_candidate:

LIKELY: OPEN_SHELL (có thể sewing), ZERO_VOLUME do closure lỗi (có thể fix), một số invalidity “ShapeFix hay chữa được”.

POSSIBLE: self-intersection nhẹ / tolerance issues / small gaps.

UNLIKELY: non-manifold nặng, self-intersection cấu trúc (thường là lỗi thiết kế, heal khó hoặc làm drift).

UNKNOWN: nếu checker không đủ hoặc thiếu evidence.

Lưu ý: mapping này nên nằm trong policy về sau; ở research mình chốt semantics trước để bạn đồng thuận.

EVIDENCE & OBSERVABILITY (DFM-first)

Gate phải cung cấp:

metrics đủ để thấy “tại sao fail”.

findings (bounded) để kỹ sư biết “ở đâu” (edge/face) mà không lưu bất kỳ file path/asset bền vững.

FAIL-FAST CONDITIONS

OCC missing → NOT_EXECUTED/STUB_MODE, tuyệt đối không crash.

Self-intersection check không implement được ở mode FAST → issue SELF_INTERSECTION_NOT_IMPLEMENTED, và healing_candidate=UNKNOWN (không được “PASS giả”). [CHƯA XÁC MINH]

TROUBLESHOOTING (gợi ý vận hành)

Nếu fail OPEN_SHELL: ưu tiên xem lại import/heal sewing tolerance và kiểm xem model có phải surface model (không solid) hay không.

Nếu fail NON_MANIFOLD: thường là lỗi thiết kế/topology khó sửa tự động; nên report rõ và dừng heal-loop sớm (tránh lãng phí).

Nếu fail SELF_INTERSECTION: khuyến nghị FULL mode cho vài mẫu trong corpus để đánh giá độ nhạy.

## 3.2 AMBR (Bindings / Module Contract / Execution)

Source: S1

C) AMBR — module binding

ambr_id: AMBR_validators_geometry_validation_gate_v1_research

status: DRAFT

module_name: validators

module_type: provider / validation_gate

execution_mode: SEQUENTIAL

pipeline_stage: Step 3 (pre-freeze) theo Master Spec.

bound_algorithm: ADN_validators_brep_integrity_gate_nonmanifold_selfintersection_volume_v1

depends_on: pythonocc-core (OCC.Core)

## 3.3 LDR (Dependencies / Libraries / Runtime constraints)

Source: S1

D) LDR — dependencies

ldr_id: LDR_pythonocc_core_geometry_validation_gate_stack_v1

status: DRAFT

library_name: pythonocc-core (OCC.Core)

purpose: topology adjacency, closure/volume checks, self-intersection checks (nếu có)

fail-safe: OCC missing → gate_decision=DEFERRED, status NOT_EXECUTED/STUB_MODE.

## 4. Open Questions / [CHƯA XÁC MINH]

API/key path chính xác để lấy geometry_context.brep_shape trong payload. Nếu không chốt, orchestrator không thể inject input một cách nhất quán và gate sẽ dễ FAIL giả.

Cách wiring rule_version (ví dụ RULESET_v1) và nguồn policy-as-config cho thresholds.validation.\*. Nếu không chốt, tính tái lập (determinism) và auditability của gate_result sẽ không ổn định giữa các run.

API OCC cụ thể cho (a) closure/volume properties và (b) self-intersection checker (FAST/FULL). Nếu không xác minh, phần 'self_intersection_check_mode' có nguy cơ chỉ là mô tả tài liệu mà không triển khai được.

Quy ước geometry_version_id trước freeze (temporary_id) và receipts_ref wiring. Nếu không có, traceability end-to-end (receipts/defects/report) bị đứt.

Giá trị mặc định min_volume_mm3 và hành vi khi self-intersection check chưa implement ở FAST (PASS/FAIL/DEFERRED). Nếu không chốt, kết quả gate có thể không nhất quán giữa môi trường và corpus.

## 5. Change Log (Editorial)

20260228T091639Z: Consolidated S1+S2 into canonical WS_CORE-VAL-SPC-v1; added Document Control Block + Mapping Table + standardized headings; removed truncated cross-reference placeholders (e.g., 'TÀI LIỆU …') without changing technical semantics.

## 6. Appendix

## 6.1 Aka / Naming notes

Source: S1/S2

Implementation file name in sources: validators.py. Logical role name used for clarity: geometry_validation_gate. No semantic rename is implied; this is an alias mapping only.

## 6.2 GEOMETRY_VALIDATION_RESULT_v0 schema sketch (supplementary)

Source: S2

E) GEOMETRY_VALIDATION_RESULT_v0 — schema sketch

GEOMETRY_VALIDATION_RESULT_v0:

result_type: "GEOMETRY_VALIDATION_RESULT_v0"

run_id

status: "OK" | "FAILED" | "NOT_EXECUTED" | "STUB_MODE"

error_code: null | "GEOM_CONTEXT_MISSING" | "OCC_MISSING" | "VALIDATION_FAILED"

disable_reason: string|null (required if NOT_EXECUTED/STUB_MODE)

rule_version: "RULESET_v1" [CHƯA XÁC MINH]

geometry_version_id: string|null [CHƯA XÁC MINH]

metrics:

nonmanifold_edge_count: int|null

self_intersection_detected: bool|null

open_shell_detected: bool|null

volume_mm3: float|null

check_mode: "FAST"|"FULL"|null

findings: list (optional; giới hạn số lượng để không phình)

item:

code: "NON_MANIFOLD_EDGE"|"SELF_INTERSECTION"|"OPEN_SHELL"|"ZERO_VOLUME"

severity: "ERROR"|"FATAL"

region_ref: REGION_REF_SCHEMA_v1 | null

message: string

issues: list[issue_obj] (POLICY_UNRESOLVED, OCC_MISSING, etc.)

produced_defects: [] (thường gate không tạo defect DFM; nó tạo “validation finding”. Nếu bạn muốn lưu vào analysis_results_stack thì coi như defect loại GEOMETRY_INVALID) [CHƯA XÁC MINH: bạn muốn gate findings đi vào defect stack hay chỉ ở result]

receipts_ref: [CHƯA XÁC MINH]

Region_ref cho gate:

Non-manifold: lý tưởng là EDGE index (AGL_EDGE_INDEX_V1).

Open shell: EDGE boundary (AGL_EDGE_INDEX_V1) hoặc GLOBAL nếu không xác định.

Self-intersection: FACE indices (AGL_FACE_INDEX_V1) nếu checker trả được.

Nếu không định vị được, để region_ref null và ghi rõ limitation; không được bịa vị trí.

## 6.3 Starter defaults (supplementary)

Source: S2

F) “starter defaults” policy cho geometry_validation_gate (đủ để chạy nghiên cứu)

Nếu bạn muốn set ngay policy để gate có hành vi xác định:

dfm_policy.thresholds.validation:

enabled: true

max_healing_iterations: 3 (theo Master Spec loop)

nonmanifold_allowed: 0

min_volume_mm3: 1.0 (chỉ để chặn zero/degenerate; giá trị này bạn có thể tăng nếu gặp chi tiết cực nhỏ) [CHƯA XÁC MINH]

self_intersection_check_mode: "FAST"
