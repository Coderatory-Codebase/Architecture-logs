# 9. Architecture Decisions

*(arc42 Section 9 — Important, expensive, risky, or contentious decisions. Full detail lives in `/adr` as individual Architecture Decision Records; this section is the index.)*

| ADR | Decision | Why it matters |
|---|---|---|
| [0001](./adr/0001-database-choice.md) | PostgreSQL + Redis, deviating from `base_server`'s default MongoDB | Highest-impact deviation from the given scaffold; protects the #1 quality goal (Data Integrity). Requires re-implementing `base_server`'s auth/user module against the new store. |
| [0002](./adr/0002-api-style.md) | REST over GraphQL, with composite read endpoints | Directly serves the caching/scalability quality goal; keeps `base_server`'s existing routing pattern intact. |
| [0003](./adr/0003-search-technology.md) | PostgreSQL full-text search over Elasticsearch | Avoids adding a fourth infrastructure piece the stated scale doesn't justify. |

New ADRs should be added here as future contentious decisions arise (e.g., if the business later asks for multi-currency, multi-vendor, or a dedicated search engine) — each gets its own file in `/adr`, numbered sequentially, never edited retroactively once accepted (a superseding decision gets a *new* ADR that references the old one).
