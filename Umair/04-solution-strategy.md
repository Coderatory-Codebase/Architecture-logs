# 4. Solution Strategy

*(arc42 Section 4 — Fundamental decisions and solution approaches)*

## 4.1 Technology Decisions (summary — full reasoning in `/adr`)

| Layer | Choice | ADR |
|---|---|---|
| Backend framework | Express + TypeScript, layered (`base_server` foundation) | — (given constraint) |
| Database (core) | PostgreSQL | ADR-0001 |
| Cache/Session/Cart | Redis | ADR-0001 |
| API style | REST, versioned, composite read endpoints | ADR-0002 |
| Search | PostgreSQL full-text (`tsvector`) | ADR-0003 |
| Frontend | Next.js (SSR + ISR) | given constraint |
| Payments | Stripe (primary) + PayPal (secondary) | given constraint |

## 4.2 How Each Quality Goal Is Addressed

| Quality Goal | Strategy |
|---|---|
| **Data Integrity** | PostgreSQL ACID transactions wrap order creation + inventory decrement as a single atomic unit. Payment webhooks are processed idempotently (idempotency key stored per Stripe/PayPal event ID) so retried webhooks never double-apply. |
| **Performance & Scalability** | Redis cache-aside for product reads (short TTL, invalidated on product update). Next.js ISR for storefront pages (regenerate on a timer, not on every request). Backend API kept stateless (session data in Redis, not in-process) so it can scale horizontally behind a load balancer. |
| **Security** | Card data never touches our servers — Stripe/PayPal handle PCI-scoped data via hosted fields/redirect flows. Auth reuses `base_server`'s httpOnly + secure + sameSite cookie pattern. Rate limiting moves from Mongo-backed to Redis-backed and is applied to *all* auth and checkout endpoints (fixing the gap found during the `base_server` review). |
| **Maintainability** | Layered structure preserved from `base_server` (Router → Controller → Service → Repository). Every non-obvious decision is captured as an ADR instead of living only in someone's memory. |
| **Usability** | Next.js ISR keeps product pages fast without full SSR cost on every request; composite REST endpoints (ADR-0002) avoid multi-round-trip loading spinners on key pages. |

## 4.3 Top-Level Approach

The system stays a **layered monolith** (not microservices) — a single deployable backend, internally organized into clear domain modules (Catalog, Cart, Checkout/Orders, Payments, Inventory, Auth, Admin, Notifications, Search). This is a deliberate choice: a single-vendor store at the stated scale does not have independent-team or independent-scaling needs that would justify microservices' operational cost (same YAGNI reasoning applied throughout this document). If a specific module (e.g., Search, or image processing) later needs independent scaling, it can be extracted later — the layered internal structure makes that extraction easier, not harder.
