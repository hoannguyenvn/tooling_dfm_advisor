# WS_AGENTS-ALL-POL-v1 (REVIEW)

## 0. Document Control Block

|                                  |                                                                      |
| -------------------------------- | -------------------------------------------------------------------- |
| Canonical Code                   | WS_AGENTS-ALL-POL-v1                                                 |
| Status                           | REVIEW                                                               |
| Run ID (UTC)                     | 20260304T112416Z                                                     |
| Created date (UTC)               | 20260304 112416                                                      |
| WS / Module / Doc Type / Version | WS=WS_AGENTS; MODULE_3LA=ALL; DOC_TYPE=POL; VERSION=v1               |
| Purpose                          | Document governance consolidation only; no runtime/pipeline changes. |

## 1. Scope

Tài liệu này gom và chuẩn hoá policy-as-config dùng chung cho toàn bộ DFM agents, bao gồm: catalog policy keys tối thiểu, severity map (complete set), và ví dụ cấu hình policy mặc định. Mọi nội dung là kế thừa trực tiếp từ các nguồn đính kèm (Source: Sx).

Non-goals: không thay đổi logic kỹ thuật, không thay đổi semantics của bất kỳ key/threshold/rule; không đề xuất pipeline/runtime mới; không thay đổi SSOT, receipt, hay các gate hiện hữu.

## 2. Included Sources & Mapping Table

|           |                                                                                |                                                     |                                     |               |                                                   |
| --------- | ------------------------------------------------------------------------------ | --------------------------------------------------- | ----------------------------------- | ------------- | ------------------------------------------------- |
| source_id | filename                                                                       | original_title                                      | original_version/status             | sections_used | notes                                             |
| S1        | DFM_POLICY_MASTER_SPEC_v2_0_FROZEN.docx                                        | DFM_POLICY_MASTER_SPEC_v2_0                         | FROZEN v2.0                         | 3.1           | Policy master spec (why/invariants/fail-fast)     |
| S5        | DFM_POLICY_KEYS_MINIMUM_v1 — BẢN HỢP NHẤT (UPDATED, CONSOLIDATED).docx         | DFM_POLICY_KEYS_MINIMUM_v1 (consolidated)           | APPROVED (Owner-locked)             | 3.2           | Primary catalog keys minimum                      |
| S3        | PHỤ LỤC CẬP NHẬT — DFM_POLICY_KEYS_MINIMUM_v1 (ADDENDUM_GEOM_ENABLERS_v1).docx | DFM_POLICY_KEYS_MINIMUM_v1 addendum (geom enablers) | APPROVED (Owner-locked)             | 3.3           | Adds validation/healing keys                      |
| S4        | TÀI LIỆU — DFM_POLICY_SEVERITY_MAP_COMPLETE_SET_v1.docx                        | DFM_POLICY_SEVERITY_MAP_COMPLETE_SET_v1             | DRAFT                               | 3.4           | Complete severity map proposal                    |
| S6        | DFM_POLICY_v1 (default).docx                                                   | DFM_POLICY_v1 (default)                             | (no explicit status)                | 3.5           | Default example policy JSON                       |
| S2        | PHẦN 1 — DFM_POLICY_KEYS_MINIMUM_v1.docx                                       | PHẦN 1 — DFM_POLICY_KEYS_MINIMUM_v1                 | APPROVED (Owner-locked) [as stated] | Appendix A    | Historical/partial; superseded by consolidated S5 |

## 3. Consolidated Policy Content

## 3.1 Policy Master Specification (Why / Invariants / Fail-Fast)

Source: S1

DFM_POLICY_MASTER_SPEC_v2_0_FROZEN

doc_id: DFM_POLICY_MASTER_SPEC_v2_0

status: FROZEN

system_name: tooling_dfm_advisor

architecture_level: INDUSTRIAL_GRADE_FREEZE

version: V2_0

freeze_date_utc: 2026-02-27T13:59:27Z

Why

Centralize all DFM thresholds to guarantee determinism and prevent agent-level hardcoding.

Preconditions

Governance policy framework exists. Versioning scheme defined.

Inputs

threshold definitions, severity map, policy_version.

Outputs

Versioned policy configuration usable by orchestration layer.

Determinism Invariant

Same geometry_version_id + policy_version must yield identical evaluation.

Audit Requirement

Policy change requires ADR and golden baseline update.

Fail-Fast

If threshold undefined, agent execution must stop.

Cross Reference

Bound to ANALYSIS_RESULT_CONTRACT_v2_0 and GOLDEN_REGRESSION_STRATEGY_v2_0.

## 3.2 Policy Keys Minimum (Catalog Keys + Units + Fail-safe)

Source: S5

DFM_POLICY_KEYS_MINIMUM_v1 — BẢN HỢP NHẤT (UPDATED, CONSOLIDATED)
(doc_id: TOOLING_DFM_ADVISOR_DFM_POLICY_KEYS_MINIMUM_v1)
status: APPROVED (Owner-locked)
effective_date_utc: [CHƯA XÁC MINH]
scope: Catalog key policy tối thiểu (units + common + thresholds + enablers + afr), dùng chung cho toàn bộ agent/provider trong tooling_dfm_advisor.
spec anchors: Master Spec + Governance Spec (policy-as-config, fail-safe, report always-on).

Mục tiêu và bất biến
Policy-as-config: mọi ngưỡng/knobs phải nằm trong config versioned; agent không hard-code ngưỡng DFM.

Fail-safe: thiếu key hoặc thiếu deps không được làm sập pipeline; agent phải trả “facts-only” + issue POLICY_UNRESOLVED / NOT_EXECUTED / STUB_MODE tùy trường hợp.

No shadow data: policy không chứa đường dẫn file hay pointer; chỉ chứa cấu hình và ngưỡng.

Units locked: length_unit = "mm" (đã chốt); angle_unit = "deg".

Vị trí lưu và precedence
Canonical: config/governance_policy.json -> dfm_policy
Override dev (gitignored): config/policy_override.local.json (chỉ dùng tạm, phải có audit khi áp dụng).

Precedence: override.local > governance_policy.json.

Quy ước “required”
Trong catalog dưới đây, mình dùng 2 mức:

REQUIRED: thiếu là POLICY_INVALID (agent phải dừng/FAILED hoặc OK nhưng không thể chạy meaningful theo design).

REQUIRED_FOR_THRESHOLD_EVAL: thiếu thì agent vẫn được phép chạy “facts-only” nhưng tuyệt đối không emit defect kiểu “over/under threshold”; phải gắn issue POLICY_UNRESOLVED.

CATALOG KEYS

4.1. dfm_policy (root)

dfm_policy.dfm_policy_version (REQUIRED): string, ví dụ "DFM_POLICY_v1"

dfm_policy.units (REQUIRED):

length_unit (REQUIRED): "mm" (locked)

angle_unit (REQUIRED): "deg" (locked)

dfm_policy.region_ref_convention_id (REQUIRED): "REGION_REF_CONVENTION_v1"

4.2. dfm_policy.common (OPTIONAL nhưng khuyến nghị)

dfm_policy.common.silhouette_eps_dot: float (0..1). Dùng cho draft/undercut “near silhouette”. Nếu thiếu, các agent dùng sign-change và ghi limitation.

dfm_policy.common.sampling_budget:

max_faces_to_report: int (default 50)

max_worst_regions: int (default 10)

dfm_policy.common.max_runtime_seconds (OPTIONAL): int|null, nếu muốn chặn tasks quá lâu (lane rule do governance quyết). [CHƯA XÁC MINH]

4.3. dfm_policy.thresholds.draft

min_draft_deg_default (REQUIRED_FOR_THRESHOLD_EVAL): float (deg)

min_draft_deg_by_surface_class (OPTIONAL): object

polish: float

light_texture: float

heavy_texture: float

reverse_dot_threshold (OPTIONAL, default 0.0): float. (Định nghĩa reverse theo dot-product; thường không nên thay.)

Fail-safe: thiếu min_draft_deg_default -> draft_agent chỉ xuất draft facts + reverse/silhouette facts, không kết luận below-threshold.

4.4. dfm_policy.thresholds.thickness

min_wall_mm (REQUIRED_FOR_THRESHOLD_EVAL): float (mm)

max_wall_mm (OPTIONAL): float (mm)

probe (OPTIONAL nhưng rất khuyến nghị để ổn định ray-probe):

self_hit_epsilon_mm: float (mm) (khuyến nghị có; nếu thiếu agent phải dùng default và issue THICKNESS_PROBE_DEFAULTS_USED)

max_probe_distance_mm: float|null (mm)

max_intersections_to_consider: int (default 8)

Fail-safe: thiếu min_wall_mm -> thickness_agent xuất distribution facts, không emit defect BELOW_MIN_WALL/ABOVE_MAX_WALL.

4.5. dfm_policy.thresholds.undercut

reverse_dot_threshold (OPTIONAL, default 0.0): float

clustering (OPTIONAL; nếu thiếu agent dùng default và issue CLUSTERING_DEFAULTS_USED):

uv_eps: float (dimensionless)

xyz_eps_mm: float (mm)

min_cluster_samples: int

max_clusters_to_report: int

sapi (OPTIONAL; nếu thiếu agent vẫn phát undercut bands nhưng không phát “pressure index decision”):

depth_ref: float (dimensionless, proxy từ -dot)

coverage_ref: float (0..1)

count_ref: int

weights: {w_size: float, w_depth: float, w_continuity: float} (sum=1)

thresholds: {review: float, side_action_likely: float} (0..1)

Fail-safe: thiếu sapi.\* -> undercut_agent vẫn báo undercut clusters + coverage/depth, nhưng decision_hint dựa SAPI phải là null + issue POLICY_UNRESOLVED_SAPI.

4.6. dfm_policy.thresholds.ribratio

max_rib_to_wall_ratio (REQUIRED_FOR_THRESHOLD_EVAL cho rib_to_wall V1): float (dimensionless)

preferred_rib_to_wall_ratio (OPTIONAL): float

min_root_radius_mm (OPTIONAL): float (mm)

Fail-safe: nếu agent đang proxy mode (bbox) thì chỉ emit “RIBRATIO_PROXY_ONLY_LOW_CONFIDENCE”, không dùng max_rib_to_wall_ratio để kết luận “vượt ngưỡng” theo nghĩa t_rib/t_wall.

4.7. dfm_policy.thresholds.sink
Bạn nên coi sink có 2 “lane”:

Proxy lane (V0): dùng khi chưa có thickness map

Local-mass lane (V1): dùng khi có thickness map

Keys:

max_sink_risk_proxy (OPTIONAL): float (0..1) (proxy lane)

local_mass (OPTIONAL nhưng khuyến nghị cho V1):

max_local_thickness_ratio (REQUIRED_FOR_THRESHOLD_EVAL khi chạy local-mass): float (dimensionless), thick_ratio = (t_local - t_nominal)/t_nominal

max_boss_to_wall_ratio (OPTIONAL): float (dimensionless) (dùng khi có boss tag)

Fail-safe: nếu không có cả max_sink_risk_proxy và local_mass.max_local_thickness_ratio -> sink_agent chỉ xuất facts (nominal + thick_ratio stats nếu có), không emit over-threshold.

4.8. dfm_policy.thresholds.weldline

max_weld_risk_proxy (REQUIRED_FOR_THRESHOLD_EVAL): float (dimensionless), proxy = edge_count/face_count

min_face_count_for_valid_proxy (OPTIONAL, default 20): int

Fail-safe: nếu thiếu max_weld_risk_proxy -> weldline_agent xuất topo facts + validity, không emit over-threshold.

4.9. dfm_policy.thresholds.validation (geometry_validation_gate)

enabled (OPTIONAL, default true): bool

self_intersection_check_mode (OPTIONAL, default "FAST"): enum {"FAST","FULL"}

min_volume_mm3 (OPTIONAL, default 1.0): float (mm^3)

nonmanifold_allowed (OPTIONAL, default 0): int

max_findings_to_report (OPTIONAL, default 20): int

max_runtime_seconds (OPTIONAL, default null): int|null

Fail-safe: nếu thiếu keys -> dùng default + issue VALIDATION_DEFAULTS_USED. Nếu checker FULL không khả dụng -> degrade về FAST + issue SELF_INTERSECTION_FULL_NOT_AVAILABLE (không “PASS giả”). [CHƯA XÁC MINH: checker availability khi implement]

4.10. dfm_policy.thresholds.healing (healer_agent)

mode (OPTIONAL, default "MINIMAL_FIX_ONLY"): enum {"MINIMAL_FIX_ONLY","AGGRESSIVE_FIX"}

sewing_tolerance_mm (OPTIONAL, default 0.05): float (mm)

max_tolerance_delta_mm (OPTIONAL, default 0.10): float (mm)

max_topology_change_allowed (OPTIONAL, default "MINOR_ONLY"): enum {"NONE","MINOR_ONLY","ALLOW"}

max_heal_time_seconds (OPTIONAL, default null): int|null

emit_observability (OPTIONAL, default true): bool

Fail-safe: unknown enum value -> POLICY_INVALID. Nếu tolerance_delta không đo được -> tolerance_delta_mm=null + issue TOLERANCE_DELTA_UNAVAILABLE (minh bạch).

4.11. dfm_policy.thresholds.afr (AFR V1 precision-first)
Mục tiêu: tag ít nhưng chắc. Nếu thiếu key thì AFR suppress tag và phát issue, không đoán.

enabled (OPTIONAL, default true): bool

max_tags_total (OPTIONAL, default 50): int

min_confidence_to_emit_tag (OPTIONAL, default 0.75): float (0..1)

classifier_probe_eps_mm (OPTIONAL, default 0.20): float (mm)

min_cylinder_radius_mm (OPTIONAL, default 0.50): float (mm)

min_cylinder_face_area_mm2 (OPTIONAL, default 5.0): float (mm^2) [CHƯA XÁC MINH: có dùng area trong implement không; nếu không dùng thì bỏ key này]

depends_on_thickness (OPTIONAL):

max_miss_rate_for_afr (OPTIONAL, default 0.10): float (0..1)

min_sample_count_for_afr (OPTIONAL, default 200): int

rib_candidate (OPTIONAL):

rib_ratio_emit_min (OPTIONAL, default 0.35): float

rib_ratio_emit_max (OPTIONAL, default 0.80): float

rib_aspect_ratio_min (OPTIONAL, default 5.0): float

thick_junction (OPTIONAL):

junction_thick_ratio_min_to_tag (OPTIONAL, default 0.50): float

Fail-safe: thiếu thickness_result hoặc thickness coverage kém -> suppress rib/junction tags + issue AFR_DEPENDS_ON_THICKNESS_LOW_COVERAGE.

Severity map keys (không bắt buộc trong MINIMUM, nhưng khuyến nghị dùng chung)
Nếu có dfm_policy.severity_map, các agent sẽ map severity nhất quán theo config (đã xây bộ “complete set”). Nếu không có, agent có thể set severity conservative (WARN) và phát issue SEVERITY_POLICY_UNRESOLVED. [CHƯA XÁC MINH: bạn có muốn enforce severity_map bắt buộc không]

JSON skeleton tối thiểu (để bạn copy vào governance_policy.json)

{
"dfm_policy": {
"dfm_policy_version": "DFM_POLICY_v1",
"region_ref_convention_id": "REGION_REF_CONVENTION_v1",
"units": { "length_unit": "mm", "angle_unit": "deg" },

"common": {
"silhouette_eps_dot": null,
"sampling_budget": { "max_faces_to_report": 50, "max_worst_regions": 10 }
},

"thresholds": {
"draft": {
"min_draft_deg_default": null,
"min_draft_deg_by_surface_class": { "polish": null, "light_texture": null, "heavy_texture": null },
"reverse_dot_threshold": 0.0
},
"thickness": {
"min_wall_mm": null,
"max_wall_mm": null,
"probe": { "self_hit_epsilon_mm": null, "max_probe_distance_mm": null, "max_intersections_to_consider": 8 }
},
"undercut": {
"reverse_dot_threshold": 0.0,
"clustering": { "uv_eps": null, "xyz_eps_mm": null, "min_cluster_samples": null, "max_clusters_to_report": 10 },
"sapi": {
"depth_ref": null, "coverage_ref": null, "count_ref": null,
"weights": { "w_size": 0.50, "w_depth": 0.40, "w_continuity": 0.10 },
"thresholds": { "review": null, "side_action_likely": null }
}
},
"ribratio": {
"max_rib_to_wall_ratio": null,
"preferred_rib_to_wall_ratio": null,
"min_root_radius_mm": null
},
"sink": {
"max_sink_risk_proxy": null,
"local_mass": { "max_local_thickness_ratio": null, "max_boss_to_wall_ratio": null }
},
"weldline": {
"max_weld_risk_proxy": null,
"min_face_count_for_valid_proxy": 20
},
"validation": {
"enabled": true,
"self_intersection_check_mode": "FAST",
"min_volume_mm3": 1.0,
"nonmanifold_allowed": 0,
"max_findings_to_report": 20,
"max_runtime_seconds": null
},
"healing": {
"mode": "MINIMAL_FIX_ONLY",
"sewing_tolerance_mm": 0.05,
"max_tolerance_delta_mm": 0.10,
"max_topology_change_allowed": "MINOR_ONLY",
"max_heal_time_seconds": null,
"emit_observability": true
},
"afr": {
"enabled": true,
"max_tags_total": 50,
"min_confidence_to_emit_tag": 0.75,
"classifier_probe_eps_mm": 0.20,
"min_cylinder_radius_mm": 0.50,
"min_cylinder_face_area_mm2": 5.0,
"depends_on_thickness": { "max_miss_rate_for_afr": 0.10, "min_sample_count_for_afr": 200 },
"rib_candidate": { "rib_ratio_emit_min": 0.35, "rib_ratio_emit_max": 0.80, "rib_aspect_ratio_min": 5.0 },
"thick_junction": { "junction_thick_ratio_min_to_tag": 0.50 }
}
}
}
}

## 3.3 Geometry Enablers Addendum (validation.\* / healing.\*)

Source: S3

PHỤ LỤC CẬP NHẬT — DFM_POLICY_KEYS_MINIMUM_v1 (ADDENDUM_GEOM_ENABLERS_v1)
doc_id: TOOLING_DFM_ADVISOR_DFM_POLICY_KEYS_MINIMUM_v1_ADDENDUM_GEOM_ENABLERS_v1
status: APPROVED (Owner-locked)
effective_date_utc: [CHƯA XÁC MINH]
scope: thêm keys cho geometry_validation_gate và healer_agent

Vị trí lưu keys
Giữ nguyên quy ước: config/governance_policy.json -> dfm_policy -> thresholds -> (validation|healing)
Override dev: config/policy_override.local.json (gitignored).

Nhóm keys: dfm_policy.thresholds.validation.\*

2.1. Mục đích
Điều khiển hành vi geometry_validation_gate (validators) trước freeze:

bật/tắt gate

mức độ kiểm self-intersection (FAST/FULL)

ngưỡng thể tích tối thiểu để chặn degenerate

số findings tối đa để report (tránh phình)

2.2. Catalog keys (semantics + units + required/optional)

A) enabled

key: dfm_policy.thresholds.validation.enabled

type: bool

required: OPTIONAL (default true)

semantics: nếu false, gate được phép trả DEFERRED/NOT_EXECUTED với issue VALIDATION_DISABLED_BY_POLICY (không crash).

risk: tắt gate làm giảm độ tin cậy DFM downstream.

B) self_intersection_check_mode

key: dfm_policy.thresholds.validation.self_intersection_check_mode

type: string enum {"FAST","FULL"}

required: OPTIONAL (default "FAST")

semantics: FAST ưu tiên tốc độ, FULL ưu tiên độ chắc chắn; nếu checker FULL không khả dụng thì gate phải báo issue SELF_INTERSECTION_FULL_NOT_AVAILABLE và degrade về FAST, không “PASS giả”. [CHƯA XÁC MINH: checker availability thực tế khi implement]

C) min_volume_mm3

key: dfm_policy.thresholds.validation.min_volume_mm3

type: float

unit: mm^3

required: OPTIONAL (default 1.0)

semantics: nếu volume \<= min_volume_mm3 thì FAIL (ZERO_OR_TOO_SMALL_VOLUME).

note: áp dụng cho solid; nếu model là surface-only thì có thể FAIL OPEN_SHELL trước. [CHƯA XÁC MINH: rule ưu tiên]

D) nonmanifold_allowed

key: dfm_policy.thresholds.validation.nonmanifold_allowed

type: int

unit: count

required: OPTIONAL (default 0)

semantics: số lượng non-manifold edges cho phép trước khi FAIL. Khuyến nghị 0 để “industrial grade”.

caveat: cho phép >0 chỉ khi có lý do và audit.

E) max_findings_to_report

key: dfm_policy.thresholds.validation.max_findings_to_report

type: int

unit: count

required: OPTIONAL (default 20)

semantics: giới hạn số finding (edge/face) đưa vào envelope để report không phình.

F) max_runtime_seconds (optional, nếu bạn muốn bảo vệ pipeline)

key: dfm_policy.thresholds.validation.max_runtime_seconds

type: int

unit: seconds

required: OPTIONAL (default null)

semantics: nếu vượt, gate trả FAILED hoặc DEFERRED với issue VALIDATION_TIMEOUT (tùy lane). [CHƯA XÁC MINH: lane rule]

Fail-safe khi thiếu keys:

Nếu thiếu các key OPTIONAL: dùng default như trên và ghi issue VALIDATION_DEFAULTS_USED (INFO/WARN tùy policy).

Nếu future bạn muốn strict: có thể nâng một số key thành REQUIRED (đặc biệt self_intersection_check_mode) nhưng hiện tại để OPTIONAL là hợp lý cho nghiên cứu.

Nhóm keys: dfm_policy.thresholds.healing.\*

3.1. Mục đích
Điều khiển healer_agent để “minimal-fix đúng mức” và minh bạch:

giới hạn can thiệp tolerance/sewing

chặn over-heal gây drift DFM

cung cấp policy_used cho observability addendum

3.2. Catalog keys (semantics + units + required/optional)

A) mode

key: dfm_policy.thresholds.healing.mode

type: string enum {"MINIMAL_FIX_ONLY","AGGRESSIVE_FIX"}

required: OPTIONAL (default "MINIMAL_FIX_ONLY")

semantics:

MINIMAL_FIX_ONLY: chỉ dùng ShapeFix_Shape.Perform với cấu hình bảo thủ; tránh thay đổi topo quá mức.

AGGRESSIVE_FIX: cho phép các fix mạnh hơn (chỉ dùng khi có lý do + corpus). [CHƯA XÁC MINH: mapping fix cụ thể trong OCC]

fail-safe: nếu unknown value → POLICY_INVALID.

B) sewing_tolerance_mm

key: dfm_policy.thresholds.healing.sewing_tolerance_mm

type: float

unit: mm

required: OPTIONAL (default 0.05)

semantics: tolerance dùng khi sewing/closing gaps (nếu healer thực hiện sewing). Nếu healer không làm sewing explicit thì vẫn echo policy_used để audit. [CHƯA XÁC MINH: healer hiện có sewing explicit hay chỉ ShapeFix generic]

C) max_tolerance_delta_mm

key: dfm_policy.thresholds.healing.max_tolerance_delta_mm

type: float

unit: mm

required: OPTIONAL (default 0.10)

semantics: giới hạn “độ thay đổi tolerance” cho phép. Nếu healer ước lượng/đo được tolerance_delta vượt ngưỡng → issue TOLERANCE_DELTA_EXCEEDED và requires_owner_review=true (hoặc FAIL theo lane). [CHƯA XÁC MINH: cơ chế đo tolerance_delta]

D) max_topology_change_allowed

key: dfm_policy.thresholds.healing.max_topology_change_allowed

type: string enum {"NONE","MINOR_ONLY","ALLOW"}

required: OPTIONAL (default "MINOR_ONLY")

semantics:

NONE: không cho phép topo counts đổi; nếu đổi → requires_owner_review.

MINOR_ONLY: cho phép thay đổi nhỏ (sliver removal), nhưng phải report topo_invariants_before/after.

ALLOW: cho phép thay đổi topo đáng kể (không khuyến nghị).

note: đây là policy-level safety rail cho DFM trust.

E) max_heal_time_seconds (optional)

key: dfm_policy.thresholds.healing.max_heal_time_seconds

type: int

unit: seconds

required: OPTIONAL (default null)

semantics: nếu healing vượt thời gian, healer trả ERROR/EXECUTED với issue HEALING_TIMEOUT (tùy lane). [CHƯA XÁC MINH]

F) emit_observability (optional)

key: dfm_policy.thresholds.healing.emit_observability

type: bool

required: OPTIONAL (default true)

semantics: nếu true, healer phải populate operations_applied, topo_invariants_before/after, tolerance_policy_used theo addendum; nếu false thì vẫn phải ít nhất populate fix_applied + validity flags. (Khuyến nghị true.)

Fail-safe khi thiếu keys:

Dùng default, đồng thời issue HEALING_DEFAULTS_USED (INFO).

Nếu policy thiếu nhưng healer cần để quyết định (ví dụ mode unknown), trả POLICY_INVALID và không “chữa bừa”.

Ví dụ block JSON để chèn vào governance_policy.json

{
"dfm_policy": {
"thresholds": {
"validation": {
"enabled": true,
"self_intersection_check_mode": "FAST",
"min_volume_mm3": 1.0,
"nonmanifold_allowed": 0,
"max_findings_to_report": 20,
"max_runtime_seconds": null
},
"healing": {
"mode": "MINIMAL_FIX_ONLY",
"sewing_tolerance_mm": 0.05,
"max_tolerance_delta_mm": 0.10,
"max_topology_change_allowed": "MINOR_ONLY",
"max_heal_time_seconds": null,
"emit_observability": true
}
}
}
}

Tác động (impact) đối với các agent DFM

draft/undercut: normal stability tăng; giảm false reverse/silhouette do topo lỗi.

thickness/sink/ribratio: ray-probe miss/hit bậy giảm mạnh nếu open shell/self-intersection bị chặn trước freeze.

report: có thể hiển thị “geometry_health” rõ (gate PASS/FAIL, healing applied, topology changed), tăng độ tin cậy khi tư vấn khách hàng.

## 3.4 Severity Map (Complete Set Proposal)

Source: S4

TÀI LIỆU — DFM_POLICY_SEVERITY_MAP_COMPLETE_SET_v1
doc_id: TOOLING_DFM_ADVISOR_DFM_POLICY_SEVERITY_MAP_COMPLETE_SET_v1
status: DRAFT
effective_date_utc: [CHƯA XÁC MINH]
units: length_unit=mm, angle_unit=deg

Nguyên tắc combine chung (áp dụng cho mọi agent)

Base severity theo “mức vi phạm” (deficit/excess/over-threshold).

Escalate theo “coverage” (violated_sample_ratio hoặc coverage_ratio nếu có area).

Override theo “continuity/cluster” nếu có (largest_cluster_ratio).

Nếu POLICY_UNRESOLVED: agent không emit defect “violation”; chỉ emit facts + issue POLICY_UNRESOLVED (severity của issue có thể WARN).

Policy block đề xuất (đặt dưới dfm_policy.severity_map)

dfm_policy:
severity_map:
combine_strategy:
method: max_then_overrides
default_issue_severity:
policy_unresolved: WARN
unit_unresolved: ERROR
occ_missing: WARN

thickness:
thin_wall:
deficit_ratio_thresholds: { warn_min: 0.05, error_min: 0.15, fatal_min: 0.30 }
coverage_ratio_thresholds: { warn_min: 0.01, error_min: 0.05, fatal_min: 0.15 }
rules:

- if_deficit_ratio_at_least: 0.30 # (min_wall - t)/min_wall
  severity: FATAL
- if_deficit_ratio_at_least: 0.15
  severity: ERROR
- if_deficit_ratio_at_least: 0.05
  severity: WARN
- else: INFO
  thick_wall:
  excess_ratio_thresholds: { warn_min: 0.10, error_min: 0.25, fatal_min: 0.50 }
  coverage_ratio_thresholds: { warn_min: 0.01, error_min: 0.05, fatal_min: 0.15 }
  rules:
- if_excess_ratio_at_least: 0.50 # (t - max_wall)/max_wall
  severity: FATAL
- if_excess_ratio_at_least: 0.25
  severity: ERROR
- if_excess_ratio_at_least: 0.10
  severity: WARN
- else: INFO
  map_quality:
  miss_rate_thresholds: { warn_min: 0.10, error_min: 0.25, fatal_min: 0.50 }
  ambiguous_hit_rate_thresholds: { warn_min: 0.10, error_min: 0.25, fatal_min: 0.50 }
  sample_count_min: { warn_min: 200, error_min: 80, fatal_min: 30 }
  rules:
- if_miss_rate_at_least: 0.50
  severity: FATAL
  code: THICKNESS_MAP_LOW_COVERAGE
- if_miss_rate_at_least: 0.25
  severity: ERROR
  code: THICKNESS_MAP_LOW_COVERAGE
- if_miss_rate_at_least: 0.10
  severity: WARN
  code: THICKNESS_MAP_LOW_COVERAGE
- if_sample_count_below: 30
  severity: FATAL
  code: THICKNESS_MAP_INSUFFICIENT_SAMPLES
- if_sample_count_below: 80
  severity: ERROR
  code: THICKNESS_MAP_INSUFFICIENT_SAMPLES
- if_sample_count_below: 200
  severity: WARN
  code: THICKNESS_MAP_INSUFFICIENT_SAMPLES

draft:
below_threshold:
deficit_deg_thresholds: { warn_min: 0.5, error_min: 1.5, fatal_min: 3.0 } # deg
coverage_ratio_thresholds: { warn_min: 0.01, error_min: 0.05, fatal_min: 0.15 }
rules:

- if_deficit_deg_at_least: 3.0
  severity: FATAL
- if_deficit_deg_at_least: 1.5
  severity: ERROR
- if_deficit_deg_at_least: 0.5
  severity: WARN
- else: INFO
  reverse_draft:
  reverse_ratio_thresholds: { warn_min: 0.001, error_min: 0.01, fatal_min: 0.05 }
  rules:
- if_reverse_ratio_at_least: 0.05
  severity: FATAL
- if_reverse_ratio_at_least: 0.01
  severity: ERROR
- if_reverse_present: true
  severity: WARN
- else: INFO
  silhouette_crossing:
  face_ratio_thresholds: { warn_min: 0.01, error_min: 0.05, fatal_min: 0.15 }
  rules:
- if_face_ratio_at_least: 0.15
  severity: FATAL
- if_face_ratio_at_least: 0.05
  severity: ERROR
- if_silhouette_present: true
  severity: WARN
- else: INFO
  overrides:
- when:
  reverse_present: true
  below_threshold_deficit_deg_at_least: 1.5
  force_severity: FATAL

undercut:
candidate:
depth_thresholds: { warn_min: 0.01, error_min: 0.10, fatal_min: 0.30 } # depth = -dot_min
coverage_ratio_thresholds: { warn_min: 0.001, error_min: 0.01, fatal_min: 0.05 }
rules:

- if_depth_at_least: 0.30
  severity: FATAL
- if_depth_at_least: 0.10
  severity: ERROR
- if_depth_at_least: 0.01
  severity: WARN
- else: INFO
  continuity_overrides:
- when: { largest_cluster_ratio_at_least: 0.70, coverage_ratio_at_least: 0.05 }
  force_severity: FATAL
- when: { largest_cluster_ratio_at_least: 0.50, coverage_ratio_at_least: 0.01 }
  force_severity: ERROR
  confidence:
  planar_face_ratio_thresholds: { warn_below: 0.60, error_below: 0.30 }
  rules:
- if_planar_face_ratio_below: 0.30
  severity: ERROR
  code: UNDERCUT_LOW_COVERAGE_NONPLANAR
- if_planar_face_ratio_below: 0.60
  severity: WARN
  code: UNDERCUT_LOW_COVERAGE_NONPLANAR

ribratio:

## V1+ (khi có rib_to_wall_ratio thật từ AFR/geometry)

rib_to_wall_ratio:
over_ratio_thresholds: { warn_min: 0.10, error_min: 0.25, fatal_min: 0.50 }
coverage_ratio_thresholds: { warn_min: 0.01, error_min: 0.05, fatal_min: 0.15 }
rules:

## over_ratio = (rib_to_wall - max_allowed)/max_allowed

- if_over_ratio_at_least: 0.50
  severity: FATAL
- if_over_ratio_at_least: 0.25
  severity: ERROR
- if_over_ratio_at_least: 0.10
  severity: WARN
- else: INFO

## V0 proxy (bbox-based) chỉ nên phát LOW_CONFIDENCE, không “kết luận rib ratio”

proxy_mode:
rules:

- if_proxy_used: true
  severity: WARN
  code: RIBRATIO_PROXY_ONLY_LOW_CONFIDENCE

sink:

## V0 proxy (0..1): max_sink_risk_proxy

proxy:
proxy_thresholds: { warn_min: 0.30, error_min: 0.55, fatal_min: 0.75 }
rules:

- if_proxy_at_least: 0.75
  severity: FATAL
- if_proxy_at_least: 0.55
  severity: ERROR
- if_proxy_at_least: 0.30
  severity: WARN
- else: INFO

## V1+ (khi có local thickness / junction metrics)

local_mass:
thick_ratio_thresholds: { warn_min: 0.20, error_min: 0.50, fatal_min: 1.00 } # (t_local - t_nominal)/t_nominal
coverage_ratio_thresholds: { warn_min: 0.01, error_min: 0.05, fatal_min: 0.15 }
rules:

- if_thick_ratio_at_least: 1.00
  severity: FATAL
- if_thick_ratio_at_least: 0.50
  severity: ERROR
- if_thick_ratio_at_least: 0.20
  severity: WARN
- else: INFO

weldline:

## V0 proxy: weld_risk_proxy = edge_count/face_count

proxy:
over_ratio_thresholds: { warn_min: 0.10, error_min: 0.25, fatal_min: 0.50 } # (proxy - max_allowed)/max_allowed
min_face_count_for_valid_proxy: 20 # tránh proxy “ảo” khi face quá ít
rules:

- if_face_count_below: 20
  severity: WARN
  code: WELDLINE_PROXY_LOW_VALIDITY_FACECOUNT
- if_over_ratio_at_least: 0.50
  severity: FATAL
- if_over_ratio_at_least: 0.25
  severity: ERROR
- if_over_ratio_at_least: 0.10
  severity: WARN
- else: INFO

Ghi chú quan trọng để “complete set” này chạy đúng mà không làm hệ thống bị “IT mạnh – DFM yếu”

ribratio: nếu vẫn là proxy bbox, chỉ được phép phát “LOW_CONFIDENCE”, không nên phát “rib ratio vượt ngưỡng” (vì metric không cùng nghĩa). Phần policy ở trên đã tách rõ proxy_mode vs rib_to_wall_ratio thật.

sink: proxy V0 (bbox anisotropy) có thể dùng để “đèn vàng” sớm; khi thickness_agent V1 đã có local thickness thì sink nên chuyển dần sang local_mass rules (t_local / t_nominal).

weldline: edge/face proxy chỉ đáng tin khi face_count đủ lớn; nếu face_count thấp, severity của “proxy risk” phải hạ xuống và thay bằng “low validity”.

## 3.5 Default Policy Example (JSON)

Source: S6

{

"dfm_policy": {

"dfm_policy_version": "DFM_POLICY_v1",

"units": { "length_unit": "mm", "angle_unit": "deg" },

"common": {

"silhouette_eps_dot": 0.015,

"sampling_budget": { "max_faces_to_report": 50, "max_worst_regions": 10 }

},

"thresholds": {

"draft": {

"min_draft_deg_default": 1.0,

"min_draft_deg_by_surface_class": {

"polish": 1.0,

"light_texture": 2.0,

"heavy_texture": 3.0

},

"reverse_dot_threshold": 0.0

},

"thickness": {

"min_wall_mm": 1.2,

"max_wall_mm": 3.5,

"probe": {

"self_hit_epsilon_mm": 0.05,

"max_probe_distance_mm": null,

"max_intersections_to_consider": 8

}

}

}

}

}

## 4. Open Questions / [CHƯA XÁC MINH]

Version alignment: Canonical code là v1 nhưng có nguồn S1 là policy master spec v2.0 (FROZEN). [CHƯA XÁC MINH] bạn muốn giữ WS_AGENTS-ALL-POL ở v1 lâu dài hay bump lên v2 để đồng bộ với DFM_POLICY_MASTER_SPEC_v2_0?

effective_date_utc / owner trong catalog keys (S5, S3) đang để [CHƯA XÁC MINH]. Nếu cần audit chặt, cần chốt các field này.

Severity map (S4) đang status=DRAFT. [CHƯA XÁC MINH] khi nào coi severity_map là REQUIRED hay OPTIONAL trong dfm_policy?

Default policy JSON (S6) hiện mới bao phủ draft/thickness; [CHƯA XÁC MINH] bạn có muốn coi S6 là ‘default baseline’ (cần đủ keys cho mọi agent) hay chỉ là ví dụ tối thiểu?

## 5. Change Log (Editorial)

Đổi tên và gom tài liệu theo chuẩn WS/canonical code; không đổi semantics.

Chuẩn hoá cấu trúc: thêm Document Control Block, Mapping Table, Open Questions, Change Log.

Giữ nội dung theo verbatim extracts từ nguồn; chỉ lọc bỏ các dòng tham chiếu bị cắt cụt dạng “TÀI LIỆU ĐẶC TẢ HỆ THỐNG …” để giảm nhiễu (không thay đổi ý nghĩa).

## Appendix A — PHẦN 1 (Original) DFM_POLICY_KEYS_MINIMUM_v1

Source: S2

TÀI LIỆU HOÀN CHỈNH — BƯỚC B
Chốt chuẩn “DFM policy keys” và “region_ref convention” để mọi agent sau không bị lệch

Tài liệu này là “spec bổ trợ” (addendum-level). Mục tiêu là khóa 2 điểm nền: (1) catalog keys cho policy-as-config của DFM, (2) chuẩn định vị vùng lỗi (region_ref) để defects/results có thể truy vết mà không tạo shadow data. Các ràng buộc này bám theo Master Spec + Governance invariants (SSOT, no shadow data, fail-safe, report always-on).

────────────────────────────────────────────────────────────
PHẦN 1 — DFM_POLICY_KEYS_MINIMUM_v1

THÔNG TIN PHIÊN BẢN (DOCUMENT METADATA)

doc_id: TOOLING_DFM_ADVISOR_DFM_POLICY_KEYS_MINIMUM_v1
status: APPROVED (Owner-locked)
effective_date_utc: [CHƯA XÁC MINH]
owner: [CHƯA XÁC MINH]
scope: catalog keys + semantics + units + required/optional + fail-safe behavior
related_specs: TOOLING_DFM_ADVISOR_MASTER_SPEC_v1, TOOLING_DFM_ADVISOR_GOVERNANCE_MASTER_SPEC_v1

MỤC TIÊU VÀ BẤT BIẾN (GOALS & INVARIANTS)

2.1. Policy-as-config, versioned, tamper-evident

Tất cả threshold/logic mapping phải nằm trong config (versioned) để kiểm toán được.

Agent không “tự đoán” ngưỡng nếu thiếu; phải trả issue POLICY_UNRESOLVED và chỉ xuất “facts” (metrics) hoặc degrade phù hợp.

2.2. Đơn vị và quy ước

Length: mm (hoặc inch) nhưng phải thống nhất toàn hệ. Nếu dùng inch, toàn bộ ngưỡng length cũng phải inch.

Angle: degrees.

Ratio: vô thứ nguyên.

Nếu unit_hint/units chưa chốt hoặc mâu thuẫn: issue UNIT_UNRESOLVED và agent không được so ngưỡng tuyệt đối theo length. [CHƯA XÁC MINH: cơ chế unit normalization end-to-end]

2.3. Không chôn “severity rules” trong code

Mapping severity theo mức vi phạm nên nằm trong policy (để minh bạch và thay đổi có audit).

VỊ TRÍ LƯU POLICY VÀ THỨ TỰ ƯU TIÊN (STORAGE & PRECEDENCE)

3.1. Vị trí khuyến nghị (giữ đúng tinh thần Governance Master Spec)

Canonical: config/governance_policy.json (versioned)

Override local (gitignored): config/policy_override.local.json (chỉ dùng dev, có audit khi áp dụng)

3.2. Precedence
policy_override.local.json (nếu tồn tại) > governance_policy.json

CẤU TRÚC JSON ĐỀ XUẤT (SCHEMA SKETCH)

Top-level (trích phần liên quan DFM, gợi ý đặt dưới governance_policy.json):

dfm_policy:

dfm_policy_version: "DFM_POLICY_v1"

units:

length_unit: "mm" | "inch"

angle_unit: "deg"

common:

silhouette_eps_dot: float (0..1, dùng cho draft silhouette; nếu thiếu -> default null và agent chỉ dùng sign-change)

sampling_budget:

max_faces_to_report: int

max_worst_regions: int

thresholds:

draft: {…}

thickness: {…}

undercut: {…}

ribratio: {…}

sink: {…}

weldline: {…}

severity_map (optional, nhưng khuyến nghị có): {…}

CATALOG KEYS TỐI THIỂU THEO AGENT

5.1. draft_agent (V1 baseline: curved + silhouette/reverse + worst regions)
Required:

thresholds.draft.min_draft_deg_default: float (deg)
Optional (khuyến nghị):

thresholds.draft.min_draft_deg_by_surface_class: object

polish: float

light_texture: float

heavy_texture: float

common.silhouette_eps_dot: float (ví dụ 0.01… nhưng giá trị cụ thể do Owner chốt) [CHƯA XÁC MINH]

thresholds.draft.reverse_dot_threshold: float (mặc định 0.0 nếu không khai báo; reverse khi dot < 0)

Fail-safe nếu thiếu:

thiếu min_draft_deg_default: issue POLICY_UNRESOLVED; vẫn tính draft metrics + reverse/silhouette facts, không kết luận “below threshold”.

5.2. thickness_agent (để chuẩn bị V1 local thickness sau này)
Required:

thresholds.thickness.min_wall: float (length_unit)
Optional:

thresholds.thickness.max_wall: float (length_unit)

thresholds.thickness.nominal_wall_strategy: "p50_local" | "user_specified" | "p50_global" [CHƯA XÁC MINH]

thresholds.thickness.allowed_variation_ratio: float (dimensionless) [CHƯA XÁC MINH]

Fail-safe nếu thiếu:

thiếu min_wall: POLICY_UNRESOLVED; chỉ xuất thickness distribution facts (min/pXX/max) nếu vẫn tính được.

5.3. undercut_agent
Required:

thresholds.undercut.reverse_dot_threshold: float (mặc định 0.0; undercut candidate khi dot < threshold)
Optional:

thresholds.undercut.planar_only: bool (để backward-compat nếu còn V0)

thresholds.undercut.nonplanar_sampling_eps: float [CHƯA XÁC MINH]

Fail-safe nếu thiếu:

nếu không có reverse_dot_threshold: mặc định 0.0 (đây là “định nghĩa vật lý” của reverse theo dot-product, không phải threshold DFM). Nếu Owner không muốn default này, phải chốt explicit.

5.4. ribratio_agent (chuẩn bị để thoát khỏi bbox-proxy về sau)
Required (DFM-thực chiến):

thresholds.ribratio.max_rib_to_wall_ratio: float (dimensionless)
Optional:

thresholds.ribratio.preferred_rib_to_wall_ratio: float

thresholds.ribratio.min_root_radius: float (length_unit) [CHƯA XÁC MINH]

thresholds.ribratio.nominal_wall_strategy: giống thickness_agent (nếu dùng chung) [CHƯA XÁC MINH]

Fail-safe nếu thiếu:

POLICY_UNRESOLVED; nếu agent đang ở proxy V0 (bbox ratio), chỉ xuất proxy facts, không “giả” rib_to_wall.

5.5. sink_agent
Required:

thresholds.sink.max_sink_risk_proxy: float (0..1) (nếu vẫn dùng proxy V0 như bbox anisotropy)

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

Optional (để lên V1 local mass/thickness):

thresholds.sink.max_boss_to_wall_ratio: float [CHƯA XÁC MINH]

thresholds.sink.max_local_thickness_ratio: float [CHƯA XÁC MINH]

Fail-safe nếu thiếu:

POLICY_UNRESOLVED; vẫn có thể xuất proxy metric nhưng không emit “over threshold”.

5.6. weldline_agent
Required:

thresholds.weldline.max_weld_risk_proxy: float (dimensionless; nếu vẫn dùng edge_count/face_count proxy V0)

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

Optional:

thresholds.weldline.min_face_count_for_valid_proxy: int [CHƯA XÁC MINH]

Fail-safe nếu thiếu:

POLICY_UNRESOLVED; vẫn compute proxy facts (edge/face counts) và cảnh báo “threshold missing”.

SEVERITY MAP TỐI THIỂU (khuyến nghị chốt sớm để report nhất quán)

Nếu bạn muốn reporter summary ổn định (INFO/WARN/ERROR/FATAL), nên có severity_map trong dfm_policy:

severity_map.draft:

reverse_draft: "ERROR"

below_threshold: "WARN" | "ERROR" theo mức thiếu [CHƯA XÁC MINH: rule]

silhouette_crossing: "WARN" | "ERROR" [CHƯA XÁC MINH]
Tương tự cho các agent khác. Nếu chưa chốt, agent phải gắn severity ở mức “WARN” mặc định và thêm note “severity_policy_unresolved”. [CHƯA XÁC MINH: bạn có muốn default này không]

VÍ DỤ NGẮN “FILLED EXAMPLE” (không gán giá trị ngưỡng nếu Owner chưa chốt)

dfm_policy:
dfm_policy_version: DFM_POLICY_v1
units:
length_unit: mm
angle_unit: deg
common:
silhouette_eps_dot: [CHƯA XÁC MINH]
sampling_budget:
max_faces_to_report: 50
max_worst_regions: 10
thresholds:
draft:
min_draft_deg_default: [CHƯA XÁC MINH]
min_draft_deg_by_surface_class:
polish: [CHƯA XÁC MINH]
light_texture: [CHƯA XÁC MINH]
heavy_texture: [CHƯA XÁC MINH]
reverse_dot_threshold: 0.0
thickness:
min_wall: [CHƯA XÁC MINH]
max_wall: [CHƯA XÁC MINH]
undercut:
reverse_dot_threshold: 0.0
ribratio:
max_rib_to_wall_ratio: [CHƯA XÁC MINH]
sink:
max_sink_risk_proxy: [CHƯA XÁC MINH]
weldline:
max_weld_risk_proxy: [CHƯA XÁC MINH]

## 6. Appendix

Cross references (as stated in sources): ANALYSIS_RESULT_CONTRACT_v2_0; RESULT_ENVELOPE_API_v1; REGION_REF_CONVENTION_v1; GOLDEN_REGRESSION_STRATEGY_v2_0.

Storage & precedence (as stated): config/governance_policy.json -> dfm_policy; override: config/policy_override.local.json (gitignored), precedence override > canonical.
