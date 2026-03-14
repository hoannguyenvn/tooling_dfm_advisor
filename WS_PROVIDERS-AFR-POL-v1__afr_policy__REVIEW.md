**tooling_dfm_advisor**

# WS_PROVIDERS-AFR-POL-v1

Canonical policy consolidation (document governance only; no runtime/pipeline changes).

## 0. Document Control Block

|                                  |                                                                      |
| -------------------------------- | -------------------------------------------------------------------- |
| Canonical Code                   | WS_PROVIDERS-AFR-POL-v1                                              |
| Status                           | REVIEW                                                               |
| Run ID (UTC)                     | 20260304T092047Z                                                     |
| WS / Module / Doc Type / Version | WS_PROVIDERS / AFR / POL / v1                                        |
| Created date (UTC)               | 20260304T092047Z                                                     |
| Purpose                          | Document governance consolidation only; no runtime/pipeline changes. |

## 1. Scope

This document consolidates operational policy, thresholds, and conventions for afr_provider V1 (precision-first) based strictly on the provided sources.

Non-goals: no new algorithms; no architecture redesign; no changes to module contracts; no new thresholds beyond what appears in sources.

## 2. Included Sources & Mapping Table

|           |                                      |                                 |                         |                                                                                            |                                                                |
| --------- | ------------------------------------ | ------------------------------- | ----------------------- | ------------------------------------------------------------------------------------------ | -------------------------------------------------------------- |
| source_id | filename                             | original_title                  | original_version/status | sections_used                                                                              | notes                                                          |
| S1        | AFR V1 Precision-First Protocol.docx | AFR V1 Precision-First Protocol | [CHƯA XÁC MINH]         | Policy intent, tag set, evidence rules, confidence rules, recommended keys, open questions | Narrative protocol; includes some [CHƯA XÁC MINH] items inside |
| S2        | afr_provider_starter defaults.docx   | afr_provider starter defaults   | [CHƯA XÁC MINH]         | Starter defaults JSON for dfm_policy.thresholds.afr                                        | Authoritative default values snapshot                          |

## 3. Consolidated Policy Content

### 3.1 Precision-first policy principles

Source: S1

Nguyên tắc precision-first (bất biến)

“Không đủ bằng chứng” ⇒ không emit tag; thay bằng issue dạng AFR_TAG_SUPPRESSED_LOW_CONFIDENCE.

Mỗi tag phải có “multi-evidence”: ít nhất 2–3 dấu hiệu độc lập (surface type + classifier + sanity checks).

Không có classifier/không có thickness_result ⇒ giảm scope, không cố tag bằng proxy yếu.

Tag nào emit ra phải có confidence_score >= min_confidence_to_emit_tag (policy). Nếu không đạt, drop.

Tag set V1 (giữ ít, chắc)
Giữ 4 tag như đã đề xuất:

HOLE_CYLINDER_CANDIDATE

### 3.2 Tag set V1 and emission gating

Source: S1

Tag set V1 (giữ ít, chắc)
Giữ 4 tag như đã đề xuất:

HOLE_CYLINDER_CANDIDATE

BOSS_CYLINDER_CANDIDATE

RIB_PLATE_CANDIDATE

THICK_JUNCTION_CANDIDATE
Ngoài ra, “CYLINDER_FACE_UNCLASSIFIED” có thể dùng nội bộ để thống kê nhưng không emit ra report V1 nếu chưa phân loại được (để tránh nhiễu).

3.1. HOLE_CYLINDER_CANDIDATE / BOSS_CYLINDER_CANDIDATE (precision-first)
Evidence set tối thiểu (đề xuất):
A) Surface evidence:

3.2. RIB_PLATE_CANDIDATE (precision-first)
Rib tag dễ bị false positive nếu chỉ nhìn shape, nên V1 precision-first yêu cầu: “có thickness_result tốt” hoặc probing đủ tin.

3.3. THICK_JUNCTION_CANDIDATE (precision-first)
Tag này “dễ đúng” nếu dựa trên thickness_result V1 (local probe), nên có thể emit nhiều hơn rib.

### 3.3 Evidence and thresholds for HOLE/BOSS

Source: S1

HOLE_CYLINDER_CANDIDATE

BOSS_CYLINDER_CANDIDATE

RIB_PLATE_CANDIDATE

THICK_JUNCTION_CANDIDATE
Ngoài ra, “CYLINDER_FACE_UNCLASSIFIED” có thể dùng nội bộ để thống kê nhưng không emit ra report V1 nếu chưa phân loại được (để tránh nhiễu).

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

Quy tắc emit tag theo từng loại (cụ thể, để giảm false positive)

3.1. HOLE_CYLINDER_CANDIDATE / BOSS_CYLINDER_CANDIDATE (precision-first)
Evidence set tối thiểu (đề xuất):
A) Surface evidence:

Face type = CYLINDER (GeomAbs_Cylinder) và radius_mm >= min_cylinder_radius_mm (loại bỏ micro-fillet/degenerate).

B) Orientation/inside-outside evidence (bắt buộc để phân HOLE vs BOSS):

Thực hiện inside/outside probe bằng SolidClassifier (P±eps\*n).

Kết quả phải “ổn định” khi thử ít nhất 2 điểm mẫu trên cùng cylinder face (để tránh classifier nhiễu ở mép/cusp).

Nếu 2 điểm cho kết quả không nhất quán ⇒ không emit HOLE/BOSS, chỉ issue AMBIGUOUS_CLASSIFICATION.

C) Sanity checks (để tránh tag nhầm)

Cylinder face area / param span đủ lớn (>= min_cylinder_face_area_mm2) [CHƯA XÁC MINH: bạn muốn dùng area threshold hay dùng length proxy].

Axis stability: axis_dir không biến thiên (vì cylinder analytic nên ổn).

Quy tắc quyết định:

Nếu classifier nói “material is inside theo phía normal” (tùy convention orientation) thì map HOLE/BOSS theo rule cố định, và rule đó phải được ghi trong evidence để audit. Nếu không chắc convention, không emit. [CHƯA XÁC MINH: convention orientation của solid trong pipeline bạn đã chốt chưa]

Precision-first fallback:

### 3.4 Evidence and thresholds for RIB

Source: S1

RIB_PLATE_CANDIDATE

THICK_JUNCTION_CANDIDATE
Ngoài ra, “CYLINDER_FACE_UNCLASSIFIED” có thể dùng nội bộ để thống kê nhưng không emit ra report V1 nếu chưa phân loại được (để tránh nhiễu).

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

Quy tắc emit tag theo từng loại (cụ thể, để giảm false positive)

3.1. HOLE_CYLINDER_CANDIDATE / BOSS_CYLINDER_CANDIDATE (precision-first)
Evidence set tối thiểu (đề xuất):
A) Surface evidence:

Face type = CYLINDER (GeomAbs_Cylinder) và radius_mm >= min_cylinder_radius_mm (loại bỏ micro-fillet/degenerate).

B) Orientation/inside-outside evidence (bắt buộc để phân HOLE vs BOSS):

Thực hiện inside/outside probe bằng SolidClassifier (P±eps\*n).

Kết quả phải “ổn định” khi thử ít nhất 2 điểm mẫu trên cùng cylinder face (để tránh classifier nhiễu ở mép/cusp).

Nếu 2 điểm cho kết quả không nhất quán ⇒ không emit HOLE/BOSS, chỉ issue AMBIGUOUS_CLASSIFICATION.

C) Sanity checks (để tránh tag nhầm)

Cylinder face area / param span đủ lớn (>= min_cylinder_face_area_mm2) [CHƯA XÁC MINH: bạn muốn dùng area threshold hay dùng length proxy].

Axis stability: axis_dir không biến thiên (vì cylinder analytic nên ổn).

Quy tắc quyết định:

Nếu classifier nói “material is inside theo phía normal” (tùy convention orientation) thì map HOLE/BOSS theo rule cố định, và rule đó phải được ghi trong evidence để audit. Nếu không chắc convention, không emit. [CHƯA XÁC MINH: convention orientation của solid trong pipeline bạn đã chốt chưa]

Precision-first fallback:

Nếu classifier API không available hoặc OCC thiếu ⇒ không emit HOLE/BOSS tags; chỉ giữ topo metrics V0 + issue AFR_CLASSIFIER_UNAVAILABLE.

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

### 3.5 Evidence and thresholds for THICK_JUNCTION

Source: S1

THICK_JUNCTION_CANDIDATE
Ngoài ra, “CYLINDER_FACE_UNCLASSIFIED” có thể dùng nội bộ để thống kê nhưng không emit ra report V1 nếu chưa phân loại được (để tránh nhiễu).

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

Quy tắc emit tag theo từng loại (cụ thể, để giảm false positive)

3.1. HOLE_CYLINDER_CANDIDATE / BOSS_CYLINDER_CANDIDATE (precision-first)
Evidence set tối thiểu (đề xuất):
A) Surface evidence:

Face type = CYLINDER (GeomAbs_Cylinder) và radius_mm >= min_cylinder_radius_mm (loại bỏ micro-fillet/degenerate).

B) Orientation/inside-outside evidence (bắt buộc để phân HOLE vs BOSS):

Thực hiện inside/outside probe bằng SolidClassifier (P±eps\*n).

Kết quả phải “ổn định” khi thử ít nhất 2 điểm mẫu trên cùng cylinder face (để tránh classifier nhiễu ở mép/cusp).

Nếu 2 điểm cho kết quả không nhất quán ⇒ không emit HOLE/BOSS, chỉ issue AMBIGUOUS_CLASSIFICATION.

C) Sanity checks (để tránh tag nhầm)

Cylinder face area / param span đủ lớn (>= min_cylinder_face_area_mm2) [CHƯA XÁC MINH: bạn muốn dùng area threshold hay dùng length proxy].

Axis stability: axis_dir không biến thiên (vì cylinder analytic nên ổn).

Quy tắc quyết định:

Nếu classifier nói “material is inside theo phía normal” (tùy convention orientation) thì map HOLE/BOSS theo rule cố định, và rule đó phải được ghi trong evidence để audit. Nếu không chắc convention, không emit. [CHƯA XÁC MINH: convention orientation của solid trong pipeline bạn đã chốt chưa]

### 3.6 Policy keys catalog and starter defaults

Source: S1, S2

Recommended policy keys (as described in S1) and the current starter defaults snapshot (S2).

|                                                               |               |        |
| ------------------------------------------------------------- | ------------- | ------ |
| policy_key                                                    | default_value | Source |
| thresholds.afr.enabled                                        | True          | S2     |
| thresholds.afr.max_tags_total                                 | 30            | S2     |
| thresholds.afr.min_confidence_to_emit_tag                     | 0.8           | S2     |
| thresholds.afr.classifier_probe_eps_mm                        | 0.2           | S2     |
| thresholds.afr.min_cylinder_radius_mm                         | 0.6           | S2     |
| thresholds.afr.min_cylinder_face_area_mm2                     | 8.0           | S2     |
| thresholds.afr.depends_on_thickness.max_miss_rate_for_afr     | 0.08          | S2     |
| thresholds.afr.depends_on_thickness.min_sample_count_for_afr  | 300           | S2     |
| thresholds.afr.rib_candidate.rib_ratio_emit_min               | 0.35          | S2     |
| thresholds.afr.rib_candidate.rib_ratio_emit_max               | 0.75          | S2     |
| thresholds.afr.rib_candidate.rib_aspect_ratio_min             | 6.0           | S2     |
| thresholds.afr.thick_junction.junction_thick_ratio_min_to_tag | 0.6           | S2     |

Starter defaults JSON (verbatim). Source: S2

{
"dfm_policy": {
"thresholds": {
"afr": {
"enabled": true,

"max_tags_total": 30,
"min_confidence_to_emit_tag": 0.80,

"classifier_probe_eps_mm": 0.20,

"min_cylinder_radius_mm": 0.60,
"min_cylinder_face_area_mm2": 8.0,

"depends_on_thickness": {
"max_miss_rate_for_afr": 0.08,
"min_sample_count_for_afr": 300
},

"rib_candidate": {
"rib_ratio_emit_min": 0.35,
"rib_ratio_emit_max": 0.75,
"rib_aspect_ratio_min": 6.0
},

"thick_junction": {
"junction_thick_ratio_min_to_tag": 0.60
}
}
}
}
}

## 4. Open Questions / [CHƯA XÁC MINH]

Source: S1

Cylinder face area / param span đủ lớn (>= min_cylinder_face_area_mm2) [CHƯA XÁC MINH: bạn muốn dùng area threshold hay dùng length proxy].

Nếu classifier nói “material is inside theo phía normal” (tùy convention orientation) thì map HOLE/BOSS theo rule cố định, và rule đó phải được ghi trong evidence để audit. Nếu không chắc convention, không emit. [CHƯA XÁC MINH: convention orientation của solid trong pipeline bạn đã chốt chưa]

t_rib_mm \<= rib_max_absolute_mm (optional, để tránh tag nhầm “khối dày” thành rib). [CHƯA XÁC MINH]

Candidate patch có tính “plate-like”: span lớn hơn chiều dày nhiều lần (span/thickness >= rib_aspect_ratio_min). Nếu chưa có span, dùng proxy “cluster_sample_count” từ undercut/draft sampling cũng được nhưng phải ghi limitation. [CHƯA XÁC MINH: bạn muốn dùng span_score hay sample-count proxy]

min_cylinder_face_area_mm2 (float, default 5.0) [CHƯA XÁC MINH]

## 5. Change Log (Editorial)

20260304T092047Z: Consolidated AFR policy sources (S1-S2) into canonical POL. No semantics change; only document governance packaging, headings normalization, and traceability annotations.

## 6. Appendix

Naming conventions: module implementation names use snake_case (afr_provider). Contracts/envelopes use UPPER_CASE versioned identifiers where applicable.
