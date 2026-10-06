---
title: "ADR 0002: Modernize Packaging, Streamline CI Workflows, and Partition Test Suite"
status: "proposed"
date: "2026-10-06"
tags: ["architecture", "adr", "packaging", "cicd", "testing", "pytest", "ruff"]
---

# ADR 0002: Modernize Packaging, Streamline CI Workflows, and Partition Test Suite

## 1. Context and Problem Statement

The repository currently relies on a fragmented build and testing setup:
1. **Packaging Fragmentation:** Python packaging is split between legacy requirements files (`requirements.txt`, `requirements-dev.txt`) and an unstandardized root `pyproject.toml`, creating confusion regarding canonical dependencies.
2. **CI Workflow Sprawl:** Continuous integration workflows in `.github/workflows/` (`pr-checks.yml`, `push-checks.yml`, `codeql.yml`, `windows-build.yml`) hardcode legacy `flake8` linters, multiple uncoordinated runners, and flat requirements installations.
3. **Flat Testing Structure:** The `tests/` directory contains 14 baseline test files mixed directly at the top level without distinction between legacy regression tests, future headless unit tests, and multi-adapter integration tests.

To establish the stable verification harness required by [ADR 0001](0001_hexagonal_architecture_and_multi_adapter_interfaces.md) and Fowler's refactoring workflow, we must clean-sheet the packaging and CI configurations while safeguarding the legacy regression test suite.

---

## 2. Decision Drivers

* **Verification Safety Net:** Ensure no existing behavior is broken during refactoring by executing the battle-tested legacy test suite.
* **Cognitive Clarity:** Eliminate confusing, duplicated build files and provide a clean, single source of configuration truth.
* **Standardization:** Adopt modern Python packaging (PEP 621) and consolidated tooling (`ruff`, `pytest`).
* **CI Determinism:** Provide a single, readable GitHub Actions workflow that executes in under two minutes across Linux and Windows.

---

## 3. Considered Options

* **Option 1: In-Place Patching.** Retain all `requirements*.txt` files and add more steps to legacy GitHub Actions scripts.
* **Option 2: Delete Legacy Tests and Start Empty.** Wipe the existing `tests/` directory and write only new tests.
* **Option 3: Clean-Sheet Packaging & CI with Partitioned Tests (Chosen).** Replace legacy requirements and root config with a modern `pyproject.toml`, streamline CI workflows into a unified `ci.yml`, and partition `tests/` into `tests/legacy/` and `tests/unit/`.

---

## 4. Decision Outcome

Chosen Option: **Option 3: Clean-Sheet Packaging & CI with Partitioned Tests**.

### 4.1 Canonical Packaging Configuration (`pyproject.toml`)
* Rebuild `pyproject.toml` using PEP 621 standard metadata (`setuptools` build-backend).
* Declare core runtime dependencies explicitly.
* Declare optional dependency groups:
  - `dev`: `pytest`, `ruff`, `mypy`.
  - `api`: `fastapi`, `uvicorn`.
  - `mcp`: `mcp`.
* Configure tool sections directly in `pyproject.toml`:
  - `[tool.ruff]`: Modern linting and formatting replacing `flake8` and `autopep8`.
  - `[tool.pytest.ini_options]`: Test discovery across all subdirectories with headless defaults.

### 4.2 Streamlined CI Pipeline (`.github/workflows/ci.yml`)
* Consolidate disparate PR and push workflows into a single, clean workflow: `.github/workflows/ci.yml`.
* Run on push to `develop` and pull requests.
* Matrix execution across Ubuntu and Windows.
* Two discrete verification gates:
  1. `ruff check .` & `ruff format --check .`
  2. `pytest tests/`

### 4.3 Test Suite Partitioning
To preserve baseline regression coverage while building the modern testing suite, restructure `tests/` as follows:

```text
tests/
|-- conftest.py            # Global fixtures and environment setup
|-- legacy/                # Preserved existing tests (test_config.py, test_l10n.py, etc.)
|-- unit/                  # Fast, isolated tests for src/edmc/
\-- integration/           # Multi-adapter and contract tests
```

---

## 5. Consequences

### Positive
* Single source of truth for dependencies in `pyproject.toml`.
* Existing legacy tests remain active and protect against regression bugs during class extraction.
* New unit tests for `src/edmc/` are cleanly isolated in `tests/unit/`.
* CI runs are fast, readable, and predictable across platforms.

### Negative / Trade-Offs
* Developers must use `pip install -e .[dev]` instead of legacy `pip install -r requirements.txt`.
