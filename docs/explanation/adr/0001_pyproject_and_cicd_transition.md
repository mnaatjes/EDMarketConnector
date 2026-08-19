---
title: "ADR 0001: Transition to pyproject.toml and CI/CD Foundation"
tags: ["adr", "architecture", "packaging", "cicd"]
created_at: "2026-08-19"
last_updated_at: "2026-08-19"
---

# ADR 0001: Transition to pyproject.toml and CI/CD Foundation

## 1. Title
ADR 0001: Transition to pyproject.toml and CI/CD Foundation

---

## 2. Status
**Accepted**

---

## 3. Context
The Elite Dangerous Market Connector (EDMC) project currently uses a legacy environment configuration based on flat files like `requirements.txt` and `requirements-dev.txt`. Python packaging standards have evolved with **PEP 517**, **PEP 518**, and **PEP 621**, introducing declarative project metadata and unified build-system declarations via `pyproject.toml`. 

To conduct a systematic codebase audit and build an automated testing and code complexity checking pipeline, we need to install specialized tools (`radon`, `vulture`, `pydeps`). Declaring these dependencies in separate requirements files or installing them ad-hoc leads to environment drift between developers and CI/CD environments. We require a unified packaging manifest that preserves legacy tool settings while enabling new verification pipelines.

---

## 4. Decision
We will transition the package configuration to a modern, PEP 621-compliant layout inside `pyproject.toml`. This transition will be executed while keeping the legacy build/run behaviors functional.

Specifically, we will:
1. Declare `setuptools` as our build backend inside the `[build-system]` table.
2. Define core project metadata and project dependencies under the `[project]` table, mirroring the contents of the current `requirements.txt`.
3. Establish a `review` dependency group under `[project.optional-dependencies]` (extras) containing our code audit and static analysis tools: `radon`, `vulture`, `pydeps`, and `pylint`.
4. Keep the existing legacy sections (`[tool.autopep8]`, `[tool.pytest.ini_options]`, `[tool.coverage.run]`, etc.) intact at the bottom of the file.

---

## 5. Consequences

### Positive:
* **Unified Tooling Configuration:** The build backend, package metadata, dependencies, and linting/testing configurations are now defined in a single file (`pyproject.toml`).
* **Isolated Auditing Installs:** Developers and CI runners can install the audit tools dynamically using a standard command:
  ```bash
  pip install -e .[review]
  ```
* **No Functional Breakage:** The legacy configurations are fully preserved, preventing any disruptions to current setups or workflows.

### Negative:
* **Dependency List Duplication:** During this transitional phase, dependencies will be defined both in `requirements.txt` and the `pyproject.toml` file to prevent breaking older scripts that explicitly read from `requirements.txt`.

### Neutral:
* Downstream packaging scripts will continue functioning normally as long as they run python files in-place or read from the flat text files.

---

## 6. Procedure & Implementation

1. **Update `pyproject.toml`:** Add the metadata, build system configuration, and review dependencies to [pyproject.toml](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/pyproject.toml).
2. **Local Validation:** Create an isolated virtual environment, run the install command with the `review` extra group, and run the static analysis utilities:
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   pip install -e .[review]
   radon cc . -a -s
   vulture .
   ```
3. **CI/CD Integration:** Set up future GitHub Actions workflows to bootstrap environments directly using the new optional dependency groups.
