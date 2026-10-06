---
title: "Risk Discipline Governance: Master Risk Register"
tags: ["architecture", "risk", "project-management", "governance"]
created_at: "2026-10-06"
last_updated_at: "2026-10-06"
---

# Risk Discipline Governance: Master Risk Register

Governed by Barry Boehm's Software Risk Taxonomy and the Unified Process Project Management discipline.

---

## 1. Risk Exposure Scoring Model

Risk Exposure ($RE$) is quantified as:
$$RE = P \times I$$

* **Probability ($P$):** Integer scale `1` (Extremely Unlikely) to `5` (Near Certainty).
* **Impact ($I$):** Integer scale `1` (Negligible / Cosmetic) to `5` (Catastrophic / Complete Failure / Data Loss).
* **Risk Exposure ($RE$):** Product range `1` to `25`.

---

## 2. Thresholds and Action Gates

* **Critical ($RE \ge 16$):** Architectural blocker. Requires immediate mitigation spike or prototype before committing to construction.
* **Moderate ($9 \le RE \le 15$):** Monitored actively; mitigation plan required during Elaboration.
* **Low ($RE < 9$):** Documented; managed via standard procedural checks.
* **Retirement Policy:** When a mitigation is merged and verified, update status to `retired` and cite resolving commit/PR. Historical entries are never deleted.

---

## 3. Documents

* [`risk-list.md`](file:///home/michael/src/github.com/mnaatjes/EDMarketConnector/architecture/risk/risk-list.md): Active Barry Boehm Master Risk Register.
