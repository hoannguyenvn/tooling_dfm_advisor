WS_AGENTS-THK-POL-v1 (REVIEW)

# 0. Document Control Block

|                                  |                                                                      |
| -------------------------------- | -------------------------------------------------------------------- |
| Field                            | Value                                                                |
| Canonical Code                   | WS_AGENTS-THK-POL-v1                                                 |
| Status                           | REVIEW                                                               |
| Run ID                           | 20260304T101954Z                                                     |
| WS / Module / Doc Type / Version | WS_AGENTS / THK / POL / v1                                           |
| Created date (UTC)               | 20260304T101954Z                                                     |
| Purpose                          | Document governance consolidation only; no runtime/pipeline changes. |

## 1. Scope

Source: S1, S2

This document consolidates policy, thresholds guidance, severity mapping, and defect advisory templates for thickness_agent. Content is inherited from provided sources; no new analysis logic is introduced.

Non-goals: changing agent algorithms, altering SSOT/pipeline contracts, or introducing new runtime dependencies beyond what sources state.

## 2. Included Sources & Mapping Table

|           |                                                      |                                            |                         |               |                                            |
| --------- | ---------------------------------------------------- | ------------------------------------------ | ----------------------- | ------------- | ------------------------------------------ |
| source_id | filename                                             | original_title                             | original_version/status | sections_used | notes                                      |
| S1        | Defect advisory template — thickness_agent.docx      | Defect advisory template — thickness_agent | [CHƯA XÁC MINH]         | 3.1, 3.3, 4   | Defect advisory template (thickness_agent) |
| S2        | TÀI LIỆU — DFM_POLICY_SEVERITY_MAP_THICKNESS_v1.docx | DFM_POLICY_SEVERITY_MAP_THICKNESS_v1       | DRAFT (per source)      | 3.2, 4        | Severity map and policy keys for thickness |

## 3. Consolidated Policy Content

## 3.1 Defect Advisory Template for thickness_agent

Source: S1

Defect advisory template — thickness_agent

THICKNESS_DEFECT_TEMPLATE_v1 (tuân DEFECT_OBJECT_API_v1 của Master Spec)

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

id

run_id

agent_source: "thickness_agent"

rule_version

geometry_version_id

timestamp_utc

severity: INFO|WARN|ERROR|FATAL

defect_type: "THICKNESS_BELOW_MIN_WALL" | "THICKNESS_ABOVE_MAX_WALL" | "THICKNESS_MAP_LOW_COVERAGE"

region_ref: REGION_REF_SCHEMA_v1 (ưu tiên FACE hoặc PATCH trên AGL)

analysis_value:

thickness_mm: float

method: "normal_probe"

hit_quality: "OK"|"AMBIGUOUS"|"MISS" (nếu defect là LOW_COVERAGE)

threshold_applied:

min_wall_mm: float|null

max_wall_mm: float|null

advisory:

machine_readable:

summary_code: "INCREASE_WALL" | "REDUCE_MASS" | "SMOOTH_TRANSITION" | "VERIFY_UNITS"

recommended_actions: list[str]

rationale_tags: list[str] (THIN_WALL, THICK_MASS, JUNCTION_RISK, LOW_COVERAGE)

human_readable:

title

why_it_matters (đúng ngôn ngữ khuôn/ép: short shot/sink/cycle time)

suggested_fix (gợi ý nguyên tắc, không auto-fix CAD)

verification (đo lại thickness map, kiểm min_wall theo vật liệu/process)

confidence_score: 0..1

assets_ephemeral_ref: (token only; không lưu path)

delivery_scrubbed: boolean

Gợi ý severity (để đưa vào policy sau, không hard-code):

BELOW_MIN_WALL: ERROR (nếu thiếu nhiều hoặc diện tích lớn), WARN (thiếu nhẹ) [CHƯA XÁC MINH: tiêu chí “nhiều/nhẹ” theo %]

ABOVE_MAX_WALL: WARN/ERROR tùy mức dày và vị trí (junction) [CHƯA XÁC MINH]

LOW_COVERAGE: WARN (cần kiểm tra solid validity / open shell)

## 3.2 Severity Map for Thickness (severity_map.thickness)

Source: S2

TÀI LIỆU — DFM_POLICY_SEVERITY_MAP_THICKNESS_v1

doc_id: TOOLING_DFM_ADVISOR_DFM_POLICY_SEVERITY_MAP_THICKNESS_v1

status: DRAFT

effective_date_utc: [CHƯA XÁC MINH]

units: length_unit = mm (đã chốt)

Mục tiêu

Biến “mức vi phạm thickness” thành severity INFO/WARN/ERROR/FATAL một cách nhất quán.

Không phụ thuộc CAE; dựa vào hình học local thickness, mức thiếu/dư và “diện tích/coverage” của vùng vi phạm.

Dùng được cho: THICKNESS_BELOW_MIN_WALL, THICKNESS_ABOVE_MAX_WALL, THICKNESS_MAP_LOW_COVERAGE.

Inputs mà thickness_agent phải cung cấp để severity_map áp dụng được

Required:

thresholds.thickness.min_wall (mm)

Optional nhưng rất nên có:

thresholds.thickness.max_wall (mm)

per-region: thickness_mm, violation_type, and region_area_estimate_mm2 [CHƯA XÁC MINH: cách ước lượng area]

global: ray_miss_rate, ambiguous_hit_rate, sample_count_total

Nếu thiếu area_estimate:

severity_map vẫn chạy nhưng dựa vào “violation ratio” + “count ratio” thay area (ít chính xác hơn).

Định nghĩa các đại lượng chuẩn hoá

3.1. Thin deficit ratio (vô thứ nguyên)

thin_deficit_ratio = (min_wall - t) / min_wall, với t < min_wall

Ví dụ: min_wall=2.0mm, t=1.6mm ⇒ deficit=0.4/2.0=0.20 (thiếu 20%)

3.2. Thick excess ratio (vô thứ nguyên)

Hai cách, chọn 1 (khuyến nghị cách A):

A) thick_excess_ratio = (t - max_wall) / max_wall, với t > max_wall

B) thick_excess_ratio = (t - nominal_wall) / nominal_wall [CHƯA XÁC MINH: nominal_wall strategy]

3.3. Violation coverage

coverage_ratio = violated_area / evaluated_area (0..1) [CHƯA XÁC MINH: nếu chưa có area, dùng violated_sample_ratio]

Severity rules đề xuất (default)

Đặt trong policy:

dfm_policy:

severity_map:

thickness:

thin_wall:

deficit_ratio_thresholds:

warn_min: 0.05

error_min: 0.15

fatal_min: 0.30

coverage_ratio_thresholds:

warn_min: 0.01

error_min: 0.05

fatal_min: 0.15

escalation_rules:

- if_deficit_at_least: 0.30

severity: FATAL

- if_deficit_at_least: 0.15

severity: ERROR

- if_deficit_at_least: 0.05

severity: WARN

- else: INFO

thick_wall:

excess_ratio_thresholds:

warn_min: 0.10

error_min: 0.25

fatal_min: 0.50

coverage_ratio_thresholds:

warn_min: 0.01

error_min: 0.05

fatal_min: 0.15

escalation_rules:

- if_excess_at_least: 0.50

severity: FATAL

- if_excess_at_least: 0.25

severity: ERROR

- if_excess_at_least: 0.10

severity: WARN

- else: INFO

map_quality:

miss_rate_thresholds:

warn_min: 0.10

error_min: 0.25

fatal_min: 0.50

ambiguous_hit_rate_thresholds:

warn_min: 0.10

error_min: 0.25

fatal_min: 0.50

sample_count_min:

warn_min: 200

error_min: 80

fatal_min: 30

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

5. Cách combine “mức thiếu/dư” và “coverage”

Thực tế mold review: một điểm cực mỏng nhưng rất nhỏ có thể chấp nhận (hoặc sửa nhỏ), còn mỏng “trải dài” thì rất nguy hiểm.

Khuyến nghị combine rule như sau (để agent tính ra final severity cho defect):

base_severity từ deficit_ratio (hoặc excess_ratio)

escalate nếu coverage_ratio vượt ngưỡng tương ứng

Pseudo-logic (policy-side, minh bạch):

severity = base_severity(deficit_ratio)

if coverage_ratio >= coverage.error_min then severity = max(severity, ERROR)

if coverage_ratio >= coverage.fatal_min then severity = max(severity, FATAL)

Nếu thiếu coverage_ratio:

thay bằng violated_sample_ratio (số sample vi phạm / sample hợp lệ)

“Owner knobs” cần chốt để severity_map phản ánh đúng thực tế xưởng

Các giá trị default ở trên là hợp lý chung, nhưng bạn nên chốt theo:

Vật liệu (PP/ABS/PC/PA…), độ co, độ cứng, yêu cầu mỹ quan. [CHƯA XÁC MINH: material_class taxonomy]

Quy trình: injection molding thường, gas-assist, RHCM… [CHƯA XÁC MINH]

Tiêu chuẩn công ty: “thin deficit 15% đã là ERROR hay chỉ WARN?”

Output yêu cầu từ thickness_agent để policy hoạt động tốt

Để severity_map chạy “đúng như kỹ sư khuôn”, thickness_agent nên xuất:

thickness_min_mm (per worst region)

thin_deficit_ratio (per worst region)

violated_sample_ratio hoặc coverage_ratio

miss_rate, sample_count_total

## 3.3 Conventions and Cross-References

Source: S1, S2

Referenced schemas/contracts appearing in sources (not redefined here): DEFECT_OBJECT_API_v1, REGION_REF_SCHEMA_v1, and policy path dfm_policy.severity_map.thickness.\*. If canonical documents exist for these, this POL only cross-references them.

## 4. Open Questions / [CHƯA XÁC MINH]

Source: S1, S2

S1: ABOVE_MAX_WALL: WARN/ERROR tùy mức dày và vị trí (junction) [CHƯA XÁC MINH]

S2: TÀI LIỆU — DFM_POLICY_SEVERITY_MAP_THICKNESS_v1
doc_id: TOOLING_DFM_ADVISOR_DFM_POLICY_SEVERITY_MAP_THICKNESS_v1
status: DRAFT
effective_date_utc: [CHƯA XÁC MINH]
units: length_unit = mm (đã chốt)

S2: Quy trình: injection molding thường, gas-assist, RHCM… [CHƯA XÁC MINH]

## 5. Change Log (Editorial)

This document is produced by document governance consolidation. Changes are limited to: canonical naming, section ordering per template, and insertion of traceability markers (“Source: Sx”). No semantics were changed.

RUN_ID=20260304T101954Z — Created WS_AGENTS-THK-POL-v1 as consolidated POL from S1–S2.

## 6. Appendix

6.1 Terminology mapping (aka)

If sources use alternative names for the same concept, they are preserved as-is; this document may add “aka” annotations in future revisions if needed.
