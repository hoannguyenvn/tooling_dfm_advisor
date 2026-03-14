WS_PROVIDERS-HEA-SPC-v1 — healer_agent (REVIEW)

|                                  |                                                                                                                         |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| Canonical Code                   | WS_PROVIDERS-HEA-SPC-v1                                                                                                 |
| Status                           | REVIEW                                                                                                                  |
| Run ID (UTC)                     | 20260304T082854Z                                                                                                        |
| Created date (UTC)               | 2026-03-04T08:28:54Z                                                                                                    |
| WS / Module / Doc Type / Version | WS_PROVIDERS / HEA / SPC / v1                                                                                           |
| Purpose                          | Document governance consolidation only; rename + consolidate sources into a canonical SPC. No runtime/pipeline changes. |

# 1. Scope

Source: S1

Scope (facts) BRepCheck_Analyzer pre-check và post-check.

Non-goals (Source: S1)

Non-goals (DFM-first) Không “làm đẹp” hình học vượt mức: không tự tạo/di chuyển features, không “làm dày/mỏng” thành, không thay đổi draft theo hướng rút.

## 2. Included Sources & Mapping Table

|           |                                                             |                                                                        |                                                                    |                                          |                                                                              |
| --------- | ----------------------------------------------------------- | ---------------------------------------------------------------------- | ------------------------------------------------------------------ | ---------------------------------------- | ---------------------------------------------------------------------------- |
| source_id | filename                                                    | original_title                                                         | original_version/status                                            | sections_used                            | notes                                                                        |
| S1        | TÀI LIỆU TRIỂN KHAI ADN AMBR LDR — MODULE healer_agent.docx | TÀI LIỆU TRIỂN KHAI ADN / AMBR / LDR — MODULE healer_agent (DFM-first) | TOOLING_DFM_ADVISOR_HEALER_AGENT_DFM_FIRST_V1_RESEARCH_v1 / DRAFT  | ADN, AMBR, LDR extracts                  | main healer_agent module document (DFM-first)                                |
| S2        | HEALER_AGENT_OBSERVABILITY_ADDENDUM_v1.docx                 | HEALER_AGENT_OBSERVABILITY_ADDENDUM_v1                                 | TOOLING_DFM_ADVISOR_HEALER_AGENT_OBSERVABILITY_ADDENDUM_v1 / DRAFT | AMBR (observability/contract) + Appendix | observability addendum for HEAL_RESULT_v0; consolidated as verbatim extracts |

## 3. Consolidated Specification Content

## 3.1 ADN (Algorithm / Logic)

Source: S1

B) ADN — healer_agent (DFM-first)

IDENTIFICATION
adn_id: ADN_healer_agent_minimal_fix_healing_with_brepcheck_shapefix_v0 (baseline)
status: APPROVED (theo tài liệu publish)

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

DFM-first extension note: Các mục dưới đây mô tả “cách dùng đúng” và “bổ sung observability/policy knobs” để healer phục vụ DFM tốt hơn; phần nào chưa có bằng chứng trong code sẽ gắn [CHƯA XÁC MINH].

PROBLEM STATEMENT
Core problem: đưa B-Rep về trạng thái “đủ hợp lệ” để:

validation gate có thể ra PASS/FAIL có ý nghĩa,

downstream agents (draft/undercut/thickness/sink/ribratio) không bị sai vì lỗi topo/hình học.
Healer thực hiện minimal-fix: check validity trước/sau, chỉ tạo healed shape khi cần, và fail-safe khi thiếu OCC.

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

Scope (facts)

BRepCheck_Analyzer pre-check và post-check.

Nếu invalid: ShapeFix_Shape.Perform() và lấy shape_healed.

Không mutate shape_in in-place; output geometry_context_out.

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

Non-goals (DFM-first)

Không “làm đẹp” hình học vượt mức: không tự tạo/di chuyển features, không “làm dày/mỏng” thành, không thay đổi draft theo hướng rút.

Không âm thầm tăng tolerance quá mức để “đậu validity” (phải có giới hạn policy và phải báo cáo minh bạch). [CHƯA XÁC MINH: hiện code có tracking tolerance_delta hay chưa]

INPUT FORMALIZATION
Required (facts)

payload dict có agl_ref, dgl_ref, và geometry.brep_shape (shape_in).

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

Optional (DFM-first, đề xuất)

run_id, rule_version, geometry_version_id (để traceability) [CHƯA XÁC MINH: healer hiện echo các field này chưa]

policy knobs cho healing:

max_heal_iterations (ở level orchestrator loop)

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

max_tolerance_delta_mm, sewing_tolerance_mm, sliver_face_threshold_mm… [CHƯA XÁC MINH: keys cụ thể; nên đưa vào policy-as-config]

Fail-safe dependency (facts)

OCC missing → status NOT_EXECUTED, NOOP/passthrough, không crash.

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

OUTPUT FORMALIZATION
Output envelope (facts): HEAL_RESULT_v0, gồm tối thiểu:

status: INITIALIZED / EXECUTED / NOT_EXECUTED / ERROR

is_valid_before, is_valid_after

fix_applied (bool)

geometry_context_out.shape_healed khi fix_applied=true, hoặc passthrough shape_in khi NOOP/fallback

refs: agl_ref, dgl_ref, healed_ref (default=agl_ref)

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

DFM-first additions (đề xuất để tăng “tính dùng được”, gắn [CHƯA XÁC MINH] nếu chưa có)

operations_applied: list[str] (ví dụ SHAPEFIX_PERFORMED, SEWING_APPLIED…)

tolerance_delta_mm: float (ước lượng mức thay đổi tolerance) [CHƯA XÁC MINH]

risk_flags:

healed_geometry_may_shift (bool) [CHƯA XÁC MINH]

requires_owner_review (bool) [CHƯA XÁC MINH]
Mục tiêu là để report nói rõ: “geometry đã bị can thiệp mức nào”.

DETERMINISM REQUIREMENTS
Facts: deterministic tương đối theo cùng input shape và OCC version; phụ thuộc OCC version.

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

DFM-first requirement:

Nếu healer can thiệp theo tolerance, mọi tolerance phải đến từ policy (rule_versioned) để kết quả có thể tái lập/audit.

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

TÀI LIỆU ĐẶC TẢ HỆ THỐNG QUẢN T…

STEP-BY-STEP LOGIC (facts + DFM-first framing)
Step 1: Initialize HEAL_RESULT_v0, status INITIALIZED, attach refs.

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

Step 2: Validate inputs: thiếu agl_ref/dgl_ref hoặc thiếu brep_shape → ERROR (envelope; không raise).

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

Step 3: OCC availability check (lazy-load): thiếu OCC → NOT_EXECUTED, fix_applied=false, passthrough.

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

Step 4: is_valid_before = BRepCheck_Analyzer(shape_in).

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

Step 5: Nếu invalid → ShapeFix_Shape.Perform(); shape_healed = ShapeFix_Shape.Shape(); fix_applied=true. Nếu valid → fix_applied=false; shape_healed=shape_in (hoặc equivalent).

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

Step 6: is_valid_after = BRepCheck_Analyzer(shape_healed).

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

Step 7: Package output status EXECUTED; geometry_context_out.

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

DFM-first “observability hooks” nên có (đề xuất)

Nếu is_valid_after=false: status vẫn EXECUTED nhưng phải phát issue HEALING_INSUFFICIENT (để validation gate/orchestrator quyết định loop/fallback). [CHƯA XÁC MINH: hiện healer đã phát issue chưa]

Nếu fix_applied=true: phải ghi “operations_applied” và (nếu có) tolerance_delta để kỹ sư biết có khả năng ảnh hưởng DFM metrics.

FAILURE MODES
Facts

Missing refs/shape → ERROR

OCC missing → NOT_EXECUTED

ShapeFix fails → ERROR hoặc EXECUTED với is_valid_after=false [CHƯA XÁC MINH: nhánh cụ thể]

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

DFM-first additional failure semantics (đề xuất)

HEALING_CAUSES_GEOMETRY_DRIFT (nếu tolerance_delta vượt policy) → FAILED hoặc EXECUTED + requires_owner_review. [CHƯA XÁC MINH: cần policy key + đo drift]

COMPLEXITY (facts-level)
Time: O(C_brepcheck + C_shapefix), phụ thuộc topo shape.
Memory: O(size_of_shape) khi tạo healed copy.

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

## 3.2 AMBR (Bindings / Module Contract / Execution)

Source: S1

C) AMBR — healer_agent (binding)

ambr_id: AMBR_healer_agent_v1 (baseline publish)

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

status: APPROVED (baseline)
module_name: healer_agent
module_type: analysis_agent (healing stage)
execution_mode: SEQUENTIAL
pipeline_role: geometry_preparation_healing_stage

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

lazy_load_deps: true
bound_algorithm: ADN_healer_agent_minimal_fix_brep_validity_v0

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

DFM-first binding expectations (để các agent downstream tin được)

Orchestrator phải ghi nhận rõ: shape_healed có được “accept” vào AGL lifecycle không, và khi accept thì geometry_version_id có thay đổi hay không. [CHƯA XÁC MINH: versioning wiring hiện có]

Healer output phải được nối vào validation gate trước freeze theo canonical pipeline.

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

Source: S1 (schema sketch extract)

E) HEAL_RESULT_v0 schema sketch (DFM-first extension)

HEAL_RESULT_v0 (baseline fields theo facts):

result_type: "HEAL_RESULT_v0" [CHƯA XÁC MINH: field name; nhưng envelope id đã có]

run_id: [CHƯA XÁC MINH]

status: INITIALIZED|EXECUTED|NOT_EXECUTED|ERROR

agl_ref, dgl_ref, healed_ref

is_valid_before: bool

is_valid_after: bool

fix_applied: bool

geometry_context_out:

shape_healed: (opaque handle)

issues: list[issue_obj] [CHƯA XÁC MINH: hiện có issues list hay chưa]

receipts_ref: [CHƯA XÁC MINH]

DFM-first optional additions:

operations_applied: list[str] [CHƯA XÁC MINH]

tolerance_delta_mm: float|null [CHƯA XÁC MINH]

warnings:

"HEAL_APPLIED_REVIEW_RECOMMENDED" (nếu fix_applied=true và downstream metrics nhạy) [CHƯA XÁC MINH]

Source: S2 (observability addendum extract)

HEALER_AGENT_OBSERVABILITY_ADDENDUM_v1
doc_id: TOOLING_DFM_ADVISOR_HEALER_AGENT_OBSERVABILITY_ADDENDUM_v1
status: DRAFT
effective_date_utc: [CHƯA XÁC MINH]
applies_to: healer_agent (HEAL_RESULT_v0)

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

principles: evidence-first, deterministic, no shadow data, fail-safe geometry deps.

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

Why add observability?
Healer “chữa” hình học có thể ảnh hưởng trực tiếp đến kết quả DFM downstream:

Draft/Undercut: thay đổi orientation, sewing, sliver removal có thể đổi normal cục bộ.

Thickness/Sink/Rib: sửa closure hoặc tự động “làm kín” có thể làm ray-probe hit khác.
Nếu healer chỉ trả “fix_applied” mà không nói “chữa kiểu gì / chữa mạnh tới đâu”, báo cáo DFM sẽ khó tin và khó debug.

What must be observable (Minimum required fields)
Các field này nên được thêm/chuẩn hóa trong HEAL_RESULT_v0 (không thay đổi semantics hiện có; chỉ bổ sung metadata).

2.1. Input/trace echo (bắt buộc)

run_id: YYYYMMDDThhmmssZ [CHƯA XÁC MINH: hiện healer có run_id chưa]

rule_version: RULESET_v1 [CHƯA XÁC MINH]

geometry_version_id_in: string|null (nếu healer chạy pre-freeze thì có thể null)

geometry_version_id_out: string|null (nếu orchestrator quyết định bump version sau khi accept) [CHƯA XÁC MINH: ai cấp version]

agl_ref, dgl_ref, healed_ref (đã có theo facts)

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

2.2. Validity deltas (bắt buộc, phần lớn đã có)

is_valid_before: bool

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

is_valid_after: bool

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

fix_applied: bool

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

status: INITIALIZED|EXECUTED|NOT_EXECUTED|ERROR

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

2.3. Operations summary (bắt buộc)

operations_applied: list[string]
Ví dụ closed-set khuyến nghị (để audit ổn định):

"NOOP"

"SHAPEFIX_PERFORM"

"SEWING_APPLIED"

"TOLERANCE_ADJUSTED"

"SLIVER_FACE_REMOVED"

"SMALL_EDGE_MERGED"

"UNKNOWN_OPERATION"
Nếu code không phân rã được chi tiết, tối thiểu phải có:

["SHAPEFIX_PERFORM"] khi fix_applied=true

["NOOP"] khi fix_applied=false

2.4. Drift / change magnitude proxy (bắt buộc tối thiểu ở mức proxy)
Healer không được “âm thầm” thay đổi hình học. Vì chưa muốn chạy so sánh nặng, tối thiểu phải có proxy rẻ:

topo_invariants_before: {face_count:int, edge_count:int, vertex_count:int}

topo_invariants_after: {face_count:int, edge_count:int, vertex_count:int}

topo_changed: bool (derived: any count differs)

Ghi chú: đây là proxy thô. Count đổi không phải lúc nào cũng xấu (sliver removal hợp lý), nhưng nó là tín hiệu quan trọng để report “đã can thiệp topology”.
[CHƯA XÁC MINH] healer hiện có đếm topo invariants chưa; nếu chưa, addendum yêu cầu bổ sung.

2.5. Tolerance change (khuyến nghị mạnh; nếu chưa có thì phải explicit null)

tolerance_delta_mm: float|null

tolerance_policy_used:

sewing_tolerance_mm: float|null

max_tolerance_delta_mm: float|null
Nếu chưa đo được tolerance_delta, phải set null và phát issue:

"TOLERANCE_DELTA_UNAVAILABLE"

Issues taxonomy (bắt buộc)
Healer phải trả issues có cấu trúc (list) để gate/orchestrator không suy diễn:

"OCC_MISSING" (→ NOT_EXECUTED)

"INPUT_MISSING" (agl_ref/dgl_ref/shape)

"HEALING_ATTEMPTED"

"HEALING_NO_EFFECT" (fix_applied=true nhưng topo/validity không đổi)

"HEALING_INSUFFICIENT" (is_valid_after=false)

"TOPOLOGY_CHANGED" (topo_changed=true)

"TOLERANCE_POLICY_UNRESOLVED" (nếu policy keys thiếu)

"TOLERANCE_DELTA_EXCEEDED" (nếu có đo và vượt max)

"HEALING_EXCEPTION_CAUGHT" (nếu ShapeFix ném lỗi nhưng được catch và đóng envelope)

Output contract sketch (HEAL_RESULT_v0 extended)
Đây là hình dạng “đề xuất” (không đòi đổi envelope id):

HEAL_RESULT_v0:

result_type: "HEAL_RESULT_v0"

run_id: string|null

status: "INITIALIZED"|"EXECUTED"|"NOT_EXECUTED"|"ERROR"

error_code: null|string

rule_version: string|null

geometry_version_id_in: string|null

geometry_version_id_out: string|null

agl_ref, dgl_ref, healed_ref

is_valid_before: bool|null

is_valid_after: bool|null

fix_applied: bool

operations_applied: list[string]

topo_invariants_before: {face_count:int, edge_count:int, vertex_count:int}|null

topo_invariants_after: {face_count:int, edge_count:int, vertex_count:int}|null

topo_changed: bool|null

tolerance_delta_mm: float|null

tolerance_policy_used: {sewing_tolerance_mm: float|null, max_tolerance_delta_mm: float|null}|null

issues: list[issue_obj]

geometry_context_out: {shape_healed: opaque_handle}|null

receipts_ref: [CHƯA XÁC MINH]

Mapping guidance (để validators/orchestrator dùng sau này, chưa phải heal-loop contract)

If status == NOT_EXECUTED and issue OCC_MISSING → validators.gate_decision = DEFERRED; retryable=true.

If is_valid_after == true and topo_changed == false → healing_candidate=LIKELY_PASS (good).

If is_valid_after == true and topo_changed == true → PASS nhưng “requires_review” (vì topology đã đổi; DFM metrics có thể đổi).

If is_valid_after == false and fix_applied == true → healing_candidate=POSSIBLE/UNLIKELY (tùy fail_reason_codes từ validators).

If fix_applied == true and HEALING_NO_EFFECT → đừng loop vô hạn; cần stop condition (sẽ chốt trong heal-loop contract).

Starter defaults (policy knobs tối thiểu cho healer observability)
Nếu muốn chốt sớm trong policy:

dfm_policy.thresholds.healing:

mode: "MINIMAL_FIX_ONLY"

max_tolerance_delta_mm: 0.10

sewing_tolerance_mm: 0.05
Nếu chưa chốt: healer phải phát issue TOLERANCE_POLICY_UNRESOLVED và để tolerance_policy_used=null (minh bạch, không tự đoán).

No shadow data compliance

operations_applied, topo counts, tolerance_delta đều là metadata; không chứa file path.

tuyệt đối không lưu debug dump của shape ra đĩa trong healer output; nếu có artifact debug thì phải đi qua ephemeral asset store và scrub sau delivery_complete (theo Master Spec).

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

## 3.3 LDR (Dependencies / Libraries / Runtime constraints)

Source: S1

D) LDR — dependencies

ldr_id: LDR_occ_core_brepcheck_shapefix_for_healer_agent_v1 (baseline publish)

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

status: APPROVED (baseline)
library_name: pythonocc-core (OCC.Core.BRepCheck, OCC.Core.ShapeFix)
purpose: validity check + minimal fix
fail-safe: OCC missing → NOT_EXECUTED (no crash)

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

## 4. Open Questions / [CHƯA XÁC MINH]

"HEAL_APPLIED_REVIEW_RECOMMENDED" (nếu fix_applied=true và downstream metrics nhạy) [CHƯA XÁC MINH]

DFM-first additions (đề xuất để tăng “tính dùng được”, gắn [CHƯA XÁC MINH] nếu chưa có)

DFM-first extension note: Các mục dưới đây mô tả “cách dùng đúng” và “bổ sung observability/policy knobs” để healer phục vụ DFM tốt hơn; phần nào chưa có bằng chứng trong code sẽ gắn [CHƯA XÁC MINH].

HEALING_CAUSES_GEOMETRY_DRIFT (nếu tolerance_delta vượt policy) → FAILED hoặc EXECUTED + requires_owner_review. [CHƯA XÁC MINH: cần policy key + đo drift]

Không âm thầm tăng tolerance quá mức để “đậu validity” (phải có giới hạn policy và phải báo cáo minh bạch). [CHƯA XÁC MINH: hiện code có tracking tolerance_delta hay chưa]

Nếu is_valid_after=false: status vẫn EXECUTED nhưng phải phát issue HEALING_INSUFFICIENT (để validation gate/orchestrator quyết định loop/fallback). [CHƯA XÁC MINH: hiện healer đã phát issue chưa]

Orchestrator phải ghi nhận rõ: shape_healed có được “accept” vào AGL lifecycle không, và khi accept thì geometry_version_id có thay đổi hay không. [CHƯA XÁC MINH: versioning wiring hiện có]

ShapeFix fails → ERROR hoặc EXECUTED với is_valid_after=false [CHƯA XÁC MINH: nhánh cụ thể]

[CHƯA XÁC MINH] healer hiện có đếm topo invariants chưa; nếu chưa, addendum yêu cầu bổ sung.

effective_date_utc: [CHƯA XÁC MINH]

geometry_version_id_out: string|null (nếu orchestrator quyết định bump version sau khi accept) [CHƯA XÁC MINH: ai cấp version]

healed_geometry_may_shift (bool) [CHƯA XÁC MINH]

healing_mode: "MINIMAL_FIX_ONLY" (để cấm những sửa chữa mạnh) [CHƯA XÁC MINH: key này chưa tồn tại; chỉ là đề xuất]

issues: list[issue_obj] [CHƯA XÁC MINH: hiện có issues list hay chưa]

max_tolerance_delta_mm, sewing_tolerance_mm, sliver_face_threshold_mm… [CHƯA XÁC MINH: keys cụ thể; nên đưa vào policy-as-config]

max_tolerance_delta_mm: 0.05 hoặc 0.10 [CHƯA XÁC MINH: phụ thuộc độ chính xác CAD và dung sai dự án]

operations_applied: list[str] [CHƯA XÁC MINH]

owner: [CHƯA XÁC MINH]

receipts_ref: [CHƯA XÁC MINH]

requires_owner_review (bool) [CHƯA XÁC MINH]

result_type: "HEAL_RESULT_v0" [CHƯA XÁC MINH: field name; nhưng envelope id đã có]

rule_version: RULESET_v1 [CHƯA XÁC MINH]

run_id, rule_version, geometry_version_id (để traceability) [CHƯA XÁC MINH: healer hiện echo các field này chưa]

run_id: YYYYMMDDThhmmssZ [CHƯA XÁC MINH: hiện healer có run_id chưa]

run_id: [CHƯA XÁC MINH]

tolerance_delta_mm: float (ước lượng mức thay đổi tolerance) [CHƯA XÁC MINH]

tolerance_delta_mm: float|null [CHƯA XÁC MINH]

[CHƯA XÁC MINH] Confirm whether healer_agent is classified under WS_PROVIDERS or WS_AGENTS/WS_CORE per Document Governance Layer mapping; target code in this pack is WS_PROVIDERS-HEA-SPC-v1 as requested. Impact: wrong WS assignment would misplace SSOT in registry and confuse rename/gap tracking.

## 5. Change Log (Editorial)

Run 20260304T082854Z: Consolidated S1+S2 into canonical SPC WS_PROVIDERS-HEA-SPC-v1. Editorial-only changes: added Document Control Block, Included Sources & Mapping Table, and explicit traceability markers ('Source: Sx') per subsection. No semantics or technical logic changes were introduced; all consolidated content is presented as verbatim extracts except for neutral connectors where needed.

## 6. Appendix

aka / naming notes (editorial): module implementation uses snake_case (healer_agent); contracts/envelopes use UPPER_CASE versioned identifiers when present in sources (e.g., HEAL_RESULT_v0). If sources use alternate spellings, they are retained in extracts and can be mapped here in future revisions.
