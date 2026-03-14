# 0. Document Control Block

Canonical Code: WS_CORE-GEN-SPC-v1

Status: REVIEW

Run ID: 20260304T085841Z

WS / Module / Doc Type / Version: WS_CORE / GEN / SPC / v1

Created date (UTC): 20260304T085841Z

Purpose: Document governance consolidation only; no runtime/pipeline changes.

## 1. Scope

[CHƯA XÁC MINH] Tài liệu này được định vị là “kernel/deps” baseline cho geometry core (core dependency baseline). Nội dung kỹ thuật được kế thừa trực tiếp từ các nguồn ADN/AMBR/LDR liên quan pythonocc_backend_provider và pythonocc-core.

Non-goals: Không định nghĩa thuật toán DFM; không thay đổi semantics; không thay đổi phân loại WS; không mô phỏng; không thêm policy/thresholds ngoài nguồn.

## 2. Included Sources & Mapping Table

|           |                                                                    |                                   |                         |               |                            |
| --------- | ------------------------------------------------------------------ | --------------------------------- | ----------------------- | ------------- | -------------------------- |
| source_id | filename                                                           | original_title                    | original_version/status | sections_used | notes                      |
| S1        | ADN_pythonocc_backend_provider_step_io_and_basic_brep_meta_v1.docx | ADN - pythonocc_backend_provider  | [CHƯA XÁC MINH]         | 3.1 ADN       | primary algorithm text     |
| S2        | AMBR_pythonocc_backend_provider_v1.docx                            | AMBR - pythonocc_backend_provider | [CHƯA XÁC MINH]         | 3.2 AMBR      | bindings/contracts         |
| S3        | LDR_pythonocc_core_v1.docx                                         | LDR - pythonocc-core (OCC.Core)   | [CHƯA XÁC MINH]         | 3.3 LDR       | deps & runtime constraints |

## 3. Consolidated Specification Content

## 3.1 ADN (Algorithm / Logic)

Source: S1

ADN - pythonocc_backend_provider

Step I/O + Basic B-Rep Metadata (Algorithm v0) - Record v1 (DFM-first)

1. Problem statement

Cung cap adapter cap thap giao tiep voi pythonocc-core (OCC.Core) de thuc hien cac thao tac toi thieu: kiem tra backend co hien dien, doc/ghi STEP, trich xuat thong tin B-Rep co ban, va tao hinh hoc mau phuc vu kiem thu.

Non-goals: khong healing; khong phan tich DFM; khong quan ly SSOT toan cuc.

2. DFM-first rationale (Why)

Backend OCC la dependency nang (C++). Can co 'trust boundary' ro rang de pipeline khong bi sap khi thieu deps.

Step import / validate / freeze phu thuoc ket qua STEP read. Neu backend khong on dinh, toan bo DFM phia sau mat do tin cay.

Do la adapter, module phai uu tien: fail-safe, traceability-friendly, khong leak du lieu/duong dan ra ngoai.

3. Preconditions

pythonocc-core co the import duoc trong runtime (neu khong: degraded mode / NOT_EXECUTED o tang caller).

Duong dan STEP hop le, ton tai va co quyen doc khi thuc hien read.

Neu thuc hien write_step: duong dan output phai nam trong 'ephemeral run workspace' (se duoc purge sau delivery_complete).

4. Inputs

Primary inputs:

step_path: str (duong dan file STEP)

shape: TopoDS_Shape (doi tuong B-Rep da co trong bo nho)

out_step_path: str (duong dan ghi STEP - chi dung cho debug/utility)

5. Outputs

Module cung cap 2 lop output (de dam bao fail-safe end-to-end):

Low-level primitives (noi bo geometry layer): TopoDS_Shape hoac exception typed (StepImportReadError, StepImportTransferError).

Safe wrappers (khuyen nghi la public API): Result Envelope UPPER_CASE de caller khong bi crash.

Result envelopes de xuat (Record v1):

OCC_BACKEND_PROBE_RESULT_v0: {status: PRESENT|ABSENT, error_code?, message?}

STEP_READ_RESULT_v0: {status: OK|NOT_EXECUTED|FAILED, shape_present: bool, errors[], warnings[]}

STEP_WRITE_RESULT_v0: {status: OK|NOT_EXECUTED|FAILED, out_step_path_sanitized, errors[], warnings[]}

BASIC_BREP_META_RESULT_v0: {status: OK|FAILED, import_ok: bool, shape_type: str, warnings[]}

Luu y: out_step_path_sanitized KHONG duoc chua absolute path trong report/receipt; chi duoc la id/alias trong workspace.

6. Algorithm overview

Thin adapter: su dung STEPControl_Reader/Writer va cac primitive OCC de thuc hien doc/ghi STEP, transfer va lay OneShape, kiem tra IsNull, va trich xuat shape_type co ban.

7. Step-by-step logic

7.1 Read STEP (unsafe internal):

Khoi tao STEPControl_Reader.

Reader.ReadFile(step_path) va kiem tra status == IFSelect_RetDone.

Reader.TransferRoots() (hoac workflow tuong duong) de chuyen doi du lieu.

shape = Reader.OneShape().

Neu shape.IsNull() == True: raise StepImportTransferError (hoac StepImportReadError theo mapping).

Return shape.

7.2 Read STEP (safe wrapper - recommended public):

Neu OCC absent: return STEP_READ_RESULT_v0(status=NOT_EXECUTED, error_code='OCC_NOT_PRESENT').

Goi unsafe read; bat StepImportReadError/StepImportTransferError.

Neu bat exception: return STEP_READ_RESULT_v0(status=FAILED, errors=[...]).

Neu thanh cong: return STEP_READ_RESULT_v0(status=OK, shape_present=True).

7.3 Write STEP (safe wrapper):

Validate out_step_path nam trong ephemeral run workspace (policy check o layer caller/validator).

Neu OCC absent: return STEP_WRITE_RESULT_v0(status=NOT_EXECUTED, error_code='OCC_NOT_PRESENT').

Dung STEPControl_Writer ghi shape ra out_step_path.

Return STEP_WRITE_RESULT_v0(status=OK).

7.4 Basic metadata extraction:

Nhan shape (TopoDS_Shape).

Trich xuat shape_type (chuoi on dinh cho audit) va import_ok.

Return BASIC_BREP_META_RESULT_v0(status=OK, import_ok=..., shape_type=...).

8. Failure modes and error taxonomy

Typed exceptions (noi bo):

StepImportReadError: status ReadFile != IFSelect_RetDone.

StepImportTransferError: transfer that bai / OneShape invalid / IsNull.

Structured error codes (safe envelopes):

OCC_NOT_PRESENT

STEP_READ_FAILED

STEP_TRANSFER_FAILED

SHAPE_IS_NULL

STEP_WRITE_FAILED

OUT_PATH_POLICY_VIOLATION

Fail-safe rule: khong exception nao duoc propagate ra ngoai neu caller su dung safe wrappers.

9. Determinism and limits

Deterministic theo: cung STEP + cung pythonocc-core/OpenCascade runtime + cung OS/locale. [CHUA XAC MINH] can ghi ro version pin.

Memory/time complexity bi chi phoi boi STEP parsing + transfer (phu thuoc kich thuoc/complexity file).

Khong co tolerance engineering theo Context o day; tolerance configuration neu can phai o layer cao hon (policy-as-config).

10. Security, IP, and data retention

Khong duoc ghi lai path file goc vao report/receipt. Neu can traceability, dung geometry_version_id va bytes_ref/workspace_id.

write_step chi duoc dung trong debug/utility va phai nam trong ephemeral workspace; se bi purge sau delivery_complete.

Khong duoc luu bat ky snapshot/mesh nao tai module nay.

11. Evidence required to approve this record

Pin pythonocc-core version + license + install channel (LDR).

Chung minh end-to-end: khi OCC thieu -> pipeline tra NOT_EXECUTED/STUB_MODE envelope, khong crash.

Golden Geometry Corpus test: it nhat 1 STEP 'clean' va 1 STEP 'complex' cho read/transfer regression.

Schema shape_type (allowed values) va dinh nghia import_ok on dinh (backward compatible).

12. Open items [CHUA XAC MINH]

Behavior cu the cua 'presence check' khi OCC absent (return false vs raise ImportError).

Policy/tolerance mapping cho STEP transfer (neu can).

Thread-safety va memory leak concerns trong pythonocc-core runtime tren Windows/Linux.

## 3.2 AMBR (Bindings / Module Contract / Execution)

Source: S2

AMBR - pythonocc_backend_provider

Module Binding Record v1 (DFM-first)

1. Bound algorithm

adn_id: ADN_pythonocc_backend_provider_step_io_and_basic_brep_meta_v1

algorithm_name: step_io_and_basic_brep_meta

algorithm_version: v0

2. Bound libraries

external_dependency: LDR_pythonocc_core_v1 (pythonocc-core / OCC.Core)

note: heavy C++ runtime dependency; treat as optional-at-runtime (degraded allowed).

3. Execution context

Execution role: geometry core adapter. Module phai co lap OCC.Core imports trong geometry layer.

does_not_write_ssot: true

provides_agl_seed: true (feeds step_import_provider / brep_service)

does_not_produce_dfm_defects: true

allowed_side_effects: read step from disk; (optional) write step to ephemeral workspace only

4. Contracts and public surface

De dam bao fail-safe, public surface (su dung boi provider/agent o layer cao hon) MUST return envelopes (no uncaught exceptions).

OCC_BACKEND_PROBE_RESULT_v0

STEP_READ_RESULT_v0

STEP_WRITE_RESULT_v0

BASIC_BREP_META_RESULT_v0

Unsafe helpers (noi bo geometry layer) co the raise typed exceptions, nhung khong duoc leak ra ngoai boundary.

5. Responsibility split

Module owns:

OCC.Core API interaction (STEPControl_Reader/Writer, TopoDS_Shape).

Exception typing + mapping to structured error codes trong safe wrappers.

Sanitization: khong tra ve absolute path ra ngoai (chi workspace alias/id).

Pipeline / caller owns:

Cap run_id, rule_version, geometry_version_id va governance receipts.

Policy enforcement cho 'ephemeral workspace' + purge sau delivery_complete.

Golden Geometry Corpus management + CI gates (EMPTY_TEST_SUITE).

6. Data flow

Inputs: step_path, shape, out_step_path (debug/utility).

Outputs: result envelopes; (optional) TopoDS_Shape noi bo thong qua safe_read_step.

SSOT: module khong ghi; ket qua duoc step_import_provider dung de khoi tao AGL pre-freeze.

7. Fail-fast vs fail-safe

Fail-fast chi duoc phep trong noi bo geometry layer; end-to-end pipeline phai fail-safe.

Neu OCC khong present: return status=NOT_EXECUTED (error_code=OCC_NOT_PRESENT).

Neu read/transfer fail: return status=FAILED + errors[]; khong throw unhandled.

Caller phai tiep tuc pipeline o degraded_mode va phai render report minh bach.

8. Observability & traceability

Module khong tu tao run_id. Neu duoc truyen run_id vao wrapper, phai echo vao envelope (optional).

Khong log thong tin nhay cam: duong dan file goc, network share path, user name.

Issue codes phai on dinh va co the map sang receipts/governance events.

9. Change impact rules

Thay doi OCC API usage anh huong: step_import_provider, brep_service, mapping_provider.

Bat buoc chay regression tren Golden Geometry Corpus.

Neu thay doi schema envelope/error codes: bump envelope version va cap nhat spec/addendum.

10. Test obligations (minimum)

Test 1: OCC absent -> STEP_READ_RESULT_v0.status == NOT_EXECUTED.

Test 2: invalid STEP -> status == FAILED + error_code STEP_READ_FAILED.

Test 3: golden STEP -> status == OK and shape_present == True.

Test 4 (optional): write_step outputs into ephemeral workspace and gets purged (integration test).

11. Open items [CHUA XAC MINH]

Exact repo package root name in project (tooling_dfm_advisor vs src/siafu_dfm_advisor) - needs alignment.

Exact 'presence check' API signature and behavior.

Pinned version and installation channel for pythonocc-core.

## 3.3 LDR (Dependencies / Libraries / Runtime constraints)

Source: S3

LDR - pythonocc-core (OCC.Core)

Library Dependency Record v1 (DFM-first)

1. Purpose

Cung cap B-Rep kernel binding de doc/ghi STEP, tao TopoDS_Shape, va su dung cac primitive nhu BRepPrimAPI_MakeBox.

2. Justification (DFM-first)

AGL B-Rep la authoritative geometry cho DFM; pythonocc-core la backend phu hop voi Master Spec.

Can pin version de dam bao determinism cho STEP transfer va topo invariants.

3. Allowed usage (APIs)

STEPControl_Reader / STEPControl_Writer

IFSelect_RetDone (status checking)

TransferRoots/OneShape workflow

TopoDS_Shape

BRepPrimAPI_MakeBox

4. Forbidden usage

Healing trong pythonocc_backend_provider (healing thuoc healer_agent/layer khac).

Mutate SSOT toan cuc hoac ghi data nhay cam ra disk ngoai ephemeral workspace.

Rai rac imports OCC.Core ra cac layer khac neu khong co AMBR/Policy cho phep.

5. Risks & constraints

Heavy C++ dependency: de bi thieu runtime/ABI mismatch tren Windows.

Thread-safety / memory management: [CHUA XAC MINH].

Transfer tolerance default khong theo Context: rui ro mismatch tolerance engineering.

Determinism phu thuoc version pin + OS + locale.

6. Version pinning & upgrade policy

Version pinning la bat buoc, nhung chua co so lieu version cu the trong facts.

Pin pythonocc-core + OpenCascade runtime version (exact) trong lockfile (conda-lock/requirements) [CHUA XAC MINH].

Upgrade chi duoc phep khi Golden Geometry Corpus regression PASS.

Rollback: quay ve last-known-good env snapshot/lockfile.

7. Verification checklist

Verify import OCC.Core on target OS images (Windows + Linux) [CHUA XAC MINH].

Verify STEP read/write basic on golden STEP.

Verify failure behavior when OCC missing: safe wrapper returns NOT_EXECUTED (no crash).

8. Open items required for APPROVED

library_version: exact version to pin.

license: SPDX id or license text reference.

install channel: conda-forge vs pip wheels; how to reproduce in CI.

Known platform constraints (GPU/AVX, VC++ runtime) [CHUA XAC MINH].

## 4. Open Questions / [CHƯA XÁC MINH]

[CHƯA XÁC MINH] Xác nhận phân loại WS: giữ WS_CORE (core dependency baseline) hay chuyển sang WS_PROVIDERS (provider). Ảnh hưởng: vị trí SSOT trong registry và cách gom các provider khác.

[CHƯA XÁC MINH] Xác nhận vMAJOR v1 cho canonical code, trong khi ADN nguồn ghi “Algorithm v0”. Ảnh hưởng: governance versioning và traceability khi phát hành.

## 5. Change Log (Editorial)

REVIEW: Tạo tài liệu canonical theo chuẩn WS/canonical code; sắp xếp lại heading + bổ sung control block/mapping/traceability. Không thay đổi semantics nội dung nguồn.

## 6. Appendix

aka mapping (nếu có): pythonocc_backend_provider (implementation name, snake_case).
