# WS_AGENTS-THK-SPC-v1 — thickness_agent Specification (Consolidated)

## 0. Document Control Block

|                                  |                                                                      |
| -------------------------------- | -------------------------------------------------------------------- |
| Canonical Code                   | WS_AGENTS-THK-SPC-v1                                                 |
| Status                           | REVIEW                                                               |
| Run ID                           | 20260304T100801Z                                                     |
| WS / Module / Doc Type / Version | WS_AGENTS / THK / SPC / v1                                           |
| Created date (UTC)               | 20260304T100801Z                                                     |
| Purpose                          | Document governance consolidation only; no runtime/pipeline changes. |

## 1. Scope

This document consolidates existing technical notes for thickness_agent (DFM-first baseline). It preserves source content and adds governance metadata and traceability.

Non-goals: no algorithm redesign; no policy invention; no code-level implementation; no CAE simulation.

## 2. Included Sources & Mapping Table

|           |                                                                                    |                                                   |                             |                                |                                                               |
| --------- | ---------------------------------------------------------------------------------- | ------------------------------------------------- | --------------------------- | ------------------------------ | ------------------------------------------------------------- |
| source_id | filename                                                                           | original_title                                    | original_version/status     | sections_used                  | notes                                                         |
| S1        | TÀI LIỆU TRIỂN KHAI ADN AMBR LDR — MODULE thickness_agent (DFM-first).docx         | MODULE thickness_agent (DFM-first) — ADN/AMBR/LDR | DRAFT (per source metadata) | ADN+AMBR+LDR core content      | Primary module spec; contains A/B/C/D sections.               |
| S2        | THICKNESS_ANALYSIS_RESULT_v0 — schema sketch (theo chuẩn Step B + region_ref).docx | THICKNESS_ANALYSIS_RESULT_v0 — schema sketch      | N/A                         | Result envelope sketch (Annex) | Used as schema reference; avoids duplicating full model spec. |

## 3. Consolidated Specification Content

### 3.1 ADN (Algorithm / Logic)

**Source: S1**

────────────────────────────────────────────────────────────
B) ADN — thickness_agent (V1 baseline)

IDENTIFICATION
adn_id: ADN_thickness_agent_local_thickness_sampling_along_normals_v1
module_name: thickness_agent
module_type: analysis_agent
algorithm_name: local_thickness_sampling_along_normals
algorithm_version: v1
status: DRAFT
owner: [CHƯA XÁC MINH]
last_reviewed_utc: [CHƯA XÁC MINH]

PROBLEM STATEMENT (DFM thực chiến)
Core problem: Ước lượng “độ dày thành” (local wall thickness) trên chi tiết nhựa ép để phục vụ quyết định khuôn và chất lượng ép:

phát hiện vùng quá mỏng (short shot, warp risk, khó điền đầy)

phát hiện vùng quá dày / junction dày (sink/void, cycle time cao)

phát hiện biến thiên độ dày lớn (co ngót không đều, cong vênh)
Mục tiêu là tạo advisory nhanh, minh bạch, không cần CAE dòng chảy, nhưng phải đúng về mặt hình học cục bộ (không chỉ bbox proxy).

Scope (V1 baseline):

Tính local thickness tại các điểm mẫu trên bề mặt bằng “ray / normal probing” để tìm khoảng cách đến mặt đối diện.

Tổng hợp phân phối thickness (min/pXX/max), tìm “worst regions” theo region_ref chuẩn.

So ngưỡng min_wall / max_wall (mm) từ DFM policy; nếu thiếu thì chỉ báo facts + POLICY_UNRESOLVED.

Non-goals:

Không dự đoán warpage, residual stress, cooling imbalance (CAE).

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

Không tự sửa CAD/đổi thiết kế.

Không đảm bảo thickness map mịn như phần mềm CAE; V1 ưu tiên bắt đúng “điểm gãy DFM” và chỉ ra vùng.

Không phân loại rib/boss chi tiết theo AFR graph (để dành V2/V3). [CHƯA XÁC MINH]

INPUT FORMALIZATION
Required:

geometry_context: brep_shape (TopoDS_Shape) từ AGL (đã import/heal; freeze không bắt buộc cho tính thickness nhưng geometry_version_id nên có nếu đã freeze). [CHƯA XÁC MINH: payload key path]

run_id (YYYYMMDDThhmmssZ) [CHƯA XÁC MINH: injection rule]

dfm_policy.units.length_unit == "mm" (đã chốt)

thresholds.thickness.min_wall (mm) — Required key (Bước B)
Optional:

thresholds.thickness.max_wall (mm)

common.sampling_budget (max_worst_regions, max_faces_to_report)

upstream mapping_result / afr_result (để traceability) — không bắt buộc cho V1, nhưng nếu có thì echo trạng thái vào upstream_check.

Fail-safe dependency:

Nếu OCC không khả dụng: status NOT_EXECUTED/STUB_MODE và issues OCC_MISSING; không crash pipeline.

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

OUTPUT FORMALIZATION
Output envelope: THICKNESS_ANALYSIS_RESULT_v0 (theo Master Spec)

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

Yêu cầu tối thiểu:

result_type, run_id, status, error_code/disable_reason, rule_version, geometry_version_id, metrics, issues, produced_defects, receipts_ref.

V1 baseline bổ sung metrics:

thickness_distribution_mm: min, p10, p50, p90, max

thin_region_count, thick_region_count

worst_regions: list(region_ref + thickness_min_mm + threshold + flags)

sampling_summary: faces_evaluated, samples_total, miss_rate (ray miss), ambiguous_hit_rate (nhiều giao điểm)

Guarantees:

Read-only SSOT (không mutate AGL).

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

Deterministic nếu sampling grid + ray rule cố định và không random; nếu có random thì seed phải dẫn xuất từ run_id.

TÀI LIỆU ĐẶC TẢ GHI CHÚ THIẾT K…

ALGORITHM OVERVIEW (trực giác kỹ thuật)
Local thickness tại điểm bề mặt được ước lượng bằng cách:

lấy pháp tuyến bề mặt n tại điểm mẫu (u,v)

bắn “tia” theo −n (vào trong vật thể) để tìm giao điểm gần nhất với bề mặt đối diện

thickness ≈ khoảng cách Euclid giữa điểm mẫu và giao điểm hợp lệ gần nhất
Đây là kỹ thuật hình học “đủ đúng” để bắt mỏng/dày cục bộ mà không cần mesh dày/CAE, miễn là xử lý đúng các trường hợp nhiều hit, lỗ, gân, undercut phức tạp.

STEP-BY-STEP LOGIC (chi tiết, có kiểm chứng)
Step 1 — Initialize envelope
Output: THICKNESS_ANALYSIS_RESULT_v0, status=OK (tentative), issues=[].

Step 2 — Validate inputs & policy

Missing brep_shape -> FAILED (GEOM_CONTEXT_MISSING)

Missing thresholds.thickness.min_wall -> OK nhưng POLICY_UNRESOLVED (không kết luận below-threshold)

Unit must be mm; nếu mâu thuẫn -> UNIT_UNRESOLVED và không compare theo mm.

Step 3 — Build sampling plan (deterministic)

Traverse faces: TopExp_Explorer(TopAbs_FACE)

Với mỗi face: xác định surface type (plane/cylinder/bspline…)

Chọn grid mẫu UV theo budget (ví dụ NxN theo area/curvature proxy).

Tổng sample_count_total bị chặn bởi policy.common.sampling_budget (nếu có).

Step 4 — Normal evaluation

Với mỗi sample (u,v): tính normal n (đã xét orientation).

Nếu normal không xác định/unstable: skip sample, ghi issue NORMAL_EVAL_FAILED (kèm region_ref mức FACE).

Step 5 — Ray / normal probing để tìm “opposite surface hit”
Quy tắc lựa chọn hit (để giảm sai):

Bắn tia theo −n (vào trong).

Thu tất cả giao điểm với shape.

Loại hit “quá gần” (self-hit) bằng epsilon khoảng cách nhỏ (policy-controlled) [CHƯA XÁC MINH].

Chọn hit hợp lệ gần nhất theo khoảng cách d>0 (first valid hit).

Nếu không có hit: miss (open shell / lỗi topo / sample ở vùng lỗ), tăng miss_rate.

Nếu có nhiều hit gần nhau: ambiguous_hit, chọn min d nhưng gắn warning AMBIGUOUS_INTERSECTION.

Step 6 — Compute thickness_at_sample_mm

thickness = distance(P_sample, P_hit)

Lưu thickness vào distribution (streaming quantiles hoặc histogram; tránh lưu toàn bộ nếu không cần).

Cập nhật “worst candidates”: mỏng nhất và dày nhất.

Step 7 — Aggregate & evaluate thresholds

thickness_min, p10/p50/p90, max

If min_wall available: flag thin_violation cho regions có thickness < min_wall.

If max_wall available: flag thick_violation cho regions có thickness > max_wall.

Emit defects theo template (mục D) cho top N vùng (tránh bắn defect tràn lan, giữ “advisor” tập trung).

Step 8 — Finalize envelope

status OK nếu tính được distribution đáng tin.

Nếu miss_rate vượt ngưỡng (policy) [CHƯA XÁC MINH], downgrade confidence và thêm issue THICKNESS_MAP_LOW_COVERAGE.

FAILURE MODES

GEOM_CONTEXT_MISSING -> FAILED

OCC_MISSING -> NOT_EXECUTED/STUB_MODE (fail-safe)

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

POLICY_UNRESOLVED -> OK nhưng không kết luận theo min/max

RAY_MISS_SYSTEMATIC -> OK nhưng LOW_COVERAGE (khuyến nghị kiểm tra solid validity / open shell)

COMPLEXITY
Time ~ O(F * S * I) với F faces, S samples, I cost intersection.
Memory: O(1) nếu dùng histogram/quantiles + topN worst_regions.

V2 UPGRADE INPUTS (để lên mức “kỹ sư khuôn dùng hằng ngày”)
Bắt buộc để V2 không đoán:

Tagging ribs/boss/functional shutoff faces (từ AFR hoặc CAD attributes) [CHƯA XÁC MINH]

Policy nâng cao: nominal_wall_strategy, allowed_variation_ratio, junction_thickness_ratio, boss_to_wall ratio [CHƯA XÁC MINH]

Golden corpus: có “đáp án” thickness map vùng mỏng/dày theo review thực tế [CHƯA XÁC MINH]

### 3.2 AMBR (Bindings / Module Contract / Execution)

**Source: S1**

────────────────────────────────────────────────────────────
C) AMBR — thickness_agent (binding)

ambr_id: AMBR_thickness_agent_v2 (đặt v2 để phân biệt với bản publish v1 hiện hành; chỉ là record id, không phải algorithm_version)
status: DRAFT
module_name: thickness_agent
module_type: analysis_agent
execution_mode: SEQUENTIAL (internal), orchestration PARALLEL safe

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

pipeline_stage: Step 5 (Parallel Analysis Agents)

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

Bound algorithm:

ADN_thickness_agent_local_thickness_sampling_along_normals_v1

output_envelope: THICKNESS_ANALYSIS_RESULT_v0

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

Responsibilities:

thickness_agent: local thickness + worst regions + defects

orchestrator/pipeline: inject run_id/rule_version/geometry_version_id, receipts/logs, always-on reporter

TÀI LIỆU ĐẶC TẢ HỆ THỐNG QUẢN T…

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

### 3.3 LDR (Dependencies / Runtime Constraints)

**Source: S1**

────────────────────────────────────────────────────────────
D) LDR — dependencies (umbrella, nghiên cứu)

ldr_id: LDR_pythonocc_core_intersection_and_surface_normals_for_thickness_agent_v1
status: DRAFT
library_name: pythonocc-core (OCC.Core)
library_version: [CHƯA XÁC MINH: pinning]
purpose: topology traverse + surface normal + ray/shape intersection

Expected OCC API families (khóa ở mức “family”, chi tiết allowed subset sẽ chốt khi code):

TopExp/TopAbs/TopoDS (traverse)

BRepAdaptor/BRepLProp/GeomAbs (surface type + normal)

Intersection utilities (shape-ray intersection) [CHƯA XÁC MINH: module/class tối ưu sẽ dùng]

Fail-safe: OCC missing -> NOT_EXECUTED/STUB_MODE, không crash.

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

## 4. Open Questions / [CHƯA XÁC MINH]

Items below are copied from sources as-is; no attempt is made to resolve them in this governance step.

**Source: S1**

THÔNG TIN PHIÊN BẢN (DOCUMENT METADATA)
doc_id: TOOLING_DFM_ADVISOR_THICKNESS_AGENT_DFM_V1_RESEARCH_v1
status: DRAFT
effective_date_utc: [CHƯA XÁC MINH]
owner: [CHƯA XÁC MINH]
units_locked: length_unit = mm, angle_unit = deg (theo Bước B)
related_specs: TOOLING_DFM_ADVISOR_MASTER_SPEC_v1, TOOLING_DFM_ADVISOR_GOVERNANCE_MASTER_SPEC_v1, TOOLING_DFM_ADVISOR_ADN_MASTER_SPEC_v1

IDENTIFICATION
adn_id: ADN_thickness_agent_local_thickness_sampling_along_normals_v1
module_name: thickness_agent
module_type: analysis_agent
algorithm_name: local_thickness_sampling_along_normals
algorithm_version: v1
status: DRAFT
owner: [CHƯA XÁC MINH]
last_reviewed_utc: [CHƯA XÁC MINH]

Không phân loại rib/boss chi tiết theo AFR graph (để dành V2/V3). [CHƯA XÁC MINH]

Loại hit “quá gần” (self-hit) bằng epsilon khoảng cách nhỏ (policy-controlled) [CHƯA XÁC MINH].

Nếu miss_rate vượt ngưỡng (policy) [CHƯA XÁC MINH], downgrade confidence và thêm issue THICKNESS_MAP_LOW_COVERAGE.

Tagging ribs/boss/functional shutoff faces (từ AFR hoặc CAD attributes) [CHƯA XÁC MINH]

Policy nâng cao: nominal_wall_strategy, allowed_variation_ratio, junction_thickness_ratio, boss_to_wall ratio [CHƯA XÁC MINH]

Golden corpus: có “đáp án” thickness map vùng mỏng/dày theo review thực tế [CHƯA XÁC MINH]

**Source: S2**

rule_version: [CHƯA XÁC MINH]

receipts_ref: [CHƯA XÁC MINH]

## 5. Change Log (Editorial)

20260304T100801Z: Consolidated sources into canonical WS_AGENTS-THK-SPC-v1; added control block, mapping table, traceability labels. No semantics changes.

## 6. Appendix

### 6.1 Result Envelope Sketch (THICKNESS_ANALYSIS_RESULT_v0)

**Source: S2**

THICKNESS_ANALYSIS_RESULT_v0 — schema sketch (theo chuẩn Step B + region_ref)

THICKNESS_ANALYSIS_RESULT_v0:

result_type: "THICKNESS_ANALYSIS_RESULT_v0"

run_id

status: OK | FAILED | NOT_EXECUTED | STUB_MODE

error_code: null | <code>

disable_reason: null | string

rule_version: [CHƯA XÁC MINH]

geometry_version_id: string

inputs_echo:

units: {length_unit: "mm"}

policy_echo:

min_wall_mm: float|null

max_wall_mm: float|null

sampling_budget: {...}|null

metrics:

sampling_summary:

face_count_evaluated: int

sample_count_total: int

ray_miss_rate: float (0..1)

ambiguous_hit_rate: float (0..1)

thickness_distribution_mm:

min: float

p10: float

p50: float

p90: float

max: float

violations:

thin_region_count: int|null (null nếu POLICY_UNRESOLVED)

thick_region_count: int|null

worst_regions: list (top N)

item:

region_ref: REGION_REF_SCHEMA_v1 (FACE hoặc PATCH) (theo REGION_REF_CONVENTION_v1)

thickness_min_mm: float

thickness_max_mm: float|null (nếu đang report thin, max có thể bỏ)

below_min_wall: bool|null

above_max_wall: bool|null

applied_thresholds_mm: {min_wall: float|null, max_wall: float|null}

confidence_score: float

issues: list[issue_obj] (POLICY_UNRESOLVED, UNIT_UNRESOLVED, LOW_COVERAGE, OCC_MISSING…)

produced_defects: list[str]

receipts_ref: [CHƯA XÁC MINH]
