# 12. Glossary

*(arc42 Section 12 — Domain and technical terms)*

| Term | Definition |
|---|---|
| **SKU** | Stock Keeping Unit — a unique identifier for a specific sellable product variant (e.g., "T-Shirt, Red, Medium"). |
| **ADR** | Architecture Decision Record — a short document capturing one significant decision: context, options, decision, consequences. |
| **Idempotency Key** | A unique value attached to an operation (e.g., a Stripe event ID) so that repeating the operation has no additional effect beyond the first time. |
| **Cache-aside** | A caching pattern where the application checks the cache first; on a miss, it reads from the database and writes the result into the cache for next time. |
| **ISR (Incremental Static Regeneration)** | A Next.js feature that serves a pre-rendered page and regenerates it in the background after a set interval, rather than rendering fresh on every request. |
| **Webhook** | An HTTP callback that an external service (Stripe/PayPal) sends to our backend when an event happens on their side (e.g., a payment succeeding). |
| **PCI Scope** | The part of a system that touches raw card data and must comply with Payment Card Industry security standards. Delegating this to Stripe/PayPal keeps our own servers out of PCI scope. |
| **Stock Reservation** | Temporarily holding inventory for a customer during checkout, before payment is confirmed, to prevent overselling. |
| **Layered Monolith** | A single deployable application whose internal code is organized into horizontal layers (e.g., Router → Controller → Service → Repository), as opposed to being split into independently deployable microservices. |
