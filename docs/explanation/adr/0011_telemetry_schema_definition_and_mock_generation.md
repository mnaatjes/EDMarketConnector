---
title: "ADR 0011: Telemetry Schema Definition and Mock Generation"
tags: ["adr", "architecture", "testing", "schema", "validation"]
created_at: "2026-08-20"
last_updated_at: "2026-08-20"
---

# ADR 0011: Telemetry Schema Definition and Mock Generation

## 1. Title
ADR 0011: Telemetry Schema Definition and Mock Generation

---

## 2. Status
**Proposed**

---

## 3. Context
We need to generate valid, authentic data values (e.g. realistic system coordinates, commander ranks, and market listings) to populate properties inside our simulated Journal Logs and Journal State Snapshots.

Furthermore, we must decide how to store and maintain the underlying schemas of these events in our codebase. To keep the project clean, we must research if these schemas are registered online, choose a coding representation (e.g. dataclasses, Pydantic, or raw JSON Schema), and define a value generation system.

---

## 4. Decision

### 4.1 Online Registry Findings
Frontier Developments does **not** register or publish official JSON Schemas on any online schema store (e.g. SchemaStore.org) or schema registry. They only distribute specifications as a human-readable text manual. The community maintains unofficial schemas inside individual library parser projects. Consequently, we must define and maintain our own schemas within the EDMC codebase.

### 4.2 Schema Storage in Code (Pydantic Models)
We will define all event and JSS schemas as **Pydantic Models** inside `src/edmc/domain/models/`:
* **Single Source of Truth:** Rather than maintaining duplicate files (static JSON Schema documents for validation AND Python dataclasses for processing), we write the schema definitions once as Pydantic models.
* **Dynamic Schema Generation:** Pydantic models automatically export their structures to standard JSON Schema formats at runtime via `.model_json_schema()`. The validation engine (`src/edmc/telemetry/validation.py`) will ingest these schemas dynamically to run validation checks.
* **Domain Model Alignment:** This satisfies our Domain-Driven Design (DDD) requirements, giving us type validation, type coercion, and JSON serialization out-of-the-box.

### 4.3 Value Generation (`MockValueGenerator`)
We will create a helper module `src/edmc/testing/generator.py` containing a `MockValueGenerator` class:
* **Purpose:** Provides test suites and the mock SDK with helper methods to generate valid values matching Pydantic model expectations (data types, ranges, constraints).
* **Interface Specification:**
  * `generate_coordinates() -> list[float]`: Returns a valid 3-element coordinate float list `[x, y, z]`.
  * `generate_cmdr_name() -> str`: Returns a randomized CMDR name string.
  * `generate_event_payload(model: Type[BaseModel]) -> dict`: Uses the model schema to automatically generate a complete mock dictionary populated with random but valid value types (using libraries like `pydantic-factories` or simple type-mapping fallbacks).
  * `generate_fsd_jump_payload(system_name: str) -> dict`: Convenience helper returning a fully populated FSDJump payload matching the v37 schema.

---

## 5. Consequences

### Positive:
* **Zero Schema Duplication:** Writing schemas as Pydantic models provides both runtime types and validation schemas in a single class definition.
* **Robust Value Generation:** The test runner is shielded from hardcoding mock values, keeping tests clean and readable.
* **API Compatibility:** Dynamic schemas generated from Pydantic are fully compliant with standard JSON Schema draft specifications.

### Negative:
* **Runtime Dependency:** Requires adding `pydantic` to our core dependencies, though this is already aligned with modernizing the data pipeline.

---

## 6. Procedure & Implementation

1. **Update `pyproject.toml`:** Add `pydantic` to the main `[project.dependencies]` or the `[project.optional-dependencies]` extra group.
2. **Implement Domain Models:** Create baseline Pydantic models (e.g. `Fileheader`, `LoadGame`, `FSDJump`) under `src/edmc/domain/models/`.
3. **Implement Generator:** Write the `MockValueGenerator` inside `src/edmc/testing/generator.py`.
