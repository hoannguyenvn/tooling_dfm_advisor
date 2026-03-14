**WS_MODEL-GEN-SPC-v1 — REVIEW**

Document pack item: Canonical Specification (SPC).

# 0. Document Control Block

|                                  |                                                                      |
| -------------------------------- | -------------------------------------------------------------------- |
| Canonical Code                   | WS_MODEL-GEN-SPC-v1                                                  |
| Status                           | REVIEW                                                               |
| Run ID                           | 20260228T085242Z                                                     |
| WS / Module / Doc Type / Version | WS_MODEL / GEN / SPC / v1                                            |
| Created date (UTC)               | 20260228T085242Z                                                     |
| Purpose                          | Document governance consolidation only; no runtime/pipeline changes. |

## 1. Scope

This canonical SPC consolidates the contract-grade data model specifications and bindings for the model layer (WS_MODEL) into a single SSOT document. It focuses on schema definitions, required fields, invariants, determinism constraints, and fail-fast validation rules.

Non-goals: No algorithm redesign, no runtime/pipeline changes, no IO or parsing implementation details beyond boundary responsibilities stated in sources. No simulation or geometry computation.

## 2. Included Sources & Mapping Table

Table below records which source documents were used and which sections were absorbed into this canonical document.

|           |                                                                                 |                                                                              |                         |                             |                  |
| --------- | ------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- | ----------------------- | --------------------------- | ---------------- |
| source_id | filename                                                                        | original_title                                                               | original_version/status | sections_used               | notes            |
| S1        | ADN_geo_model_api_v1_geometry_and_defect_contract_schema_v1.docx                | ADN — geo_model_api_v1 — Geometry & Defect Contract Schema — v1              | REVIEW                  | absorbed (verbatim extract) | ADN_geo_model    |
| S2        | ADN_models_api_v1_context_snapshot_and_report_artifact_typed_containers_v1.docx | ADN — models_api_v1 — ContextSnapshot + ReportArtifact Typed Containers — v1 | REVIEW                  | absorbed (verbatim extract) | ADN_models       |
| S3        | ADN_step_import_api_v1_step_file_io_contract_and_sha256_integrity_v1.docx       | ADN — step_import_api_v1 — STEP File IO Contract + SHA256 Integrity — v1     | REVIEW                  | absorbed (verbatim extract) | ADN_step_import  |
| S4        | AMBR_geo_model_api_v1_v1.docx                                                   | AMBR — geo_model_api_v1 — Binding v1                                         | REVIEW                  | absorbed (verbatim extract) | AMBR_geo_model   |
| S5        | AMBR_models_api_v1_v1.docx                                                      | AMBR — models_api_v1 — Binding v1                                            | REVIEW                  | absorbed (verbatim extract) | AMBR_models      |
| S6        | AMBR_step_import_api_v1_v1.docx                                                 | AMBR — step_import_api_v1 — Binding v1                                       | REVIEW                  | absorbed (verbatim extract) | AMBR_step_import |
| S7        | ANALYSIS_RESULT_CONTRACT_v2_0_FROZEN.docx                                       | ANALYSIS_RESULT_CONTRACT_v2_0_FROZEN                                         | FROZEN                  | absorbed (verbatim extract) | ANALYSIS_RESULT  |
| S8        | LDR_re_v1.docx                                                                  | LDR — re (stdlib) — SHA256 Regex Validation — v1                             | REVIEW                  | absorbed (verbatim extract) | LDR_re           |
| S9        | LDR_enum_v1.docx                                                                | LDR — enum (stdlib) — Closed-set State Enums — v1                            | REVIEW                  | absorbed (verbatim extract) | LDR_enum         |
| S10       | LDR_dataclasses_v1.docx                                                         | LDR — dataclasses (stdlib) — Frozen Typed Containers — v1                    | REVIEW                  | absorbed (verbatim extract) | LDR_dataclasses  |

## 3. Consolidated Specification Content

## 3.1 ADN (Algorithm / Logic)

### 3.1.1 GEO_MODEL_API_v1

*Source: S1 (ADN_geo_model_api_v1_geometry_and_defect_contract_schema_v1.docx)*

ADN — geo_model_api_v1 — Geometry & Defect Contract Schema — v1

Status: REVIEW (V1 candidate). Scope: contract schema + boundary validation only.

**1. Identification**

adn_id: ADN_geo_model_api_v1_geometry_and_defect_contract_schema_v1

contract_id: GEO_MODEL_API_v1

module_name: geo_model_api_v1

module_kind: contract_module (data model definition)

algorithm_name: geometry_and_defect_schema

algorithm_version: v1

length_unit: mm (per policy)

owner: [CHƯA XÁC MINH]

authority_refs: Master Spec + Governance Spec + Addendum V1

**2. Why**

GEO_MODEL_API_v1 là SSOT contract cho geometry state (GeometryStack) và kết quả phân tích (AnalysisResultsStack). Contract này bắt buộc phải: (a) bất biến (immutable) để chống shadow mutation, (b) traceable (run_id, rule_version, geometry_version_id), (c) mang đủ 5 global UI states để phục vụ audit, và (d) cung cấp Defect rows có thể truy vết (defect_id + region_ref).

**3. Preconditions**

Naming invariants: contract_id UPPER_CASE \*\_API_v1; module_name snake_case ASCII.

run_id bắt buộc theo định dạng YYYYMMDDThhmmssZ (ISO8601 rút gọn).

Không được chứa file path / filesystem pointer trong bất kỳ field nào (đặc biệt defect location/region_ref).

Enums phải là closed-set (không dùng free-form strings).

**4. Data Structures (V1 minimum required set)**

4.1 GeoModelResponse (root payload) — REQUIRED fields:

run_id: str (YYYYMMDDThhmmssZ)

rule_version: str (version của rules/policy snapshot áp dụng)

geometry_version_id: str (ID phiên bản hình học sau freeze)

timestamp_utc: str (UTC; format canonical)

global_state: GlobalStateV1 (5 trạng thái toàn cục)

geometry: GeometryModelV1

defects: list[DefectV1] (có thể rỗng)

analysis_summary: AnalysisSummaryV1 (tổng hợp mức hoàn tất/độ tin cậy)

4.2 GlobalStateV1 — REQUIRED fields (canonical UI contract):

geometry_fidelity: GeometryFidelity (BREP / MESH / HYBRID)

degraded_mode: bool

afr_state: AfrState

analysis_completeness: AnalysisCompleteness

report_validity: ReportValidity

4.3 GeometryModelV1 — REQUIRED fields (minimum):

topology_invariants: dict[str, int] (keys phải thuộc catalog chuẩn; values >= 0)

fidelity: GeometryFidelity (phải match global_state.geometry_fidelity)

provenance: list[str] (nhật ký biến đổi; không chứa file path; append-only semantics)

4.4 DefectV1 — REQUIRED fields (traceable defect row):

defect_id: str (stable within geometry_version_id; unique)

agent_source: str (snake_case \*\_agent)

rule_version: str (echo từ root)

geometry_version_id: str (echo từ root)

timestamp_utc: str (UTC)

severity: Severity (LOW/MED/HIGH/CRITICAL)

defect_kind: str (closed-set per agent family; [CHƯA XÁC MINH] catalog)

region_ref: RegionRefV1 (khớp REGION_REF_CONVENTION_v1)

message: str (human-readable)

confidence_score: float ∈ [0.0, 1.0]

metrics: dict[str, float|int|str] (optional; values must be scalar; no arrays)

4.5 RegionRefV1 — REQUIRED fields (REGION_REF_CONVENTION_v1):

layer: AGL | DGL | HYBRID

kind: FACE | EDGE | VERTEX | PATCH

id_scheme: str (ví dụ: topo_index_v1)

id: str (stable within geometry_version_id)

uv_hint: optional (u,v)

point_hint_mm: optional (x,y,z) in mm

bbox_hint_mm: optional (min,max) in mm

ABSOLUTE FORBIDDEN: file path, mesh file pointer, screenshot path

**5. Validation rules (post_init)**

confidence_score must be in [0.0, 1.0] (else ValueError).

timestamp_utc must be present and non-empty (else ValueError).

All topology_invariants values must be non-negative (else ValueError).

Enum fields must be valid members (else ValueError).

No string field may include absolute filesystem paths (sanitization check; else ValueError).

defect_id uniqueness within defects list must hold (else ValueError).

Catalog enforcement for topology_invariants keys:

Policy key for allowed keys: thresholds.validation.topology_invariants.allowed_keys (proposed) [CHƯA XÁC MINH].

Handling unknown key: FAIL-FAST by default; optional WARNING mode controlled by policy [CHƯA XÁC MINH].

**6. Failure modes**

INVALID_CONFIDENCE_SCORE → ValueError

NEGATIVE_TOPOLOGY_INVARIANT → ValueError

MISSING_TIMESTAMP_UTC → ValueError

INVALID_ENUM_VALUE → ValueError

DEFECT_ID_NOT_UNIQUE → ValueError

FORBIDDEN_PATH_LEAK → ValueError

TOPOLOGY_INVARIANTS_KEYS_UNSTANDARDIZED → ValueError/WARNING per policy [CHƯA XÁC MINH]

**7. Evidence / Verification**

Unit tests: confidence_score bounds; missing timestamp; negative invariants; invalid enum; duplicate defect_id.

Unit tests: forbidden path detection (Windows + POSIX patterns).

Schema compatibility test: reporter consumes GeoModelResponse without key errors (contract conformance).

Golden Geometry QA: root payload required fields present for each corpus sample (contract completeness).

**8. Fail-fast**

Any violation of required fields or invariants MUST raise ValueError at construction time.

No silent fallback to defaults for rule_version/geometry_version_id/run_id.

**9. Troubleshooting**

ValueError về enums: kiểm tra caller đang dùng đúng member names; không truyền string tuỳ ý.

ValueError về forbidden path: kiểm tra provenance/message đang vô tình chứa absolute path; scrub trước khi set.

ValueError topology invariants: kiểm tra upstream normalizer/count logic; không set giá trị âm.

**10. Non-goals**

Không compute geometry, không chạy DFM agents.

Không ghi file, không network IO.

Không quyết định delivery/report formatting (chỉ cung cấp payload cho reporter).

### 3.1.2 MODELS_API_v1

*Source: S2 (ADN_models_api_v1_context_snapshot_and_report_artifact_typed_containers_v1.docx)*

ADN — models_api_v1 — ContextSnapshot + ReportArtifact Typed Containers — v1

Status: REVIEW (V1 candidate). DFM-first: contract-grade typed containers for context + report metadata.

**1. Identification**

adn_id: ADN_models_api_v1_context_snapshot_and_report_artifact_typed_containers_v1

contract_id: MODELS_API_v1

module_name: models_api_v1

module_kind: contract_module (data model definition)

algorithm_name: context_snapshot_and_report_artifact_typed_containers

algorithm_version: v1

owner: [CHƯA XÁC MINH]

authority_refs: Master Spec + Governance Spec + Addendum V1

**2. Why**

Module này chuẩn hóa dữ liệu bắc cầu giữa pipeline stages và lớp Advisory Artifact. Hai cấu trúc tối thiểu cần khoá: ContextSnapshot (ảnh chụp ngữ cảnh) và ReportArtifact (siêu dữ liệu báo cáo). DFM-first yêu cầu: traceability bắt buộc (run_id/rule_version/geometry_version_id), không rò rỉ đường dẫn file, và closed-set states để UI/report không drift.

**3. Preconditions**

Naming: contract_id UPPER_CASE \*\_API_v1; module_name snake_case ASCII.

run_id bắt buộc theo định dạng YYYYMMDDThhmmssZ.

Report delivery final chỉ PDF/HTML (report_format closed-set).

Không được chứa file path / filesystem pointers trong bất kỳ field string nào.

**4. Data Structures (V1 minimum required set)**

4.1 ContextSnapshotV1 — REQUIRED fields:

context_id: str (opaque id; NOT a file path)

context_state: ContextState (OPEN | FROZEN | INVALID)

rule_version: str (policy snapshot applied)

timestamp_utc: str (UTC)

notes: optional[str] (no paths)

4.2 ReportArtifactV1 — REQUIRED fields:

artifact_id: str (opaque id; NOT a file path)

report_state: ReportState (DRAFT | VALID | INVALID | DELIVERY_COMPLETE)

report_format: ReportFormat (PDF | HTML) # forbidden: JSON/DOCX

run_id: str (YYYYMMDDThhmmssZ)

rule_version: str

geometry_version_id: str

timestamp_utc: str (UTC)

report_validity: ReportValidity (VALID | INVALID | DEGRADED) # aligns UI global state

ephemeral_purge_state: EphemeralPurgeState (PENDING | PURGED | FAILED)

4.3 Closed-set enums:

ContextState: OPEN, FROZEN, INVALID

ReportState: DRAFT, VALID, INVALID, DELIVERY_COMPLETE

ReportFormat: PDF, HTML

ReportValidity: VALID, INVALID, DEGRADED

EphemeralPurgeState: PENDING, PURGED, FAILED

**5. Validation rules (fail-fast)**

run_id must match YYYYMMDDThhmmssZ (else ValueError(INVALID_RUN_ID)).

report_format must be PDF or HTML only (else ValueError(REPORT_FORMAT_FORBIDDEN)).

No string field may include absolute filesystem paths or traversal tokens (else ValueError(FORBIDDEN_PATH_LEAK)).

If report_state == DELIVERY_COMPLETE: ephemeral_purge_state must be PURGED (else ValueError(DELIVERY_COMPLETE_WITHOUT_PURGE)).

If context_state == FROZEN: context_id must be non-empty and stable (caller-owned) [CHƯA XÁC MINH stability definition].

**6. Failure modes**

INVALID_RUN_ID → ValueError

REPORT_FORMAT_FORBIDDEN → ValueError

FORBIDDEN_PATH_LEAK → ValueError

DELIVERY_COMPLETE_WITHOUT_PURGE → ValueError

INVALID_ENUM_VALUE → ValueError

**7. Evidence / Verification**

Unit tests: invalid run_id rejected; forbidden formats rejected; delivery_complete requires purge.

Unit tests: forbidden path detection for Windows/POSIX patterns.

Integration: reporter uses ReportArtifactV1 to stamp run_id/rule_version/geometry_version_id in final PDF/HTML metadata.

Governance: receipt for EPHEMERAL_PURGE_RESULT\_\* should map to ReportArtifact.ephemeral_purge_state.

**8. Fail-fast**

Invalid instances MUST raise ValueError at construction time; no silent defaults for run_id/rule_version/geometry_version_id.

No IO/network side-effects.

**9. Troubleshooting**

REPORT_FORMAT_FORBIDDEN: kiểm tra caller không cấu hình JSON/DOCX; chỉ cho phép PDF/HTML.

DELIVERY_COMPLETE_WITHOUT_PURGE: đảm bảo delivery stage chạy purge và ghi receipt trước khi set DELIVERY_COMPLETE.

FORBIDDEN_PATH_LEAK: scrub notes/ids không chứa path; id phải opaque.

**10. Non-goals**

Không tạo report content, không render PDF/HTML.

Không tạo/ghi assets (PNG/SVG/Mesh).

Không mutate SSOT/tooling model runtime; chỉ định nghĩa schema.

### 3.1.3 STEP_IMPORT_API_v1

*Source: S3 (ADN_step_import_api_v1_step_file_io_contract_and_sha256_integrity_v1.docx)*

ADN — step_import_api_v1 — STEP File IO Contract + SHA256 Integrity — v1

Status: REVIEW (V1 candidate). Scope: data contract + schema validation only.

**1. Identification**

adn_id: ADN_step_import_api_v1_step_file_io_contract_and_sha256_integrity_v1

contract_id: STEP_IMPORT_API_v1

module_name: step_import_api_v1

module_kind: contract_module (data model definition)

algorithm_name: step_file_io_contract_and_sha256_integrity

algorithm_version: v1

owner: [CHƯA XÁC MINH]

authority_refs: Master Spec + Governance Spec + Addendum V1

**2. Why**

Contract này là rào chắn đầu vào cho STEP trước khi vào geometry backend: (a) chống tampering bằng SHA256 metadata, (b) fail-fast các payload sai, (c) cung cấp traceability bắt buộc (run_id, rule_version), và (d) tránh rò rỉ đường dẫn file/asset pointers.

**3. Preconditions**

run_id bắt buộc theo định dạng YYYYMMDDThhmmssZ.

Không được lưu/serialize absolute filesystem paths vào SSOT, receipts, hoặc report.

sha256_hex phải là 64 ký tự hex; được normalize về lowercase.

Hệ đơn vị chiều dài chuẩn hệ thống: mm (unit_hint chỉ là gợi ý nhập).

**4. Data Structures (V1 minimum required set)**

4.1 StepImportRequestV1 — REQUIRED fields:

run_id: str (YYYYMMDDThhmmssZ)

rule_version: str (snapshot rules/policy version)

source_ref: str (opaque identifier; NOT a file path)

source_filename: str (sanitized; no directory separators)

bytes_size: int (>=0)

sha256_hex: str (64 hex; normalized lowercase)

unit_hint: UnitHint (MM | INCH | AUTO)

timestamp_utc: str (UTC)

4.2 STEP_IMPORT_RESULT_v0 — result envelope (schema-level) — REQUIRED fields:

status: StepImportStatus (PASS | FAIL | NOT_EXECUTED)

run_id: str

rule_version: str

timestamp_utc: str

geometry_version_id: optional[str] (required iff status==PASS)

errors: list[ErrorItem] (must be empty iff status==PASS)

warnings: list[WarningItem] (optional)

degraded_mode: bool

metrics: dict[str, int|float|str] (optional; scalar only)

4.3 Enums (closed-set):

UnitHint: MM, INCH, AUTO

StepImportStatus: PASS, FAIL, NOT_EXECUTED

ErrorSeverity: LOW, MED, HIGH, CRITICAL (optional if you already have Severity in GEO_MODEL_API_v1)

**5. Validation rules (fail-fast)**

sha256_hex must match regex ^[0-9a-fA-F]{64}$; normalize to lowercase; else ValueError(INVALID_SHA256_FORMAT).

bytes_size must be >= 0; else ValueError(NEGATIVE_BYTES_SIZE).

If status == PASS: errors must be empty AND geometry_version_id must be present; else ValueError(PASS_INVARIANT_VIOLATION).

If status != PASS: geometry_version_id must be None/empty; else ValueError(GEOMETRY_VERSION_ID_INCONSISTENT).

source_filename must not contain path separators or drive letters; else ValueError(FORBIDDEN_PATH_LEAK).

source_ref must not be an absolute path nor contain directory traversal tokens; else ValueError(FORBIDDEN_PATH_LEAK).

unit_hint must be one of the closed-set members; else ValueError(UNIT_HINT_INVALID).

**6. Responsibility split (DFM-first boundary)**

step_import_api_v1 OWNS: schema + validation of sha256/bytes_size/invariants + forbidden-path checks.

step_import_provider OWNS: actual file IO, hashing computation/verification, and STEP parsing/transfer into backend.

governance/orchestrator OWNS: receipts, policy snapshotting (rule_version), and enforcement when SHA256 mismatch occurs.

**7. Failure modes (structured)**

INVALID_SHA256_FORMAT → ValueError

NEGATIVE_BYTES_SIZE → ValueError

PASS_INVARIANT_VIOLATION → ValueError

UNIT_HINT_INVALID → ValueError

FORBIDDEN_PATH_LEAK → ValueError

MISSING_GEOMETRY_VERSION_ID when PASS → ValueError

SHA256_MISMATCH handling → [CHƯA XÁC MINH] (should be FAIL with error code in provider/governance)

**8. Evidence / Verification**

Unit tests: sha256 regex acceptance/rejection; normalization to lowercase; bytes_size negative; PASS invariant; forbidden path patterns (Windows/POSIX).

Contract conformance tests: StepImportRequestV1 and STEP_IMPORT_RESULT_v0 required fields present.

Golden Geometry QA: every corpus run must carry request metadata (sha256/bytes_size) in receipt (without file path). [CHƯA XÁC MINH receipt wiring]

**9. Fail-fast**

Any invalid request/result instance MUST raise ValueError at construction time (no silent fallback).

No absolute paths in any serialized fields; violations MUST fail-fast.

**10. Non-goals**

Không đọc/parse STEP.

Không compute SHA256 từ file bytes.

Không tạo AGL/DFM defects.

Không làm delivery/report formatting.

### 3.1.4 ANALYSIS_RESULT_CONTRACT_v2_0

*Source: S7 (ANALYSIS_RESULT_CONTRACT_v2_0_FROZEN.docx)*

ANALYSIS_RESULT_CONTRACT_v2_0_FROZEN

doc_id: ANALYSIS_RESULT_CONTRACT_v2_0

status: FROZEN

system_name: tooling_dfm_advisor

architecture_level: INDUSTRIAL_GRADE_FREEZE

version: V2_0

freeze_date_utc: 2026-02-27T13:59:27Z

Why

Unify analysis output schema for all agents.

Preconditions

AGL state must be FROZEN.

Mandatory Fields

status, run_id, geometry_version_id, policy_version, rule_version, agent_source, measurements, issues.

Issue Schema

issue_id, severity, region_ref, message, confidence_score.

Serialization Rule

Canonical JSON ordering required for deterministic hashing.

Determinism Binding

Result hash stable under identical geometry_version_id and policy_version.

Fail-Fast

Unknown top-level field invalidates result.

Cross Reference

Dependent on DFM_POLICY_MASTER_SPEC_v2_0 and AGL_LIFECYCLE_MODEL_v2_0.

## 3.2 AMBR (Bindings / Module Contract / Execution)

### 3.2.1 Binding: geo_model_api_v1

*Source: S4 (AMBR_geo_model_api_v1_v1.docx)*

AMBR — geo_model_api_v1 — Binding v1

Status: REVIEW (V1 candidate).

**1. Module identification**

ambr_id: AMBR_geo_model_api_v1_v1

module_name: geo_model_api_v1

module_kind: contract_module

binding_version: v1

repo_path: tooling_dfm_advisor/contracts/geo_model_api_v1.py

execution_mode: SEQUENTIAL (schema definition)

ssot_permissions: read_only=true; may_mutate_tooling_model=false

owner: [CHƯA XÁC MINH]

**2. Bound ADN**

adn_id: ADN_geo_model_api_v1_geometry_and_defect_contract_schema_v1

contract_id: GEO_MODEL_API_v1

**3. Dependencies (LDR)**

LDR_dataclasses_v1 (stdlib; reused)

LDR_enum_v1 (stdlib; required for closed-set states)

**4. Determinism & side effects**

Deterministic: same inputs → same instance or same ValueError.

Forbidden: IO, network, logging absolute paths.

Allowed: post_init validation only.

**5. Contract upgrade policy**

Breaking changes (remove/rename required field; change enum members/meaning) → create GEO_MODEL_API_v2.

Non-breaking additive fields → stay in v1; update docs and conformance tests.

Any change requires updating expected contract conformance tests + golden corpus gating.

**6. Test obligations (minimum)**

Schema conformance unit tests for all fail-fast rules.

Golden corpus: each sample must produce GeoModelResponse with required fields.

Reporter compatibility test: report_generator consumes payload without KeyError.

**7. Audit & traceability**

GeoModelResponse MUST carry run_id, rule_version, geometry_version_id, timestamp_utc.

Defect rows MUST carry defect_id + agent_source + region_ref.

### 3.2.2 Binding: models_api_v1

*Source: S5 (AMBR_models_api_v1_v1.docx)*

AMBR — models_api_v1 — Binding v1

Status: REVIEW (V1 candidate).

**1. Module identification**

ambr_id: AMBR_models_api_v1_v1

module_name: models_api_v1

module_kind: contract_module

binding_version: v1

repo_path: tooling_dfm_advisor/contracts/models_api_v1.py

execution_mode: SEQUENTIAL (schema definition)

ssot_permissions: read_only=true; may_mutate_tooling_model=false

owner: [CHƯA XÁC MINH]

**2. Bound ADN**

adn_id: ADN_models_api_v1_context_snapshot_and_report_artifact_typed_containers_v1

contract_id: MODELS_API_v1

algorithm_version: v1

**3. Dependencies (LDR)**

LDR_dataclasses_v1 (stdlib; @dataclass(frozen=True) recommended)

LDR_enum_v1 (stdlib; closed-set enums; may reuse existing)

LDR_re_v1 (stdlib; optional if run_id validation uses regex)

**4. Determinism & side effects**

Deterministic: same fields → same instance or ValueError.

Forbidden: file IO, network IO, logging of filesystem paths.

Allowed: post_init validation only.

**5. Change impact rules**

Breaking change (remove/rename required field; change enum members/meaning) → create MODELS_API_v2 and update \_contract_index.

Additive optional fields → remain in v1; update conformance tests.

Any change requires updating reporter/governance conformance tests.

**6. Test obligations (minimum)**

Unit tests for all fail-fast rules.

No-path-leak tests.

Reporter integration test: final report metadata includes run_id/rule_version/geometry_version_id and format is PDF/HTML only.

**7. Audit & traceability**

ReportArtifactV1 MUST carry run_id/rule_version/geometry_version_id.

ephemeral_purge_state MUST be PURGED before report_state DELIVERY_COMPLETE.

### 3.2.3 Binding: step_import_api_v1

*Source: S6 (AMBR_step_import_api_v1_v1.docx)*

AMBR — step_import_api_v1 — Binding v1

Status: REVIEW (V1 candidate).

**1. Module identification**

ambr_id: AMBR_step_import_api_v1_v1

module_name: step_import_api_v1

module_kind: contract_module

binding_version: v1

repo_path: tooling_dfm_advisor/contracts/step_import_api_v1.py

execution_mode: SEQUENTIAL (schema definition)

ssot_permissions: read_only=true; may_mutate_tooling_model=false

owner: [CHƯA XÁC MINH]

**2. Bound algorithm (ADN)**

adn_id: ADN_step_import_api_v1_step_file_io_contract_and_sha256_integrity_v1

contract_id: STEP_IMPORT_API_v1

algorithm_name: step_file_io_contract_and_sha256_integrity

algorithm_version: v1

**3. Dependencies (LDR)**

LDR_re_v1 (stdlib; SHA256 regex validation)

LDR_dataclasses_v1 (reused; stdlib)

LDR_typing_v1 (reused/optional; stdlib)

LDR_enum_v1 (reused; stdlib) — if using enums for closed-set.

**4. Determinism & side effects**

Deterministic validation; same inputs → same instance or same ValueError.

Forbidden: file IO, hashing computation, network IO.

Forbidden: storing or emitting absolute filesystem paths.

**5. Change impact rules**

Breaking schema change (rename/remove required field; change meaning of status/enums) → create STEP_IMPORT_API_v2 and update \_contract_index mapping.

Non-breaking additive fields → stay in v1; update conformance tests + docs.

**6. Test obligations (minimum)**

Unit tests for all fail-fast rules and forbidden-path sanitizer.

Contract conformance tests for required fields.

Integration smoke test: step_import_provider accepts StepImportRequestV1 and produces STEP_IMPORT_RESULT_v0 without unhandled exceptions (OCC absent allowed → NOT_EXECUTED).

**7. Audit & traceability**

Request/Result MUST carry run_id and rule_version.

Receipts MUST store sha256_hex and bytes_size, but MUST NOT store absolute path.

source_ref is opaque; may be used to find ephemeral storage entry (outside this module).

## 3.3 LDR (Dependencies / Libraries / Runtime constraints)

### 3.3.1 stdlib re

*Source: S8 (LDR_re_v1.docx)*

LDR — re (stdlib) — SHA256 Regex Validation — v1

Status: REVIEW (V1 candidate).

**1. Identification**

ldr_id: LDR_re_v1

library_name: re

library_version: stdlib (python)

decision_version: v1

language_runtime: Python [CHƯA XÁC MINH version pin]

used_by: step_import_api_v1

**2. Purpose**

Dùng re.compile/fullmatch để validate SHA256 string (64 hex chars) một cách deterministic và fail-fast.

**3. Allowed usage**

re.compile(r'^[0-9a-fA-F]{64}$') to build \_SHA256_RE.

Use fullmatch/match; prefer fullmatch to avoid partial acceptance.

**4. Forbidden usage**

Không dùng regex phức tạp hoặc pattern không neo (anchored) gây chấp nhận hash sai.

Không dùng user-controlled regex patterns.

**5. Risks & mitigations**

Risk: pattern too loose → accept invalid hash; mitigate by unit tests boundary.

Risk: python version drift; mitigate by pin Python in CI and record in manifest/receipt. [CHƯA XÁC MINH implementation]

**6. Verification checklist**

Unit tests: accept lower/upper hex; reject wrong length/non-hex; ensure normalization to lowercase.

CI: verify python version pin (runtime report) [CHƯA XÁC MINH].

### 3.3.2 stdlib enum

*Source: S9 (LDR_enum_v1.docx)*

LDR — enum (stdlib) — Closed-set State Enums — v1

Status: REVIEW (V1 candidate).

**1. Identification**

ldr_id: LDR_enum_v1

library_name: enum

library_version: stdlib (python)

decision_version: v1

language_runtime: Python [CHƯA XÁC MINH version pin]

used_by: geo_model_api_v1 (and any module requiring closed-set states)

**2. Purpose**

Dùng enum để khoá closed-set cho AfrState/GeometryFidelity/AnalysisCompleteness/ReportValidity/Severity, tránh free-form strings làm gãy UI contract và làm drift dữ liệu.

**3. Allowed usage**

enum.Enum for string-like members (recommended).

No dynamic enum creation at runtime.

**4. Forbidden usage**

Không dùng string constants tự do thay thế enum.

Không thay đổi member values mà không bump contract major (v2).

**5. Risks & mitigations**

Risk: enum member changes are breaking → mitigate by strict upgrade policy (GEO_MODEL_API_v2).

Risk: python version drift → mitigate by pin Python in CI and runtime manifest.

**6. Verification checklist**

Unit test: invalid enum value should fail-fast at GeoModelResponse init.

CI: record python version in receipt/manifest [CHƯA XÁC MINH implementation].

### 3.3.3 stdlib dataclasses

*Source: S10 (LDR_dataclasses_v1.docx)*

LDR — dataclasses (stdlib) — Frozen Typed Containers — v1

Status: REVIEW (V1 candidate).

**1. Identification**

ldr_id: LDR_dataclasses_v1

library_name: dataclasses

library_version: stdlib (python)

decision_version: v1

language_runtime: Python [CHƯA XÁC MINH version pin]

used_by_modules: models_api_v1 (and other contract modules)

**2. Purpose**

Dùng @dataclass(frozen=True) để tạo typed containers bất biến, giảm rủi ro silent mutation trong SSOT/report metadata.

**3. Allowed usage**

@dataclass(frozen=True) for immutable containers.

Type annotations for all fields.

\_\_post_init\_\_ validation (fail-fast) allowed.

**4. Forbidden usage**

Mutable dataclass to store runtime-changing state.

Embedding file paths/pointers inside container strings.

**5. Risks & mitigations**

Python version drift may affect dataclass features → mitigate by pin python version in CI/runtime manifests.

Schema drift without versioned contracts → mitigate by MODELS_API_v2 for breaking changes + contract conformance tests.

**6. Verification checklist**

CI: record python version pin [CHƯA XÁC MINH pipeline wiring].

Unit tests: containers are frozen (mutation raises).

## 4. Open Questions / [CHƯA XÁC MINH]

**[CHƯA XÁC MINH]** CI: verify python version pin (runtime report) [CHƯA XÁC MINH].

**[CHƯA XÁC MINH]** Handling unknown key: FAIL-FAST by default; optional WARNING mode controlled by policy [CHƯA XÁC MINH].

**[CHƯA XÁC MINH]** Policy key for allowed keys: thresholds.validation.topology_invariants.allowed_keys (proposed) [CHƯA XÁC MINH].

**[CHƯA XÁC MINH]** SHA256_MISMATCH handling → [CHƯA XÁC MINH] (should be FAIL with error code in provider/governance)

**[CHƯA XÁC MINH]** TOPOLOGY_INVARIANTS_KEYS_UNSTANDARDIZED → ValueError/WARNING per policy [CHƯA XÁC MINH]

**[CHƯA XÁC MINH]** defect_kind: str (closed-set per agent family; [CHƯA XÁC MINH] catalog)

**[CHƯA XÁC MINH]** owner: [CHƯA XÁC MINH]

## 5. Change Log (Editorial)

This document is a governance consolidation (rename + merge + formatting). No runtime or technical semantics changes are intended.

Consolidated sources S1-S10 into WS_MODEL-GEN-SPC-v1 (SPC).

Standardized file naming and inserted Document Control Block, mapping table, and traceability markers (Source: Sx).

No technical semantics changes intended; content is primarily verbatim extracts from sources.

## 6. Appendix

## Cross References (canonical codes only)

WS_GOV-ALL-POL-v1 (governance policy schema and thresholds)
WS_GOV-ORC-SPC-v1 (orchestrator deterministic coordination)
WS_GOV-ALL-SPC-v1 (system and governance master specifications)
