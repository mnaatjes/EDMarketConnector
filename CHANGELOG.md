# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

---

## [Unreleased]

### Session Metadata
* **Antigravity Conversation UUID:** `c03c325a-62b8-455e-8237-de7969300127`
* **Strategic Milestone:** Concluded in-depth repository audit, root directory taxonomy, and test baseline isolation. Transitioning to a dedicated, decoupled sibling repository (`ed-telemetry` / `edmc-core`) overseen from parent workspace directory `~/src/github.com/mnaatjes/` to maintain this repository as an authoritative domain reference manual.

---

## [2026-10-06] - Architecture Governance, Root Audit & Packaging Baseline

### Completed
* **Unified Process Governance:**
  * Initialized `architecture/` engineering plane adhering to UP dual-root standards (`risk/`, `use-cases/`, `adr/`, `designs/`, `rfcs/`, `api/`, `notes/`).
  * Consolidated legacy codebase audit in [`architecture/notes/legacy_repository_audit.md`](architecture/notes/legacy_repository_audit.md).
  * Cataloged all 54 root files and 11 directories in [`architecture/notes/root_directory_reference.md`](architecture/notes/root_directory_reference.md).
  * Formalized Fowler's *Extract Class* and *Encapsulate Implementation* workflow guide in [`architecture/notes/fowler_refactoring_patterns_workflow.md`](architecture/notes/fowler_refactoring_patterns_workflow.md).
* **Decisions & Design Specifications:**
  * **ADR 0001 (Accepted):** Hexagonal Architecture, Multi-Adapter Interfaces, and CI/CD Testing Modernization.
  * **ADR 0002 (Accepted):** Modernize Packaging, Streamline CI Workflows, and Partition Test Suite.
  * **SDD-001 (Draft/In-Progress):** Hexagonal Architecture Core and Multi-Adapter Topology (Phases 1–5).
  * **SDD-002 (Completed):** Packaging Modernization, Test Partitioning, and CI Pipeline Topology.
* **Packaging & Test Infrastructure (SDD-001 Phase 1 / SDD-002):**
  * Clean-sheet [`pyproject.toml`](pyproject.toml) configured with PEP 621 metadata, dependencies, extras (`dev`, `api`, `mcp`), Ruff linter/formatter, and Pytest configuration.
  * Redirected `requirements.txt` and `requirements-dev.txt` to editable package installations.
  * Partitioned `tests/` into `tests/legacy/` (protecting 129 baseline regression tests) and initialized clean `tests/unit/`.
  * Integrated `xvfb` headless test execution for Tkinter components in headless CI/Linux environments.
  * Replaced sprawling PR/push workflows with a single unified matrix workflow in [`.github/workflows/ci.yml`](.github/workflows/ci.yml).

### Strategic Transition
* **Next Action:** Repository frozen as an immutable reference model. Next development session will launch from workspace root `~/src/github.com/mnaatjes/` to create and supervise the greenfield decoupled architecture project.
