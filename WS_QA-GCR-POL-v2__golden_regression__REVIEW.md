WS_QA-GCR-POL-v2

# 0. Document Control Block

|                                  |                                                                      |
| -------------------------------- | -------------------------------------------------------------------- |
| Canonical Code                   | WS_QA-GCR-POL-v2                                                     |
| Status                           | REVIEW                                                               |
| Run ID (UTC)                     | 20260304T114345Z                                                     |
| Created date (UTC)               | 20260304 114345                                                      |
| WS / Module / Doc Type / Version | WS=WS_QA; MODULE_3LA=GCR; DOC_TYPE=POL; VERSION=v2                   |
| Purpose                          | Document governance consolidation only; no runtime/pipeline changes. |

## 1. Scope

Tài liệu này chuẩn hoá và gom nội dung policy liên quan đến chiến lược Golden Regression cho tooling_dfm_advisor, nhằm đảm bảo tính determinism của DFM khi refactor. Nội dung được kế thừa trực tiếp từ nguồn; không bổ sung logic mới.

Non-goals:

Không thay đổi thuật toán, kiến trúc, hay pipeline runtime/CI.

Không thay đổi semantics của chiến lược regression; chỉ chuẩn hoá cấu trúc tài liệu và tên file.

## 2. Included Sources & Mapping Table

Bảng dưới đây liệt kê nguồn được dùng trong bản canonical này (traceability bắt buộc).

|           |                                             |                                 |                         |                                                                                                                       |                                  |
| --------- | ------------------------------------------- | ------------------------------- | ----------------------- | --------------------------------------------------------------------------------------------------------------------- | -------------------------------- |
| source_id | filename                                    | original_title                  | original_version/status | sections_used                                                                                                         | notes                            |
| S1        | GOLDEN_REGRESSION_STRATEGY_v2_0_FROZEN.docx | GOLDEN_REGRESSION_STRATEGY_v2_0 | FROZEN                  | Why; Corpus Structure; Binding Rule; Baseline Governance; CI Enforcement; Fail-Fast; Cross Reference; source metadata | verbatim extract; packaging only |

## 3. Consolidated Policy Content

## Why

Source: S1

Ensure DFM determinism across refactors.

## Corpus Structure

Source: S1

Each case includes STEP + expected_defects.json.

## Binding Rule

Source: S1

Regression bound to policy_version and geometry_version_id.

## Baseline Governance

Source: S1

Baseline change requires ADR and policy increment.

## CI Enforcement

Source: S1

Mismatch causes build failure.

## Fail-Fast

Source: S1

Hash mismatch stops pipeline immediately.

## Cross Reference

Source: S1

Depends on RESULT_CONTRACT_v2_0 and POLICY_v2_0.

## 4. Open Questions / [CHƯA XÁC MINH]

[CHƯA XÁC MINH] Định nghĩa/canonical code của RESULT_CONTRACT_v2_0 và POLICY_v2_0 được tham chiếu trong nguồn. Nếu thiếu, việc traceability giữa golden regression và result/policy contracts sẽ không hoàn chỉnh.

[CHƯA XÁC MINH] Quy ước hash/receipt nào là ‘source of truth’ cho expected_defects.json (format, schema, signer) trong corpus regression.

## 5. Change Log (Editorial)

Đổi tên và đóng gói lại tài liệu theo canonical code WS_QA-GCR-POL-v2 và STATUS=REVIEW.

Chuẩn hoá cấu trúc theo template POL (Control Block, Mapping Table, Consolidated Content, Open Questions, Change Log).

Không thay đổi semantics; nội dung policy được giữ theo verbatim extracts từ nguồn.

## 6. Appendix

Source metadata (verbatim):

Source: S1

GOLDEN_REGRESSION_STRATEGY_v2_0_FROZEN

doc_id: GOLDEN_REGRESSION_STRATEGY_v2_0

status: FROZEN

system_name: tooling_dfm_advisor

architecture_level: INDUSTRIAL_GRADE_FREEZE

version: V2_0

freeze_date_utc: 2026-02-27T13:59:27Z
