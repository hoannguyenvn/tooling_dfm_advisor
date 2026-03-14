**WS_GOV-ORC-SPC-v1 — orchestrator**

*Status: REVIEW | Run ID (UTC): 20260228T083954Z*

# 0. Document Control Block

|                                  |                                                                      |
| -------------------------------- | -------------------------------------------------------------------- |
| Canonical Code                   | WS_GOV-ORC-SPC-v1                                                    |
| Status                           | REVIEW                                                               |
| Run ID                           | 20260228T083954Z                                                     |
| WS / Module / Doc Type / Version | WS_GOV / ORC / SPC / v1                                              |
| Created date (UTC)               | 20260228T083954Z                                                     |
| Purpose                          | Document governance consolidation only; no runtime/pipeline changes. |

## 1. Scope

This document is the canonical SPC (SSOT) for the orchestrator module (WS_GOV / ORC). It consolidates algorithm/logic (ADN), bindings/contracts (AMBR), and runtime/dependency constraints (LDR) from the provided sources, preserving technical content and traceability.

Non-goals: no redesign of architecture or pipeline; no new rules beyond the sources; no code changes.

## 2. Included Sources & Mapping Table

|           |                                                             |                                                                   |                         |                                          |                                                                                |
| --------- | ----------------------------------------------------------- | ----------------------------------------------------------------- | ----------------------- | ---------------------------------------- | ------------------------------------------------------------------------------ |
| source_id | filename                                                    | original_title                                                    | original_version/status | sections_used                            | notes                                                                          |
| S1        | ADN_orchestrator_deterministic_pipeline_coordinator_v1.docx | ADN — orchestrator — Deterministic Pipeline Coordinator — v1      | REVIEW                  | 3.1 ADN (primary)                        | Algorithm/logic SSOT extract.                                                  |
| S2        | AMBR_orchestrator_v1.docx                                   | AMBR — orchestrator — Binding v1                                  | REVIEW                  | 3.2 AMBR (primary)                       | Bindings, contracts, security, test obligations.                               |
| S3        | LDR_importlib_v1.docx                                       | LDR — importlib (stdlib) — Allowlisted Dynamic Stage Loading — v1 | REVIEW                  | 3.3 LDR (primary)                        | Dependency decision for allowlisted dynamic loading.                           |
| S4        | AMBR_contract_index_provider_v1.docx                        | AMBR — contract_index_provider — Binding v1                       | REVIEW                  | 3.2 AMBR (dependency extract) + Appendix | Used only to capture dependency constraints; full module SSOT is out-of-scope. |
| S5        | LDR_contract_index_provider_stdlib_v1.docx                  | LDR — contract_index_provider — Language/Stdlib Dependencies — v1 | REVIEW                  | 3.3 LDR (dependency extract) + Appendix  | Used only to capture dependency constraints; full module SSOT is out-of-scope. |

## 3. Consolidated Specification Content

## 3.1 ADN (Algorithm / Logic)

Source: S1

ADN — orchestrator — Deterministic Pipeline Coordinator + Allowlist + Receipts — v1

Status: REVIEW (V1 candidate). DFM-first: enforce deterministic stage plan, governance-by-contract, fail-safe aggregation, and non-bypassable allowlists.

**1. Identification**

adn_id: ADN_orchestrator_deterministic_pipeline_coordinator_v1

module_name: orchestrator

module_kind: orchestrator_module (coordinator)

algorithm_name: deterministic_pipeline_coordinator

algorithm_version: v1

output_envelope: ORCHESTRATION_RESULT_v0

owner: [CHƯA XÁC MINH]

authority_refs: Master Spec + Governance Spec + Addendum V1

**2. Why**

orchestrator điều phối pipeline theo thứ tự tất định để biến một đầu vào STEP thành SSOT geometry versioned + tập kết quả DFM + báo cáo cuối. Mục tiêu cốt lõi: (a) deterministic stage ordering, (b) handoff hình học nhất quán (active_geometry/SSOT) giữa các stage, (c) không crash toàn bộ pipeline khi stage optional fail, và (d) bảo vệ khỏi module injection bằng allowlist/registry.

**3. Preconditions**

run_id bắt buộc theo định dạng YYYYMMDDThhmmssZ; orchestrator MUST NOT tự tạo run_id mới.

Policy snapshot phải cung cấp rule_version (string) để đóng dấu vào mọi envelopes/receipts.

Dynamic import chỉ được phép từ stage_allowlist (từ contract_index_provider/step_registry); tuyệt đối cấm import tên module tùy ý.

Không được ghi/giữ lại asset bền vững (PNG/SVG/Mesh). Ephemeral assets chỉ được phép tồn tại cho đến delivery_complete, sau đó purge.

Fail-safe cho deps nặng: thiếu pythonocc-core không được làm chết pipeline; stage phải trả NOT_EXECUTED hoặc degraded envelopes.

**4. Inputs (DFM-first preferred)**

**4.1 OrchestrationRequestV1 (recommended contract; may be embedded in governance context):**

run_id: str

rule_version: str

step_import_request: StepImportRequestV1 (STEP_IMPORT_API_v1)

context_snapshot: optional ContextSnapshotV1 (MODELS_API_v1)

requested_stages: optional list[str] (subset of allowlist; if absent use canonical stage plan)

stage_payloads: optional dict[str, object] (typed per stage; no raw dict in production)

**4.2 Legacy input (allowed only during migration):**

payload: dict with per-stage subpayloads; MUST be normalized to typed contracts before stage execution. [CHƯA XÁC MINH current payload keys].

**5. Canonical stage plan (deterministic)**

Required stages (minimum) and order:

stage_1: step_import_provider (STEP bytes → import envelope + AGL init)

stage_2: validators (geometry_validation_gate)

stage_3: healer_agent (optional heal-loop; bounded retries)

stage_4: mapping_provider (AGL→DGL mapping)

stage_5: afr_provider (proxy AFR)

stage_6: DFM agents (draft/thickness/undercut/ribratio/sink/weldline …)

stage_7: reporter (composition)

stage_8: delivery (PDF/HTML only + purge ephemeral assets)

Optional stages MUST be explicitly listed and may be skipped only with an explicit SKIPPED state in their envelopes.

**6. Actions / Algorithm (step-by-step)**

Initialize ORCHESTRATION_RESULT_v0 with status=INITIALIZED; set run_id, rule_version, timestamp_utc.

Resolve stage_allowlist from contract_index_provider + step_registry (source: governance). If stage not allowed → FAIL-FAST (STAGE_NOT_ALLOWED).

Normalize input payloads to typed contracts (STEP_IMPORT_API_v1, GEO_MODEL_API_v1, MODELS_API_v1). If normalization fails → FAIL-FAST (INPUT_CONTRACT_INVALID).

For each stage in canonical plan (or requested subset):

a) Dynamic import stage module using allowlisted module path (importlib).

b) Resolve entrypoint function name (fixed per stage interface). If missing → record error ENTRYPOINT_NOT_FOUND; mark stage FAILED; continue if stage is optional else mark orchestration as PARTIAL/ERROR depending on policy.

c) Execute stage with typed inputs. Catch all exceptions and convert to stage error STAGE_EXECUTION_FAILED (retryable=false) while preserving pipeline stability.

d) Update active_geometry / tooling_model SSOT only at defined handoff points (after import/validation/heal/freeze). No other stage may mutate SSOT.

e) Inject mapping_result/afr_result into downstream DFM agents inputs (data-only injection).

Aggregate per-stage envelopes, errors, warnings. Compute final_status: COMPLETE (all required stages OK), PARTIAL (some optional failed or degraded), ERROR (required stage failed or fail-fast triggered).

If final_status == COMPLETE and delivery ran: ensure EPHEMERAL_PURGE_RESULT\_\* indicates PURGED before setting delivery_complete=true.

Return ORCHESTRATION_RESULT_v0.

**7. Output envelope (ORCHESTRATION_RESULT_v0 — V1 minimum schema)**

status: INITIALIZED | COMPLETE | PARTIAL | ERROR

run_id: str

rule_version: str

geometry_version_id_final: optional[str] (required if COMPLETE/PARTIAL with frozen geometry)

timestamp_utc: str

stage_results: dict[str, object] (each MUST be a versioned result envelope; no bare dict)

errors: list[{code:str, message:str, stage?:str, retryable:bool}]

warnings: list[{code:str, message:str, stage?:str}]

global_state: optional GlobalStateV1 (from GEO_MODEL_API_v1)

report_artifact: optional ReportArtifactV1 (MODELS_API_v1)

degraded_mode: bool (true if any required stage NOT_EXECUTED/FAILED or any integrity check SKIPPED/FAILED)

**8. Failure modes (structured)**

STAGE_NOT_ALLOWED → FAIL-FAST (security)

INPUT_CONTRACT_INVALID → FAIL-FAST

MODULE_IMPORT_FAILED → stage FAILED; required stage may cause ERROR

ENTRYPOINT_NOT_FOUND → stage FAILED; required stage may cause ERROR

STAGE_EXECUTION_FAILED → stage FAILED; required stage may cause ERROR

TRACE_ID_TODO → forbidden in v1; trace uses run_id; optional trace_id may be derived but must be specified in governance [CHƯA XÁC MINH].

STEP_CHECKSUM_NOT_VERIFIED → orchestration must be PARTIAL/ERROR depending on policy; recommended FAIL-FAST for tamper-evident breach.

**9. Evidence / Verification**

Unit tests: stage allowlist enforcement (reject unknown stage).

Unit tests: deterministic stage ordering and aggregation (same inputs → same sequence).

Unit tests: module import failure aggregated without crash for optional stages.

Integration: missing pythonocc-core produces NOT_EXECUTED envelopes and orchestration PARTIAL (not crash).

Golden Geometry QA: regression only when orchestrator changes handoff semantics (freeze/mapping injection).

Receipt checks: every local run produces receipt with receipt_sha256 + receipt_sig_hmac_sha256; ORCHESTRATION_RESULT_v0 references run_id/rule_version.

**10. Fail-fast rules**

Any stage name outside allowlist → FAIL-FAST.

Any attempt to deliver report in non-PDF/HTML format → FAIL-FAST (REPORT_FORMAT_FORBIDDEN).

delivery_complete cannot be set unless ephemeral purge succeeded (DELIVERY_COMPLETE_WITHOUT_PURGE).

No silent success: stage_results entries must be versioned envelopes with explicit status.

**11. Troubleshooting**

MODULE_IMPORT_FAILED: verify stage module path matches \_provider/\_agent naming and exists under tooling_dfm_advisor/…

ENTRYPOINT_NOT_FOUND: verify stage implements expected interface function; align with report_interfaces/agent interfaces.

Unexpected PARTIAL: check which stage returned NOT_EXECUTED or integrity_check_state SKIPPED/FAILED; inspect stage error codes.

Security failures: stage not allowed → update contract_index_provider/step_registry; do not bypass with raw import strings.

**12. Non-goals**

Không thực hiện tính toán hình học/DFM trong orchestrator.

Không tự chữa hình học ngoài việc gọi healer_agent theo protocol.

Không ghi file/asset; delivery subsystem owns rendering + purge.

## 3.2 AMBR (Bindings / Module Contract / Execution)

Source: S2

AMBR — orchestrator — Binding v1

Status: REVIEW (V1 candidate).

**1. Module identification**

ambr_id: AMBR_orchestrator_v1

module_name: orchestrator

module_kind: orchestrator_module

binding_version: v1

repo_path: tooling_dfm_advisor/orchestrator/orchestrator.py [CHƯA XÁC MINH actual repo path]

execution_mode: SEQUENTIAL

ssot_permissions: may_mutate_tooling_model=true (handoff points only); must_not_mutate_after_freeze=true

owner: [CHƯA XÁC MINH]

**2. Bound algorithm**

adn_id: ADN_orchestrator_deterministic_pipeline_coordinator_v1

algorithm_name: deterministic_pipeline_coordinator

algorithm_version: v1

output_envelope: ORCHESTRATION_RESULT_v0

**3. Bound contracts (required)**

STEP_IMPORT_API_v1 (StepImportRequestV1)

GEO_MODEL_API_v1 (GeoModelResponse / GlobalStateV1)

MODELS_API_v1 (ContextSnapshotV1, ReportArtifactV1)

\_contract_index (contract_index_provider) for dynamic resolution

**4. Dependencies (LDR)**

LDR_importlib_v1 (stdlib) — dynamic loading under allowlist only

LDR_typing_v1 (optional; stdlib) — type hints only

**5. Security and allowlist**

Stage names and module paths MUST be resolved from allowlist/registry; never from user-controlled strings.

Orchestrator MUST refuse to import stages not present in registry (STAGE_NOT_ALLOWED).

**6. Determinism & side effects**

Deterministic ordering: canonical stage plan; requested_stages must preserve relative order constraints.

Side effects limited to SSOT handoff points and appending audit receipts; no file/asset persistence.

**7. Change impact rules**

Stage order changes are breaking to process semantics: require ADN update + targeted tests + golden regression review.

Changing injection behavior (mapping/afr into DFM agents) requires golden expected defect updates.

Any new stage requires updating contract_index_provider and registry + tests (completeness).

**8. Test obligations (minimum)**

Orchestrator sequencing test: required stages executed in canonical order.

Allowlist test: unknown stage rejected fail-fast.

Aggregation test: optional stage failure does not crash pipeline; final_status becomes PARTIAL.

Receipt test: receipts carry run_id, rule_version; tamper-evident fields present.

**9. Audit & traceability**

ORCHESTRATION_RESULT_v0 MUST include run_id/rule_version; should surface geometry_version_id_final.

Stage results must be preserved verbatim (no mutation by orchestrator).

### 3.2.1 Dependency Extract — contract_index_provider (governance allowlist/registry)

Source: S4

AMBR — contract_index_provider — Binding v1

Status: REVIEW (V1 candidate)

**1. Module identification**

ambr_id: AMBR_contract_index_provider_v1

module_name: contract_index_provider

module_type: provider

binding_version: v1

repo_path: tooling_dfm_advisor/contracts/contract_index_provider.py

execution_mode: SEQUENTIAL (static lookup)

ssot_permissions: read_only=true; may_mutate_tooling_model=false

**2. Bound algorithm**

adn_id: ADN_contract_index_provider_contract_module_lookup_table_v1

algorithm_name: contract_module_lookup_table

algorithm_version: v1

**3. Dependencies**

No external libraries required (stdlib only).

Python stdlib only (dict, typing).

Use of 'from \_\_future\_\_ import annotations' is OPTIONAL; allowed under language-feature exemption policy.

**4. Determinism & side effects**

Deterministic by construction (static dict).

Forbidden: file IO, network IO, dynamic import execution.

Forbidden: logging sensitive paths or storing any external pointers.

**5. Change impact rules**

Adding a new contract requires: (1) update baseline catalog; (2) regenerate provider; (3) update expected set test; (4) bump binding_version if schema changes.

Any mismatch between expected set and index MUST fail CI (CONTRACT_INDEX_STALE).

**6. Audit & traceability**

If regenerated by a step: step produces receipt JSON with receipt_sha256 + receipt_sig_hmac_sha256; include run_id.

Provenance field MUST reference baseline catalog identifier (not a filesystem path).

## 3.3 LDR (Dependencies / Libraries / Runtime constraints)

Source: S3

LDR — importlib (stdlib) — Allowlisted Dynamic Stage Loading — v1

Status: REVIEW (V1 candidate).

**1. Identification**

ldr_id: LDR_importlib_v1

library_name: importlib

library_version: stdlib (python) [CHƯA XÁC MINH version pin]

decision_version: v1

used_by_modules: orchestrator

**2. Purpose**

Dùng importlib.import_module để nạp stage modules một cách loose-coupling, nhưng chỉ dưới allowlist/registry để ngăn module injection.

**3. Allowed usage**

importlib.import_module(allowed_module_path)

getattr(module, entrypoint_name) where entrypoint_name is a fixed constant per stage interface

**4. Forbidden usage**

Import module names derived from user input or untrusted strings.

Dynamic execution of modules not registered in stage_allowlist/step_registry.

Loading modules outside tooling_dfm_advisor package namespace.

**5. Risks & mitigations**

Security: module injection → mitigate via allowlist enforced by orchestrator (STAGE_NOT_ALLOWED).

Observability drift if missing run_id/rule_version → mitigate by strict contract requirements.

Python runtime drift → mitigate by pinning python in CI and recording in receipts/manifests [CHƯA XÁC MINH pipeline].

**6. Verification checklist**

Unit test: orchestrator refuses to import non-allowlisted stage.

Unit test: module import errors are captured and aggregated.

### 3.3.1 Dependency LDR Extract — contract_index_provider (stdlib only)

Source: S5

LDR — contract_index_provider — Language/Stdlib Dependencies — v1

Status: REVIEW (V1 candidate)

**1. Decision record**

ldr_id: LDR_contract_index_provider_stdlib_v1

decision_version: v1

language_runtime: Python (version pin: [CHƯA XÁC MINH])

external_libraries: None

**2. Allowed usage**

Python stdlib: typing, importlib (callers), dict.

Optional: \_\_future\_\_.annotations (language feature) if exemption policy allows.

**3. Forbidden usage**

No third-party packages.

No filesystem persistence.

No network access.

No emitting absolute paths in logs/results.

**4. Risks**

Python version drift may affect typing/annotations behavior (mitigation: pin python version in CI/runtime).

Caller importlib resolution may fail if directory layout/naming not compliant.

**5. Verification checklist**

CI: verify python version pin (runtime report).

CI: contract index completeness test passes.

CI: dry import of each mapped module_name passes.

## 4. Open Questions / [CHƯA XÁC MINH]

Items below are present in sources or required by the template but not verified with sufficient evidence.

|        |                                                                                                                                           |                                                         |                                                    |
| ------ | ----------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------- | -------------------------------------------------- |
| source | item                                                                                                                                      | question / missing data                                 | impact if missing                                  |
| S1     | owner: [CHƯA XÁC MINH]                                                                                                                    | Provide authoritative value / confirm in upstream spec. | Blocks full SSOT completeness and/or audit fields. |
| S1     | TRACE_ID_TODO → forbidden in v1; trace uses run_id; optional trace_id may be derived but must be specified in governance [CHƯA XÁC MINH]. | Provide authoritative value / confirm in upstream spec. | Blocks full SSOT completeness and/or audit fields. |
| S2     | owner: [CHƯA XÁC MINH]                                                                                                                    | Provide authoritative value / confirm in upstream spec. | Blocks full SSOT completeness and/or audit fields. |
| S5     | language_runtime: Python (version pin: [CHƯA XÁC MINH])                                                                                   | Provide authoritative value / confirm in upstream spec. | Blocks full SSOT completeness and/or audit fields. |
| N/A    | Status target assumed REVIEW because request omitted STATUS_TARGET.                                                                       | Confirm desired STATUS_TARGET for this pack.            | May require rename to the correct STATUS.          |

## 5. Change Log (Editorial)

20260228T083954Z: Consolidated provided ADN/AMBR/LDR documents into canonical WS_GOV-ORC-SPC-v1. No semantics changes; structure and headings normalized; traceability added via Source: Sx markers.

Terminology normalization: none beyond heading alignment; original terms preserved; any aliasing should be added in Appendix if needed.

## 6. Appendix (optional)

Cross references (canonical codes recommended):

- WS_GOV-ALL-POL-v1 (governance policy keys / thresholds / conventions).

- WS_MODEL-GEN-SPC-v1 (Contracts/APIs SSOT: STEP_IMPORT_API_v1, GEO_MODEL_API_v1, MODELS_API_v1).

- WS_GOV-GEN-SPC-v1 (contract_index_provider SSOT) [CHƯA XÁC MINH module 3LA assignment; GEN per minimal DGL list].
