# 10. Quality Requirements

*(arc42 Section 10 — Detailed quality scenarios, linked to the ranked goals in Section 1)*

| # | Quality Attribute | Scenario | Stimulus | Expected Response |
|---|---|---|---|---|
| 1 | Data Integrity | Two customers try to buy the last unit of a product simultaneously | Concurrent checkout requests hit the same SKU | Only one order succeeds in reserving the final unit; the other is told "out of stock" before payment is attempted — never after a customer is charged |
| 1 | Data Integrity | Stripe redelivers a webhook (network retry) | Duplicate `payment_intent.succeeded` event received | Order is marked paid exactly once; no duplicate confirmation email, no double inventory decrement |
| 2 | Performance | Product page requested during a traffic spike (e.g., a sale event) | 10x normal request volume on top product pages | Served from Next.js ISR cache / Redis cache-aside — database load stays flat, not proportional to traffic |
| 2 | Scalability | Sustained increase in checkout volume | Backend CPU/connection saturation on existing instances | New stateless API instance added behind the load balancer with no code change and no session-affinity issues |
| 3 | Security | Attacker attempts credential-stuffing against login | Repeated failed login attempts from one IP/account | Redis-backed rate limiter blocks further attempts after threshold, on the login endpoint specifically (the gap found in the original `base_server` review) |
| 4 | Maintainability | New developer joins and needs to add a "wishlist" feature | Requirement to add a new domain module | Can be added as a new `src/APIs/wishlist/` module following the existing Router→Controller→Service→Repository convention, without touching unrelated modules |
| 5 | Usability | Customer opens a product page on mobile with average connection | Page load request | First meaningful content visible without waiting on a live database round-trip (served via ISR) |
