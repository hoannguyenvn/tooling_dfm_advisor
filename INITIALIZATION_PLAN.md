# Initialization Plan: `tooling_dfm_advisor`

This document outlines the plan for the hybrid bootstrap initialization of the `tooling_dfm_advisor` project, addressing the items in the `GAP_MAP.md`.

## Phase 1: Top-Down Foundation

This phase establishes the core governance and configuration structure.

1.  **Create Core Directories**:
    - Create the top-level directories: `config/`, `docs/audit/receipts/logs/`, `docs/runbooks/GENESIS_KIT/`, and `tooling_dfm_advisor/`.

2.  **Initialize Governance Configuration**:
    - Create `config/governance_policy.json`. This file will be populated with the schema and default values extracted from `WS_GOV-ALL-POL-v1`, including DFM thresholds, severity maps, and placeholders for decision maps.
    - Create `docs/runbooks/step_registry.json`. This will be populated with the canonical steps identified in the WS documents (e.g., `GOV_VALIDATE_REGISTRY_AND_STATE`, `step_import_provider`, `geometry_validation_gate`, etc.).

3.  **Initialize Audit State**:
    - Create `docs/audit/SESSION_STATE.json` with the initial "PAUSED" state as defined in `WS_GOV-ALL-SPC-v1`.
    - Create empty placeholder files: `docs/audit/SESSION_AUDIT_LOG.md` and `docs/audit/LAST_SESSION_CHECKPOINT.md`.

## Phase 2: Bottom-Up Skeleton & Contracts

This phase builds the Python package skeleton and defines the data contracts that allow components to communicate.

1.  **Create Python Package Structure**:
    - Create the subdirectories within `tooling_dfm_advisor/`: `agents`, `contracts`, `geometry`, `orchestration`, `providers`, `reporter`, and `ui_contract`.
    - Add an empty `__init__.py` to each directory to make them recognizable as Python packages.

2.  **Define Core Contracts**:
    - Create Python files within `tooling_dfm_advisor/contracts/` to hold the data classes for API contracts and result envelopes (e.g., `STEP_IMPORT_API_v1`, `ORCHESTRATION_RESULT_v0`). These will be typed using Python's `typing` and `dataclasses`.

3.  **Implement `contract_index_provider`**:
    - Create and implement `tooling_dfm_advisor/contracts/contract_index_provider.py`. This will contain a static dictionary mapping stage names to their corresponding module paths, acting as the security allowlist for the orchestrator.

4.  **Create Module Stubs**:
    - Create empty Python files for all agents, providers, and other modules (e.g., `orchestrator.py`, `thickness_agent.py`, `step_import_provider.py`).
    - Each file will contain basic class or function stubs with `pass` statements, docstrings indicating their purpose, and type hints for inputs and outputs that reference the newly created contracts. This makes the system structure explicit without requiring full logic implementation.

## Phase 3: Wiring and Basic Implementation

This phase focuses on implementing the core pipeline coordinator and ensuring modules can be loaded.

1.  **Implement the Orchestrator**:
    - Implement the core logic of the `orchestrator` as described in `WS_GOV-ORC-SPC-v1`. This includes:
        - Loading the `step_registry.json`.
        - Using `importlib` to dynamically and safely load modules based on the `contract_index_provider`.
        - Executing the stages in the correct canonical order.
        - Aggregating results into the `ORCHESTRATION_RESULT_v0` envelope.
        - Implementing fail-safe logic (e.g., catching exceptions from stages and marking them as `FAILED` without crashing).

2.  **Implement Fail-Safe Backends**:
    - Implement the `pythonocc_backend_provider` with its core fail-safe behavior. It must include a presence check for `pythonocc-core` and return a `NOT_EXECUTED` envelope if the dependency is missing.

3.  **Create Basic Tests**:
    - Set up a testing framework (e.g., `pytest`).
    - Create initial tests for bootstrap-critical components:
        - A test to ensure the `orchestrator` can load all registered stages.
        - A test to verify the `pythonocc_backend_provider` fails gracefully when `pythonocc-core` is not in the environment.
        - A test to ensure the `contract_index_provider` is complete and all its paths are resolvable.

## Phase 4: Documentation and Verification

This final phase focuses on user-facing documentation and a final check of the initialized repository state.

1.  **Create Project README**:
    - Create a `README.md` file in the root directory. It will include:
        - A brief description of the project.
        - Instructions for setting up the development environment (including the `pythonocc-core` dependency).
        - Instructions on how to run the pipeline via the orchestrator.
        - An overview of the directory structure.

2.  **Final Verification**:
    - Perform a final verification pass, producing a `VERIFICATION.md` report that:
        - Confirms that all directories and stub files from the plan have been created.
        - Lists all remaining stubs that require full implementation.
        - Explicitly records any unresolved contradictions or questions that arose during initialization, as required by the initial prompt.
