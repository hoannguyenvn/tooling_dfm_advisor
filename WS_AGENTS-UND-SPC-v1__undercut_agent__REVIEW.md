WS_AGENTS-UND-SPC-v1 (REVIEW)

# 0. Document Control Block

|                                  |                                                                      |
| -------------------------------- | -------------------------------------------------------------------- |
| Canonical Code                   | WS_AGENTS-UND-SPC-v1                                                 |
| Status                           | REVIEW                                                               |
| Run ID (UTC)                     | 20260304T104002Z                                                     |
| Created date (UTC)               | 20260304T104002Z                                                     |
| WS / Module / Doc Type / Version | WS_AGENTS / UND / SPC / v1                                           |
| Purpose                          | Document governance consolidation only; no runtime/pipeline changes. |

## 1. Scope

This document consolidates the undercut_agent specification for DFM-first screening. Content is inherited from sources; no technical semantics are changed.

Non-goals: no CAE simulation; no automatic geometry fixes; no redesign of architecture/pipeline; no change to SSOT, contracts, or execution semantics.

## 2. Included Sources & Mapping Table

|               |                                                                |                                                                             |                             |                   |                                                                                            |
| ------------- | -------------------------------------------------------------- | --------------------------------------------------------------------------- | --------------------------- | ----------------- | ------------------------------------------------------------------------------------------ |
| **source_id** | **filename**                                                   | **original_title**                                                          | **original_version/status** | **sections_used** | **notes**                                                                                  |
| S1            | TÀI LIỆU TRIỂN KHAI ADN AMBR LDR — MODULE undercut_agent .docx | TÀI LIỆU TRIỂN KHAI ADN / AMBR / LDR — MODULE undercut_agent (DFM-first V1) | DRAFT                       | A,B,C,D,E,F       | Primary undercut_agent DFM-first V1 spec (ADN/AMBR/LDR + schema sketch + defect template). |

## 3. Consolidated Specification Content

## 3.1 ADN (Algorithm / Logic)

*Source: S1*

B) ADN — undercut_agent (V1 baseline)

IDENTIFICATION
adn_id: ADN_undercut_agent_undercut_sampling_and_band_clustering_with_side_action_pressure_index_v1
module_name: undercut_agent
module_type: analysis_agent
algorithm_name: undercut_sampling_and_band_clustering_with_side_action_pressure_index
algorithm_version: v1
status: DRAFT

PROBLEM STATEMENT (DFM thực chiến)
Core problem: phát hiện vùng undercut theo hướng rút khuôn đã user_confirmed và cung cấp “tín hiệu áp lực cơ cấu” (SAPI) để kỹ sư khuôn quyết định mức độ cần side action (slide/lifter/split). V1 phải làm được trên mặt cong (fillet, loft, boss, dome…) vì đây là phần chính yếu của sản phẩm nhựa; planar-only làm giảm giá trị DFM.

Scope (V1 baseline):

Tính undercut candidate trên cả planar và non-planar faces bằng sampling normal theo (u,v).

Gom cụm (clustering) các candidate thành “band/patch undercut” để tránh report rải rác và để có coverage/extent phục vụ quyết định khuôn.

Tính chỉ số SAPI (side-action pressure index) dựa trên (độ “ngược” dot), coverage, và tính liên tục của band; đây là proxy chi phí/độ phức tạp khuôn, không phải CAE.

Xuất worst clusters theo region_ref chuẩn, không lưu path/pointer (no shadow data).

Non-goals:

Không phân loại chắc chắn “slide hay lifter” (để V2 khi có thêm logic hướng undercut vector / clearance check).

Không dự đoán hành vi tháo khuôn dưới biến dạng đàn hồi nhựa.

Không tự thay đổi ejection_direction hay parting; chỉ advisory.

INPUT FORMALIZATION
Required:

geometry_context: AGL B-Rep shape (TopoDS_Shape). [CHƯA XÁC MINH: payload key path cụ thể]

operational_vector: ejection_direction (unit vector) và user_confirmed=true (hard precondition).

dfm_policy.units.length_unit = "mm" (không dùng trực tiếp trong dot nhưng giữ thống nhất hệ).

dfm_policy.thresholds.undercut.reverse_dot_threshold (mặc định hợp lý là 0.0; đây là định nghĩa hình học “ngược hướng”, không phải ngưỡng DFM tùy ý).

Optional (khuyến nghị để V1 ổn định/kiểm soát runtime):

dfm_policy.common.sampling_budget: max_worst_regions, max_faces_to_report.

dfm_policy.common.silhouette_eps_dot (để nhận diện vùng gần dot≈0 ổn định hơn sign-change).

undercut_clustering parameters (đặt trong dfm_policy.thresholds.undercut.clustering.\*) [CHƯA XÁC MINH: bạn muốn thêm keys này vào policy catalog mức nào; V1 vẫn có thể chạy với default nội bộ nhưng nên đưa vào policy để audit].

Fail-safe dependency:

Nếu OCC/pythonocc-core thiếu: status NOT_EXECUTED/STUB_MODE, issue OCC_MISSING; không crash pipeline.

OUTPUT FORMALIZATION
Output envelope: UNDERCUT_ANALYSIS_RESULT_v0 (tuân RESULT_ENVELOPE_API_v1 tối thiểu: result_type, run_id, status, error_code/disable_reason, rule_version, geometry_version_id, metrics, issues, produced_defects, receipts_ref).

V1 metrics (đủ cho quyết định khuôn):

sampling_summary:

face_count_total, face_count_evaluated

sample_count_total

nonplanar_face_count_evaluated

undercut_facts:

candidate_sample_ratio (dot < threshold)

candidate_face_ratio

undercut_depth_distribution: max_depth, p90_depth (depth = max(0, -(dot - reverse_dot_threshold)))

clustering_summary:

cluster_count

largest_cluster_coverage_ratio

worst_clusters: top N

side_action_pressure:

SAPI_global (0..1)

SAPI_max_cluster (0..1)

decision_hint: NONE / REVIEW / SIDE_ACTION_LIKELY (policy-mapped; advisory only) [CHƯA XÁC MINH: closed-set bạn muốn dùng]

Region refs:

worst_clusters[i].region_ref phải theo REGION_REF_CONVENTION_v1 (AGL + FACE/PATCH + AGL_FACE_INDEX_V1, có uv_bbox nếu là PATCH). Không lưu đường dẫn file.

ALGORITHM OVERVIEW (V1 baseline, deterministic, không CAE)

Khái niệm nền:

pull_dir = p (unit).

normal tại sample = n (unit, đã xét orientation).

dot = n·p.
Undercut candidate khi dot < reverse_dot_threshold (thường 0.0). “Độ ngược” (undercut_depth) dùng để xếp hạng mức độ rủi ro.

V1 khác V0 ở 3 điểm:
(1) Không planar-only: mọi face đều có thể đánh giá bằng sampling (u,v) phù hợp với loại bề mặt (analytic/freeform).
(2) Không chỉ “đếm candidate”: gom candidate thành cluster/band để nói rõ “một vùng undercut liên tục” (thực tế mold cost phụ thuộc band/extent).
(3) Tạo SAPI để chuyển từ “phát hiện” sang “áp lực cơ cấu”.

STEP-BY-STEP LOGIC

Step 1 — Initialize envelope
Tạo UNDERCUT_ANALYSIS_RESULT_v0, status=OK (tentative), issues=[].

Step 2 — Validate operational_vector

thiếu ejection_direction → FAILED (PULL_DIR_MISSING)

norm=0 → FAILED (PULL_DIR_INVALID)

user_confirmed != true → FAILED (EJECTION_NOT_CONFIRMED)
Đây là hard fail vì undercut không có nghĩa nếu chưa chốt hướng rút.

Step 3 — Validate geometry + OCC availability

thiếu brep_shape → FAILED (GEOM_CONTEXT_MISSING)

OCC missing → NOT_EXECUTED/STUB_MODE (OCC_MISSING), return envelope (không crash).

Step 4 — Face traversal + sampling plan (curved-surface coverage)

Traverse faces: TopExp_Explorer(shape, TopAbs_FACE).

Với mỗi face: xác định surface type (GeomAbs) và lập grid (u,v) deterministic.

analytic surfaces: grid cố định (ví dụ 3×3 hoặc 5×5 tùy budget).

freeform: grid thưa + tăng density theo curvature proxy (nếu truy cập được) [CHƯA XÁC MINH: phương pháp curvature mà bạn muốn, nếu có].

Mục tiêu: bắt min dot / sign-change, không cần heatmap mịn.

Step 5 — Normal evaluation + candidate detection

Tại mỗi sample (u,v): tính normal n bằng BRepLProp_SLProps (hoặc tương đương), đảo chiều nếu face orientation TopAbs_REVERSED.

dot = n·p.

candidate nếu dot < reverse_dot_threshold.

undercut_depth = max(0, reverse_dot_threshold - dot) (với threshold=0 => depth=-dot nếu dot\<0).
Ghi nhận sample-level facts (không cần lưu toàn bộ, chỉ stream stats + giữ top candidates).

Step 6 — Band clustering (gom cụm undercut)
Mục tiêu là ra “vùng undercut liên tục” thay vì hàng nghìn điểm.

Cụm hoá tối thiểu (deterministic):

Node = candidate sample (kèm face_id + uv_point).

Connect edges nếu:

cùng face và khoảng cách UV < uv_eps, hoặc

khác face nhưng faces adjacent (share edge) và khoảng cách 3D giữa points < xyz_eps.

Union-Find hoặc BFS để tạo clusters.
Các eps nên nằm trong policy (clustering.uv_eps, clustering.xyz_eps). Nếu chưa có, dùng default conservative và gắn issue CLUSTERING_DEFAULTS_USED. [CHƯA XÁC MINH]

Cluster metrics:

cluster_sample_count

cluster_coverage_ratio = cluster_sample_count / total_valid_samples (fallback) hoặc / evaluated_area nếu có area.

depth_max, depth_p90

cluster_span_score: span của cluster theo phương chiếu lên mặt phẳng vuông góc pull_dir (proxy độ “dài band”). [CHƯA XÁC MINH: bạn có muốn span_score không; nếu không có thì bỏ]

Step 7 — Side-Action Pressure Index (SAPI)
SAPI là proxy “áp lực phải dùng cơ cấu” dựa trên độ ngược + coverage + continuity. Không có CAE nên đây là heuristic minh bạch và policy-controlled.

Định nghĩa V1 (đề xuất, tất cả tham số nên đặt trong policy):

depth_score = clamp01(depth_p90 / depth_ref) với depth_ref [CHƯA XÁC MINH] (gợi ý 0.30)

size_score = clamp01(coverage_ratio / coverage_ref) với coverage_ref [CHƯA XÁC MINH] (gợi ý 0.05)

continuity_score = clamp01(span_score / span_ref) nếu có span_score; nếu chưa có thì continuity_score = clamp01(cluster_sample_count / count_ref) [CHƯA XÁC MINH]

SAPI_cluster = w1size_score + w2depth_score + w3\*continuity_score, với w1+w2+w3=1 (ví dụ 0.5/0.4/0.1) [CHƯA XÁC MINH]

SAPI_global = max over clusters (vì “một band nặng” có thể quyết định toàn bộ cấu trúc khuôn)

Decision_hint (advisory, policy-mapped):

if SAPI_max_cluster >= sapi_side_action_likely_threshold => SIDE_ACTION_LIKELY

elif >= sapi_review_threshold => REVIEW

else NONE
Các threshold này thuộc dfm_policy.thresholds.undercut.sapi_thresholds.\* [CHƯA XÁC MINH]

Step 8 — Emit defects (top N clusters)

Emit defect UNDERCUT_BAND_DETECTED cho các cluster vượt ngưỡng coverage/depth tối thiểu.

Emit defect UNDERCUT_SIDE_ACTION_PRESSURE_HIGH nếu SAPI_cluster vượt threshold.

Mỗi defect phải có region_ref (FACE hoặc PATCH với uv_bbox) theo convention v1; không lưu file paths.

Step 9 — Finalize envelope

Fill worst_clusters, SAPI metrics, issues, produced_defects; status OK/FAILED/NOT_EXECUTED.

FAILURE MODES

PULL_DIR_MISSING / PULL_DIR_INVALID / EJECTION_NOT_CONFIRMED → FAILED (agent hard fail; pipeline vẫn phải report).

GEOM_CONTEXT_MISSING → FAILED

OCC_MISSING → NOT_EXECUTED/STUB_MODE (fail-safe).

POLICY_UNRESOLVED (nếu thiếu clustering/sapi thresholds) → OK nhưng chỉ facts + issue POLICY_UNRESOLVED, confidence giảm.

COMPLEXITY
Time ~ O(F\*S + K) với F faces, S samples/face, K clustering edges (đã giới hạn bởi sampling_budget). Memory ~ O(N_candidates) tạm thời cho clustering; giới hạn bởi policy để không nổ.

V2 UPGRADE INPUTS (để phân loại slide/lifter “thật hơn”)

Clearance check bằng ray casting theo direction mở khuôn và direction side-action dự kiến (still heuristic, không CAE).

Hướng undercut vector field và phân tích “đường thoát” để phân loại: slide (ngang) vs lifter (xiên) dựa trên góc với pull_dir. [CHƯA XÁC MINH: taxonomy phân loại bạn muốn]

AFR feature graph để gắn undercut với lỗ/boss/rib (giảm false positives).

## 3.2 AMBR (Bindings / Module Contract / Execution)

*Source: S1*

A) NAMING & IDENTIFIERS (NORMALIZED)
file_name: undercut_agent.py
module_name: undercut_agent
module_type: analysis_agent (dfm undercut detection)

baseline facts (V0): chỉ xét planar faces, lấy normal tại điểm đại diện, nếu dot(normal_face, pull_dir) < 0 thì undercut candidate; fail-safe khi thiếu OCC.

algorithm_name (V1 proposed): undercut_sampling_and_band_clustering_with_side_action_pressure_index
algorithm_version (V1 proposed): v1
result_envelope_id: UNDERCUT_ANALYSIS_RESULT_v0 (giữ v0 để tương thích pipeline/report; bổ sung fields V1 dạng optional).

C) AMBR — undercut_agent (binding)

ambr_id: AMBR_undercut_agent_v2_research
status: DRAFT
module_name: undercut_agent
module_type: analysis_agent
execution_mode: SEQUENTIAL (internal), orchestration PARALLEL-safe.

pipeline_stage: Step 5 (Parallel Analysis Agents).

bound_algorithm: ADN_undercut_agent_undercut_sampling_and_band_clustering_with_side_action_pressure_index_v1
bound_policy_keys:

dfm_policy.thresholds.undercut.reverse_dot_threshold

dfm_policy.thresholds.undercut.sapi_thresholds.\* [CHƯA XÁC MINH]

dfm_policy.common.sampling_budget

## 3.3 LDR (Dependencies / Runtime constraints)

*Source: S1*

D) LDR — dependencies (research)

ldr_id: LDR_pythonocc_core_surface_normals_topology_for_undercut_agent_v1
status: DRAFT
library_name: pythonocc-core (OCC.Core.TopExp, TopAbs, BRepAdaptor, BRepLProp, GeomAbs, gp)

purpose: traverse faces + evaluate surface normals + deterministic sampling.
risk: heavy dependency; must fail-safe to NOT_EXECUTED/STUB_MODE.

## 4. Open Questions / [CHƯA XÁC MINH]

The following items are explicitly marked as [CHƯA XÁC MINH] in the sources and remain unresolved:

Q1: THÔNG TIN PHIÊN BẢN (DOCUMENT METADATA)
doc_id: TOOLING_DFM_ADVISOR_UNDERCUT_AGENT_DFM_V1_RESEARCH_v1
status: DRAFT
effective_date_utc: [CHƯA XÁC MINH]
owner: [CHƯA XÁC MINH]
scope: nâng undercut từ V0 “planar-only dot\<0” lên V1 “curved-surface coverage + clustering + side-action pressure index (SAPI)” trong vai trò advisory pre-tooling (không CAE, không auto-fix).

Q2: Union-Find hoặc BFS để tạo clusters.
Các eps nên nằm trong policy (clustering.uv_eps, clustering.xyz_eps). Nếu chưa có, dùng default conservative và gắn issue CLUSTERING_DEFAULTS_USED. [CHƯA XÁC MINH]

Q3: depth_score = clamp01(depth_p90 / depth_ref) với depth_ref [CHƯA XÁC MINH] (gợi ý 0.30)

Q4: size_score = clamp01(coverage_ratio / coverage_ref) với coverage_ref [CHƯA XÁC MINH] (gợi ý 0.05)

Q5: continuity_score = clamp01(span_score / span_ref) nếu có span_score; nếu chưa có thì continuity_score = clamp01(cluster_sample_count / count_ref) [CHƯA XÁC MINH]

Q6: SAPI_cluster = w1size_score + w2depth_score + w3\*continuity_score, với w1+w2+w3=1 (ví dụ 0.5/0.4/0.1) [CHƯA XÁC MINH]

Q7: else NONE
Các threshold này thuộc dfm_policy.thresholds.undercut.sapi_thresholds.\* [CHƯA XÁC MINH]

Q8: dfm_policy.thresholds.undercut.sapi_thresholds.\* [CHƯA XÁC MINH]

Q9: rule_version [CHƯA XÁC MINH]

Q10: receipts_ref: [CHƯA XÁC MINH]

## 5. Change Log (Editorial)

This revision performs document governance consolidation only:

- Renamed source file into canonical naming: WS_AGENTS-UND-SPC-v1\_\_undercut_agent\_\_REVIEW.docx

- Added Document Control Block and Included Sources & Mapping Table.

- Preserved technical content via verbatim extracts and added traceability 'Source: Sx' at each subsection.

- Removed repetitive placeholder lines (e.g., truncated cross-reference placeholders) that do not carry technical meaning.

- Moved result schema sketch and defect advisory template to Appendix to avoid duplicating global model specifications; referenced by canonical codes where applicable.

## 6. Appendix

Cross references (canonical only): WS_MODEL-GEN-SPC-v1 (result container baseline), WS_MODEL-RGN-POL-v1 (region_ref convention).

## Appendix A — UNDERCUT_ANALYSIS_RESULT_v0 schema sketch (source extract)

*Source: S1*

E) UNDERCUT_ANALYSIS_RESULT_v0 — schema sketch (V1 additions)

UNDERCUT_ANALYSIS_RESULT_v0:

result_type: "UNDERCUT_ANALYSIS_RESULT_v0"

run_id

status: OK|FAILED|NOT_EXECUTED|STUB_MODE

error_code/disable_reason

rule_version [CHƯA XÁC MINH]

geometry_version_id

inputs_echo:

operational_vector: {ejection_direction: {x,y,z}, user_confirmed:true}

policy_echo: {reverse_dot_threshold: 0.0, sampling_budget: {...}, sapi_thresholds: {...}|null}

metrics:

sampling_summary: {face_count_evaluated, sample_count_total, nonplanar_face_count_evaluated}

undercut_facts: {candidate_sample_ratio, depth_max, depth_p90}

clustering_summary: {cluster_count, largest_cluster_coverage_ratio}

side_action_pressure: {sapi_global, sapi_max_cluster, decision_hint}

worst_clusters: \[
{
region_ref: REGION_REF_SCHEMA_v1,
coverage_ratio: float,
depth_p90: float,
sapi_cluster: float,
over_sapi_threshold: bool|null,
confidence_score: float
}
\]

issues: list[issue_obj]

produced_defects: list[str]

receipts_ref: [CHƯA XÁC MINH]

## Appendix B — Defect advisory template (source extract)

*Source: S1*

F) Defect advisory template — undercut_agent V1

UNDERCUT_DEFECT_TEMPLATE_v1:

id

run_id

agent_source: "undercut_agent"

rule_version

geometry_version_id

timestamp_utc

severity: INFO|WARN|ERROR|FATAL (theo dfm_policy.severity_map.undercut.\* mà bạn đã chốt trong complete set)

defect_type:

"UNDERCUT_BAND_DETECTED"

"UNDERCUT_SIDE_ACTION_PRESSURE_HIGH"

"UNDERCUT_LOW_COVERAGE_NONPLANAR"

operational_vector: {ejection_direction, user_confirmed:true}

region_ref: REGION_REF_SCHEMA_v1 (AGL FACE/PATCH)

analysis_value:

depth_p90: float

coverage_ratio: float

sapi_cluster: float

method: "V1_SAMPLED_CLUSTERED"

threshold_applied:

reverse_dot_threshold: float

sapi_threshold: float|null

advisory:

machine_readable:

summary_code: "REVIEW_PARTING" | "SIDE_ACTION_LIKELY" | "SIMPLIFY_GEOMETRY" | "CONFIRM_EJECTION"

recommended_actions: [...]

rationale_tags: ["UNDERCUT", "BAND", "HIGH_SAPI", "NONPLANAR"]

human_readable:

title

why_it_matters (kẹt khuôn/drag, bắt buộc slide/lifter/split)

suggested_fix (đổi split/parting, thêm side action, sửa geometry)

verification (xem lại pull direction, kiểm band continuity, cross-check draft)

confidence_score

assets_ephemeral_ref (token only)

delivery_scrubbed
