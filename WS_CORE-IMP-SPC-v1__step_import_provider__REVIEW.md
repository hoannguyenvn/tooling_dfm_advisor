**WS_CORE-IMP-SPC-v1 — REVIEW**

Document pack item: Canonical specification (SPC).

# 0. Document Control Block

|                                  |                                                                                                   |
| -------------------------------- | ------------------------------------------------------------------------------------------------- |
| Canonical Code                   | WS_CORE-IMP-SPC-v1                                                                                |
| Status                           | REVIEW                                                                                            |
| Run ID                           | 20260228T090654Z                                                                                  |
| WS / Module / Doc Type / Version | WS_CORE / IMP / SPC / v1                                                                          |
| Created date (UTC)               | 20260228T090654Z                                                                                  |
| Purpose                          | Document governance consolidation only; rename + merge + formatting. No runtime/pipeline changes. |

## 1. Scope

This SPC consolidates the step_import_provider specification (algorithm logic, bindings/contracts, and dependencies) into a single canonical document for WS_CORE.

Non-goals:

- No changes to technical semantics, algorithm behavior, or contract meaning.
- No pipeline/runtime redesign; no new stages are introduced.
- No geometry healing/normalization beyond what is already described in sources.

## 2. Included Sources & Mapping Table

|           |                                                           |                                                            |                         |               |                                                          |
| --------- | --------------------------------------------------------- | ---------------------------------------------------------- | ----------------------- | ------------- | -------------------------------------------------------- |
| source_id | filename                                                  | original_title                                             | original_version/status | sections_used | notes                                                    |
| S1        | ADN_step_import_provider_step_import_and_agl_init_v1.docx | ADN - step_import_provider - step_import_and_agl_init - v1 | REVIEW                  | 3.1 ADN       | Primary logic/algorithm text.                            |
| S2        | AMBR_step_import_provider_v1.docx                         | AMBR - step_import_provider - v1                           | REVIEW                  | 3.2 AMBR      | Bindings to contracts/envelopes + execution constraints. |
| S3        | LDR_step_import_provider_dependencies_v1.docx             | LDR - step_import_provider dependencies - v1               | REVIEW                  | 3.3 LDR       | Dependencies and stdlib exemptions.                      |

## 3. Consolidated Specification Content

## 3.1 ADN (Algorithm / Logic)

**Source: S1 (ADN_step_import_provider_step_import_and_agl_init_v1.docx)**

ADN - step_import_provider - step_import_and_agl_init - v1

Trạng thái: REVIEW (V1 chuẩn hóa theo Compliance Audit + định hướng DFM-first).

Nguồn: chuyển hóa từ hồ sơ facts-only của module step_importer; các mục thiếu bằng chứng vẫn giữ nhãn [CHUA XAC MINH].

## 1. Thông tin định danh

|                   |                                                      |
| ----------------- | ---------------------------------------------------- |
| Truong            | Gia tri                                              |
| adn_id            | ADN_step_import_provider_step_import_and_agl_init_v1 |
| module_name       | step_import_provider                                 |
| module_type       | provider                                             |
| algorithm_name    | step_import_and_agl_init                             |
| algorithm_version | v1                                                   |
| pipeline_stage    | Step 2 - Geometry Preparation Core                   |
| contracts_in      | STEP_IMPORT_API_v1                                   |
| envelopes_out     | STEP_IMPORT_RESULT_v1                                |

Ghi chu naming: module cung cap du lieu bat buoc co hau to '\_provider' (snake_case).

## 2. Why

Buoc nay nap du lieu STEP (path/bytes/bytes_ref) va khoi tao Authoritative Geometry Layer (AGL) o trang thai pre-freeze. DFM-first: neu khong co AGL hop le thi cac agent DFM ve sau chi duoc phep chay o che do degraded hoac NOT_EXECUTED de bao toan SSOT va tinh minh bach bao cao.

## 3. Preconditions

- Input phai cung cap run_id dung format YYYYMMDDThhmmssZ (vi du: 20260212T071350Z).
- Payload phai co it nhat 1 trong 3 tham chieu STEP: step_path hoac step_bytes hoac bytes_ref.
- Neu dung step_path: file phai ton tai tai thoi diem thuc thi.
- Geometry backend co the thieu; bat buoc fail-safe (khong lam crash pipeline).

## 4. Inputs (STEP_IMPORT_API_v1)

Input contract STEP_IMPORT_API_v1 (normative for V1):

- run_id: string (YYYYMMDDThhmmssZ) - bat buoc.
- step_path: string (optional, exclusive-or voi step_bytes/bytes_ref).
- step_bytes: bytes (optional).
- bytes_ref: string (optional) - tham chieu den kho luu tru tam thoi (ephemeral) do governance quan ly. [CHUA XAC MINH co che bytes_ref].
- rule_version: string (bat buoc neu da co o governance_policy; neu upstream chua cap, set 'UNKNOWN' va bao issue).

## 5. Actions

Thu tuc thuc thi (SEQUENTIAL):

- A1. Xac dinh input_mode: PATH / BYTES / BYTES_REF.
- A2. Validate input_mode va dieu kien toi thieu (neu thieu -> fail-fast voi error code INPUT_MISSING_REFERENCE).
- A3. Neu PATH mode: kiem tra ton tai file (neu khong ton tai -> fail-fast STEP_PATH_NOT_FOUND).
- A4. Goi gateway duy nhat: brep_service.load_step_geometry(...) de nap STEP -> B-Rep va khoi tao AGL pre-freeze.
- A5. Dien giai ket qua backend: READY / NOT_EXECUTED / ERROR. Backend thieu -> NOT_EXECUTED + warning BACKEND_NOT_AVAILABLE (khong crash).
- A6. Dong goi vao STEP_IMPORT_RESULT_v1 va tra ve.

## 6. Outputs (STEP_IMPORT_RESULT_v1)

Output envelope STEP_IMPORT_RESULT_v1 (UPPER_CASE + suffix v1, tamper-evident qua receipts o tang governance):

- status: READY | NOT_EXECUTED | ERROR
- run_id: string (echo tu input)
- geometry_version_id: string (id cua phien ban hinh hoc AGL pre-freeze). Neu chua tao duoc thi 'UNKNOWN' + issue MISSING_GEOMETRY_VERSION_ID. [CHUA XAC MINH quy tac tao id].
- rule_version: string (echo tu input; neu UNKNOWN phai co issue MISSING_RULE_VERSION)
- agl_ref: object (tham chieu AGL trong SSOT; khong duoc luu file path/pointer). [CHUA XAC MINH schema REGION_REF_CONVENTION cho AGL].
- brep_summary: object (topology_invariants/coarse stats) [CHUA XAC MINH field list].
- errors: list[ {code, message, details?} ]
- warnings: list[ {code, message, details?} ]
- degraded_mode: bool (true neu status != READY hoac co warning nghiem trong).

## 7. Fail-fast, Fail-safe, va error taxonomy

Fail-fast (input/IO) - dung status=ERROR va error codes co cau truc:

- INPUT_MISSING_REFERENCE: khong co step_path/step_bytes/bytes_ref.
- STEP_PATH_NOT_FOUND: step_path khong ton tai.

Fail-safe (deps/backends) - khong duoc nem exception lam chet pipeline:

- BACKEND_NOT_AVAILABLE: backend geometry thieu -> status=NOT_EXECUTED + warning.
- BACKEND_LOAD_FAILED: backend co nhung load that bai -> status=ERROR + error; chi tiet error_code closed-set [CHUA XAC MINH].

## 8. Non-goals

- Khong healing, khong thay doi topology/geometry ngoai viec import.
- Khong chay DFM analyses.
- Khong tao report artifact, khong tao PNG/SVG/Mesh output.

## 9. Evidence & Observability

- Receipt bat buoc: tao JSON receipt + receipt_sha256 + receipt_sig_hmac_sha256 o tang orchestration/governance (ngoai scope module).
- Log bat buoc: ghi ro input_mode, status, error/warning codes, va thong tin backend availability.
- Determinism: cung STEP + cung backend state -> ket qua tuong duong (muc tieu).

## 10. Troubleshooting

- Neu nhan INPUT_MISSING_REFERENCE: kiem tra payload STEP_IMPORT_API_v1 co it nhat 1 tham chieu STEP.
- Neu nhan STEP_PATH_NOT_FOUND: kiem tra duong dan va quyen truy cap file; tranh dung relative path trong production.
- Neu status=NOT_EXECUTED voi BACKEND_NOT_AVAILABLE: kiem tra cai dat geometry backend (pythonocc-core) o moi truong; he thong phai van xuat report voi degraded_mode=true.
- Neu BACKEND_LOAD_FAILED: thu doi input_mode (bytes vs path) hoac kiem tra file STEP bi hong; can receipt stderr log de truy vet.

## 3.2 AMBR (Bindings / Module Contract / Execution)

**Source: S2 (AMBR_step_import_provider_v1.docx)**

AMBR - step_import_provider - v1

Trạng thái: REVIEW. AMBR nay rang buoc module step_import_provider voi ADN va cac contract/envelope theo bo tieu chuan naming + governance.

## 1. Module identification

|                |                                    |
| -------------- | ---------------------------------- |
| Truong         | Gia tri                            |
| ambr_id        | AMBR_step_import_provider_v1       |
| package        | tooling_dfm_advisor.geometry       |
| file_name      | step_import_provider.py            |
| module_name    | step_import_provider               |
| module_type    | provider                           |
| execution_mode | SEQUENTIAL                         |
| pipeline_stage | Step 2 - Geometry Preparation Core |
| status         | REVIEW                             |

## 2. Bound algorithm

|                     |                                                      |
| ------------------- | ---------------------------------------------------- |
| Truong              | Gia tri                                              |
| adn_id              | ADN_step_import_provider_step_import_and_agl_init_v1 |
| algorithm_name      | step_import_and_agl_init                             |
| algorithm_version   | v1                                                   |
| adn_status_required | APPROVED                                             |

## 3. Bound contracts & envelopes

- input_contracts: STEP_IMPORT_API_v1
- output_envelopes: STEP_IMPORT_RESULT_v1
- ssot_write_scope: duoc phep khoi tao GeometryStack/AGL pre-freeze; cam healing va cam topology mutation.

## 4. Bound libraries / dependencies

External libraries: none (stdlib duoc xem la EXEMPT_STDLIB neu governance cho phep).

- stdlib: pathlib (path/existence checks) - EXEMPT_STDLIB [CHUA XAC MINH key policy].
- stdlib: typing (type hints only) - EXEMPT_STDLIB [CHUA XAC MINH].
- internal dependency: tooling_dfm_advisor.geometry.brep_service (sole gateway to load_step_geometry).

## 5. Responsibility split

Module owns:

- Xac dinh input_mode va validate input.
- Kiem tra ton tai file neu PATH mode.
- Goi duy nhat brep_service.load_step_geometry.
- Dong goi STEP_IMPORT_RESULT_v1 voi status/errors/warnings co cau truc.

Forbidden here:

- Healing / geometry normalization beyond import.
- Bypass gateway (khong goi truc tiep OCC/low-level backend).
- Any persistent asset writes (khong ghi mesh/PNG/SVG).

## 6. Fail-safe requirements

- Neu geometry backend thieu: MUST return status=NOT_EXECUTED + warning BACKEND_NOT_AVAILABLE; MUST NOT crash pipeline.
- Moi exception runtime phai duoc bat va chuyen hoa thanh error structure trong STEP_IMPORT_RESULT_v1 (khong throw unhandled).

## 7. Audit & traceability

- run_id bat buoc (YYYYMMDDThhmmssZ) - propagate end-to-end.
- geometry_version_id bat buoc neu AGL tao thanh cong; neu khong phai giai thich bang issue.
- rule_version bat buoc; neu upstream chua cap -> 'UNKNOWN' + issue.
- Receipts: JSON + receipt_sha256 + receipt_sig_hmac_sha256 (governance layer).

## 8. Change impact & tests (Industrial Governance)

Bat buoc kem theo regression tests (Golden Geometry QA):

- Moi thay doi semantics import/AGL init: phai cap nhat Expected Defect Set hoac golden corpus tests tuong ung (neu co). [CHUA XAC MINH danh muc golden corpus].
- CI must fail neu khong co tests (EMPTY_TEST_SUITE).
- Test bat buoc toi thieu: path mode ok, path not found, bytes mode ok, backend missing -> NOT_EXECUTED (fail-safe).

## 3.3 LDR (Dependencies / Libraries / Runtime constraints)

**Source: S3 (LDR_step_import_provider_dependencies_v1.docx)**

LDR - step_import_provider dependencies - v1

Trạng thái: REVIEW. LDR nay ghi nhan cac phu thuoc (stdlib + internal gateway) cua step_import_provider.

## 1. LDR record - internal dependency: brep_service

|                  |                                                              |
| ---------------- | ------------------------------------------------------------ |
| Truong           | Gia tri                                                      |
| ldr_id           | LDR_internal_module_brep_service_for_step_import_provider_v1 |
| status           | REVIEW                                                       |
| library_name     | tooling_dfm_advisor.geometry.brep_service                    |
| library_version  | internal [CHUA XAC MINH]                                     |
| language_runtime | python                                                       |
| used_by_modules  | step_import_provider                                         |
| decision_version | v1                                                           |

Purpose: gateway duy nhat de nap STEP -> B-Rep va khoi tao AGL pre-freeze thong qua ham load_step_geometry.

## Allowed usage

- Chi duoc goi brep_service.load_step_geometry (sole gateway).
- Chi duoc truyen du lieu STEP tu STEP_IMPORT_API_v1, khong tu doc/ghi file theo cach rieng.

## Forbidden usage

- Cam goi truc tiep OCC/low-level backend tu step_import_provider (bypass gateway).
- Cam side effects khac ngoai SSOT init (khong ghi file, khong network).

## Risks & mitigations

- Rui ro: backend nang, phu thuoc platform; co the thieu tren Windows/CI.
- Giam rui ro: gateway phai cung cap ket qua co status NOT_EXECUTED + warning BACKEND_NOT_AVAILABLE; step_import_provider chi propagate, khong crash.

## 2. LDR record - stdlib exemptions

Stdlib duoc uu tien EXEMPT_STDLIB de giam overhead LDR. Neu governance_policy chua co exemption, giu record nay de truy vet.

|                |              |        |                                               |
| -------------- | ------------ | ------ | --------------------------------------------- |
| ldr_id         | library_name | status | note                                          |
| LDR_pathlib_v1 | pathlib      | REVIEW | Stdlib; dung cho Path.exists trong PATH mode. |
| LDR_typing_v1  | typing       | REVIEW | Stdlib; type hints only.                      |

## 3. Compliance notes

- Khong duoc them external geometry libs vao step_import_provider truc tiep; neu can thi phai qua brep_service va cap nhat LDR/AMBR.
- Moi thay doi signature load_step_geometry: bat buoc bump decision_version va cap nhat regression tests (Golden Geometry QA).

## 4. Open Questions / [CHUA XAC MINH]

- [CHUA XAC MINH] Confirm whether STATUS_TARGET should remain REVIEW for this canonical SPC (request specifies REVIEW, but confirm intended freeze path).
- [CHUA XAC MINH] Confirm canonical codes for referenced contracts/envelopes if they are also tracked as separate WS_MODEL docs (STEP_IMPORT_API_v1, STEP_IMPORT_RESULT_v1).

## 5. Change Log (Editorial)

This document is a governance consolidation (rename + merge + formatting). No runtime or technical semantics changes are intended.

Inserted Document Control Block and Included Sources & Mapping Table to enforce traceability.

Grouped content into ADN/AMBR/LDR as required for SPC; preserved source text and added explicit Source markers.
