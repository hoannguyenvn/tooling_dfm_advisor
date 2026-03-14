# 0. Document Control Block

Canonical Code: WS_PROVIDERS-MAP-SPC-v1

Status: REVIEW

Run ID: 20260304T094012Z

WS / Module / Doc Type / Version: WS_PROVIDERS / MAP / SPC / v1

Created date (UTC): 20260304T094012Z

Purpose: Document governance consolidation only; no runtime/pipeline changes.

## 1. Scope

This canonical SPC consolidates the provider specification for mapping_provider (AGL→DGL mapping backbone) from the supplied sources. Technical content is preserved via verbatim extracts; no semantics change is introduced.

Excluded / Non-goals: Any content about other providers (e.g., mesh_service_provider, contract_index_provider) is out of scope for this SPC and is not consolidated here.

## 2. Included Sources & Mapping Table

|           |                                                          |                            |                           |               |                                         |
| --------- | -------------------------------------------------------- | -------------------------- | ------------------------- | ------------- | --------------------------------------- |
| source_id | filename                                                 | original_title             | original_version/status   | sections_used | notes                                   |
| S1        | ADN_mapping_provider_agl_to_dgl_mapping_backbone_v1.docx | ADN mapping_provider       | v1 / REVIEW (as provided) | 3.1 ADN       | Primary algorithm/logic record          |
| S2        | AMBR_mapping_provider_v1.docx                            | AMBR mapping_provider      | v1 / REVIEW (as provided) | 3.2 AMBR      | Module binding/execution/contract notes |
| S3        | LDR_occ_core_mapping_stack_v1.docx                       | LDR occ_core_mapping_stack | v1 / REVIEW (as provided) | 3.3 LDR       | Dependency boundary for OCC.Core subset |

## 3. Consolidated Specification Content

## 3.1 ADN (Algorithm / Logic)

Source: S1

ADN — mapping_provider — AGL→DGL Mapping Backbone + Proxy Metrics — v1

Status: REVIEW (V1 candidate). DFM-first: derived mesh proxy + traceable mapping; integrity check may be SKIPPED but must be explicit.

1. Identification

adn_id: ADN_mapping_provider_agl_to_dgl_mapping_backbone_v1

module_name: mapping_provider

module_type: provider (geometry/mapping)

algorithm_name: agl_to_dgl_mapping_backbone

algorithm_version: v1

result_envelope: MAPPING_RESULT_v0 (schema locked here)

length_unit: mm (system policy)

owner: [CHƯA XÁC MINH]

authority_refs: Master Spec + Governance Spec + Addendum V1

2. Why

mapping_provider tạo backbone truy vết giữa AGL (B-Rep reference) và DGL (mesh proxy) để downstream agents có thể dùng hình học dẫn xuất (DGL) mà vẫn kiểm toán được theo geometry_version_id và run_id. Đầu ra phải mang proxy metrics (face/triangle/vertex counts) để xác nhận DGL đã được tạo.

3. Preconditions

Input MUST carry run_id (YYYYMMDDThhmmssZ), rule_version, geometry_version_id.

AGL đã freeze (topology stable) hoặc tối thiểu là stable trong phạm vi geometry_version_id.

Không được mutate AGL; chỉ được tạo derived DGL + mapping table.

Không ghi file, không network IO, không leak file paths/pointers vào output/receipt/report.

Nếu pythonocc-core thiếu: MUST return NOT_EXECUTED (fail-safe), không throw unhandled.

Mesh tolerance 0.5: assumed in length_unit=mm. Nếu chưa normalize unit về mm → kết quả mesh có thể sai. [CHƯA XÁC MINH upstream normalization guarantee].

4. Inputs

Preferred typed input (governance-by-contract):

GeoModelResponse (GEO_MODEL_API_v1) — reads: geometry context (brep_handle/agl_ref) + metadata.

Optionally reads: policy snapshot for mesh params (tolerance).

Legacy compatibility (allowed only if upstream not yet refactored):

payload dict keys (facts): agl_ref, dgl_ref, geometry {brep_shape}, geometry_context_in [CHƯA XÁC MINH exact nesting].

5. Actions / Algorithm (step-by-step)

Initialize MAPPING_RESULT_v0 envelope with status=NOT_EXECUTED, degraded_mode=true; set run_id/rule_version/geometry_version_id.

Validate presence: if missing agl_ref → status=FAILED, error=MAP_MISSING_AGL_REF (normalize from TODO_MAP_MISSING_AGL_REF), return.

Validate geometry context: if missing brep_shape/brep_handle → status=FAILED, error=GEOM_CONTEXT_MISSING, return.

Dependency probe: lazy-import OCC modules required for tessellation and traversal. If import fails → status=NOT_EXECUTED, error=NOT_SUPPORTED_NO_GEOM_LIB, return.

Tessellate TopoDS_Shape via BRepMesh_IncrementalMesh with fixed parameters (tolerance=0.5). Record mesh stats.

Traverse topology/mesh to compute proxy metrics: face_count, triangle_count, vertex_count.

Create mapping_id (deterministic within run/geometry_version_id) and produce mapping table handles (b2m/m2b) as internal refs (not file paths).

Spatial integrity check: if enabled by policy, run sample→project-back→Δ check; if disabled, set integrity_check_state=SKIPPED_V0 and add warning INTEGRITY_CHECK_SKIPPED.

Return MAPPING_RESULT_v0 with status=EXECUTED if tessellation succeeded; degraded_mode=false unless integrity check failed or skipped per policy.

6. Outputs (MAPPING_RESULT_v0 — schema locked)

Minimum required fields:

status: EXECUTED | NOT_EXECUTED | FAILED

run_id: str (YYYYMMDDThhmmssZ)

rule_version: str

geometry_version_id: str

timestamp_utc: str (UTC)

agl_ref: str

dgl_ref: str

mapping_id: str

proxy_metrics: {face_count:int, triangle_count:int, vertex_count:int}

mesh_params: {tolerance_mm: float}

integrity_check_state: PASSED | FAILED | SKIPPED_V0

integrity_metrics: optional {sample_count:int, max_delta_mm: float, mean_delta_mm: float}

geometry_context_out: opaque ref (no file paths)

errors: list[{code:str, message:str, retryable:bool}]

warnings: list[{code:str, message:str}] (optional)

degraded_mode: bool (true iff status!=EXECUTED OR integrity_check_state!=PASSED)

Explicitly forbidden in any field: absolute path, mesh file pointer, screenshot path.

7. Failure modes (structured)

MAP_MISSING_AGL_REF: missing agl_ref → FAILED, retryable=false.

GEOM_CONTEXT_MISSING: missing geometry/brep handle → FAILED, retryable=false.

NOT_SUPPORTED_NO_GEOM_LIB: OCC missing/unloadable → NOT_EXECUTED, retryable=false.

MESH_GENERATION_FAILED: tessellation exception → FAILED, retryable=false.

INTEGRITY_CHECK_FAILED: Δ exceeds threshold → EXECUTED with integrity_check_state=FAILED, degraded_mode=true (downstream may gate). [CHƯA XÁC MINH exact gating policy].

8. Evidence / Verification

Unit test: missing OCC → NOT_EXECUTED with NOT_SUPPORTED_NO_GEOM_LIB; no exception escapes.

Unit test: missing agl_ref → FAILED with MAP_MISSING_AGL_REF.

Golden corpus: at least 1 geometry should produce deterministic proxy_metrics under pinned OCC version.

Integrity check test: if enabled, a known-bad synthetic case should trigger FAILED; if disabled, SKIPPED_V0 must be present.

No-path-leak test: all strings scrubbed against Windows/POSIX absolute path patterns.

9. Fail-fast

Never return empty dict; always return MAPPING_RESULT_v0 with explicit status.

Never throw unhandled exceptions outward.

If integrity check is not implemented: must be explicit via integrity_check_state=SKIPPED_V0 and warning; never pretend PASSED.

10. Troubleshooting

Always NOT_EXECUTED: verify pythonocc-core availability/pinning and lazy-import probe paths.

Triangle/vertex counts vary: check OCC version drift; ensure tolerance_mm fixed; pin OCC version in LDR and CI.

Units mismatch suspected: verify upstream normalization to mm; if input in inch, tolerance=0.5 becomes wrong scale.

Integrity always SKIPPED: confirm policy enables integrity check; otherwise accept degraded_mode gating downstream.

11. Non-goals

Không healing, không sửa AGL.

Không tạo report, không lưu mesh ra disk.

Không quyết định final gating (orchestrator/governance owns).

12. Filled example (run_id=20260212T071350Z)

MAPPING_RESULT_v0: status=EXECUTED, run_id=20260212T071350Z, rule_version=ruleset_2026_02_12_v1, geometry_version_id=geom_9f3a..., proxy_metrics={face_count:12,triangle_count:420,vertex_count:230}, mesh_params={tolerance_mm:0.5}, integrity_check_state=SKIPPED_V0, degraded_mode=true.

## 3.2 AMBR (Bindings / Module Contract / Execution)

Source: S2

AMBR — mapping_provider — Binding v1

Status: REVIEW (V1 candidate).

1. Module identification

ambr_id: AMBR_mapping_provider_v1

module_name: mapping_provider

module_type: provider (geometry/mapping)

binding_version: v1

repo_path: tooling_dfm_advisor/geometry/mapping_provider.py [CHƯA XÁC MINH actual repo path]

execution_mode: SEQUENTIAL

implementation_level_tag: Level_7_deterministic (per facts)

ssot_permissions: may_write_DGL=true; must_not_mutate_AGL=true

owner: [CHƯA XÁC MINH]

2. Bound algorithm

adn_id: ADN_mapping_provider_agl_to_dgl_mapping_backbone_v1

algorithm_name: agl_to_dgl_mapping_backbone

algorithm_version: v1

result_envelope: MAPPING_RESULT_v0

3. Dependencies (LDR)

LDR_occ_core_mapping_stack_v1 (pythonocc-core subset for tessellation + traversal).

Internal geometry adapter (optional): pythonocc_backend_provider, if repo enforces OCC isolation. [CHƯA XÁC MINH current boundary].

4. Determinism & side effects

Determinism relies on fixed mesh params and pinned OCC version; otherwise metrics may drift across OS/OCC versions.

Forbidden: mutate AGL; forbidden: write mesh to disk; forbidden: store file paths/pointers.

Allowed: write DGL + mapping table into GeometryStack (run-local SSOT).

Ephemeral policy: any mesh artifacts must be purgeable after delivery_complete (system-wide). mapping_provider must tag DGL as ephemeral. [CHƯA XÁC MINH storage scheme].

5. Responsibility split

mapping_provider owns: tessellation + proxy metrics + mapping table refs + integrity check state field.

orchestrator/governance owns: gating policy (regeneration limits, degraded_mode triggers) and receipts.

reporter owns: presentation only; does not recompute mapping.

6. Change impact rules

mesh tolerance changes are behavior changes: must update rule_version/policy snapshot and golden expected results.

Schema change to MAPPING_RESULT_v0: breaking unless additive → if breaking, create MAPPING_RESULT_v1 and update consumers.

If integrity check is implemented (from SKIPPED_V0 → PASSED/FAILED), must add regression tests and update policy gates.

7. Test obligations (minimum)

OCC missing: NOT_EXECUTED no-crash test.

Golden corpus: at least 1 sample with expected proxy_metrics under pinned env (or threshold-based assertions).

Integrity state: SKIPPED_V0 must be explicit if not implemented.

No-path-leak test.

8. Audit & traceability

Envelope MUST carry run_id/rule_version/geometry_version_id.

Mapping table refs must be opaque; do not serialize OCC objects into receipts.

## 3.3 LDR (Dependencies / Libraries / Runtime constraints)

Source: S3

LDR — OCC.Core mapping stack (subset) — v1

Status: REVIEW (V1 candidate). Scope: minimal pythonocc-core APIs allowed for mapping_provider.

1. Identification

ldr_id: LDR_occ_core_mapping_stack_v1

library_name: pythonocc-core (OCC.Core)

library_version: [CHƯA XÁC MINH pin]

license: [CHƯA XÁC MINH]

install_channel: [CHƯA XÁC MINH]

language_runtime: Python [CHƯA XÁC MINH version pin]

used_by_modules: mapping_provider

2. Purpose

Cung cấp tessellation (BRepMesh_IncrementalMesh) và traversal topo/mesh để sinh DGL proxy và proxy metrics. Giới hạn subset API để giảm rủi ro môi trường và dễ enforce fail-safe.

3. Allowed usage (explicit APIs)

OCC.Core.BRepMesh.BRepMesh_IncrementalMesh (tessellation)

OCC.Core.TopExp.TopExp_Explorer (topology traversal)

OCC.Core.TopAbs (TopAbs_FACE/EDGE/VERTEX)

OCC.Core.BRep (BRep_Tool) — read-only access only

TopoDS/TopLoc namespaces as required for traversal (read-only)

4. Forbidden usage

No healing/mutation APIs in mapping_provider.

No file IO via OCC in mapping_provider.

No global mutable singleton OCC state that survives run boundaries. [CHƯA XÁC MINH current code].

No additional OCC modules beyond allowed subset unless a new LDR is approved.

5. Fail-safe requirement

If pythonocc-core missing/unloadable: mapping_provider must return NOT_EXECUTED with NOT_SUPPORTED_NO_GEOM_LIB.

No crashes allowed in pipeline due to missing C++ deps.

6. Determinism risks

Tessellation output may vary across OCC versions/OS → mitigation: strict version pin + golden corpus assertions.

Performance/memory scales with triangle_count; tolerance must be policy-controlled to avoid blow-up.

7. Verification checklist

CI: verify pythonocc-core presence/version on geometry-enabled runners OR validate NOT_EXECUTED fallback tests.

CI: run golden mapping tests (proxy_metrics) on pinned environment.

## 4. Open Questions / [CHƯA XÁC MINH]

Items below are carried forward verbatim from sources and remain unresolved:

owner: [CHƯA XÁC MINH]

owner: [CHƯA XÁC MINH]

license: [CHƯA XÁC MINH]

install_channel: [CHƯA XÁC MINH]

## 5. Change Log (Editorial)

20260304T094012Z: Consolidated sources S1–S3 into canonical WS_PROVIDERS-MAP-SPC-v1; added Document Control Block, Mapping Table, traceability headers, and standardized file naming. No technical semantics change.

## 6. Appendix

Appendix A — Excluded source candidates (not consolidated in this SPC):

- ADN_mesh_service_provider_mesh_token_reservation_stub_v1.docx

- AMBR_mesh_service_provider_v1.docx

- LDR_mesh_service_provider_stdlib_v1.docx

- ADN_contract_index_provider_contract_module_lookup_table_v1.docx

- DFM_POLICY_MASTER_SPEC_v1.docx

- LDR_pythonocc_core_topology_explorer_v1.docx
