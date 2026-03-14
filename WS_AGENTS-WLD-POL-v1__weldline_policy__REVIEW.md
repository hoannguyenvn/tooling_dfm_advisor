WS_AGENTS-WLD-POL-v1 (REVIEW)

# 0. Document Control Block

|                                  |                                                                                       |
| -------------------------------- | ------------------------------------------------------------------------------------- |
| Canonical Code                   | WS_AGENTS-WLD-POL-v1                                                                  |
| Status                           | REVIEW                                                                                |
| Run ID (UTC)                     | 20260304T122755Z                                                                      |
| Created date (UTC)               | 2026-03-04T12:27:55Z                                                                  |
| WS / Module / Doc Type / Version | WS_AGENTS / WLD / POL / v1                                                            |
| Purpose                          | DFM policy for weldline_agent; document governance only; no runtime/pipeline changes. |

## 1. Scope

Source: S1, S2, S3, S4, S5

Mục tiêu: ánh xạ weldline susceptibility (proxy) thành severity INFO/WARN/ERROR/FATAL dựa trên topo proxy (edge_count/face_count) và validity rules (min face count).

Lưu ý bất biến: không CAE và không gate context => policy phải ưu tiên minh bạch confidence/validity, tránh kết luận sai về vị trí weldline.

Non-goals: không thay đổi thuật toán agent; không thiết kế lại pipeline; không thay đổi contract semantics; không bổ sung quy tắc mới ngoài nguồn. Nếu cần nội dung mới để hoàn thiện template sẽ ghi [CHƯA XÁC MINH].

## 2. Included Sources & Mapping Table

|           |                                                                        |                                                                   |                         |                                                      |                                  |
| --------- | ---------------------------------------------------------------------- | ----------------------------------------------------------------- | ----------------------- | ---------------------------------------------------- | -------------------------------- |
| source_id | filename                                                               | original_title                                                    | original_version/status | sections_used                                        | notes                            |
| S1        | TÀI LIỆU — DFM_POLICY_SEVERITY_MAP_COMPLETE_SET_v1.docx                | TÀI LIỆU — DFM_POLICY_SEVERITY_MAP_COMPLETE_SET_v1                | [CHƯA XÁC MINH]         | severity map block (weldline + combine strategy)     | primary severity rules           |
| S2        | PHẦN 1 — DFM_POLICY_KEYS_MINIMUM_v1.docx                               | PHẦN 1 — DFM_POLICY_KEYS_MINIMUM_v1                               | [CHƯA XÁC MINH]         | keys + fail-safe (section 5.6)                       | required/optional keys semantics |
| S3        | DFM_POLICY_KEYS_MINIMUM_v1 — BẢN HỢP NHẤT (UPDATED, CONSOLIDATED).docx | DFM_POLICY_KEYS_MINIMUM_v1 — BẢN HỢP NHẤT (UPDATED, CONSOLIDATED) | [CHƯA XÁC MINH]         | JSON skeleton thresholds subtree                     | starter defaults skeleton        |
| S4        | TÀI LIỆU TRIỂN KHAI ADN AMBR LDR — MODULE weldline_agent .docx         | TÀI LIỆU TRIỂN KHAI ADN AMBR LDR — MODULE weldline_agent          | [CHƯA XÁC MINH]         | defect advisory template + validity/region_ref notes | defect schema + codes            |
| S5        | GOVERNANCE_POLICY_SCHEMA_v1.docx                                       | GOVERNANCE_POLICY_SCHEMA_v1                                       | [CHƯA XÁC MINH]         | schema placement of thresholds/severity_map          | anchor to GOV policy schema      |

## 3. Consolidated Policy Content

## 3.1 Document header + mục tiêu

Source: S1

TÀI LIỆU — DFM_POLICY_SEVERITY_MAP_COMPLETE_SET_v1
doc_id: TOOLING_DFM_ADVISOR_DFM_POLICY_SEVERITY_MAP_COMPLETE_SET_v1
status: DRAFT
effective_date_utc: [CHƯA XÁC MINH]
units: length_unit=mm, angle_unit=deg

## 3.2 Policy keys (required/optional) + fail-safe

Source: S2

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

## 3.3 Thresholds subtree (starter defaults skeleton)

Source: S3

{
"dfm_policy": {
"thresholds": {
"weldline": {
"max_weld_risk_proxy": null,
"min_face_count_for_valid_proxy": 20
}
}
}
}

## 3.4 Severity map rules (agent-specific)

Source: S1

dfm_policy:
severity_map:
combine_strategy:
method: max_then_overrides
default_issue_severity:
policy_unresolved: WARN
unit_unresolved: ERROR
occ_missing: WARN

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

## 3.5 Defect advisory template (for consistency with reporter)

Source: S4

Ghi chú region_ref:

Với topo proxy thuần global, defect sẽ không có “vùng chắc chắn”. Nếu buộc phải có region_ref, chỉ dùng kind="PATCH" với id_scheme="AGL_FACE_INDEX_V1" cho “top suspect faces” khi có trigger_hints đáng tin; nếu không có thì để defect là “GLOBAL” và region_ref null. [CHƯA XÁC MINH: hệ thống có cho phép defect không có region_ref không; Master Spec mô tả defect có region_ref nhưng thực tế weldline proxy không định vị được].

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

────────────────────────────────────────────────────────────
F) Defect advisory template — weldline_agent

WELDLINE_DEFECT_TEMPLATE_v1:

id

run_id

agent_source: "weldline_agent"

rule_version

geometry_version_id

timestamp_utc

severity: INFO|WARN|ERROR|FATAL (theo dfm_policy.severity_map.weldline.proxy)

defect_type:

"WELDLINE_PROXY_OVER_THRESHOLD"

"WELDLINE_PROXY_LOW_VALIDITY_FACECOUNT"

"WELDLINE_NOT_EXECUTED_OCC_MISSING"

region_ref: null | REGION_REF_SCHEMA_v1 (chỉ set khi có trigger_hints định vị đáng tin) [CHƯA XÁC MINH]

analysis_value:

edge_count: int

face_count: int

weld_risk_proxy: float|null

validity: {is_valid_proxy: bool, reason: string}

method: "TOPO_PROXY_V1_VALIDITY_AWARE"

threshold_applied:

max_weld_risk_proxy: float|null

advisory:

machine_readable:

summary_code: "REVIEW_GATE_STRATEGY" | "ADD_OVERFLOW" | "ADD_VENT" | "REVIEW_FLOW_SPLIT_FEATURES"

recommended_actions: list[str]

rationale_tags: ["TOPO_COMPLEXITY", "PROXY_ONLY", "LOW_VALIDITY"]

human_readable:

title

why_it_matters (đường hàn làm yếu cơ tính, ảnh hưởng mỹ quan)

suggested_fix (định hướng: xem gate/overflow/vent; tránh chia dòng không cần)

verification (nếu critical: khuyến nghị CAE hoặc thử khuôn/DOE)

confidence_score

assets_ephemeral_ref (token only)

delivery_scrubbed

## 4. Open Questions / [CHƯA XÁC MINH]

|                 |                                                                                                                                                                                                                                                      |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| source_id       | open_question_excerpt                                                                                                                                                                                                                                |
| S1              | TÀI LIỆU — DFM_POLICY_SEVERITY_MAP_COMPLETE_SET_v1 doc_id: TOOLING_DFM_ADVISOR_DFM_POLICY_SEVERITY_MAP_COMPLETE_SET_v1 status: DRAFT effective_date_utc: [CHƯA XÁC MINH] units: length_unit=mm, angle_unit=deg                                       |
| S2              | doc_id: TOOLING_DFM_ADVISOR_DFM_POLICY_KEYS_MINIMUM_v1 status: APPROVED (Owner-locked) effective_date_utc: [CHƯA XÁC MINH] owner: [CHƯA XÁC MINH] scope: catalog keys + semantics + units + required/optional + fail-safe behavior related_specs...  |
| S2              | Nếu unit_hint/units chưa chốt hoặc mâu thuẫn: issue UNIT_UNRESOLVED và agent không được so ngưỡng tuyệt đối theo length. [CHƯA XÁC MINH: cơ chế unit normalization end-to-end]                                                                       |
| S2              | common.silhouette_eps_dot: float (ví dụ 0.01… nhưng giá trị cụ thể do Owner chốt) [CHƯA XÁC MINH]                                                                                                                                                    |
| S2              | thresholds.thickness.nominal_wall_strategy: "p50_local"                                                                                                                                                                                              |
| S2              | thresholds.thickness.allowed_variation_ratio: float (dimensionless) [CHƯA XÁC MINH]                                                                                                                                                                  |
| S2              | thresholds.undercut.nonplanar_sampling_eps: float [CHƯA XÁC MINH]                                                                                                                                                                                    |
| S2              | thresholds.ribratio.min_root_radius: float (length_unit) [CHƯA XÁC MINH]                                                                                                                                                                             |
| S2              | thresholds.ribratio.nominal_wall_strategy: giống thickness_agent (nếu dùng chung) [CHƯA XÁC MINH]                                                                                                                                                    |
| S2              | thresholds.sink.max_boss_to_wall_ratio: float [CHƯA XÁC MINH]                                                                                                                                                                                        |
| S2              | thresholds.sink.max_local_thickness_ratio: float [CHƯA XÁC MINH]                                                                                                                                                                                     |
| S2              | thresholds.weldline.min_face_count_for_valid_proxy: int [CHƯA XÁC MINH]                                                                                                                                                                              |
| S2              | below_threshold: "WARN"                                                                                                                                                                                                                              |
| S2              | silhouette_crossing: "WARN"                                                                                                                                                                                                                          |
| S2              | dfm_policy: dfm_policy_version: DFM_POLICY_v1 units: length_unit: mm angle_unit: deg common: silhouette_eps_dot: [CHƯA XÁC MINH] sampling_budget: max_faces_to_report: 50 max_worst_regions: 10 thresholds: draft: min_draft_deg_default: \[CHƯA ... |
| S3              | DFM_POLICY_KEYS_MINIMUM_v1 — BẢN HỢP NHẤT (UPDATED, CONSOLIDATED) (doc_id: TOOLING_DFM_ADVISOR_DFM_POLICY_KEYS_MINIMUM_v1) status: APPROVED (Owner-locked) effective_date_utc: [CHƯA XÁC MINH] scope: Catalog key policy tối thiểu (units + comm...  |
| S3              | dfm_policy.common.max_runtime_seconds (OPTIONAL): int                                                                                                                                                                                                |
| S3              | Fail-safe: nếu thiếu keys -> dùng default + issue VALIDATION_DEFAULTS_USED. Nếu checker FULL không khả dụng -> degrade về FAST + issue SELF_INTERSECTION_FULL_NOT_AVAILABLE (không “PASS giả”). \[CHƯA XÁC MINH: checker availability khi impleme... |
| S3              | min_cylinder_face_area_mm2 (OPTIONAL, default 5.0): float (mm^2) [CHƯA XÁC MINH: có dùng area trong implement không; nếu không dùng thì bỏ key này]                                                                                                  |
| S3              | Severity map keys (không bắt buộc trong MINIMUM, nhưng khuyến nghị dùng chung) Nếu có dfm_policy.severity_map, các agent sẽ map severity nhất quán theo config (đã xây bộ “complete set”). Nếu không có, agent có thể set severity conservative ...  |
| S4              | THÔNG TIN PHIÊN BẢN (DOCUMENT METADATA) doc_id: TOOLING_DFM_ADVISOR_WELDLINE_AGENT_DFM_V1_RESEARCH_v1 status: DRAFT effective_date_utc: [CHƯA XÁC MINH] owner: [CHƯA XÁC MINH] units_locked: length_unit=mm (không trực tiếp dùng), ratio vô thứ...  |
| S4              | Bổ sung “trigger hints” (không CAE): đếm/nhận diện proxy các yếu tố thường gây chia dòng như “nhiều lỗ xuyên / nhiều đảo topo / nhiều nhánh” ở mức đếm (không định vị chính xác nếu chưa có feature tagging). \[CHƯA XÁC MINH: trigger set tối ưu... |
| S4              | geometry_context.brep_shape (TopoDS_Shape) [CHƯA XÁC MINH: payload key path]                                                                                                                                                                         |
| S4              | hole_like_feature_proxy_count: int [CHƯA XÁC MINH: cách đo]                                                                                                                                                                                          |
| S4              | multi_component_proxy: bool [CHƯA XÁC MINH: nếu shape có nhiều solid]                                                                                                                                                                                |
| S4              | [CHƯA XÁC MINH] nếu có thể, đếm “hole-like features” bằng proxy (ví dụ số lượng vòng biên tròn/cylindrical faces) để tăng cảnh giác chia dòng. Nếu không có thì bỏ qua phần này, không suy diễn. Step 7: Evaluate vs threshold:                      |
| S4              | Gate context (ngay cả “gate side” coarse) => tăng giá trị định vị vùng nghi ngờ weldline. [CHƯA XÁC MINH: hệ thống có muốn thu context gate không]                                                                                                   |
| S4              | AFR feature graph (holes/boss/ribs) => trigger_hints định vị theo region_ref chính xác hơn. [CHƯA XÁC MINH]                                                                                                                                          |
| S4              | Mapping DGL integrity tốt => có thể suy ra “flow split corridors” theo mesh graph ở mức heuristic (không solver). [CHƯA XÁC MINH]                                                                                                                    |
| S4              | ldr_id: LDR_pythonocc_core_topology_explorer_for_weldline_agent_v1 status: DRAFT library_name: pythonocc-core (OCC.Core.TopExp, OCC.Core.TopAbs) library_version: [CHƯA XÁC MINH] purpose: topo enumeration (edge/face counts) for WeldRiskProxy...  |
| S4              | rule_version [CHƯA XÁC MINH]                                                                                                                                                                                                                         |
| S4              | receipts_ref: [CHƯA XÁC MINH]                                                                                                                                                                                                                        |
| S4              | Với topo proxy thuần global, defect sẽ không có “vùng chắc chắn”. Nếu buộc phải có region_ref, chỉ dùng kind="PATCH" với id_scheme="AGL_FACE_INDEX_V1" cho “top suspect faces” khi có trigger_hints đáng tin; nếu không có thì để defect là “GLO...  |
| S4              | region_ref: null                                                                                                                                                                                                                                     |
| [CHƯA XÁC MINH] | Defect region_ref cho weldline proxy: nguồn nêu khả năng region_ref=null hoặc GLOBAL; cần Owner xác nhận DEFECT_OBJECT_API_v1 có cho phép region_ref null trong trường hợp global proxy hay không.                                                   |

## 5. Change Log (Editorial)

Thay đổi biên tập/tái cấu trúc để chuẩn hoá theo Document Governance Layer; không đổi semantics.

- Tạo mới canonical POL theo WS, thêm Document Control Block + Mapping Table (Run ID: 20260304T122755Z).

- Trích nguyên văn các đoạn policy keys, severity rules, defect template từ nguồn; sắp xếp lại theo template bắt buộc.

- Nếu có nội dung cần điền nhưng nguồn không có: ghi [CHƯA XÁC MINH].

## 6. Appendix

Cross references (canonical codes): WS_AGENTS-ALL-POL-v1 (global DFM policy); WS_MODEL-RGN-POL-v1 (REGION_REF_CONVENTION_v1).

Naming conventions: module snake_case; contracts/envelopes UPPER_CASE versioned.
