---
title: "ADR 0007: Deprecating Built-In Plugins in Favor of Core Adapter Modules"
tags: ["adr", "architecture", "plugins", "adapters"]
created_at: "2026-08-19"
last_updated_at: "2026-08-19"
---

# ADR 0007: Deprecating Built-In Plugins in Favor of Core Adapter Modules

## 1. Title
ADR 0007: Deprecating Built-In Plugins in Favor of Core Adapter Modules

---

## 2. Status
**Proposed**

---

## 3. Context
The legacy EDMC architecture treats core system integrations (such as Inara, EDSM, and EDDN telemetry outbound transmissions) as dynamically loaded plugins residing in the `plugins/` directory. This "pseudo-plugin" approach introduces several problems:
1. **Commingled Concerns:** Core application features are treated the same as third-party user extensions, complicating build distributions.
2. **Untestable Code:** Because plugins are imported dynamically at runtime via loop hooks in the main GUI window thread, they are highly coupled to the UI and difficult to unit test or mock.
3. **Noisy Coverage Reports:** The default testing suite is configured to track coverage of the `plugins/` folder (`--cov plugins`), generating empty data warnings and noisy coverage tables for unit test runs that do not invoke these dynamic scripts.

We need to establish a clean separation where core features are compiled system modules, and the `plugins/` folder is reserved strictly for third-party runtime expansions.

---

## 4. Decision
We will deprecate the use of the `plugins/` directory for core system integrations. We will decouple the `plugins/` folder from our active test validation suites and plan a transition to an event-driven adapter structure.

Specifically, we will:
1. Remove `--cov plugins` from [pyproject.toml](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/pyproject.toml)'s `addopts` coverage settings to eliminate coverage warnings and clear up local testing runs.
2. Formulate the target refactoring architecture:
   * **Core Event Broker:** A centralized publisher-subscriber engine (`src/edmc/core/broker.py`) that dispatches typed domain events (e.g. `FSDJumpEvent`).
   * **Compile-Time Adapters:** Relocate all core integrations into a structured module package: `src/edmc/adapters/` (e.g., `InaraAdapter`, `EddnAdapter`). These adapters will subscribe to the event broker and handle payload transmutation and transmission.
   * **Isolated Plugin Loader:** Keep the dynamic plugin folder loading system only for user-installed third-party scripts.

---

## 5. Consequences

### Positive:
* **Clean Test Output:** Eliminates the coverage warnings during `pytest` runs, keeping coverage statistics focused strictly on our active codebase in `src/`.
* **Decoupled Integrations:** Moving integrations into `src/edmc/adapters/` allows us to mock network layers easily and write fast, headless unit tests.
* **Separation of System and User Code:** Developers and users have a clear visual and logical boundary between what is core application code and what is an external plugin.

### Negative:
* Legacy core plugins will not be monitored for test coverage during the migration phase, though their functions remain runnable locally.

---

## 6. Procedure & Implementation

1. **Update `pyproject.toml`:** Edit the `[tool.pytest.ini_options]` table in [pyproject.toml](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/pyproject.toml) to remove the `--cov plugins` option.
2. **Execute Local Validation:** Run `pytest` locally to confirm the coverage report is now focused only on the active unit test suite, and verify that the `No data was collected` warning is resolved.
3. **Future Execution:** The full migration of the core scripts into `src/edmc/adapters/` will be planned and implemented in a subsequent refactoring step.
