WS_PROVIDERS-IMP-SPC-v1 — Consolidated Specification (SPC)

# 0. Document Control Block

|                                  |                                                                      |
| -------------------------------- | -------------------------------------------------------------------- |
| Canonical Code                   | WS_PROVIDERS-IMP-SPC-v1                                              |
| Status                           | REVIEW                                                               |
| Run ID (UTC)                     | 20260304T080555Z                                                     |
| WS / Module / Doc Type / Version | WS_PROVIDERS / IMP / SPC / v1                                        |
| Created date (UTC)               | 20260304T080555Z                                                     |
| Purpose                          | Document governance consolidation only; no runtime/pipeline changes. |

## 1. Scope

Source: S1 (ADN) + S2 (AMBR) + S3 (LDR).

2. Why

Module này cô lập tương tác với backend B-Rep (pythonocc) để pipeline DFM không bị sập vì thiếu/ lỗi dependency C++. Nó tạo payload AGL pre-freeze: shape handle (opaque) + trạng thái + lỗi có cấu trúc + basic meta để các bước validate/heal/freeze quyết định.

11. Non-goals

Không healing, không normalize units/axis, không freeze.

Không tạo defect, không AFR.

Không xuất report, không tạo PNG/SVG.

## 2. Included Sources & Mapping Table

|           |                                                                 |                                                                        |                         |               |                                              |
| --------- | --------------------------------------------------------------- | ---------------------------------------------------------------------- | ----------------------- | ------------- | -------------------------------------------- |
| source_id | filename                                                        | original_title                                                         | original_version/status | sections_used | notes                                        |
| S1        | ADN_brep_service_provider_step_to_shape_backend_wrapper_v1.docx | ADN — brep_service_provider — STEP→B-Rep Wrapper + Basic Meta — v1     | REVIEW (V1 candidate)   | 3.1 ADN       | Primary algorithm/logic (verbatim extracts)  |
| S2        | AMBR_brep_service_provider_v1.docx                              | AMBR — brep_service_provider — Binding v1                              | REVIEW (V1 candidate)   | 3.2 AMBR      | Bindings/execution/IO (verbatim extracts)    |
| S3        | LDR_pythonocc_backend_provider_adapter_for_brep_service_v1.docx | LDR — pythonocc_backend_provider adapter surface for brep_service — v1 | REVIEW (V1 candidate)   | 3.3 LDR       | Adapter/deps constraints (verbatim extracts) |

## 3. Consolidated Specification Content

## 3.1 ADN (Algorithm / Logic)

Source: S1

ADN — brep_service_provider — STEP→B-Rep Wrapper + Basic Meta — v1

Status: REVIEW (V1 candidate). DFM-first: fail-safe wrapper only; no healing/normalization/freeze.

1. Identification

adn_id: ADN_brep_service_provider_step_to_shape_backend_wrapper_v1

module_name: brep_service_provider

module_type: provider (geometry core)

algorithm_name: step_to_shape_backend_wrapper

algorithm_version: v1

result_envelope: BREP_LOAD_RESULT_v0

owner: [CHƯA XÁC MINH]

authority_refs: Master Spec + Governance Spec + Addendum V1

2. Why

Module này cô lập tương tác với backend B-Rep (pythonocc) để pipeline DFM không bị sập vì thiếu/ lỗi dependency C++. Nó tạo payload AGL pre-freeze: shape handle (opaque) + trạng thái + lỗi có cấu trúc + basic meta để các bước validate/heal/freeze quyết định.

3. Preconditions

Caller đã có run_id (YYYYMMDDThhmmssZ) và rule_version (policy snapshot).

Không được ghi file, không network IO, không tạo asset PNG/SVG/Mesh.

Thiếu pythonocc-core: MUST return NOT_EXECUTED/NOT_AVAILABLE (structured) — tuyệt đối không throw unhandled.

Không leak absolute filesystem paths vào payload/receipts/report/SSOT.

4. Inputs (DFM-first preferred)

Preferred typed input (governance-by-contract):

request: StepImportRequestV1 (from STEP_IMPORT_API_v1) — uses run_id, rule_version, sha256_hex, bytes_size, unit_hint, source_ref (opaque).

step_bytes: bytes (resolved by caller via ephemeral storage; NOT a path).

Legacy compatibility (allowed only for local/dev; forbidden in production receipts):

step_path: str (absolute/relative path) — MUST be scrubbed before any receipt/report serialization. [CHƯA XÁC MINH: whether legacy path exists in current code]

5. Actions / Algorithm (step-by-step)

Initialize BREP_LOAD_RESULT_v0 with status=NOT_EXECUTED (default), degraded_mode=true; attach run_id/rule_version.

Probe backend availability via pythonocc_backend_provider.is_available(). If false → set error GEOM_BACKEND_NOT_INSTALLED, return.

Try load STEP into shape: pythonocc_backend_provider.load_step_as_shape(step_bytes or step_path). If exception → status=FAILED, error STEP_IMPORT_READ_FAILED, return.

If success: set status=EXECUTED; set brep_handle (opaque, non-serializable); degraded_mode=false.

Try compute_basic_meta(brep_handle). If fails: keep status=EXECUTED but add warning META_EXTRACTION_FAILED (non-fatal).

Return envelope.

6. Outputs (BREP_LOAD_RESULT_v0 — schema locked)

status: EXECUTED | NOT_EXECUTED | FAILED

run_id: str (YYYYMMDDThhmmssZ)

rule_version: str

timestamp_utc: str (UTC)

brep_handle: opaque object (not JSON serializable; MUST NOT be written to receipts)

brep_status: NOT_AVAILABLE | AVAILABLE | ERROR [CHƯA XÁC MINH exact legacy names]; recommend align to status above

basic_meta: dict[str, int|float|str] (optional; scalar only; units [CHƯA XÁC MINH until normalization])

errors: list[{code:str, message:str, retryable:bool}]

warnings: list[{code:str, message:str}] (optional)

degraded_mode: bool (true iff status!=EXECUTED)

provenance: list[str] (optional; MUST NOT include file paths)

7. Failure modes (structured)

GEOM_BACKEND_NOT_INSTALLED → NOT_EXECUTED (retryable=false)

STEP_IMPORT_READ_FAILED → FAILED (retryable=false)

INVALID_STEP_INPUT → FAILED (retryable=false) [CHƯA XÁC MINH detection rules]

META_EXTRACTION_FAILED → WARNING only (non-fatal)

8. Evidence / Verification

Unit test: backend missing → NOT_EXECUTED, error=GEOM_BACKEND_NOT_INSTALLED, no exception escapes.

Unit test: invalid STEP bytes → FAILED, error=STEP_IMPORT_READ_FAILED.

Golden Geometry QA: 1–2 STEP samples must load to EXECUTED and produce stable basic_meta keys (values may vary with backend version). [CHƯA XÁC MINH stable meta catalog]

No-path-leak test: envelope strings/provenance do not contain Windows/POSIX absolute path patterns.

9. Fail-fast

Never return empty dict; always return BREP_LOAD_RESULT_v0 with explicit status.

No unhandled exceptions outward.

Absolute path leakage detection MUST fail-fast before serialization to receipt/report.

10. Troubleshooting

Always NOT_EXECUTED: verify pythonocc-core installation and pythonocc_backend_provider.is_available() logic; ensure environment pin matches LDR.

STEP_IMPORT_READ_FAILED: validate STEP bytes integrity (sha256) upstream; check file not corrupted; check OCC version compatibility.

META extraction warnings: treat as non-blocking; proceed to validators/healer which may still operate.

11. Non-goals

Không healing, không normalize units/axis, không freeze.

Không tạo defect, không AFR.

Không xuất report, không tạo PNG/SVG.

12. Filled example (run_id=20260212T071350Z)

BREP_LOAD_RESULT_v0: status=EXECUTED, run_id=20260212T071350Z, rule_version=ruleset_2026_02_12_v1, errors=[], warnings=[], degraded_mode=false, basic_meta={face_count:12, edge_count:36, vertex_count:24}.

## 3.2 AMBR (Bindings / Module Contract / Execution)

Source: S2

AMBR — brep_service_provider — Binding v1

Status: REVIEW (V1 candidate).

1. Module identification

ambr_id: AMBR_brep_service_provider_v1

module_name: brep_service_provider

module_type: provider (geometry core)

binding_version: v1

repo_path: tooling_dfm_advisor/geometry/brep_service_provider.py

execution_mode: SEQUENTIAL

ssot_permissions: does_not_write_ssot=true; returns data for AGL pre-freeze

owner: [CHƯA XÁC MINH]

2. Bound algorithm

adn_id: ADN_brep_service_provider_step_to_shape_backend_wrapper_v1

algorithm_name: step_to_shape_backend_wrapper

algorithm_version: v1

result_envelope: BREP_LOAD_RESULT_v0

3. Dependencies

Internal adapter: pythonocc_backend_provider (availability probe + STEP load + basic meta).

External: pythonocc-core (indirect; must be isolated behind adapter).

Forbidden: direct OCC imports in brep_service_provider (must remain in pythonocc_backend_provider).

4. Inputs/Outputs binding

Inputs: StepImportRequestV1 + step_bytes (preferred). Legacy step_path allowed only in local dev (no receipts).

Outputs: BREP_LOAD_RESULT_v0 (always returned).

5. Determinism & side effects

Determinism depends on backend + STEP content; module itself is deterministic given backend behavior.

Forbidden: filesystem writes; forbidden: storing step_bytes on disk; forbidden: leaking file paths.

Allowed: try/except mapping of backend exceptions to structured errors.

6. Change impact rules

Any change to BREP_LOAD_RESULT_v0 schema is breaking unless additive → if breaking, create BREP_LOAD_RESULT_v1 and update consumers.

Backend adapter surface changes require updating LDR adapter record + regression tests (golden corpus).

Error code taxonomy changes require updating reporter/governance mapping tables.

7. Test obligations (minimum)

Golden corpus test: at least 1 valid STEP must return EXECUTED in pinned environment.

OCC missing test: MUST return NOT_EXECUTED (no crash).

No-path-leak test (Windows + POSIX patterns).

8. Audit & traceability

Envelope MUST carry run_id and rule_version from request.

brep_handle must never be serialized to receipt; only counts/basic_meta scalars may be stored.

## 3.3 LDR (Dependencies / Runtime Constraints)

Source: S3

LDR — pythonocc_backend_provider adapter surface for brep_service — v1

Status: REVIEW (V1 candidate). Scope: internal adapter API constraints.

1. Identification

ldr_id: LDR_pythonocc_backend_provider_adapter_for_brep_service_v1

dependency_name: tooling_dfm_advisor.geometry.pythonocc_backend_provider

dependency_type: internal adapter module

decision_version: v1

used_by_modules: brep_service_provider

owner: [CHƯA XÁC MINH]

2. Purpose

Cô lập tương tác với pythonocc-core để brep_service_provider không phụ thuộc trực tiếp OCC. Giúp fail-safe khi thiếu deps nặng, và giới hạn bề mặt API được phép gọi.

3. Allowed APIs (explicit)

is_available() -> bool

load_step_as_shape(step_bytes: bytes | step_path: str) -> shape_handle

compute_basic_meta(shape_handle) -> dict[str, scalar]

4. Forbidden usage

No healing/mutation APIs exposed to brep_service_provider.

No file IO helpers exposed (write STEP, export mesh) to brep_service_provider.

No global singleton state that survives run boundaries. [CHƯA XÁC MINH current implementation]

5. Fail-safe requirement

Adapter must not raise unhandled exceptions to callers; provide typed exceptions or return envelopes upstream. In V1, brep_service_provider catches and maps exceptions.

Adapter is the only module allowed to import OCC.\* directly.

6. Upgrade / rollback policy

Any additional OCC API usage requires new/updated LDR record and approval.

pythonocc-core pinning is handled by its own LDR (e.g., LDR_pythonocc_core_v1) [CHƯA XÁC MINH location].

Rollback: revert to last-known-good environment pin (LKG). [CHƯA XÁC MINH]

7. Verification checklist

Unit: is_available returns false when OCC missing.

Integration: valid STEP loads on pinned environment.

Policy: no-path-leak static scan on error messages.

## 4. Open Questions / [CHƯA XÁC MINH]

The following items are carried over verbatim from sources (no semantics changes).

- owner: [CHƯA XÁC MINH]
- step_path: str (absolute/relative path) — MUST be scrubbed before any receipt/report serialization. [CHƯA XÁC MINH: whether legacy path exists in current code]
- brep_status: NOT_AVAILABLE | AVAILABLE | ERROR [CHƯA XÁC MINH exact legacy names]; recommend align to status above
- basic_meta: dict[str, int|float|str] (optional; scalar only; units [CHƯA XÁC MINH until normalization])
- INVALID_STEP_INPUT → FAILED (retryable=false) [CHƯA XÁC MINH detection rules]
- Golden Geometry QA: 1–2 STEP samples must load to EXECUTED and produce stable basic_meta keys (values may vary with backend version). [CHƯA XÁC MINH stable meta catalog]
- owner: [CHƯA XÁC MINH]
- owner: [CHƯA XÁC MINH]
- No global singleton state that survives run boundaries. [CHƯA XÁC MINH current implementation]
- pythonocc-core pinning is handled by its own LDR (e.g., LDR_pythonocc_core_v1) [CHƯA XÁC MINH location].
- Rollback: revert to last-known-good environment pin (LKG). [CHƯA XÁC MINH]

## 5. Change Log (Editorial)

20260304T080555Z: Created WS_PROVIDERS-IMP-SPC-v1 as a consolidated SPC. Editorial-only: canonical naming, control block, mapping table, and section re-ordering. No algorithm/contract semantics changed.

Terminology normalization performed: none (kept source terms; any divergences should be handled as 'aka' in Appendix if found).

## 6. Appendix (optional)

Aka mapping: [CHƯA XÁC MINH] — no conflicting naming detected in provided sources.
