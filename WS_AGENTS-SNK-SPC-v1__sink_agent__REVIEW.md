WS_AGENTS-SNK-SPC-v1 — sink_agent (DFM-first)

# 0. Document Control Block

|                                  |                                                                      |
| -------------------------------- | -------------------------------------------------------------------- |
| Field                            | Value                                                                |
| Canonical Code                   | WS_AGENTS-SNK-SPC-v1                                                 |
| Status                           | REVIEW                                                               |
| Run ID (UTC)                     | 20260304T111100Z                                                     |
| WS / Module / Doc Type / Version | WS_AGENTS / SNK / SPC / v1                                           |
| Created date (UTC)               | 20260304T111100Z                                                     |
| Purpose                          | Document governance consolidation only; no runtime/pipeline changes. |

## 1. Scope

*Source: S1*

PROBLEM STATEMENT (DFM thực chiến)
Core problem: Ước lượng rủi ro sink/void/cycle-time hotspot do “khối dày cục bộ” và “biến thiên độ dày lớn” trong chi tiết nhựa ép, để kỹ sư khuôn quyết định: core-out, làm rỗng boss, giảm mass, tăng transition radius, chỉnh rib/boss, hoặc thay đổi bố trí cooling/gate/vent (ở mức advisory, không CAE).

Scope (V1 baseline):

Dùng thickness map (local thickness) để tìm vùng có thickness cao bất thường so với “nominal wall” của chi tiết.

Định vị “worst regions” theo REGION_REF_CONVENTION_v1 (FACE/PATCH trên AGL).

So ngưỡng dfm_policy.thresholds.sink.local_mass.max_local_thickness_ratio (dimensionless) và/hoặc max_boss_to_wall_ratio nếu có tagging boss.

Nếu chưa có thickness map: fallback sang proxy V0 (bbox anisotropy) nhưng bắt buộc gắn “PROXY_ONLY_LOW_CONFIDENCE”.

Non-goals:

Không mô phỏng co rút/warpage/cooling (CAE).

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

Không “đoán vị trí sink” nếu chỉ có global proxy; proxy chỉ là tín hiệu cảnh báo.

Non-goals (verbatim extract):

Không mô phỏng co rút/warpage/cooling (CAE).

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

Không “đoán vị trí sink” nếu chỉ có global proxy; proxy chỉ là tín hiệu cảnh báo.

Không tự thay đổi CAD.

## 2. Included Sources & Mapping Table

|           |                                                                       |                                                                      |                                                                          |                                                                                                 |                                                                                                                    |
| --------- | --------------------------------------------------------------------- | -------------------------------------------------------------------- | ------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| source_id | filename                                                              | original_title                                                       | original_version/status                                                  | sections_used                                                                                   | notes                                                                                                              |
| S1        | TÀI LIỆU TRIỂN KHAI ADN AMBR LDR — MODULE sink_agent (DFM-first).docx | TÀI LIỆU TRIỂN KHAI ADN / AMBR / LDR — MODULE sink_agent (DFM-first) | status: DRAFT; doc_id: TOOLING_DFM_ADVISOR_SINK_AGENT_DFM_V1_RESEARCH_v1 | A) Naming & Identifiers; B) ADN; C) AMBR; D) LDR; E) Schema sketch; F) Defect advisory template | Consolidated into canonical SPC; content preserved as verbatim extracts; added governance metadata + traceability. |

## 3. Consolidated Specification Content

## 3.1 ADN (Algorithm / Logic)

*Source: S1*

────────────────────────────────────────────────────────────
B) ADN — sink_agent (V1 baseline)

IDENTIFICATION
adn_id: ADN_sink_agent_local_mass_sink_risk_from_thickness_map_v1
module_name: sink_agent
module_type: analysis_agent
algorithm_name: local_mass_sink_risk_from_thickness_map
algorithm_version: v1
status: DRAFT

PROBLEM STATEMENT (DFM thực chiến)
Core problem: Ước lượng rủi ro sink/void/cycle-time hotspot do “khối dày cục bộ” và “biến thiên độ dày lớn” trong chi tiết nhựa ép, để kỹ sư khuôn quyết định: core-out, làm rỗng boss, giảm mass, tăng transition radius, chỉnh rib/boss, hoặc thay đổi bố trí cooling/gate/vent (ở mức advisory, không CAE).

Scope (V1 baseline):

Dùng thickness map (local thickness) để tìm vùng có thickness cao bất thường so với “nominal wall” của chi tiết.

Định vị “worst regions” theo REGION_REF_CONVENTION_v1 (FACE/PATCH trên AGL).

So ngưỡng dfm_policy.thresholds.sink.local_mass.max_local_thickness_ratio (dimensionless) và/hoặc max_boss_to_wall_ratio nếu có tagging boss.

Nếu chưa có thickness map: fallback sang proxy V0 (bbox anisotropy) nhưng bắt buộc gắn “PROXY_ONLY_LOW_CONFIDENCE”.

Non-goals:

Không mô phỏng co rút/warpage/cooling (CAE).

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

Không “đoán vị trí sink” nếu chỉ có global proxy; proxy chỉ là tín hiệu cảnh báo.

Không tự thay đổi CAD.

INPUT FORMALIZATION
Required (để chạy local-mass V1 đúng nghĩa):

thickness_result: output của thickness_agent (thickness_distribution_mm + worst_regions + sampling quality). [CHƯA XÁC MINH: wiring field name]

dfm_policy.units.length_unit == "mm"

dfm_policy.thresholds.sink.local_mass.max_local_thickness_ratio (dimensionless) hoặc ít nhất dfm_policy.thresholds.sink.max_sink_risk_proxy nếu phải fallback.

Optional (tăng chất lượng nhưng không bắt buộc V1):

“nominal_wall_strategy”: dùng p50 thickness làm nominal (khuyến nghị baseline), hoặc user_specified. [CHƯA XÁC MINH: bạn muốn chốt strategy ở policy hay lấy mặc định p50]

afr tagging boss/rib/junction để phân loại nguyên nhân sink (V2).

Fail-safe dependency:

Nếu OCC thiếu nhưng thickness_result đã có (tính từ earlier run) thì sink_agent vẫn có thể chạy purely-on-data. Nếu không có thickness_result và OCC thiếu: status NOT_EXECUTED/STUB_MODE, issue OCC_MISSING.

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

OUTPUT FORMALIZATION
Output envelope: SINK_ANALYSIS_RESULT_v0 (or v1 nếu bạn bump).
Bắt buộc theo Master Spec: result_type, run_id, status, error_code/disable_reason, rule_version, geometry_version_id, metrics, issues, produced_defects, receipts_ref.

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

Metrics V1 baseline (đủ để kỹ sư khuôn ra quyết định):

nominal_wall_mm: p50_thickness_mm (hoặc user_specified)

thick_ratio_distribution: max, p90 (dimensionless), trong đó thick_ratio = (t_local − t_nominal)/t_nominal

worst_regions: top N {region_ref, t_local_mm, thick_ratio, applied_threshold, confidence}

coverage/quality echo: lấy từ thickness_result (miss_rate, sample_count_total)

Guarantees:

Read-only SSOT; không mutate AGL.

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

Nếu thiếu policy threshold: POLICY_UNRESOLVED, chỉ xuất facts (nominal + thick_ratio stats) và không emit “over threshold defect”.

STEP-BY-STEP LOGIC
Step 1: Initialize envelope SINK_ANALYSIS_RESULT_v0.
Step 2: Resolve inputs:

Nếu có thickness_result hợp lệ: dùng local-mass path.

Nếu không có thickness_result: dùng bbox proxy path (V0) theo sink_agent hiện hữu.

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

Step 3: Determine nominal_wall_mm:

nominal = thickness_distribution_mm.p50 (baseline)

Nếu policy có user_specified_nominal: override. [CHƯA XÁC MINH]
Step 4: Compute thick_ratio for candidate regions:

ưu tiên lấy top “thick regions” từ thickness_result (tức vùng t_local cao)

thick_ratio = (t_local − nominal)/nominal
Step 5: Evaluate against sink thresholds:

if max_local_thickness_ratio exists: flag over_threshold khi thick_ratio >= threshold

else: POLICY_UNRESOLVED
Step 6: Emit defects (top N):

DEFECT_TYPE: SINK_LOCAL_MASS_OVER_THRESHOLD (hoặc SINK_PROXY_HIGH_RISK nếu fallback)

region_ref theo chuẩn (FACE hoặc PATCH trên AGL).
Step 7: Finalize envelope: status OK / FAILED / STUB_MODE.

FAILURE MODES

THICKNESS_RESULT_MISSING: fallback proxy hoặc POLICY_UNRESOLVED (tùy policy).

POLICY_UNRESOLVED: không kết luận “over threshold”, chỉ facts.

UNIT_UNRESOLVED: ERROR (vì thickness mm mà unit không chắc → sai hoàn toàn).

OCC_MISSING và không có thickness_result: NOT_EXECUTED/STUB_MODE.

LOW_COVERAGE từ thickness_result: sink_agent phải downgrade confidence và phát issue SINK_LOW_CONFIDENCE_THICKNESS_COVERAGE.

COMPLEXITY
Nếu dùng thickness_result: O(N_worst_regions) ~ nhỏ.
Nếu fallback bbox proxy: O(1) + OCC bbox.

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

V2 UPGRADE INPUTS (để sink thành “advisor mạnh”)

AFR tagging boss/rib/junction + nominal wall mapping theo feature (không chỉ global p50). [CHƯA XÁC MINH]

Transition analysis: gradient thickness/step change và “thick island detection” bằng clustering/area. [CHƯA XÁC MINH]

Process/material context: packing sensitivity, cosmetic class để severity mapping chuẩn hơn. [CHƯA XÁC MINH]

## 3.2 AMBR (Bindings / Module Contract / Execution)

*Source: S1*

────────────────────────────────────────────────────────────
C) AMBR — sink_agent (binding)

ambr_id: AMBR_sink_agent_v2_research
status: DRAFT
module_name: sink_agent
module_type: analysis_agent
execution_mode: SEQUENTIAL (internal), orchestration PARALLEL-safe

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

pipeline_stage: Step 5 (Parallel Analysis Agents)

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

Bound algorithm: ADN_sink_agent_local_mass_sink_risk_from_thickness_map_v1
Bound contracts/inputs: depends on thickness_agent output (THICKNESS_ANALYSIS_RESULT\_\*) [CHƯA XÁC MINH: exact field wiring]

Responsibility split:

sink_agent: local mass risk aggregation + defect emission + fallback proxy labeling

pipeline/orchestrator: inject run_id/rule_version/geometry_version_id, persist receipts, always run reporter

TÀI LIỆU ĐẶC TẢ HỆ THỐNG QUẢN T…

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

## 3.3 LDR (Dependencies / Runtime Constraints)

*Source: S1*

────────────────────────────────────────────────────────────
D) LDR — dependencies (research)

ldr_id: LDR_sink_agent_depends_on_thickness_map_and_optional_occ_bbox_v1
status: DRAFT
dependencies:

internal: thickness_agent result envelope (data dependency)

optional external: pythonocc-core (only for fallback bbox proxy path)

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

risk note: ưu tiên “data-only path” để giảm phụ thuộc OCC tại sink_agent, tăng độ bền vận hành.

## 6. Appendix

## 6.1 Result schema sketch (informative; not SSOT)

*Source: S1*

────────────────────────────────────────────────────────────
E) SINK_ANALYSIS_RESULT schema sketch (DFM-first)

SINK_ANALYSIS_RESULT_v0 (đề xuất mở rộng theo V1 baseline; fields mới optional):

result_type: "SINK_ANALYSIS_RESULT_v0"

run_id

status: OK|FAILED|NOT_EXECUTED|STUB_MODE

rule_version [CHƯA XÁC MINH]

geometry_version_id

inputs_echo:

units: {length_unit:"mm"}

policy_echo:

max_sink_risk_proxy: float|null

max_local_thickness_ratio: float|null

nominal_wall_strategy: "p50_thickness"|...|null

metrics:

method_used: "LOCAL_MASS_FROM_THICKNESS" | "BBOX_PROXY_V0"

nominal_wall_mm: float|null

thick_ratio_distribution:

max: float|null

p90: float|null

worst_regions: \[
{
"region_ref": REGION_REF_SCHEMA_v1,
"t_local_mm": float|null,
"thick_ratio": float|null,
"over_threshold": bool|null,
"applied_threshold": float|null,
"confidence_score": float
}
\]

upstream_quality:

thickness_miss_rate: float|null

thickness_sample_count: int|null

issues: [...]

produced_defects: [...]

receipts_ref: [CHƯA XÁC MINH]

## 6.2 Defect advisory template (informative; policy mapping may belong to POL)

*Source: S1*

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

effective_date_utc and owner in source metadata are unspecified. What are the authoritative values, and where should they be governed?

Envelope versioning rule: when should SINK_ANALYSIS_RESULT bump from v0 to v1 (and what repository rule enforces this)?

Exact wiring/field names for thickness_result dependency are unspecified in source (e.g., thickness_distribution_mm, worst_regions). Confirm contract fields.

Nominal wall strategy: should user_specified_nominal be supported via policy, and what is the default (p50 thickness)?

rule_version and receipts_ref fields are marked [CHƯA XÁC MINH] in schema sketch; confirm governance source of truth.

## 5. Change Log (Editorial)

Converted source document into canonical WS_AGENTS-SNK-SPC-v1 structure (governance-only).

Standardized headings to template sections and added Document Control Block, Included Sources & Mapping Table.

Inserted traceability markers (Source: S1) for each major subsection; preserved technical content as verbatim extracts.

No semantics changes; any missing data remains tagged as [CHƯA XÁC MINH] and surfaced under Open Questions.
