# TOOLING DFM ADVISOR

This repository contains the `tooling_dfm_advisor`, a system for running Design for Manufacturing (DFM) analysis on 3D CAD models.

The project has been initialized based on a set of canonical Word Spec (WS) documents, following a hybrid bootstrap process. The current state of the repository is a skeleton implementation with core governance, orchestration, and module stubs in place.

## Project Structure

The repository is organized into the following key directories:

- **`config/`**: Contains the `governance_policy.json` which drives the pipeline's behavior, thresholds, and rules.
- **`docs/`**:
    - **`audit/`**: Contains the append-only audit logs, session state, and tamper-evident receipts for every pipeline run.
    - **`runbooks/`**: Includes the `step_registry.json`, which defines all valid stages for the orchestration pipeline.
- **`tooling_dfm_advisor/`**: The main Python source code package.
    - **`orchestration/`**: The core pipeline coordinator.
    - **`contracts/`**: Defines the data contracts (DTOs) and the security allowlist for loadable modules.
    - **`providers/`**: Core services like STEP import and geometry validation.
    - **`agents/`**: The individual DFM analysis agents (e.g., thickness, draft).
    - **`reporter/`**: Components for composing and delivering the final analysis report.
- **`tests/`**: Contains tests for the application.
- **`WS_*.md` / `WS_*.docx`**: The original authority documents that define the system's architecture and requirements.

## Getting Started

### Prerequisites

1.  **Python**: A Python interpreter is required. The specific version should be pinned by the project's dependency manager.
2.  **pythonocc-core**: This is a critical but heavy dependency for geometry processing. The pipeline is designed to fail gracefully if it is not installed. Installation instructions can be found on the [pythonocc-core website](http://www.pythonocc.org/download/).
3.  **pytest**: For running the tests.

### Installation

1.  It is recommended to use a virtual environment.
2.  Install dependencies:
    ```bash
    # On Windows, you might need to use 'python -m pip' or 'py -m pip'
    pip install -r requirements.txt
    ```

### Running the Pipeline

The orchestration pipeline can be executed directly for testing purposes:

```bash
# On Windows, you might need to use 'python' or 'py'
python tooling_dfm_advisor/orchestration/orchestrator.py
```

This will run the bootstrap implementation of the orchestrator, which will dynamically load and execute the stub for each stage defined in the `step_registry.json`.

### Running Tests

To run the test suite:

```bash
pytest
```

## Current Status

This is a skeleton implementation. The core directory structure, configuration, and orchestration logic for dynamic module loading are in place. All provider and agent modules are stubs and require full implementation.
