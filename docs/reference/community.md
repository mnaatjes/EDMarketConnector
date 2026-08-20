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
