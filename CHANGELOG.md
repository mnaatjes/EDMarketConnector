# Changelog - EDMarketConnector Review and Refactoring Project

This changelog records the milestones, accepted architectural changes, and documentation additions completed during the review and refactoring of the EDMarketConnector fork. Entries are in reverse-chronological order.

---

## [2026-08-19] - Initial Repository Review and Packaging Transition

This milestone represents the completion of the read-only static codebase audit, import dependency mapping, and packaging transition phase.

### Added
* **ADR 0001 (Accepted):** Documented and implemented the transition of core project metadata and configurations to `pyproject.toml` (PEP 621), creating the `review` extra dependency group.
* **ADR 0002 (Accepted):** Documented the deprecation strategy for legacy requirements files, replacing `requirements.txt` and `requirements-dev.txt` with editable package redirects pointing to `pyproject.toml`.
* **ADR 0003 (Accepted):** Documented the adoption of `ruff` as the unified CI/CD linter/formatter (replacing `flake8` and `autopep8`) and provided steps to prune excess packages from local virtual environments.
* **Diataxis Documentation Structure:**
  * [docs/explanation/packaging_and_cicd_transition.md](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/docs/explanation/packaging_and_cicd_transition.md): Explanation of packaging standards and the CI/CD pipeline roadmap.
  * [docs/how-to/environment_setup_and_code_audit.md](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/docs/how-to/environment_setup_and_code_audit.md): Guide for initializing virtual environments, installing packages, and running testing/auditing tools.
  * [docs/reference/legacy/dead_code_audit.md](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/docs/reference/legacy/dead_code_audit.md): Summary report of Vulture's dead-code findings, separating unused scratch codes from critical public API interfaces.
  * [docs/reference/legacy/component_roles.md](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/docs/reference/legacy/component_roles.md): Mapping of the technical roles and responsibilities of the core EDMC modules.
  * UML diagrams and dependency charts located under [docs/reference/legacy/](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/docs/reference/legacy/).

### Changed
* [pyproject.toml](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/pyproject.toml): Updated to declare `setuptools` as the build-backend, configure package dependencies, add `dev`/`review` extra groups, and append `ruff` configurations while preserving legacy testing/coverage configs.
* [requirements.txt](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/requirements.txt): Converted to a package redirect pointing to base `pyproject.toml` configuration.
* [requirements-dev.txt](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/requirements-dev.txt): Converted to a package redirect pointing to `dev` extra dependencies in `pyproject.toml`.
* [README.md](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/README.md): Rewritten to outline the fork's review/refactoring goals at the top while pushing legacy documentation into its own dedicated section.
* [GEMINI.md](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/GEMINI.md): Updated directives to specify the in-repository audit rules, Diataxis documentation ownership, and the global ADR authoring standards link.
