# ADR-0002: API Style — REST vs GraphQL

**Status:** Recommended

## Context

The storefront (Next.js) is **read-heavy**: product listings, product detail pages, and category pages dominate traffic and directly benefit from HTTP-level caching (CDN edge caching, browser cache, `Cache-Control`/`ETag` headers, and Next.js ISR). Admin and checkout flows are comparatively write-heavy and lower-volume. Our #2 quality goal explicitly requires reasoning about caching and scaling.

`base_server` is already built as a REST-style layered app (Router → Controller → Service).

## Options Considered

**REST**
- (+) Native HTTP caching semantics work out of the box with CDNs and browsers — directly serves the caching/scaling quality goal.
- (+) Zero rework of `base_server`'s existing routing/controller pattern.
- (+) Pairs naturally with Next.js `fetch` + ISR (Incremental Static Regeneration) for product pages.
- (–) Risk of over-fetching/under-fetching for complex nested views (e.g., a product page needing product + reviews + related items in one round trip).

**GraphQL**
- (+) Flexible querying, single endpoint, avoids over/under-fetching for complex nested UI.
- (–) Typically served over a single `POST` endpoint — much harder to cache at the CDN/HTTP layer; requires extra caching infrastructure (persisted queries, Apollo/Mercurius-level caching) to get back what REST gives for free.
- (–) Added operational and learning surface not justified by a single-vendor storefront's actual query complexity — this is the same YAGNI concern as the polyglot-database option in ADR-0001.

## Decision

**REST**, with resource-based, versioned endpoints (`/api/v1/...`), plus a small number of deliberately **composite, purpose-built read endpoints** for complex pages (e.g., `GET /api/v1/products/:slug/page-data` returning product + reviews + related items together) — this avoids GraphQL's operational cost while still avoiding REST's classic multi-round-trip problem for the few views that need it.

## Consequences

- **Positive:** Full alignment with `base_server`'s existing foundation (no rework); CDN/HTTP caching is available immediately, directly serving the stated scaling goal; simpler for a solo developer to reason about and operate.
- **Negative:** Composite endpoints must be designed deliberately per-view rather than relying on generic CRUD — this is a small ongoing design discipline, not a one-time cost.
