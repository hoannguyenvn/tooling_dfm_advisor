**WS_AGENTS-UND-POL-v1 — undercut policy (document governance only)**

# 0. Document Control Block

|                                  |                                                                      |
| -------------------------------- | -------------------------------------------------------------------- |
| Canonical Code                   | WS_AGENTS-UND-POL-v1                                                 |
| Status                           | REVIEW                                                               |
| Run ID                           | 20260304T104808Z                                                     |
| WS / Module / Doc Type / Version | WS_AGENTS / UND / POL / v1                                           |
| Created date (UTC)               | 20260304T104808Z                                                     |
| Purpose                          | Document governance consolidation only; no runtime/pipeline changes. |

## 1. Scope

*Source: S1*

Mục tiêu

Ánh xạ kết quả undercut (ứng viên vùng ngược hướng mở khuôn) thành severity INFO/WARN/ERROR/FATAL.

Dùng cho 2 lớp cảnh báo:
A) UNDERCUT_CANDIDATE (phát hiện vùng undercut)
B) UNDERCUT_ANALYSIS_LOW_CONFIDENCE / UNDERCUT_NOT_COVERED (độ tin cậy thấp do planar-only / thiếu coverage / OCC missing)

Non-goals: không thay đổi thuật toán undercut_agent; không thiết kế lại pipeline; không thay đổi contract semantics; không bổ sung quy tắc mới ngoài nguồn. Nếu cần nội dung mới để hoàn thiện template sẽ ghi [CHƯA XÁC MINH].

## 2. Included Sources & Mapping Table

|           |                                                     |                                                |                         |                                                       |          |
| --------- | --------------------------------------------------- | ---------------------------------------------- | ----------------------- | ----------------------------------------------------- | -------- |
| source_id | filename                                            | original_title                                 | original_version/status | sections_used                                         | notes    |
| S1        | TÀI LIỆU — DFM_POLICY_SEVERITY_MAP_UNDERCUT_v1.docx | TÀI LIỆU — DFM_POLICY_SEVERITY_MAP_UNDERCUT_v1 | DRAFT                   | severity map + thresholds + rules (verbatim extracts) | primary  |
| S2        | Undercut_Starter defaults (điền vào policy).docx    | Undercut_Starter defaults (điền vào policy)    | [CHƯA XÁC MINH]         | starter defaults JSON (verbatim)                      | defaults |

## 3. Consolidated Policy Content

## 3.1 Document header + mục tiêu

*Source: S1*

TÀI LIỆU — DFM_POLICY_SEVERITY_MAP_UNDERCUT_v1
doc_id: TOOLING_DFM_ADVISOR_DFM_POLICY_SEVERITY_MAP_UNDERCUT_v1
status: DRAFT
effective_date_utc: [CHƯA XÁC MINH]
units: dot dimensionless; angles (nếu dùng) là deg

Mục tiêu

Ánh xạ kết quả undercut (ứng viên vùng ngược hướng mở khuôn) thành severity INFO/WARN/ERROR/FATAL.

Dùng cho 2 lớp cảnh báo:
A) UNDERCUT_CANDIDATE (phát hiện vùng undercut)
B) UNDERCUT_ANALYSIS_LOW_CONFIDENCE / UNDERCUT_NOT_COVERED (độ tin cậy thấp do planar-only / thiếu coverage / OCC missing)

## 3.2 Inputs undercut_agent cần cung cấp

*Source: S1*

Inputs undercut_agent cần cung cấp
Required:

ejection_direction (unit vector, user_confirmed=true)

per-face/per-region:

dot_min (min dot của các điểm mẫu trên face/region) hoặc “is_undercut_candidate” boolean

undercut_candidate_count
Optional nhưng rất nên có:

undercut_coverage_ratio (violated_area/evaluated_area) hoặc violated_sample_ratio

clustering metrics:

undercut_cluster_count

largest_cluster_ratio (largest_cluster_area / violated_area) [CHƯA XÁC MINH]

surface_coverage:

faces_planar_evaluated / faces_total (để biết “planar-only bias”)

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

Nếu chưa có clustering/area:

severity_map dùng candidate_count + violated_sample_ratio.

## 3.3 Đại lượng chuẩn hoá và công thức

*Source: S1*

Đại lượng chuẩn hoá
3.1. Undercut depth proxy (mức “ngược”)

undercut_depth = max(0, -dot_min) (dot_min âm càng sâu → undercut mạnh)
Giải thích: dot = n·p; dot\<0 nghĩa là normal ngược hướng pull, càng âm càng “ngược mạnh”.

3.2. Coverage ratio

coverage_ratio = undercut_area / evaluated_area (0..1) [CHƯA XÁC MINH]
Fallback: violated_sample_ratio

3.3. Continuity proxy (nếu có)

largest_cluster_ratio (0..1): undercut tập trung thành dải lớn → nặng hơn undercut lẻ tẻ.

## 3.4 Severity rules (default) + combine logic

*Source: S1*

Severity rules đề xuất (default)
Đặt vào policy:

dfm_policy:
severity_map:
undercut:
candidate:
depth_thresholds:
warn_min: 0.01
error_min: 0.10
fatal_min: 0.30
coverage_ratio_thresholds:
warn_min: 0.001
error_min: 0.01
fatal_min: 0.05
rules:

- if_depth_at_least: 0.30
  severity: FATAL
- if_depth_at_least: 0.10
  severity: ERROR
- if_depth_at_least: 0.01
  severity: WARN
- else: INFO

continuity_overrides:

## Nếu undercut tuy không “sâu” nhưng chạy thành dải lớn -> tăng severity

- when:
  largest_cluster_ratio_at_least: 0.50
  coverage_ratio_at_least: 0.01
  force_severity: ERROR
- when:
  largest_cluster_ratio_at_least: 0.70
  coverage_ratio_at_least: 0.05
  force_severity: FATAL

planar_only_bias:

## Nếu agent đang planar-only (V0) hoặc coverage mặt cong thấp -> cảnh báo độ tin cậy

face_planar_ratio_thresholds:
warn_max: 0.60
error_max: 0.30
rules:

- if_face_planar_ratio_below: 0.30
  severity: ERROR
  code: UNDERCUT_ANALYSIS_LOW_COVERAGE_NONPLANAR
- if_face_planar_ratio_below: 0.60
  severity: WARN
  code: UNDERCUT_ANALYSIS_LOW_COVERAGE_NONPLANAR

fail_safe:

- if_occ_missing: true
  severity: WARN
  code: UNDERCUT_NOT_EXECUTED_OCC_MISSING

5. Cách combine severity trong thực tế

Base severity theo undercut_depth (mức âm của dot_min).

Escalate theo coverage_ratio.

Override theo continuity (largest_cluster_ratio) nếu có.

Đồng thời phát “low confidence” nếu phân tích chỉ planar hoặc coverage mặt cong quá thấp (để không tạo false sense of safety).

Logic combine khuyến nghị (policy-side):

severity = base_severity(depth)

if coverage_ratio >= coverage.error_min then severity = max(severity, ERROR)

if coverage_ratio >= coverage.fatal_min then severity = max(severity, FATAL)

apply continuity_overrides

emit additional issue if planar_only_bias triggers (đây là issue về confidence, không thay cho undercut finding)

Output tối thiểu undercut_agent nên emit để map hoạt động tốt

dot_min_global hoặc dot_min_per_worst_region

undercut_candidate_count

violated_sample_ratio (nếu chưa có area)

faces_planar_evaluated, faces_total (để planar bias warning)

Giải thích ngưỡng default (để bạn chỉnh theo kinh nghiệm khuôn)

undercut_depth 0.01: gần “gần vuông” với pull, thường có thể là noise/sampling/fillet nhỏ → WARN.

0.10: undercut rõ ràng → thường phải đổi split hoặc thêm cơ cấu → ERROR.

0.30: undercut mạnh (normal gần ngược hoàn toàn) → gần như chắc chắn cần side action hoặc redesign → FATAL.

coverage_ratio 0.05: undercut trải rộng → rủi ro chi phí khuôn rất cao (nhiều slide/lifter hoặc split phức tạp).

## 3.5 Starter defaults (JSON) để điền vào policy

*Source: S2*

Undercut_Starter defaults (điền vào policy)

{
"dfm_policy": {
"thresholds": {
"undercut": {
"reverse_dot_threshold": 0.0,

"clustering": {
"uv_eps": 0.12,
"xyz_eps_mm": 1.0,
"min_cluster_samples": 6,
"max_clusters_to_report": 10
},

"sapi": {
"depth_ref": 0.30,
"coverage_ref": 0.05,
"count_ref": 30,

"weights": {
"w_size": 0.50,
"w_depth": 0.40,
"w_continuity": 0.10
},

"thresholds": {
"review": 0.35,
"side_action_likely": 0.60
}
}
}
},

"common": {
"sampling_budget": {
"max_faces_to_report": 50,
"max_worst_regions": 10
}
}
}
}

## 4. Open Questions / [CHƯA XÁC MINH]

*Source: S1*

TÀI LIỆU — DFM_POLICY_SEVERITY_MAP_UNDERCUT_v1
doc_id: TOOLING_DFM_ADVISOR_DFM_POLICY_SEVERITY_MAP_UNDERCUT_v1
status: DRAFT
effective_date_utc: [CHƯA XÁC MINH]
units: dot dimensionless; angles (nếu dùng) là deg

Ảnh hưởng: nếu không chốt được điểm này, policy có thể thiếu trường/thiếu ngưỡng cho mapping hoặc gây khó khăn cho audit traceability.

*Source: S1*

largest_cluster_ratio (largest_cluster_area / violated_area) [CHƯA XÁC MINH]

Ảnh hưởng: nếu không chốt được điểm này, policy có thể thiếu trường/thiếu ngưỡng cho mapping hoặc gây khó khăn cho audit traceability.

*Source: S1*

coverage_ratio = undercut_area / evaluated_area (0..1) [CHƯA XÁC MINH]
Fallback: violated_sample_ratio

Ảnh hưởng: nếu không chốt được điểm này, policy có thể thiếu trường/thiếu ngưỡng cho mapping hoặc gây khó khăn cho audit traceability.

## 5. Change Log (Editorial)

Thay đổi biên tập/tái cấu trúc để chuẩn hoá theo Document Governance Layer; không đổi semantics. Bổ sung Document Control Block + Mapping Table; sắp xếp lại nội dung thành các mục 3.1–3.5; thêm traceability “Source: Sx” cho mọi subsection.

## 6. Appendix

Cross references (canonical codes): WS_MODEL-RGN-POL-v1 (REGION_REF_CONVENTION_v1); WS_MODEL-GEN-SPC-v1 (models/contracts baseline).
