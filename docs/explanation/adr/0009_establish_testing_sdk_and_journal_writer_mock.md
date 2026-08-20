---
title: "ADR 0009: Establish Testing SDK and Journal Writer Mock"
tags: ["adr", "architecture", "testing", "ingestion", "mocking"]
created_at: "2026-08-19"
last_updated_at: "2026-08-19"
---

# ADR 0009: Establish Testing SDK and Journal Writer Mock

## 1. Title
ADR 0009: Establish Testing SDK and Journal Writer Mock

---

## 2. Status
**Proposed**

---

## 3. Context
We need to verify the correctness of the new `edmc.ingestion` file watcher and parser modules. Running these tests manually against a live game client is impossible in a headless CI/CD environment. 

To achieve deterministic testing from simple file detection up to granular JSON line parsing, we need a robust mock utility (`MockJournalWriter`) that simulates the file writing behavior of the Elite Dangerous game client. Because this simulator is highly valuable to the broader EDMC developer community for testing third-party plugins, we will design and export it as a public testing SDK utility under `src/edmc/testing/`.

---

## 4. Evidence & Research Findings

Detailed system path specifications, directory discovery algorithms, symbolic redirection details, Linux Proton edge cases, file format definitions, local game client test logs, and target specifications are fully documented in the [Journal v37 Reference Specification](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/docs/reference/journal/v37/specification.md).

### 4.1 Summary of Key Specs (from Reference)
* **Default Directory Paths:**
  * **Windows:** `%USERPROFILE%\Saved Games\Frontier Developments\Elite Dangerous\`
  * **Linux (Steam/Proton):** `~/.steam/steam/steamapps/compatdata/359320/pfx/drive_c/users/steamuser/Saved Games/Frontier Developments/Elite Dangerous/`
* **Target Schema:** Adhere to Player Journal v37 (verified locally on live game client version `4.4.0.3` Odyssey Live).
* **Ingestion Method:** Binary unbuffered file tailing (`'rb', 0`) tracking byte offsets.
* **Rigid Pathing:** The directory cannot be configured inside the game; it must be relocated via OS-level symbolic links or custom configurations (`journaldir` override).

---

## 5. Decision
We will establish a public testing SDK package under `src/edmc/testing/` containing a robust, file-based `MockJournalWriter` simulator.

### 5.1 `MockJournalWriter` Design Specification
* **File-Based Output:** The writer will write actual JSONL files to a temporary directory target. This enables testing of both the directory file watchers and the tailing logic.
* **API Interface:** The class will expose programmatic controls for tests to drive the simulation:
  * `__init__(output_dir: Path)`: Initializes the mock directory.
  * `start_game(cmdr_name: str = "TestCmdr")`: Creates a new journal file and writes the `Fileheader`, `LoadGame`, and `Rank` sequence.
  * `write_event(event_name: str, payload: dict)`: Appends a specific JSON event line to the active file.
  * `rotate_log()`: Simulates restarting the game client by closing the current file and opening a new sequential timestamped log file.
  * `stop_game()`: Writes a `Shutdown` event and closes the file handle.

### 5.2 Exposing Supported Journal Version
To inform developers and users of the active Journal version compatibility:
1. **Repository Main README:** We will add a "Compatibility & Specifications" section in the root `README.md` explicitly stating the supported Journal specification (e.g. v37) and game version compatibility.
2. **Testing Package Facade:** We will declare a package-level variable `__journal_version__ = "37"` inside the public gateway `src/edmc/testing/__init__.py` and document the API coverage inside the package docstrings.
3. **Changelog Releases:** Any subsequent changes or incremental key additions to our parsed event models will be logged as milestones under `CHANGELOG.md` with explicit version numbers.

---

## 6. Consequences

### Positive:
* **Deterministic Integration Testing:** We can verify the entire ingestion pipeline (directory watching, file tailing, byte-offset seek, event parsing) in clean unit tests.
* **Benefits to Third-Party Developers:** Plugin authors can import the mock SDK to test their plugins headlessly in their own CI/CD pipelines.

### Negative:
* **I/O Overhead:** Because it writes to actual files, running these tests creates filesystem I/O (mitigated by using fast temporary directories `/tmp` or `shm` inside test runners).

---

## 7. Procedure & Implementation

1. **Create Package:** Scaffold the `src/edmc/testing/` directory.
2. **Implement Writer:** Write `MockJournalWriter` inside [src/edmc/testing/journal_writer.py](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/src/edmc/testing/journal_writer.py).
3. **Expose Gateway:** Export `MockJournalWriter` in [src/edmc/testing/__init__.py](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/src/edmc/testing/__init__.py).
4. **Integration Test:** Write a unit test inside `tests/unit/test_ingestion_simulation.py` that utilizes `MockJournalWriter` to write mock events and verifies that a basic parser catches them.
