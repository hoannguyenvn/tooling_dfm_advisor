# 0. Document Control Block

|                                  |                                                                      |
| -------------------------------- | -------------------------------------------------------------------- |
| Canonical Code                   | WS_MODEL-RGN-POL-v1                                                  |
| Status                           | REVIEW                                                               |
| Run ID                           | 20260228T085922Z                                                     |
| WS / Module / Doc Type / Version | WS_MODEL / RGN / POL / v1                                            |
| Created date (UTC)               | 20260228T085922Z                                                     |
| Purpose                          | Document governance consolidation only; no runtime/pipeline changes. |

## 1. Scope

Tài liệu này chuẩn hoá và khóa chuẩn policy/convention cho REGION_REF (định vị vùng lỗi/region tham chiếu trong result/defect). Nội dung kế thừa trực tiếp từ các nguồn được liệt kê ở mục 2; không thay đổi semantics.

Non-goals: không thiết kế lại kiến trúc, không thay đổi logic phân tích DFM, không sửa đổi pipeline/runtime, không thêm quy tắc mới ngoài nguồn.

## 2. Included Sources & Mapping Table

|           |                                        |                          |                                    |                                                  |                                                                                          |
| --------- | -------------------------------------- | ------------------------ | ---------------------------------- | ------------------------------------------------ | ---------------------------------------------------------------------------------------- |
| source_id | filename                               | original_title           | original_version/status            | sections_used                                    | notes                                                                                    |
| S1        | PHẦN 2 — REGION_REF_CONVENTION_v1.docx | REGION_REF_CONVENTION_v1 | APPROVED (Owner-locked) [từ nguồn] | Full document (verbatim extract; re-headed only) | Nguồn SSOT mô tả convention REGION_REF_SCHEMA_v1, id_scheme closed-set, fail-fast rules. |

## 3. Consolidated Policy Content (Verbatim Source Extract)

Source: S1

PHẦN 2 — REGION_REF_CONVENTION_v1

THÔNG TIN PHIÊN BẢN (DOCUMENT METADATA)

doc_id: TOOLING_DFM_ADVISOR_REGION_REF_CONVENTION_v1
status: APPROVED (Owner-locked)
effective_date_utc: [CHƯA XÁC MINH]
owner: [CHƯA XÁC MINH]
scope: chuẩn tham chiếu vùng lỗi/định vị kết quả trong result/defect, không tạo shadow data
related_specs: TOOLING_DFM_ADVISOR_MASTER_SPEC_v1, TOOLING_DFM_ADVISOR_GOVERNANCE_MASTER_SPEC_v1

MỤC TIÊU VÀ BẤT BIẾN (GOALS & INVARIANTS)

2.1. Định vị được “vùng tệ nhất” cho kỹ sư khuôn

Report phải chỉ ra region “tệ nhất” (worst region) để người xem quyết định parting/slide/đổi draft.

2.2. Không shadow data

Tuyệt đối cấm lưu đường dẫn file bền vững (file path), pointer, hoặc bất kỳ tham chiếu gây rò rỉ dữ liệu ngoài SSOT/receipt.

Nếu có ảnh/section/mesh cục bộ: chỉ được lưu token ephemeral_ref và bắt buộc scrub sau delivery_complete.

2.3. Phạm vi ổn định (stability scope)

region_ref chỉ được cam kết ổn định trong phạm vi geometry_version_id.

Không cam kết ổn định giữa 2 geometry_version_id khác nhau (vì heal/freeze có thể thay topology).

REGION_REF_SCHEMA_v1 — cấu trúc chuẩn

region_ref (object):

schema_id: "REGION_REF_SCHEMA_v1"

geometry_version_id: string

layer: "AGL" | "DGL" | "HYBRID"

kind: "FACE" | "EDGE" | "VERTEX" | "PATCH"

id_scheme: string (closed-set, xem mục 4)

id: string | int

hints (optional):

uv_hint:

uv_point: {u: float, v: float} | null

uv_bbox: {u_min: float, u_max: float, v_min: float, v_max: float} | null

point_hint_xyz: {x: float, y: float, z: float} | null

bbox_hint_xyz: {xmin: float, xmax: float, ymin: float, ymax: float, zmin: float, zmax: float} | null

stability_scope: "WITHIN_GEOMETRY_VERSION_ID"

notes: string | null

ID_SCHEME CLOSED-SET (chốt để khỏi lệch)

4.1. AGL (B-Rep) schemes

"AGL_FACE_INDEX_V1": id là số thứ tự face trong “indexed map” canonical của AGL tại thời điểm freeze.

"AGL_EDGE_INDEX_V1": tương tự cho edge.

"AGL_VERTEX_INDEX_V1": tương tự cho vertex.

4.2. DGL (Mesh) schemes (dành cho tương lai, optional)

"DGL_TRI_INDEX_V1": triangle index

"DGL_VERTEX_INDEX_V1": vertex index

4.3. HYBRID scheme (khi có mapping b2m/m2b đáng tin)

"HYBRID_FACE_TO_TRISET_V1": id là face_id + list token của triset (khuyến nghị chỉ lưu hash/token, không lưu list dài). [CHƯA XÁC MINH: mapping integrity đã đủ chưa]

QUY TẮC TẠO “INDEXED MAP” CHO AGL (để FACE_INDEX có nghĩa)

Yêu cầu hệ thống (đặt ở bước freeze / hoặc ngay sau freeze):

Tạo topo index map canonical bằng TopTools_IndexedMapOfShape (FACE/EDGE/VERTEX) theo AGL shape đã freeze.

Lưu map này vào SSOT (GeometryStack) như “topo_index_map_ref” hoặc tương đương để mọi agent tham chiếu thống nhất. [CHƯA XÁC MINH: field chỗ nào trong GEOMETRY_STACK_API_v1]

Nếu topo index map chưa tồn tại:

Agent vẫn được phép tạo “transient index map” tại runtime để báo cáo, nhưng phải gắn issue REGION_REF_NOT_CANONICAL và giảm confidence (vì risk drift). [CHƯA XÁC MINH: bạn có chấp nhận transient không]

QUY TẮC SỬ DỤNG REGION_REF TRONG RESULT/DEFECT

6.1. Trong \*\_ANALYSIS_RESULT_v0

worst_regions: bắt buộc dùng region_ref với AGL_FACE_INDEX_V1 (ưu tiên).

Nếu cần thể hiện “mảng vùng”, dùng kind="PATCH" + uv_bbox (trên cùng face_id).

6.2. Trong defect object

defect.region_ref: bắt buộc có geometry_version_id + layer + kind + id_scheme + id.

hints là optional; không bắt buộc để tránh phình và tránh lộ dữ liệu.

Cấm lưu path ảnh; chỉ dùng assets_ephemeral_ref token theo Master Spec.

FAIL-FAST / FAIL-SAFE RULES

Fail-fast (tạo issue severity cao):

region_ref thiếu geometry_version_id hoặc thiếu layer/kind/id_scheme/id → INVALID_REGION_REF

Fail-safe:

Nếu thiếu uv_hint/point_hint: vẫn hợp lệ nếu có face_id.

Nếu agent không thể lấy canonical map: dùng transient và phát issue (không crash).

FILLED EXAMPLE

region_ref:
schema_id: REGION_REF_SCHEMA_v1
geometry_version_id: geom_20260223T090000Z_abcd1234
layer: AGL
kind: FACE
id_scheme: AGL_FACE_INDEX_V1
id: 123
hints:
uv_hint:
uv_bbox: {u_min: 0.2, u_max: 0.35, v_min: 0.6, v_max: 0.8}
stability_scope: WITHIN_GEOMETRY_VERSION_ID
notes: "worst draft region on curved face; no durable paths stored"

## 4. Open Questions / [CHƯA XÁC MINH]

Source: S1

doc_id: TOOLING_DFM_ADVISOR_REGION_REF_CONVENTION_v1
status: APPROVED (Owner-locked)
effective_date_utc: [CHƯA XÁC MINH]
owner: [CHƯA XÁC MINH]
scope: chuẩn tham chiếu vùng lỗi/định vị kết quả trong result/defect, không tạo shadow data
related_specs: TOOLING_DFM_ADVISOR_MASTER_SPEC_v1, TOOLING_DFM_ADVISOR_GOVERNANCE_MASTER_SPEC_v1

## 5. Change Log (Editorial)

20260228T085922Z: Consolidated governance-only. Đổi tên file theo canonical code, thêm Document Control Block + Mapping Table + Open Questions, giữ nguyên nội dung kỹ thuật (verbatim) từ S1.

## 6. Appendix

Cross references (from S1): TOOLING_DFM_ADVISOR_MASTER_SPEC_v1; TOOLING_DFM_ADVISOR_GOVERNANCE_MASTER_SPEC_v1.
