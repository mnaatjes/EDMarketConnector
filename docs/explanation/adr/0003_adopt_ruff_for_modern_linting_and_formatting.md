---
title: "ADR 0003: Adopt Ruff for Modern Linting and Formatting"
tags: ["adr", "architecture", "linting", "formatting", "tooling"]
created_at: "2026-08-19"
last_updated_at: "2026-08-19"
---

# ADR 0003: Adopt Ruff for Modern Linting and Formatting

## 1. Title
ADR 0003: Adopt Ruff for Modern Linting and Formatting

---

## 2. Status
**Proposed**

---

## 3. Context
The legacy linting and formatting stack in EDMC relies on `flake8` coupled with 8 syntax/linting plugins and `autopep8` for code styling. Maintaining this large list of separate tools complicates our dependency tree and slows down local environment setup and CI/CD check execution. 

We need a unified, high-performance static analysis tool that integrates directly with `pyproject.toml` to replace the legacy tools in our modern verification pipeline. Furthermore, during our current "In-Repository Audit" phase, we want to maintain a minimal local virtual environment containing only our review/complexity tools, avoiding the footprint of the heavy development dependencies until full development begins.

---

## 4. Decision
We will adopt **Ruff** as the primary linter and formatter for the updated verification pipeline. All pipeline dependencies will be defined inside the `[project.optional-dependencies]` table of [pyproject.toml](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/pyproject.toml).

To implement this decision and maintain environment isolation:
1. We will add `ruff` to the `dev` dependency group in `pyproject.toml` and configure its rules inside the `[tool.ruff]` tables.
2. We will isolate the `[project.optional-dependencies] review` group to contain only essential audit tools: `radon`, `vulture`, `pydeps`, and `pylint`.
3. We will provide automated commands to prune/uninstall non-essential dev packages from active local virtual environments to maintain minimalism during the review.

---

## 5. Consequences

### Positive:
* **Dependency Simplification:** Replaces 10 legacy dev dependencies (`flake8`, `autopep8`, and their plugins) with a single tool (`ruff`).
* **Consolidated Configuration:** All formatting, import sorting, and linting rules are stored inside `pyproject.toml`.
* **Fast Execution:** Ruff executes code analysis and formatting checks orders of magnitude faster than Python-based legacy tools.

### Negative:
* **Style Divergence:** Small formatting discrepancies may arise between `autopep8` and `ruff format` when we transition to re-development. (This formatting transition will be deferred until the review phase completes).

---

## 6. Procedure & Implementation

### A. Cleaning Up Your Local Environment
To uninstall all legacy development dependencies and keep your environment clean for the auditing phase, execute the following commands in your active virtual environment:

```bash
# 1. Uninstall dev packages and test tools (retaining base requirements)
pip uninstall -y flake8 flake8-annotations-coverage flake8-cognitive-complexity flake8-comprehensions flake8-docstrings flake8-json flake8-noqa flake8-use-fstring mypy pep8-naming safety autopep8 pre-commit mistune pytest pytest-cov coverage coverage-conditional-plugin astroid tomlkit

# 2. Re-install only the minimal review/audit packages in editable mode
pip install -e .[review]
```

### B. Integrating Ruff (Future Re-development Phase)
When moving out of the audit phase, we will add `ruff` to `pyproject.toml` under:
```toml
[project.optional-dependencies]
dev = [
    "ruff",
    "pytest",
    "mypy"
]
```
And configure it directly inside `pyproject.toml`:
```toml
[tool.ruff]
line-length = 120
select = ["E", "F", "I", "N"]
```
