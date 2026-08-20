---
title: "ADR 0002: Deprecating requirements.txt Files in Favor of pyproject.toml Supremacy"
tags: ["adr", "architecture", "packaging", "dependencies"]
created_at: "2026-08-19"
last_updated_at: "2026-08-19"
---

# ADR 0002: Deprecating requirements.txt Files in Favor of pyproject.toml Supremacy

## 1. Title
ADR 0002: Deprecating requirements.txt Files in Favor of pyproject.toml Supremacy

---

## 2. Status
**Proposed**

---

## 3. Context
Following the acceptance of [ADR 0001](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/docs/explanation/adr/0001_pyproject_and_cicd_transition.md), we have initiated the transition to a declarative `pyproject.toml` file. We now need to address the developer dependencies defined in the legacy [requirements-dev.txt](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/requirements-dev.txt). 

Maintaining duplicate dependency lists across both `pyproject.toml` and flat text files (`requirements.txt`, `requirements-dev.txt`) introduces configuration drift. However, completely deleting these legacy requirements files immediately is high-risk, as it would instantly break:
1. Automated CI/CD runner environments that call `pip install -r requirements-dev.txt`.
2. Local scripts and third-party setups built on historical guidelines.
3. Pre-commit configuration file filters.

We must decide whether to retain these text files indefinitely, delete them immediately, or institute a deprecation strategy that bridges the transition.

---

## 4. Decision
We will establish **pyproject.toml supremacy** by consolidating all dependency definitions within the single project file. To avoid breaking legacy systems, we will implement a **Phased Deprecation using Legacy Redirects**.

Specifically, we will:
1. Migrate all dependencies from [requirements-dev.txt](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/requirements-dev.txt) into `pyproject.toml` under the `[project.optional-dependencies] dev` key.
2. Replace the contents of [requirements.txt](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/requirements.txt) with a single-line package redirect:
   ```text
   -e .
   ```
3. Replace the contents of [requirements-dev.txt](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/requirements-dev.txt) with a single-line package redirect:
   ```text
   -e .[dev]
   ```
4. Configure all future development workflows and documents to advocate for `pip install -e .[dev]` or `pip install -e .[review]`.

---

## 5. Consequences

### Positive:
* **Single Source of Truth:** All project configurations, packaging settings, and dependency definitions (production, dev, review) are located inside `pyproject.toml`.
* **Zero Script Breakage:** Any script, hook, or pipeline executing `pip install -r requirements-dev.txt` continues to function. The redirect instructs `pip` to read and install dependencies from the `[project.optional-dependencies] dev` section of `pyproject.toml`.
* **Clean Deprecation Path:** Legacy files can be safely deleted in a future major version release once all CI/CD workflows are updated.

### Negative:
* **Editable Install Requirement:** Using `-e` redirects in the requirements files requires pip to run in editable mode, which generates an `.egg-info` or `.dist-info` reference in the repository directory (already excluded via `.gitignore`).

---

## 6. Procedure & Implementation

1. **Populate `pyproject.toml`:** Add the full lists of linting, testing, and formatting tools from [requirements-dev.txt](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/requirements-dev.txt) into `[project.optional-dependencies] dev` inside [pyproject.toml](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/pyproject.toml).
2. **Apply Redirects:** Edit [requirements.txt](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/requirements.txt) and [requirements-dev.txt](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/requirements-dev.txt) to overwrite their contents with the respective `-e .` and `-e .[dev]` targets.
3. **Local Testing:** Test the redirects by creating a clean virtual environment and running:
   ```bash
   pip install -r requirements-dev.txt
   ```
   Verify that all dev dependencies and testing frameworks (`pytest`) install correctly.
4. **Update Documentation:** Update setup instructions in [docs/how-to/environment_setup_and_code_audit.md](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/docs/how-to/environment_setup_and_code_audit.md).
