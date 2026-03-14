WS_CORE-GEN-POL-v2 — Core General Policy (Document Governance Pack)

# 0. Document Control Block

**Canonical Code**: WS_CORE-GEN-POL-v2

**Status**: REVIEW

**Run ID**: 20260304T075821Z

**WS / Module / Doc Type / Version**: WS_CORE / GEN / POL / v2

**Created date (UTC)**: 2026-03-04T07:58:21Z

**Purpose**: Document governance consolidation only; no runtime/pipeline changes.

## 1. Scope

This canonical POL consolidates core (WS_CORE / GEN) governance policies and operational conventions related to geometry lifecycle and the validate↔heal↔freeze loop. Source content is preserved (verbatim extracts) and re-structured for naming/version control only.

Non-goals: No new algorithms, no changes to existing technical semantics, no re-architecture, no new runtime pipeline proposals, no simulation.

## 2. Included Sources & Mapping Table

|           |                                      |                                 |                         |                                                                                                      |                                                                                                               |
| --------- | ------------------------------------ | ------------------------------- | ----------------------- | ---------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| source_id | filename                             | original_title                  | original_version/status | sections_used                                                                                        | notes                                                                                                         |
| S1        | AGL_LIFECYCLE_MODEL_v2_0_FROZEN.docx | AGL_LIFECYCLE_MODEL_v2_0_FROZEN | FROZEN / v2.0           | All content (verbatim)                                                                               | Existing core lifecycle policy; preserved as-is.                                                              |
| S2        | HEAL-LOOP PROTOCOL_v1.docx           | HEAL-LOOP PROTOCOL_v1           | DRAFT / v1              | All substantive sections (verbatim); truncated cross-ref placeholders moved to Appendix Extract Note | Protocol between validation gate ↔ healing ↔ freeze; preserved as-is; open questions kept as [CHƯA XÁC MINH]. |

## 3. Consolidated Policy Content

## 3.1 AGL Lifecycle Model (verbatim extract)

**Source: S1**

AGL_LIFECYCLE_MODEL_v2_0_FROZEN

doc_id: AGL_LIFECYCLE_MODEL_v2_0

status: FROZEN

system_name: tooling_dfm_advisor

architecture_level: INDUSTRIAL_GRADE_FREEZE

version: V2_0

freeze_date_utc: 2026-02-27T13:59:27Z

Why

Formalize geometry immutability and version traceability.

States

INIT → HEALED → VALIDATED → FROZEN (terminal).

Freeze Rule

Assign deterministic geometry_version_id based on canonical fingerprint.

Agent Access Rule

Agents execute only when state == FROZEN.

Evidence

Freeze receipt must include geometry hash.

Fail-Fast

Mutation after freeze triggers governance violation.

Cross Reference

Feeds ANALYSIS_RESULT_CONTRACT_v2_0 and GOLDEN_REGRESSION_STRATEGY_v2_0.

## 3.2 Heal-loop Protocol (verbatim extract)

**Source: S2**

HEAL-LOOP PROTOCOL_v1
Ràng buộc giữa validators (geometry_validation_gate) + healer_agent + freeze, theo DFM-first và governance đúng chuẩn

THÔNG TIN PHIÊN BẢN
doc_id: TOOLING_DFM_ADVISOR_HEAL_LOOP_PROTOCOL_v1
status: DRAFT
effective_date_utc: [CHƯA XÁC MINH]
scope: chỉ mô tả protocol/contract vận hành giữa 3 khối: validation gate ↔ healing ↔ freeze (không mô tả CAE/DFM agents).
spec anchors: Canonical pipeline (validation gate + freeze + report always-on), fail-safe khi thiếu OCC, SSOT AGL bất biến sau freeze, receipts tamper-evident và anti-loop.

Why (DFM-first)
Heal-loop tồn tại để đảm bảo “DFM metrics downstream” (draft/undercut/thickness/sink/ribratio) không bị sai vì B-Rep lỗi (open shell, non-manifold, self-intersection…) và để AGL freeze có ý nghĩa công nghiệp. Nếu không có heal-loop, bạn sẽ thấy 2 vấn đề: (a) agent báo lỗi DFM “ảo”, (b) pipeline dễ dead-end khi gặp file STEP bẩn.

Invariants / Non-goals
2.1. Invariants (không được vi phạm)

Không crash pipeline khi thiếu OCC: validators/healer phải trả NOT_EXECUTED/STUB_MODE; pipeline vẫn phải xuất report tối thiểu (“Geometry Health Report”).

Không mutate AGL in-place sau freeze; mọi chữa phải diễn ra trước freeze.

Tamper-evident: mỗi vòng validate/heal phải có receipt/log; sửa tay receipts/audit → hard-fail theo governance.

Anti-loop: mỗi vòng lặp chỉ hợp lệ nếu có “progress evidence” (xem mục 6), vượt giới hạn thì phải PAUSE/FALLBACK, không vòng lặp vô hạn.

Healer chỉ minimal-fix (ShapeFix + BRepCheck) và fail-safe khi thiếu OCC.

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

2.2. Non-goals

Không “đậu gate bằng mọi giá” bằng cách tăng tolerance vô hạn. Healing phải bị chặn bởi policy knobs (mode/tolerances/topology change).

Không thay thiết kế CAD; chỉ chữa để hình học hợp lệ.

Roles / Responsibility split

validators (geometry_validation_gate): đọc shape hiện tại, trả GEOMETRY_VALIDATION_RESULT_v0: PASS/FAIL/DEFERRED + fail_reason_codes + healing_candidate + retryable. (Gate không heal.)

healer_agent: minimal-fix (BRepCheck + ShapeFix), trả HEAL_RESULT_v0: is_valid_before/after, fix_applied, operations_applied, topo_invariants delta, tolerance policy used (theo observability addendum), fail-safe OCC missing.

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

freeze step: khi gate PASS, tạo AGL bất biến + geometry_version_id + topo index map canonical; từ đây mọi defect region_ref ổn định “within geometry_version_id”.

orchestrator: điều phối loop, áp dụng policy, ghi receipts/audit, đảm bảo report always-on.

Preconditions (điều kiện trước khi vào heal-loop)

Có brep_shape “working shape” sau import (và có thể sau một lần heal trước đó).

Policy keys đã có: dfm_policy.thresholds.validation.\* và dfm_policy.thresholds.healing.\* (enabled/mode/tolerances/etc.).

Có run_id (YYYYMMDDThhmmssZ) và kênh receipt/audit hoạt động (HMAC key, lock).

ejection_direction chưa bắt buộc tại heal-loop (nó thuộc Contextual Inception), nhưng heal-loop phải hoàn tất trước khi chạy các DFM agents phụ thuộc normal/probing.

Inputs/Outputs (contract tối thiểu)
5.1. Inputs vào loop (tại mỗi iteration i)

shape_i (TopoDS_Shape) + refs (agl_ref/dgl_ref nếu có).

policy snapshot (validation/healing keys) theo rule_version.

counters: heal_attempt_index i, max_heal_attempts (từ policy). [CHƯA XÁC MINH: key max_heal_attempts bạn muốn đặt ở validation hay healing; Master Spec yêu cầu có giới hạn vòng lặp.]

5.2. Outputs mỗi iteration

receipt_validate_i + GEOMETRY_VALIDATION_RESULT_v0

nếu heal xảy ra: receipt_heal_i + HEAL_RESULT_v0

nếu PASS: receipt_freeze + GEOMETRY_FREEZE_RESULT_v0 + geometry_version_id (AGL immutable)

5.3. Terminal outputs

PASS → freeze thành công → chuyển sang providers/agents + report

FAIL sau khi hết điều kiện heal hoặc progress guard → không freeze, nhưng vẫn xuất report (Geometry Health + DFM degraded).

Progress guard (anti-loop criteria)
Một iteration “heal rồi validate lại” chỉ được phép tiếp tục nếu có ít nhất một bằng chứng tiến triển:

validity improved: validators FAIL→PASS, hoặc fail_reason_codes giảm/đổi theo hướng “nhẹ hơn”

healer improved: HEAL_RESULT.is_valid_after=true (trong bối cảnh BRepCheck), hoặc fix_applied=true kèm thay đổi topo/tolerance trong giới hạn policy

no-progress: fix_applied=true nhưng validators fail_reason_codes không đổi và topo_invariants không đổi → coi là “HEALING_NO_EFFECT”, không được lặp lại cùng chế độ; phải dừng hoặc đổi strategy (policy lane).

Protocol steps (chi tiết theo “tài liệu vận hành”)

7.1. Step L0 — Validate (Iteration i)
Why: xác định shape_i có đủ điều kiện freeze không, và nếu fail thì fail loại gì để quyết có nên heal.
Inputs: shape_i, dfm_policy.thresholds.validation.\*, run_id.
Actions:

Nếu validation.enabled=false → gate_decision=DEFERRED, issue VALIDATION_DISABLED_BY_POLICY; chuyển sang fallback report (khuyến nghị).

Chạy checks theo mode (FAST/FULL): non-manifold, open shell/volume, self-intersection (nếu implement).

Outputs:

GEOMETRY_VALIDATION_RESULT_v0 với:

gate_decision: PASS/FAIL/DEFERRED

fail_reason_codes

healing_candidate: LIKELY/POSSIBLE/UNLIKELY/UNKNOWN

retryable (true nếu OCC missing / env)
Evidence:

receipt_validate_i (stdout/stderr hash + HMAC)

Fail-fast:

OCC missing → NOT_EXECUTED/STUB_MODE, retryable=true, stop loop và báo degraded mode.

Troubleshooting:

FACECOUNT/volume degenerate → kiểm import/heal settings; có thể file chỉ là surface model.

7.2. Step L1 — Decision: heal or stop
Why: tránh over-heal và tránh loop vô hạn.
Preconditions: có GEOMETRY_VALIDATION_RESULT_v0.
Decision rules (policy-driven, default khuyến nghị):

Nếu gate_decision=PASS → đi freeze ngay (L3).

Nếu gate_decision=DEFERRED:

nếu retryable=true (OCC missing): stop, report degraded.

nếu không retryable: stop, report.

Nếu gate_decision=FAIL:

nếu healing_candidate=UNLIKELY → stop sớm, report (đừng phí vòng heal).

nếu heal_attempt_index >= max_heal_attempts → stop, report.

nếu progress guard đã kích hoạt NO_PROGRESS ở vòng trước → stop hoặc PAUSE_FOR_AUTHORITY_DECISION (tùy governance lane).

Outputs:

decision_code (closed-set) cho orchestrator: HEAL_NEXT | FREEZE_NEXT | FALLBACK_REPORT | PAUSE_FOR_AUTHORITY_DECISION. [CHƯA XÁC MINH: decision_code catalog trong policy; governance yêu cầu closed-set + decision_map.]

7.3. Step L2 — Heal (Iteration i)
Why: minimal-fix để tăng khả năng PASS ở lần validate tiếp theo, nhưng không làm drift thiết kế.
Inputs: shape_i, dfm_policy.thresholds.healing.\*, run_id.
Actions (facts baseline):

BRepCheck_Analyzer shape_i (is_valid_before)

Nếu invalid: ShapeFix_Shape.Perform() → shape\_{i+1}

Post-check BRepCheck_Analyzer (is_valid_after)

Fail-safe OCC missing → NOT_EXECUTED passthrough shape_i

TÀI LIỆU TRIỂN KHAI ADN AMBR LD…

Outputs:

HEAL_RESULT_v0 + shape\_{i+1} (hoặc passthrough)

operations_applied + topo_invariants delta + tolerance_policy_used (theo addendum observability)
Evidence:

receipt_heal_i + log; ghi rõ policy snapshot used.

Fail-fast:

healing.mode invalid → POLICY_INVALID, stop.

topology change vượt max_topology_change_allowed hoặc tolerance_delta vượt max_tolerance_delta_mm → requires_owner_review và stop (không tiếp tục auto-loop).
Troubleshooting:

HEALING_NO_EFFECT: kiểm xem fail_reason_codes thuộc loại non-manifold nặng; heal khó, nên dừng sớm.

7.4. Step L0 again — Re-Validate (Iteration i+1)
Why: chỉ gate mới quyết “đủ điều kiện freeze”.
Inputs: shape\_{i+1}.
Actions: chạy validators lại, tạo receipt_validate\_{i+1}.
Outputs: GEOMETRY_VALIDATION_RESULT\_{i+1}.
Evidence: compare fail_reason_codes, metrics, and/or validity flags để xác nhận progress.
Fail-fast: nếu fail_reason_codes không đổi và healer báo HEALING_NO_EFFECT → trigger NO_PROGRESS guard, stop loop.

7.5. Step L3 — Freeze AGL (Terminal success)
Why: tạo SSOT bất biến, chốt geometry_version_id, chuẩn bị mapping/topo index để defect region_ref ổn định.

Preconditions: gate_decision=PASS.
Inputs: shape_passed, run_id.
Actions:

Freeze AGL: generate geometry_version_id, persist AGL immutable.

Generate topo index map canonical (FACE/EDGE/VERTEX) để support REGION_REF_CONVENTION_v1. [CHƯA XÁC MINH: field storage trong GEOMETRY_STACK_API_v1]
Outputs:

GEOMETRY_FREEZE_RESULT_v0 (status OK) + geometry_version_id
Evidence:

receipt_freeze + audit entry “GEOMETRY_FROZEN”.

Fail-fast:

nếu freeze không tạo được geometry_version_id hoặc không thể đảm bảo immutability → FAILED, fallback report.

7.6. Step L4 — Fallback report (Terminal fail-safe)
Why: industrial invariant “report always-on”; không freeze vẫn phải báo rõ lý do và degraded_mode.

Inputs: last validation/heal receipts, last gate result, run_id.
Actions:

Reporter tạo Geometry Health Report: fail_reason_codes, heal attempts, progress/no-progress, OCC status, policy snapshot.
Outputs:

Report artifact (JSON/PDF/HTML tùy layer).

Fail-fast conditions (hard stop)

OCC missing và retryable=true → không chạy heal/validate nặng; dừng và report degraded.

max_heal_attempts reached → stop loop, report.

NO_PROGRESS guard triggered → stop loop, PAUSE/FALLBACK (không lặp lại).

Healing vượt policy safety rails (tolerance/topology change) → requires_owner_review và stop.

Troubleshooting playbook (ngắn, theo pattern)

FAIL: NON_MANIFOLD → thường là lỗi thiết kế/topology; heal-loop nên dừng sớm (UNLIKELY).

FAIL: OPEN_SHELL / ZERO_VOLUME → heal thường LIKELY (sewing/closure), nhưng nếu 1–2 vòng không tiến triển → stop để tránh drift.

FAIL: SELF_INTERSECTION → nếu FAST không chắc, có thể bật FULL cho 1–2 mẫu để xác nhận, nhưng phải có max_runtime_seconds để bảo vệ pipeline.

Healer fix_applied=true nhưng topo_changed lớn → dừng và yêu cầu review (rủi ro drift DFM).

Những điểm còn [CHƯA XÁC MINH] cần Owner chốt để protocol “đóng băng” thành chuẩn vận hành

max_heal_attempts sẽ đặt ở dfm_policy.thresholds.validation hay thresholds.healing (khuyến nghị: validation.max_heal_attempts vì đây là gate loop).

decision_code catalog + decision_map placement trong governance_policy.json (để phù hợp closed-set decision).

self-intersection checker API thực tế trong OCC và cách map “suspected vs confirmed” vào fail_reason_codes.

nơi lưu topo index map canonical trong SSOT.

### Appendix Extract Note for S2

The following truncated cross-reference placeholders were removed from the main consolidated section for readability (no semantic content):

- TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG QUẢN T…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG QUẢN T…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG QUẢN T…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG QUẢN T…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG QUẢN T…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG QUẢN T…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG QUẢN T…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG QUẢN T…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG QUẢN T…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG QUẢN T…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG QUẢN T…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG QUẢN T…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG toolin…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG QUẢN T…
- TÀI LIỆU ĐẶC TẢ HỆ THỐNG QUẢN T…

## 4. Open Questions / [CHƯA XÁC MINH]

- effective_date_utc for TOOLING_DFM_ADVISOR_HEAL_LOOP_PROTOCOL_v1 is not specified in the source.
- Placement of max_heal_attempts policy key: thresholds.validation.\* vs thresholds.healing.\* (source flags as [CHƯA XÁC MINH]).
- decision_code catalog (closed-set) and where it lives in governance policy config (source flags as [CHƯA XÁC MINH]).
- Concrete OCC API for self-intersection check and how to map suspected vs confirmed into fail_reason_codes (source flags as [CHƯA XÁC MINH]).
- Where to store topo index map canonical in SSOT / geometry stack (source flags as [CHƯA XÁC MINH]).
- Doc-type fit: heal-loop protocol reads like an operational protocol (SPC-like); confirm that it is intended to live under WS_CORE-GEN-POL-v2 rather than a separate SPC/ADR.

## 5. Change Log (Editorial)

- Renamed and consolidated sources into canonical code WS_CORE-GEN-POL-v2; applied status REVIEW.
- Preserved source technical text as verbatim extracts; only re-structured headings and added traceability markers.
- Removed truncated cross-reference placeholders (e.g., 'TÀI LIỆU ĐẶC TẢ ...') from main sections and recorded them in Appendix Extract Notes.
- Added Open Questions section to explicitly mark missing/undecided items as [CHƯA XÁC MINH].

## 6. Appendix

6.1 Naming conventions reminder (no semantics change): module implementations in snake_case; contracts/schemas in UPPER_CASE versioned. If source uses different names, keep original and add 'aka' mapping here.

6.2 Cross-references: When a section references another canonical POL/SPC, reference by canonical code only (no file paths).
