**WS_GOV-ALL-SPC-v1 - Governance Consolidated Specification**

*Status: REVIEW | Run ID: 20260228T081009Z | Document governance consolidation only*

# 0. Document Control Block

|                                      |                                                                      |
| ------------------------------------ | -------------------------------------------------------------------- |
| **Canonical Code**                   | WS_GOV-ALL-SPC-v1                                                    |
| **Status**                           | REVIEW                                                               |
| **Run ID (UTC)**                     | 20260228T081009Z                                                     |
| **Created date (UTC)**               | 2026-02-28T08:10:09Z                                                 |
| **WS / Module / Doc Type / Version** | WS_GOV / ALL / SPC / v1                                              |
| **Purpose**                          | Document governance consolidation only; no runtime/pipeline changes. |

## 1. Scope

This SPC consolidates governance-as-code specifications for tooling_dfm_advisor under WS_GOV / ALL. It normalizes document naming and packaging and preserves the technical content of the provided sources. The consolidated scope includes: governance principles, state/lock model, receipts integrity, registry resolution chain, decision-code closed-set behavior, and the non-bypassable quality gates required for DFM-first operations.

Non-goals: No new governance logic is introduced; no architecture redesign; no new pipelines; no changes to semantics. Any missing information is recorded as [CHƯA XÁC MINH] with explicit questions and impact.

Scope alignment note: The system-level master spec (S1) and system addendum (S3) contain broader, cross-WS requirements. Only the governance-relevant portions are included here; non-governance content (e.g., full DFM agent specs) is excluded and should remain under its own WS canonical documents.

## 2. Included Sources & Mapping Table

|               |                                                                          |                                                                |                             |                                                                             |                             |
| ------------- | ------------------------------------------------------------------------ | -------------------------------------------------------------- | --------------------------- | --------------------------------------------------------------------------- | --------------------------- |
| **source_id** | **filename**                                                             | **original_title**                                             | **original_version/status** | **sections_used**                                                           | **notes**                   |
| S1            | TÀI LIỆU ĐẶC TẢ HỆ THỐNG tooling_dfm_advisor (MASTER SPECIFICATION).docx | TOOLING_DFM_ADVISOR_MASTER_SPEC_v1                             | APPROVED                    | Governance-relevant content (see sections 3.x); full trace via Source tags. | Cross-WS governance extract |
| S2            | TÀI LIỆU ĐẶC TẢ HỆ THỐNG QUẢN TRỊ tooling_dfm_advisor SIPM SHIP.docx     | TOOLING_DFM_ADVISOR_GOVERNANCE_MASTER_SPEC_v1                  | APPROVED                    | Governance-relevant content (see sections 3.x); full trace via Source tags. | Primary                     |
| S3            | TOOLING_DFM_ADVISOR_MASTER_SPEC_ADDENDUM_v1.docx                         | TOOLING_DFM_ADVISOR MASTER SPECIFICATION – ADDENDUM V1         | [CHƯA XÁC MINH]             | Governance-relevant content (see sections 3.x); full trace via Source tags. | Cross-WS governance extract |
| S4            | TOOLING_DFM_ADVISOR_GOVERNANCE_MASTER_SPEC_ADDENDUM_v1.docx              | TOOLING_DFM_ADVISOR GOVERNANCE (SIPM/SHIP-FIRST) – ADDENDUM V1 | [CHƯA XÁC MINH]             | Governance-relevant content (see sections 3.x); full trace via Source tags. | Primary                     |

## 3. Consolidated Specification Content

## 3.1 ADN (Algorithm / Logic)

This section preserves governance logic, operational loop behavior, and non-bypassable decision rules.

### 3.1.1 Governance operational logic (verbatim extracts)

*Source: S2 - TOOLING_DFM_ADVISOR_GOVERNANCE_MASTER_SPEC_v1*

TÀI LIỆU ĐẶC TẢ HỆ THỐNG QUẢN TRỊ tooling_dfm_advisor (SIPM / SHIP-FIRST GOVERNANCE MASTER SPECIFICATION)

Tài liệu này là bản xuất bản đầy đủ, đã chuẩn hóa theo cùng phong cách của bản MASTER SPEC tooling_dfm_advisor trước đó, và đồng thời tuân thủ bộ tiêu chuẩn Naming/Directory/Governance Invariants. Mục tiêu: vận hành được trong thực tế, không deadlock/loop, tamper-evident, fail-safe, và có khả năng brownfield onboarding.

THÔNG TIN PHIÊN BẢN (DOCUMENT METADATA)

spec_id: TOOLING_DFM_ADVISOR_GOVERNANCE_MASTER_SPEC_v1

status: APPROVED

effective_date_utc: [CHƯA XÁC MINH] (điền khi phát hành)

owner: PM / System Owner

tool_name: govctl

policy_version_default: GOV_POLICY_v1

TẦM NHÌN VÀ MỤC TIÊU (VISION & OBJECTIVES)

1.1. Mục tiêu cốt lõi
Hệ thống cung cấp quy trình Governance-as-Code có thể vận hành trơn tru trong thực tế (local/dev/CI), với các mục tiêu:

Tránh deadlock hoặc vòng lặp vô hạn trong điều phối.

Tuyệt đối không bypass các rào chắn chất lượng (non-bypassable quality).

Không làm suy giảm velocity một cách không kiểm soát (trade-off phải được đo và audit).

Fail-safe coordination: không ghi đè trạng thái đồng thời; pause an toàn.

Policy-as-config: quy tắc ở config; tool chỉ thi hành logic.

1.3. Chỉ số đo lường hiệu quả (KPIs)
Hệ thống đạt chuẩn khi:

PAUSE giữa tác vụ dài không làm hỏng state; không tạo concurrent overwrite.

QUY ƯỚC ĐẶT TÊN (NAMING CONVENTIONS) – ZERO TOLERANCE

2.1. Định danh hệ thống, tool và artefacts

Tên hệ thống: tooling_dfm_advisor (snake_case, không khoảng trắng).

Tool điều phối: govctl (snake_case).

CẤU TRÚC THƯ MỤC BẮT BUỘC (MANDATORY DIRECTORY LAYOUT)

A) Commit required (trong repo)

GENESIS_KIT/ (bắt buộc tồn tại; xem Mục 5.3)

SESSION_AUDIT_LOG.md

LAST_SESSION_CHECKPOINT.md (khuyến nghị; append-only resume anchor)

logs/\*.stdout.log, \*.stderr.log

Quy tắc append-only:

NGUYÊN TẮC VẬN HÀNH BẤT BIẾN (INVARIANT PRINCIPLES)

Hệ thống dựa trên 8 hard decisions (non-bypassable):

4.2. State concurrency (lock + pause-request)

Mọi ghi state phải có lock hợp lệ; PAUSE dùng cờ PAUSE_REQUESTED và cooperative abort.

4.3. Governance update (policy-as-config, versioned)

Policy change phải audit: POLICY_UPDATED (ai đổi, đổi gì, lý do).

GENESIS_KIT tối thiểu

4.5. Unknown step hard-stop
Nếu current_step hoặc next_allowed_step không phân giải được từ:

FAIL-FAST

set current_step = PAUSE_FOR_AUTHORITY_DECISION

current_step = PAUSE_FOR_AUTHORITY_DECISION

audit event: DECISION_UNMAPPED.

4.7. Anti-loop strategy (retry bounded + progress guard)

Vượt max_retry_per_step:

chuyển PAUSE_FOR_AUTHORITY_DECISION

không được “đẻ step ad-hoc” trong state.

hoặc thiếu bất kỳ core step nào của GENESIS.
Executor bị cấm chạy khi GENESIS không đầy đủ.

KIẾN TRÚC TÀI LIỆU VÀ MÔ HÌNH TRẠNG THÁI (ARTEFACTS & STATE MODEL)

5.1. Minimum governance kernel (bắt buộc)
Commit required:

lock_metadata (nếu cần, nhưng lock file là nguồn chính)

TUYỆT ĐỐI KHÔNG lưu:

semantic claims (“đã xong feature X”)

trường điều khiển tự do (NEXT_ACTION, next_action, decision.next_action…)
Nếu phát hiện các trường cấm → govctl validate-state FAIL-FAST.

CƠ CHẾ VẬN HÀNH CỐT LÕI (CORE MECHANISMS)

sort keys theo thứ tự alphabet

không có whitespace thừa

rebuild canonical payload

validate hashes của stdout/stderr (nếu có)
Nếu mismatch → status = TAMPERED, hard-fail.

6.2. Lock & pause-request (state concurrency)
Advisory lock:

Pause-request:

tác vụ dài phải chạy trong wrapper kiểm tra PAUSE_REQUESTED theo chu kỳ 5–10s

policy_version (ví dụ GOV_POLICY_v1)

max_retry_per_step

genesis_core_steps (list)

ci_quality_rules (EMPTY_TEST_SUITE, PASS_SCAFFOLD time-bomb…)
Policy update là “safe lane” nhưng bắt buộc:

validate toàn diện

audit event POLICY_UPDATED (actor, diff, justification)

6.4. Brownfield integration
3 bước bắt buộc:

init GENESIS_KIT tối thiểu (xem 7.3)

baseline capture:

QUY TẮC THỰC THI & KIỂM SOÁT CHẤT LƯỢNG (QUALITY GATES)

7.1. Non-bypassable quality

EMPTY_TEST_SUITE:

Time-bomb scaffold:

cho phép PASS_SCAFFOLD tồn tại trong số sprint giới hạn (policy)

hết hạn: mọi CODE_CHANGE → FAIL cho đến khi scaffold được thay bằng implementation thật

7.2. Lanes & anti-loop
Lane selection:

govctl lane-check quyết định Fast/Safe dựa trên config + git diff (không để AI tự chọn).
Safe lane bắt buộc có ADR (nếu dự án áp dụng ADR).
Anti-loop:

vượt max_retry_per_step → PAUSE_FOR_AUTHORITY_DECISION.

GOV_CAPTURE_COMMAND_EVIDENCE

RUN_CI_SKELETON

PAUSE_FOR_AUTHORITY_DECISION

AUTHORITY_SELECT_NEXT_STEP

REVERT_LAST_CHANGE_GIT (bắt buộc nếu dự án dùng Git)

VÒNG LẶP VẬN HÀNH TIÊU CHUẨN (STANDARD OPERATIONAL LOOP)

Chu trình vận hành chuẩn gồm 9 bước, phải theo thứ tự:

resolve chain: CANONICAL → LKG → GENESIS

hard-fail nếu thiếu GENESIS core

govctl validate-state

resolve current_step/next_allowed_step

govctl sanity-check

git head/dirty

repo_fingerprint

govctl lane-check

xác định Fast/Safe lane theo config + diff

AUTHORITY_SELECT_NEXT_STEP

chọn đúng 1 next_allowed_step hợp lệ

executor run (wrapper protected)

chạy step được chọn

tôn trọng PAUSE_REQUESTED (cooperative abort)

verify sha256/hmac + attached logs hashes + exit code

append audit log

update state

nếu pause requested: tuân thủ pause flow, không overwrite khi lock chưa release

ĐIỀU KIỆN “GO FOR IMPLEMENTATION” (ENTRY CRITERIA)

Tài liệu đủ điều kiện chuyển sang implement govctl + wrappers khi:

Unknown step hard-stop + decision_map closed-set được mô tả và có audit events.

PAUSE_REQUESTED + cooperative abort được mô tả và không cho phép concurrent overwrite.

CI non-bypassable gates: EMPTY_TEST_SUITE + scaffold time-bomb.

PHẦN [CHƯA XÁC MINH] CẦN CHỐT TRONG POLICY-AS-CONFIG (KHÔNG CẢN TRỞ IMPLEMENT NẾU ĐÃ CÓ FAIL-SAFE)

### 3.1.2 Normative governance deltas (DFM-first, non-bypassable)

*Source: S4 - Governance addendum deltas*

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

This section consolidates governance artefacts, contracts/schemas, directory layout, and receipt/state interfaces.

### 3.2.1 Governance contracts, schemas, artefacts (verbatim extracts)

*Source: S2 - Contracts & templates*

tool_api: GOVCTL_API_v1

state_schema: SESSION_STATE_SCHEMA_v1

receipt_schema: RECEIPT_SCHEMA_v1

registry_schema: STEP_REGISTRY_SCHEMA_v1

genesis_contract: GENESIS_KIT_CONTRACT_v1

Tối ưu sử dụng context (policy + repo fingerprint + receipts) để ra quyết định có thể kiểm chứng.

Tamper-evident: mọi sửa tay đều bị phát hiện (receipt integrity).

Brownfield onboarding: áp dụng thực dụng lên dự án hiện hữu bằng GENESIS + template steps + baseline capture.

Unbrick Registry/State và khôi phục hoạt động < 15 phút bằng LKG + GENESIS.

govctl validate-receipt phát hiện ngay lập tức receipt bị sửa tay (HMAC mismatch) và hard-fail.

Policy updates có audit trace và không trộn lẫn vào SESSION_STATE.json.

Brownfield onboarding thành công qua template steps + baseline capture mà không phải viết lại toàn bộ hệ thống cũ.

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

config/

governance_policy.json (versioned; policy_version)

docs/runbooks/

step_registry.json

step_registry.LKG.json

docs/audit/ (append-only)

SESSION_STATE.json (coordination-only state)

SESSION_STATE.lock

docs/audit/receipts/

receipts/\*.json

GOV_RECEIPT_HMAC_KEY=<secret>

(khuyến nghị) config/policy_override.local.json

docs/audit/SESSION_AUDIT_LOG.md và LAST_SESSION_CHECKPOINT.md tuyệt đối chỉ append.

Nếu phát hiện chỉnh sửa tay các file append-only hoặc receipts: FAIL-FAST.

4.1. Receipt integrity (tamper-evident)
Mọi receipt tại local/CI phải có:

receipt_sha256 (sha256 của canonical payload)

receipt_sig_hmac_sha256 (hmac_sha256(secret, receipt_sha256))
Bất kỳ mismatch → TAMPERED + FAIL-FAST.

Cấm concurrent overwrite của SESSION_STATE.json.

Rules nằm trong config/governance_policy.json và có version.

SESSION_STATE.json chỉ dùng điều phối và counters; không chứa semantic claims.

4.4. Brownfield integration (genesis + templates + baseline)
Onboarding dự án cũ phải đi qua:

template steps

baseline capture (repo fingerprint + CI receipt đầu tiên)

step_registry.json (canonical), hoặc

step_registry.LKG.json, hoặc

GENESIS_KIT registry
thì bắt buộc:

set session_state = PAUSED

ghi audit event: UNKNOWN_STEP_ID (kèm field_name và step_id).

4.6. Decision codes only (closed-set)
Cấm trường điều khiển tự do (NEXT_ACTION, next_action, decision.next_action…).
Mọi rẽ nhánh dùng decision_code thuộc closed-set và map qua decision_map trong policy.
Nếu ánh xạ thất bại:

session_state = PAUSED

Retry hợp lệ chỉ khi receipt mới:

pass validate-receipt (HMAC ok)

và có tiến triển (NO_PROGRESS guard không kích hoạt; dựa trên git diff / receipt delta / step evidence).

decision_code = RETRY_EXHAUSTED

4.8. GENESIS invariants (non-bypassable)
govctl validate-registry phải hard-fail nếu:

thiếu docs/runbooks/GENESIS_KIT/

docs/audit/SESSION_AUDIT_LOG.md (append-only events)

docs/audit/SESSION_STATE.json (coordination-only)

docs/audit/SESSION_STATE.lock

docs/runbooks/step_registry.json và step_registry.LKG.json

docs/runbooks/GENESIS_KIT/

docs/audit/receipts/ (receipts + logs)

config/governance_policy.json (versioned)

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

decision_code (closed-set; không phải step_id)

6.1. Receipt integrity bằng HMAC (RECEIPT_SCHEMA_v1)
Receipt bắt buộc có:

receipt_sha256

receipt_sig_hmac_sha256

Canonical JSON payload (để tính receipt_sha256):

không bao gồm hai trường chữ ký (receipt_sha256, receipt_sig_hmac_sha256)

govctl validate-receipt:

parse JSON schema

recompute sha256, compare receipt_sha256

recompute hmac, compare receipt_sig_hmac_sha256

file: docs/audit/SESSION_STATE.lock

tối thiểu fields: pid (hoặc run_id), owner_step_id, start_utc, ttl_seconds
Quy tắc:

phải có lock trước khi ghi SESSION_STATE.json

file: docs/audit/PAUSE_REQUESTED (atomic create)

lệnh PAUSE chỉ tạo cờ; không ghi đè SESSION_STATE.json
Cooperative abort:

nếu phát hiện: dừng an toàn, tạo receipt ABORTED_BY_PAUSE, release lock, exit code đặc biệt

chỉ sau khi lock được release, controller mới set SESSION_STATE.json = PAUSED.

6.3. Policy-as-config (GOVERNANCE_POLICY_SCHEMA_v1)
config/governance_policy.json chứa:

decision_map (decision_code -> next_step_id)

template steps (theo mức thay đổi: DOC_ONLY_CHANGE, CODE_CHANGE_SMALL, CODE_CHANGE_LARGE…)

step_id: BROWNFIELD_BASELINE_CAPTURE

ghi git head hash, repo_fingerprint, tạo CI receipt đầu tiên, set trạng thái khởi điểm
Từ điểm baseline trở đi áp dụng governance loop mới.

nếu phát hiện số test = 0 → receipt status = FAIL, error_code = EMPTY_TEST_SUITE

retry chỉ hợp lệ nếu receipt mới pass validate-receipt và có progress (NO_PROGRESS guard không kích hoạt).

7.3. GENESIS core steps bắt buộc (closed list)
validate-registry hard-fail nếu thiếu bất kỳ step nào:

GOV_RECOVER_REGISTRY

GOV_VALIDATE_REGISTRY_AND_STATE

GOV_VALIDATE_RECEIPT

govctl validate-registry

parse + schema validate

detect forbidden fields (NEXT_ACTION…)

nếu unresolved step → PAUSE_FOR_AUTHORITY_DECISION + audit UNKNOWN_STEP_ID

validate last receipt signature

tạo receipt + sign HMAC

govctl validate-receipt

append vào docs/audit/SESSION_AUDIT_LOG.md

acquire lock → write SESSION_STATE.json safely → release lock

ĐỊNH DẠNG CẤU TRÚC DỮ LIỆU CHUẨN (TEMPLATES)

9.1. Receipt template (RECEIPT_SCHEMA_v1)
{
"schema_version": 1,
"tool_api": "GOVCTL_API_v1",
"tool_version": "govctl 1.0.0",
"run_id": "20260212T071350Z",
"kind": "ci_local",
"step_id": "RUN_CI_SKELETON",
"command_line": "pytest -q",
"working_dir": ".",
"start_utc": "2026-02-12T07:13:50Z",
"end_utc": "2026-02-12T07:14:40Z",
"exit_code": 0,
"status": "PASS",
"stdout_path": "docs/audit/receipts/logs/20260212T071350Z.stdout.log",
"stdout_sha256": "<sha256>",
"stderr_path": "docs/audit/receipts/logs/20260212T071350Z.stderr.log",
"stderr_sha256": "<sha256>",
"receipt_sha256": "\<sha256_of_canonical_payload>",
"receipt_sig_hmac_sha256": "\<hmac_sha256(key, receipt_sha256)>"
}

9.2. SESSION_STATE.json template (SESSION_STATE_SCHEMA_v1)
{
"schema_version": 1,
"session_state": "PAUSED",
"current_step": "PAUSE_FOR_AUTHORITY_DECISION",
"next_allowed_step": "AUTHORITY_SELECT_NEXT_STEP",
"decision_code": null,
"last_known_git": { "head": null, "dirty": null, "repo_fingerprint": null },
"last_ci_receipt": { "id": null, "status": null, "timestamp_utc": null },
"retry_counters": {}
}

9.3. SESSION_STATE.lock template
pid=12345
run_id=20260212T071350Z
owner_step_id=RUN_CI_SKELETON
start_utc=2026-02-12T07:13:50Z
ttl_seconds=600

Naming chuẩn hóa (tooling_dfm_advisor, govctl, run_id, step_id, decision_code).

Directory layout đúng (config/, docs/runbooks/ + GENESIS_KIT/, docs/audit/ append-only + lock + receipts).

validate-registry chain CANONICAL→LKG→GENESIS được mô tả và bắt buộc.

Receipt HMAC tamper-evident có canonicalization rõ ràng.

Bộ decision_code closed-set chính thức (danh sách đầy đủ) và mapping trong decision_map.

Quy tắc “NO_PROGRESS guard” đo progress chính xác theo tiêu chí nào (git diff threshold, receipt delta, file manifest).

### 3.2.2 System-level governance invariants referenced by WS_GOV

*Source: S1 - Governance invariants and pipeline-level audit constraints*

Tài liệu này là bản xuất bản đầy đủ, đã chuẩn hóa theo Bộ tiêu chuẩn Naming/Directory/Report/Governance Invariants mới. Mục tiêu là đảm bảo tài liệu đủ điều kiện chuyển tiếp sang giai đoạn lập trình mà không phát sinh “điểm gãy quy trình” khi triển khai CI/CD và orchestrator (importlib, step_registry, receipts, audit).

canonical_ui_contract: CANONICAL_UI_CONTRACT_v1

Có report artifact hỗ trợ audit (PDF/HTML). Mọi mở rộng vượt trần này phải được xem là một hệ thống khác.

QUY ƯỚC ĐẶT TÊN (NAMING CONVENTIONS) – ZERO TOLERANCE

CANONICAL_UI_CONTRACT_v1

REPORT_DELIVERY_RESULT_v0

2.4. Run ID (Traceability)
Mọi luồng thực thi phải có run_id định dạng ISO 8601 rút gọn: YYYYMMDDThhmmssZ
Ví dụ: 20260212T071350Z
run_id là định danh xuyên suốt receipts/defects/report.

CẤU TRÚC THƯ MỤC BẮT BUỘC (MANDATORY DIRECTORY LAYOUT)

Mã nguồn phải cô lập giữa Core Logic, Governance, và Audit/Receipts. Cấu trúc chuẩn:

governance_policy.json (Policy-as-Config; rule_version, thresholds, gates, toggles)

governance_allowlist.json (nếu cần; danh sách thư viện/feature được phép)

step_registry.json (đăng ký các step; orchestrator tra cứu)

docs/audit/ (append-only)

SESSION_AUDIT_LOG.md (nhật ký sự kiện)

SESSION_STATE.lock (khóa tương tranh)

LAST_SESSION_CHECKPOINT.md (mỏ neo resume; append-only)

docs/audit/receipts/

receipts/\*.json (biên lai từng bước)

reporter/ (reporter_agent, visualizer_agent nếu có)

orchestration/ (pipeline runner, receipt writer, state machine)

ui_contract/ (CANONICAL_UI_CONTRACT_v1 mappings)

Quy tắc: docs/audit là append-only. Không sửa/xóa bằng tay. Nếu phát hiện chỉnh tay: FAIL-FAST theo invariant tamper-evident.

NGUYÊN TẮC VẬN HÀNH BẤT BIẾN (INDUSTRIAL GOVERNANCE INVARIANTS)

4.3. SSOT tuyệt đối
TOOLING_MODEL_API_v1 là nguồn chân lý duy nhất. UI/Visualization/Narrative/Report là view/artifact, không có quyền mutate SSOT.

4.4. No Shadow Data
Không cho phép external geometry state, cache hình học ngoài kiểm toán, hoặc ghi dữ liệu không có receipt. Bất kỳ asset trung gian nào phải thuộc “ephemeral storage” và bị xóa sau delivery_complete.

4.5. Ephemeral Asset Storage (Non-bypassable)
Toàn bộ PNG/SVG/Mesh phát sinh trong phân tích:

Chỉ tồn tại trong vùng tạm (ephemeral workspace) gắn run_id.

Sau khi xóa: mọi tham chiếu trong Defect/Report phải được scrub (không còn đường dẫn file), chỉ giữ hash/manifest nếu cần truy vết.

4.6. Golden Geometry QA (Regression)
Bắt buộc duy trì Golden Geometry Corpus (STEP đã kiểm duyệt) + Expected Defect Set.
CI regression chạy theo corpus này.
CI Skeleton gate: nếu có thay đổi mã nguồn mà không có test mới đi kèm ở phạm vi liên quan → FAIL với mã lỗi: EMPTY_TEST_SUITE.

4.7. Canonical UI Contract (Non-bypassable)
UI không phải SSOT; UI không được healing, không sửa defect, không bật/tắt agent thủ công.
UI bắt buộc hiển thị 5 Global State:

report_validity
Ẩn bất kỳ state nào → vi phạm CANONICAL_UI_CONTRACT_v1.

4.8. Tamper-evident receipts (Non-bypassable)
Mọi lệnh/step thực thi phải sinh receipt JSON cục bộ kèm:

receipt_sha256

receipt_sig_hmac_sha256
Nếu phát hiện receipt bị chỉnh tay hoặc hash/signature mismatch → FAIL-FAST ngay.

Pipeline vẫn phải tạo report (tối thiểu Geometry Health Report).

5.1. SSOT: tooling_model (TOOLING_MODEL_API_v1)
tooling_model là container runtime duy nhất đại diện trạng thái hệ thống tại một thời điểm.
UI/Visualization/Narrative/Report không thuộc tooling_model và không có quyền ghi ngược vào tooling_model.
Không tồn tại agent-local state ghi ngược vào SSOT.

Quy tắc: SSOT chỉ chứa DGL dùng chung. Dẫn xuất đặc thù của agent phải là ephemeral và bị hủy sau khi agent hoàn tất.

Append-only.

assets_ephemeral_ref:

snapshot_png_ref (ephemeral token; không lưu path bền vững)

section_svg_ref (ephemeral token; không lưu path bền vững)

receipts_ref (receipt id/path trong receipts store)

REPORT_FORMAT_FORBIDDEN

RECEIPT_TAMPER_DETECTED

EMPTY_TEST_SUITE

Pipeline gồm 8 bước; bắt buộc chạy Bước 7 (Report) trong mọi chế độ (kể cả thất bại). Pipeline chỉ hoàn tất khi report được tạo và (nếu vận hành giao khách hàng) delivery_complete.

Bước 0 (bắt buộc, trước Bước 1): run_id allocation + receipt bootstrap

Tạo receipt “RUN_INIT_RESULT_v0” (hoặc gộp vào bước 1) và ghi SESSION_STATE.json anchor.

Output: CONTEXT_INCEPTION_RESULT_v0 (status OK hoặc FAILED). Luôn có receipt.

tolerance mặc định phải đến từ config/governance_policy.json theo rule_version.

tolerance engineering formula: [CHƯA XÁC MINH] (phải được định nghĩa trong governance_policy.json hoặc ADN tương ứng; nếu chưa có thì giới hạn sử dụng tolerance fixed + audit warning).

Chuyển Fallback Mode: dừng analysis agents, vẫn tạo report tối thiểu “Geometry Health Report”.

Ghi hash hình học, geometry_version_id, topology invariants (solid/face/edge count, bounding box, Euler).

ghi rõ nguyên nhân vào report.

Bước 7: report_composition_and_delivery (Reporter subsystem)

reporter_agent (read-only) tạo báo cáo ONLY PDF hoặc HTML.

Report metadata bắt buộc hiển thị:

Output: REPORT_DELIVERY_RESULT_v0 (status OK/FAILED; nếu format khác PDF/HTML → FAIL-FAST REPORT_FORMAT_FORBIDDEN).

Bước 8: verification_and_audit

Audit closure:

kiểm receipts đầy đủ

ghi SESSION_AUDIT_LOG.md append-only

Output: VERIFICATION_AUDIT_RESULT_v0.

XÓA SẠCH ephemeral assets (PNG/SVG/Mesh) theo run_id.

Ghi receipt “EPHEMERAL_PURGE_RESULT_v0” kèm bằng chứng xóa.

8.3. Reporter subsystem
reporter_agent là actor tạo report, không phải analysis_agent.
visualizer_agent (nếu tồn tại) chỉ phục vụ render PNG/SVG trong vùng ephemeral.

QUẢN TRỊ RỦI RO & BÁO CÁO (QUALITY GOVERNANCE & REPORTING)

Quy tắc ngưỡng chất lượng theo Material Class/Process Class/Geometry Fidelity phải nằm trong config/governance_policy.json và gắn rule_version.

report phải gắn cờ “Low Confidence”.

9.2. Golden geometry QA
Golden Geometry Corpus + Expected Defect Set dùng cho regression test CI, tách biệt khỏi pipeline vận hành thực tế.

9.3. Report delivery rules (non-bypassable)

Output format: ONLY PDF/HTML.

Sau delivery_complete: ephemeral purge + scrub refs + receipt.

CANONICAL UI CONTRACT (CANONICAL_UI_CONTRACT_v1)

UI là audit surface, không phải SSOT.
UI bắt buộc hiển thị 8 bước pipeline và 5 global states:

report_validity

degraded_mode: từ DGL regeneration limit / fallback decisions.

report_validity: từ REPORT_DELIVERY_RESULT_v0 + receipt verification.

RECEIPTS & AUDIT (TAMPER-EVIDENT EXECUTION)

11.1. Receipt generation
Mỗi step tạo 01 receipt JSON trong docs/audit/receipts/receipts/.
Receipt tối thiểu:

step_id (khớp step_registry.json)

receipt_sha256

receipt_sig_hmac_sha256

11.2. Append-only audit logs

docs/audit/SESSION_AUDIT_LOG.md: append-only.

docs/audit/LAST_SESSION_CHECKPOINT.md: append-only, phục vụ resume.

SESSION_STATE.lock + SESSION_STATE.json: quản lý tương tranh và anchor orchestrator.

11.3. Tamper detection
Bất kỳ mismatch sha256/hmac → FAIL-FAST RECEIPT_TAMPER_DETECTED.

STEP REGISTRY (docs/runbooks/step_registry.json)

REPORT_COMPOSE_DELIVER

VERIFICATION_AUDIT

EPHEMERAL_PURGE

Naming đã chuẩn hóa (snake_case + hậu tố; contracts/result envelopes versioned).

Directory layout tồn tại đúng chuẩn.

step_registry.json có đủ step_id cần thiết.

receipts tamper-evident được mô tả và được triển khai theo spec.

report delivery chỉ PDF/HTML + layout 6 phần + metadata đầy đủ.

ephemeral purge + scrub refs là non-bypassable.

CI gates có Golden Geometry regression và EMPTY_TEST_SUITE.

Công thức tolerance engineering theo Context (hiện spec chỉ mô tả nguyên tắc; cần định nghĩa trong governance_policy.json để tránh suy diễn).

### 3.2.3 System addendum deltas that impact governance

*Source: S3 - DFM-first deltas affecting governance enforcement*

Delta controls for DFM-first compliance, contracts, and report delivery invariants

Naming is zero-tolerance. Agents/providers are snake_case with suffix \_agent/\_provider; contracts are UPPER_CASE with \_vX; result envelopes are UPPER_CASE with algorithm suffix \_vN; run_id format is YYYYMMDDThhmmssZ (UTC).

Final report delivery is restricted to PDF or HTML only. Any other deliverable format is forbidden and MUST trigger FAIL-FAST with error_code REPORT_FORMAT_FORBIDDEN.

Report visuals are constrained: snapshots as PNG; cross-sections as SVG (vector). The report model MUST not embed persistent file paths.

Ephemeral assets (PNG/SVG/Mesh and any agent-specific derived geometry) MUST be purged immediately after delivery_complete. Defect/report references MUST be scrubbed (no file paths). Purge MUST produce a receipt EPHEMERAL_PURGE_RESULT_v1.

Golden Geometry QA is mandatory: regression tests MUST run on a curated STEP corpus with expected defect sets. Any code change without associated tests in the impacted scope MUST fail with EMPTY_TEST_SUITE (or equivalent CI rule).

Canonical UI Contract is mandatory: UI cannot mutate SSOT and MUST display the 5 global states: geometry_fidelity, degraded_mode, afr_state, analysis_completeness, report_validity.

- disable_reason (required when NOT_EXECUTED/STUB_MODE)

- receipts_ref (local receipt id; no external pointers)

Any asset references MUST be ephemeral tokens and MUST be scrubbed after delivery_complete.

Iterative healing is bounded (default 3 loops). Each loop MUST produce evidence (tolerance_delta, topology invariants, geometry_version_id change).

If geometry cannot be validated within bounds, the system MUST enter fallback mode: analysis agents may be skipped, but report generation MUST still complete with a Geometry Health Report.

5. Reporter subsystem boundary and deliverables

Reporter is split into:

- Report composition: mapping GeoModelResponse to a canonical report_model.

- Report delivery: rendering report_model to PDF/HTML and managing ephemeral assets.

Composition MUST output a versioned envelope (e.g., REPORT_COMPOSITION_RESULT_v1).

Delivery MUST output REPORT_DELIVERY_RESULT_v1 and MUST also emit EPHEMERAL_PURGE_RESULT_v1 after successful delivery_complete.

Naming audit passes (imports resolvable; module/file/class names match conventions).

Running pipeline in missing-backend mode returns NOT_EXECUTED/STUB_MODE envelopes and still produces a minimal report artifact (PDF/HTML).

REPORT_FORMAT_FORBIDDEN triggers FAIL-FAST when a non-PDF/HTML deliverable is requested.

After delivery_complete, ephemeral workspace is empty; purge receipt EPHEMERAL_PURGE_RESULT_v1 exists and validates.

Golden Geometry regression suite runs; changes to affected modules include tests; EMPTY_TEST_SUITE gate remains non-bypassable.

UI surfaces the 5 global states and does not mutate SSOT.

## 3.3 LDR (Dependencies / Libraries / Runtime constraints)

This section consolidates runtime prerequisites for governance execution (secrets, local-only files, TTL/locks) and delivery-related gates.

### 3.3.1 Runtime prerequisites (verbatim extracts)

*Source: S2 - Local-only secrets and runtime constraints*

1.2. Tuyên bố thực tế (Reality statement)
Local environment không thể ngăn tuyệt đối can thiệp thủ công; do đó hệ thống tập trung vào:

B) Local-only (không commit)

.secrets.env hoặc environment variables:

dùng cho override tạm thời; phải nằm trong .gitignore.

Local-only:

lock còn TTL → STATE_LOCKED

quá TTL → cho phép steal lock nhưng bắt buộc audit event: LOCK_STOLEN

lock_ttl_seconds

Chính sách TTL cụ thể theo môi trường (local vs CI) và điều kiện steal lock.

### 3.3.2 Delivery/purge/QA gates referenced by governance

*Source: S3 - Report delivery and purge invariants (cross-WS)*

Delta controls for DFM-first compliance, contracts, and report delivery invariants

Final report delivery is restricted to PDF or HTML only. Any other deliverable format is forbidden and MUST trigger FAIL-FAST with error_code REPORT_FORMAT_FORBIDDEN.

Report visuals are constrained: snapshots as PNG; cross-sections as SVG (vector). The report model MUST not embed persistent file paths.

Ephemeral assets (PNG/SVG/Mesh and any agent-specific derived geometry) MUST be purged immediately after delivery_complete. Defect/report references MUST be scrubbed (no file paths). Purge MUST produce a receipt EPHEMERAL_PURGE_RESULT_v1.

Golden Geometry QA is mandatory: regression tests MUST run on a curated STEP corpus with expected defect sets. Any code change without associated tests in the impacted scope MUST fail with EMPTY_TEST_SUITE (or equivalent CI rule).

Canonical UI Contract is mandatory: UI cannot mutate SSOT and MUST display the 5 global states: geometry_fidelity, degraded_mode, afr_state, analysis_completeness, report_validity.

Any asset references MUST be ephemeral tokens and MUST be scrubbed after delivery_complete.

If geometry cannot be validated within bounds, the system MUST enter fallback mode: analysis agents may be skipped, but report generation MUST still complete with a Geometry Health Report.

5. Reporter subsystem boundary and deliverables

Reporter is split into:

- Report composition: mapping GeoModelResponse to a canonical report_model.

- Report delivery: rendering report_model to PDF/HTML and managing ephemeral assets.

Composition MUST output a versioned envelope (e.g., REPORT_COMPOSITION_RESULT_v1).

Delivery MUST output REPORT_DELIVERY_RESULT_v1 and MUST also emit EPHEMERAL_PURGE_RESULT_v1 after successful delivery_complete.

Running pipeline in missing-backend mode returns NOT_EXECUTED/STUB_MODE envelopes and still produces a minimal report artifact (PDF/HTML).

REPORT_FORMAT_FORBIDDEN triggers FAIL-FAST when a non-PDF/HTML deliverable is requested.

After delivery_complete, ephemeral workspace is empty; purge receipt EPHEMERAL_PURGE_RESULT_v1 exists and validates.

Golden Geometry regression suite runs; changes to affected modules include tests; EMPTY_TEST_SUITE gate remains non-bypassable.

*Source: S4 - Governance decision codes and policy keys*

Operational deltas for policy-as-config, decision codes, and non-bypassable gates

This addendum defines additional governance requirements that must be enforced by govctl, policy-as-config, and CI gates.

It focuses on preventing process breaks (unknown steps, unmapped decisions, unverified receipts) and on enforcing new industrial invariants (report format, ephemeral purge, golden corpus).

Governance policy MUST explicitly encode decision_map entries for report delivery and ephemeral purge outcomes (including failure paths).

Receipts MUST carry policy_version and rule_version, and MUST be tamper-evident with receipt_sha256 + receipt_sig_hmac_sha256.

CI MUST enforce EMPTY_TEST_SUITE and GOLDEN_CORPUS_MISSING as non-bypassable quality rules.

3. Decision codes and step ids (closed-set)

1. Receipt schema deltas (RECEIPT_SCHEMA_v1 compatible)

Receipts remain compatible with RECEIPT_SCHEMA_v1, but MUST include these additional fields (either as top-level or within a stable metadata object):

- policy_version

- artifact_manifest_sha256 (when producing report artifacts)

Any missing required field MUST cause validate-receipt to fail for steps that claim delivery_complete.

5. Policy-as-config keys to add (minimum)

- Compute hashes of approved specification artefacts (base specs + addenda).

- If repo changes touch contracts/, agents/, providers/, reporter/, or orchestration/ AND spec-sync hashes are not updated, emit decision_code SPEC_DRIFT and pause.

validate-state rejects forbidden free-form control fields; decision_code must be closed-set.

validate-receipt detects tampering (sha256/hmac mismatch) and fails fast.

CI gates enforce EMPTY_TEST_SUITE and GOLDEN_CORPUS_MISSING.

v1 (REVIEW): initial governance addendum enforcing delivery/purge and spec-sync gates.

## 4. Open Questions / [CHƯA XÁC MINH]

*Source: S1*

effective_date_utc: [CHƯA XÁC MINH] (điền khi phát hành)

Impact: Without effective_date_utc, audit trails cannot unambiguously establish when the spec/addendum becomes authoritative.

*Source: S1*

tolerance engineering formula: [CHƯA XÁC MINH] (phải được định nghĩa trong governance_policy.json hoặc ADN tương ứng; nếu chưa có thì giới hạn sử dụng tolerance fixed + audit warning).

Impact: Without an explicit tolerance engineering formula in policy-as-config, validation/healing parameters may drift or be inconsistently applied across environments.

*Source: S1*

PHẦN [CHƯA XÁC MINH] CẦN CHỐT TRONG POLICY-AS-CONFIG (KHÔNG CẢN TRỞ CODING NẾU ĐÃ CÓ FAIL-SAFE)

Impact: [CHƯA XÁC MINH] Not enough detail to quantify impact; confirm required fields/keys in governance_policy.json.

*Source: S2*

effective_date_utc: [CHƯA XÁC MINH] (điền khi phát hành)

Impact: Without effective_date_utc, audit trails cannot unambiguously establish when the spec/addendum becomes authoritative.

*Source: S2*

PHẦN [CHƯA XÁC MINH] CẦN CHỐT TRONG POLICY-AS-CONFIG (KHÔNG CẢN TRỞ IMPLEMENT NẾU ĐÃ CÓ FAIL-SAFE)

Impact: [CHƯA XÁC MINH] Not enough detail to quantify impact; confirm required fields/keys in governance_policy.json.

## 5. Change Log (Editorial)

v1 REVIEW: Document governance consolidation. Content is preserved from sources and re-packaged into the WS canonical SPC template. No semantic changes; only re-ordering, headings normalization, and insertion of traceability tags (Source: Sx).

Terminology normalization (non-semantic): When source documents use equivalent labels (e.g., 'SIPM/SHIP-first governance' vs 'govctl governance'), the original wording is retained in the extract blocks; any 'aka' mapping should be added only if future sources introduce naming variants.

## 6. Appendix

Appendix is reserved for glossary and cross-references. Cross-WS canonical references should be recorded here as they are created.
