# ADR-0003: Search Technology

**Status:** Recommended

## Context

The Search module needs to let customers find products by name/description/category. Options range from a dedicated search engine to using the primary database's built-in text search.

## Options Considered

**A. Elasticsearch (or similar dedicated search engine)**
- (+) Best-in-class relevance ranking, fuzzy matching, faceted search.
- (–) A fourth infrastructure piece to operate (on top of Postgres, Redis, and the API) for a *single-vendor* catalog that is not large. This repeats the same over-engineering pattern rejected in ADR-0001 for the database.

**B. PostgreSQL full-text search (`tsvector`/`tsquery`, optionally `pg_trgm` for fuzzy matching)**
- (+) No new infrastructure — already have PostgreSQL as the primary store (ADR-0001).
- (+) Sufficient relevance and fuzzy-matching quality for a single-vendor catalog of realistic size (hundreds to low thousands of SKUs, not millions).
- (–) Ranking/relevance features are less sophisticated than a dedicated engine at very large scale.

## Decision

**Option B.** PostgreSQL full-text search, with a `tsvector` column indexed (GIN index) on product name/description/category, plus `pg_trgm` for typo-tolerant matching.

## Consequences

- **Positive:** Zero additional infrastructure; consistent with the "single data store for the core, Redis only for caching" strategy from ADR-0001.
- **Negative:** If the catalog later grows dramatically or relevance quality becomes a real business complaint, this decision should be revisited — Elasticsearch becomes justified once the scale/quality trade-off actually shows up, not before.
