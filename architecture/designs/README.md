---
title: "Software Design Documents Governance and Index"
tags: ["architecture", "designs", "sdd", "governance"]
created_at: "2026-10-06"
last_updated_at: "2026-10-06"
---

# Software Design Documents Governance and Index

Governed by IEEE 1016-2009 (Systems Design—Software Design Descriptions) and Google Design Doc standards.

---

## 1. Quality Invariants

1. **Pre-Implementation Requirement:** An SDD must be drafted, reviewed, and approved before implementing non-trivial subsystems.
2. **The Vacation Test:** Must be sufficiently detailed for an independent engineer to build and verify the subsystem without consulting the author.
3. **The Skeptic Test:** Rigorously justify why the subsystem is required and why simpler alternatives are insufficient.
4. **Mandatory Visual Models:** Must include at least two Mermaid diagrams (Structural Component/Class diagram and Dynamic Sequence/Activity diagram).
5. **PR Sequencing:** Must outline phased PR milestones with verification criteria.

---

## 2. Frontmatter Template

Files follow `NNNN_descriptive_title.md`:

```yaml
---
title: "SDD-NNN: Descriptive Subsystem Title"
status: "in-progress" # draft | under-review | approved | in-progress | completed | superseded
authors: ["@mnaatjes"]
reviewers: ["Engineering Team"]
created_at: "YYYY-MM-DD"
last_updated_at: "YYYY-MM-DD"
related_adrs: []
related_rfcs: []
---
```

---

## 3. Registered Designs

| ID | Title | Status | Date | Related ADRs |
| :---: | :--- | :---: | :---: | :--- |
| **SDD-001** | [SDD-001: Hexagonal Architecture Core and Multi-Adapter Topology](0001_hexagonal_architecture_and_adapter_topology.md) | **draft** | 2026-10-06 | [ADR 0001](../adr/0001_hexagonal_architecture_and_multi_adapter_interfaces.md) |
| **SDD-002** | [SDD-002: Packaging Modernization, Test Partitioning, and CI Pipeline Topology](0002_packaging_modernization_and_test_partitioning.md) | **completed** | 2026-10-06 | [ADR 0002](../adr/0002_modernize_packaging_workflows_and_partition_tests.md) |

