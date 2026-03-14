**WS_CORE-GEN-POL-v2 (FROZEN)**

# 0. Document Control Block

|                                  |                                                                      |
| -------------------------------- | -------------------------------------------------------------------- |
| Canonical Code                   | WS_CORE-GEN-POL-v2                                                   |
| Status                           | FROZEN                                                               |
| Run ID                           | 20260304T073523Z                                                     |
| WS / Module / Doc Type / Version | WS_CORE / GEN / POL / v2                                             |
| Created date (UTC)               | 2026-03-04T07:35:23Z                                                 |
| Purpose                          | Document governance consolidation only; no runtime/pipeline changes. |
| Source doc_id                    | AGL_LIFECYCLE_MODEL_v2_0                                             |
| Source status                    | FROZEN                                                               |
| Source version                   | V2_0                                                                 |
| Source freeze_date_utc           | 2026-02-27T13:59:27Z                                                 |
| system_name                      | tooling_dfm_advisor                                                  |
| architecture_level               | INDUSTRIAL_GRADE_FREEZE                                              |

## 1. Scope

Phạm vi: Mô tả policy/convention về vòng đời AGL (geometry immutability + version traceability) dùng cho WS_CORE (GEN).

Source: S1 — Formalize geometry immutability and version traceability.

Non-goals: Không thay đổi thuật toán, không thiết kế lại pipeline, không bổ sung rule mới ngoài nội dung nguồn. Đây chỉ là chuẩn hoá tên + cấu trúc tài liệu.

## 2. Included Sources & Mapping Table

|           |                                      |                          |                         |                                                                                                  |                                                                         |
| --------- | ------------------------------------ | ------------------------ | ----------------------- | ------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------- |
| source_id | filename                             | original_title           | original_version/status | sections_used                                                                                    | notes                                                                   |
| S1        | AGL_LIFECYCLE_MODEL_v2_0_FROZEN.docx | AGL_LIFECYCLE_MODEL_v2_0 | V2_0 / FROZEN           | All sections (Why, States, Freeze Rule, Agent Access Rule, Evidence, Fail-Fast, Cross Reference) | Mapping: v2.0 -> v2; status kept as FROZEN; content preserved verbatim. |

## 3. Consolidated Policy Content

## 3.1 Purpose / Why

**Source: S1**

Why

Formalize geometry immutability and version traceability.

## 3.2 Lifecycle States

**Source: S1**

States

INIT → HEALED → VALIDATED → FROZEN (terminal).

## 3.3 Freeze Rule

**Source: S1**

Freeze Rule

Assign deterministic geometry_version_id based on canonical fingerprint.

## 3.4 Agent Access Rule

**Source: S1**

Agent Access Rule

Agents execute only when state == FROZEN.

## 3.5 Evidence Requirements

**Source: S1**

Evidence

Freeze receipt must include geometry hash.

## 3.6 Fail-Fast Rules

**Source: S1**

Fail-Fast

Mutation after freeze triggers governance violation.

## 3.7 Cross References

**Source: S1**

Cross Reference

Feeds ANALYSIS_RESULT_CONTRACT_v2_0 and GOLDEN_REGRESSION_STRATEGY_v2_0.

## 4. Open Questions / [CHƯA XÁC MINH]

[CHƯA XÁC MINH] Định nghĩa chính xác “canonical fingerprint” và thuật toán/field set dùng để tính geometry_version_id (để đảm bảo tính deterministic và stable giữa các environment).

[CHƯA XÁC MINH] Cấu trúc receipt tối thiểu cho ‘Freeze receipt’ (field names, schema version, nơi lưu geometry hash) ngoài câu mô tả chung trong nguồn.

[CHƯA XÁC MINH] Nếu cần trạng thái trung gian khác (ví dụ IMPORTED/NORMALIZED) trước INIT, hoặc thêm state cho DGL; hiện nguồn chỉ nêu INIT→HEALED→VALIDATED→FROZEN.

## 5. Change Log (Editorial)

- 20260304T073523Z: Packaging-only. Renamed to canonical code WS_CORE-GEN-POL-v2; inserted Document Control Block, Mapping Table, numbering. No semantics change; content preserved verbatim from S1.

## 6. Appendix

## 6.1 Verbatim Source Extract (S1)

Source: S1 (full text)

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
