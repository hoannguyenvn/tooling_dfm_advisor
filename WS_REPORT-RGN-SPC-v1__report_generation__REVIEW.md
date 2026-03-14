**WS_REPORT-RGN-SPC-v1 (REVIEW)**

# 0. Document Control Block

|                                  |                                                                      |
| -------------------------------- | -------------------------------------------------------------------- |
| Canonical Code                   | WS_REPORT-RGN-SPC-v1                                                 |
| Status                           | REVIEW                                                               |
| Run ID (UTC)                     | 20260304T113618Z                                                     |
| Created date (UTC)               | 20260304 113618                                                      |
| WS / Module / Doc Type / Version | WS=WS_REPORT; MODULE_3LA=RGN; DOC_TYPE=SPC; VERSION=v1               |
| Purpose                          | Document governance consolidation only; no runtime/pipeline changes. |

## 1. Scope

This canonical SPC consolidates the existing Reporter-layer composition and interface documents under WS_REPORT-RGN. Content is inherited from sources and reorganized for naming/traceability only; no technical semantics are changed.

Non-goals: Any new algorithm/rules not present in sources; any code implementation; any runtime/pipeline redesign.

## 2. Included Sources & Mapping Table

|           |                                                                       |                                                                                 |                         |               |                           |
| --------- | --------------------------------------------------------------------- | ------------------------------------------------------------------------------- | ----------------------- | ------------- | ------------------------- |
| source_id | filename                                                              | original_title                                                                  | original_version/status | sections_used | notes                     |
| S1        | ADN_report_generator_geomodelresponse_to_report_model_v1.docx         | ADN - report_generator                                                          | REVIEW                  | 3.1 ADN       | module=report_generator   |
| S2        | AMBR_report_generator_v1.docx                                         | AMBR - report_generator                                                         | REVIEW                  | 3.2 AMBR      | module=report_generator   |
| S3        | AMBR_report_interfaces_v1.docx                                        | AMBR (V1, REVIEW) - report_interfaces                                           | REVIEW                  | 3.2 AMBR      | module=report_interfaces  |
| S4        | ADN_report_interfaces_reporter_contract_protocol_definition_v1.docx   | ADN (V1, REVIEW) - report_interfaces - reporter_contract_protocol_definition_v1 | REVIEW                  | 3.1 ADN       | module=report_interfaces  |
| S5        | AMBR_report_module_stub_v1.docx                                       | AMBR V1 — report_module_stub (Module Binding & Runtime Constraints)             | REVIEW                  | 3.2 AMBR      | module=report_module_stub |
| S6        | LDR_report_module_stub_v1.docx                                        | LDR V1 — report_module_stub (Library Dependency Record)                         | REVIEW                  | 3.3 LDR       | module=report_module_stub |
| S7        | ADN_report_module_stub_reporter_scaffold_and_healthcheck_stub_v1.docx | ADN V1 — report_module_stub (Reporter Scaffold & Healthcheck)                   | REVIEW                  | 3.1 ADN       | module=report_module_stub |
| S8        | LDR_internal_contract_geo_model_api_v1_for_report_generator_v1.docx   | LDR - report_generator                                                          | REVIEW                  | 3.3 LDR       | module=report_generator   |
| S9        | LDR_report_interfaces_dependencies_v1.docx                            | LDR (V1, REVIEW) - report_interfaces - dependency records                       | REVIEW                  | 3.3 LDR       | module=report_interfaces  |

## 3. Consolidated Specification Content

## 3.1 ADN (Algorithm / Logic)

### ADN - report_generator

Source: S1

Verbatim extract (governance-only formatting):

ADN - report_generator

Algorithm: geomodelresponse_to_report_model (v1)

Document Metadata

Note: This ADN is a compliance-normalized V1 specification. It supersedes the prior facts-only extraction of the v0 flat-dict transform.

Problem Statement

Transform a structured GeoModelResponse (read-only SSOT view) into a deterministic, machine-readable report model that can be rendered into a customer-facing advisory artifact. The report model must match the canonical 6-section layout and must carry traceable metadata (run_id, rule_version, geometry_version_id).

Scope

Flatten and aggregate defect data into a traceable defect table (Defect ID + severity + region reference + advisory).

Compute severity counts and a single summary_level using priority FATAL > ERROR > WARN > INFO.

Populate the canonical 6 report sections with minimum viable content (DFM-first).

Emit a structured result envelope REPORT_COMPOSITION_RESULT_v1 that never crashes the pipeline.

Non-goals

No PDF/HTML rendering, no UI generation.

No filesystem I/O and no persistence decisions.

No geometry healing and no mutation of SSOT/tooling model.

No creation of new defect findings; consumes existing defects only.

Inputs

Primary input: GeoModelResponse (GEO_MODEL_API_v1). The module MUST treat the object as read-only.

Required fields (minimum contract subset)

run_id (format: YYYYMMDDThhmmssZ) - propagated unchanged.

geometry_version_id - propagated unchanged.

defects: list[Defect] (may be empty).

Defect minimum fields used by this algorithm

defect_id (string) - if missing, a deterministic surrogate_id MUST be derived (see Determinism).

severity (enum: INFO|WARN|ERROR|FATAL).

region_ref (stable reference per REGION_REF_CONVENTION_v1).

title/message (human-readable short text).

advisory (recommended action text) or advisory_tags (optional).

External inputs (governance)

rule_version (string) - provided by orchestrator/governance_policy.json; embedded into report metadata.

analysis_completeness, degraded_mode, geometry_fidelity (if available in GeoModelResponse; else marked UNKNOWN).

Outputs

Output is a structured envelope REPORT_COMPOSITION_RESULT_v1.

REPORT_COMPOSITION_RESULT_v1 schema (normative)

status: OK | DEGRADED | FAILED

run_id: string

rule_version: string

geometry_version_id: string

report_model: dict (see below)

issues: list[dict] (structured, machine-readable; empty if OK)

report_model schema (canonical 6-section layout)

report_model MUST include exactly these top-level keys (additional keys permitted only under extensions.\*):

metadata

executive_summary

geometry_health_status

defect_table

advisory_narrative

confidence_and_limitations

geometry_fidelity_honesty_notes

extensions (optional)

metadata

run_id, rule_version, geometry_version_id

generated_at_utc (ISO 8601 UTC)

report_format_allowed: PDF|HTML (final delivery constraint; composition does not render)

geometry_health_status

summary_level: FATAL|ERROR|WARN|INFO|OK

severity_counts: {info,warn,error,fatal}

geometry_fidelity: brep|mesh|hybrid|unknown

degraded_mode: true|false|unknown

analysis_completeness: complete|partial|unknown

defect_table

rows: list[defect_row]

columns: list[str] (stable ordering for rendering)

row_count

defect_row (minimum)

defect_id

severity

agent_name (if available)

region_ref

title

advisory

confidence_score (optional)

asset_refs (optional; MUST NOT contain file paths)

Algorithm

Step-by-step logic

Validate presence of required top-level fields in GeoModelResponse (run_id, geometry_version_id, defects). If invalid: return status=FAILED with issues and an empty but well-formed report_model.

Initialize severity counters (INFO/WARN/ERROR/FATAL) to 0.

For each defect: normalize severity to the allowed enum; if invalid: record an issue and map severity to WARN (conservative) for reporting.

Derive defect_id: prefer defect.defect_id. If missing, compute surrogate_id = SHA256(run_id + geometry_version_id + stable_region_ref + title) truncated to 12 hex chars.

Build defect_table rows with a stable column set and stable ordering (see Determinism).

Compute summary_level using priority: FATAL > ERROR > WARN > INFO. If no defects: summary_level=OK.

Populate all 6 report sections. If some information is unavailable, mark values as 'unknown' and add a limitation note.

Return REPORT_COMPOSITION_RESULT_v1 with status=OK if no issues, status=DEGRADED if issues exist but report_model is deliverable.

Determinism Requirements

Output MUST be deterministic for the same input GeoModelResponse content.

defect_table ordering MUST be stable: sort key = (severity_rank desc, defect_id asc).

No timestamps except generated_at_utc in metadata; generated_at_utc does not affect determinism of defect ordering and counts.

Failure Modes and Fail-safe Behavior

Missing/invalid input structure: return status=FAILED; do not raise unhandled exceptions.

Unknown severity: record issue and map to WARN; status=DEGRADED.

Missing defect_id: generate surrogate_id deterministically; status=DEGRADED (limitation).

Asset references: MUST NOT include absolute paths/pointers; if such values appear upstream, scrub to hash-only and record issue.

Compliance Mapping (New Standard)

Naming: module snake_case 'report_generator'; envelope UPPER_CASE 'REPORT_COMPOSITION_RESULT_v1'.

Traceability: report_model.metadata includes run_id, rule_version, geometry_version_id.

Delivery: final artifact MUST be PDF/HTML only (enforced by delivery stage; composition declares constraint).

Visuals: asset_refs allow PNG snapshots and SVG sections as ephemeral assets (no persistence, no paths).

Complexity

Time: O(N_defects)

Memory: O(N_defects) for defect_table rows

doc_id | ADN_report_generator_geomodelresponse_to_report_model_v1

status | REVIEW

issue_date_utc | 2026-02-24T13:41:55Z

effective_date_utc | [CHUA XAC MINH]

module_name | report_generator

module_type | reporter

package | tooling_dfm_advisor.reporter

file_name | report_generator.py

algorithm_name | geomodelresponse_to_report_model

algorithm_version | v1

output_envelope | REPORT_COMPOSITION_RESULT_v1

input_contract | GEO_MODEL_API_v1 (GeoModelResponse)

pipeline_stage | Step 7 - report_composition_and_delivery (composition sub-step)

### ADN (V1, REVIEW) - report_interfaces - reporter_contract_protocol_definition_v1

Source: S4

Verbatim extract (governance-only formatting):

ADN (V1, REVIEW) - report_interfaces - reporter_contract_protocol_definition_v1

1. Problem statement

Define a strict, versioned interface contract (Protocol) for Reporter-layer components that consume GEO_MODEL_API_v1 outputs and produce a canonical, traceable report composition result. The contract prevents signature drift across multiple implementations (composer, renderer, delivery adapter) and enforces governance invariants: determinism, fail-safe operation, and traceability metadata propagation.

2. Scope

Declare Protocol GeoModelApiV1Consumer with a single entry-point method invoke(payload) -> result_envelope.

Normalize payload type to the typed contract (GeoModelResponse from GEO_MODEL_API_v1), not raw dict.

Normalize return type to an UPPER_CASE result envelope with version suffix: REPORT_COMPOSITION_RESULT_v1.

Specify mandatory traceability metadata propagation: run_id, geometry_version_id; and required rule_version in output (see Notes).

Define fail-safe status semantics: OK / DEGRADED / NOT_EXECUTED / FAILED, with structured issues list.

3. Non-goals

No business logic, no geometry computation, no healing.

No filesystem IO, no PDF/HTML rendering, and no UI concerns inside this module.

This module does not define the full GEO_MODEL_API_v1 schema; it references it as the SSOT contract.

4. Inputs

Input payload type (normative): GeoModelResponse (from GEO_MODEL_API_v1).

Minimum required fields for Reporter consumption:

run_id: str, format YYYYMMDDThhmmssZ (ISO 8601 basic, UTC).

geometry_version_id: str.

defects: list[Defect] (may be empty).

Note: rule_version may not be present in the current GEO_MODEL_API_v1 contract. In that case, implementations MUST still set rule_version in the output, and MUST append an issue code MISSING_RULE_VERSION to preserve audit clarity.

5. Outputs

Return type (normative): REPORT_COMPOSITION_RESULT_v1 (result envelope).

Minimum required envelope fields:

status: str in {OK, DEGRADED, NOT_EXECUTED, FAILED}.

run_id: str (same as input).

geometry_version_id: str (same as input).

rule_version: str (must be present; may be 'UNKNOWN' if upstream missing, with issue).

report_model: dict - canonical 6-part layout (see below).

issues: list[dict] - each item: {code, severity, message}.

degraded_mode: bool - true iff status in {DEGRADED, NOT_EXECUTED}.

Canonical report_model layout (normative keys):

executive_summary

geometry_status

defect_table

advisory_content

confidence_and_limits

geometry_fidelity_notes

This module only constrains the presence of these sections and traceability metadata; the concrete content schema is owned by the composing implementation (e.g., report_generator) and the reporter delivery layer.

6. Determinism and ordering rules

Protocol definition is static; however, any implementation conforming to this Protocol MUST be deterministic.

Given identical input payload and identical policy configuration, output MUST be identical (byte-wise for stable JSON serialization of report_model is recommended).

If defects are listed, implementations MUST document whether they preserve input order or apply a deterministic sort.

7. Failure modes and fail-safe requirements

Implementations MUST NOT throw unhandled exceptions that break the pipeline.

If payload is invalid / missing required fields: return status=FAILED and include issue code INVALID_PAYLOAD_SCHEMA.

If downstream prerequisites are missing (e.g., delivery backend unavailable): return status=NOT_EXECUTED with issue code PREREQ_MISSING.

If partial report can be produced: return status=DEGRADED with issue code PARTIAL_REPORT.

Fail-fast boundary: this module is a contract definition and contains no runtime logic; fail-fast applies to implementations during validation and delivery stages.

8. Evidence and audit hooks

All implementations must propagate run_id and geometry_version_id into the result envelope.

The result envelope issues list must be sufficient for post-mortem audit (code + severity + message).

Implementations should ensure receipt logging occurs at the orchestrator/delivery layer; this protocol mandates the metadata needed for receipts.

9. Change impact and versioning policy

Any breaking change to invoke signature or to required output keys requires a new protocol version (e.g., GeoModelApiV1ConsumerV2) and a new ADN record.

Tightening types (dict -> TypedDict/dataclass) is considered breaking unless strictly backward compatible.

doc_id | TOOLING_DFM_ADVISOR_REPORT_INTERFACES_ADN_v1

status | REVIEW

effective_date_utc | 2026-02-24

module_name | report_interfaces

package | tooling_dfm_advisor.reporter

file_name | report_interfaces.py

module_type | reporter_interface_protocol

algorithm_name | reporter_contract_protocol_definition

algorithm_version | v1

pipeline_stage | Step 7 - Report Composition & Delivery

### ADN V1 — report_module_stub (Reporter Scaffold & Healthcheck)

Source: S7

Verbatim extract (governance-only formatting):

ADN V1 — report_module_stub (Reporter Scaffold & Healthcheck)

1. Document Metadata

1. Problem Statement

Cung cấp một scaffold tối thiểu cho lớp Reporter để phục vụ tích hợp hệ thống và kiểm tra sức khỏe (healthcheck) trong giai đoạn sơ khởi. Scaffold này bảo đảm pipeline có “điểm neo” ở lớp báo cáo ngay cả khi chưa có nghiệp vụ render/xuất artifact thật.

3. Scope

Khởi tạo lớp ReporterModule với 2 endpoint: healthcheck(...) và consume_geo_model_api_v1(...).

healthcheck trả về phong bì kết quả versioned để phục vụ audit/CI wiring.

consume_geo_model_api_v1 nhận payload theo hợp đồng GEO_MODEL_API_v1 và trả về phong bì kết quả fail-safe ở chế độ STUB_MODE (không render/không IO).

4. Non-goals

Không thực hiện logic tổng hợp báo cáo thực tế (không layout 6 phần, không render PDF/HTML).

Không thực hiện IO (file write / network) và không tạo bất kỳ asset tạm (PNG/SVG/Mesh).

Không mutate SSOT và không sửa đổi Defect.

Không thay thế các module reporter thực thi (report_generator, delivery layer).

5. Naming & Identifiers (Normalized)

1. Referenced Contracts

GEO_MODEL_API_v1 (payload type: GeoModelResponse).

REPORT_COMPOSITION_RESULT_v1 (canonical result envelope for report composition stage; this stub returns it with STUB_MODE).

7. Inputs / Preconditions

7.1 healthcheck(run_id: str)

Input: run_id (bắt buộc; format YYYYMMDDThhmmssZ).

Preconditions: Không có; stub phải luôn chạy được, không phụ thuộc OCC/mesh/AFR.

7.2 consume_geo_model_api_v1(payload: GeoModelResponse)

Input: payload theo GEO_MODEL_API_v1 (tối thiểu có run_id, geometry_version_id; defects có thể rỗng).

Preconditions: payload đã được upstream validators/healer/freeze gate xử lý. Stub không làm validation sâu để tránh trùng trách nhiệm.

8. Outputs (Formalized)

8.1 REPORTER_HEALTHCHECK_RESULT_v0

status: 'OK' | 'DEGRADED' (stub thường 'OK').

run_id: string (required).

module_name: 'report_module_stub'.

algorithm_version: 'v0'.

issues: list[structured_issue] (có thể rỗng).

8.2 REPORT_COMPOSITION_RESULT_v1 (stub-mode)

status: 'STUB_MODE' hoặc 'NOT_EXECUTED'.

run_id: string (from payload).

geometry_version_id: string (from payload).

rule_version: string ('UNKNOWN' nếu payload thiếu; kèm issue MISSING_RULE_VERSION).

report_model: null hoặc skeleton tối thiểu (không render, không IO).

issues: gồm tối thiểu issue code 'REPORTER_STUB_ACTIVE'.

degraded_mode: true.

9. Step-by-step Logic

Define class ReporterModule (CamelCase).

Implement healthcheck(run_id): validate format run_id ở mức regex đơn giản; nếu sai, trả DEGENERATE với issue RUN_ID_INVALID (không raise).

Implement consume_geo_model_api_v1(payload): trích run_id, geometry_version_id, rule_version (nếu có).

Return REPORT_COMPOSITION_RESULT_v1 với status=STUB_MODE và issues=['REPORTER_STUB_ACTIVE', ...]. Không tạo file/asset, không gọi renderer.

10. Failure Modes & Fail-safe

Tuyệt đối không ném exception ra ngoài; mọi lỗi được quy đổi thành envelope status='FAILED' với issue code có cấu trúc.

Không được trả {} rỗng vì gây 'silent success'.

Nếu payload thiếu field bắt buộc tối thiểu (run_id/geometry_version_id), trả status='FAILED' và issue 'PAYLOAD_MIN_FIELDS_MISSING'.

11. Determinism & Complexity

Determinism: Tất định (pure stub), phụ thuộc duy nhất vào payload.

Time: O(1). Memory: O(1).

12. Governance Notes (Time-bomb Scaffold)

Stub này là “time-bomb scaffold”: orchestrator/CI phải có cơ chế chặn sử dụng stub ở chế độ production (ví dụ: policy key thresholds.validation.reporter_stub_allowed_until_utc hoặc max_sprints). Nếu vượt ngưỡng, phải FAIL-FAST ở step level với mã lỗi REPORTER_STUB_FORBIDDEN.

Field | Value

doc_id | ADN_report_module_stub_reporter_scaffold_and_healthcheck_stub_v1

status | REVIEW

effective_date_utc | [CHƯA XÁC MINH]

source_basis | Updated from approved facts-only extraction; normalized for compliance + DFM-first.

module_name | report_module_stub

package | tooling_dfm_advisor.reporter

file_name | report_module_stub.py

primary_class | ReporterModule

module_type | reporter_stub_scaffold

algorithm_name | reporter_scaffold_and_healthcheck_stub

algorithm_version | v0

pipeline_stage_support | Step 7 (Report Composition) + Step 8 (Audit/UI integration scaffold)

Field | Value

normalized_file_name | report_module_stub.py

normalized_module_name | report_module_stub

normalized_class_name | ReporterModule

result_envelopes | REPORTER_HEALTHCHECK_RESULT_v0; REPORT_COMPOSITION_RESULT_v1 (status=STUB_MODE/NOT_EXECUTED)

run_id_format | YYYYMMDDThhmmssZ (ISO 8601 rút gọn)

## 3.2 AMBR (Bindings / Module Contract / Execution)

### AMBR - report_generator

Source: S2

Verbatim extract (governance-only formatting):

AMBR - report_generator

Module binding and runtime constraints (v1)

Document Metadata

Module Responsibilities

Consume GeoModelResponse (GEO_MODEL_API_v1) read-only and compose a canonical report model.

Emit REPORT_COMPOSITION_RESULT_v1 without crashing the pipeline.

Do not render PDF/HTML; do not persist artifacts; no filesystem IO.

Dependencies

Internal contracts

tooling_dfm_advisor.contracts.geo_model_api_v1 (GeoModelResponse, Defect, RegionRef)

External libraries

None required beyond Python stdlib (typing/dataclasses if used).

Execution Context

Pipeline stage: Step 7 (report_composition_and_delivery) - composition sub-step.

SSOT permissions: read_only=true; may_mutate_tooling_model=false.

Determinism: strict; no nondeterministic ordering.

Fail-safe and Error Taxonomy

The module MUST not raise unhandled exceptions. All errors are reported via REPORT_COMPOSITION_RESULT_v1.issues with structured error codes.

Minimum error codes (normative)

REPORT_INPUT_INVALID

REPORT_SEVERITY_INVALID

REPORT_DEFECT_ID_MISSING

REPORT_ASSET_REF_FORBIDDEN

Compliance Constraints

Final delivery formats are restricted to PDF or HTML only; any attempt to deliver other formats is FAIL-FAST at delivery stage (REPORT_FORMAT_FORBIDDEN).

Report metadata MUST include run_id, rule_version, geometry_version_id.

No file paths/pointers in outputs; only hash-only or opaque asset IDs for ephemeral visuals.

Test Obligations (DFM-first)

Unit tests MUST cover: empty defects (summary_level=OK), mixed severities (priority), invalid severity mapping, missing defect_id surrogate generation, and input missing fields returning status=FAILED.

Regression tests SHOULD run on Golden Geometry Corpus and Expected Defect Set as part of CI; adding code without tests triggers FAIL-FAST (EMPTY_TEST_SUITE).

doc_id | AMBR_report_generator_v1

status | REVIEW

issue_date_utc | 2026-02-24T13:41:55Z

effective_date_utc | [CHUA XAC MINH]

package | tooling_dfm_advisor.reporter

file_name | report_generator.py

module_name | report_generator

module_type | reporter

execution_mode | SEQUENTIAL

bound_adn_id | ADN_report_generator_geomodelresponse_to_report_model_v1

output_envelope | REPORT_COMPOSITION_RESULT_v1

### AMBR (V1, REVIEW) - report_interfaces

Source: S3

Verbatim extract (governance-only formatting):

AMBR (V1, REVIEW) - report_interfaces

1. Bound algorithm record

1. Bound libraries and dependencies

External (stdlib):

typing.Protocol (and related typing helpers if used: runtime_checkable, TypedDict).

Internal (project contracts):

tooling_dfm_advisor.contracts.geo_model_api_v1 (GeoModelResponse, Defect).

3. Responsibility split

report_interfaces owns: protocol signature, typed payload/return constraints, status semantics, and versioning rules.

implementations own: transformation logic, validation guards, report_model content schema, and delivery mechanics (PDF/HTML only at delivery stage).

governance/orchestrator owns: receipts, tamper-evident audit logs, and ephemeral asset purge after delivery_complete.

4. Data flow

Input path: GeoModelResponse produced by upstream pipeline (GEO_MODEL_API_v1).

Output path: REPORT_COMPOSITION_RESULT_v1 consumed by reporter delivery layer / UI adapter.

5. Audit & traceability

Mandatory: run_id, geometry_version_id must propagate end-to-end.

Required output field: rule_version (UNKNOWN allowed only with issue MISSING_RULE_VERSION).

Module contains no receipts; however, it constrains metadata required for receipts downstream.

6. Change impact rule

Changing invoke signature or required envelope keys impacts all reporter implementations and their tests.

Breaking changes require protocol version bump and coordinated migration across reporter layer.

doc_id | TOOLING_DFM_ADVISOR_REPORT_INTERFACES_AMBR_v1

status | REVIEW

effective_date_utc | 2026-02-24

ambr_id | AMBR_report_interfaces_v1

package | tooling_dfm_advisor.reporter

file_name | report_interfaces.py

module_name | report_interfaces

module_type | reporter_interface_protocol

execution_mode | SEQUENTIAL (design-time typing; optional runtime_checkable)

adn_id | ADN_report_interfaces_reporter_contract_protocol_definition_v1

algorithm_name | reporter_contract_protocol_definition

algorithm_version | v1

pipeline_stage | Step 7 - Report Composition & Delivery

### AMBR V1 — report_module_stub (Module Binding & Runtime Constraints)

Source: S5

Verbatim extract (governance-only formatting):

AMBR V1 — report_module_stub (Module Binding & Runtime Constraints)

1. Document Metadata

1. Bound Algorithm

1. Bound Libraries

External libs: none.

Internal modules (required): tooling_dfm_advisor.contracts.geo_model_api_v1 (GeoModelResponse typing); optional import of report_interfaces for protocol consistency.

4. Execution Context

Role: Reporter scaffold for integration + healthcheck wiring.

Lifecycle: Temporary only; must be replaced by production reporter implementation.

5. Responsibility Split

report_module_stub owns: class scaffold + healthcheck + consume placeholder; returns compliant envelopes.

report_generator / delivery layer own: report composition logic, 6-part layout, PDF/HTML rendering, asset management and purge, and report validity signals.

6. Data Flow

Input: GeoModelResponse (GEO_MODEL_API_v1).

Output: REPORT_COMPOSITION_RESULT_v1 with status STUB_MODE/NOT_EXECUTED (no artifacts).

SSOT interaction: forbidden (no mutation).

7. Constraints & Forbidden Operations

Forbidden: any file/network IO, any image/mesh generation, any OCC/geometry backend calls.

Forbidden: returning raw {} (must return versioned envelopes).

8. Audit & Traceability

healthcheck must accept run_id and echo it in REPORTER_HEALTHCHECK_RESULT_v0.

consume must propagate run_id + geometry_version_id and mark degraded_mode=true.

Orchestrator must create receipt JSON for steps that invoke this stub; stub should not attempt to write receipts itself.

9. Change Impact Rules

When replacing stub with production module: update bindings to report_interfaces/report_generator contracts and enforce PDF/HTML-only.

Add regression tests using Golden Geometry Corpus; CI must fail on EMPTY_TEST_SUITE if tests are missing.

Remove or disable REPORTER_STUB_ACTIVE pathway; enforce REPORTER_STUB_FORBIDDEN in production lanes.

Field | Value

ambr_id | AMBR_report_module_stub_v1

status | REVIEW

package | tooling_dfm_advisor.reporter

file_name | report_module_stub.py

module_name | report_module_stub

module_type | reporter_stub_scaffold

execution_mode | SEQUENTIAL

Field | Value

adn_id | ADN_report_module_stub_reporter_scaffold_and_healthcheck_stub_v1

algorithm_name | reporter_scaffold_and_healthcheck_stub

algorithm_version | v0

## 3.3 LDR (Dependencies / Libraries / Runtime constraints)

### LDR V1 — report_module_stub (Library Dependency Record)

Source: S6

Verbatim extract (governance-only formatting):

LDR V1 — report_module_stub (Library Dependency Record)

1. Document Metadata

1. External Libraries

None. This module must not import external libraries.

3. Internal Dependencies

tooling_dfm_advisor.contracts.geo_model_api_v1 (type binding for payload).

tooling_dfm_advisor.reporter.report_interfaces (optional; protocol consistency).

4. Allowed Usage

Typing/Protocols only; basic string/regex validation for run_id.

Return compliant envelopes only (REPORTER_HEALTHCHECK_RESULT_v0 and REPORT_COMPOSITION_RESULT_v1).

5. Forbidden Usage

Any IO (file/network), any asset generation (PNG/SVG/Mesh), any OCC/geometry/mesh backend calls.

Any dependency that would pull heavy C++ runtimes (pythonocc-core) into reporter scaffold.

6. Risks & Mitigations

Risk: false green if stub appears successful. Mitigation: always emit issue REPORTER_STUB_ACTIVE and set degraded_mode=true in output.

Risk: stub persists too long. Mitigation: time-bomb gate enforced by policy + CI/orchestrator (REPORTER_STUB_FORBIDDEN).

7. Upgrade Policy

When production reporter is implemented, this LDR must be revised to include the renderer/exporter dependencies explicitly.

Any new dependency must be justified with purpose, version pin, and failure-mode/fail-safe plan.

Field | Value

ldr_id | LDR_report_module_stub_v1

status | REVIEW

language_runtime | python

used_by_modules | report_module_stub

### LDR - report_generator

Source: S8

Verbatim extract (governance-only formatting):

LDR - report_generator

Library dependency record (v1)

Document Metadata

Purpose

Bind reporter composition to a versioned internal contract to prevent schema drift and to keep traceability (run_id, geometry_version_id) stable across the pipeline.

Allowed Usage

Read-only access to GeoModelResponse and Defect fields required for report composition.

Use RegionRef as a stable, path-free reference for traceability.

Forbidden Usage

No mutation of GeoModelResponse/Defect instances.

No introduction of file paths/pointers into any output derived from contract fields.

Risks and Upgrade Policy

Breaking changes to GEO_MODEL_API_v1 (field rename, enum changes) REQUIRE contract version bump (GEO_MODEL_API_v2) and explicit migration notes.

Reporter tests MUST be updated alongside any contract upgrade to avoid runtime failures.

doc_id | LDR_internal_contract_geo_model_api_v1_for_report_generator_v1

status | REVIEW

issue_date_utc | 2026-02-24T13:41:55Z

effective_date_utc | [CHUA XAC MINH]

library_name | tooling_dfm_advisor.contracts.geo_model_api_v1

library_version | [CHUA XAC MINH]

language_runtime | python

used_by_modules | report_generator

referenced_adn_id | ADN_report_generator_geomodelresponse_to_report_model_v1

### LDR (V1, REVIEW) - report_interfaces - dependency records

Source: S9

Verbatim extract (governance-only formatting):

LDR (V1, REVIEW) - report_interfaces - dependency records

1. Dependency: typing (stdlib)

Purpose:

Use Protocol for structural typing of reporter consumer interfaces.

Optionally use runtime_checkable and TypedDict to tighten schema at type-check level.

Allowed usage:

Protocol definitions and method signatures only.

No runtime business logic in this module.

Forbidden usage:

No filesystem IO, no rendering, no geometry backend imports inside interface module.

Upgrade policy:

If signature or required fields change, bump protocol version and update dependent modules/tests.

2. Dependency: tooling_dfm_advisor.contracts.geo_model_api_v1 (internal contract)

Purpose:

Provide typed payload GeoModelResponse/Defect for reporter layer consumers to avoid schema drift.

Constraints & risks:

If GEO_MODEL_API_v1 evolves without version bump, reporter implementations may break at runtime.

If rule_version is missing upstream, reporter must still surface it as UNKNOWN with an issue code.

Alternatives considered:

Using untyped dict payloads was rejected due to weak compile-time checks and higher drift risk.

doc_id | TOOLING_DFM_ADVISOR_REPORT_INTERFACES_LDR_v1

status | REVIEW

effective_date_utc | 2026-02-24

used_by_module | tooling_dfm_advisor.reporter.report_interfaces

ldr_id | LDR_typing_for_report_interfaces_v1

library_name | typing

library_version | python_stdlib

language_runtime | python

ldr_id | LDR_internal_contract_geo_model_api_v1_for_report_interfaces_v1

library_name | tooling_dfm_advisor.contracts.geo_model_api_v1

library_version | [CHƯA XÁC MINH]

language_runtime | python

## 4. Open Questions / [CHƯA XÁC MINH]

The following [CHƯA XÁC MINH] items appear in source documents and remain unresolved:

- (S1) effective_date_utc | [CHUA XAC MINH]

- (S2) effective_date_utc | [CHUA XAC MINH]

- (S7) effective_date_utc | [CHƯA XÁC MINH]

- (S8) effective_date_utc | [CHUA XAC MINH]

- (S8) library_version | [CHUA XAC MINH]

- (S9) library_version | [CHƯA XÁC MINH]

## 5. Change Log (Editorial)

Editorial changes in this canonical document are limited to: renaming to canonical code, adding control block, adding mapping table, and reorganizing content into the required SPC structure (ADN/AMBR/LDR). No technical semantics were changed.

Normalization notes: Source texts are included as verbatim extracts with minimal heading adjustments and explicit traceability markers (Source: Sx).

## 6. Appendix (Optional)

Cross references (canonical codes): WS_MODEL-GEN-SPC-v1 (GEO_MODEL_API_v1 / models), WS_MODEL-RGN-POL-v1 (REGION_REF_CONVENTION_v1).

Name conventions reminder: module implementation names are snake_case; contracts/envelopes are UPPER_CASE versioned. If a source uses alternate naming, treat it as 'aka' and do not change semantics.
