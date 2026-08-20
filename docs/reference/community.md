---
title: "Community Developer Resource Guide"
tags: ["reference", "community", "schemas", "repositories"]
created_at: "2026-08-20"
last_updated_at: "2026-08-20"
---

# Community Developer Resource Guide

This reference document catalogues key open-source repositories, schemas, and parser libraries developed by the Elite Dangerous developer community. These resources serve as valuable references for defining our telemetry models and test fixtures.

---

## 1. Schema Repositories

### EDCD EDDN Gateway Schemas
* **Description:** The official JSON Schema files used by the Elite Dangerous Data Network (EDDN) to validate data uploads from client apps. These provide exact property formats for companion state snapshots.
* **Format:** JSON Schema (`.json`) files.
* **Key Folder:** `/schemas/` (includes commodity, outfitting, shipyard, and faction schemas).
* **Reference Link:** [EDCD/EDDN GitHub Repository](https://github.com/EDCD/EDDN/tree/master/schemas)
* **Pinpoint Specifications for Ingestion & Egress:**
  * **Event Coverage:** EDDN only processes 7 specific journal log events: `Docked`, `FSDJump`, `Scan`, `Location`, `SAASignalsFound`, `CarrierJump`, and `CodexEntry`.
  * **Envelope Mapping to State Snapshots:**
    * `commodity-v3.0.json` maps to `Market.json`.
    * `outfitting-v3.0.json` maps to `Outfitting.json`.
    * `shipyard-v2.0.json` maps to `Shipyard.json`.
    * `navroute-v1.0.json` maps to `NavRoute.json`.
    * `fcmaterials_journal-v1.0.json` maps to `FCMaterials.json`.
  * **Sanitization/Elision Rules (from `journal-README.md`):**
    * Strip all keys ending with `_Localised`.
    * In `Docked`: strip `Wanted`, `ActiveFine`, and `CockpitBreach`.
    * In `FSDJump`: strip `Wanted`, `BoostUsed`, `FuelLevel`, `FuelUsed`, `JumpDist`, and player-faction reputations (`MyReputation`, `SquadronFaction`, etc.).
    * In `Location`: strip `Wanted`, `Latitude`, `Longitude`, and player-faction reputations.

### ED Journal Schemas (jixxed)
* **Description:** A comprehensive community compiler hosting JSON Schemas for every individual Elite Dangerous Player Journal event type. This is extremely useful for structured schema validation and testing.
* **Web Directory Link:** [ED Journal Schemas Web Directory](https://schemas.edomh.nl/)
* **Reference Link:** [jixxed/ed-journal-schemas GitHub Repository](https://github.com/jixxed/ed-journal-schemas)
* **Pinpoint Specifications & Findings:**
  * **Structure:** Contains 274 subdirectories, with each directory housing a dedicated JSON Schema file (adhering to Draft 2020-12) for a specific game event or JSS companion file (such as `Fileheader`, `Status`, `Market`, `Cargo`).
  * **Inheritance:** All schemas inherit from a parent `/schemas/_Event.json` definition, which enforces the baseline properties: `timestamp` (date-time format) and `event` (string name).
  * **Content Definitions:** Schemas strictly define type constraints (e.g. integer, string, boolean), declare mandatory fields via `required`, and disable unknown keys via `additionalProperties: false`.
  * **Mock Value Utility:** Every schema includes an `"examples"` key-value array showing authentic values (e.g. game build strings like `"r282108/r0 "`). The `MockValueGenerator` should read these arrays to populate properties with authentic values.

### Elite Dangerous Player Journal Manual (Lombra)
* **Description:** The primary community-maintained documentation and API reference guide covering files, startup processes, telemetry events, and companion state snapshots.
* **Web Manual Link:** [Elite Dangerous Player Journal Manual](https://elite-journal.readthedocs.io/en/latest/)
* **Reference Link:** [Lombra/elite-api-docs GitHub Repository](https://github.com/Lombra/elite-api-docs.git)
* **Pinpoint Specifications & Findings:**
  * **Document Sources:** The repository contains the raw source files (such as `File Format.md`, `Status File.md`, and `Station Services.md`) used to render the documentation.
  * **Status.json Bitmasks:** The [Status File.md](file:///home/michael/src/github.com/Lombra/elite-api-docs/docs/Status%20File.md) source details the exact bitwise mappings for `Flags` (bits 0 to 31, e.g. Bit 0 for `Docked`, Bit 4 for `Supercruise`) and `Flags2` (foot states like `OnFoot`, `LowOxygen`), which are critical for developing our status domain parser.

---

## 2. Parsing and Type Definition Libraries

### kayahr/ed-journal (Highly Recommended)
* **Description:** A highly comprehensive, community-maintained TypeScript parser library for Elite Dangerous logs. It defines TypeScript interfaces for all 150+ journal event types.
* **Architectural Utility:** Although written in TypeScript, these interfaces map out every possible JSON property, data type, and option for all game events. This is the ideal reference when writing our Python Pydantic models.
* **Reference Link:** [kayahr/ed-journal GitHub Repository](https://github.com/kayahr/ed-journal)

---

## 3. Desktop Application Test Fixtures

### EDDiscovery
* **Description:** A C#-based desktop tool tracking player histories. Contains a rich variety of real player journal logs checked into their repository for regression testing.
* **Architectural Utility:** Useful for identifying rare event logs and testing edge-case parsing behaviors.
* **Reference Link:** [EDDiscovery GitHub Repository](https://github.com/EDDiscovery/EDDiscovery)

### EDCD EDDI
* **Description:** A C#-based voice and telemetry assistant. Houses test logs and event schemas inside its test projects.
* **Reference Link:** [EDCD/EDDI GitHub Repository](https://github.com/EDCD/EDDI)
