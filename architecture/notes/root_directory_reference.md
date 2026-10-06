---
title: "Root Directory Structure and File Reference"
tags: ["reference", "architecture", "audit", "notes", "taxonomy"]
created_at: "2026-10-06"
last_updated_at: "2026-10-06"
---

# Root Directory Structure and File Reference

This reference document provides an exhaustive, classified inventory of all 54 files and 11 legacy subdirectories situated in the repository root. It clarifies the operational purpose of each artifact within the legacy architecture and outlines its target modernization fate under Ports & Adapters (Hexagonal Architecture).

---

## 1. Directory Structure Inventory

| Directory Path | Purpose & Contents in Legacy Architecture | Primary Consumers | Target Modernization Fate |
| :--- | :--- | :--- | :--- |
| `architecture/` | Maintainer/engineering plane for Unified Process documentation (risk, ADRs, designs, use-cases, notes). | Engineering & AI agents | Retain as authoritative architectural SSoT. |
| `config/` | Platform configuration backends (`__init__.py`, `linux.py`, `windows.py`). Circularly coupled to root modules. | Global (`EDMarketConnector`, `monitor`, `plug`) | Refactor into decoupled configuration ports and platform adapters. |
| `coriolis-data/` | Submodule or cache directory for Coriolis ship/module definitions. | `coriolis-update-files.py` | Isolate under ingestion/data build tooling. |
| `docs/` | User and operator documentation plane (Diátaxis framework). | End users, packagers | Maintain as customer/operator plane. |
| `hotkey/` | Global hotkey listener modules (`__init__.py`, `linux.py`, `windows.py`). | `EDMarketConnector.py` | Encapsulate as platform UI input adapters. |
| `img/` | UI icon assets and documentation screenshots (`win.png`, `mac.png`, UI diagrams). | UI widgets, docs | Relocate UI assets to `resources/img/` or `ui/assets/`. |
| `L10n/` | Localization translation files (`.strings` format for CS, DE, ES, FR, IT, RU, etc.). | `l10n.py` | Encapsulate under `core/i18n/` or `resources/l10n/`. |
| `plugins/` | Built-in first-party adapters (`eddn.py`, `edsm.py`, `inara.py`, `edsy.py`, `coriolis.py`, etc.). | `plug.py` | Deprecate as dynamic plugins; migrate first-party syncs to `egress/` adapters. |
| `resources/` | Windows application manifests (`.manifest`), icon source files (`.xcf`), and Inno Setup templates. | `build.py`, `installer.py` | Retain as build and packaging asset repository. |
| `scripts/` | Maintainer shell scripts (`linux-setup.sh`, `mypy-all.sh`) and localization auditing scripts. | CI, developers | Keep isolated for devops and packaging automation. |
| `tests/` | Pytest test suites and legacy test runners. | CI/CD | Modernize into distinct `tests/unit/` and `tests/integration/`. |
| `util/` | Minor string utility functions (`text.py`). | `EDMarketConnector.py` | Consolidate into core utility modules. |

---

## 2. Python Code Modules (.py)

### 2.1 Application Entry Points & Core GUI Loop

| File Path | Description & Purpose | Primary Symbols / Classes | Coupling & Intertwined Systems | Target Fate |
| :--- | :--- | :--- | :--- | :--- |
| `EDMarketConnector.py` | God-module GUI bootstrapper. Initializes Tkinter main loop, builds menus, coordinates background threads, and dispatches plugin events. | `class A`, `class B` (dead scratch), `EVENT_KEYPRESS`, main UI functions. | Heavily coupled to `monitor`, `config`, `plug`, `theme`, `prefs`, `l10n`. | Split into decoupled `ui/` presentation views and `core/` application controllers. |
| `EDMC.py` | Headless command-line entry point. Runs telemetry capture and plugin events without spawning a Tkinter window. | `class DummyRoot`, CLI option parser. | Imports `monitor`, `config`, `edmc_data`, `plug`. | Port into `ui/cli.py` or `edmc.entrypoints.cli`. |
| `dashboard.py` | In-flight status file watcher. Tracks live player state flags (cargo, shields, pips, fuel) from Elite's `Status.json`. | `class Dashboard`, `class FileSystemEventHandler`. | Couples watchdog file polling directly to UI update loops. | Port into an ingestion adapter (`ingestion/status_watcher.py`). |
| `prefs.py` | Preferences configuration dialog window. Manages settings UI, credentials entry, and plugin settings tabs. | `class PreferencesDialog`, `class PrefsVersion`. | Tkinter widgets coupled directly to `config` read/writes and `plugin_browser`. | Extract settings view into `ui/views/preferences.py`. |
| `plugin_browser.py` | In-app browser UI allowing users to view, enable, and inspect installed third-party plugins. | `class PluginBrowserMixIn`. | Couples HTTP requests for plugin manifests to Tkinter notebook widgets. | Extract to `ui/views/plugin_browser.py`. |
| `stats.py` | CMDR status summary window. Displays commander ranks, statistics, and handles ship list CSV exports. | `class StatsDialog`, `class ShipRet`. | Couples CAPI raw dictionaries to Tkinter tabular displays and CSV writers. | Separate data parsing from `ui/views/stats.py`. |

### 2.2 Telemetry Ingestion & Core Processing

| File Path | Description & Purpose | Primary Symbols / Classes | Coupling & Intertwined Systems | Target Fate |
| :--- | :--- | :--- | :--- | :--- |
| `monitor.py` | Core Journal file watcher and parser. Polls log folders, detects active files, and emits events on journal lines. | `class EDLogs`, `class FileSystemEventHandler`, `class JournalHandler`. | Bundles watchdog filesystem notifications, JSON parsing, and plugin callbacks. | Split into `ingestion/journal_watcher.py` and `domain/journal_parser.py`. |
| `companion.py` | Frontier Developments Companion API (CAPI) client. Handles OAuth login, token refresh, and profile fetching. | `class Session`, `class CAPIData`, `class ServerError`. | Couples raw HTTP requests (`requests`) to auth state persistence and journal sync. | Refactor into `ingestion/capi/` adapter with typed domain models. |
| `edmc_data.py` | Public telemetry dictionary and constant definitions (flags, ship names, rank tables, outfitting IDs). | `Flags*`, `Flags2*`, `ship_name_map`, `commodity_map`. | Public contract imported across plugins, `monitor`, and UI. | Retain as backward-compatible facade; port definitions to typed `domain/enums.py`. |
| `journal_lock.py` | Multi-process file locking mechanism to prevent simultaneous reads/writes to journal files across processes. | `class JournalLock`, `class LockException`. | Uses OS-level Win32 / POSIX file locks. | Encapsulate under `core/storage/file_lock.py`. |
| `killswitch.py` | Remote emergency killswitch client. Fetches remote JSON rules to disable compromised features or plugins. | `class KillSwitches`, `class SingleKill`. | Fetches data over HTTP and modifies live plugin dispatch tables. | Move to `core/security/killswitch.py`. |
| `timeout_session.py` | Custom `requests.Session` factory with built-in default timeouts to avoid thread hanging on external HTTP calls. | `class TimeoutAdapter`, `class Session`. | Wraps `requests.adapters.HTTPAdapter`. | Move to `core/network/session.py`. |

### 2.3 Plugin Architecture

| File Path | Description & Purpose | Primary Symbols / Classes | Coupling & Intertwined Systems | Target Fate |
| :--- | :--- | :--- | :--- | :--- |
| `plug.py` | Plugin loader and hook dispatcher. Dynamically loads modules from `plugins/` and calls hook callbacks (`journal_entry`). | `class Plugin`, `class LastError`, `load_plugins`. | Loads both internal services (EDDN) and external third-party scripts. | Retain for third-party extensions; extract core first-party services to `egress/`. |

### 2.4 Data Export & Third-Party Integration

| File Path | Description & Purpose | Primary Symbols / Classes | Coupling & Intertwined Systems | Target Fate |
| :--- | :--- | :--- | :--- | :--- |
| `commodity.py` | Exports station market commodity data into various CSV, BPC, and Trade Dangerous formats. | `export()`, `bpc()`, `tsv()`. | Parses CAPI and Journal market payloads into tabular disk files. | Migrate to `egress/exporters/commodity.py`. |
| `outfitting.py` | Parses and exports station outfitting module inventories. | `lookup()`, `export()`. | Coupled to `edmc_data` maps and `config`. | Migrate to `egress/exporters/outfitting.py`. |
| `shipyard.py` | Exports station shipyard ship listings as CSV. | `export()`. | Coupled to `companion.CAPIData` and `edmc_data`. | Migrate to `egress/exporters/shipyard.py`. |
| `loadout.py` | Exports ship loadouts in CAPI JSON format for external loadout planners. | `export()`. | Parses CAPI ship definitions into JSON files. | Migrate to `egress/exporters/loadout.py`. |
| `edshipyard.py` | Exports ship loadouts formatted for the legacy EDshipyard web calculator. | `export()`. | String parsing of module names and stats. | Migrate to legacy exporter adapter. |
| `td.py` | Exporter for Trade Dangerous database integration. | `export()`. | Reads market data and formats system/station CSVs. | Migrate to `egress/exporters/trade_dangerous.py`. |

### 2.5 UI Presentation Widgets, Styling & Theming

| File Path | Description & Purpose | Primary Symbols / Classes | Coupling & Intertwined Systems | Target Fate |
| :--- | :--- | :--- | :--- | :--- |
| `theme.py` | UI color and theme manager. Manages dark/light palette overrides across Tk and ttk widgets. | `class _Theme`. | Manages styling workarounds for Windows Tkinter widgets. | Relocate to `ui/theme.py`. |
| `myNotebook.py` | Custom `ttk.Notebook` sub-classes resolving background artifact bugs on Windows. | `class Notebook`, `class Frame`, `class Label`, `class EntryMenu`. | Direct ttk widget subclassing. | Relocate to `ui/widgets/notebook.py`. |
| `ttkHyperlinkLabel.py` | Custom clickable hyperlink label widget for Tkinter. | `class HyperlinkLabel`. | Subclasses `tk.Label` / `ttk.Label` with browser hooks. | Relocate to `ui/widgets/hyperlink.py`. |
| `l10n.py` | Application localization engine. Loads translation strings from `L10n/*.strings`. | `class Translations`, `class _Locale`. | Custom Mac-style `.strings` parser; used globally. | Move to `core/i18n/engine.py`. |

### 2.6 System, Protocol & Platform Integration

| File Path | Description & Purpose | Primary Symbols / Classes | Coupling & Intertwined Systems | Target Fate |
| :--- | :--- | :--- | :--- | :--- |
| `protocol.py` | Custom URL protocol handler for `edmc://` OAuth callbacks. | `class GenericProtocolHandler`, `class WindowsProtocolHandler`, `class LinuxProtocolHandler`. | Uses Win32 `WndProc` hooks on Windows and local HTTP daemon on Linux. | Move to `platform/protocol/`. |
| `update.py` | Automatic update checker. Queries GitHub releases and WinSparkle endpoints. | `check_for_fdev_updates()`, `update_files()`. | Directly replaces files on disk on Windows. | Move to `core/updater/`. |
| `EDMCLogging.py` | Logging subsystem. Configures rotating file loggers, log levels, and custom formatters. | `class LogFormatter`, log configuration methods. | Global logging singleton imported everywhere. | Standardize under `core/logging/`. |
| `EDMCSystemProfiler.py` | Windows system profiler utility to generate diagnostic dumps of OS/hardware state for bug reports. | Diagnostic collection routines. | Windows-specific Win32 / WMI system inspections. | Move to `util/diagnostics.py`. |
| `common_utils.py` | General helper functions (path resolution, timestamp utilities). | Path and date formatting routines. | Imported across multiple core modules. | Move to `core/utils/`. |
| `constants.py` | General application constants (URLs, default timeouts, version metadata). | Constant definitions. | Imported by `config` and `update`. | Merge with core configuration constants. |
| `util_ships.py` | Ship naming and filename formatting utilities. | `ship_file_name()`. | String sanitizer used during ship loadout exports. | Consolidate into `domain/models/ship.py`. |

### 2.7 Build, Packaging & Maintenance Scripts

| File Path | Description & Purpose | Primary Symbols / Classes | Coupling & Intertwined Systems | Target Fate |
| :--- | :--- | :--- | :--- | :--- |
| `build.py` | Frozen binary build script (py2exe) for generating Windows executables. | `_discover_project_modules()`, build routines. | Configures py2exe bundling. | Maintain in build tooling; modernize with PyInstaller/PyOxidizer. |
| `installer.py` | Inno Setup compiler script for building Windows installer EXEs. | Inno Setup runner. | Invokes Inno Setup against `build.py` output. | Maintain in packaging scripts. |
| `collate.py` | Developer script to collate and extract localized strings across source files. | Translation extraction parser. | Scans `.py` files for translation tags. | Move to `scripts/collate.py`. |
| `coriolis-update-files.py` | Developer script to update local Coriolis module and ship data definitions. | Data fetcher and transformer. | Queries Coriolis GitHub repos. | Move to `scripts/coriolis_update.py`. |
| `debug_webserver.py` | Mock HTTP server used during development to inspect outbound telemetry payloads. | `class HTTPRequestHandler`. | Local debugging tool. | Move to `tests/mocks/` or testing SDK. |

---

## 3. Data Files & Configuration (.json, .toml, .txt)

| File Name | Purpose in Legacy Architecture | Primary Consumers | Target Modernization Fate |
| :--- | :--- | :--- | :--- |
| `modules.json` | Master lookup dictionary of ship outfitting modules (IDs, sizes, classes, mass). | `outfitting.py`, `edmc_data.py` | Relocate to `data/` or embed in domain package. |
| `ships.json` | Master lookup dictionary of ship models, internal slot layouts, and hull specs. | `shipyard.py`, `stats.py`, `edmc_data.py` | Relocate to `data/` or embed in domain package. |
| `pyproject.toml` | Modern Python packaging and tool configuration file (PEP 621, Ruff, Pytest). | Packaging & linters | Maintain as primary build and dependency configuration. |
| `requirements.txt` | Flat pip dependency list for base runtime dependencies. | pip, legacy CI | Replace with editable `pyproject.toml` dependencies. |
| `requirements-dev.txt` | Flat pip dependency list for developer tools (linters, pytest, py2exe). | pip, developers | Replace with `[project.optional-dependencies]` in `pyproject.toml`. |

---

## 4. UI Assets & Media (.ico, .png, .wav, .TTF)

| File Name | Purpose in Legacy Architecture | Primary Consumers | Target Modernization Fate |
| :--- | :--- | :--- | :--- |
| `EDMarketConnector.ico` | Main Windows application icon file. | Windows shell, Tkinter window | Relocate to `resources/icons/`. |
| `io.edcd.EDMarketConnector.png` | Linux desktop icon asset (PNG format). | Linux desktop entry | Relocate to `resources/icons/`. |
| `EUROCAPS.TTF` | Custom TTF font bundled for UI styling. | Tkinter UI styling | Relocate to `resources/fonts/`. |
| `snd_good.wav` | Audio chime played upon successful data sync. | Audio notification player | Relocate to `resources/sounds/`. |
| `snd_bad.wav` | Audio alert played upon network error or sync failure. | Audio notification player | Relocate to `resources/sounds/`. |

---

## 5. Shell & Operating System Integration (.bat, .desktop)

| File Name | Purpose in Legacy Architecture | Primary Consumers | Target Modernization Fate |
| :--- | :--- | :--- | :--- |
| `io.edcd.EDMarketConnector.desktop` | Linux FreeDesktop application launcher entry. | Linux desktop environments | Relocate to `resources/linux/`. |
| `EDMarketConnector - TRACE.bat` | Windows batch launcher running the app with verbose TRACE logging enabled. | Windows users debugging issues | Relocate to `scripts/windows/`. |
| `EDMarketConnector - localserver-auth.bat` | Windows batch launcher running the app using the local HTTP server auth handler. | Developers debugging CAPI | Relocate to `scripts/windows/`. |
| `EDMarketConnector - reset-ui.bat` | Windows batch launcher passing `--reset-ui` to restore default window geometry. | Users with off-screen windows | Relocate to `scripts/windows/`. |

---

## 6. Project Documentation & Governance (.md, LICENSE)

| File Name | Purpose in Legacy Architecture | Target Modernization Fate |
| :--- | :--- | :--- |
| `README.md` | Root landing document detailing the fork's mission, UP architecture, and Git branching rules. | Maintain as primary entry point for fork contributors. |
| `ChangeLog.md` | Upstream legacy changelog detailing historical version releases. | Retain as historical upstream reference. |
| `Contributing.md` | Upstream contributor guidelines and pull request instructions. | Retain and harmonize with fork contribution guidelines. |
| `PLUGINS.md` | Upstream documentation explaining the third-party plugin API and available hooks. | Maintain as plugin API reference. |
| `LICENSE` | GNU General Public License v2 (GPL-2.0) text. | Retain unmodified at repository root. |
