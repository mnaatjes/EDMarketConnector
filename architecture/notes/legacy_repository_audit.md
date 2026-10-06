---
title: "Legacy Repository Audit and Findings"
tags: ["audit", "legacy", "architecture", "findings"]
created_at: "2026-10-06"
last_updated_at: "2026-10-06"
---

# Legacy Repository Audit and Findings

This ledger consolidates the initial architectural review, static analysis metrics, and structural findings gathered from the upstream Elite Dangerous Market Connector (EDMC) fork prior to modular refactoring attempts.

---

## 1. Executive Summary

The upstream EDMC codebase is organized as a flat, single-tier Python project with root-level script execution. While functionally resilient across multiple game client versions, the legacy architecture exhibits high coupling between presentation, configuration, telemetry parsing, and third-party data egress.

### Key Architectural Bottlenecks Identified

1. **God-Module UI Controller (`EDMarketConnector.py`):**
   * Spans ~2,300+ lines in the repository root.
   * Commingles Tkinter UI loop lifecycle, background worker threads, direct filesystem access, plugin discovery hooks, and system tray integration.
   * Contains unreferenced dead code artifacts (e.g., `class A`, `class B`, `test_logging`).

2. **Commingled First-Party Plugins (`plugins/`):**
   * Core third-party telemetry sync adapters (`EDDN`, `EDSM`, `Inara`) are packaged identically to untrusted community user extensions inside `plugins/`.
   * Dynamic loading occurs synchronously via main-thread hooks during UI initialization, preventing isolated headless execution and unit test verification.

3. **Circular Configuration Architecture (`config/`):**
   * Central configuration manager (`config/__init__.py`) dynamically loads OS backends (`config/linux.py` or `config/windows.py`) based on runtime platform inspection (`sys.platform`).
   * The platform backends circularly import helper functions and global state back from `config/__init__.py`.

4. **Bundled Telemetry & Polling Logic (`monitor.py`):**
   * Journal log monitoring, filesystem directory polling, JSON record parsing, and event routing are tightly coupled in `monitor.py` and `edmc_data.py`.
   * Lack of distinct domain event models or contracts makes mocking game client event streams difficult without a live game instance.

---

## 2. Legacy Module Inventory & Responsibilities

| File Path | Role | Responsibilities | Key Dependencies / Coupling |
| :--- | :--- | :--- | :--- |
| `EDMarketConnector.py` | UI Bootstrapper & App Controller | Boots Tkinter window, manages menus, coordinates background threads, executes plugin lifecycle. | Couplings: `monitor`, `config`, `plugins`, `EDMCLogging`. |
| `EDMC.py` | CLI / Headless Runner | Alternative headless entry point for CLI telemetry capture. | Couplings: `monitor`, `config`, `edmc_data`. |
| `monitor.py` | Journal Event Listener | Watches journal directories, parses JSON log streams, emits UI/plugin callbacks. | Couplings: `config`, `edmc_data`, `companion`. |
| `config/__init__.py` | Global Configuration Manager | Exposes configuration singleton, resolves storage backends. | Circular coupling with `config/linux.py` and `config/windows.py`. |
| `config/linux.py` | Linux Storage Backend | File-based preference storage in user home/XDG configuration paths. | Imports `config`. |
| `config/windows.py` | Windows Storage Backend | Direct Windows Registry preference persistence via Win32 APIs. | Imports `config`. |
| `companion.py` | Frontier CAPI Client | Interacts with Frontier Developments Companion API (OAuth, profile retrieval). | Direct HTTP calls via `requests`, couples auth state to local filesystem. |
| `edmc_data.py` | Constants & Data Dictionaries | Definitions for ship parameters, outfitting, commodities, and Journal status flags. | Public API imported by external community plugins. |
| `EDMCLogging.py` | Central Logging Subsystem | Handles rotating file loggers and formatting overrides. | Imported globally across almost all modules. |
| `protocol.py` | Custom URL Protocol Handler | Registers and handles `edmc://` URL protocol invocation across OS platforms. | OS-specific win32 hooks vs. Linux HTTP daemon listeners. |
| `dashboard.py` | In-flight Status Dashboard | Polls and inspects game status flags (fuel, pips, docking). | Watchdog file listener coupled to UI state. |

---

## 3. Static Analysis & Code Quality Findings

### 3.1 Dead Code & Public API Boundary (Vulture Audit)

A dead-code audit via static analysis flagged multiple unreferenced symbols. These fall into three distinct classifications that govern safe deprecation:

* **Category A: Public API Constants & Flags (DO NOT REMOVE):**
  * `edmc_data.py`: `Flags*` (e.g., `FlagsDocked`, `FlagsShieldsUp`), `Flags2*` (Odyssey state flags), `GuiFocus*` constants. These are consumed externally by third-party community plugins. Removing them breaks downstream ecosystems.
  * `EDMCLogging.py`: `default_time_format`, `default_msec_format`.
* **Category B: Build & Development Helpers (PRESERVE):**
  * `build.py`: `_discover_project_modules`, `_classify_modules`. Used in py2exe/frozen build tooling.
  * `debug_webserver.py`: Overridden HTTP server handlers (`log_message`, `do_POST`).
* **Category C: True Dead Code Artifacts (SAFE TO DEPRECATE):**
  * `EDMarketConnector.py`: `test_logging` (line 2123), `class A` (line 2316), `class B` (line 2319), unreferenced variables `EVENT_KEYPRESS`, `EVENT_BUTTON`.
  * `EDMC.py`: Unreferenced exit codes `EXIT_SYS_ERR`, `EXIT_VERIFICATION`.

### 3.2 Complexity Hotspots (Radon Cyclomatic Complexity)

Key functions exhibiting high complexity and risk of side-effects:
* `EDMarketConnector.py`: Main window event handler and plugin hook dispatcher (Complexity Rank C / D in nested loop logic).
* `protocol.py`: `WndProc` (Complexity Rank B / 9) and `WindowsProtocolHandler.worker` (Rank B / 8).
* `dashboard.py`: `Dashboard.start` (Complexity Rank C / 13) and `Dashboard.poll` / `Dashboard.process` (Rank B / 6).
* `companion.py`: CAPI authentication state parsing and session refresh loops.

---

## 4. Preservation Guidelines for Future Architecture

When designing a modern object-oriented or Ports & Adapters architecture for this codebase:

1. **Retain Public Contract Boundaries:** Maintain compatibility layers or facade exports for `edmc_data` and plugin hooks so legacy community plugins continue to function.
2. **Decouple UI from Core Logic:** The core domain (telemetry handling, CAPI communication, configuration management) must be runnable 100% headless without Tkinter dependencies.
3. **Explicit Adapters for External Services:** Extract Inara, EDDN, and EDSM from dynamic `plugins/` into typed outbound adapters (`egress`).
4. **Mock SDK Dependency:** Headless CI/CD verification requires a deterministic journal event emitter (`MockJournalWriter`) to simulate game client telemetry without manual gameplay.
