WS_GOV-ALL-SPC-v1 (REVIEW)

# 0. Document Control Block

|                                  |                                                                      |
| -------------------------------- | -------------------------------------------------------------------- |
| Canonical Code                   | WS_GOV-ALL-SPC-v1                                                    |
| Status                           | REVIEW                                                               |
| Run ID (UTC)                     | 20260304T115902Z                                                     |
| Created date (UTC)               | 2026-03-04T11:59:02Z                                                 |
| WS / Module / Doc Type / Version | WS_GOV / ALL / SPC / v1                                              |
| Purpose                          | Document governance consolidation only; no runtime/pipeline changes. |

## 1. Scope

Source: S1–S4

This canonical SPC consolidates the governance-layer system specifications and addenda that define operational invariants, naming/directory requirements, state/receipt discipline, and DFM-first non-bypassable deltas. Content is inherited from sources via verbatim extracts and re-ordered for consistency only.

Non-goals: No algorithm redesign, no pipeline changes, no new technical rules beyond what is present in sources.

## 2. Included Sources & Mapping Table

|           |                                                                          |                                                                     |                         |                                  |                                       |
| --------- | ------------------------------------------------------------------------ | ------------------------------------------------------------------- | ----------------------- | -------------------------------- | ------------------------------------- |
| source_id | filename                                                                 | original_title                                                      | original_version/status | sections_used                    | notes                                 |
| S1        | TÀI LIỆU ĐẶC TẢ HỆ THỐNG QUẢN TRỊ tooling_dfm_advisor SIPM SHIP.docx     | TÀI LIỆU ĐẶC TẢ HỆ THỐNG QUẢN TRỊ tooling_dfm_advisor SIPM SHIP     | [CHƯA XÁC MINH]         | Selected extracts; see section 3 | Primary authority source for this SPC |
| S2        | TÀI LIỆU ĐẶC TẢ HỆ THỐNG tooling_dfm_advisor (MASTER SPECIFICATION).docx | TÀI LIỆU ĐẶC TẢ HỆ THỐNG tooling_dfm_advisor (MASTER SPECIFICATION) | [CHƯA XÁC MINH]         | Selected extracts; see section 3 | Primary authority source for this SPC |
| S3        | TOOLING_DFM_ADVISOR_MASTER_SPEC_ADDENDUM_v1.docx                         | TOOLING_DFM_ADVISOR_MASTER_SPEC_ADDENDUM_v1                         | [CHƯA XÁC MINH]         | Selected extracts; see section 3 | Primary authority source for this SPC |
| S4        | TOOLING_DFM_ADVISOR_GOVERNANCE_MASTER_SPEC_ADDENDUM_v1.docx              | TOOLING_DFM_ADVISOR_GOVERNANCE_MASTER_SPEC_ADDENDUM_v1              | [CHƯA XÁC MINH]         | Selected extracts; see section 3 | Primary authority source for this SPC |

## 3. Consolidated Specification Content

## 3.1 ADN (Algorithm / Logic)

Note: In governance WS, 'ADN' is interpreted as the operational logic/invariants/workflows described in the sources.

### Source Extract: S1 — TÀI LIỆU ĐẶC TẢ HỆ THỐNG QUẢN TRỊ tooling_dfm_advisor SIPM SHIP.docx

Source: S1

TÀI LIỆU ĐẶC TẢ HỆ THỐNG QUẢN TRỊ tooling_dfm_advisor (SIPM / SHIP-FIRST GOVERNANCE MASTER SPECIFICATION)

Tài liệu này là bản xuất bản đầy đủ, đã chuẩn hóa theo cùng phong cách của bản MASTER SPEC tooling_dfm_advisor trước đó, và đồng thời tuân thủ bộ tiêu chuẩn Naming/Directory/Governance Invariants. Mục tiêu: vận hành được trong thực tế, không deadlock/loop, tamper-evident, fail-safe, và có khả năng brownfield onboarding.

THÔNG TIN PHIÊN BẢN (DOCUMENT METADATA)

spec_id: TOOLING_DFM_ADVISOR_GOVERNANCE_MASTER_SPEC_v1

status: APPROVED

effective_date_utc: [CHƯA XÁC MINH] (điền khi phát hành)

owner: PM / System Owner

tool_name: govctl

tool_api: GOVCTL_API_v1

policy_version_default: GOV_POLICY_v1

state_schema: SESSION_STATE_SCHEMA_v1

receipt_schema: RECEIPT_SCHEMA_v1

registry_schema: STEP_REGISTRY_SCHEMA_v1

genesis_contract: GENESIS_KIT_CONTRACT_v1

TẦM NHÌN VÀ MỤC TIÊU (VISION & OBJECTIVES)

1.1. Mục tiêu cốt lõi
Hệ thống cung cấp quy trình Governance-as-Code có thể vận hành trơn tru trong thực tế (local/dev/CI), với các mục tiêu:

Tránh deadlock hoặc vòng lặp vô hạn trong điều phối.

Tối ưu sử dụng context (policy + repo fingerprint + receipts) để ra quyết định có thể kiểm chứng.

Tuyệt đối không bypass các rào chắn chất lượng (non-bypassable quality).

Không làm suy giảm velocity một cách không kiểm soát (trade-off phải được đo và audit).

1.2. Tuyên bố thực tế (Reality statement)
Local environment không thể ngăn tuyệt đối can thiệp thủ công; do đó hệ thống tập trung vào:

Tamper-evident: mọi sửa tay đều bị phát hiện (receipt integrity).

Fail-safe coordination: không ghi đè trạng thái đồng thời; pause an toàn.

Policy-as-config: quy tắc ở config; tool chỉ thi hành logic.

Brownfield onboarding: áp dụng thực dụng lên dự án hiện hữu bằng GENESIS + template steps + baseline capture.

1.3. Chỉ số đo lường hiệu quả (KPIs)
Hệ thống đạt chuẩn khi:

Unbrick Registry/State và khôi phục hoạt động < 15 phút bằng LKG + GENESIS.

govctl validate-receipt phát hiện ngay lập tức receipt bị sửa tay (HMAC mismatch) và hard-fail.

PAUSE giữa tác vụ dài không làm hỏng state; không tạo concurrent overwrite.

Policy updates có audit trace và không trộn lẫn vào SESSION_STATE.json.

Brownfield onboarding thành công qua template steps + baseline capture mà không phải viết lại toàn bộ hệ thống cũ.

QUY ƯỚC ĐẶT TÊN (NAMING CONVENTIONS) – ZERO TOLERANCE

2.1. Định danh hệ thống, tool và artefacts

Tên hệ thống: tooling_dfm_advisor (snake_case, không khoảng trắng).

Tool điều phối: govctl (snake_case).

Step IDs trong registry: UPPER_CASE, dạng ENUM, không khoảng trắng. Ví dụ: RUN_CI_SKELETON, AUTHORITY_SELECT_NEXT_STEP.

Decision codes: UPPER_CASE, closed-set. Ví dụ: REGISTRY_CORRUPT, RETRY_EXHAUSTED.

2.2. Run ID
Mọi luồng phải có run_id định dạng ISO 8601 rút gọn: YYYYMMDDThhmmssZ.
Ví dụ: 20260212T071350Z.
run_id bắt buộc xuất hiện trong: receipts, audit logs, lock, state transitions.

2.3. Contracts & schemas versioned
Mọi schema/contract bắt buộc gắn \_v[X] và được xem là “API ổn định”:

RECEIPT_SCHEMA_v1

SESSION_STATE_SCHEMA_v1

STEP_REGISTRY_SCHEMA_v1

GOVERNANCE_POLICY_SCHEMA_v1

GENESIS_KIT_CONTRACT_v1

CẤU TRÚC THƯ MỤC BẮT BUỘC (MANDATORY DIRECTORY LAYOUT)

A) Commit required (trong repo)

config/

governance_policy.json (versioned; policy_version)

docs/runbooks/

step_registry.json

step_registry.LKG.json

GENESIS_KIT/ (bắt buộc tồn tại; xem Mục 5.3)

docs/audit/ (append-only)

SESSION_AUDIT_LOG.md

SESSION_STATE.json (coordination-only state)

SESSION_STATE.lock

LAST_SESSION_CHECKPOINT.md (khuyến nghị; append-only resume anchor)

docs/audit/receipts/

receipts/\*.json

logs/\*.stdout.log, \*.stderr.log

B) Local-only (không commit)

.secrets.env hoặc environment variables:

GOV_RECEIPT_HMAC_KEY=<secret>

(khuyến nghị) config/policy_override.local.json

dùng cho override tạm thời; phải nằm trong .gitignore.

Quy tắc append-only:

docs/audit/SESSION_AUDIT_LOG.md và LAST_SESSION_CHECKPOINT.md tuyệt đối chỉ append.

Nếu phát hiện chỉnh sửa tay các file append-only hoặc receipts: FAIL-FAST.

NGUYÊN TẮC VẬN HÀNH BẤT BIẾN (INVARIANT PRINCIPLES)

Hệ thống dựa trên 8 hard decisions (non-bypassable):

4.1. Receipt integrity (tamper-evident)
Mọi receipt tại local/CI phải có:

receipt_sha256 (sha256 của canonical payload)

receipt_sig_hmac_sha256 (hmac_sha256(secret, receipt_sha256))
Bất kỳ mismatch → TAMPERED + FAIL-FAST.

4.2. State concurrency (lock + pause-request)

Cấm concurrent overwrite của SESSION_STATE.json.

Mọi ghi state phải có lock hợp lệ; PAUSE dùng cờ PAUSE_REQUESTED và cooperative abort.

4.3. Governance update (policy-as-config, versioned)

Rules nằm trong config/governance_policy.json và có version.

SESSION_STATE.json chỉ dùng điều phối và counters; không chứa semantic claims.

Policy change phải audit: POLICY_UPDATED (ai đổi, đổi gì, lý do).

4.4. Brownfield integration (genesis + templates + baseline)
Onboarding dự án cũ phải đi qua:

GENESIS_KIT tối thiểu

template steps

baseline capture (repo fingerprint + CI receipt đầu tiên)

4.5. Unknown step hard-stop
Nếu current_step hoặc next_allowed_step không phân giải được từ:

step_registry.json (canonical), hoặc

step_registry.LKG.json, hoặc

GENESIS_KIT registry
thì bắt buộc:

FAIL-FAST

set session_state = PAUSED

set current_step = PAUSE_FOR_AUTHORITY_DECISION

ghi audit event: UNKNOWN_STEP_ID (kèm field_name và step_id).

4.6. Decision codes only (closed-set)
Cấm trường điều khiển tự do (NEXT_ACTION, next_action, decision.next_action…).
Mọi rẽ nhánh dùng decision_code thuộc closed-set và map qua decision_map trong policy.
Nếu ánh xạ thất bại:

session_state = PAUSED

current_step = PAUSE_FOR_AUTHORITY_DECISION

audit event: DECISION_UNMAPPED.

4.7. Anti-loop strategy (retry bounded + progress guard)

Retry hợp lệ chỉ khi receipt mới:

pass validate-receipt (HMAC ok)

và có tiến triển (NO_PROGRESS guard không kích hoạt; dựa trên git diff / receipt delta / step evidence).

Vượt max_retry_per_step:

decision_code = RETRY_EXHAUSTED

chuyển PAUSE_FOR_AUTHORITY_DECISION

không được “đẻ step ad-hoc” trong state.

4.8. GENESIS invariants (non-bypassable)
govctl validate-registry phải hard-fail nếu:

thiếu docs/runbooks/GENESIS_KIT/

hoặc thiếu bất kỳ core step nào của GENESIS.
Executor bị cấm chạy khi GENESIS không đầy đủ.

KIẾN TRÚC TÀI LIỆU VÀ MÔ HÌNH TRẠNG THÁI (ARTEFACTS & STATE MODEL)

5.1. Minimum governance kernel (bắt buộc)
Commit required:

docs/audit/SESSION_AUDIT_LOG.md (append-only events)

docs/audit/SESSION_STATE.json (coordination-only)

docs/audit/SESSION_STATE.lock

docs/runbooks/step_registry.json và step_registry.LKG.json

docs/runbooks/GENESIS_KIT/

docs/audit/receipts/ (receipts + logs)

config/governance_policy.json (versioned)

Local-only:

GOV_RECEIPT_HMAC_KEY trong .secrets.env hoặc env var

config/policy_override.local.json (gitignored)

5.2. SESSION_STATE.json (coordination-only; SESSION_STATE_SCHEMA_v1)
SESSION_STATE.json là mỏ neo điều phối. ĐƯỢC PHÉP lưu:

schema_version

session_state: RUNNING | PAUSED

current_step, next_allowed_step (step_id hợp lệ)

retry_counters (map step_id -> count)

last_known_git: { head, dirty, repo_fingerprint }

last_ci_receipt: { id, status, timestamp_utc }

lock_metadata (nếu cần, nhưng lock file là nguồn chính)

decision_code (closed-set; không phải step_id)

TUYỆT ĐỐI KHÔNG lưu:

semantic claims (“đã xong feature X”)

trường điều khiển tự do (NEXT_ACTION, next_action, decision.next_action…)
Nếu phát hiện các trường cấm → govctl validate-state FAIL-FAST.

CƠ CHẾ VẬN HÀNH CỐT LÕI (CORE MECHANISMS)

6.1. Receipt integrity bằng HMAC (RECEIPT_SCHEMA_v1)
Receipt bắt buộc có:

receipt_sha256

receipt_sig_hmac_sha256

Canonical JSON payload (để tính receipt_sha256):

sort keys theo thứ tự alphabet

không có whitespace thừa

không bao gồm hai trường chữ ký (receipt_sha256, receipt_sig_hmac_sha256)

govctl validate-receipt:

parse JSON schema

rebuild canonical payload

recompute sha256, compare receipt_sha256

recompute hmac, compare receipt_sig_hmac_sha256

validate hashes của stdout/stderr (nếu có)
Nếu mismatch → status = TAMPERED, hard-fail.

6.2. Lock & pause-request (state concurrency)
Advisory lock:

file: docs/audit/SESSION_STATE.lock

tối thiểu fields: pid (hoặc run_id), owner_step_id, start_utc, ttl_seconds
Quy tắc:

phải có lock trước khi ghi SESSION_STATE.json

lock còn TTL → STATE_LOCKED

quá TTL → cho phép steal lock nhưng bắt buộc audit event: LOCK_STOLEN

Pause-request:

file: docs/audit/PAUSE_REQUESTED (atomic create)

lệnh PAUSE chỉ tạo cờ; không ghi đè SESSION_STATE.json
Cooperative abort:

tác vụ dài phải chạy trong wrapper kiểm tra PAUSE_REQUESTED theo chu kỳ 5–10s

nếu phát hiện: dừng an toàn, tạo receipt ABORTED_BY_PAUSE, release lock, exit code đặc biệt

chỉ sau khi lock được release, controller mới set SESSION_STATE.json = PAUSED.

6.3. Policy-as-config (GOVERNANCE_POLICY_SCHEMA_v1)
config/governance_policy.json chứa:

policy_version (ví dụ GOV_POLICY_v1)

decision_map (decision_code -> next_step_id)

max_retry_per_step

lock_ttl_seconds

genesis_core_steps (list)

ci_quality_rules (EMPTY_TEST_SUITE, PASS_SCAFFOLD time-bomb…)
Policy update là “safe lane” nhưng bắt buộc:

validate toàn diện

audit event POLICY_UPDATED (actor, diff, justification)

6.4. Brownfield integration
3 bước bắt buộc:

init GENESIS_KIT tối thiểu (xem 7.3)

template steps (theo mức thay đổi: DOC_ONLY_CHANGE, CODE_CHANGE_SMALL, CODE_CHANGE_LARGE…)

baseline capture:

step_id: BROWNFIELD_BASELINE_CAPTURE

ghi git head hash, repo_fingerprint, tạo CI receipt đầu tiên, set trạng thái khởi điểm
Từ điểm baseline trở đi áp dụng governance loop mới.

QUY TẮC THỰC THI & KIỂM SOÁT CHẤT LƯỢNG (QUALITY GATES)

7.1. Non-bypassable quality

EMPTY_TEST_SUITE:

nếu phát hiện số test = 0 → receipt status = FAIL, error_code = EMPTY_TEST_SUITE

Time-bomb scaffold:

cho phép PASS_SCAFFOLD tồn tại trong số sprint giới hạn (policy)

hết hạn: mọi CODE_CHANGE → FAIL cho đến khi scaffold được thay bằng implementation thật

7.2. Lanes & anti-loop
Lane selection:

govctl lane-check quyết định Fast/Safe dựa trên config + git diff (không để AI tự chọn).
Safe lane bắt buộc có ADR (nếu dự án áp dụng ADR).
Anti-loop:

retry chỉ hợp lệ nếu receipt mới pass validate-receipt và có progress (NO_PROGRESS guard không kích hoạt).

vượt max_retry_per_step → PAUSE_FOR_AUTHORITY_DECISION.

7.3. GENESIS core steps bắt buộc (closed list)
validate-registry hard-fail nếu thiếu bất kỳ step nào:

GOV_RECOVER_REGISTRY

GOV_VALIDATE_REGISTRY_AND_STATE

GOV_CAPTURE_COMMAND_EVIDENCE

GOV_VALIDATE_RECEIPT

RUN_CI_SKELETON

PAUSE_FOR_AUTHORITY_DECISION

AUTHORITY_SELECT_NEXT_STEP

REVERT_LAST_CHANGE_GIT (bắt buộc nếu dự án dùng Git)

VÒNG LẶP VẬN HÀNH TIÊU CHUẨN (STANDARD OPERATIONAL LOOP)

Chu trình vận hành chuẩn gồm 9 bước, phải theo thứ tự:

govctl validate-registry

resolve chain: CANONICAL → LKG → GENESIS

hard-fail nếu thiếu GENESIS core

govctl validate-state

parse + schema validate

resolve current_step/next_allowed_step

detect forbidden fields (NEXT_ACTION…)

nếu unresolved step → PAUSE_FOR_AUTHORITY_DECISION + audit UNKNOWN_STEP_ID

govctl sanity-check

git head/dirty

repo_fingerprint

validate last receipt signature

govctl lane-check

xác định Fast/Safe lane theo config + diff

AUTHORITY_SELECT_NEXT_STEP

chọn đúng 1 next_allowed_step hợp lệ

executor run (wrapper protected)

chạy step được chọn

tạo receipt + sign HMAC

tôn trọng PAUSE_REQUESTED (cooperative abort)

govctl validate-receipt

verify sha256/hmac + attached logs hashes + exit code

append audit log

append vào docs/audit/SESSION_AUDIT_LOG.md

update state

acquire lock → write SESSION_STATE.json safely → release lock

[CHƯA XÁC MINH] Source S1 contains additional content beyond the extract cap (220 paragraphs). If full verbatim inclusion is required, provide instruction to remove the cap.

### Source Extract: S4 — TOOLING_DFM_ADVISOR_GOVERNANCE_MASTER_SPEC_ADDENDUM_v1.docx

Source: S4

TOOLING_DFM_ADVISOR GOVERNANCE (SIPM/SHIP-FIRST) – ADDENDUM V1

Operational deltas for policy-as-config, decision codes, and non-bypassable gates

1. Purpose and authority

This addendum defines additional governance requirements that must be enforced by govctl, policy-as-config, and CI gates.

It focuses on preventing process breaks (unknown steps, unmapped decisions, unverified receipts) and on enforcing new industrial invariants (report format, ephemeral purge, golden corpus).

2. Normative deltas (MUST)

Governance policy MUST explicitly encode decision_map entries for report delivery and ephemeral purge outcomes (including failure paths).

Receipts MUST carry policy_version and rule_version, and MUST be tamper-evident with receipt_sha256 + receipt_sig_hmac_sha256.

Unknown step hard-stop remains mandatory; additionally, a SPEC_DRIFT hard-stop MUST pause execution when contract/envelope naming deviates from the approved spec/addendum set.

CI MUST enforce EMPTY_TEST_SUITE and GOLDEN_CORPUS_MISSING as non-bypassable quality rules.

Delivery invariants MUST be enforced as gates: REPORT_FORMAT_FORBIDDEN and EPHEMERAL_PURGE_REQUIRED are fail-fast conditions.

3. Decision codes and step ids (closed-set)

1. Receipt schema deltas (RECEIPT_SCHEMA_v1 compatible)

Receipts remain compatible with RECEIPT_SCHEMA_v1, but MUST include these additional fields (either as top-level or within a stable metadata object):

- policy_version

- rule_version

- geometry_version_id (when geometry is in-scope)

- artifact_manifest_sha256 (when producing report artifacts)

- ephemeral_purge_evidence: {purged_item_count, workspace_id, verified_empty: bool} (no file paths)

Any missing required field MUST cause validate-receipt to fail for steps that claim delivery_complete.

5. Policy-as-config keys to add (minimum)

1. Spec Sync Gate (non-bypassable)

govctl MUST implement a spec-sync check before executing any CODE_CHANGE lane:

- Compute hashes of approved specification artefacts (base specs + addenda).

- If repo changes touch contracts/, agents/, providers/, reporter/, or orchestration/ AND spec-sync hashes are not updated, emit decision_code SPEC_DRIFT and pause.

This prevents code from drifting away from the authority documents.

7. Verification checklist (Go/No-Go)

validate-registry resolves CANONICAL -> LKG -> GENESIS; missing GENESIS core steps hard-fail.

validate-state rejects forbidden free-form control fields; decision_code must be closed-set.

validate-receipt detects tampering (sha256/hmac mismatch) and fails fast.

CI gates enforce EMPTY_TEST_SUITE and GOLDEN_CORPUS_MISSING.

Delivery_complete requires EPHEMERAL_PURGE_RESULT evidence; absence triggers EPHEMERAL_PURGE_REQUIRED.

Spec Sync Gate pauses on SPEC_DRIFT when impacted code changes occur without spec hash updates.

8. Change log

v1 (REVIEW): initial governance addendum enforcing delivery/purge and spec-sync gates.

## 3.2 AMBR (Bindings / Module Contract / Execution)

Note: In governance WS, 'AMBR' is interpreted as contracts/schemas, directory layout, state model, and execution bindings described in sources.

### Source Extract: S2 — TÀI LIỆU ĐẶC TẢ HỆ THỐNG tooling_dfm_advisor (MASTER SPECIFICATION).docx

Source: S2

TÀI LIỆU ĐẶC TẢ HỆ THỐNG: tooling_dfm_advisor (MASTER SPECIFICATION)

Tài liệu này là bản xuất bản đầy đủ, đã chuẩn hóa theo Bộ tiêu chuẩn Naming/Directory/Report/Governance Invariants mới. Mục tiêu là đảm bảo tài liệu đủ điều kiện chuyển tiếp sang giai đoạn lập trình mà không phát sinh “điểm gãy quy trình” khi triển khai CI/CD và orchestrator (importlib, step_registry, receipts, audit).

THÔNG TIN PHIÊN BẢN (DOCUMENT METADATA)

spec_id: TOOLING_DFM_ADVISOR_MASTER_SPEC_v1

status: APPROVED

effective_date_utc: [CHƯA XÁC MINH] (điền khi phát hành)

owner: PM / System Owner

rule_version_default: RULESET_v1

canonical_ui_contract: CANONICAL_UI_CONTRACT_v1

TẦM NHÌN, MỤC TIÊU & PHẠM VI (VISION & SCOPE)

1.1. Tầm nhìn
tooling_dfm_advisor là hệ thống Context-Aware Digital Twin cho chi tiết nhựa ép, sẵn sàng triển khai công nghiệp cho các nghiệp vụ:

Post-Design Validation (xác thực thiết kế sau khi hoàn thành).

Pre-Mold Design Review (đánh giá trước khi thiết kế khuôn).

Batch/Queue Operation (vận hành quy mô lớn theo hàng đợi).

Hệ thống kết hợp:

Hình học chính xác: B-Rep (Boundary Representation).

Hình học dẫn xuất hiệu năng cao: Mesh (tessellated).

Nhận diện đặc trưng: AFR (Automatic Feature Recognition).

Quy tắc sản xuất theo ngữ cảnh: Context = {material, machine, process}.

1.2. Mục tiêu cốt lõi
Hệ thống không chỉ phát hiện vấn đề (analysis engine) mà bắt buộc tạo “Advisory Artifact” để con người đọc, kiểm toán, ký duyệt. Advisory Artifact được xem là đầu ra công nghiệp.

1.3. Giới hạn hệ thống (Explicit Non-Goals & Boundary)
Không làm:

Không CAE (flow/warpage/cooling/shrinkage).

Không auto-fix, không tối ưu hóa thiết kế.

Không thay thế CAD hoặc phần mềm thiết kế khuôn.

Không tạo hình học mới hoặc chỉnh sửa topology.

Không đưa ra phán quyết pass/fail mang tính pháp lý hay điều khiển sản xuất tự động.

Canonical Ceiling (định nghĩa “hệ thống tooling_dfm_advisor hợp lệ” tối thiểu):

Có Geometry Preparation Core và Geometry Validation Gate.

Có cơ chế Geometry Freeze (AGL bất biến).

Có ít nhất 01 analysis_agent trả lời được 01 câu hỏi sản xuất cụ thể.

Có report artifact hỗ trợ audit (PDF/HTML). Mọi mở rộng vượt trần này phải được xem là một hệ thống khác.

QUY ƯỚC ĐẶT TÊN (NAMING CONVENTIONS) – ZERO TOLERANCE

2.1. Agents & Providers

Module/actor phải snake_case toàn chữ thường.

Analysis agents: hậu tố \_agent. Ví dụ: draft_agent, undercut_agent, thickness_agent, weldline_agent, sink_agent, ribratio_agent.

Providers: hậu tố \_provider. Ví dụ: importer_provider, healer_agent (lưu ý healer là agent xử lý chuẩn bị), afr_provider, mapping_provider.

2.2. Contracts & APIs (Versioned)
Mọi contract/API định danh dạng UPPER_CASE và bắt buộc có \_v[X]. Ví dụ:

TOOLING_MODEL_API_v1

STEP_IMPORT_API_v1

GEOMETRY_STACK_API_v1

INTELLIGENCE_STACK_API_v1

ANALYSIS_RESULTS_STACK_API_v1

CANONICAL_UI_CONTRACT_v1

2.3. Result Envelopes (Versioned)
Kết quả trả về từ agent/provider phải UPPER_CASE và có hậu tố phiên bản thuật toán \_v0 (hoặc vN nếu thay đổi thuật toán):

STEP_IMPORT_RESULT_v0

GEOMETRY_PREP_RESULT_v0

GEOMETRY_VALIDATION_RESULT_v0

GEOMETRY_FREEZE_RESULT_v0

MAPPING_RESULT_v0

DRAFT_ANALYSIS_RESULT_v0, UNDERCUT_ANALYSIS_RESULT_v0, THICKNESS_ANALYSIS_RESULT_v0, WELDLINE_ANALYSIS_RESULT_v0, SINK_ANALYSIS_RESULT_v0, RIBRATIO_ANALYSIS_RESULT_v0

REPORT_DELIVERY_RESULT_v0

2.4. Run ID (Traceability)
Mọi luồng thực thi phải có run_id định dạng ISO 8601 rút gọn: YYYYMMDDThhmmssZ
Ví dụ: 20260212T071350Z
run_id là định danh xuyên suốt receipts/defects/report.

CẤU TRÚC THƯ MỤC BẮT BUỘC (MANDATORY DIRECTORY LAYOUT)

Mã nguồn phải cô lập giữa Core Logic, Governance, và Audit/Receipts. Cấu trúc chuẩn:

config/

governance_policy.json (Policy-as-Config; rule_version, thresholds, gates, toggles)

governance_allowlist.json (nếu cần; danh sách thư viện/feature được phép)

docs/runbooks/

step_registry.json (đăng ký các step; orchestrator tra cứu)

GENESIS_KIT/ (bộ khởi tạo cốt lõi cho hệ thống mới)

docs/audit/ (append-only)

SESSION_AUDIT_LOG.md (nhật ký sự kiện)

SESSION_STATE.lock (khóa tương tranh)

SESSION_STATE.json (mỏ neo điều phối; trạng thái phiên)

LAST_SESSION_CHECKPOINT.md (mỏ neo resume; append-only)

docs/audit/receipts/

receipts/\*.json (biên lai từng bước)

logs/\*.stdout.log, \*.stderr.log (log đầu ra từng bước)

tooling_dfm_advisor/ (mã nguồn chính)

geometry/ (B-Rep/Mesh backend)

contracts/ (data models; APIs versioned)

agents/ (analysis agents)

providers/ (importer_provider, afr_provider, mapping_provider, healer_agent)

reporter/ (reporter_agent, visualizer_agent nếu có)

orchestration/ (pipeline runner, receipt writer, state machine)

ui_contract/ (CANONICAL_UI_CONTRACT_v1 mappings)

Quy tắc: docs/audit là append-only. Không sửa/xóa bằng tay. Nếu phát hiện chỉnh tay: FAIL-FAST theo invariant tamper-evident.

NGUYÊN TẮC VẬN HÀNH BẤT BIẾN (INDUSTRIAL GOVERNANCE INVARIANTS)

4.1. Advisory System, không auto-reject
Hệ thống tư vấn, không auto-reject. Không có quyết định pháp lý “pass/fail”.

4.2. Minh bạch thuật toán
Không dùng heuristic mù mờ/ML black-box không kiểm soát. Nếu có heuristic: phải mô tả rõ rule_version, threshold, điều kiện áp dụng.

4.3. SSOT tuyệt đối
TOOLING_MODEL_API_v1 là nguồn chân lý duy nhất. UI/Visualization/Narrative/Report là view/artifact, không có quyền mutate SSOT.

4.4. No Shadow Data
Không cho phép external geometry state, cache hình học ngoài kiểm toán, hoặc ghi dữ liệu không có receipt. Bất kỳ asset trung gian nào phải thuộc “ephemeral storage” và bị xóa sau delivery_complete.

4.5. Ephemeral Asset Storage (Non-bypassable)
Toàn bộ PNG/SVG/Mesh phát sinh trong phân tích:

Chỉ tồn tại trong vùng tạm (ephemeral workspace) gắn run_id.

BẮT BUỘC xóa sạch ngay sau trạng thái delivery_complete.

Không để “shadow data” trên máy chủ/máy trạm.

Sau khi xóa: mọi tham chiếu trong Defect/Report phải được scrub (không còn đường dẫn file), chỉ giữ hash/manifest nếu cần truy vết.

4.6. Golden Geometry QA (Regression)
Bắt buộc duy trì Golden Geometry Corpus (STEP đã kiểm duyệt) + Expected Defect Set.
CI regression chạy theo corpus này.
CI Skeleton gate: nếu có thay đổi mã nguồn mà không có test mới đi kèm ở phạm vi liên quan → FAIL với mã lỗi: EMPTY_TEST_SUITE.

4.7. Canonical UI Contract (Non-bypassable)
UI không phải SSOT; UI không được healing, không sửa defect, không bật/tắt agent thủ công.
UI bắt buộc hiển thị 5 Global State:

geometry_fidelity

degraded_mode

afr_state

analysis_completeness

report_validity
Ẩn bất kỳ state nào → vi phạm CANONICAL_UI_CONTRACT_v1.

4.8. Tamper-evident receipts (Non-bypassable)
Mọi lệnh/step thực thi phải sinh receipt JSON cục bộ kèm:

receipt_sha256

receipt_sig_hmac_sha256
Nếu phát hiện receipt bị chỉnh tay hoặc hash/signature mismatch → FAIL-FAST ngay.

4.9. Fail-safe cho thư viện hình học nặng
Nếu thiếu dependency C++ nặng (ví dụ pythonocc-core):

Không crash pipeline.

Provider/agent trả về envelope status = NOT_EXECUTED hoặc STUB_MODE với error_code có cấu trúc.

Pipeline vẫn phải tạo report (tối thiểu Geometry Health Report).

KIẾN TRÚC TỔNG THỂ (ARCHITECTURE)

5.1. SSOT: tooling_model (TOOLING_MODEL_API_v1)
tooling_model là container runtime duy nhất đại diện trạng thái hệ thống tại một thời điểm.
UI/Visualization/Narrative/Report không thuộc tooling_model và không có quyền ghi ngược vào tooling_model.
Không tồn tại agent-local state ghi ngược vào SSOT.

5.2. Cấu trúc tooling_model (Stacks – versioned contracts)
A) geometry_stack (GEOMETRY_STACK_API_v1)

AGL (Authoritative Geometry Layer): B-Rep, nguồn chân lý topology/kích thước danh nghĩa.

Trạng thái AGL: bất biến sau “freeze”.

DGL (Derived Geometry Layer): mesh + canonical mapping table (b2m, m2b) gắn geometry_version_id của AGL.

Quy tắc: SSOT chỉ chứa DGL dùng chung. Dẫn xuất đặc thù của agent phải là ephemeral và bị hủy sau khi agent hoàn tất.

B) intelligence_stack (INTELLIGENCE_STACK_API_v1)

Context providers: material, machine, process.

AFR feature graph: không mutate hình học.

afr_state hợp lệ: afr_verified hoặc afr_uncertain.

C) analysis_results_stack (ANALYSIS_RESULTS_STACK_API_v1)

Append-only.

Chứa defect objects (không sửa/xóa).

HỢP ĐỒNG DỮ LIỆU CỐT LÕI (CORE DATA CONTRACTS)

6.1. Defect object (DEFECT_OBJECT_API_v1)
Tối thiểu fields:

id (traceable)

run_id (YYYYMMDDThhmmssZ)

agent_source (snake_case, ví dụ draft_agent)

rule_version (ví dụ RULESET_v1)

geometry_version_id

timestamp_utc (ISO 8601 đầy đủ hoặc rút gọn; phải chỉ rõ UTC)

geometry_vector: {brep_face_normal, mesh_smoothed_normal, hybrid_preference}

operational_vector: ejection_direction (user_confirmed)

analysis_value, threshold_applied

advisory (machine-readable + human-readable summary)

confidence_score

assets_ephemeral_ref:

snapshot_png_ref (ephemeral token; không lưu path bền vững)

section_svg_ref (ephemeral token; không lưu path bền vững)

delivery_scrubbed (boolean): true sau delivery_complete (khẳng định đã scrub refs)

6.2. Result envelope (RESULT_ENVELOPE_API_v1)
Áp dụng cho mọi \*\_RESULT_v0.
Fields tối thiểu:

result_type (ví dụ DRAFT_ANALYSIS_RESULT_v0)

run_id

status: OK | FAILED | NOT_EXECUTED | STUB_MODE

error_code (structured)

disable_reason (nếu NOT_EXECUTED/STUB_MODE)

rule_version

geometry_version_id (nếu có)

metrics (tuỳ loại)

produced_defects: [defect_id…] (nếu có)

receipts_ref (receipt id/path trong receipts store)

6.3. Error codes (gợi ý cấu trúc, không giới hạn)

OCC_MISSING

STEP_IMPORT_FAILED

GEOMETRY_VALIDATION_FAILED

HEALING_EXCEEDED_LIMIT

MAPPING_SPATIAL_INTEGRITY_FAILED

DGL_REGEN_EXCEEDED_LIMIT

REPORT_FORMAT_FORBIDDEN

RECEIPT_TAMPER_DETECTED

EMPTY_TEST_SUITE

QUY TRÌNH THỰC THI CHUẨN (CANONICAL PIPELINE)

Pipeline gồm 8 bước; bắt buộc chạy Bước 7 (Report) trong mọi chế độ (kể cả thất bại). Pipeline chỉ hoàn tất khi report được tạo và (nếu vận hành giao khách hàng) delivery_complete.

Bước 0 (bắt buộc, trước Bước 1): run_id allocation + receipt bootstrap

Sinh run_id theo YYYYMMDDThhmmssZ (UTC).

Tạo receipt “RUN_INIT_RESULT_v0” (hoặc gộp vào bước 1) và ghi SESSION_STATE.json anchor.

Bước 1: contextual_inception (Contextual Inception) – provider

Thu thập/chốt material, machine, process.

Thu thập operational_vector: ejection_direction.

Bắt buộc user_confirmed = true cho ejection_direction.

Output: CONTEXT_INCEPTION_RESULT_v0 (status OK hoặc FAILED). Luôn có receipt.

Bước 2: geometry_preparation_core – providers tuần tự

STEP import & normalization (STEP_IMPORT_API_v1):

Nhập STEP, chuẩn hóa đơn vị, đưa hệ trục về OXYZ, chuẩn hóa normal để giảm mơ hồ.

Không thay đổi topology.

Output: STEP_IMPORT_RESULT_v0.

Xác nhận ejection_direction (must remain user_confirmed).

healing_agent:

ShapeFix, edge sewing, sliver face removal, thống nhất tolerance.

tolerance mặc định phải đến từ config/governance_policy.json theo rule_version.

Output: GEOMETRY_PREP_RESULT_v0.

Bước 3: geometry_freeze_agl – validation gate + freeze

geometry_validation_gate:

Kiểm self-intersection, non-manifold, volume integrity.

Output: GEOMETRY_VALIDATION_RESULT_v0.

iterative_healing (tối đa 3 vòng):

Mỗi vòng: ghi tolerance_delta, phát hành geometry_version_id mới, provenance metadata.

Không thay đổi topology cấp cao và không vượt dung sai kỹ thuật theo Context.

tolerance engineering formula: [CHƯA XÁC MINH] (phải được định nghĩa trong governance_policy.json hoặc ADN tương ứng; nếu chưa có thì giới hạn sử dụng tolerance fixed + audit warning).

Nếu thất bại sau 3 vòng:

Chuyển Fallback Mode: dừng analysis agents, vẫn tạo report tối thiểu “Geometry Health Report”.

Regression snapshot:

Ghi hash hình học, geometry_version_id, topology invariants (solid/face/edge count, bounding box, Euler).

Freeze AGL:

Output: GEOMETRY_FREEZE_RESULT_v0.

Bước 4: data_enrichment_dgl – providers

mapping_provider:

Sinh mesh (DGL) + canonical mapping table (b2m, m2b).

Spatial Integrity Check: sample points on mesh → project to B-Rep → compute Δ.

Nếu Δ > tolerance: invalidate DGL và regenerate.

regeneration_limit: tối đa 3 lần. Vượt quá:

set geometry_fidelity downgraded

set degraded_mode = true

ghi rõ nguyên nhân vào report.

Output: MAPPING_RESULT_v0.

Bước 5: parallel_analysis – analysis agents (song song, read-only)

Các analysis_agent chạy đồng thời, không chia sẻ trạng thái, không mutate SSOT.

Nếu agent lỗi/thiếu dependency:

không làm sập pipeline

trả envelope NOT_EXECUTED hoặc STUB_MODE kèm error_code và disable_reason

Default agents:

draft_agent → DRAFT_ANALYSIS_RESULT_v0

undercut_agent → UNDERCUT_ANALYSIS_RESULT_v0

thickness_agent → THICKNESS_ANALYSIS_RESULT_v0

weldline_agent → WELDLINE_ANALYSIS_RESULT_v0

sink_agent → SINK_ANALYSIS_RESULT_v0

ribratio_agent → RIBRATIO_ANALYSIS_RESULT_v0

Bước 6: advisory_synthesis

Tổng hợp tư vấn machine-readable, confidence_score, limitation tags.

Output: ADVISORY_SYNTHESIS_RESULT_v0.

Bước 7: report_composition_and_delivery (Reporter subsystem)

reporter_agent (read-only) tạo báo cáo ONLY PDF hoặc HTML.

Visuals:

snapshots: PNG

sections: SVG (vector)

Canonical layout 6 phần bắt buộc:

Executive Summary

Geometry Health Status

[CHƯA XÁC MINH] Source S2 contains additional content beyond the extract cap (220 paragraphs). If full verbatim inclusion is required, provide instruction to remove the cap.

### Source Extract: S3 — TOOLING_DFM_ADVISOR_MASTER_SPEC_ADDENDUM_v1.docx

Source: S3

TOOLING_DFM_ADVISOR MASTER SPECIFICATION – ADDENDUM V1

Delta controls for DFM-first compliance, contracts, and report delivery invariants

1. Purpose and authority

This addendum introduces non-bypassable deltas that must be enforced in documentation, policy-as-config, and implementation.

It is intended to prevent drift between module V1 documents (ADN/AMBR/LDR) and the system-level specification.

If any statement conflicts with the base spec, this addendum takes precedence for the items listed in Section 2.

2. Normative deltas (MUST)

Naming is zero-tolerance. Agents/providers are snake_case with suffix \_agent/\_provider; contracts are UPPER_CASE with \_vX; result envelopes are UPPER_CASE with algorithm suffix \_vN; run_id format is YYYYMMDDThhmmssZ (UTC).

All provider/agent execution MUST be fail-safe: no unhandled exceptions may terminate the pipeline. Failures MUST be returned as versioned result envelopes with status and structured error_code/disable_reason.

Final report delivery is restricted to PDF or HTML only. Any other deliverable format is forbidden and MUST trigger FAIL-FAST with error_code REPORT_FORMAT_FORBIDDEN.

Report visuals are constrained: snapshots as PNG; cross-sections as SVG (vector). The report model MUST not embed persistent file paths.

Ephemeral assets (PNG/SVG/Mesh and any agent-specific derived geometry) MUST be purged immediately after delivery_complete. Defect/report references MUST be scrubbed (no file paths). Purge MUST produce a receipt EPHEMERAL_PURGE_RESULT_v1.

Golden Geometry QA is mandatory: regression tests MUST run on a curated STEP corpus with expected defect sets. Any code change without associated tests in the impacted scope MUST fail with EMPTY_TEST_SUITE (or equivalent CI rule).

Canonical UI Contract is mandatory: UI cannot mutate SSOT and MUST display the 5 global states: geometry_fidelity, degraded_mode, afr_state, analysis_completeness, report_validity.

3. Contract tightening for DFM-first pipeline

3.1 Result envelope minimum fields

All \*\_RESULT_vN envelopes MUST include at least:

- result_type

- run_id

- status: OK | FAILED | NOT_EXECUTED | STUB_MODE

- error_code (structured; null when OK)

- disable_reason (required when NOT_EXECUTED/STUB_MODE)

- rule_version

- geometry_version_id (when geometry is in-scope)

- issues[] (machine-readable; may be empty)

- receipts_ref (local receipt id; no external pointers)

3.2 Defect object minimum fields

All defects MUST include run_id, agent_source, rule_version, geometry_version_id, and must not contain persistent file paths.

Any asset references MUST be ephemeral tokens and MUST be scrubbed after delivery_complete.

4. Geometry core enablers

geometry_validation_gate is mandatory before AGL freeze. Validation failures MUST be explicit in GEOMETRY_VALIDATION_RESULT_vN.

Iterative healing is bounded (default 3 loops). Each loop MUST produce evidence (tolerance_delta, topology invariants, geometry_version_id change).

If geometry cannot be validated within bounds, the system MUST enter fallback mode: analysis agents may be skipped, but report generation MUST still complete with a Geometry Health Report.

5. Reporter subsystem boundary and deliverables

Reporter is split into:

- Report composition: mapping GeoModelResponse to a canonical report_model.

- Report delivery: rendering report_model to PDF/HTML and managing ephemeral assets.

Composition MUST output a versioned envelope (e.g., REPORT_COMPOSITION_RESULT_v1).

Delivery MUST output REPORT_DELIVERY_RESULT_v1 and MUST also emit EPHEMERAL_PURGE_RESULT_v1 after successful delivery_complete.

6. Policy-as-config keys to add (minimum)

1. Verification checklist (Go/No-Go)

Naming audit passes (imports resolvable; module/file/class names match conventions).

Running pipeline in missing-backend mode returns NOT_EXECUTED/STUB_MODE envelopes and still produces a minimal report artifact (PDF/HTML).

REPORT_FORMAT_FORBIDDEN triggers FAIL-FAST when a non-PDF/HTML deliverable is requested.

After delivery_complete, ephemeral workspace is empty; purge receipt EPHEMERAL_PURGE_RESULT_v1 exists and validates.

Golden Geometry regression suite runs; changes to affected modules include tests; EMPTY_TEST_SUITE gate remains non-bypassable.

UI surfaces the 5 global states and does not mutate SSOT.

8. Change log

v1 (REVIEW): initial addendum introducing mandatory deltas for DFM-first compliance.

## 3.3 LDR (Dependencies / Libraries / Runtime constraints)

Source: S1–S4

The provided sources are primarily governance/system specifications and do not consistently enumerate concrete software libraries and runtime dependencies as a separate section. Where explicit dependencies are mentioned (e.g., hashing/HMAC concepts, directory layout, receipt logging), they are preserved in the extracts above. Any missing explicit dependency list remains [CHƯA XÁC MINH].

## 4. Open Questions / [CHƯA XÁC MINH]

|           |                                                                                                                                                                                          |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| source_id | open_question_excerpt                                                                                                                                                                    |
| S1        | effective_date_utc: [CHƯA XÁC MINH] (điền khi phát hành)                                                                                                                                 |
| S1        | PHẦN [CHƯA XÁC MINH] CẦN CHỐT TRONG POLICY-AS-CONFIG (KHÔNG CẢN TRỞ IMPLEMENT NẾU ĐÃ CÓ FAIL-SAFE)                                                                                       |
| S2        | effective_date_utc: [CHƯA XÁC MINH] (điền khi phát hành)                                                                                                                                 |
| S2        | tolerance engineering formula: [CHƯA XÁC MINH] (phải được định nghĩa trong governance_policy.json hoặc ADN tương ứng; nếu chưa có thì giới hạn sử dụng tolerance fixed + audit warning). |
| S2        | PHẦN [CHƯA XÁC MINH] CẦN CHỐT TRONG POLICY-AS-CONFIG (KHÔNG CẢN TRỞ CODING NẾU ĐÃ CÓ FAIL-SAFE)                                                                                          |

## 5. Change Log (Editorial)

This document is produced by rename + consolidation + re-structuring only; no semantics change is intended.

- Canonicalized filename and inserted control block/mapping tables (Run ID: 20260304T115902Z).

- Consolidated multiple governance/system specs and addenda into one WS_GOV ALL SPC with traceability markers.

- Any text not present in sources is marked [CHƯA XÁC MINH].

## 6. Appendix

6.1 Glossary / Abbreviations: [CHƯA XÁC MINH]

6.2 Cross references: See WS_GOV-ALL-POL-v1 and module-specific SPC/POL documents for detailed thresholds/severity maps.
