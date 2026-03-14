WS_CORE-VAL-POL-v2 - Geometry Validation Gate Policy

Canonical policy document (governance-only consolidation).

# 0. Document Control Block

|                                  |                                                                      |
| -------------------------------- | -------------------------------------------------------------------- |
| Canonical Code                   | WS_CORE-VAL-POL-v2                                                   |
| Status                           | REVIEW                                                               |
| Run ID                           | 20260304T072414Z                                                     |
| Created date (UTC)               | 2026-03-04T07:24:14Z                                                 |
| WS / Module / Doc Type / Version | WS_CORE / VAL / POL / v2                                             |
| Purpose                          | Document governance consolidation only; no runtime/pipeline changes. |
| Source doc_id                    | GEOMETRY_VALIDATION_GATE_v2_0                                        |
| Source status                    | FROZEN                                                               |
| Source freeze_date_utc           | 2026-02-27T13:59:27Z                                                 |

## 1. Scope

This document defines policy-level rules for the Geometry Validation Gate in WS_CORE (module VAL). It captures the minimum validation checks, failure modes, evidence requirements, and cross-reference constraints that must be applied before AGL freeze.

Non-goals: No algorithm redesign; no new checks beyond the source; no code or implementation guidance.

## 2. Included Sources & Mapping Table

|           |                                           |                                      |                             |                                                                                   |                                                                                                          |
| --------- | ----------------------------------------- | ------------------------------------ | --------------------------- | --------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| source_id | filename                                  | original_title                       | original_version/status     | sections_used                                                                     | notes                                                                                                    |
| S1        | GEOMETRY_VALIDATION_GATE_v2_0_FROZEN.docx | GEOMETRY_VALIDATION_GATE_v2_0_FROZEN | version=V2_0; status=FROZEN | Why; Minimum Checks; Failure Modes; Outputs; Evidence; Fail-Fast; Cross Reference | Mapped v2.0 -> v2 per Document Governance Layer. Source is labeled FROZEN; target status set by request. |

## 3. Consolidated Policy Content

## 3.1 Why

Source: S1

Prevent invalid geometry from entering freeze stage.

## 3.2 Minimum Checks

Source: S1

Non-manifold, zero volume, self intersection, degenerate bbox.

## 3.3 Failure Modes

Source: S1

HARD_FAIL or DEGRADED_MODE.

## 3.4 Outputs

Source: S1

Validation result with structured error codes.

## 3.5 Evidence

Source: S1

Validation receipt attached to run_id.

## 3.6 Fail-Fast

Source: S1

Critical geometry error blocks freeze.

## 3.7 Cross Reference

Source: S1

Required before AGL_LIFECYCLE_MODEL_v2_0 freeze.

## 4. Open Questions / [CHUA XAC MINH]

- Policy-as-config mapping: which governance_policy.json keys should represent these checks (e.g., thresholds.validation.\*), and what are their default values?

- Structured error-code taxonomy: the source states 'structured error codes' but does not enumerate codes or severity mapping.

- DEGRADED_MODE criteria: when is degraded_mode allowed vs HARD_FAIL, and does it permit AGL freeze or only report generation?

- Tolerance parameters for checks (self-intersection tolerances, zero-volume thresholds, bbox degeneracy thresholds) are not specified.

- Cross-reference target: AGL_LIFECYCLE_MODEL_v2_0 is referenced but not provided in this pack; required to finalize alignment.

## 5. Change Log (Editorial)

20260304T072414Z: Created canonical POL WS_CORE-VAL-POL-v2 by renaming and consolidating S1. Content preserved; only headings/metadata/mapping tables added.

Note: Source document is labeled status=FROZEN; this canonical output is produced with STATUS=REVIEW as requested for review workflow. Semantics are unchanged.

## 6. Appendix

## 6.1 Verbatim Source Extract (S1)

Source: S1

GEOMETRY_VALIDATION_GATE_v2_0_FROZEN

doc_id: GEOMETRY_VALIDATION_GATE_v2_0

status: FROZEN

system_name: tooling_dfm_advisor

architecture_level: INDUSTRIAL_GRADE_FREEZE

version: V2_0

freeze_date_utc: 2026-02-27T13:59:27Z

Why

Prevent invalid geometry from entering freeze stage.

Minimum Checks

Non-manifold, zero volume, self intersection, degenerate bbox.

Failure Modes

HARD_FAIL or DEGRADED_MODE.

Outputs

Validation result with structured error codes.

Evidence

Validation receipt attached to run_id.

Fail-Fast

Critical geometry error blocks freeze.

Cross Reference

Required before AGL_LIFECYCLE_MODEL_v2_0 freeze.
