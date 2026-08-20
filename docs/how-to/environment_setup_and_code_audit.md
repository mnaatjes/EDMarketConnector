---
title: "Setting Up the Development Environment and Running Code Audits"
tags: ["how-to", "environment", "setup", "testing", "audit"]
created_at: "2026-08-19"
last_updated_at: "2026-08-19"
---

# Setting Up the Development Environment and Running Code Audits

This guide provides practical, step-by-step instructions to initialize a local development environment, install the required packages using modern packaging commands, run test suites, and execute code-quality audits on the Elite Dangerous Market Connector (EDMC) codebase.

---

## Prerequisites

Before starting, ensure you have:
* **Python 3.10 or newer** installed on your system.
* **Git** installed and configured.
* Access to a terminal/shell.

---

## Step 1: Create and Activate a Virtual Environment

Isolate the project's dependencies from your system's python directories:

1. Open your terminal in the repository root directory.
2. Initialize a virtual environment named `.venv`:
   ```bash
   python3 -m venv .venv
   ```
3. Activate the virtual environment:
   * **Linux / macOS:**
     ```bash
     source .venv/bin/activate
     ```
   * **Windows (Command Prompt):**
     ```cmd
     .venv\Scripts\activate.bat
     ```
   * **Windows (PowerShell):**
     ```powershell
     .venv\Scripts\Activate.ps1
     ```

---

## Step 2: Install the Package with Review Tools

Using the configuration declared in [pyproject.toml](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/pyproject.toml), install the application in editable (`-e`) mode along with the `review` optional dependency group:

```bash
pip install -e .[review]
```

This installs:
* All core application dependencies.
* Development/auditing tools: `radon`, `vulture`, `pydeps`, and `pylint`.
* A pointer to your local source code, allowing changes to take effect immediately.

---

## Step 3: Run the Test Suite

Before making any modifications, run the automated tests to verify that your environment is fully functional and matches the baseline:

```bash
pytest
```

This executes all tests configured in the `tests/` directory and prints a coverage report indicating which sections of the code are currently tested.

---

## Step 4: Execute Code Audits

Use the installed review tools to analyze code complexity, locate unused code, and build module dependency maps.

### A. Check Code Complexity (Radon)
Analyze the cyclomatic complexity of Python source files. The `-a` flag prints the average complexity, and `-s` shows the complexity rank (A is simplest, F is most complex):

```bash
radon cc . -a -s
```

### B. Locate Unused Code (Vulture)
Vulture scans your repository for unused classes, functions, variables, and properties. It helps identify legacy paths that are safe to remove:

```bash
vulture .
```

### C. Map Code Dependencies (Pydeps)
Generate a visual import dependency graph to understand module coupling. The following command excludes tkinter to focus on internal logic and saves the graph as an SVG:

```bash
pydeps --max-bacon=2 --exclude tkinter -o docs/reference/dependency_graph.svg .
```
*(Open the resulting `dependency_graph.svg` file in any web browser to inspect the visual dependency hierarchy.)*
