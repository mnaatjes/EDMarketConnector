---
title: "ADR 0012: Telemetry Schema Collation and Extraction Pipeline"
tags: ["adr", "architecture", "telemetry", "schemas", "pipeline"]
created_at: "2026-08-20"
last_updated_at: "2026-08-20"
---

# ADR 0012: Telemetry Schema Collation and Extraction Pipeline

## 1. Title
ADR 0012: Telemetry Schema Collation and Extraction Pipeline

---

## 2. Status
**Proposed**

---

## 3. Context
We need to compile the single source of truth for all journal events (logs) and Journal State Snapshots (JSS) schemas, properties, value ranges, and types. Since the schemas do not exist officially online, we have cloned the relevant community projects (jixxed schemas, EDDN schemas, Lombra documentation) to our local environment under `/home/michael/src/github.com/`. 

We need to define a systematic pipeline to locate, isolate, collate, and parse this raw data to produce our codebase schemas and Pydantic models.

---

## 4. Decision
We will establish a formal **Schema Collation and Extraction Pipeline** to compile these inputs.

### 4.1 Input Sources (Cloned Repositories)
We will extract raw telemetry constraints from the following local directories:
1. **jixxed schemas:** `/home/michael/src/github.com/jixxed/ed-journal-schemas/schemas/` (Individual event schemas).
2. **EDDN schemas:** `/home/michael/src/github.com/EDCD/EDDN/schemas/` (Egress envelope mappings and sanitization READMEs).
3. **Lombra documentation:** `/home/michael/src/github.com/Lombra/elite-api-docs/docs/` (Markdown specifications of status flags).

### 4.2 Collation Pipeline Stages
We will process the raw schemas and documentation in four sequential stages:

```text
[ jixxed/schemas/ ] --------+
                            |
[ EDCD/EDDN/schemas/ ] -----+--> (1. Locate & Isolate) --> (2. Collate & Resolve)
                            |                                    |
[ Lombra/docs/ ] -----------+                                    v
                                                        (3. Translate to Pydantic)
                                                                 |
                                                                 v
                                                        (4. Load Domain & Schemas)
```

1. **Locate & Isolate:** 
   * Identify and isolate target schemas for the core bootstrapping and telemetry events: `Fileheader`, `LoadGame`, `Rank`, `Progress`, `Docked`, `FSDJump`, and `Location`.
   * Identify and isolate JSS companion files: `Status.json`, `Market.json`, `Outfitting.json`, `Shipyard.json`, and `Cargo.json`.
2. **Collation & Resolve:**
   * Merge the isolated event schemas with their parent type definition (`_Event.json`) to resolve the complete schema.
   * Extract the `"examples"` value arrays from the schemas to populate test generation libraries.
   * Map `Status.json` properties to the bitwise integer flag maps documented in Lombra's `Status File.md`.
3. **Translate (Python Pydantic Models):**
   * Translate the resolved schemas into strongly-typed **Pydantic Models** under `src/edmc/domain/models/`.
   * Map JSON Schema types (`string`, `integer`, `boolean`, `array`) to Python typings (`str`, `int`, `bool`, `list`).
   * Translate the `Status.json` bitmasks into Pydantic model computed properties or fields.
4. **Load Domain & Schemas:**
   * Export the Pydantic classes to expose them as compile-time types inside the core code.
   * Run Pydantic's `.model_json_schema()` to dynamically generate the JSON Schema files used by our runtime validator.
   * Save the generated JSON Schemas under `src/edmc/telemetry/schemas/` to keep them clean of developer comments or artifacts.

---

## 5. Consequences

### Positive:
* **Systematic Schema Transition:** Defines a clear, repeatable process to construct Pydantic models from community-vetted specifications.
* **Hermetic Schemas:** Eliminates manually drafting schemas by using community JSON Schemas as direct code templates.
* **100% Type Safe:** Guarantees that our parser and mock generators are strictly validated against verified schemas.

### Negative:
* None. The pipeline is run locally by developers during the initial refactoring phase.

---

## 6. Procedure & Implementation

1. **Locate Base Schemas:** Read the base schemas (`Fileheader`, `LoadGame`, `FSDJump`) in the local jixxed clone.
2. **Implement Pydantic Models:** Convert the schemas to Pydantic model definitions in [src/edmc/domain/models/](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/src/edmc/domain/models/).
3. **Configure Validation Engine:** Write code to load schemas directly from these models at runtime.
