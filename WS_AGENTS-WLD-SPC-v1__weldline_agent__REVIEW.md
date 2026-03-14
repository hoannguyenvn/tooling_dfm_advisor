WS_AGENTS-WLD-SPC-v1 — weldline_agent (DFM-first)

# 0. Document Control Block

|                                  |                                                                              |
| -------------------------------- | ---------------------------------------------------------------------------- |
| Field                            | Value                                                                        |
| Canonical Code                   | WS_AGENTS-WLD-SPC-v1                                                         |
| Status                           | REVIEW                                                                       |
| Run ID (UTC)                     | 20260304T111830Z                                                             |
| WS / Module / Doc Type / Version | WS_AGENTS / WLD / SPC / v1                                                   |
| Created date (UTC)               | 20260304T111830Z                                                             |
| Purpose                          | Document governance consolidation only; no runtime/pipeline changes.         |
| Primary Module (implementation)  | weldline_agent                                                               |
| Result Envelope                  | WELDLINE_ANALYSIS_RESULT_v0 (per source; kept for report pipeline stability) |

## 1. Scope

Source: S1

This SPC consolidates the DFM-first weldline_agent specification (algorithm + bindings + dependencies) from existing source documents. It preserves the source technical content, reorganized into ADN/AMBR/LDR sections, with traceability.

Non-goals: No CAE flow simulation; no gate/runner design; no CAD modification; no architecture redesign; no new heuristics beyond what sources state.

## 2. Included Sources & Mapping Table

|           |                                                                |                                                                          |                                                                                           |                                                |                                                                                          |
| --------- | -------------------------------------------------------------- | ------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------- | ---------------------------------------------- | ---------------------------------------------------------------------------------------- |
| source_id | filename                                                       | original_title                                                           | original_version/status                                                                   | sections_used                                  | notes                                                                                    |
| S1        | TÀI LIỆU TRIỂN KHAI ADN AMBR LDR — MODULE weldline_agent .docx | TÀI LIỆU TRIỂN KHAI ADN / AMBR / LDR — MODULE weldline_agent (DFM-first) | doc_id: TOOLING_DFM_ADVISOR_WELDLINE_AGENT_DFM_V1_RESEARCH_v1; status: DRAFT (per source) | ADN, AMBR, LDR, schema sketch, defect template | Single-source consolidation; placeholders removed as editorial cleanup (see Change Log). |

## 3. Consolidated Specification Content

## 3.1 ADN (Algorithm / Logic)

Source: S1

B) ADN — weldline_agent (DFM-first)

IDENTIFICATION

adn_id: ADN_weldline_agent_topo_proxy_with_validity_and_flow_split_triggers_v1

module_name: weldline_agent

module_type: analysis_agent

algorithm_name: topo_proxy_with_validity_and_flow_split_triggers

algorithm_version: v1

status: DRAFT

PROBLEM STATEMENT (DFM thực chiến)

Core problem: Cảnh báo sớm “khả năng nhạy weldline” (weldline susceptibility) của chi tiết nhựa dựa trên hình học (không CAE), để kỹ sư khuôn chủ động đánh giá nguy cơ yếu cơ tính/đường hàn trên vùng chịu lực hoặc vùng thẩm mỹ, và xem xét: đổi bố trí gate, đổi hướng flow, thêm overflow/vent, chỉnh thiết kế tạo đường dòng hợp lý.

Reality constraints (bất biến, không được vượt rào):

Không có mô phỏng dòng chảy và không biết gate/runner layout => không thể dự đoán “vị trí weldline thật” với độ tin cậy cao. Vì vậy V1 chỉ được phép:

(a) đưa ra chỉ số rủi ro tổng thể (global proxy),

(b) nêu “yếu tố kích hoạt chia dòng” (flow split triggers) ở mức gợi ý vùng cần chú ý, kèm confidence thấp/medium, không khẳng định là weldline chắc chắn.

Scope (V1 baseline):

Giữ proxy chính của V0: WeldRiskProxy = EdgeCount / FaceCount.

Bổ sung “validity checks” để tránh proxy gây hiểu nhầm: face_count tối thiểu, divide-by-zero, hình học suy biến.

Bổ sung “trigger hints” (không CAE): đếm/nhận diện proxy các yếu tố thường gây chia dòng như “nhiều lỗ xuyên / nhiều đảo topo / nhiều nhánh” ở mức đếm (không định vị chính xác nếu chưa có feature tagging). [CHƯA XÁC MINH: trigger set tối ưu theo thực tế sản phẩm mục tiêu]

Output phải có: method_used, validity_state, confidence_score và issues rõ ràng.

Non-goals:

Không dự đoán đường weldline trên bề mặt theo tọa độ chính xác như Moldflow.

Không auto-suggest gate location như quyết định cuối (có thể gợi ý câu hỏi cần xem xét).

Không thay đổi CAD.

INPUT FORMALIZATION

Required:

geometry_context.brep_shape (TopoDS_Shape) [CHƯA XÁC MINH: payload key path]

dfm_policy.thresholds.weldline.max_weld_risk_proxy (ngưỡng proxy) — nếu thiếu thì POLICY_UNRESOLVED (chỉ facts).

Optional:

dfm_policy.thresholds.weldline.min_face_count_for_valid_proxy (starter default đang là 20)

afr_result (nếu có) để gắn “complex_topology_proxy” vào confidence (V1 có thể dùng như meta, không bắt buộc).

Fail-safe dependency:

Nếu OCC thiếu: NOT_EXECUTED/STUB_MODE (không crash).

Nếu shape hợp lệ nhưng FaceCount=0: tuyệt đối không phép chia lỗi; trả ERROR có cấu trúc.

OUTPUT FORMALIZATION

Output envelope: WELDLINE_ANALYSIS_RESULT_v0, tuân RESULT_ENVELOPE_API_v1 tối thiểu.

Metrics V1 baseline (đủ cho review khuôn):

method_used: "TOPO_PROXY_V0" | "TOPO_PROXY_V1_VALIDITY_AWARE"

topo_counts: face_count, edge_count, vertex_count [vertex_count optional]

weld_risk_proxy: edge_count/face_count (nếu face_count>0)

validity:

is_valid_proxy: bool

validity_reason_code: "FACECOUNT_TOO_LOW"|"DIV_BY_ZERO"|"OK"|...

trigger_hints (optional, không được coi là vị trí weldline):

hole_like_feature_proxy_count: int [CHƯA XÁC MINH: cách đo]

multi_component_proxy: bool [CHƯA XÁC MINH: nếu shape có nhiều solid]

evaluation:

over_threshold: bool|null

applied_threshold: float|null

confidence_score: 0..1 (giảm nếu validity kém hoặc trigger_hints thiếu cơ sở)

Guarantees:

Read-only SSOT; không mutate AGL.

Deterministic: cùng shape + cùng OCC version => topo counts và proxy cố định.

STEP-BY-STEP LOGIC

Step 1: Initialize envelope WELDLINE_ANALYSIS_RESULT_v0.

Step 2: Validate inputs:

thiếu brep_shape => FAILED (GEOM_CONTEXT_MISSING)

OCC missing => NOT_EXECUTED/STUB_MODE, issue OCC_MISSING

Step 3: Count topology (proxy core):

edge_count: TopExp_Explorer(shape, TopAbs_EDGE)

face_count: TopExp_Explorer(shape, TopAbs_FACE)

Step 4: Validity checks:

if face_count \<= 0 => ERROR DIVIDE_BY_ZERO_FACECOUNT

if face_count < policy.min_face_count_for_valid_proxy (default 20) => is_valid_proxy=false, add issue WELDLINE_PROXY_LOW_VALIDITY_FACECOUNT; confidence giảm; (tùy policy) không emit “over-threshold defect”, chỉ warning.

Step 5: Compute weld_risk_proxy nếu valid:

weld_risk_proxy = edge_count / face_count

Step 6: Optional trigger hints (không CAE, chỉ gợi ý):

[CHƯA XÁC MINH] nếu có thể, đếm “hole-like features” bằng proxy (ví dụ số lượng vòng biên tròn/cylindrical faces) để tăng cảnh giác chia dòng. Nếu không có thì bỏ qua phần này, không suy diễn.

Step 7: Evaluate vs threshold:

nếu policy.max_weld_risk_proxy tồn tại và proxy valid: over_threshold = weld_risk_proxy > max_weld_risk_proxy

nếu thiếu policy => POLICY_UNRESOLVED (facts only)

Step 8: Emit defects (top-level, global)

Nếu over_threshold và proxy valid: emit defect WELDLINE_PROXY_OVER_THRESHOLD (severity do policy.severity_map.weldline.proxy)

Nếu proxy low validity: emit defect hoặc issue “LOW_VALIDITY” theo policy (khuyến nghị chỉ issue WARN).

Step 9: Finalize envelope: status OK/FAILED/NOT_EXECUTED.

FAILURE MODES

GEOM_CONTEXT_MISSING => FAILED

OCC_MISSING => NOT_EXECUTED/STUB_MODE (fail-safe)

FaceCount=0 => ERROR DIVIDE_BY_ZERO_FACECOUNT (fail-safe)

POLICY_UNRESOLVED => OK nhưng chỉ facts, không kết luận over-threshold

PROXY_LOW_VALIDITY_FACECOUNT => OK nhưng confidence thấp, khuyến nghị “review manually / need CAE for location”.

COMPLEXITY

Time: O(E + F) cho đếm topo; Memory: O(1).

V2 UPGRADE INPUTS (để weldline_agent “DFM mạnh” hơn mà vẫn không CAE)

Gate context (ngay cả “gate side” coarse) => tăng giá trị định vị vùng nghi ngờ weldline. [CHƯA XÁC MINH: hệ thống có muốn thu context gate không]

AFR feature graph (holes/boss/ribs) => trigger_hints định vị theo region_ref chính xác hơn. [CHƯA XÁC MINH]

Mapping DGL integrity tốt => có thể suy ra “flow split corridors” theo mesh graph ở mức heuristic (không solver). [CHƯA XÁC MINH]

## 3.2 AMBR (Bindings / Module Contract / Execution)

Source: S1

C) AMBR — weldline_agent (binding)

ambr_id: AMBR_weldline_agent_v2_research

status: DRAFT

module_name: weldline_agent

module_type: analysis_agent

execution_mode: SEQUENTIAL (internal), orchestration PARALLEL-safe

pipeline_stage: Step 5 (Parallel Analysis Agents)

Bound algorithm: ADN_weldline_agent_topo_proxy_with_validity_and_flow_split_triggers_v1

Bound policy keys:

dfm_policy.thresholds.weldline.max_weld_risk_proxy

dfm_policy.thresholds.weldline.min_face_count_for_valid_proxy

Responsibility split:

weldline_agent: compute topo proxy + validity + (optional) trigger hints; emit defects/issue accordingly.

pipeline/orchestrator: inject run_id/rule_version/geometry_version_id; receipts; always-on reporter.

## 3.3 LDR (Dependencies / Runtime constraints)

Source: S1

D) LDR — dependencies

ldr_id: LDR_pythonocc_core_topology_explorer_for_weldline_agent_v1

status: DRAFT

library_name: pythonocc-core (OCC.Core.TopExp, OCC.Core.TopAbs)

library_version: [CHƯA XÁC MINH]

purpose: topo enumeration (edge/face counts) for WeldRiskProxy.

risk: heavy C++ dep; must fail-safe to NOT_EXECUTED/STUB_MODE when absent.

## 6. Appendix

Source: S1

## 6.1 WELDLINE_ANALYSIS_RESULT_v0 — schema sketch (as provided)

E) WELDLINE_ANALYSIS_RESULT_v0 — schema sketch (V1 validity-aware)

WELDLINE_ANALYSIS_RESULT_v0:

result_type: "WELDLINE_ANALYSIS_RESULT_v0"

run_id

status: OK|FAILED|NOT_EXECUTED|STUB_MODE

error_code / disable_reason

rule_version [CHƯA XÁC MINH]

geometry_version_id

inputs_echo:

units: {length_unit:"mm"} (echo theo policy, dù metric là ratio)

policy_echo:

max_weld_risk_proxy: float|null

min_face_count_for_valid_proxy: int|null

metrics:

method_used: "TOPO_PROXY_V1_VALIDITY_AWARE"

topo_counts:

face_count: int

edge_count: int

weld_risk_proxy: float|null

validity:

is_valid_proxy: bool

validity_reason_code: "OK"|"FACECOUNT_TOO_LOW"|"DIV_BY_ZERO_FACECOUNT"

evaluation:

over_threshold: bool|null

applied_threshold: float|null

trigger_hints: { ... } | null (optional, V2+)

confidence_score: float

issues: list[issue_obj]

produced_defects: list[str]

receipts_ref: [CHƯA XÁC MINH]

Ghi chú region_ref:

Với topo proxy thuần global, defect sẽ không có “vùng chắc chắn”. Nếu buộc phải có region_ref, chỉ dùng kind="PATCH" với id_scheme="AGL_FACE_INDEX_V1" cho “top suspect faces” khi có trigger_hints đáng tin; nếu không có thì để defect là “GLOBAL” và region_ref null. [CHƯA XÁC MINH: hệ thống có cho phép defect không có region_ref không; Master Spec mô tả defect có region_ref nhưng thực tế weldline proxy không định vị được].

## 6.2 Defect advisory template — weldline_agent (as provided)

F) Defect advisory template — weldline_agent

WELDLINE_DEFECT_TEMPLATE_v1:

id

run_id

agent_source: "weldline_agent"

rule_version

geometry_version_id

timestamp_utc

severity: INFO|WARN|ERROR|FATAL (theo dfm_policy.severity_map.weldline.proxy)

defect_type:

"WELDLINE_PROXY_OVER_THRESHOLD"

"WELDLINE_PROXY_LOW_VALIDITY_FACECOUNT"

"WELDLINE_NOT_EXECUTED_OCC_MISSING"

region_ref: null | REGION_REF_SCHEMA_v1 (chỉ set khi có trigger_hints định vị đáng tin) [CHƯA XÁC MINH]

analysis_value:

edge_count: int

face_count: int

weld_risk_proxy: float|null

validity: {is_valid_proxy: bool, reason: string}

method: "TOPO_PROXY_V1_VALIDITY_AWARE"

threshold_applied:

max_weld_risk_proxy: float|null

advisory:

machine_readable:

summary_code: "REVIEW_GATE_STRATEGY" | "ADD_OVERFLOW" | "ADD_VENT" | "REVIEW_FLOW_SPLIT_FEATURES"

recommended_actions: list[str]

rationale_tags: ["TOPO_COMPLEXITY", "PROXY_ONLY", "LOW_VALIDITY"]

human_readable:

title

why_it_matters (đường hàn làm yếu cơ tính, ảnh hưởng mỹ quan)

suggested_fix (định hướng: xem gate/overflow/vent; tránh chia dòng không cần)

verification (nếu critical: khuyến nghị CAE hoặc thử khuôn/DOE)

confidence_score

assets_ephemeral_ref (token only)

delivery_scrubbed

## 4. Open Questions / [CHƯA XÁC MINH]

Source: S1

effective_date_utc: [CHƯA XÁC MINH]

owner: [CHƯA XÁC MINH]

[CHƯA XÁC MINH] nếu có thể, đếm “hole-like features” bằng proxy (ví dụ số lượng vòng biên tròn/cylindrical faces) để tăng cảnh giác chia dòng. Nếu không có thì bỏ qua phần này, không suy diễn.

AFR feature graph (holes/boss/ribs) => trigger_hints định vị theo region_ref chính xác hơn. [CHƯA XÁC MINH]

Mapping DGL integrity tốt => có thể suy ra “flow split corridors” theo mesh graph ở mức heuristic (không solver). [CHƯA XÁC MINH]

library_version: [CHƯA XÁC MINH]

rule_version [CHƯA XÁC MINH]

receipts_ref: [CHƯA XÁC MINH]

region_ref: null | REGION_REF_SCHEMA_v1 (chỉ set khi có trigger_hints định vị đáng tin) [CHƯA XÁC MINH]

## 5. Change Log (Editorial)

This document is a governance consolidation (rename + restructure) only; no semantic changes intended.

- Normalized filename to canonical code: WS_AGENTS-WLD-SPC-v1\_\_weldline_agent\_\_REVIEW.docx (from source filename).

- Reorganized content into required SPC structure (Control Block, Scope, Mapping Table, ADN/AMBR/LDR, Open Questions, Appendix).

- Removed repeated non-informational placeholder lines (e.g., truncated references like 'TÀI LIỆU ĐẶC TẢ HỆ THỐNG ...') and separator lines; original technical statements preserved.

- Added explicit traceability markers 'Source: S1' at subsection boundaries.
