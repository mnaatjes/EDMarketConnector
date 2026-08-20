---
title: "ADR 0005: Pytest Restructuring and CI/CD Integration"
tags: ["adr", "architecture", "testing", "cicd"]
created_at: "2026-08-19"
last_updated_at: "2026-08-19"
---

# ADR 0005: Pytest Restructuring and CI/CD Integration

## 1. Title
ADR 0005: Pytest Restructuring and CI/CD Integration

---

## 2. Status
**Proposed**

---

## 3. Context
As we migrate the flat EDMC codebase into a modular `src/` layout, the legacy test scripts in the [tests/](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/tests/) directory (which import modules directly from the root namespace) will break. While we need to preserve these legacy tests as a behavioral reference for the application's functionality, they must not block the CI/CD pipeline of our new greenfield modular codebase.

Furthermore, we need to structure our testing approach to support:
1. **Unit Tests:** Fast, isolated checks testing single components headlessly with mocked I/O.
2. **Integration Tests:** Verifying combined module workflows (such as loading configuration backends or writing event streams).

---

## 4. Decision
We will restructure the [tests/](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/tests/) directory and update our Pytest configurations to isolate legacy tests and classify new validation suites.

Specifically, we will:
1. Create a `tests/legacy/` directory and move all existing legacy test files and folders (e.g. `tests/test_config.py`, `tests/killswitch.py/`) into it.
2. Create `tests/unit/` and `tests/integration/` directories.
3. Update [pyproject.toml](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/pyproject.toml)'s `[tool.pytest.ini_options]` table to only discover tests inside the new directories by default:
   ```toml
   testpaths = ["tests/unit", "tests/integration"]
   ```
4. Integrate the pipeline verification tasks into the CI/CD runs. The automated runners will only execute `pytest` on the greenfield suites (`tests/unit` and `tests/integration`). Developers can still run the legacy tests locally by executing:
   ```bash
   pytest tests/legacy/
   ```

---

## 5. Consequences

### Positive:
* **Clean CI/CD Pipeline:** The automated test runner checks only the new modular system's code, keeping builds green during the transition.
* **Separation of Concerns:** Test scripts are clearly separated by scope (isolated units vs integrated systems), facilitating faster test runs.
* **Behavior Reference Preserved:** Legacy verification code remains fully accessible in `tests/legacy/` for copy-paste migrations or reference.

### Negative:
* **Temporary Coverage Drop:** Since the legacy tests are excluded from the default test paths, global code coverage statistics will drop until the corresponding components are migrated and covered by new tests in `tests/unit/`.

---

## 6. Procedure & Implementation

1. **Create Directories:** Initialize `tests/legacy/`, `tests/unit/`, and `tests/integration/`.
2. **Relocate Legacy Files:** Move all pre-existing files and subdirectories from `tests/` into `tests/legacy/`.
3. **Configure Pytest:** Update `testpaths` inside [pyproject.toml](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/pyproject.toml) to target only `["tests/unit", "tests/integration"]`.
4. **Local Verification:** Run `pytest` to confirm that the test runner successfully ignores the legacy tests (exiting with a "no tests collected" status or passing on empty greenfield suites) while still allowing manual runs of the legacy tests using `pytest tests/legacy`.
