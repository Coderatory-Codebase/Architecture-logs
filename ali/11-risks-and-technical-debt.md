# 11. Risks & Technical Debt

*(arc42 Section 11 — Known problems, risks, and technical debt, with mitigation notes)*

| Risk | Likelihood/Impact | Mitigation |
|---|---|---|
| **Mongo → PostgreSQL migration effort underestimated** | High impact | `base_server`'s auth module is not a drop-in reuse under ADR-0001 — this should be the *first* implementation task, scheduled and time-boxed on its own, not discovered mid-build. |
| **Partial failure between payment and inventory** (payment succeeds, but the inventory-decrement step fails due to a crash or bug) | Medium likelihood, high impact | Needs a reconciliation job that periodically checks for orders marked "paid" whose inventory step never completed, and completes or flags them. This is real work not yet designed in detail — flagged here deliberately rather than assumed away. |
| **Stuck reserved stock** (customer abandons checkout after reservation, before payment) | Medium likelihood, medium impact | Needs a scheduled job to release reservations older than a timeout window back to available stock. Not yet designed. |
| **Single-currency assumption (2.1)** | Low likelihood now, high rework if it changes | If international expansion later needs multi-currency, this touches Pricing, Orders, and Reporting simultaneously — worth revisiting early if there's any signal this is coming, rather than late. |
| **Solo-developer bus factor** | Structural, not a "bug" | This document itself is the main mitigation — decisions and reasoning are written down so the system remains maintainable even if memory fades or someone else eventually joins. |
| **Redis as a single managed instance (7)** | Low likelihood, medium impact | Cache-aside means a Redis outage degrades performance (falls back to DB) rather than causing an outage — but cart and session data living only in Redis means a Redis loss *does* lose active carts/sessions. Worth an explicit decision later on whether cart durability matters enough to also persist it. |
| **No load testing performed** | Certain gap at this stage | All "production-scale" reasoning here is design-time analysis, not measured. Before treating any capacity number as real, actual load testing against the built system is required — this document reasons about scale, it does not prove it. |
