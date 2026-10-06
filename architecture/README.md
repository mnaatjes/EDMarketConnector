---
title: "Architecture Directory Governance and Contract"
tags: ["architecture", "governance", "up"]
created_at: "2026-10-06"
last_updated_at: "2026-10-06"
---

# Architecture Directory Governance and Contract

This directory serves as the authoritative Single Source of Truth (SSoT) for internal technical governance, requirements specifications, architectural trade-offs, and implementation blueprints for this repository under the Unified Process (UP).

---

## 1. Dual-Root Partition

* **Maintainer / Engineering Plane (`architecture/`):** Reserved exclusively for internal engineering ledgers, formal requirements, design documents, and decision records.
* **Customer / Operator Plane (`docs/`):** Reserved exclusively for living, mutable user-facing manuals and runbooks governed by the Diátaxis framework.

---

## 2. Canonical Directory Structure

```text
architecture/
|-- README.md                      # Directory contract & governance index (this document)
|-- notes/                         # Legacy repository audit and preserved findings
|-- risk/                          # Project Management Discipline (Master Risk Register)
|-- use-cases/                     # Requirements Discipline (RUP Use-Case Specifications)
|-- adr/                           # Analysis & Design Discipline (MADR 3.0 Decision Records)
|-- designs/                       # Analysis & Design / Implementation (IEEE 1016 SDDs)
|-- rfcs/                          # Requirements & Architectural Consensus Proposals
\-- api/                           # Interface & Boundary Contract Specifications
```

---

## 3. Directory Contracts

* [`risk/`](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/architecture/risk/README.md): Barry Boehm Master Risk Register ($RE = P \times I$).
* [`use-cases/`](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/architecture/use-cases/README.md): Cockburn Sea-Level RUP Use-Case Specifications.
* [`adr/`](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/architecture/adr/README.md): Immutable Markdown Architectural Decision Records (MADR 3.0).
* [`designs/`](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/architecture/designs/README.md): Software Design Documents adhering to IEEE 1016 / Google Design Doc standards.
* [`rfcs/`](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/architecture/rfcs/README.md): Collaborative engineering proposals evaluating alternative trade-offs.
* [`api/`](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/architecture/api/README.md): Boundary interface contracts (OpenAPI 3.1 / AsyncAPI).
* [`notes/`](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/architecture/notes/legacy_repository_audit.md): Historical repository audits and findings.
