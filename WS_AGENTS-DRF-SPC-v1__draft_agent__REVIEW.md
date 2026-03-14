**0. Document Control Block**

Canonical Code: WS_AGENTS-DRF-SPC-v1

Status: REVIEW

Run ID: 20260304T095029Z

WS / Module / Doc Type / Version: WS_AGENTS / DRF / SPC / v1

Created date (UTC): 20260304T095029Z

Purpose: Document governance consolidation only; no runtime/pipeline changes.

**1. Scope**

This document consolidates the draft_agent specification (ADN/AMBR/LDR) under WS_AGENTS-DRF-SPC-v1. Content is inherited from sources and reorganized for governance control. No technical semantics are changed.

Non-goals: No algorithm redesign, no new thresholds/policy invention, no pipeline/runtime changes, no simulation, no code generation.

**2. Included Sources & Mapping Table**

|               |                                                                                                   |                                                                                              |                             |                                                                |                                                                                                                         |
| ------------- | ------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- | --------------------------- | -------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **source_id** | **filename**                                                                                      | **original_title**                                                                           | **original_version/status** | **sections_used**                                              | **notes**                                                                                                               |
| S1            | TÀI LIỆU TRIỂN KHAI ADN AMBR LDR — MODULE draft_agent.docx                                        | TOOLING_DFM_ADVISOR_DRAFT_AGENT_ADN_AMBR_LDR_DRAFT_v1                                        | DRAFT                       | ADN/AMBR/LDR + Open Questions                                  | Primary module spec source (verbatim extracts)                                                                          |
| S2            | DRAFT_ANALYSIS_RESULT_v0 — schema sketch (đề xuất tối thiểu, phù hợp RESULT_ENVELOPE_API_v1).docx | DRAFT_ANALYSIS_RESULT_v0 — schema sketch (đề xuất tối thiểu, phù hợp RESULT_ENVELOPE_API_v1) | [CHƯA XÁC MINH]             | Referenced only (Result schema cross-ref); excerpt in Appendix | Schema sketch is referenced to avoid duplication; canonical home is WS_MODEL-GEN-SPC-v1 [CHƯA XÁC MINH: per audit note] |

**3. Consolidated Specification Content**

**3.1 ADN (Algorithm / Logic)**

Source: S1

────────────────────────────────────────────────────────────
B) ADN (ALGORITHM DESIGN NOTES) — INSTANCE

ADN — THÔNG TIN ĐỊNH DANH (IDENTIFICATION)

adn_id: ADN_draft_agent_draft_angle_sampling_with_silhouette_detection_v0
module_name: draft_agent
module_type: analysis_agent
algorithm_name: draft_angle_sampling_with_silhouette_detection
algorithm_version: v0
status: DRAFT
owner: [CHƯA XÁC MINH]
last_reviewed_utc: [CHƯA XÁC MINH]
related_ticket_ids: [CHƯA XÁC MINH]

PROBLEM STATEMENT (Bài toán kỹ thuật)

Core problem: Phân tích góc thoát khuôn (draft) theo đúng hướng rút khuôn đã được user_confirmed, áp dụng được cho cả mặt phẳng và mặt cong (analytic + freeform), đồng thời phát hiện vùng “reverse draft” và vùng “silhouette crossing” (dot≈0 đổi dấu) để hỗ trợ quyết định parting/split và cảnh báo nguy cơ drag marks / kẹt khuôn.

Scope (V1_BASELINE):

Tính draft angle tại các điểm mẫu trên bề mặt (bao gồm mặt cong).

Tổng hợp per-face và global: min/percentiles, vùng tệ nhất (worst regions).

Phát hiện:
(a) reverse draft (đối nghịch hướng rút),
(b) silhouette crossing trên cùng một mặt (dot đổi dấu), thường hàm ý parting chạy qua patch hoặc cần tách mặt.

Xuất defects có tính truy vết (run_id, geometry_version_id, ejection_direction, threshold).

Non-goals:

Không tự động đổi hướng rút, không tự chỉnh CAD, không tối ưu hình học.

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

Không thay thế CAE/CAE-like draft map cực mịn; V1 ưu tiên “đúng trọng điểm + kiểm chứng được”.

Không tự kết luận giải pháp khuôn (slide/lifter) như quyết định cuối; chỉ advisory.

Không phân loại “mặt chức năng” (shut-off/datum/no-draft-needed) nếu chưa có tagging/AFR đủ mạnh (đây là tiêu chí nâng cấp V2). [CHƯA XÁC MINH]

INPUT FORMALIZATION (Chuẩn hóa đầu vào)

Input: payload (dict) theo pipeline conventions.
Required inputs (hard preconditions):

geometry_context: có B-Rep shape (TopoDS_Shape) ở AGL (read-only). [CHƯA XÁC MINH: đường dẫn field trong payload]

operational_vector: ejection_direction (3D vector) và user_confirmed=true. Nếu thiếu user_confirmed thì draft_agent phải fail-safe với lỗi có cấu trúc (không “đoán”).

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

run_id: YYYYMMDDThhmmssZ (để trace receipts/defects). [CHƯA XÁC MINH: run_id được inject ở orchestrator hay context provider]

Policy inputs (policy-as-config):

min_draft_deg_default (float, degrees)

(optional) min_draft_deg_by_surface_class: {polish/light_texture/heavy_texture: deg} [CHƯA XÁC MINH]

(optional) surface_class tagging source (per-face / per-part) [CHƯA XÁC MINH]
Nếu policy threshold thiếu: draft_agent vẫn được phép tính “draft facts” (min/percentiles, reverse/silhouette), nhưng không được kết luận pass/fail theo threshold; phải phát issue POLICY_UNRESOLVED và giảm confidence.

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

Degraded mode requirement:

Nếu OCC/pythonocc-core không khả dụng: status = NOT_EXECUTED hoặc STUB_MODE (không crash pipeline).

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

OUTPUT FORMALIZATION (Chuẩn hóa đầu ra)

Output envelope: DRAFT_ANALYSIS_RESULT_v0 (dict) tuân thủ RESULT_ENVELOPE_API_v1 (tối thiểu).

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

Fields tối thiểu (theo Master Spec):

result_type: DRAFT_ANALYSIS_RESULT_v0

run_id

status: OK | FAILED | NOT_EXECUTED | STUB_MODE

error_code / issues (structured)

rule_version [CHƯA XÁC MINH: lấy từ config/governance_policy.json]

geometry_version_id [CHƯA XÁC MINH: cấp ở bước freeze hay earlier]

metrics: draft metrics

produced_defects: [defect_id…]

receipts_ref: [CHƯA XÁC MINH: receipt id/path]

Draft metrics đề xuất (V1_BASELINE):
Global:

draft_min_deg_global

draft_p10_deg, draft_p50_deg, draft_p90_deg (từ phân phối điểm mẫu)

reverse_sample_ratio (tỷ lệ điểm có dot\<0 theo hướng rút)

silhouette_face_count (số mặt có dot đổi dấu trong mẫu)

worst_regions: list (top N), mỗi item gồm {face_id, uv_hint, draft_min_deg_face, flags}

Per-face summary (tùy chọn theo kích thước output):

face_count_evaluated

face_count_skipped (non-evaluable / missing params)

face_findings: list (giới hạn N, ưu tiên mặt “tệ nhất”)

Guarantees:

Read-only SSOT hình học; không mutate AGL.

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

Deterministic: cùng shape + cùng ejection_direction + cùng sampling_policy_version → kết quả tương đương (trong sai số do sampling grid, nếu dùng). Seed sampling (nếu random) bắt buộc suy ra từ run_id để tất định.

TÀI LIỆU ĐẶC TẢ GHI CHÚ THIẾT K…

DETERMINISM REQUIREMENTS (Tất định)

Deterministic Level (intended): Level 7 (deterministic w.r.t. same inputs + pinned geometry backend). [CHƯA XÁC MINH: OCC version pinning trong LDR/policy]
Nếu sampling là grid cố định theo (u,v) và số mẫu cố định theo surface type → tất định tuyệt đối.
Nếu sampling adaptive theo curvature: vẫn tất định nếu quy tắc adaptive là closed-form (không random, không phụ thuộc thời gian máy).

STEP-BY-STEP LOGIC (Logic từng bước)

Step 1 — Initialize envelope
Inputs: run_id, agl_ref/dgl_ref (nếu có), geometry_version_id (nếu có)
Output: DRAFT_ANALYSIS_RESULT_v0, status=OK (tentative), issues=[]

Step 2 — Validate ejection_direction and confirmation
Criteria:

missing ejection_direction → FAILED (EJECTION_DIR_MISSING)

norm(ejection_direction)=0 → FAILED (EJECTION_DIR_INVALID)

user_confirmed != true → FAILED (EJECTION_NOT_CONFIRMED)
Evidence: echo operational_vector vào metrics/policy_echo (không lưu path bền vững).

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

Step 3 — Validate geometry context
Criteria:

missing brep_shape → FAILED (GEOM_CONTEXT_MISSING)

OCC missing → STUB_MODE/NOT_EXECUTED (OCC_MISSING), không crash pipeline.

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

Step 4 — Traverse faces (B-Rep topology)
Traverse: TopExp_Explorer(shape, TopAbs_FACE)
For each face:

Resolve surface type: plane / cylinder / cone / sphere / torus / bspline / other

Establish sampling plan:

analytic surfaces: fixed grid (ví dụ 3×3) trên miền tham số

freeform: grid coarser + optionally refine theo curvature proxy [CHƯA XÁC MINH: curvature access method]
Outputs per face: sample set S = { (u,v) }

Step 5 — Compute normals and signed dot
For each sample:

n = unit surface normal (adjusted by TopAbs orientation)

p = unit ejection_direction

dot = n·p

draft_deg = arcsin(|dot|) in degrees (draft=0° tương ứng tường song song hướng rút; draft tăng khi bề mặt nghiêng “thoát” hơn)
Flags:

reverse_sample if dot < 0

silhouette_candidate if |dot| \<= eps (eps từ policy) [CHƯA XÁC MINH]

Step 6 — Aggregate per-face metrics and detect silhouette crossing
Per face:

draft_min_deg_face = min(draft_deg)

reverse_present_face = any(dot\<0)

silhouette_crossing_face = (exists dot>0 and exists dot\<0) hoặc (sign change across samples)
Store worst faces by draft_min_deg_face, ưu tiên reverse_present.

Step 7 — Compare against policy (nếu có threshold) và emit defects
If policy.min_draft exists:

identify samples/faces where draft_min_deg_face < threshold

emit defect(s) với fields tối thiểu: id, run_id, agent_source=draft_agent, rule_version, geometry_version_id, operational_vector=ejection_direction, analysis_value=draft_min_deg_face, threshold_applied=threshold, advisory summary.

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

Else:

issue POLICY_UNRESOLVED; không emit defect “below threshold”, chỉ emit factual warnings (reverse/silhouette) nếu cần.

Step 8 — Finalize envelope

status: OK nếu compute thành công; FAILED nếu input thiếu; STUB_MODE/NOT_EXECUTED nếu OCC thiếu

metrics + worst_regions + produced_defects + issues

attach receipts_ref [CHƯA XÁC MINH: mechanism]

FAILURE MODES (Fail-safe + structured)

EJECTION_DIR_MISSING / EJECTION_DIR_INVALID / EJECTION_NOT_CONFIRMED → FAILED (hard stop cho agent; pipeline vẫn tiếp tục đến report).

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

GEOM_CONTEXT_MISSING → FAILED

OCC_MISSING → STUB_MODE hoặc NOT_EXECUTED (không crash)

POLICY_UNRESOLVED → OK nhưng issues chứa POLICY_UNRESOLVED, confidence giảm

NORMAL_EVAL_FAILED_ON_FACE → OK nhưng face bị skip; ghi warning (không crash toàn agent)

COMPLEXITY ANALYSIS (Độ phức tạp)

Time: O(F * S) với F = số faces, S = số samples/face (policy-controlled).
Memory: O(1) hoặc O(N_worst) nếu chỉ lưu top N worst_regions; tránh lưu toàn bộ per-sample để không phình report.

V2 UPGRADE INPUTS (không thực thi trong V1; định nghĩa “điều kiện nhảy V2”)

Bắt buộc để V2 không “đoán”:

Policy catalog đầy đủ: min_draft theo surface_class × material/process class (được version hóa). [CHƯA XÁC MINH]

Tagging/recognition cho “functional faces” (no-draft-needed) từ: CAD attributes / naming / AFR feature graph hoặc user marking. [CHƯA XÁC MINH]

Golden geometry corpus + expected draft findings (đủ để regression). [CHƯA XÁC MINH]
Impact nếu thiếu: V2 sẽ tăng false positive/negative và làm báo cáo mất uy tín trong review khuôn.

**3.2 AMBR (Bindings / Module Contract / Execution)**

Source: S1

Source: S2 (reference note) — Result schema DRAFT_ANALYSIS_RESULT_v0 should be maintained under WS_MODEL-GEN-SPC-v1 to avoid duplication; this SPC references it and keeps only a small excerpt in Appendix.

────────────────────────────────────────────────────────────
C) AMBR (ARCHITECTURE / MODULE BINDING RECORD) — INSTANCE

AMBR — MODULE IDENTIFICATION

ambr_id: AMBR_draft_agent_v1
status: DRAFT
binding_version: v1
file_name: draft_agent.py
module_name: draft_agent
module_type: analysis_agent
execution_mode: SEQUENTIAL (internal), orchestration có thể PARALLEL giữa các agents.

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

pipeline_stage: Step 5 (Parallel Analysis Agents) theo Master Spec.

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

BOUND ALGORITHM

adn_id: ADN_draft_agent_draft_angle_sampling_with_silhouette_detection_v0
algorithm_name: draft_angle_sampling_with_silhouette_detection
algorithm_version: v0
output_envelope: DRAFT_ANALYSIS_RESULT_v0 [CHƯA XÁC MINH: envelope id đã được đăng ký chuẩn trong contracts chưa]

BOUND LIBRARIES (Phụ thuộc)

python_stdlib:

math (asin, degrees) [CHƯA XÁC MINH: implementation detail]

typing (type hints)

external_geometry (pythonocc-core) — required for V1:

OCC.Core.TopExp (TopExp_Explorer)

OCC.Core.TopAbs (TopAbs_FACE, orientation)

OCC.Core.TopoDS (TopoDS_Face/Shape)

OCC.Core.BRepAdaptor (BRepAdaptor_Surface)

OCC.Core.BRepLProp (BRepLProp_SLProps) hoặc tương đương để lấy normal

OCC.Core.GeomAbs (surface type enums)

OCC.Core.gp (gp_Dir/gp_Vec)

Fail-safe rule:

Nếu OCC không available: agent phải return STUB_MODE/NOT_EXECUTED (không raise uncaught exception).

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

EXECUTION CONTEXT

Role: DFM analysis agent (draft)
SSOT permissions: read-only trên AGL; không mutate.

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

Determinism: deterministic nếu sampling policy fixed + OCC pinned. [CHƯA XÁC MINH: OCC pinning]

RESPONSIBILITY SPLIT

draft_agent owns:

tính draft trên mặt cong + reverse/silhouette detection

tạo defect advisory cho draft (khi threshold có)

pipeline/orchestrator owns:

cấp run_id, rule_version, geometry_version_id

đảm bảo ejection_direction đã user_confirmed

persist receipts/logs và luôn chạy reporter ngay cả khi agent fail.

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

TÀI LIỆU ĐẶC TẢ HỆ THỐNG QUẢN T…

DATA FLOW

Input path:

GeometryStack AGL (brep_shape)

operational_vector ejection_direction

policy thresholds

Output path:

DRAFT_ANALYSIS_RESULT_v0 envelope

produced defects appended vào analysis_results_stack (append-only) [CHƯA XÁC MINH: wiring cụ thể]

EDGE CASE OWNERSHIP

Thiếu policy threshold: draft_agent báo POLICY_UNRESOLVED và chỉ xuất “facts”

Mặt không đánh giá được normal: skip mặt và warning (không sập agent)

ejection_direction không confirmed: FAILED (hard), vì kết quả mất ý nghĩa kỹ thuật

CHANGE IMPACT RULE

Thay sampling_policy hoặc định nghĩa draft angle → có thể đổi kết quả; phải bump algorithm_version (v0→v1) và cập nhật expected defects trong golden corpus.

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

**3.3 LDR (Dependencies / Libraries / Runtime constraints)**

Source: S1

────────────────────────────────────────────────────────────
D) LDR (LIBRARY DEPENDENCY RECORD) — INSTANCE

LDR — IDENTIFICATION

ldr_id: LDR_pythonocc_core_surface_normal_and_topology_for_draft_agent_v1
status: DRAFT
library_name: pythonocc-core (OCC.Core)
library_version: [CHƯA XÁC MINH: version pinning]
used_by_modules: draft_agent
purpose: duyệt topology faces + truy xuất loại bề mặt + tính normal bề mặt tại điểm tham số để suy ra draft & reverse/silhouette detection

JUSTIFICATION

Draft trên mặt cong cần normal theo tham số; không thể làm đúng chỉ bằng bbox hay planar-only.

OCC.Core cung cấp primitive ổn định (TopExp, BRepAdaptor, BRepLProp, GeomAbs, gp) để thực hiện nhanh, tất định, không cần CAE.

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

RISK NOTES (engineering)

Phụ thuộc C++ runtime nặng; phải lazy-load + fail-safe envelope để tránh crash pipeline khi thiếu OCC.

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

Kết quả phụ thuộc version OCC (đặc biệt ở normal evaluation trên surface phức tạp); nên pin version và đưa vào receipts/build info. [CHƯA XÁC MINH]

ALTERNATIVES CONSIDERED

Mesh-based normal (từ DGL): giảm phụ thuộc B-Rep, nhưng tăng sai số do tessellation và đòi spatial mapping tốt; phù hợp hơn cho V2/V3 khi mapping_provider + integrity check đã đủ mạnh. [CHƯA XÁC MINH]

**4. Open Questions / [CHƯA XÁC MINH]**

Source: S1

────────────────────────────────────────────────────────────
E) DANH SÁCH THIẾU DỮ LIỆU CẦN CHỐT (để chuyển DRAFT → APPROVED)

[CHƯA XÁC MINH] 1) Envelope ID canonical cho draft_agent (DRAFT_ANALYSIS_RESULT_v0) đã được đăng ký đầy đủ trong contracts chưa?
Impact: nếu không chốt, downstream reporter và UI contract khó ổn định.

[CHƯA XÁC MINH] 2) Key path chuẩn của payload: brep_shape nằm ở đâu (geometry.brep_shape hay geometry_context.shape…)?
Impact: agent dễ drift chữ ký, khó wiring.

[CHƯA XÁC MINH] 3) Policy keys tối thiểu: min_draft_deg_default, eps_silhouette, sampling_density.
Impact: nếu không có, agent chỉ ra “facts” nhưng không thể kết luận theo chuẩn DFM.

[CHƯA XÁC MINH] 4) Định danh face_id/uv_hint để “report vùng tệ nhất” mà vẫn không tạo shadow data.
Impact: không định vị được vùng lỗi → báo cáo kém giá trị với kỹ sư khuôn.

[CHƯA XÁC MINH] Whether DRAFT_ANALYSIS_RESULT_v0 schema should be annexed inside WS_AGENTS-DRF-SPC-v1 or referenced only via WS_MODEL-GEN-SPC-v1. Impact: duplication risk vs local completeness for reviewers.

**5. Change Log (Editorial)**

20260304T095029Z: Consolidated draft_agent docs into canonical WS_AGENTS-DRF-SPC-v1. Normalized file naming and added traceability mapping tables. No semantics change.

20260304T095029Z: Referenced DRAFT_ANALYSIS_RESULT_v0 schema sketch (S2) without fully duplicating it; cross-referenced WS_MODEL-GEN-SPC-v1. No semantics change.

**6. Appendix**

**Appendix A — Excerpt: DRAFT_ANALYSIS_RESULT_v0 schema sketch**

Source: S2

DRAFT_ANALYSIS_RESULT_v0 — schema sketch (đề xuất tối thiểu, phù hợp RESULT_ENVELOPE_API_v1)

Lưu ý: Master Spec đã quy định mọi \*\_RESULT_v0 phải theo RESULT_ENVELOPE_API_v1 với các trường tối thiểu (result_type, run_id, status, error_code, disable_reason, rule_version, geometry_version_id, metrics, produced_defects, receipts_ref…). Ở đây mình chỉ “phác thảo” schema draft_agent cần, và đánh dấu [CHƯA XÁC MINH] những chỗ phụ thuộc contract cụ thể/đường wiring.

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

DRAFT_ANALYSIS_RESULT_v0 (dict):

result_type: "DRAFT_ANALYSIS_RESULT_v0"

run_id: "YYYYMMDDThhmmssZ"

status: "OK" | "FAILED" | "NOT_EXECUTED" | "STUB_MODE"

error_code: null | \<STRUCTURED_ERROR_CODE>

disable_reason: null | <string> (bắt buộc nếu NOT_EXECUTED/STUB_MODE)

rule_version: "RULESET_v1" [CHƯA XÁC MINH: tên rule_version thực tế trong config/governance_policy.json]

geometry_version_id: <string> [CHƯA XÁC MINH: cấp ở bước freeze hay trước freeze]

agent_source: "draft_agent" (khuyến nghị echo; Master Spec đặt agent_source ở defect object, nhưng result envelope echo giúp debug)

inputs_echo:

operational_vector:

ejection_direction: {x: float, y: float, z: float}

user_confirmed: true

policy_echo:

min_draft_deg_default: float | null

min_draft_deg_by_surface_class: dict | null [CHƯA XÁC MINH]

silhouette_eps_dot: float | null [CHƯA XÁC MINH]

sampling_policy_id: string | null [CHƯA XÁC MINH]

geometry_fidelity: "brep" | "mesh" | "hybrid" | null [CHƯA XÁC MINH: lấy từ GeoModel/SSOT]

metrics:

sampling_summary:

face_count_total: int [CHƯA XÁC MINH: có đếm total hay chỉ evaluated]
