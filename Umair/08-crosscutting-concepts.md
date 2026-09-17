# 8. Crosscutting Concepts

*(arc42 Section 8 — Overarching technical topics that recur across modules)*

| Concept | Approach |
|---|---|
| **Authentication** | JWT access + refresh tokens in httpOnly/secure/sameSite cookies — reused from `base_server` as-is; this pattern was already sound in the original review. |
| **Authorization** | Role check middleware (Customer vs Admin) applied per-route, extending `base_server`'s existing `authenticate` middleware. |
| **Error Handling** | Centralized error middleware + `errorObject` builder, reused from `base_server`. **Fix applied:** the original codebase's redundant per-controller try/catch (on top of `asyncHandler` already forwarding to `next()`) is removed — one mechanism, not two, per the DRY finding from the earlier code review. |
| **Validation** | Joi schemas per-request, reused from `base_server`'s pattern. |
| **Rate Limiting** | Moved from Mongo-backed to **Redis-backed** (ADR-0001 already justifies Redis's presence). Applied to **all** auth and checkout endpoints — closing the gap found in the original `base_server` review, where only `/self` was protected. |
| **Caching** | Cache-aside pattern via Redis for catalog reads; explicit invalidation on product/inventory update (not just TTL expiry, to avoid serving stale stock counts). |
| **Idempotency** | Payment webhooks (Stripe/PayPal) are keyed by their event ID in a dedicated idempotency table — a retried webhook is a no-op, not a duplicate order update. This is the single most important crosscutting concern in the whole system, directly protecting the #1 quality goal (Data Integrity). |
| **Logging & Observability** | Winston (reused from `base_server`) extended with a **request-correlation ID** attached at the edge and propagated through service calls — this closes the observability gap flagged in the original review, where no request tracing existed. |
| **API Versioning** | `/api/v1/...` prefix from day one, so breaking changes later don't require a big-bang migration. |
| **Security Headers/CORS** | `helmet` + CORS reused from `base_server`, with the CORS origin now read from environment config instead of the hardcoded placeholder found during the review. |
