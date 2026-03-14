WS_AGENTS-SNK-POL-v1 (REVIEW)

# 0. Document Control Block

|                                  |                                                                                   |
| -------------------------------- | --------------------------------------------------------------------------------- |
| Canonical Code                   | WS_AGENTS-SNK-POL-v1                                                              |
| Status                           | REVIEW                                                                            |
| Run ID (UTC)                     | 20260304T122755Z                                                                  |
| Created date (UTC)               | 2026-03-04T12:27:55Z                                                              |
| WS / Module / Doc Type / Version | WS_AGENTS / SNK / POL / v1                                                        |
| Purpose                          | DFM policy for sink_agent; document governance only; no runtime/pipeline changes. |

## 1. Scope

Source: S1, S2, S3, S4, S5

Mục tiêu: ánh xạ rủi ro sink/void/hotspot do khối dày cục bộ thành severity INFO/WARN/ERROR/FATAL dựa trên local thickness ratio (khi có) và proxy lane (khi chưa có thickness map).

Hai lane policy: proxy lane (V0) và local_mass lane (V1).

Non-goals: không thay đổi thuật toán agent; không thiết kế lại pipeline; không thay đổi contract semantics; không bổ sung quy tắc mới ngoài nguồn. Nếu cần nội dung mới để hoàn thiện template sẽ ghi [CHƯA XÁC MINH].

## 2. Included Sources & Mapping Table

|           |                                                                        |                                                                   |                         |                                              |                                  |
| --------- | ---------------------------------------------------------------------- | ----------------------------------------------------------------- | ----------------------- | -------------------------------------------- | -------------------------------- |
| source_id | filename                                                               | original_title                                                    | original_version/status | sections_used                                | notes                            |
| S1        | TÀI LIỆU — DFM_POLICY_SEVERITY_MAP_COMPLETE_SET_v1.docx                | TÀI LIỆU — DFM_POLICY_SEVERITY_MAP_COMPLETE_SET_v1                | [CHƯA XÁC MINH]         | severity map block (sink + combine strategy) | primary severity rules           |
| S2        | PHẦN 1 — DFM_POLICY_KEYS_MINIMUM_v1.docx                               | PHẦN 1 — DFM_POLICY_KEYS_MINIMUM_v1                               | [CHƯA XÁC MINH]         | keys + fail-safe (section 5.5)               | required/optional keys semantics |
| S3        | DFM_POLICY_KEYS_MINIMUM_v1 — BẢN HỢP NHẤT (UPDATED, CONSOLIDATED).docx | DFM_POLICY_KEYS_MINIMUM_v1 — BẢN HỢP NHẤT (UPDATED, CONSOLIDATED) | [CHƯA XÁC MINH]         | JSON skeleton thresholds subtree             | starter defaults skeleton        |
| S4        | TÀI LIỆU TRIỂN KHAI ADN AMBR LDR — MODULE sink_agent (DFM-first).docx  | TÀI LIỆU TRIỂN KHAI ADN AMBR LDR — MODULE sink_agent (DFM-first)  | [CHƯA XÁC MINH]         | defect advisory template + policy_echo keys  | defect schema + codes            |
| S5        | GOVERNANCE_POLICY_SCHEMA_v1.docx                                       | GOVERNANCE_POLICY_SCHEMA_v1                                       | [CHƯA XÁC MINH]         | schema placement of thresholds/severity_map  | anchor to GOV policy schema      |

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

5.5. sink_agent
Required:

thresholds.sink.max_sink_risk_proxy: float (0..1) (nếu vẫn dùng proxy V0 như bbox anisotropy)

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

Optional (để lên V1 local mass/thickness):

thresholds.sink.max_boss_to_wall_ratio: float [CHƯA XÁC MINH]

thresholds.sink.max_local_thickness_ratio: float [CHƯA XÁC MINH]

Fail-safe nếu thiếu:

POLICY_UNRESOLVED; vẫn có thể xuất proxy metric nhưng không emit “over threshold”.

## 3.3 Thresholds subtree (starter defaults skeleton)

Source: S3

{
"dfm_policy": {
"thresholds": {
"sink": {
"max_sink_risk_proxy": null,
"local_mass": {
"max_local_thickness_ratio": null,
"max_boss_to_wall_ratio": null
}
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

## 3.5 Defect advisory template (for consistency with reporter)

Source: S4

────────────────────────────────────────────────────────────
F) Defect advisory template — sink_agent

SINK_DEFECT_TEMPLATE_v1:

id

run_id

agent_source: "sink_agent"

rule_version

geometry_version_id

timestamp_utc

severity: INFO|WARN|ERROR|FATAL (từ severity_map.sink.\* trong policy)

defect_type:

"SINK_LOCAL_MASS_OVER_THRESHOLD"

"SINK_PROXY_HIGH_RISK"

"SINK_LOW_CONFIDENCE_THICKNESS_COVERAGE"

region_ref: REGION_REF_SCHEMA_v1 (AGL FACE/PATCH)

analysis_value:

t_local_mm: float|null

nominal_wall_mm: float|null

thick_ratio: float|null

proxy_value: float|null (nếu proxy path)

method: "LOCAL_MASS_FROM_THICKNESS"|"BBOX_PROXY_V0"

threshold_applied:

max_local_thickness_ratio: float|null

max_sink_risk_proxy: float|null

advisory:

machine_readable:

summary_code: "CORE_OUT" | "REDUCE_MASS" | "IMPROVE_TRANSITION" | "REVIEW_THICKNESS_MAP"

recommended_actions: [ ... ]

rationale_tags: ["THICK_JUNCTION", "HOTSPOT_RISK", "PROXY_ONLY", ...]

human_readable:

title

why_it_matters (sink/void/cycle time)

suggested_fix (core-out, hollow boss, ribbing, smooth transition)

verification (kiểm thickness map, kiểm junction thickness ratio, xem lại cosmetic requirement)

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
| S4              | THÔNG TIN PHIÊN BẢN (DOCUMENT METADATA) doc_id: TOOLING_DFM_ADVISOR_SINK_AGENT_DFM_V1_RESEARCH_v1 status: DRAFT effective_date_utc: [CHƯA XÁC MINH] owner: [CHƯA XÁC MINH] units_locked: length_unit = mm                                            |
| S4              | Ghi chú versioning: nếu bạn muốn “đúng chặt” theo quy ước envelope \_vN, thì đổi sang SINK_ANALYSIS_RESULT_v1 khi chuyển từ bbox proxy sang local-thickness logic. [CHƯA XÁC MINH: rule versioning envelope trong repo hiện đang enforce thế nào]    |
| S4              | thickness_result: output của thickness_agent (thickness_distribution_mm + worst_regions + sampling quality). [CHƯA XÁC MINH: wiring field name]                                                                                                      |
| S4              | “nominal_wall_strategy”: dùng p50 thickness làm nominal (khuyến nghị baseline), hoặc user_specified. [CHƯA XÁC MINH: bạn muốn chốt strategy ở policy hay lấy mặc định p50]                                                                           |
| S4              | Nếu policy có user_specified_nominal: override. [CHƯA XÁC MINH] Step 4: Compute thick_ratio for candidate regions:                                                                                                                                   |
| S4              | AFR tagging boss/rib/junction + nominal wall mapping theo feature (không chỉ global p50). [CHƯA XÁC MINH]                                                                                                                                            |
| S4              | Transition analysis: gradient thickness/step change và “thick island detection” bằng clustering/area. [CHƯA XÁC MINH]                                                                                                                                |
| S4              | Process/material context: packing sensitivity, cosmetic class để severity mapping chuẩn hơn. [CHƯA XÁC MINH]                                                                                                                                         |
| S4              | Bound algorithm: ADN_sink_agent_local_mass_sink_risk_from_thickness_map_v1 Bound contracts/inputs: depends on thickness_agent output (THICKNESS_ANALYSIS_RESULT\_\*) [CHƯA XÁC MINH: exact field wiring]                                             |
| S4              | rule_version [CHƯA XÁC MINH]                                                                                                                                                                                                                         |
| S4              | receipts_ref: [CHƯA XÁC MINH]                                                                                                                                                                                                                        |
| [CHƯA XÁC MINH] | Giá trị threshold 'max_sink_risk_proxy' và 'local_mass.max_local_thickness_ratio' trong policy hiện đang null trong skeleton; cần Owner chốt theo material/process để agent có thể emit over-threshold thay vì facts-only.                           |

## 5. Change Log (Editorial)

Thay đổi biên tập/tái cấu trúc để chuẩn hoá theo Document Governance Layer; không đổi semantics.

- Tạo mới canonical POL theo WS, thêm Document Control Block + Mapping Table (Run ID: 20260304T122755Z).

- Trích nguyên văn các đoạn policy keys, severity rules, defect template từ nguồn; sắp xếp lại theo template bắt buộc.

- Nếu có nội dung cần điền nhưng nguồn không có: ghi [CHƯA XÁC MINH].

## 6. Appendix

Cross references (canonical codes): WS_AGENTS-ALL-POL-v1 (global DFM policy); WS_MODEL-RGN-POL-v1 (REGION_REF_CONVENTION_v1).

Naming conventions: module snake_case; contracts/envelopes UPPER_CASE versioned.
