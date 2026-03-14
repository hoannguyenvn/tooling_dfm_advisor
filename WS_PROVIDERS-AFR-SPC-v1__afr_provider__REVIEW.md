WS_PROVIDERS-AFR-SPC-v1 — afr_provider — Consolidated Specification

Status: REVIEW | Run ID (UTC): 20260304T090817Z

# 0. Document Control Block

**Canonical Code:** WS_PROVIDERS-AFR-SPC-v1

**Status:** REVIEW

**Run ID:** 20260304T090817Z

**Created date (UTC):** 20260304T090817Z

**WS / Module / Doc Type / Version:** WS_PROVIDERS / AFR / SPC / v1

**Purpose:** Document governance consolidation only; no runtime/pipeline changes.

## 1. Scope

Source: S1, S2, S3, S4 (verbatim extracts; no semantics change).

## 1.1 Non-goals

Source: S1/S2/S3 (verbatim extracts).

## 2. Included Sources & Mapping Table

|           |                                                             |                                    |                                             |                                          |                                                           |
| --------- | ----------------------------------------------------------- | ---------------------------------- | ------------------------------------------- | ---------------------------------------- | --------------------------------------------------------- |
| source_id | filename                                                    | original_title                     | original_version/status                     | sections_used                            | notes                                                     |
| S1        | TÀI LIỆU TRIỂN KHAI ADN AMBR LDR — MODULE afr_provider.docx | MODULE afr_provider (ADN/AMBR/LDR) | DRAFT (per source)                          | ADN/AMBR/LDR extracts; scope/non-goals   | Primary module compilation document                       |
| S2        | ADN_afr_provider_topology_proxy_feature_recognition_v1.docx | ADN afr_provider topology proxy    | REVIEW (per source)                         | ADN (Algorithm/Logic) core               | Authority for topology-proxy + fail-safe envelope         |
| S3        | AMBR_afr_provider_v1.docx                                   | AMBR afr_provider v1               | REVIEW (per source)                         | AMBR (Bindings/Execution/Deps) core      | Authority for bindings + dependency statements            |
| S4        | AFR V1 Precision-First Protocol.docx                        | AFR V1 Precision-First Protocol    | [CHƯA XÁC MINH] (not explicit in file name) | ADN addendum (precision-first tag rules) | Supplemental protocol; does not override source semantics |

## 3. Consolidated Specification Content

## 3.1 ADN (Algorithm / Logic)

**Source: S2 — ADN_afr_provider_topology_proxy_feature_recognition_v1.docx**

ADN — afr_provider — Topology Proxy AFR + Fail-safe Envelope — v1

Status: REVIEW (V1 candidate). DFM-first: proxy AFR used for pipeline robustness, not detailed feature classification.

1. Identification

adn_id: ADN_afr_provider_topology_proxy_feature_recognition_v1

module_name: afr_provider

module_type: provider

algorithm_name: topology_proxy_feature_recognition

algorithm_version: v1

result_envelope: AFR_RESULT_v0 (schema locked by this V1 record)

owner: [CHƯA XÁC MINH]

authority_refs: Master Spec + Governance Spec + Addendum V1

2. Why

afr_provider cung cấp ‘geometry intelligence’ mức proxy để: (a) phân loại độ phức tạp hình học nhanh, (b) đóng góp cho afr_state và analysis_completeness, và (c) đảm bảo pipeline không sập khi thiếu pythonocc-core bằng cơ chế NOT_EXECUTED/STUB_MODE.

3. Preconditions

Input payload phải cung cấp run_id, rule_version, geometry_version_id (tối thiểu) để truy vết.

Không được mutate hình học, không healing, không IO, không ghi file.

Thiếu pythonocc-core: phải trả envelope NOT_EXECUTED (hoặc STUB_MODE legacy) thay vì raise unhandled exception.

Determinism: cùng shape đầu vào → cùng topo metrics + proxy classification.

4. Inputs

Preferred typed input (DFM-first, governance-by-contract):

GeoModelResponse (GEO_MODEL_API_v1) — reads: geometry context / brep handle (opaque ref) + metadata.

Legacy compatibility (allowed only if upstream chưa refactor contracts):

dict payload with keys: geometry_context_in / geometry / agl_ref / dgl_ref / mapping_result / trace_id [CHƯA XÁC MINH exact key path].

5. Actions / Algorithm

Initialize AFR_RESULT_v0 envelope with status=INITIALIZED; fill run_id/rule_version/geometry_version_id if present.

Extract B-Rep shape handle from input (opaque). If missing → status=FAILED, errors=[BREP_SHAPE_MISSING], return.

Dependency probe: lazy-import OCC.Core.TopExp and OCC.Core.TopAbs. If import fails → status=NOT_EXECUTED, errors=[NOT_SUPPORTED_NO_GEOM_LIB], set degraded_mode=true, return.

Topology scan: use TopExp_Explorer to count faces, edges, vertices.

Proxy classification rule: if face_count \<= 6 → features_proxy=simple_prismatic_proxy else complex_topology_proxy.

Pack results: status=EXECUTED; metrics={face_count, edge_count, vertex_count}; warnings optional.

6. Outputs (AFR_RESULT_v0 — schema locked)

Minimum required fields:

status: INITIALIZED | EXECUTED | NOT_EXECUTED | FAILED

run_id: str (YYYYMMDDThhmmssZ)

rule_version: str

geometry_version_id: str

metrics: {face_count:int, edge_count:int, vertex_count:int}

features_proxy: simple_prismatic_proxy | complex_topology_proxy | unknown_proxy

errors: list[{code:str, message:str, retryable:bool}]

warnings: list[{code:str, message:str}] (optional)

metadata: {trace_id?:str, agl_ref?:str, dgl_ref?:str} (no filesystem paths)

degraded_mode: bool (true iff status!=EXECUTED)

7. Mapping to global states (downstream contract)

afr_state mapping (recommended):

- EXECUTED → afr_state=AFR_VERIFIED

- NOT_EXECUTED/FAILED → afr_state=AFR_UNCERTAIN; analysis_completeness may downgrade [CHƯA XÁC MINH exact policy].

This module DOES NOT mutate global_state; it only emits envelope for orchestrator to map.

8. Failure modes (structured)

NOT_SUPPORTED_NO_GEOM_LIB: OCC import fail → status=NOT_EXECUTED; retryable=false.

BREP_SHAPE_MISSING: input missing shape handle → status=FAILED; retryable=false.

GEOM_CONTEXT_MISSING: missing geometry context → status=FAILED; retryable=false.

RETRYABLE_DEP_FAILURE: transient import/load failure → status=NOT_EXECUTED; retryable=true [CHƯA XÁC MINH probe criteria].

9. Evidence / Verification

Unit test: OCC absent → status=NOT_EXECUTED, code=NOT_SUPPORTED_NO_GEOM_LIB, no exception escapes.

Unit test: missing shape → status=FAILED with BREP_SHAPE_MISSING.

Unit test: stable counts for a fixed golden STEP shape (face/edge/vertex) and proxy classification threshold.

Static check: envelope string fields do NOT contain absolute paths.

10. Fail-fast

Provider must never throw unhandled exception outward; convert to envelope errors.

Forbidden: silent success (empty dict). status must reflect execution outcome.

11. Troubleshooting

Always NOT_EXECUTED: verify pythonocc-core availability/pinning; ensure lazy import path correct.

BREP_SHAPE_MISSING: check upstream contract that should provide brep handle; confirm payload wiring.

Non-deterministic counts: ensure input shape is frozen and stable within geometry_version_id.

12. Non-goals

Không nhận dạng feature chi tiết (draft/rib/boss/etc.).

Không healing, không sửa B-Rep, không mesh.

Không quyết định report rendering; reporter chỉ đọc kết quả đã đóng gói.

13. Filled example (run_id=20260212T071350Z)

AFR_RESULT_v0 example: status=EXECUTED, run_id=20260212T071350Z, rule_version=ruleset_2026_02_12_v1, geometry_version_id=geom_9f3a..., metrics={face_count:12,edge_count:36,vertex_count:24}, features_proxy=complex_topology_proxy, degraded_mode=false.

**Source: S4 — AFR V1 Precision-First Protocol.docx**

Dưới đây là “AFR V1 Precision-First Protocol” để mình khóa lại thiết kế V1 (không CAE, không full feature graph), tập trung vào 4 tag tối thiểu và cách kiểm soát false positive.

Nguyên tắc precision-first (bất biến)

“Không đủ bằng chứng” ⇒ không emit tag; thay bằng issue dạng AFR_TAG_SUPPRESSED_LOW_CONFIDENCE.

Mỗi tag phải có “multi-evidence”: ít nhất 2–3 dấu hiệu độc lập (surface type + classifier + sanity checks).

Không có classifier/không có thickness_result ⇒ giảm scope, không cố tag bằng proxy yếu.

Tag nào emit ra phải có confidence_score >= min_confidence_to_emit_tag (policy). Nếu không đạt, drop.

Tag set V1 (giữ ít, chắc)
Giữ 4 tag như đã đề xuất:

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

Nếu classifier API không available hoặc OCC thiếu ⇒ không emit HOLE/BOSS tags; chỉ giữ topo metrics V0 + issue AFR_CLASSIFIER_UNAVAILABLE.

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

3.2. RIB_PLATE_CANDIDATE (precision-first)
Rib tag dễ bị false positive nếu chỉ nhìn shape, nên V1 precision-first yêu cầu: “có thickness_result tốt” hoặc probing đủ tin.

Evidence set tối thiểu:
A) Dependency evidence:

thickness_result present AND thickness_miss_rate \<= max_miss_rate_for_afr (policy), sample_count đủ. Nếu không đạt ⇒ không tag rib, issue AFR_DEPENDS_ON_THICKNESS_LOW_COVERAGE.

B) Thickness evidence:

Có cặp (t_rib_mm, t_wall_mm) đo được ổn định (>=2 sample points) và:

t_rib_mm / t_wall_mm nằm trong “candidate band” (ví dụ 0.25–0.90 để bắt rib, nhưng chỉ emit khi vượt “confidence band” hẹp hơn, ví dụ 0.35–0.80).

t_rib_mm \<= rib_max_absolute_mm (optional, để tránh tag nhầm “khối dày” thành rib). [CHƯA XÁC MINH]

C) Shape evidence:

Candidate patch có tính “plate-like”: span lớn hơn chiều dày nhiều lần (span/thickness >= rib_aspect_ratio_min). Nếu chưa có span, dùng proxy “cluster_sample_count” từ undercut/draft sampling cũng được nhưng phải ghi limitation. [CHƯA XÁC MINH: bạn muốn dùng span_score hay sample-count proxy]

Precision-first rule:

Chỉ emit rib tag khi đủ A+B+C. Thiếu 1 trong 3 ⇒ không emit.

3.3. THICK_JUNCTION_CANDIDATE (precision-first)
Tag này “dễ đúng” nếu dựa trên thickness_result V1 (local probe), nên có thể emit nhiều hơn rib.

Evidence set:

thickness_result present AND thickness_miss_rate \<= max_miss_rate_for_afr

thick_ratio = (t_local − t_nominal)/t_nominal >= junction_thick_ratio_min_to_tag

region_ref rõ (FACE/PATCH) theo convention v1

Precision-first rule:

Nếu policy cho junction_thick_ratio_min_to_tag chưa có ⇒ không emit tag; issue POLICY_UNRESOLVED (để tránh tag bừa).

Confidence scoring (đơn giản, minh bạch)
Khuyến nghị dùng “min-of-evidence” (để tránh 1 evidence mạnh che lấp 1 evidence yếu):

confidence = min(conf_surface, conf_classifier, conf_sanity) cho HOLE/BOSS

confidence = min(conf_thickness_quality, conf_ratio, conf_aspect) cho RIB

confidence = min(conf_thickness_quality, conf_thick_ratio) cho JUNCTION

Và chỉ emit tag nếu confidence >= min_confidence_to_emit_tag.

Các policy keys tối thiểu để vận hành precision-first (khuyến nghị đưa vào catalog)
Nếu bạn đồng ý, nhóm keys sau nên được thêm dưới dfm_policy.thresholds.afr (giống validation/healing vừa làm), để không hard-code:

enabled (bool, default true)

min_confidence_to_emit_tag (float, default 0.75)

classifier_probe_eps_mm (float, default 0.20)

min_cylinder_radius_mm (float, default 0.50)

min_cylinder_face_area_mm2 (float, default 5.0) [CHƯA XÁC MINH]

max_tags_total (int, default 50)

depends_on_thickness:

max_miss_rate_for_afr (float, default 0.10)

min_sample_count_for_afr (int, default 200)

rib_candidate:

rib_ratio_emit_min (float, default 0.35)

rib_ratio_emit_max (float, default 0.80)

rib_aspect_ratio_min (float, default 5.0)

thick_junction:

junction_thick_ratio_min_to_tag (float, default 0.50)

Output behavior thay đổi khi “precision-first”

afr_provider vẫn giữ core V0: topo counts + simple/complex proxy + fail-safe STUB_MODE.

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

V1: feature_tags sẽ ít hơn rõ rệt, nhưng mỗi tag có:

evidence list rõ ràng

confidence_score cao

region_ref chuẩn (FACE/PATCH)

Khi tag bị suppress: không emit tag; chỉ issue kiểu:

AFR_TAG_SUPPRESSED_LOW_CONFIDENCE

AFR_CLASSIFIER_UNAVAILABLE

AFR_DEPENDS_ON_THICKNESS_LOW_COVERAGE

**Source: S1 — TÀI LIỆU TRIỂN KHAI ADN AMBR LDR — MODULE afr_provider.docx**

TÀI LIỆU TRIỂN KHAI ADN / AMBR / LDR — MODULE afr_provider (DFM-first V1)

THÔNG TIN PHIÊN BẢN (DOCUMENT METADATA)
doc_id: TOOLING_DFM_ADVISOR_AFR_PROVIDER_DFM_FIRST_V1_RESEARCH_v1
status: DRAFT
effective_date_utc: [CHƯA XÁC MINH]
owner: [CHƯA XÁC MINH]
baseline facts: afr_provider hiện V0 chủ yếu đếm topo (face/edge/vertex) + proxy classify “simple_prismatic_proxy vs complex_topology_proxy”, có STUB_MODE khi thiếu OCC (fail-safe).

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

spec anchors: SSOT/fail-safe/reporting invariants theo Master Spec/Governance.

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

TÀI LIỆU ĐẶC TẢ HỆ THỐNG QUẢN T…

A) NAMING & IDENTIFIERS (NORMALIZED)
file_name: afr_provider.py [CHƯA XÁC MINH: path trong repo]
module_name: afr_provider
module_type: provider (feature recognition proxy → minimal tags)

algorithm_name (V1 proposed): minimal_feature_tags_from_brep_topology_and_local_thickness
algorithm_version (V1 proposed): v1
result_envelope_id: AFR_RESULT_v0 (giữ id hiện hữu để không phá reporter/pipeline; thêm fields V1 dạng optional).

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

B) ADN — afr_provider (DFM-first V1)

WHY (DFM-first)
AFR không phải để “khoe AI”; nó là “tagging tối thiểu” để các agent DFM phía sau không suy diễn:

sink_agent: cần biết boss/junction để ưu tiên hotspot sink.

ribratio_agent: cần biết rib candidates để tính t_rib/t_wall thật thay vì bbox proxy.

draft_agent V2: cần “functional/cosmetic surface hints” để giảm false positive (ví dụ shut-off/land không yêu cầu draft).

weldline_agent: có hole/boss/rib tags thì “flow split triggers” có căn cứ hơn.
Nếu AFR chỉ có topo counts, các agent V1 vẫn chạy được, nhưng V2 sẽ rất dễ “đoán”.

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

SCOPE (V1 baseline: minimal tags, high precision, không cố recall 100%)
V1 chỉ làm “tags ít nhưng chắc”, ưu tiên precision để báo cáo sạch. Tag tối thiểu đề xuất:

Core tags (ưu tiên cao):

HOLE_CYLINDER_CANDIDATE (lỗ dạng trụ)

BOSS_CYLINDER_CANDIDATE (boss dạng trụ)

RIB_PLATE_CANDIDATE (rib dạng tấm mỏng) — yêu cầu có thickness_result hoặc probing cục bộ

THICK_JUNCTION_CANDIDATE (vùng dày cục bộ) — có thể lấy từ thickness_agent worst_regions

Optional tags (để V2):

SHUTOFF_FACE_CANDIDATE, COSMETIC_A_SURFACE_CANDIDATE … [CHƯA XÁC MINH: vì cần input dự án/tiêu chuẩn bề mặt; không nên tự đoán trong V1]

Non-goals:

Không xây “feature graph” đầy đủ như phần mềm CAD/CAE.

Không sửa CAD, không tạo geometry mới.

Không khẳng định “through-hole vs blind-hole” nếu không đủ bằng chứng; chỉ tag candidate + confidence.

INPUT FORMALIZATION
Required (theo facts V0):

payload có brep_shape trong geometry context; có thể có agl_ref/dgl_ref, trace_id, mapping_result.

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

Fail-safe (facts):

OCC thiếu → trả STUB_MODE + NOT_SUPPORTED_NO_GEOM_LIB, không crash.

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

Optional inputs để V1 mạnh hơn (DFM-first):

thickness_result từ thickness_agent (để rib/thick junction tags có cơ sở). [CHƯA XÁC MINH: wiring field name]

dfm_policy.units.length_unit = "mm" (đã chốt)

thresholds.afr.\* (nếu bạn muốn đưa vào policy-as-config; hiện chưa có trong catalog) [CHƯA XÁC MINH]

OUTPUT FORMALIZATION
Output: AFR_RESULT_v0 envelope (giữ cấu trúc V0 + mở rộng). V0 đã có: status, metrics topo counts, features_proxy simple/complex, errors/warnings.

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

V1 bổ sung thêm:

feature_tags: list[feature_tag_obj] (bounded list, ví dụ max 50)

tag_summary: counts theo tag_type + confidence buckets

upstream_quality_echo: thickness coverage/miss_rate nếu dùng thickness_result

feature_tag_obj (schema sketch):

tag_type: string enum {
"HOLE_CYLINDER_CANDIDATE",
"BOSS_CYLINDER_CANDIDATE",
"RIB_PLATE_CANDIDATE",
"THICK_JUNCTION_CANDIDATE"
}

region_ref: REGION_REF_SCHEMA_v1 (AGL FACE hoặc PATCH; ưu tiên FACE_ID + uv_hint nếu cần)

attributes: dict (ví dụ radius_mm, axis_dir, length_mm, t_rib_mm, t_wall_mm…)

confidence_score: float 0..1

evidence: list[string] (ngắn, machine-readable, ví dụ ["CYLINDER_FACE", "INSIDE_OUTSIDE_TEST", "THICKNESS_MAP"])

limitations: string|null (nếu chỉ candidate)

Guarantees:

Read-only SSOT.

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

Deterministic nếu sampling/probing rules cố định; nếu có heuristic “probe inside/outside” thì epsilon phải policy-driven để tái lập. [CHƯA XÁC MINH]

ALGORITHM OVERVIEW (V1 minimal tags — hướng kỹ thuật, không CAE)

5.1. HOLE/BOSS cylinder candidate (hình học B-Rep)
Mục tiêu: phân biệt “cylinder face thuộc cavity (hole)” hay “cylinder face thuộc protrusion (boss)” ở mức candidate.

Heuristic đề xuất (ưu tiên precision):

Traverse faces; với mỗi face, xác định surface type bằng BRepAdaptor_Surface/GeomAbs.

Nếu face là CYLINDER: lấy (axis_dir, radius, param bounds).

Phân loại HOLE vs BOSS bằng “inside/outside probe”:

Lấy một điểm P trên mặt trụ (u,v mid).

Lấy normal n tại P (đã xét orientation).

Tạo 2 điểm: P_in = P − epsn, P_out = P + epsn (eps theo mm, ví dụ 0.2mm hoặc theo tolerance policy)

Dùng SolidClassifier để xem P_in nằm “trong solid” hay “ngoài solid”.

Nếu P_in trong solid (và P_out ngoài) theo quy ước orientation, đó thường là boss; ngược lại thường là hole.
[CHƯA XÁC MINH: API classifier cụ thể khi code; đây là thiết kế kỹ thuật]

Nếu classifier không khả dụng hoặc mơ hồ → chỉ tag “CYLINDER_CANDIDATE” với confidence thấp (hoặc bỏ tag để giữ báo cáo sạch).

5.2. RIB plate candidate (tận dụng thickness map để “không đoán”)
Rib DFM đúng nghĩa cần t_rib và t_wall.

Heuristic V1:

Chỉ xét rib khi có thickness_result đáng tin (miss_rate thấp).

Candidate rib patch = vùng có local thickness nhỏ hơn nominal wall đáng kể và hình thái “kéo dài” (span lớn) nhưng bề dày nhỏ.

Nominal wall: dùng p50 thickness của part (baseline) hoặc t_wall local quanh chân rib (nếu đo được). [CHƯA XÁC MINH: bạn muốn chốt strategy nào trong policy]

Tag RIB_PLATE_CANDIDATE chỉ khi:

có đủ (t_rib_mm, t_wall_mm) và ratio nằm trong khoảng hợp lý (để giảm false tags). [CHƯA XÁC MINH: khoảng ratio ban đầu]

5.3. THICK_JUNCTION candidate (để sink_agent dùng ngay)

Lấy trực tiếp từ thickness_agent worst_regions: vùng thickness cao, thick_ratio cao.

Tag THICK_JUNCTION_CANDIDATE với confidence dựa trên thickness coverage.

STEP-BY-STEP LOGIC
Step 1: Init AFR_RESULT_v0, status=INITIALIZED.

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

Step 2: Normalize geometry input từ payload để lấy brep_shape (V0 đã có logic normalize).

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

Step 3: OCC availability check (lazy-load). Nếu fail → STUB_MODE + NOT_SUPPORTED_NO_GEOM_LIB, return.

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

Step 4: Topology scan V0: đếm Face/Edge/Vertex; set features_proxy (\<=6 faces => simple_prismatic_proxy else complex_topology_proxy).

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

Step 5: Feature tags V1 (bounded & deterministic):

5a) Cylinder scan: tìm cylinder faces và phân loại HOLE/BOSS candidate theo inside/outside probe; tạo region_ref FACE.

5b) Nếu có thickness_result: tạo THICK_JUNCTION_CANDIDATE và RIB_PLATE_CANDIDATE theo heuristic; region_ref PATCH (FACE + uv_bbox) hoặc FACE.
Step 6: Pack output:

status=EXECUTED

metrics topo + feature_tags + tag_summary

warnings/issues nếu:

THICKNESS_RESULT_MISSING (chỉ ảnh hưởng rib/junction tags)

CLASSIFIER_UNAVAILABLE (hole/boss classification hạ confidence)

deterministic: không random; nếu cần sampling nhiều điểm, dùng grid cố định.

FAILURE MODES

GEOM_CONTEXT_MISSING → status=EXECUTED nhưng có error trong envelope (theo style V0: “không raise”, trả structured error). [CHƯA XÁC MINH: mã lỗi cụ thể]

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

OCC missing → STUB_MODE + NOT_SUPPORTED_NO_GEOM_LIB (facts)

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

CLASSIFICATION_AMBIGUOUS → không tag hoặc tag confidence thấp + issue AFR_AMBIGUOUS_CLASSIFICATION (khuyến nghị: prefer “không tag” để báo cáo sạch)

THICKNESS_LOW_COVERAGE → tag rib/junction confidence thấp + issue AFR_DEPENDS_ON_THICKNESS_LOW_COVERAGE

COMPLEXITY

Topo scan: O(F+E+V) (đã có ở V0).

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

Cylinder classification: O(F_cyl) + classifier cost (nhẹ nếu chỉ probe 1–3 điểm/face).

Rib/junction tags: O(N_worst_regions) nếu lấy từ thickness_result.

V2 UPGRADE INPUTS (để AFR thành “xương sống DFM”)

Feature graph (boss/rib/hole adjacency), root radius measurement, shutoff/parting candidates. [CHƯA XÁC MINH]

CAD attributes (naming, color, layer) để tag cosmetic A-surface thay vì đoán.

C) AMBR — afr_provider binding

ambr_id: AMBR_afr_provider_v2_research
status: DRAFT
module_name: afr_provider
module_type: provider
execution_mode: SEQUENTIAL (lazy-load deps)

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

pipeline_stage: Step 2 “Geometry Intelligence / Feature Recognition” theo Master Spec (facts).

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

Bound algorithm:

ADN_afr_provider_minimal_feature_tags_from_brep_topology_and_local_thickness_v1 (new)

Compatibility:

Preserves AFR_RESULT_v0 baseline fields; adds feature_tags optional.

D) LDR — dependencies (V1)

Baseline LDR (đã có cho topo explorer):

LDR_pythonocc_core_topology_explorer_v1 (TopExp_Explorer, TopAbs)

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

V1 add-on LDR (mới, để phân loại cylinder và inside/outside probe):
ldr_id: LDR_pythonocc_core_surface_classifier_for_afr_v1
status: DRAFT
library_name: pythonocc-core (OCC.Core.BRepAdaptor, OCC.Core.BRepLProp, OCC.Core.GeomAbs, OCC.Core.gp, OCC.Core.BRepClass3d 또는 tương đương) [CHƯA XÁC MINH: API exact]
purpose: surface type detection + normal eval + inside/outside classification for HOLE/BOSS candidates

E) AFR_RESULT_v0 — schema sketch (V1 extension)

AFR_RESULT_v0 (existing V0 core, extended):

result_type: "AFR_RESULT_v0"

run_id [CHƯA XÁC MINH]

status: INITIALIZED|EXECUTED|STUB_MODE

metrics: {face_count, edge_count, vertex_count}

features_proxy: "simple_prismatic_proxy" | "complex_topology_proxy"

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

feature_tags: [feature_tag_obj] (optional, V1)

tag_summary: {tag_type: count, ...} (optional)

issues/warnings: structured list

## 3.2 AMBR (Bindings / Module Contract / Execution)

**Source: S3 — AMBR_afr_provider_v1.docx**

AMBR — afr_provider — Binding v1

Status: REVIEW (V1 candidate).

1. Module identification

ambr_id: AMBR_afr_provider_v1

module_name: afr_provider

module_type: provider

binding_version: v1

repo_path: tooling_dfm_advisor/providers/afr_provider.py [CHƯA XÁC MINH repo has /providers]

execution_mode: SEQUENTIAL (lazy-load deps)

ssot_interaction: does_not_mutate_ssot=true; reads_brep_shape=true

owner: [CHƯA XÁC MINH]

2. Bound algorithm

adn_id: ADN_afr_provider_topology_proxy_feature_recognition_v1

algorithm_name: topology_proxy_feature_recognition

algorithm_version: v1

result_envelope: AFR_RESULT_v0

3. Dependencies (LDR)

LDR_pythonocc_core_topology_explorer_v1 (OCC.Core.TopExp, OCC.Core.TopAbs)

No other external libs allowed in this provider.

4. Determinism & side effects

Deterministic mapping: same frozen shape → same counts + proxy class.

Forbidden: IO/network; forbidden: healing/mutation; forbidden: storing any file paths/pointers.

Allowed: lazy-import dependency probe; conversion of internal exceptions to envelope errors.

5. Responsibility split

afr_provider owns: topo counts + proxy classification + fail-safe envelope behavior.

orchestrator/governance owns: mapping AFR_RESULT_v0 → global afr_state + analysis_completeness + receipts linkage.

6. Change impact rules

Changing threshold (face_count\<=6) is behavior change → must update golden expected AFR outputs and bump rule_version/policy snapshot.

Any schema change to AFR_RESULT_v0 is breaking unless additive → if breaking, create AFR_RESULT_v1 and update consumers.

7. Test obligations (minimum)

Golden Geometry QA: at least 1 prismatic sample (\<=6 faces) and 1 complex sample (>6 faces).

OCC absent test: NOT_EXECUTED envelope is returned.

No-path-leak test: envelope strings are scrubbed.

## 3.3 LDR (Dependencies / Libraries / Runtime constraints)

Source: S3 and S1 (dependency statements).

**Source: S3/S1 — dependency extracts**

3. Dependencies (LDR)

LDR_pythonocc_core_topology_explorer_v1 (OCC.Core.TopExp, OCC.Core.TopAbs)

Allowed: lazy-import dependency probe; conversion of internal exceptions to envelope errors.

OCC absent test: NOT_EXECUTED envelope is returned.

TÀI LIỆU TRIỂN KHAI ADN / AMBR / LDR — MODULE afr_provider (DFM-first V1)

THÔNG TIN PHIÊN BẢN (DOCUMENT METADATA)
doc_id: TOOLING_DFM_ADVISOR_AFR_PROVIDER_DFM_FIRST_V1_RESEARCH_v1
status: DRAFT
effective_date_utc: [CHƯA XÁC MINH]
owner: [CHƯA XÁC MINH]
baseline facts: afr_provider hiện V0 chủ yếu đếm topo (face/edge/vertex) + proxy classify “simple_prismatic_proxy vs complex_topology_proxy”, có STUB_MODE khi thiếu OCC (fail-safe).

OCC thiếu → trả STUB_MODE + NOT_SUPPORTED_NO_GEOM_LIB, không crash.

Step 3: OCC availability check (lazy-load). Nếu fail → STUB_MODE + NOT_SUPPORTED_NO_GEOM_LIB, return.

OCC missing → STUB_MODE + NOT_SUPPORTED_NO_GEOM_LIB (facts)

D) LDR — dependencies (V1)

Baseline LDR (đã có cho topo explorer):

LDR_pythonocc_core_topology_explorer_v1 (TopExp_Explorer, TopAbs)

V1 add-on LDR (mới, để phân loại cylinder và inside/outside probe):
ldr_id: LDR_pythonocc_core_surface_classifier_for_afr_v1
status: DRAFT
library_name: pythonocc-core (OCC.Core.BRepAdaptor, OCC.Core.BRepLProp, OCC.Core.GeomAbs, OCC.Core.gp, OCC.Core.BRepClass3d 또는 tương đương) [CHƯA XÁC MINH: API exact]
purpose: surface type detection + normal eval + inside/outside classification for HOLE/BOSS candidates

[CHƯA XÁC MINH] LDR_pythonocc_core_topology_explorer_v1 source document not provided in this pack; confirm if an existing canonical LDR doc should be referenced or upload the LDR doc for full traceability.

## 4. Open Questions / [CHƯA XÁC MINH]

THÔNG TIN PHIÊN BẢN (DOCUMENT METADATA)
doc_id: TOOLING_DFM_ADVISOR_AFR_PROVIDER_DFM_FIRST_V1_RESEARCH_v1
status: DRAFT
effective_date_utc: [CHƯA XÁC MINH]
owner: [CHƯA XÁC MINH]
baseline facts: afr_provider hiện V0 chủ yếu đếm topo (face/edge/vertex) + proxy classify “simple_prismatic_proxy vs complex_topology_proxy”, có STUB_MODE khi thiếu OCC (fail-safe).

thresholds.afr.\* (nếu bạn muốn đưa vào policy-as-config; hiện chưa có trong catalog) [CHƯA XÁC MINH]

Deterministic nếu sampling/probing rules cố định; nếu có heuristic “probe inside/outside” thì epsilon phải policy-driven để tái lập. [CHƯA XÁC MINH]

Feature graph (boss/rib/hole adjacency), root radius measurement, shutoff/parting candidates. [CHƯA XÁC MINH]

run_id [CHƯA XÁC MINH]

owner: [CHƯA XÁC MINH]

t_rib_mm \<= rib_max_absolute_mm (optional, để tránh tag nhầm “khối dày” thành rib). [CHƯA XÁC MINH]

min_cylinder_face_area_mm2 (float, default 5.0) [CHƯA XÁC MINH]

[CHƯA XÁC MINH] Provide or reference 'LDR_pythonocc_core_topology_explorer_v1' to complete dependency traceability for afr_provider.

[CHƯA XÁC MINH] User input canonical code was 'WS_PROVIDERS-AFR-SPC-v' (missing major). Confirm v1 is intended. Current output assumes v1 based on source filenames and mapping note.

## 5. Change Log (Editorial)

20260304T090817Z: Created WS_PROVIDERS-AFR-SPC-v1 consolidated from S1–S4. Governance-only: renamed sources to canonical filename; added Control Block + Mapping Table + traceability markers. No semantics changes.

20260304T090817Z: Normalized TARGET_CANONICAL_CODE to 'WS_PROVIDERS-AFR-SPC-v1' (v1) due to missing major in user input; treated as packaging correction only.

## 6. Appendix

6.1 Naming normalization notes (aka mapping)

Source: S1–S3. Module implementation uses snake_case (afr_provider). Contracts/envelopes use UPPER_CASE versioned identifiers (e.g., AFR_RESULT_v0). If alternate names exist in sources, they are preserved and mapped via 'aka' statements above.
