# Directives and Context for Repository Review and Refactoring

This document defines the structural directives and context guidelines for the EDMarketConnector fork review, documentation, and refactoring process.

---
## 1. Purpose and Structure

This repository is a fork of the official Elite Dangerous Market Connector.

* **Goal:** Review, Map, and Understand the Components and Structure of this EDMC repository.
  1. Identify all necessary components.
  2. Audit structure and attempt to rationalize it.
  3. Engineer a better solution for EDMC on this fork.

* **Development & Refactoring Plan:**
  * **In-Repository Audit:** The audit and initial review will be conducted directly inside this repository on dedicated branches (e.g., `review/structure-mapping`) keeping the legacy codebase in its original flat layout to preserve runnability and facilitate analysis.
  * **Future Modularization:** The legacy application was built with a flat structure. Our target is to modularize the application by progressively refactoring components into a clean `src/` directory.

## 2. Diataxis Documentation Structure Layout
All documentation files under `docs/` must be structured into the following Diataxis directories. Note that these directories are reserved exclusively for our review and new architecture documentation, distinct from any pre-existing legacy documentation:
* **`docs/tutorials/`**: Guided learning pathways for beginners.
* **`docs/how-to/`**: Practical, step-by-step guides to solve specific tasks or execute runbooks.
* **`docs/reference/`**: Technical descriptions, architecture specifications, schemas, standards, and codebase maps generated during the audit.
* **`docs/explanation/`**: High-level background concepts, design rationales, architectural critiques, and Architectural Decision Records (ADRs). All ADRs must follow the format and rules defined in the global [ADR Authoring Standards](file:///home/michael/.gemini/config/adr_authoring.md).

## 3. Metadata Frontmatter Syntax
All documentation pages must include the following YAML frontmatter metadata format at the top of the file:
```yaml
---
title: "Document Title"
tags: ["tag-name-1", "tag-name-2"]
created_at: "YYYY-MM-DD"
last_updated_at: "YYYY-MM-DD"
---
```

## 4. Review Tooling Directives
Systematic review of the existing codebase requires analysis and visualization tooling. The following tools will be used during the audit:
* `pydeps` for visual import dependency graphing.
* `radon` for code complexity metrics and maintainability indexes.
* `vulture` for identifying dead or unused code paths.

## 5. Architectural Directives
To modularize and modernize the EDMarketConnector codebase, all refactoring work must adhere to the following design directives:
1. **Single Responsibility Principle (SRP):** Each module or class must focus on one isolated concern (e.g., separating journal file parsing from network request handling) to limit the ripple effects of code changes.
2. **Inversion of Control (IoC):** Dependencies (such as platform configurations) must be injected into components dynamically rather than hardcoding static imports, eliminating circular dependencies.
3. **Ensure Maintainability and Scalability:** Write clean module interfaces with decoupled boundaries, making it simple to write isolated unit tests and add new features.
4. **Decouple Business Logic:** Isolate core data transformations, log parsing rules, and mathematical operations from infrastructural dependencies like local filesystems or remote servers.
5. **Decouple Presentation Layer (GUI):** Keep the Tkinter interface clean of business processing. The GUI should communicate solely via event dispatchers or injected controllers, allowing the application to run headlessly.
6. **Domain-Driven Design (DDD) & Event Sourcing:** Represent Journal telemetry events as structured, immutable domain models that are processed and dispatched as event streams to API consumers.
7. **Ports & Adapters (Hexagonal) Architecture:** Segregate the codebase into a dependency-free Core Domain (core/, domain/) and external, pluggable Adapters (ingestion/, egress/, ui/). Direct dependencies between adapters are strictly forbidden; communication must pass through abstract ports defined in the core.

---

**Active Conversation UUID**: 854aafe7-4a26-4761-b59b-074b9c871b80
