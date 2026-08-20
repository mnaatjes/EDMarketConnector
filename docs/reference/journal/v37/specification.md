---
title: "Journal v37 Reference Specification"
tags: ["reference", "journal", "specifications", "paths", "v37"]
created_at: "2026-08-19"
last_updated_at: "2026-08-19"
---

# Journal v37 Reference Specification

This reference document outlines the directory locations, rigid path constraints, Linux Proton edge cases, file schemas, and reading tailer behaviors for the Elite Dangerous Player Journal v37 format (Odyssey Live client).

---

## 1. Operating System Directory Paths

### 1.1 Windows Default Path
* **Location:** `%USERPROFILE%\Saved Games\Frontier Developments\Elite Dangerous\`
* **Programmatic Resolution:** Resolved dynamically using the Windows shell API `SHGetKnownFolderPath` with `FOLDERID_SavedGames`.

### 1.2 Linux Proton (Steam) Default Path
* **Location:** `~/.steam/steam/steamapps/compatdata/359320/pfx/drive_c/users/steamuser/Saved Games/Frontier Developments/Elite Dangerous/`
* **Note:** `359320` is the Steam AppID for the Elite Dangerous client.

---

## 2. Rigid Path Rules & Custom Overrides

* **Rigid Path Hardcoding:** The game client hardcodes the relative path inside the active user's environment. It does not provide settings inside game menus or `AppConfig.xml` to redirect this folder.
* **Symbolic Redirection:** Users wishing to relocate the folder must configure OS-level Symbolic Links (`mklink /D` on Windows or `ln -s` on Linux) to point to other folders or cloud sync drives.
* **EDMC Custom Paths:** EDMC provides an override key (`journaldir`) in its configuration file. When present, the ingestion observer monitors this custom path instead of querying default directories.

---

## 3. Linux/Proton Pathing Edge Cases

Automatic discovery of the journal directory on Linux is prone to failure due to the following sandbox and pathing configurations:
* **Flatpak Steam Sandbox:** Flatpak isolates Steam's filesystem space, placing the compatibility prefix in:
  `~/.var/app/com.valvesoftware.Steam/.local/share/Steam/steamapps/compatdata/359320/...`
* **Secondary Steam Library Mounts:** If the game is installed on a separate SSD library drive, the prefix is stored under that drive's Steam library directory instead of the default home directory.
* **Wine/Lutris prefixes:** Standalone Elite Launcher installations map prefixes to custom directories, such as `~/Games/elite-dangerous/drive_c/...`.

---

## 4. File Format and Telemetry Rules

* **File Naming Format:** Files are generated at game launch using the format:
  `Journal.YYYY-MM-DDTHHMMSS.NN.log` (e.g. `Journal.2026-08-19T210000.01.log`).
* **Format Structure:** The files are structured as **JSON Lines (JSONL)**. Each line is a single, independent UTF-8 JSON object terminated by `\n` or `\r\n`.
* **Execution Sequence:**
  1. The game creates a new file at startup and writes a `Fileheader` event.
  2. It writes environment events (`Language`, `Commander`, `LoadGame`, `Rank`).
  3. During gameplay, events are appended in real-time as actions occur (e.g. `FSDJump`, `Docked`).
  4. At exit, it appends a `Shutdown` event and closes the file handle.

### 4.1 Sample Local In-Game Header Event (gameversion 4.4.0.3)
```json
{ 
  "timestamp": "2026-08-14T03:29:39Z", 
  "event": "Fileheader", 
  "part": 1, 
  "language": "English/UK", 
  "Odyssey": true, 
  "gameversion": "4.4.0.3", 
  "build": "r330683/r0 " 
}
```

---

## 5. Ingestion Tailer Behavior

* **Binary Unbuffered Reading:** EDMC opens log files in unbuffered binary read-only mode (`open(logfile, 'rb', 0)`). This prevents text-mode newlines or write-buffering from causing read failures when the game is writing to the same file.
* **Byte Offset Tailing:** At startup, the parser reads all lines in the newest log file to catch up. Once it reaches the end, it records the byte position using `loghandle.tell()` and transitions to polling mode, periodically seeking to the offset to check for new data.

---

## 6. Journal State Snapshots

Unlike the append-only `Journal.*.log` files, the following files are rewritten in-place (completely overwritten) in the same directory by the game client:

* **`Status.json`:** Contains frequently-changing cockpit GUI status flags, system indicators, and user controls. Updated in real-time as state modifications occur.
* **`Market.json`:** Contains commodity price sheets and demand data. Generated when the player interacts with a station's commodities interface.
* **`Outfitting.json`:** Contains the list of outfitting modules and prices available at the station. Generated when the player opens the outfitting interface.
* **`Shipyard.json`:** Contains the list of purchaseable ships and prices available at the station. Generated when the shipyard interface is accessed.
* **`Cargo.json`:** Tracks the player's active ship cargo inventory. Updated when cargo items are collected, consumed, or sold.
* **`ShipLocker.json` / `Backpack.json`:** Tracks Odyssey micro-resources and inventory contents.
* **`NavRoute.json`:** Stores plotted navigation routes and jump targets. Updated when a route is recalculated.
