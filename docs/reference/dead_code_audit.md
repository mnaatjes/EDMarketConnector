---
title: "Dead Code and Public API Analysis Report"
tags: ["reference", "audit", "dead-code", "vulture"]
created_at: "2026-08-19"
last_updated_at: "2026-08-19"
---

# Dead Code and Public API Analysis Report

This reference document summarizes the findings from the static dead-code analysis run using `vulture` on the root repository (excluding virtual environments and test directories). 

The full raw data is stored in [docs/reference/dead_code_report.txt](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/docs/reference/dead_code_report.txt).

---

## 1. Classification Methodology

Not all code flagged as "unused" by Vulture is safe to delete. In a plugin-based architecture like EDMC, files are analyzed under three distinct classifications:

* **Category A: Public API Constants & Telemetry Flags (DO NOT DELETE)**
  * Variables, attributes, and classes designed to be imported or queried by third-party plugins. Since Vulture only parses this repository, it cannot detect usage in external plugins.
* **Category B: Developer & Build Script Helpers (PRESERVE)**
  * Helper functions or variables in scripts like `build.py` or `debug_webserver.py`. These are utilized during execution pipelines, testing, or development, rather than core runtime.
* **Category C: True Dead Code / Scratch Artifacts (SAFE TO DELETE)**
  * Local variables, obsolete test functions, or helper classes that have no external interface and are not referenced anywhere.

---

## 2. Key Findings & Categorization

### Category A: Public API Constants & Telemetry Flags (Preserve)
These parameters are exposed to the plugin ecosystem. Pruning them will break third-party integrations.

* **[edmc_data.py](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/edmc_data.py):**
  * `Flags*` (e.g. `FlagsDocked`, `FlagsShieldsUp`, `FlagsSupercruise`): Status flags representing ship states sent in the Journal log.
  * `Flags2*` (e.g. `Flags2OnFoot`, `Flags2LowOxygen`): Status expansion flags introduced in the Odyssey release.
  * `GuiFocus*` (e.g. `GuiFocusStationServices`, `GuiFocusGalaxyMap`): Constants indicating current UI focus.
* **[EDMCLogging.py](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/EDMCLogging.py):**
  * Custom log overrides like `default_time_format` and `default_msec_format`.

### Category B: Developer & Build Helpers (Preserve)
These functions exist inside setup/configuration scripts and should be maintained to support builds.

* **[build.py](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/build.py):**
  * `_discover_project_modules`: Legacy helper for package assembly.
  * `_classify_modules`: Internal build module classifier.
* **[debug_webserver.py](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/debug_webserver.py):**
  * Overridden handlers (`log_message`, `do_POST`) used during mock HTTP request testing.

### Category C: True Dead Code / Scratch Artifacts (Safe to Delete)
These are candidate locations for code removal during the refactoring phase.

* **[EDMarketConnector.py](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/EDMarketConnector.py):**
  * `test_logging` (Line 2123): An unused function that appears to be an obsolete debug runner.
  * `class A` (Line 2316) & `class B` (Line 2319): Scratch class definitions left in the main runner script.
  * `EVENT_KEYPRESS` (Line 509) & `EVENT_BUTTON` (Line 510): Unreferenced variable assignments.
* **[EDMC.py](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/EDMC.py):**
  * `EXIT_VERIFICATION` & `EXIT_SYS_ERR` (Line 56): Unreferenced exit code integers.
