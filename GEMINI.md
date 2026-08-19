# Directives and Context for `/root/docs` directory

This document defines the structural directives and context guidelines for homelab system documentation. For historical updates and completed milestones, refer to the [Documentation Changelog](file:///root/docs/CHANGELOG.md).
Testing testing 1.. 2... 3
---
## 1. Purpose and Structure

This repository is a fork of official Elite Dangerous Market Connector

* **Goal:** Review, Map, and Understand the Components and Structure of this EDMC repository
  1. Identify all necessary components
  2. Audit structure and attempt to rationalize it
  3. Engineer a better solution for EDMC on my fork

## 2. Diataxis Documentation Structure Layout
All documentation files under `~/docs/` must be structured into the following Diataxis directories:
* **`tutorials/`**: Guided learning pathways for beginners.
* **`how-to/`**: Practical, step-by-step guides to solve specific tasks or execute runbooks.
* **`reference/`**: Technical descriptions, architecture specifications, schemas, standards, and glossaries.
* **`explanation/`**: High-level background concepts, design rationales, and Architectural Decision Records (ADRs).

> Note: To Date, repository is a fork of EDMC and therefore subject to the existing structures of `docs/` directory

## 3. Metadata Frontmatter Syntax
All documentation pages must include the following YAML frontmatter metadata format at the top of the file:
```yaml
---
title: "Document Title"
tags: ["tag-name-1", "tag-name-2"]
created_at: "YYYY-MM-DD"
last_updated_at: "YYYY-MM-DD"
---
```

---

**Active Conversation UUID**:
