# Project Reconstruction: tooling_dfm_advisor

This document reconstructs the intended state of the `tooling_dfm_advisor` repository based on the canonical WS (Word Spec) documents.

## 1. High-Level Architecture & Principles

The system is a "DFM-first" (Design for Manufacturing) analysis tool. It operates on a deterministic, evidence-driven pipeline coordinated by a central `orchestrator`.

- **System Name**: `tooling_dfm_advisor`
- **Core Logic**: The pipeline takes a STEP file, validates and "freezes" the geometry, runs a series of DFM analysis agents, and produces a final report in PDF or HTML format.
- **Governance**: A CLI tool, `govctl`, is intended to manage and validate the governance process.
- **Single Source of Truth (SSOT)**: A `tooling_model` managed via versioned APIs (`TOOLING_MODEL_API_v1`) represents the canonical state. Geometry is immutable after being "frozen".
- **Evidence & Audit**: Every step must produce a tamper-evident JSON receipt, signed with HMAC-SHA256. All operations are logged in an append-only audit log.
- **Fail-Safety**: The pipeline is designed to be resilient. Failure of an optional stage or absence of a heavy dependency (like `pythonocc-core`) should result in a `degraded_mode` and a partial report, not a system crash.
- **Ephemeral Assets**: All intermediate files (images, meshes) must be stored in a temporary workspace and purged after the final report is delivered.

## 2. Mandatory Directory Structure

The WS documents mandate a strict, non-bypassable directory structure.

```
.
├── config/
│   ├── governance_policy.json
│   ├── (optional) governance_allowlist.json
│   └── (local) policy_override.local.json
├── docs/
│   ├── audit/
│   │   ├── receipts/
│   │   │   ├── logs/
│   │   │   │   └── (*.stdout.log, *.stderr.log)
│   │   │   └── (*.json)
│   │   ├── SESSION_AUDIT_LOG.md   (Append-only)
│   │   ├── SESSION_STATE.json
│   │   ├── SESSION_STATE.lock
│   │   └── LAST_SESSION_CHECKPOINT.md (Append-only)
│   └── runbooks/
│       ├── step_registry.json
│       ├── step_registry.LKG.json
│       └── GENESIS_KIT/
├── tooling_dfm_advisor/
│   ├── __init__.py
│   ├── agents/
│   │   ├── __init__.py
│   │   ├── draft_agent.py
│   │   ├── healer_agent.py
│   │   ├── ribratio_agent.py
│   │   ├── sink_agent.py
│   │   ├── thickness_agent.py
│   │   ├── undercut_agent.py
│   │   └── weldline_agent.py
│   ├── contracts/
│   │   ├── __init__.py
│   │   └── contract_index_provider.py
│   ├── geometry/
│   │   ├── __init__.py
│   │   └── pythonocc_backend_provider.py
│   ├── orchestration/
│   │   ├── __init__.py
│   │   └── orchestrator.py
│   ├── providers/
│   │   ├── __init__.py
│   │   ├── afr_provider.py
│   │   ├── geometry_validation_gate.py
│   │   ├── mapping_provider.py
│   │   └── step_import_provider.py
│   ├── reporter/
│   │   ├── __init__.py
│   │   ├── reporter_agent.py
│   │   └── visualizer_agent.py
│   └── ui_contract/
│       └── __init__.py
├── (optional) .secrets.env
└── (optional) govctl.py
```

## 3. Key Modules and Components

The implementation will be a Python package named `tooling_dfm_advisor`.

- **`govctl.py` (or similar)**: A command-line interface for interacting with the governance model (e.g., validating state, receipts).
- **`tooling_dfm_advisor.orchestration.orchestrator`**: The heart of the system. It dynamically loads stages from a registry and executes them in a deterministic order.
- **`tooling_dfm_advisor.contracts`**: Contains Python data classes representing the versioned APIs and data transfer objects (e.g., `StepImportRequestV1`, `ORCHESTRATION_RESULT_v0`). The `contract_index_provider` within defines the allowlist of loadable modules.
- **`tooling_dfm_advisor.providers`**: Core services.
    - `step_import_provider`: Handles reading STEP files.
    - `geometry_validation_gate`: Validates the geometry's integrity (e.g., checks for open shells).
    - `mapping_provider`: Manages the mapping between analysis geometry (AGL) and design geometry (DGL).
    - `afr_provider`: Performs Automatic Feature Recognition.
- **`tooling_dfm_advisor.geometry`**:
    - `pythonocc_backend_provider`: A low-level, fail-safe adapter for the `pythonocc-core` library.
- **`tooling_dfm_advisor.agents`**:
    - `healer_agent`: Performs minimal, bounded fixes on geometry.
    - DFM Agents: A suite of agents for specific analyses (`thickness`, `draft`, `undercut`, etc.).
- **`tooling_dfm_advisor.reporter`**:
    - `reporter_agent`: Composes the final report model from the results of the analysis agents.
    - `visualizer_agent`: (Optional) Generates temporary visual assets (PNGs, SVGs) for the report.
    - A delivery component is responsible for rendering the final PDF/HTML and ensuring the `ephemeral_purge` of temporary files.

## 4. Core Dependencies

- **`pythonocc-core`**: The primary dependency for B-Rep (geometry) operations. Its absence must be handled gracefully.
- **Python Standard Library**: Notably `importlib` for dynamic module loading and `typing` for contracts.
