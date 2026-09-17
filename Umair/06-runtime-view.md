# 6. Runtime View

*(arc42 Section 6 — Key runtime scenarios)*

## 6.1 Scenario: Checkout & Payment

```mermaid
sequenceDiagram
    participant C as Customer (Next.js)
    participant API as Backend API
    participant INV as Inventory
    participant DB as PostgreSQL
    participant PAY as Payments Module
    participant S as Stripe

    C->>API: POST /checkout (cart)
    API->>INV: reserve stock (within DB transaction)
    INV->>DB: decrement available, increment reserved
    API->>DB: create Order (status=pending)
    API->>PAY: create PaymentIntent
    PAY->>S: create PaymentIntent
    S-->>PAY: client_secret
    PAY-->>C: client_secret (customer completes payment on frontend)
    S->>API: webhook: payment_intent.succeeded
    API->>PAY: verify idempotency key (event ID)
    PAY->>DB: mark Order status=paid (single transaction)
    API->>INV: convert reserved stock to sold (same transaction)
    API->>C: order confirmation (via polling or websocket, TBD)
```

**Key design point:** stock reservation happens *before* payment confirmation (to prevent overselling during the payment gap), and is converted to a permanent decrement only inside the same transaction that marks the order paid. If payment fails or times out, a scheduled job releases reserved stock back to available (see 11-risks-and-technical-debt.md — this reconciliation job is a real piece of work, not automatic).

## 6.2 Scenario: Product Page Read (cache-aside)

```mermaid
sequenceDiagram
    participant C as Customer
    participant FE as Next.js (ISR)
    participant API as Backend API
    participant R as Redis
    participant DB as PostgreSQL

    C->>FE: GET /products/:slug
    alt ISR page still fresh
        FE-->>C: cached HTML (no backend call)
    else ISR page stale, regenerate
        FE->>API: GET /api/v1/products/:slug/page-data
        API->>R: GET product:slug
        alt cache hit
            R-->>API: product data
        else cache miss
            API->>DB: query product + variants + reviews
            DB-->>API: product data
            API->>R: SET product:slug (TTL)
        end
        API-->>FE: product data
        FE-->>C: rendered page (cached for next request window)
    end
```
