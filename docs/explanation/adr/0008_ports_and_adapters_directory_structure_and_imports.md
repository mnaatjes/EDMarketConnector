---
title: "ADR 0008: Ports and Adapters Directory Structure and Imports"
tags: ["adr", "architecture", "packaging", "structure", "imports"]
created_at: "2026-08-19"
last_updated_at: "2026-08-19"
---

# ADR 0008: Ports and Adapters Directory Structure and Imports

## 1. Title
ADR 0008: Ports and Adapters Directory Structure and Imports

---

## 2. Status
**Proposed**

---

## 3. Context
We have established the target Ports & Adapters (Hexagonal) architecture for the EDMarketConnector refactoring project. To ensure clean boundaries and prevent this design from degrading into tightly coupled logic during development, we must establish strict rules for directory layout, package initialization, public module exposure, executable entry points, and sub-module encapsulation.

---

## 4. Decision
We will implement a structured package hierarchy under `src/edmc/`, enforce the Facade Pattern using `__init__.py` files as public gateways, configure modular execution entries, and isolate sub-module communication via abstract ports.

### 4.1 Directory Structure & Entry Points
The package directory will reside under `src/edmc/`. The entry points will be structured as follows:

* **Module Entry (`src/edmc/__main__.py`):** Holds the bootstrapping and initialization sequence of the application. Placing this file inside the package allows the application to be executed as a standard module:
  ```bash
  python -m edmc
  ```
* **Root Executor Wrapper (`run.py`):** A thin, single-line executable script located in the repository root directory. It imports and executes the entry point, facilitating easy CLI access for local runs:
  ```python
  from edmc.__main__ import main
  if __name__ == "__main__":
      main()
  ```

### 4.2 `__init__.py` Gatekeeping Rules (Facade Pattern)
To prevent internal details of a sub-module from leaking across packages:
1. Every sub-directory under `src/edmc/` must contain an `__init__.py` file.
2. The `__init__.py` file acts as a **Public API Gateway**. It must explicitly expose only the public interfaces, classes, and protocols of the sub-module using the `__all__` list.
3. Cross-module imports must target the parent package directory, not the internal implementation modules.
   * **Correct (Interface Target):**
     ```python
     from edmc.ingestion import JournalSource
     ```
   * **Incorrect (Internal Implementation Target):**
     ```python
     from edmc.ingestion.watcher import FileWatcher
     ```

### 4.3 Sub-Module Decoupling Rules
To maintain modularity at both the `src/edmc/` layer and within individual sub-module domains:
* **Abstract Ports:** All communication between different sub-packages must pass through abstract interfaces (protocols or abstract base classes) defined in a central `src/edmc/core/ports/` module or declared in the target package's gateway.
* **No Direct Adapter Imports:** A driven adapter (e.g. `src/edmc/egress/inara.py`) is strictly forbidden from importing from a driving adapter (e.g. `src/edmc/ingestion/watcher.py`). They must only import the abstract ports or standard domain models (`src/edmc/domain/`).
* **Dependency Injection:** Adapters are registered dynamically with the core dependency injection container at startup (configured in `__main__.py`).

---

## 5. Consequences

### Positive:
* **Enforced Encapsulation:** Changing internal class logic in a module (e.g., swapping a file watcher for a directory scanner) will not break other packages, as long as the public interface exposed in `__init__.py` remains unchanged.
* **Mockable Interfaces:** Sub-modules are completely isolated, making it easy to swap concrete adapters for mock implementations during unit testing.
* **Standard Packaging Compliance:** The project layout complies with modern PEP packaging guidelines.

### Negative:
* **Import Overhead:** Developers must maintain the `__all__` lists inside `__init__.py` facades.

---

## 6. Procedure & Implementation

1. **Initialize the Directories:** Create the empty directory tree under `src/edmc/` during the implementation phase.
2. **Implement Facades:** Add empty `__init__.py` files under each subdirectory, ready to be populated with export definitions.
3. **Create Entry Files:** Write `src/edmc/__main__.py` containing the main execution shell, and create `run.py` at the root.
