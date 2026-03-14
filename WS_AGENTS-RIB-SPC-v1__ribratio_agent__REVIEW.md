**WS_AGENTS-RIB-SPC-v1 — ribratio_agent Specification (Document Governance Only)**

# 0. Document Control Block

|                    |                                                                                                    |
| ------------------ | -------------------------------------------------------------------------------------------------- |
| Canonical Code     | WS_AGENTS-RIB-SPC-v1                                                                               |
| Status             | REVIEW                                                                                             |
| Run ID             | 20260304T110208Z                                                                                   |
| Created date (UTC) | 20260304T110208Z                                                                                   |
| Purpose            | Consolidated SPC (ADN/AMBR/LDR) for ribratio_agent; document governance only; no semantics change. |

## 1. Scope

Source: S1

Scope (V1 baseline):

Phát hiện “candidate ribs” ở mức hình học (không cần full AFR graph): rib = protrusion dạng tấm mỏng, cao, bám vào wall, có 2 mặt bên gần song song.

Ước lượng t_rib (bề dày rib) và t_wall (bề dày wall gần chân rib) bằng local thickness probing (tận dụng thickness_agent V1 nếu có).

Tính rib_to_wall_ratio = t_rib / t_wall.

So ngưỡng dfm_policy.thresholds.ribratio.max_rib_to_wall_ratio (dimensionless). (Đây là key “đúng nghĩa” DFM.)

Nếu không đủ điều kiện tính rib_to_wall thật: chạy proxy_mode (bbox proxy) nhưng bắt buộc gắn RIBRATIO_PROXY_ONLY_LOW_CONFIDENCE (không được emit defect “rib ratio vượt ngưỡng” theo nghĩa t_rib/t_wall).

Non-goals:

Không mô phỏng điền đầy/áp suất (CAE).

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

Không tự chỉnh CAD.

Không cố gắng nhận diện mọi rib 100%; V1 ưu tiên bắt rib “điển hình” và worst-case.

## 2. Included Sources & Mapping Table

|           |                                                               |                                                               |                                         |                                                             |                                                                          |
| --------- | ------------------------------------------------------------- | ------------------------------------------------------------- | --------------------------------------- | ----------------------------------------------------------- | ------------------------------------------------------------------------ |
| source_id | filename                                                      | original_title                                                | original_version/status                 | sections_used                                               | notes                                                                    |
| S1        | TÀI LIỆU TRIỂN KHAI ADN AMBR LDR — MODULE ribratio_agent.docx | MODULE ribratio_agent (DFM-first) — ADN/AMBR/LDR consolidated | status in source: DRAFT; doc_id present | ADN + AMBR + LDR + schema sketch + defect template extracts | Consolidated into canonical SPC; verbatim extracts; headings normalized. |

## 3. Consolidated Specification Content

## 3.1 ADN (Algorithm / Logic)

Source: S1

B) ADN — ribratio_agent (V1 baseline)

IDENTIFICATION

adn_id: ADN_ribratio_agent_rib_to_wall_ratio_from_local_thickness_and_rib_detection_v1

module_name: ribratio_agent

module_type: analysis_agent

algorithm_name: rib_to_wall_ratio_from_local_thickness_and_rib_detection

algorithm_version: v1

status: DRAFT

PROBLEM STATEMENT (DFM thực chiến)

Core problem: Đánh giá rủi ro sink/warp và khó ép do thiết kế gân tăng cứng (rib) có tỷ lệ không phù hợp so với thành chính:

rib quá dày so với wall → sink tại chân rib, cycle time cao

rib quá mỏng / quá cao → khó điền đầy, gãy rib, weldline/air trap tăng

Kỹ sư khuôn cần được chỉ ra: rib nào nguy hiểm nhất, ratio rib_to_wall, và khuyến nghị xử lý (giảm rib thickness, core-out, tăng radius chân, đổi bố trí). Đây là “rào DFM kinh điển” nên phải là local, không dùng bbox proxy.

Scope (V1 baseline):

Phát hiện “candidate ribs” ở mức hình học (không cần full AFR graph): rib = protrusion dạng tấm mỏng, cao, bám vào wall, có 2 mặt bên gần song song.

Ước lượng t_rib (bề dày rib) và t_wall (bề dày wall gần chân rib) bằng local thickness probing (tận dụng thickness_agent V1 nếu có).

Tính rib_to_wall_ratio = t_rib / t_wall.

So ngưỡng dfm_policy.thresholds.ribratio.max_rib_to_wall_ratio (dimensionless). (Đây là key “đúng nghĩa” DFM.)

Nếu không đủ điều kiện tính rib_to_wall thật: chạy proxy_mode (bbox proxy) nhưng bắt buộc gắn RIBRATIO_PROXY_ONLY_LOW_CONFIDENCE (không được emit defect “rib ratio vượt ngưỡng” theo nghĩa t_rib/t_wall).

Non-goals:

Không mô phỏng điền đầy/áp suất (CAE).

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

Không tự chỉnh CAD.

Không cố gắng nhận diện mọi rib 100%; V1 ưu tiên bắt rib “điển hình” và worst-case.

INPUT FORMALIZATION

Required (để làm rib_to_wall thật):

geometry_context.brep_shape (AGL)

thickness_result từ thickness_agent (local thickness distribution + worst regions) [CHƯA XÁC MINH: field wiring cụ thể]

policy thresholds:

dfm_policy.thresholds.ribratio.max_rib_to_wall_ratio (required cho quyết định pass/fail)

Optional:

dfm_policy.thresholds.ribratio.min_root_radius_mm (để cảnh báo chân rib nhọn) [CHƯA XÁC MINH]

afr_provider output nếu có feature graph (để tăng recall/precision rib detection) [CHƯA XÁC MINH: afr hiện proxy topo]

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

Fail-safe:

OCC missing: NOT_EXECUTED/STUB_MODE, trừ khi agent chạy purely-on-data (có sẵn rib candidates từ upstream) [CHƯA XÁC MINH: bạn có muốn hỗ trợ mode này không]

OUTPUT FORMALIZATION

Output envelope: RIBRATIO_ANALYSIS_RESULT_v0 (dict), tuân RESULT_ENVELOPE_API_v1 tối thiểu.

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

V1 baseline metrics (đúng nhu cầu khuôn):

method_used: "RIB_TO_WALL_V1" | "BBOX_PROXY_V0"

nominal_wall_mm: (khuyến nghị lấy t_wall local tại chân rib; nếu chưa có, fallback p50 thickness toàn part)

rib_candidates_count

rib_ratio_distribution: max, p90 (dimensionless)

worst_ribs: top N, mỗi item gồm:

region_ref (FACE/PATCH trên AGL tại chân rib hoặc mặt bên rib)

t_rib_mm

t_wall_mm

rib_to_wall_ratio

over_threshold (bool|null)

applied_threshold (float|null)

confidence_score

upstream_quality echo: thickness miss_rate/sample_count (để biết map có đáng tin)

ALGORITHM OVERVIEW (V1 baseline – đủ dùng, không CAE)

Hai bài toán: (a) tìm rib candidates, (b) đo t_rib và t_wall.

(a) Rib candidate detection (hình học):

Tìm các face “dạng tấm”: có hai face bên đối diện gần song song, khoảng cách giữa chúng nhỏ (t_rib), và vùng này kéo dài theo một hướng (rib height).

Heuristic để bắt rib nhanh (không cần perfect):

Lọc các vùng có thickness nhỏ hơn “t_wall” đáng kể? (rib thường mỏng hơn wall hoặc xấp xỉ 0.4–0.7 wall).

Dùng adjacency graph: rib side faces thường là 2 mặt dài, đối diện, chung edge với một “top face” nhỏ. [CHƯA XÁC MINH: mức sâu adjacency bạn muốn]

Nếu có AFR sau này: dùng rib feature trực tiếp.

(b) Thickness measurement:

t_rib: đo thickness tại mid của 2 mặt bên rib (normal probing), chọn min/median.

t_wall: đo thickness ở vùng chân rib (patch quanh giao tuyến rib–wall).

Rib ratio: t_rib / t_wall.

STEP-BY-STEP LOGIC

Step 1: Initialize envelope RIBRATIO_ANALYSIS_RESULT_v0.

Step 2: Validate policy: nếu thiếu max_rib_to_wall_ratio → POLICY_UNRESOLVED, không emit “over-threshold” defect.

Step 3: Resolve thickness_result:

Nếu có thickness map đủ coverage → tiếp tục V1.

Nếu LOW_COVERAGE → vẫn chạy nhưng giảm confidence và thêm issue RIBRATIO_LOW_CONFIDENCE_THICKNESS_COVERAGE.

Step 4: Detect rib candidates (deterministic):

Traverse faces, build candidate patches theo tiêu chí “thin plate-like protrusion”.

Giới hạn theo sampling_budget để không nổ runtime.

Step 5: Measure t_rib và t_wall cho từng candidate:

t_rib: lấy một số sample ổn định trên side faces.

t_wall: sample quanh chân rib (tránh self-hit, dùng epsilon mm theo policy thickness.probe.self_hit_epsilon_mm nếu dùng chung) [CHƯA XÁC MINH: reuse key hay tạo key riêng]

Step 6: Compute ratio và rank worst ribs.

Step 7: Evaluate vs threshold và emit defects top N:

DEFECT_TYPE: RIBRATIO_OVER_THRESHOLD (đúng nghĩa t_rib/t_wall)

region_ref: FACE/PATCH ở chân rib hoặc side face (chốt theo convention v1).

Step 8: Nếu không thể làm V1 (thiếu thickness_result/thiếu OCC): fallback proxy_mode:

chỉ emit issue + warning “proxy only”, tuyệt đối không kết luận rib ratio thật.

Step 9: Finalize status OK/FAILED/NOT_EXECUTED.

FAILURE MODES

POLICY_UNRESOLVED: OK nhưng không emit “over-threshold” defect; chỉ facts + issue.

THICKNESS_RESULT_MISSING: fallback proxy_only_low_confidence.

UNIT_UNRESOLVED: ERROR (vì mm sai → ratio vẫn dimensionless nhưng t_rib/t_wall lấy từ mm; nếu unit sai thì cả hai sai cùng hệ, ratio có thể vẫn “đúng giả”, nên phải gắn lỗi unit để tránh false trust).

OCC_MISSING: NOT_EXECUTED nếu không có dữ liệu upstream.

COMPLEXITY

Nếu có thickness_result và candidate detection hạn chế: O(F + N_candidates * S_probe).

Nếu proxy mode: O(1)/bbox.

V2 UPGRADE INPUTS

AFR rib graph + boss junction classification để tăng recall/precision. [CHƯA XÁC MINH]

Root radius measurement (min_root_radius_mm) và rib height ratio checks (h/t) [CHƯA XÁC MINH: bạn muốn thêm key nào]

Golden corpus có “expected rib findings” theo review khuôn.

────────────────────────────────────────────────────────────

## 3.2 AMBR (Bindings / Module Contract / Execution)

Source: S1

C) AMBR — ribratio_agent (binding)

ambr_id: AMBR_ribratio_agent_v2_research

status: DRAFT

module_name: ribratio_agent

module_type: analysis_agent

execution_mode: SEQUENTIAL (internal), orchestration PARALLEL-safe

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

pipeline_stage: Step 5 (Parallel Analysis Agents)

TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…

bound_algorithm: ADN_ribratio_agent_rib_to_wall_ratio_from_local_thickness_and_rib_detection_v1

bound_policy_keys: dfm_policy.thresholds.ribratio.max_rib_to_wall_ratio (+ optional min_root_radius_mm)

────────────────────────────────────────────────────────────

## 3.3 LDR (Dependencies / Runtime Constraints)

Source: S1

D) LDR — dependencies (research)

ldr_id: LDR_ribratio_agent_depends_on_thickness_map_and_optional_occ_topology_v1

status: DRAFT

dependencies:

internal data dependency: thickness_agent output (local thickness map)

external optional: pythonocc-core for topology traverse + probing (nếu tính trực tiếp)

fail-safe: nếu thiếu OCC hoặc thiếu thickness_result → proxy_mode + low confidence, hoặc NOT_EXECUTED theo policy.

────────────────────────────────────────────────────────────

## 4. Open Questions / [CHƯA XÁC MINH]

Source: S1

1. effective_date_utc: [CHƯA XÁC MINH]

1. owner: [CHƯA XÁC MINH]

1. dfm_policy.thresholds.ribratio.min_root_radius_mm (để cảnh báo chân rib nhọn) [CHƯA XÁC MINH]

1. AFR rib graph + boss junction classification để tăng recall/precision. [CHƯA XÁC MINH]

1. rule_version [CHƯA XÁC MINH]

1. receipts_ref: [CHƯA XÁC MINH]

## 5. Change Log (Editorial)

20260304T110208Z: Created canonical SPC WS_AGENTS-RIB-SPC-v1 from S1. Governance-only consolidation: normalized headings, added control block, mapping table, explicit Source tags, and extracted [CHƯA XÁC MINH] items. No technical semantics change.

## 6. Appendix

## 6.1 RIBRATIO_ANALYSIS_RESULT_v0 — schema sketch (extract)

Source: S1

E) RIBRATIO_ANALYSIS_RESULT_v0 — schema sketch (V1 baseline)

RIBRATIO_ANALYSIS_RESULT_v0:

result_type: "RIBRATIO_ANALYSIS_RESULT_v0"

run_id

status: OK|FAILED|NOT_EXECUTED|STUB_MODE

rule_version [CHƯA XÁC MINH]

geometry_version_id

inputs_echo:

units: {length_unit:"mm"}

policy_echo:

max_rib_to_wall_ratio: float|null

preferred_rib_to_wall_ratio: float|null

min_root_radius_mm: float|null

metrics:

method_used: "RIB_TO_WALL_V1" | "BBOX_PROXY_V0"

rib_candidates_count: int

rib_ratio_distribution:

max: float|null

p90: float|null

worst_ribs: \[

{

"region_ref": REGION_REF_SCHEMA_v1,

"t_rib_mm": float|null,

"t_wall_mm": float|null,

"rib_to_wall_ratio": float|null,

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

────────────────────────────────────────────────────────────

## 6.2 Defect advisory template — ribratio_agent (extract)

Source: S1

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
