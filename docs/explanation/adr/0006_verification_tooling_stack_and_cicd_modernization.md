---
title: "ADR 0006: Verification Tooling Stack and CI/CD Modernization"
tags: ["adr", "architecture", "tooling", "cicd", "packaging"]
created_at: "2026-08-19"
last_updated_at: "2026-08-19"
---

# ADR 0006: Verification Tooling Stack and CI/CD Modernization

## 1. Title
ADR 0006: Verification Tooling Stack and CI/CD Modernization

---

## 2. Status
**Proposed**

---

## 3. Context
We have updated the repository directives in [GEMINI.md](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/GEMINI.md) to define six architectural guidelines (SRP, IoC, Maintainability, Business Logic Decoupling, UI Separation, and DDD/Event Sourcing). To guarantee that future contributions adhere to these guidelines, we need to transition our verification tools and automate checks within our CI/CD pipelines.

The current CI/CD configuration under [.github/workflows/](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/.github/workflows) is legacy, hardcoding installation steps for `flake8` and `pytest` using flat requirements files. We need to formalize a unified verification stack, configure it inside `pyproject.toml`, and update the workflows directory to run these modern checks.

---

## 4. Decision
We will adopt a standardized verification tooling stack explicitly mapped to our architectural directives, configure these tools inside [pyproject.toml](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/pyproject.toml), and update the GitHub Actions configurations.

### 4.1 Tool-to-Directive Matrix
* **Ruff (`ruff check` & `ruff format`):** Enforces styling and code quality guidelines (preventing circular dependencies and logical styling bugs).
* **Mypy (`mypy`):** Enforces strict interface boundaries for Dependency Injection (IoC) and validates Domain Models (DDD).
* **Pytest (`pytest` & `pytest-cov`):** Validates Single Responsibility boundaries (SRP) and ensures coverage of isolated business logic.
* **Radon (`radon`):** Limits cyclomatic complexity to guarantee maintainability.
* **Vulture (`vulture`):** Scans for unused code to prevent logic bloat.
* **Pyreverse / Pydeps:** Visually audits package coupling and UI-to-business-logic segregation.

### 4.2 `pyproject.toml` Updates
We will warehouse these tools under the `[project.optional-dependencies]` tables:
* `dev`: Contains `ruff`, `pytest`, `pytest-cov`, and `mypy` (the active verification suite).
* `review`: Contains `radon`, `vulture`, `pydeps`, and `pylint` (for passive audits).

We will also add configuration tables for `ruff` and `mypy` directly to [pyproject.toml](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/pyproject.toml):
```toml
[tool.ruff]
line-length = 120
lint.select = ["E", "F", "I", "N"]

[tool.mypy]
python_version = "3.10"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
```

### 4.3 CI/CD Workflow Updates
We will modernize the workflow YAML files under [.github/workflows/](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/.github/workflows) by replacing legacy pip installation calls with modern target installations.

For example, in [pr-checks.yml](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/.github/workflows/pr-checks.yml):
```yaml
# BEFORE (Legacy requirements installation)
- name: Install dependencies
  run: pip install -r requirements-dev.txt

- name: Lint with flake8
  run: flake8 .

# AFTER (Modernized targets)
- name: Install verification stack
  run: pip install -e .[dev]

- name: Lint and Format check with Ruff
  run: ruff check . && ruff format --check .

- name: Type check with Mypy
  run: mypy src/
```

---

## 5. Consequences

### Pipeline State: Before vs. After

| Stage | Before (Legacy) | After (Modernized) |
| :--- | :--- | :--- |
| **Dependency Resolution** | Installs from flat `requirements-dev.txt`. | Installs directly from `pyproject.toml` via `pip install .[dev]`. |
| **Linting & Formatting** | Runs `flake8` (Python-based, slow) with 8 separate plugins and `autopep8`. | Runs `ruff` (Rust-based, extremely fast), consolidating all checks in one command. |
| **Type Checking** | No automated `mypy` check run in PR actions. | Runs automated `mypy` on `src/` to block untyped code merges. |
| **Testing** | Runs `pytest` on all tests commingled in the root `tests/` directory. | Runs `pytest` only on modular suites in `tests/unit/` and `tests/integration/`. |

---

## 6. Procedure & Implementation

1. **Configure `pyproject.toml`:** Add tool tables for `ruff`, `mypy`, and coverage options to [pyproject.toml](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/pyproject.toml).
2. **Update Workflow Files:** Edit [.github/workflows/pr-checks.yml](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/.github/workflows/pr-checks.yml) and [push-checks.yml](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/.github/workflows/push-checks.yml) to run Ruff and Mypy instead of Flake8, and point the test triggers to the new virtual environment setup.
3. **Verify locally:** Run `ruff check .` and `mypy` locally in the virtual environment to ensure baseline configurations are clean before pushing.
