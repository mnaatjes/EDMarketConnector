# Elite: Dangerous Market Connector (EDMC) - Fork & Refactor Project

This repository is a fork of the official Elite: Dangerous Market Connector (EDMC). It is dedicated to auditing the codebase, mapping its architecture, and systematically refactoring it into a clean, modern, and modularized layout.

---

## 1. Project Objectives & Current Status

The work in this repository is currently focused on an **In-Repository Audit** to understand the components and structure before making major changes.

* **Goals:**
  1. **Audit & Map:** Identify all core components, event paths, and background workers in the legacy flat structure.
  2. **Modernize Packaging:** Transition dependency management to a declarative, PEP 621-compliant `pyproject.toml` file to support automated quality checks.
  3. **Refactor & Modularize:** Progressively migrate the flat application logic from the repository root into a structured `src/` directory.

* **Active Directives:**
  * Refer to [GEMINI.md](GEMINI.md) for the active development directives and conventions.
  * Architectural decisions are documented in the [docs/explanation/adr/](docs/explanation/adr/) directory, starting with [ADR 0001: Transition to pyproject.toml and CI/CD Foundation](docs/explanation/adr/0001_pyproject_and_cicd_transition.md).
  * System documentation follows the Diataxis standard and is located in the [docs/](docs/) folder.

---

## 2. Target Architecture (Ports & Adapters / Hexagonal)

To decouple the codebase, we are restructuring the application into a **Ports & Adapters (Hexagonal)** layout under the `src/` directory.

### Directory Geography:
```text
src/
└── edmc/
    ├── core/            # System hub, Event Broker, Bootstrapper, and Dependency Injection container.
    ├── domain/          # Pure, dependency-free domain models (e.g. JournalEvent, CmdrState).
    ├── ingestion/       # Driving Adapters (File Watcher, Log Line Tailer, JSON Parser).
    ├── egress/          # Driven Adapters (Inara, EDSM, EDDN HTTP/ZMQ API clients).
    └── ui/              # User Interface Adapter (Tkinter GUI windows and menus).
```

* **Core Domain (`core/`, `domain/`):** Contains the business rules and event orchestration. It has no dependencies on external frameworks, networking libraries (`requests`), or graphics engines (`tkinter`).
* **Adapters (`ingestion/`, `egress/`, `ui/`):** Implement the specific interfaces (ports) to interact with external systems (filesystems, network APIs, displays). They only communicate with each other via the core Event Broker.

---

## 3. Legacy EDMarketConnector Project Description

*Below is the original documentation for the Elite: Dangerous Market Connector project.*

---

Any questions or offers of help can be directed to the EDCD Discord #edmc channel:

[![Discord chat](https://img.shields.io/discord/164411426939600896.svg?style=social&label=Discord%20chat)](https://discord.gg/usQ5e6n)

### Elite: Dangerous Market Connector (EDMC)

This application is only of use to PC players of the game Elite Dangerous (and its expansions). It won't work with PS4 or Xbox accounts.

It utilises the Journal files written by the game on the user's computer, together with data from the API Frontier Developments supplies in order to feed this data to various third party sites that the user may find useful.

See [the Wiki documentation](https://github.com/EDCD/EDMarketConnector/wiki) for more details.

### Installation & Uninstall
Please see the [Installation & Setup](https://github.com/EDCD/EDMarketConnector/wiki/Installation-&-Setup) wiki page.

### Running from source
Please see the [Running from source](https://github.com/EDCD/EDMarketConnector/wiki/Running-from-source) wiki page.

### Plugins
Plugins extend the behaviour of this app. See the [Plugins](https://github.com/EDCD/EDMarketConnector/wiki/Plugins) wiki page for more information.

If you would like to write a plugin please see [PLUGINS.md](PLUGINS.md).

### Troubleshooting
Please see the [Troubleshooting](https://github.com/EDCD/EDMarketConnector/wiki/Troubleshooting) wiki page.

### Reporting a problem
Please report a problem as a new GitHub [issue](https://github.com/EDCD/EDMarketConnector/issues/new?assignees=&labels=bug%2C+unconfirmed&template=bug_report.md&title=).
See [Reporting a problem](https://github.com/EDCD/EDMarketConnector/wiki/Troubleshooting#reporting-a-problem) for further guidance, including how to find the necessary log files to attach to the report.

### Packaging for distribution
Please see [docs/Releasing.md](docs/Releasing.md).

### Disclaimer
This app uses the “Companion” web API that Frontier originally supplied for their Elite Dangerous iOS app and now [support](https://forums.frontier.co.uk/threads/open-letter-to-frontier-developments.218658/page-19#post-3371472) for third-party apps. If that API ceases to function in the future then much of this application's functionality will be curtailed (although it could still utilise [Journal files](https://forums.frontier.co.uk/threads/commanders-log-manual-and-data-sample.275151/#post4562494)).

### Acknowledgements
Please see the [Acknowledgements](https://github.com/EDCD/EDMarketConnector/blob/main/docs/Acknowledgements%20and%20License.md) wiki page.

### License
Copyright © 2015-2019 Jonathan Harris, 2020-2024 EDCD

Licensed under the [GNU Public License (GPL)](http://www.gnu.org/licenses/gpl-2.0.html) version 2 or later.
