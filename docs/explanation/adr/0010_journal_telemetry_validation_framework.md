---
title: "ADR 0010: Journal Telemetry Validation Framework"
tags: ["adr", "architecture", "testing", "validation", "telemetry"]
created_at: "2026-08-20"
last_updated_at: "2026-08-20"
---

# ADR 0010: Journal Telemetry Validation Framework

## 1. Title
ADR 0010: Journal Telemetry Validation Framework

---

## 2. Status
**Proposed**

---

## 3. Context
We need to verify the correctness of the generated outputs from our mock SDK. We also need to process real-world log files and Journal State Snapshots (JSS) inside the main application without introducing runtime fragility. 

If our production parser is too rigid, a minor, undocumented game update could crash the user's background EDMC process on type-mismatches. Conversely, if our test validation is too loose, we risk passing tests on invalid payload structures.

We need a unified Telemetry module structure that:
1. Houses both the log parser and JSS snapshot parser.
2. Supports **Strict Validation** for our test pipelines (failing on schema mismatch).
3. Supports **Tolerant Parsing** at runtime (logging warnings on validation anomalies, but continuing execution safely).
4. Establishes a scalable directory structure inside the repository to warehouse genuine, versioned test articles (logs, snapshots, and third-party outputs).

---

## 4. Decision
We will establish a unified `src/edmc/telemetry/` package to manage all ingestion, parsing, and validation schemas for both Journal Logs and Journal State Snapshots. We will implement a dual-severity validation architecture, and define clear repository paths for testing data fixtures.

### 4.1 Repository Data Directory Layouts
To scale our support for multiple journal versions, external schemas, and test payloads, we establish two root-level directory structures:

* **Staging Directory (`data/raw/`):**
  * Located at the repository root: `/data/raw/`.
  * Used as a temporary local staging workspace for developers to collect un-sanitized game logs, JSS profiles, and raw api outputs.
  * This folder is added to `.gitignore` to prevent any personal user metadata from being committed.
* **Test Fixture Warehouse (`tests/data/`):**
  * Located under the test directory: `/tests/data/`.
  * Checked into Git version control. Contains sanitized, anonymized baseline test articles organized by version:
    * `tests/data/v37/logs/`: Genuine, anonymized `Journal.*.log` files.
    * `tests/data/v37/snapshots/`: Genuine, anonymized `Status.json`, `Market.json`, etc.
    * `tests/data/v37/egress/`: Sample API payloads/schemas matching what is sent to EDSM, Inara, and EDDN.

```text
data/
└── raw/                 # (Git Ignored) Raw developer staging folder
    ├── logs/            # Copy of un-sanitized player log files
    ├── snapshots/       # Copy of un-sanitized JSS files (Market.json, etc.)
    └── egress/          # Raw api outputs from external endpoints

tests/
└── data/                # (Git Tracked) Sanitized test fixtures
    └── v37/
        ├── logs/        # Sanitized Journal Log files (e.g. Journal.260820.01.log)
        ├── snapshots/   # Sanitized JSS files (e.g. Market.json, Status.json)
        └── egress/      # Expected API schema/payload mock outputs
```

### 4.2 Telemetry Package Layout (`src/edmc/telemetry/`)
* `parser.py`: Tailer and line parser for append-only Journal Logs.
* `snapshots.py`: Parser for dynamic Journal State Snapshots (JSS).
* `validation.py`: Core schema validation engine utilizing `jsonschema`.
* `schemas/`: Directory containing static JSON Schema files (.json) for events and snapshots.

```text
src/edmc/
└── telemetry/
    ├── __init__.py      # Exposes parser, snapshot, and validator interfaces
    ├── parser.py        # Log parsing logic
    ├── snapshots.py     # Snapshot parsing logic
    ├── validation.py    # Dual-severity validation logic using jsonschema
    └── schemas/         # Sub-folder housing static event JSON schemas
```

### 4.3 Dual-Severity Validation Logic
* **Strict Mode (Testing Pipeline):** When executing tests using our public `edmc.testing` mock SDK, the validator runs with strict error propagation. Any schema violation (such as type mismatches or missing required keys) immediately raises a `ValidationError`, halting the test.
* **Lenient Mode (Production/Run Pipeline):** During live gameplay execution, the telemetry ingestion pipeline runs the validator in warning-only mode. If the game client writes an unexpected key or changes a type signature, EDMC will log a warning to the diagnostic files but continue parsing and transmitting the remaining valid telemetry payload.

### 4.4 Sources for Raw Baseline Test Articles
To obtain unaltered, genuine logs and JSS companion files, we will rely exclusively on the developer's local game installation:
* **Local System Source:** Copy files directly from the active game client's saved games folder. On Linux (Steam/Proton), this resides in:
  `~/.steam/steam/steamapps/compatdata/359320/pfx/drive_c/users/steamuser/Saved Games/Frontier Developments/Elite Dangerous/`

### 4.5 Local ETL & Sanitization Pipeline
To securely migrate these local logs to the git-tracked test suites:
1. **Extract:** A local python script copies raw files from the system's game folder into the git-ignored staging area `data/raw/`.
2. **Transform (Sanitize):** The script parses each log line and snapshot JSON object in staging, stripping out sensitive identifiers (such as CMDR Name, FID, and Credits) and replacing them with standardized test values.
3. **Load:** The sanitized, anonymous files are written to `/tests/data/v37/` to serve as baseline test fixtures.

---

## 5. Consequences

### Positive:
* **Decoupled Telemetry Domain:** Consolidates all parsing and validation rules inside a single package, separating them from both the GUI and the egress network adapters.
* **Robust Production Runs:** Tolerant parsing prevents user-facing app crashes during undocumented game updates.
* **Hermetic Testing:** Keeping sanitized test fixtures under `/tests/data/` allows tests to run out-of-the-box in offline CI/CD runners without needing network access.
* **Isolates Source Code from Data:** Staging in `data/raw/` ensures no raw files pollute the `src/` directory.

### Negative:
* **Dependency Expansion:** Adds `jsonschema` to the development dependencies table.

---

## 6. Procedure & Implementation

1. **Update `pyproject.toml`:** Add `jsonschema` to the `dev` dependency table.
2. **Update `.gitignore`:** Ensure `data/raw/` is excluded from Git tracking.
3. **Scaffold Telemetry Package:** Create `src/edmc/telemetry/` and its sub-directories.
4. **Scaffold Data Warehouse:** Create the `tests/data/v37/logs/`, `tests/data/v37/snapshots/`, and `tests/data/v37/egress/` directories.
5. **Implement Dual Validator:** Write the validator class inside `src/edmc/telemetry/validation.py`.
