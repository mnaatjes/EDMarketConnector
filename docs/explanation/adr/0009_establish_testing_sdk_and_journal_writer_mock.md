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

### 4.1 Operating System Journal Paths
* **Windows (Native):** Elite Dangerous defaults to writing logs inside the Windows "Saved Games" directory (resolved via Windows shell API `FOLDERID_SavedGames`):
  `C:\Users\<Username>\Saved Games\Frontier Developments\Elite Dangerous\`
* **Linux (Proton/Steam):** The game runs under Proton, which creates a separate virtual Windows directory prefix for each game. The path varies based on the Steam library location, typically resolved to:
  `~/.steam/steam/steamapps/compatdata/359320/pfx/drive_c/users/steamuser/Saved Games/Frontier Developments/Elite Dangerous/`

### 4.2 Rigid Pathing and Redirections
* **No Native Configuration:** The Elite Dangerous game client hardcodes the relative path and does *not* support altering the directory via configuration files (e.g., `AppConfig.xml` cannot change this target path).
* **Symbolic Link Workarounds:** Players wishing to move the directory to another drive or cloud folder must use OS-level Symbolic Links (`mklink /D` on Windows or `ln -s` on Linux).
* **EDMC Custom Folder Settings:** EDMC allows overriding the default discovery path via the `journaldir` parameter inside the user configurations.

### 4.3 Linux/Proton Pathing Edge Cases
* **Flatpak Steam Installs:** Isolates the system directories, shifting paths to sandboxed containers: `~/.var/app/com.valvesoftware.Steam/.local/share/Steam/...`
* **Custom Steam Libraries:** If the game is installed on a secondary drive library folder, the Proton prefix (`compatdata/359320/`) resides on that drive's library path instead of the home directory.
* **Standalone / Wine Launchers:** Non-Steam standalone setups running via Lutris or custom Wine prefixes map virtual drives to arbitrary user folders (e.g., `~/Games/elite-dangerous/drive_c/...`).
* *Due to these path variations, automatic Linux path discovery is highly unreliable, reinforcing the requirement to maintain manual settings fallbacks.*

### 4.4 Official API Manual Specifications
* **Target Specification:** We will target and adhere to **Player Journal v37** (which maps to Elite Dangerous Odyssey up to Update 14, released May 2023).
* **Proof of Specification Validity:**
  * Community-maintained reference specifications, such as the [Elite Dangerous Player Journal Manual v37](https://elite-journal.readthedocs.io/en/latest/), and Frontier's live-updating endpoint at `https://hosting.zaonce.net/manual/Elite_Dangerous_Player_Journal.json` establish v37 as the active baseline.
  * While content updates in 2025/2026 (including the System Colonization, Powerplay 2.0, and Kestrel/Highliner updates) have introduced new data fields and events, Frontier appends these keys dynamically to the v37 schema rather than incrementing the major journal version. v37 remains the primary schema base.

### 4.5 Game Journal Writing Rules
* **File Naming Format:** Files are generated at game launch using the format:
  `Journal.YYYY-MM-DDTHHMMSS.NN.log` (e.g. `Journal.2026-08-19T210000.01.log`).
* **Format Structure:** The files are structured as **JSON Lines (JSONL)**. Each line is a single, independent UTF-8 JSON object terminated by `\n` or `\r\n`.
* **Execution Sequence:**
  1. The game creates a new file at startup and writes a `Fileheader` event.
  2. It writes environment events (`Language`, `Commander`, `LoadGame`, `Rank`).
  3. During gameplay, events are appended in real-time as actions occur (e.g. `FSDJump`, `Docked`).
  4. At exit, it appends a `Shutdown` event and closes the file handle.

### 4.6 Ingest Tailer Behavior (How EDMC Reads Logs)
* **Binary Unbuffered Reading:** EDMC opens log files in unbuffered binary read-only mode (`open(logfile, 'rb', 0)`). This prevents text-mode newlines or write-buffering from causing read failures when the game is writing to the same file.
* **Byte Offset Tailing:** At startup, the parser reads all lines in the newest log file to catch up. Once it reaches the end, it records the byte position using `loghandle.tell()` and transitions to polling mode, periodically seeking to the offset to check for new data.

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
