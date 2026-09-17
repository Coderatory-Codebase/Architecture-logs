# 7. Deployment View

*(arc42 Section 7 — Hardware, infrastructure, deployment)*

```mermaid
graph TB
    subgraph "Edge / CDN"
        CDN[CDN - static assets, images]
    end
    subgraph "Frontend Hosting"
        FE[Next.js - Vercel or equivalent, ISR + Edge cache]
    end
    subgraph "Backend Infra"
        LB[Load Balancer]
        API1[API Instance 1 - stateless]
        API2[API Instance 2 - stateless]
        LB --> API1
        LB --> API2
    end
    subgraph "Data Layer"
        PG[(PostgreSQL - primary)]
        PGR[(PostgreSQL - read replica, for reporting)]
        R[(Redis - managed, cache+session+cart)]
    end
    FE --> CDN
    FE --> LB
    API1 --> PG
    API2 --> PG
    API1 --> R
    API2 --> R
    PG -.replicates.-> PGR
```

| Node | Notes |
|---|---|
| **Next.js hosting** | Chosen for its native ISR/edge-cache support, directly serving the Performance quality goal. |
| **Backend API instances** | Kept **stateless** deliberately — no in-memory session/cart state — so horizontal scaling is just "add another instance behind the load balancer," no sticky sessions needed. |
| **PostgreSQL primary + read replica** | Reporting/admin queries (sales dashboards) run against the replica so they never compete with checkout-path writes for connections. |
| **Redis** | Single managed instance to start; a solo-developer constraint (2.2) rules out running a self-managed Redis cluster from day one. Documented as a scaling lever, not a day-one requirement. |
| **Docker** | `base_server` already ships dev/prod Dockerfiles — reused as the containerization baseline for the API instances. |

**Deliberately deferred (not needed at current stated scale):** container orchestration (Kubernetes), multi-region deployment, database sharding. Introducing these now, for a single-vendor store's stated scale, would repeat the same YAGNI mistake flagged in ADR-0001 for the database.
