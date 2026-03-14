**0. Document Control Block**

Canonical Code: WS_AGENTS-DRF-POL-v1

Status: REVIEW

Run ID: 20260304T095853Z

WS / Module / Doc Type / Version: WS_AGENTS / DRF / POL / v1

Created date (UTC): 20260304T095853Z

Purpose: Document governance consolidation only; no runtime/pipeline changes.

**1. Scope**

This document consolidates draft_agent policy artifacts: severity mapping rules and defect advisory template fields, for use as policy-as-config and reporter-consumable governance references.

Non-goals: changing draft_agent algorithm, redefining result envelopes, or introducing new runtime behaviors. This is a documentation consolidation and naming/version control action only.

**2. Included Sources & Mapping Table**

|           |                                                                                        |                                        |                         |                                                          |                                                    |
| --------- | -------------------------------------------------------------------------------------- | -------------------------------------- | ----------------------- | -------------------------------------------------------- | -------------------------------------------------- |
| source_id | filename                                                                               | original_title                         | original_version/status | sections_used                                            | notes                                              |
| S1        | Defect advisory template — dành riêng cho draft_agent (theo DEFECT_OBJECT_API_v1).docx | Defect advisory template — draft_agent | N/A                     | Policy content: defect advisory template & defect fields | Verbatim extracts; retains [CHƯA XÁC MINH] markers |
| S2        | TÀI LIỆU — DFM_POLICY_SEVERITY_MAP_DRAFT_v1.docx                                       | DFM_POLICY_SEVERITY_MAP_DRAFT_v1       | DRAFT                   | Policy content: severity map rules/thresholds/overrides  | Verbatim extracts; retains [CHƯA XÁC MINH] markers |

**3. Consolidated Policy Content**

**3.1 Draft defect advisory template (DEFECT_OBJECT_API_v1 alignment)**

Source: S1

DRAFT_DEFECT_TEMPLATE_v0 (dict):

id: "\<defect_id>" (unique, traceable)

run_id: "YYYYMMDDThhmmssZ"

agent_source: "draft_agent"

rule_version: "RULESET_v1" [CHƯA XÁC MINH]

geometry_version_id: "\<geometry_version_id>"

timestamp_utc: "\<ISO_8601_UTC>"

severity: "INFO"|"WARN"|"ERROR"|"FATAL" (dùng thống nhất để reporter đếm)

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

defect_type: "DRAFT_BELOW_THRESHOLD" | "REVERSE_DRAFT" | "SILHOUETTE_CROSSING" | "EJECTION_NOT_CONFIRMED" [CHƯA XÁC MINH: taxonomy defect_type đã chốt chưa]

operational_vector:

ejection_direction: {x: float, y: float, z: float}

user_confirmed: true

geometry_vector:

brep_face_normal: {x: float, y: float, z: float} | null (tại điểm tệ nhất)

mesh_smoothed_normal: null (V1 baseline không dùng)

hybrid_preference: "brep" [CHƯA XÁC MINH]

region_ref:

face_id: string|int [CHƯA XÁC MINH]

uv_hint: {u: float, v: float} | null

point_hint: {x: float, y: float, z: float} | null

note: "do not store durable file paths" (nhắc invariant no shadow data)

analysis_value:

draft_min_deg: float

dot_at_worst: float

threshold_applied:

min_draft_deg: float | null (null nếu POLICY_UNRESOLVED)

surface_class: "polish"|"light_texture"|"heavy_texture"|null [CHƯA XÁC MINH]

advisory:

machine_readable:

summary_code: "INCREASE_DRAFT" | "CHANGE_PARTING" | "ADD_SIDE_ACTION" | "REVIEW_EJECTION_DIR" (closed-set khuyến nghị)

recommended_actions: list[str] (ngắn, có thể map sang UI)

rationale_tags: list[str] (VD: ["CURVED_SURFACE", "SILHOUETTE", "REVERSE_DRAFT"])

human_readable:

title: string

why_it_matters: string (giải thích theo ngôn ngữ khuôn: kẹt/drag mark/stick)

suggested_fix: string (gợi ý hướng xử lý, không auto-fix CAD)

verification: string (cách kiểm: đo draft theo pull dir, kiểm parting line, kiểm texture requirement)

confidence_score: float (0..1)

assets_ephemeral_ref:

snapshot_png_ref: string|null

section_svg_ref: string|null

delivery_scrubbed: false|true (phải true sau delivery_complete)

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

Gợi ý mapping severity (không bắt buộc, nhưng nên chốt để báo cáo “đúng tinh thần khuôn”):

REVERSE_DRAFT:

thường là ERROR; lên FATAL nếu vùng lớn/ở A-surface/không có phương án split rõ ràng. [CHƯA XÁC MINH: quy tắc severity theo diện tích/criticality]

DRAFT_BELOW_THRESHOLD:

WARN nếu thiếu nhẹ; ERROR nếu thiếu nhiều hoặc ở vùng dài theo hướng rút (nguy cơ drag). [CHƯA XÁC MINH]

SILHOUETTE_CROSSING:

WARN/ERROR tùy dự án; mục tiêu chính là “buộc kỹ sư quyết định parting”. [CHƯA XÁC MINH]

Lưu ý quan trọng: các rule severity ở trên là “DFM practice” và phải được đưa vào policy-as-config (rule_versioned) nếu bạn muốn hệ thống minh bạch/kiểm toán được, thay vì hard-code trong agent.

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

TÀI LIỆU ĐẶC TẢ HỆ THỐNG QUẢN T…

**3.2 Severity map for draft_agent (INFO/WARN/ERROR/FATAL)**

Source: S2

TÀI LIỆU — DFM_POLICY_SEVERITY_MAP_DRAFT_v1
doc_id: TOOLING_DFM_ADVISOR_DFM_POLICY_SEVERITY_MAP_DRAFT_v1
status: DRAFT
effective_date_utc: [CHƯA XÁC MINH]
units: angle_unit = deg, dot dimensionless

Mục tiêu

Ánh xạ kết quả draft (đặc biệt trên mặt cong) thành severity INFO/WARN/ERROR/FATAL theo logic khuôn: kẹt/drag mark, xước bề mặt, khó đẩy, bắt buộc đổi parting hoặc thêm side action.

Bao phủ 3 nhóm defect chính của draft_agent V1 baseline:
A) DRAFT_BELOW_THRESHOLD (thiếu draft)
B) REVERSE_DRAFT (dot < 0 theo ejection_dir)
C) SILHOUETTE_CROSSING (dot đổi dấu trên cùng patch/face → parting line chạy trên patch hoặc phải split)

Inputs draft_agent cần cung cấp để severity_map áp dụng được
Required:

thresholds.draft.min_draft_deg_default (deg) hoặc min_draft_deg_by_surface_class (deg)

per-region / per-face:

draft_min_deg_region (deg)

reverse_present (bool) hoặc reverse_sample_ratio_region

silhouette_crossing (bool)
Optional nhưng rất nên có:

coverage_ratio:

below_threshold_coverage_ratio (violated_area / evaluated_area) hoặc violated_sample_ratio

reverse_coverage_ratio (tỷ lệ sample dot\<0)

silhouette_coverage_ratio (tỷ lệ face có sign-change) hoặc count ratio

surface_class tag: polish/light_texture/heavy_texture [CHƯA XÁC MINH: nguồn tagging]

Nếu thiếu coverage_ratio:

severity_map vẫn chạy dựa vào “độ thiếu draft” + cờ reverse/silhouette, nhưng sẽ kém chính xác khi phân biệt “điểm nhỏ” vs “vùng lớn”.

Đại lượng chuẩn hoá
3.1. Draft deficit (độ thiếu draft so với ngưỡng)

deficit_deg = max(0, threshold_deg - draft_min_deg_region)

deficit_ratio = deficit_deg / max(threshold_deg, eps) (eps tránh chia 0; eps là hằng nhỏ) [CHƯA XÁC MINH]

3.2. Reverse strength

reverse_present: dot < 0 xuất hiện (boolean)

reverse_ratio = (#samples dot\<0)/(#samples valid)

3.3. Silhouette indicator

silhouette_crossing_face_count / face_count_evaluated

hoặc boolean silhouette_crossing_region (true nếu sign-change trên một face)

Severity rules đề xuất (default)
Đặt vào policy:

dfm_policy:
severity_map:
draft:
below_threshold:
deficit_deg_thresholds:
warn_min: 0.5
error_min: 1.5
fatal_min: 3.0
coverage_ratio_thresholds:
warn_min: 0.01
error_min: 0.05
fatal_min: 0.15
rules:

- if_deficit_deg_at_least: 3.0
  severity: FATAL
- if_deficit_deg_at_least: 1.5
  severity: ERROR
- if_deficit_deg_at_least: 0.5
  severity: WARN
- else: INFO

reverse_draft:
reverse_ratio_thresholds:
warn_min: 0.001
error_min: 0.01
fatal_min: 0.05
rules:

- if_reverse_ratio_at_least: 0.05
  severity: FATAL
- if_reverse_ratio_at_least: 0.01
  severity: ERROR
- if_reverse_present: true
  severity: WARN
- else: INFO

silhouette_crossing:
face_ratio_thresholds:
warn_min: 0.01
error_min: 0.05
fatal_min: 0.15
rules:

- if_face_ratio_at_least: 0.15
  severity: FATAL
- if_face_ratio_at_least: 0.05
  severity: ERROR
- if_silhouette_present: true
  severity: WARN
- else: INFO

overrides:

# Nếu reverse_draft xuất hiện ở cùng vùng với below_threshold lớn -> nâng severity

- when:
  reverse_present: true
  below_threshold_deficit_deg_at_least: 1.5
  force_severity: FATAL

5. Cách combine các loại lỗi (thực tế mold review)
   Khuyến nghị combine theo “max severity” giữa các sub-defects, sau đó apply override:

severity_region = max(
severity(below_threshold),
severity(reverse_draft),
severity(silhouette_crossing)
)

apply overrides (ví dụ reverse + thiếu nhiều -> FATAL)

Gợi ý “surface class” (để bạn nâng độ thực chiến mà không cần CAE)
Nếu đã có tagging surface_class, bạn nên dùng ngưỡng “deficit_deg_thresholds” khác nhau theo class, vì texture cần draft lớn hơn polish:

below_threshold_by_surface_class:

polish: {warn_min: 0.5, error_min: 1.5, fatal_min: 3.0}

light_texture: {warn_min: 1.0, error_min: 2.0, fatal_min: 4.0} [CHƯA XÁC MINH]

heavy_texture: {warn_min: 1.5, error_min: 3.0, fatal_min: 5.0} [CHƯA XÁC MINH]

Nếu chưa có surface_class:

dùng default như mục 4 và gắn issue SURFACE_CLASS_UNRESOLVED trong draft result/defect (để người review biết đây là “general rule”).

Output tối thiểu draft_agent cần emit để policy chạy đúng

draft_min_deg_region

applied_threshold_deg

below_threshold_coverage_ratio (hoặc violated_sample_ratio)

reverse_present hoặc reverse_ratio

silhouette_present hoặc face_ratio

Lý do kỹ thuật của các ngưỡng default

deficit 0.5°: thường là “thiếu nhẹ”, có thể chấp nhận tùy bề mặt → WARN.

deficit 1.5°: bắt đầu rõ rủi ro drag mark/stick ở nhiều dự án → ERROR.

deficit ≥3°: thường phải xử lý thật (đổi draft/parting/split/texture) → FATAL trong vai trò “pre-tooling gate”.

reverse_present: dù nhỏ cũng đáng WARN vì có nguy cơ kẹt nếu vùng nằm trên hướng mở khuôn chính; reverse_ratio tăng thì severity tăng nhanh.

**3.3 Referenced policy keys and inputs (extracted)**

Source: S1, S2

[CHƯA XÁC MINH] No policy keys could be extracted automatically from S2 text formatting.

**4. Open Questions / [CHƯA XÁC MINH]**

The following items are marked [CHƯA XÁC MINH] in sources and remain unresolved:

rule_version: "RULESET_v1" [CHƯA XÁC MINH]

hybrid_preference: "brep" [CHƯA XÁC MINH]

face_id: string|int [CHƯA XÁC MINH]

surface_class: "polish"|"light_texture"|"heavy_texture"|null [CHƯA XÁC MINH]

WARN nếu thiếu nhẹ; ERROR nếu thiếu nhiều hoặc ở vùng dài theo hướng rút (nguy cơ drag). [CHƯA XÁC MINH]

WARN/ERROR tùy dự án; mục tiêu chính là “buộc kỹ sư quyết định parting”. [CHƯA XÁC MINH]

TÀI LIỆU — DFM_POLICY_SEVERITY_MAP_DRAFT_v1
doc_id: TOOLING_DFM_ADVISOR_DFM_POLICY_SEVERITY_MAP_DRAFT_v1
status: DRAFT
effective_date_utc: [CHƯA XÁC MINH]
units: angle_unit = deg, dot dimensionless

deficit_ratio = deficit_deg / max(threshold_deg, eps) (eps tránh chia 0; eps là hằng nhỏ) [CHƯA XÁC MINH]

light_texture: {warn_min: 1.0, error_min: 2.0, fatal_min: 4.0} [CHƯA XÁC MINH]

heavy_texture: {warn_min: 1.5, error_min: 3.0, fatal_min: 5.0} [CHƯA XÁC MINH]

**5. Change Log (Editorial)**

20260304T095853Z: Consolidated S1+S2 into WS_AGENTS-DRF-POL-v1 (REVIEW). Renamed under WS governance. No semantics change; content retained as verbatim extracts with traceability.

**6. Appendix (aka mapping / notes)**

If upstream documents use alternate naming for the same concepts (e.g., defect_type taxonomy, rule_version naming), this document preserves the original text and only provides 'aka' mapping when explicitly present in sources.
