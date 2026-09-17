# ADR-0001: Database Choice for Transactional Core

**Status:** Recommended (pending your confirmation — this is a deliberate deviation from `base_server`'s default)

## Context

`base_server` ships with **MongoDB via Mongoose** as its default persistence layer. Our top-ranked quality goal (see 01-introduction-and-goals.md) is **Data Integrity**: orders must never double-charge a customer, inventory must never be oversold, and payment records must never be lost or duplicated. Our #2 goal is **Performance & Scalability**, which explicitly calls for reasoning about caching.

The project is confirmed **single-vendor** (not a multi-tenant marketplace), which matters: it limits how much schema flexibility we actually need for the catalog, and rules out justifying heavy polyglot infrastructure on "future multi-tenant" grounds.

## Options Considered

**A. Keep pure MongoDB (base_server's default)**
- (+) Zero migration effort from the given scaffold; Mongoose already familiar.
- (+) Flexible schema, convenient for product variants (size/color/attributes).
- (–) Multi-document ACID transactions exist since MongoDB 4.0 but are heavier and less idiomatic than relational transactions for money-critical flows (order create + stock decrement + payment record, all-or-nothing).
- (–) Cross-entity reporting (sales by day, best-sellers, revenue by category) requires aggregation pipelines that get complex fast compared to SQL joins.

**B. Full polyglot: MongoDB (catalog) + PostgreSQL (orders/payments) + Redis (cache)**
- (+) "Best tool for each job" in theory.
- (–) Three data stores to operate, back up, and monitor, for a *single-vendor* store — this is disproportionate operational complexity for a solo developer. It directly violates the YAGNI/KISS principles we established earlier: the multi-tenant catalog flexibility that would justify a separate document store isn't a real requirement here.

**C. PostgreSQL as the single source of truth (JSONB columns for flexible product attributes) + Redis for caching/sessions/cart**
- (+) ACID guarantees exactly where they matter most: Orders, Payments, Inventory.
- (+) Mature relational reporting for the business-facing side (sales, inventory, revenue queries).
- (+) JSONB columns give most of Mongo's schema flexibility for product variants without a second database engine.
- (+) Redis is directly justified by the explicitly stated need to reason about caching — cache-aside for product reads, session storage, cart storage, and can also replace `base_server`'s current Mongo-backed rate limiter with a faster, more standard Redis-backed one.
- (–) Real rework required: `base_server`'s existing Mongoose-based user/auth module needs to be re-implemented against PostgreSQL (via Prisma or TypeORM) — this is not a free change.
- (–) Loses some of Mongo's "just add a field" convenience for genuinely unstructured data (mitigated by JSONB where it's actually needed).

## Decision

**Option C.** PostgreSQL for the transactional core (Users, Products, Orders, OrderItems, Payments, Inventory, Addresses), with JSONB for flexible product attributes, plus Redis for caching, sessions, cart, and rate-limiting.

This is a conscious deviation from `base_server`'s shipped MongoDB default. The Router → Controller → Service layering pattern from `base_server` is kept; only the Repository/Model layer's underlying engine changes.

## Consequences

- **Positive:** Strong consistency exactly where money and stock are involved; simpler single-database operations (one primary store to manage, not three); Redis gives genuine, justified caching rather than caching bolted on as an afterthought.
- **Negative:** The existing `base_server` auth/user Mongoose code needs re-implementation, not reuse as-is. This is real, non-trivial migration work that should be scheduled as an explicit early task, not discovered mid-build.
- **Revisit if:** the business later needs a genuine multi-vendor marketplace with wildly varying per-seller product schemas — at that point, re-evaluating a document store for the catalog specifically becomes justified again.
