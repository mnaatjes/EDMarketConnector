---
title: "ADR 0004: Isolate Legacy Dependencies for Greenfield Development"
tags: ["adr", "architecture", "packaging", "dependencies"]
created_at: "2026-08-19"
last_updated_at: "2026-08-19"
---

# ADR 0004: Isolate Legacy Dependencies for Greenfield Development

## 1. Title
ADR 0004: Isolate Legacy Dependencies for Greenfield Development

---

## 2. Status
**Proposed**

---

## 3. Context
As we transition from the auditing phase to the active refactoring phase, we are establishing a clean, modular target architecture inside a new `src/` directory. If we keep the legacy runtime requirements (e.g. `requests`, `pillow`, `watchdog`) inside the main `[project.dependencies]` block, the dependencies of the legacy flat application remain commingled with our new modular architecture. 

To ensure a clean, greenfield development namespace and prevent legacy package versions from cluttering the requirements of our new modular application, we need to segregate these dependencies at the packaging level.

---

## 4. Decision
We will isolate all legacy dependencies into dedicated, optional dependency groups inside [pyproject.toml](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/pyproject.toml). The base `[project.dependencies]` array will be kept empty until the new modular system requires runtime packages of its own.

Specifically, we will:
1. Empty the base `[project.dependencies]` array.
2. Move legacy core packages into `[project.optional-dependencies] legacy`.
3. Move legacy dev/testing/formatting tools into `[project.optional-dependencies] legacy-dev`.
4. Keep the modern auditing tools under `[project.optional-dependencies] review`.
5. Keep the modern linter/formatter/testing tools under `[project.optional-dependencies] dev` (e.g., `ruff`, `pytest`, `mypy`).
6. Update the legacy requirements files to route to these new groups:
   * [requirements.txt](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/requirements.txt) will contain `-e .[legacy]`.
   * [requirements-dev.txt](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/requirements-dev.txt) will contain `-e .[legacy-dev]`.

---

## 5. Consequences

### Positive:
* **Clean Greenfield Environment:** The root package installation (`pip install -e .`) installs only the new system.
* **Granular Dependency Targeting:** Developers can choose to install only the minimal modern dev dependencies (`pip install -e .[dev]`) or opt-in to the legacy systems (`pip install -e .[legacy-dev]`).
* **Preserved Compatibility:** Existing CI/CD pipelines and legacy scripts remain completely operational due to the requirement file redirects.

### Negative:
* **Slightly Verbose Commands:** Developers wishing to run both legacy and new development environments concurrently must install multiple extra groups (e.g. `pip install -e .[dev,legacy-dev]`).

---

## 6. Procedure & Implementation

1. **Update `pyproject.toml`:** Modify the dependency tables in [pyproject.toml](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/pyproject.toml) to implement this layout.
2. **Apply Redirect updates:** Update [requirements.txt](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/requirements.txt) to redirect to `-e .[legacy]`, and [requirements-dev.txt](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/requirements-dev.txt) to redirect to `-e .[legacy-dev]`.
3. **Verify:** Re-create the virtual environment and verify that executing legacy setups still succeeds via the redirected requirements files.
