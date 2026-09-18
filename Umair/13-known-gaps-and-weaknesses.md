# 13. Known Gaps & Weaknesses

Gaps identified in the current architecture that must be resolved before implementation, organized by severity.

---

## Major

**GAP-01 — Payment webhooks have no signature verification**
- **Evidence:** `03-context-and-scope.md:34–40` identifies inbound webhooks; `08-crosscutting-concepts.md:13` defines event-ID idempotency only.
- **Problem:** The architecture protects against duplicate webhooks but not forged or stale provider callbacks.
- **Impact:** An attacker could mark an unpaid order as paid.
- **Recommendation:** Verify signatures over the raw payload, enforce a replay window, check provider account context, and add tests for verification failure.
- **Status:** Open

**GAP-02 — Checkout compensation is not designed**
- **Evidence:** `06-runtime-view.md:16–31` spans a database transaction and an external PaymentIntent; `11-risks-and-technical-debt.md:8–9` acknowledges reconciliation/release jobs are not yet designed.
- **Problem:** An external payment call cannot participate in the local ACID transaction — recovery from partial failure is part of the core correctness model, not an optional extra.
- **Impact:** Orders, payments, and reserved stock can diverge after crashes, timeouts, or checkout abandonment.
- **Recommendation:** Define saga states, deadlines, idempotent retries, a reconciliation job, alerting, and a manual-repair path.
- **Status:** Open

**GAP-03 — Order/Payment state machines are missing**
- **Evidence:** `05-building-block-view.md:26–28` names lifecycle ownership; `06-runtime-view.md` only ever uses `pending` and `paid`.
- **Problem:** Failure, expiry, cancellation, refund, chargeback, fulfillment, and partial states have no defined transitions or owning module.
- **Impact:** Different modules can make incompatible decisions about the same order; customers see unreliable status.
- **Recommendation:** Publish separate state machines (Order, Payment, Reservation) with explicit transition authority and side effects per transition.
- **Status:** Open

## Moderate

**GAP-04 — Authorization is role-only**
- **Evidence:** `08-crosscutting-concepts.md:8` defines Customer vs Admin route checks but no order/cart/address/payment ownership predicates.
- **Impact:** A customer may be able to access another customer's data by guessing/incrementing an identifier.
- **Recommendation:** Define and test resource-level (record-level) authorization for every operation, not just role gates.

**GAP-05 — Fire-and-forget email cannot guarantee non-duplication**
- **Evidence:** `03-context-and-scope.md:38`, `05-building-block-view.md:30` use fire-and-forget dispatch; `10-quality-requirements.md:8` promises no duplicate confirmation email.
- **Impact:** Crashes lose notifications; naive retries duplicate them — the current design cannot deliver on its own stated requirement.
- **Recommendation:** Use a transactional outbox pattern with idempotent delivery records.

**GAP-06 — Single Redis instance removes required user state on failure**
- **Evidence:** `07-deployment-view.md:23,39` uses one Redis instance; `11-risks-and-technical-debt.md:12` already acknowledges cart/session loss.
- **Impact:** A Redis incident logs users out and destroys in-progress carts — Redis is not merely a cache in this design.
- **Recommendation:** Choose and document persistence/replication, cart reconstruction, session fallback behavior, and RPO/RTO targets.

**GAP-07 — API and webhook schemas are absent**
- **Evidence:** Only `/api/v1` and one page-data endpoint are defined; no checkout, order, admin, payment, or provider payload contracts exist.
- **Impact:** Independent components and contract tests can silently disagree with each other.
- **Recommendation:** Publish OpenAPI specs and webhook/event schemas, including error shapes and versioning/compatibility rules.

**GAP-08 — Quality scenarios lack a test architecture**
- **Evidence:** `10-quality-requirements.md:5–13` defines useful scenarios; `11-risks-and-technical-debt.md:13` states capacity is unmeasured.
- **Impact:** The top integrity and scale claims in this document remain unproven assertions, not verified guarantees.
- **Recommendation:** Map every quality scenario to an automated test type (unit/integration/contract/load/chaos), with environment, thresholds, and ownership.

## Minor

**GAP-09 — ADR status conflicts with its use as a decision**
- **Evidence:** `adr/0001-database-choice.md:3` says "Recommended (pending your confirmation)" while `04-solution-strategy.md:10–13` and later views treat PostgreSQL + Redis as already selected.
- **Impact:** Implementers can't tell whether the foundational persistence choice is actually approved.
- **Recommendation:** Mark the ADR `Accepted`, or keep every dependent section explicitly provisional until it is.

---

## Remediation Priority

1. **GAP-01** — payment forgery (active security hole, highest exploitability)
2. **GAP-02** — checkout compensation (core correctness gap)
3. **GAP-03** — state machines (needed before 01/02 can be specified precisely)
4. GAP-04 through GAP-08, in any order
5. **GAP-09** — cheap fix, do alongside the others
