---
title: "Master Risk Register"
tags: ["architecture", "risk", "register"]
created_at: "2026-10-06"
last_updated_at: "2026-10-06"
---

# Master Risk Register

Barry Boehm Risk Exposure ($RE = P \times I$) log for the EDMarketConnector refactoring and modernization lifecycle.

---

## Active Risk Log

| Risk ID | Title / Risk Description | Probability ($P$) | Impact ($I$) | Exposure ($RE$) | Status | Mitigation Strategy |
| :--- | :--- | :---: | :---: | :---: | :---: | :--- |
| **RISK-01** | **Breaking Third-Party Plugin Compatibility:** Legacy external plugins depend on global variables, root script exports, and Tkinter hook callbacks. | 4 | 5 | **20** | active | Provide backward-compatible facade modules and adapt legacy hook lifecycles. |
| **RISK-02** | **Journal Log Schema Drift:** Undocumented game updates introduce unhandled fields, breaking strict Pydantic/dataclass parsers. | 4 | 4 | **16** | active | Implement resilient permissive ingestion with schema validation fallbacks. |
| **RISK-03** | **Circular Dependency Regressions:** Decoupling `config` backends risks regressions on Windows/Linux environments. | 3 | 4 | **12** | active | Introduce abstract repository interfaces (ports) and OS-specific dependency injection. |
| **RISK-04** | **Headless Testing Untestability:** UI intertwined with polling loops prevents automated headless CI execution. | 4 | 3 | **12** | active | Extract UI presentation into decoupled adapter and construct mock journal emitter. |
