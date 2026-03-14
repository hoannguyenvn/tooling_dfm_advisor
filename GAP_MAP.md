# Gap Map: `tooling_dfm_advisor`

This document outlines the gap between the repository's current state and the required state as defined by the authority WS documents and the `PROJECT_RECONSTRUCTION.md`.

The current state consists only of the Word Spec (`.docx`, `.md`) documents and the `PROJECT_RECONSTRUCTION.md` file. The gap comprises the entire application source code, configuration, and runtime documentation.

## 1. Missing Directory Structure

The entire directory structure required for the application is missing.

- `config/`
- `docs/audit/`
- `docs/runbooks/`
- `tooling_dfm_advisor/` (and all its subdirectories)
    - `agents/`
    - `contracts/`
    - `geometry/`
    - `orchestration/`
    - `providers/`
    - `reporter/`
    - `ui_contract/`

## 2. Missing Core Configuration Files

These are the bootstrap-critical configuration and registry files.

- `config/governance_policy.json`
- `docs/runbooks/step_registry.json`
- `docs/runbooks/GENESIS_KIT/` (directory and its contents)

## 3. Missing Governance and Audit Infrastructure

The core components for state management, audit, and concurrency control are absent.

- `docs/audit/SESSION_STATE.json`
- `docs/audit/SESSION_AUDIT_LOG.md`
- `docs/audit/SESSION_STATE.lock`

## 4. Missing Application Source Code

All Python source code is missing. This includes:

- **Top-level Scripts**:
    - `govctl.py` (or equivalent CLI entrypoint)
- **`tooling_dfm_advisor` Package**:
    - **Orchestration**: `orchestration/orchestrator.py`
    - **Contracts**: All versioned API data classes and the `contracts/contract_index_provider.py`.
    - **Providers**: `step_import_provider.py`, `geometry_validation_gate.py`, etc.
    - **Geometry**: `geometry/pythonocc_backend_provider.py`.
    - **Agents**: `healer_agent.py`, `thickness_agent.py`, etc.
    - **Reporter**: `reporter/reporter_agent.py`, etc.

## 5. Missing Test Infrastructure

There is no testing framework, test runners, or any test files. The `GOLDEN_CORPUS_MISSING` and `EMPTY_TEST_SUITE` CI gates imply a significant testing infrastructure is required.

## 6. Missing Dependency Management

There are no `requirements.txt`, `pyproject.toml`, or `conda-lock.yml` files to manage dependencies like `pythonocc-core`.

## 7. Missing Project Documentation

- `README.md`: A user-facing README explaining the project, how to set it up, and how to run it.
- Developer documentation on how to add new agents, providers, or contracts.
