# 0. Document Control Block

Canonical Code: WS_GOV-ALL-POL-v1
Status: REVIEW
Run ID: 20260228T082540Z
WS / Module / Doc Type / Version: WS_GOV / ALL / POL / v1
Created date (UTC): 2026-02-28 08:25:40Z
Purpose: Document governance consolidation only; no runtime/pipeline changes.
System name: tooling_dfm_advisor
Authority: Document Governance Layer (WS naming + lifecycle).

## 1. Scope

Consolidate and normalize governance policy artifacts for WS_GOV (module ALL) into a single POL document. This POL focuses on policy-as-config keys, invariants, naming/directory constraints, and upstream integrity enforcement rules.

Non-goals: No change to technical semantics, algorithms, or architecture; no new policy rules beyond sources; no code/pipeline implementation; no decision_map expansion beyond what sources provide.

## 2. Included Sources & Mapping Table

|           |                                                                       |                                                                               |                                         |                                                                                                                                                 |                                                                                |
| --------- | --------------------------------------------------------------------- | ----------------------------------------------------------------------------- | --------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| source_id | filename                                                              | original_title                                                                | original_version/status                 | sections_used                                                                                                                                   | notes                                                                          |
| S1        | TOOLING_DFM_ADVISOR_PROCESS_AND_COMMAND_MASTER_v1_PATCHED_MIN_v1.docx | TOOLING_DFM_ADVISOR_PROCESS_AND_COMMAND_MASTER_v1 (Master process & commands) | APPROVED (as stated in source)          | Metadata (naming/directory refs); governance invariants; anti-loop + hard-stop conditions                                                       | Templates/prompts excluded as out-of-scope for POL (see Excluded / Non-goals). |
| S2        | UPSTREAM_INTEGRITY_CONTRACT_v2_0_FROZEN.docx                          | UPSTREAM_INTEGRITY_CONTRACT_v2_0                                              | FROZEN                                  | Why; Required Artifacts; Missing Behavior; Audit Rule; Fail-Fast; Cross References                                                              | Included verbatim in Appendix as reference contract.                           |
| S3        | GOVERNANCE_POLICY_SCHEMA_v1.docx                                      | GOVERNANCE_POLICY_SCHEMA_v1 (policy-as-config schema snapshot)                | [CHƯA XÁC MINH] (source is schema text) | schema_version/policy_version/rule_version_default; dfm_policy keys; thresholds; severity_map; retry/ttl placeholders; decision_map placeholder | Included verbatim in Appendix as raw schema.                                   |

## 3. Consolidated Policy Content

## 3.1 Policy identity and versioning

*Source: S3*

This document consolidates the policy schema snapshot as provided in the source. Key identity fields are preserved verbatim via Appendix A.

Known fields in source: schema_version, policy_version, rule_version_default, dfm_policy.dfm_policy_version, region_ref_convention_id.

## 3.2 Governance invariants, naming, and directory constraints

This subsection extracts governance invariants and non-bypassable naming/directory rules used by the system governance layer.

### 3.2.1 Naming and directory references (verbatim extract)

*Source: S1*

TÀI LIỆU MASTER: QUY TRÌNH & BỘ LỆNH ĐIỀU KHIỂN HỆ THỐNG tooling_dfm_advisor

Tài liệu này là bản xuất bản đầy đủ, đã chuẩn hóa theo cùng phong cách của các MASTER SPEC trước đó và tuân thủ các chuẩn Naming/Directory/Governance Invariants: dùng tooling_dfm_advisor (snake_case), run_id ISO 8601 rút gọn, step_id UPPER_CASE, decision_code closed-set, artefacts versioned (…\_v1), receipts tamper-evident, pause/lock non-bypassable, và templates có fail-fast rõ ràng.

THÔNG TIN PHIÊN BẢN (DOCUMENT METADATA)

doc_id: TOOLING_DFM_ADVISOR_PROCESS_AND_COMMAND_MASTER_v1

status: APPROVED

effective_date_utc: [CHƯA XÁC MINH]

revision_utc: 2026-02-25

revision_note: Applied DFM_FIRST_MIN_PATCH_v1 (Naming/Directory/Report/UI/Ephemeral/Envelope) without changing domain logic.

rule_version: [CHƯA XÁC MINH] (version of governance_policy.json / DFM ruleset used for runs)

owner: System Owner / PM

related_specs:

TOOLING_DFM_ADVISOR_MASTER_SPEC_v1

TOOLING_DFM_ADVISOR_GOVERNANCE_MASTER_SPEC_v1

naming_reference:

system_name: tooling_dfm_advisor

run_id_format: YYYYMMDDThhmmssZ

step_id_format: UPPER_CASE (registry-defined)

decision_code_format: UPPER_CASE (closed-set, policy-mapped)

result_envelope_format: \*\_RESULT_v0 (UPPER_CASE; ví dụ: THICKNESS_ANALYSIS_RESULT_v0; status ∈ {OK,FAILED,NOT_EXECUTED,STUB_MODE})

contract_api_format: \*\_API_v[X] (ví dụ: TOOLING_MODEL_API_v1, GEO_MODEL_API_v1, STEP_IMPORT_API_v1)

directory_reference: Mandatory layout (canonical, non-bypassable) — xem chi tiết ngay dưới:

config/: Policy-as-Config (bắt buộc)

- governance_policy.json

docs/runbooks/: Runbooks + step registry (bắt buộc)

- step_registry.json

- GENESIS_KIT/

docs/audit/: Append-only audit anchor (bắt buộc)

- SESSION_AUDIT_LOG.md

- SESSION_STATE.json

- SESSION_STATE.lock

docs/audit/receipts/: Receipts + logs (bắt buộc)

- \*.json (receipt)

- \*.stdout.log, \*.stderr.log (hoặc /logs/\*.stdout.log, /logs/\*.stderr.log)

tooling_dfm_advisor/: Source root (bắt buộc)

- geometry/

- contracts/

- agents/

- reporter/

### 3.2.2 Invariants (verbatim extract)

*Source: S1*

3.1. Ba nguyên tắc bất biến (Invariants)

No-assumption: không suy diễn; thiếu dữ liệu phải gắn [CHƯA RÕ TRONG TÀI LIỆU GỐC] và hỏi/đòi evidence.

ONE_STEP_ONLY: mỗi vòng lặp chỉ tạo và thực thi đúng 01 step.

Evidence-driven: mọi claim phải có evidence (receipt/log/test output/diff).

### 3.2.3 Anti-loop and hard-stop conditions (verbatim extract)

*Source: S1*

5.1. Anti-loop vận hành

Lỗi 2 lần liên tiếp cùng một pattern: bắt buộc đổi chiến lược; cấm lặp lệnh cũ nếu không có bằng chứng thay đổi.

Retry chỉ hợp lệ khi receipt mới pass validate-receipt và có progress (NO_PROGRESS guard không kích hoạt).

Vượt max_retry_per_step: chuyển PAUSE_FOR_AUTHORITY_DECISION.

5.2. Hard-stop conditions (bắt buộc dừng)

Unknown step_id: dừng cứng, PAUSE_FOR_AUTHORITY_DECISION, audit UNKNOWN_STEP_ID.

Missing dependency: không đoán cách cài; trả NOT_EXECUTED/STUB_MODE hoặc yêu cầu cài đặt kèm cách kiểm chứng.

Spec mơ hồ: dừng và hỏi; gắn [CHƯA RÕ TRONG TÀI LIỆU GỐC].

Test fail: cô lập nguyên nhân, bổ sung regression test (golden corpus nếu thuộc DFM logic).

5.3. Rubric đánh giá (PASS/FAIL basis)

Đánh giá dựa trên:

Correctness (đúng yêu cầu).

Robustness (không dễ gãy).

Determinism (tất định).

Observability (có log/receipt rõ).

Traceability (run_id, step_id, hashes).

Clarity (facts-only; không ambiguity).

## 3.3 Upstream integrity enforcement policy

Defines required upstream artifacts and execution blocking rules for downstream agents.

*Source: S2*

UPSTREAM_INTEGRITY_CONTRACT_v2_0_FROZEN

doc_id: UPSTREAM_INTEGRITY_CONTRACT_v2_0

status: FROZEN

system_name: tooling_dfm_advisor

architecture_level: INDUSTRIAL_GRADE_FREEZE

version: V2_0

freeze_date_utc: 2026-02-27T13:59:27Z

Why

Guarantee agents execute only with validated upstream artifacts.

Required Artifacts

afr_state, mapping_result, geometry_version_id.

Missing Behavior

Required missing → NOT_EXECUTED. Optional missing → confidence downgrade.

Audit Rule

Upstream verification logged per run.

Fail-Fast

Missing required artifact blocks execution.

Cross Reference

Integrates with ANALYSIS_RESULT_CONTRACT_v2_0.

## 3.4 DFM policy keys, thresholds, and severity maps

The DFM policy key catalog and severity maps are taken from the schema snapshot. For readability, the canonical source is included verbatim in Appendix A; this section provides a structured index of the major groups.

*Source: S3*

|                         |                                             |                                                                                         |
| ----------------------- | ------------------------------------------- | --------------------------------------------------------------------------------------- |
| Group                   | Key path                                    | Notes (as in source)                                                                    |
| Units                   | dfm_policy.units.length_unit / angle_unit   | length_unit='mm', angle_unit='deg'                                                      |
| Common budgets          | dfm_policy.common.sampling_budget.\*        | max_faces_to_report=50; max_worst_regions=10                                            |
| Thresholds              | dfm_policy.thresholds.{draft                | thickness                                                                               |
| Severity combine        | dfm_policy.severity_map.combine_strategy.\* | method='max_then_overrides'; defaults for policy_unresolved/unit_unresolved/occ_missing |
| Per-agent severity maps | dfm_policy.severity_map.{thickness          | draft                                                                                   |
| Decision map            | decision_map                                | Placeholder '\_\_TODO\_\_' in schema snapshot                                           |
| Retry/lock              | max_retry_per_step / lock_ttl_seconds       | null in schema snapshot                                                                 |

## 4. Open Questions / [CHƯA XÁC MINH]

|                                          |                                                                                                     |                                                                              |
| ---------------------------------------- | --------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| Item                                     | Question / missing data                                                                             | Impact                                                                       |
| S1.effective_date_utc                    | Source S1 marks effective_date_utc as [CHƯA XÁC MINH].                                              | Blocks establishing formal effective date for governance rules in documents. |
| S1.rule_version                          | Source S1 marks rule_version as [CHƯA XÁC MINH].                                                    | Blocks unambiguous linkage between receipts and the active ruleset.          |
| S3.decision_map                          | Schema snapshot contains a decision_map placeholder ('\_\_TODO\_\_').                               | Blocks completing governance decision_code closed-set mapping.               |
| S3.max_retry_per_step / lock_ttl_seconds | Schema snapshot sets these fields as null.                                                          | Blocks enforcement of retry/lock TTL parameters unless defined elsewhere.    |
| Cross-doc linkage                        | Confirm canonical code(s) of referenced contracts: ANALYSIS_RESULT_CONTRACT_v2_0 (mentioned in S2). | Blocks consistent cross-reference list inside WS_GOV registry.               |

## 5. Change Log (Editorial)

Editorial consolidation only: renamed and consolidated multiple governance policy sources into one canonical POL document; no semantics changed.

Terminology normalization: kept original terms; where ambiguity exists, preserved source wording and added no new definitions.

## 6. Appendix

## Appendix A - Raw GOVERNANCE_POLICY_SCHEMA_v1 (verbatim)

*Source: S3*

{

"schema_version": "GOVERNANCE_POLICY_SCHEMA_v1",

"policy_version": "GOV_POLICY_v1",

"rule_version_default": "RULESET_v1",

"dfm_policy": {

"dfm_policy_version": "DFM_POLICY_v1",

"region_ref_convention_id": "REGION_REF_CONVENTION_v1",

"units": {

"length_unit": "mm",

"angle_unit": "deg"

},

"common": {

"silhouette_eps_dot": null,

"sampling_budget": {

"max_faces_to_report": 50,

"max_worst_regions": 10

}

},

"thresholds": {

"draft": {

"min_draft_deg_default": null,

"min_draft_deg_by_surface_class": {

"polish": null,

"light_texture": null,

"heavy_texture": null

},

"reverse_dot_threshold": 0.0

},

"thickness": {

"min_wall_mm": null,

"max_wall_mm": null,

"probe": {

"self_hit_epsilon_mm": null,

"max_probe_distance_mm": null,

"max_intersections_to_consider": 8

}

},

"undercut": {

"reverse_dot_threshold": 0.0,

"planar_only": null

},

"ribratio": {

"max_rib_to_wall_ratio": null,

"preferred_rib_to_wall_ratio": null,

"min_root_radius_mm": null

},

"sink": {

"max_sink_risk_proxy": null,

"local_mass": {

"max_local_thickness_ratio": null,

"max_boss_to_wall_ratio": null

}

},

"weldline": {

"max_weld_risk_proxy": null,

"min_face_count_for_valid_proxy": 20

}

},

"severity_map": {

"combine_strategy": {

"method": "max_then_overrides",

"default_issue_severity": {

"policy_unresolved": "WARN",

"unit_unresolved": "ERROR",

"occ_missing": "WARN"

}

},

"thickness": {

"thin_wall": {

"deficit_ratio_thresholds": { "warn_min": 0.05, "error_min": 0.15, "fatal_min": 0.30 },

"coverage_ratio_thresholds": { "warn_min": 0.01, "error_min": 0.05, "fatal_min": 0.15 },

"rules": \[

{ "if_deficit_ratio_at_least": 0.30, "severity": "FATAL" },

{ "if_deficit_ratio_at_least": 0.15, "severity": "ERROR" },

{ "if_deficit_ratio_at_least": 0.05, "severity": "WARN" },

{ "else": "INFO" }

\]

},

"thick_wall": {

"excess_ratio_thresholds": { "warn_min": 0.10, "error_min": 0.25, "fatal_min": 0.50 },

"coverage_ratio_thresholds": { "warn_min": 0.01, "error_min": 0.05, "fatal_min": 0.15 },

"rules": \[

{ "if_excess_ratio_at_least": 0.50, "severity": "FATAL" },

{ "if_excess_ratio_at_least": 0.25, "severity": "ERROR" },

{ "if_excess_ratio_at_least": 0.10, "severity": "WARN" },

{ "else": "INFO" }

\]

},

"map_quality": {

"miss_rate_thresholds": { "warn_min": 0.10, "error_min": 0.25, "fatal_min": 0.50 },

"ambiguous_hit_rate_thresholds": { "warn_min": 0.10, "error_min": 0.25, "fatal_min": 0.50 },

"sample_count_min": { "warn_min": 200, "error_min": 80, "fatal_min": 30 },

"rules": \[

{ "if_miss_rate_at_least": 0.50, "severity": "FATAL", "code": "THICKNESS_MAP_LOW_COVERAGE" },

{ "if_miss_rate_at_least": 0.25, "severity": "ERROR", "code": "THICKNESS_MAP_LOW_COVERAGE" },

{ "if_miss_rate_at_least": 0.10, "severity": "WARN", "code": "THICKNESS_MAP_LOW_COVERAGE" },

{ "if_sample_count_below": 30, "severity": "FATAL", "code": "THICKNESS_MAP_INSUFFICIENT_SAMPLES" },

{ "if_sample_count_below": 80, "severity": "ERROR", "code": "THICKNESS_MAP_INSUFFICIENT_SAMPLES" },

{ "if_sample_count_below": 200, "severity": "WARN", "code": "THICKNESS_MAP_INSUFFICIENT_SAMPLES" }

\]

}

},

"draft": {

"below_threshold": {

"deficit_deg_thresholds": { "warn_min": 0.5, "error_min": 1.5, "fatal_min": 3.0 },

"coverage_ratio_thresholds": { "warn_min": 0.01, "error_min": 0.05, "fatal_min": 0.15 },

"rules": \[

{ "if_deficit_deg_at_least": 3.0, "severity": "FATAL" },

{ "if_deficit_deg_at_least": 1.5, "severity": "ERROR" },

{ "if_deficit_deg_at_least": 0.5, "severity": "WARN" },

{ "else": "INFO" }

\]

},

"reverse_draft": {

"reverse_ratio_thresholds": { "warn_min": 0.001, "error_min": 0.01, "fatal_min": 0.05 },

"rules": \[

{ "if_reverse_ratio_at_least": 0.05, "severity": "FATAL" },

{ "if_reverse_ratio_at_least": 0.01, "severity": "ERROR" },

{ "if_reverse_present": true, "severity": "WARN" },

{ "else": "INFO" }

\]

},

"silhouette_crossing": {

"face_ratio_thresholds": { "warn_min": 0.01, "error_min": 0.05, "fatal_min": 0.15 },

"rules": \[

{ "if_face_ratio_at_least": 0.15, "severity": "FATAL" },

{ "if_face_ratio_at_least": 0.05, "severity": "ERROR" },

{ "if_silhouette_present": true, "severity": "WARN" },

{ "else": "INFO" }

\]

},

"overrides": \[

{

"when": { "reverse_present": true, "below_threshold_deficit_deg_at_least": 1.5 },

"force_severity": "FATAL"

}

\]

},

"undercut": {

"candidate": {

"depth_thresholds": { "warn_min": 0.01, "error_min": 0.10, "fatal_min": 0.30 },

"coverage_ratio_thresholds": { "warn_min": 0.001, "error_min": 0.01, "fatal_min": 0.05 },

"rules": \[

{ "if_depth_at_least": 0.30, "severity": "FATAL" },

{ "if_depth_at_least": 0.10, "severity": "ERROR" },

{ "if_depth_at_least": 0.01, "severity": "WARN" },

{ "else": "INFO" }

\]

},

"continuity_overrides": \[

{

"when": { "largest_cluster_ratio_at_least": 0.70, "coverage_ratio_at_least": 0.05 },

"force_severity": "FATAL"

},

{

"when": { "largest_cluster_ratio_at_least": 0.50, "coverage_ratio_at_least": 0.01 },

"force_severity": "ERROR"

}

\],

"confidence": {

"planar_face_ratio_thresholds": { "warn_below": 0.60, "error_below": 0.30 },

"rules": \[

{ "if_planar_face_ratio_below": 0.30, "severity": "ERROR", "code": "UNDERCUT_LOW_COVERAGE_NONPLANAR" },

{ "if_planar_face_ratio_below": 0.60, "severity": "WARN", "code": "UNDERCUT_LOW_COVERAGE_NONPLANAR" }

\]

}

},

"ribratio": {

"rib_to_wall_ratio": {

"over_ratio_thresholds": { "warn_min": 0.10, "error_min": 0.25, "fatal_min": 0.50 },

"coverage_ratio_thresholds": { "warn_min": 0.01, "error_min": 0.05, "fatal_min": 0.15 },

"rules": \[

{ "if_over_ratio_at_least": 0.50, "severity": "FATAL" },

{ "if_over_ratio_at_least": 0.25, "severity": "ERROR" },

{ "if_over_ratio_at_least": 0.10, "severity": "WARN" },

{ "else": "INFO" }

\]

},

"proxy_mode": {

"rules": \[

{ "if_proxy_used": true, "severity": "WARN", "code": "RIBRATIO_PROXY_ONLY_LOW_CONFIDENCE" }

\]

}

},

"sink": {

"proxy": {

"proxy_thresholds": { "warn_min": 0.30, "error_min": 0.55, "fatal_min": 0.75 },

"rules": \[

{ "if_proxy_at_least": 0.75, "severity": "FATAL" },

{ "if_proxy_at_least": 0.55, "severity": "ERROR" },

{ "if_proxy_at_least": 0.30, "severity": "WARN" },

{ "else": "INFO" }

\]

},

"local_mass": {

"thick_ratio_thresholds": { "warn_min": 0.20, "error_min": 0.50, "fatal_min": 1.00 },

"coverage_ratio_thresholds": { "warn_min": 0.01, "error_min": 0.05, "fatal_min": 0.15 },

"rules": \[

{ "if_thick_ratio_at_least": 1.00, "severity": "FATAL" },

{ "if_thick_ratio_at_least": 0.50, "severity": "ERROR" },

{ "if_thick_ratio_at_least": 0.20, "severity": "WARN" },

{ "else": "INFO" }

\]

}

},

"weldline": {

"proxy": {

"over_ratio_thresholds": { "warn_min": 0.10, "error_min": 0.25, "fatal_min": 0.50 },

"rules": \[

{ "if_face_count_below": 20, "severity": "WARN", "code": "WELDLINE_PROXY_LOW_VALIDITY_FACECOUNT" },

{ "if_over_ratio_at_least": 0.50, "severity": "FATAL" },

{ "if_over_ratio_at_least": 0.25, "severity": "ERROR" },

{ "if_over_ratio_at_least": 0.10, "severity": "WARN" },

{ "else": "INFO" }

\]

}

}

}

},

"decision_map": {

"\_\_TODO\_\_": "\_\_OUT_OF_SCOPE_FOR_THIS_PACKAGING\_\_"

},

"max_retry_per_step": null,

"lock_ttl_seconds": null

}

## Appendix B - UPSTREAM_INTEGRITY_CONTRACT_v2_0 (verbatim)

*Source: S2*

UPSTREAM_INTEGRITY_CONTRACT_v2_0_FROZEN

doc_id: UPSTREAM_INTEGRITY_CONTRACT_v2_0

status: FROZEN

system_name: tooling_dfm_advisor

architecture_level: INDUSTRIAL_GRADE_FREEZE

version: V2_0

freeze_date_utc: 2026-02-27T13:59:27Z

Why

Guarantee agents execute only with validated upstream artifacts.

Required Artifacts

afr_state, mapping_result, geometry_version_id.

Missing Behavior

Required missing → NOT_EXECUTED. Optional missing → confidence downgrade.

Audit Rule

Upstream verification logged per run.

Fail-Fast

Missing required artifact blocks execution.

Cross Reference

Integrates with ANALYSIS_RESULT_CONTRACT_v2_0.
