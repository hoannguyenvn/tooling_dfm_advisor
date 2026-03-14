WS_AGENTS-RIB-POL-v1 (REVIEW)

# 0. Document Control Block

|                                  |                                                                                       |
| -------------------------------- | ------------------------------------------------------------------------------------- |
| Canonical Code                   | WS_AGENTS-RIB-POL-v1                                                                  |
| Status                           | REVIEW                                                                                |
| Run ID (UTC)                     | 20260304T122755Z                                                                      |
| Created date (UTC)               | 2026-03-04T12:27:55Z                                                                  |
| WS / Module / Doc Type / Version | WS_AGENTS / RIB / POL / v1                                                            |
| Purpose                          | DFM policy for ribratio_agent; document governance only; no runtime/pipeline changes. |

## 1. Scope

Source: S1, S2, S3, S4, S5

Mục tiêu: ánh xạ rủi ro thiết kế gân (rib) thành severity INFO/WARN/ERROR/FATAL dựa trên rib_to_wall_ratio (khi có) và cơ chế fail-safe khi chỉ có proxy.

Ứng dụng: dùng để reporter tổng hợp finding nhất quán; tránh kết luận sai khi agent đang ở proxy_mode.

Non-goals: không thay đổi thuật toán agent; không thiết kế lại pipeline; không thay đổi contract semantics; không bổ sung quy tắc mới ngoài nguồn. Nếu cần nội dung mới để hoàn thiện template sẽ ghi [CHƯA XÁC MINH].

## 2. Included Sources & Mapping Table

|           |                                                                        |                                                                   |                         |                                                  |                                  |
| --------- | ---------------------------------------------------------------------- | ----------------------------------------------------------------- | ----------------------- | ------------------------------------------------ | -------------------------------- |
| source_id | filename                                                               | original_title                                                    | original_version/status | sections_used                                    | notes                            |
| S1        | TÀI LIỆU — DFM_POLICY_SEVERITY_MAP_COMPLETE_SET_v1.docx                | TÀI LIỆU — DFM_POLICY_SEVERITY_MAP_COMPLETE_SET_v1                | [CHƯA XÁC MINH]         | severity map block (ribratio + combine strategy) | primary severity rules           |
| S2        | PHẦN 1 — DFM_POLICY_KEYS_MINIMUM_v1.docx                               | PHẦN 1 — DFM_POLICY_KEYS_MINIMUM_v1                               | [CHƯA XÁC MINH]         | keys + fail-safe (section 5.4)                   | required/optional keys semantics |
| S3        | DFM_POLICY_KEYS_MINIMUM_v1 — BẢN HỢP NHẤT (UPDATED, CONSOLIDATED).docx | DFM_POLICY_KEYS_MINIMUM_v1 — BẢN HỢP NHẤT (UPDATED, CONSOLIDATED) | [CHƯA XÁC MINH]         | JSON skeleton thresholds subtree                 | starter defaults skeleton        |
| S4        | TÀI LIỆU TRIỂN KHAI ADN AMBR LDR — MODULE ribratio_agent.docx          | TÀI LIỆU TRIỂN KHAI ADN AMBR LDR — MODULE ribratio_agent          | [CHƯA XÁC MINH]         | defect advisory template + policy references     | defect schema + codes            |
| S5        | GOVERNANCE_POLICY_SCHEMA_v1.docx                                       | GOVERNANCE_POLICY_SCHEMA_v1                                       | [CHƯA XÁC MINH]         | schema placement of thresholds/severity_map      | anchor to GOV policy schema      |

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

5.4. ribratio_agent (chuẩn bị để thoát khỏi bbox-proxy về sau)
Required (DFM-thực chiến):

thresholds.ribratio.max_rib_to_wall_ratio: float (dimensionless)
Optional:

thresholds.ribratio.preferred_rib_to_wall_ratio: float

thresholds.ribratio.min_root_radius: float (length_unit) [CHƯA XÁC MINH]

thresholds.ribratio.nominal_wall_strategy: giống thickness_agent (nếu dùng chung) [CHƯA XÁC MINH]

Fail-safe nếu thiếu:

POLICY_UNRESOLVED; nếu agent đang ở proxy V0 (bbox ratio), chỉ xuất proxy facts, không “giả” rib_to_wall.

## 3.3 Thresholds subtree (starter defaults skeleton)

Source: S3

{
"dfm_policy": {
"thresholds": {
"ribratio": {
"max_rib_to_wall_ratio": null,
"preferred_rib_to_wall_ratio": null,
"min_root_radius_mm": null
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

## 3.5 Defect advisory template (for consistency with reporter)

Source: S4

────────────────────────────────────────────────────────────
F) Defect advisory template — ribratio_agent

RIBRATIO_DEFECT_TEMPLATE_v1:

id

run_id

agent_source: "ribratio_agent"

rule_version

geometry_version_id

timestamp_utc

severity: INFO|WARN|ERROR|FATAL (từ policy severity_map.ribratio.\*)

defect_type:

"RIBRATIO_OVER_THRESHOLD"

"RIBRATIO_PROXY_ONLY_LOW_CONFIDENCE"

"RIBRATIO_LOW_CONFIDENCE_THICKNESS_COVERAGE"

region_ref: REGION_REF_SCHEMA_v1 (AGL FACE/PATCH)

analysis_value:

t_rib_mm: float|null

t_wall_mm: float|null

rib_to_wall_ratio: float|null

method: "RIB_TO_WALL_V1"|"BBOX_PROXY_V0"

threshold_applied:

max_rib_to_wall_ratio: float|null

advisory:

machine_readable:

summary_code: "REDUCE_RIB_THICKNESS" | "INCREASE_ROOT_RADIUS" | "CORE_OUT" | "REVIEW_NOMINAL_WALL"

recommended_actions: [...]

rationale_tags: ["SINK_RISK", "THICK_JUNCTION", "RIB_TOO_THICK", "LOW_CONFIDENCE"]

human_readable:

title

why_it_matters (sink ở chân rib, cycle time, cosmetic)

suggested_fix (t_rib giảm, transition mượt, radius chân tăng)

verification (đo lại ratio, đối chiếu tiêu chuẩn nội bộ)

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
| S4              | THÔNG TIN PHIÊN BẢN (DOCUMENT METADATA) doc_id: TOOLING_DFM_ADVISOR_RIBRATIO_AGENT_DFM_V1_RESEARCH_v1 status: DRAFT effective_date_utc: [CHƯA XÁC MINH] owner: [CHƯA XÁC MINH] units_locked: length_unit = mm related_specs: TOOLING_DFM_ADVISOR...  |
| S4              | thickness_result từ thickness_agent (local thickness distribution + worst regions) [CHƯA XÁC MINH: field wiring cụ thể]                                                                                                                              |
| S4              | dfm_policy.thresholds.ribratio.min_root_radius_mm (để cảnh báo chân rib nhọn) [CHƯA XÁC MINH]                                                                                                                                                        |
| S4              | afr_provider output nếu có feature graph (để tăng recall/precision rib detection) [CHƯA XÁC MINH: afr hiện proxy topo]                                                                                                                               |
| S4              | OCC missing: NOT_EXECUTED/STUB_MODE, trừ khi agent chạy purely-on-data (có sẵn rib candidates từ upstream) [CHƯA XÁC MINH: bạn có muốn hỗ trợ mode này không]                                                                                        |
| S4              | Dùng adjacency graph: rib side faces thường là 2 mặt dài, đối diện, chung edge với một “top face” nhỏ. [CHƯA XÁC MINH: mức sâu adjacency bạn muốn]                                                                                                   |
| S4              | t_wall: sample quanh chân rib (tránh self-hit, dùng epsilon mm theo policy thickness.probe.self_hit_epsilon_mm nếu dùng chung) [CHƯA XÁC MINH: reuse key hay tạo key riêng] Step 6: Compute ratio và rank worst ribs. Step 7: Evaluate vs thresh...  |
| S4              | AFR rib graph + boss junction classification để tăng recall/precision. [CHƯA XÁC MINH]                                                                                                                                                               |
| S4              | Root radius measurement (min_root_radius_mm) và rib height ratio checks (h/t) [CHƯA XÁC MINH: bạn muốn thêm key nào]                                                                                                                                 |
| S4              | rule_version [CHƯA XÁC MINH]                                                                                                                                                                                                                         |
| S4              | receipts_ref: [CHƯA XÁC MINH]                                                                                                                                                                                                                        |
| [CHƯA XÁC MINH] | Quan hệ giữa proxy_mode (bbox proxy) và việc có/không emit RIBRATIO_OVER_THRESHOLD: nguồn đã nêu 'proxy chỉ LOW_CONFIDENCE'; cần Owner xác nhận enforcement ở code hay chỉ ở policy.                                                                 |

## 5. Change Log (Editorial)

Thay đổi biên tập/tái cấu trúc để chuẩn hoá theo Document Governance Layer; không đổi semantics.

- Tạo mới canonical POL theo WS, thêm Document Control Block + Mapping Table (Run ID: 20260304T122755Z).

- Trích nguyên văn các đoạn policy keys, severity rules, defect template từ nguồn; sắp xếp lại theo template bắt buộc.

- Nếu có nội dung cần điền nhưng nguồn không có: ghi [CHƯA XÁC MINH].

## 6. Appendix

Cross references (canonical codes): WS_AGENTS-ALL-POL-v1 (global DFM policy); WS_MODEL-RGN-POL-v1 (REGION_REF_CONVENTION_v1).

Naming conventions: module snake_case; contracts/envelopes UPPER_CASE versioned.
