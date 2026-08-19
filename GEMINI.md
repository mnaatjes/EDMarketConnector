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
* **`docs/explanation/`**: High-level background concepts, design rationales, architectural critiques, and Architectural Decision Records (ADRs).

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

---

**Active Conversation UUID**: 854aafe7-4a26-4761-b59b-074b9c871b80
