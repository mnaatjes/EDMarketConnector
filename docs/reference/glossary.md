---
title: "Project Glossary"
tags: ["reference", "glossary", "terminology"]
created_at: "2026-08-20"
last_updated_at: "2026-08-20"
---

# Project Glossary

This document defines the standardized terminology used across the EDMarketConnector codebase and documentation to prevent architectural or logical confusion.

---

## 1. Core Terminology

### Journal Log
* **Description:** The chronological, append-only files written by the Elite Dangerous game client during a gameplay session.
* **File Format:** JSON Lines (JSONL). Each line is an independent, complete JSON object.
* **Naming Scheme:** `Journal.YYYY-MM-DDTHHMMSS.NN.log`
* **Behavior:** Incremental and permanent for the duration of the session. New events are appended to the end of the file in real-time.

### Journal State Snapshot
* **Description:** The companion, single-object JSON files written by the game client to the same directory to record transient sub-system states.
* **File Format:** Standard JSON file containing a single root object or list.
* **Naming Scheme:** `<Name>.json` (e.g. `Market.json`, `Status.json`, `Cargo.json`).
* **Behavior:** Dynamic and transient. The file is completely overwritten (replaced) in-place by the game client when its state changes or when the player interacts with specific station interface panels. It does not maintain historical logs.

### Ingestion Pipeline
* **Description:** The system components (`src/edmc/ingestion/`) responsible for watching the saved games directory, tailing the active Journal Log, reading updated Journal State Snapshots, and parsing their raw contents.

### Egress Adapters
* **Description:** The system components (`src/edmc/egress/`) responsible for receiving domain events from the broker, transmuting them into target payload formats, and uploading them to remote APIs (like EDSM, Inara, or EDDN).
