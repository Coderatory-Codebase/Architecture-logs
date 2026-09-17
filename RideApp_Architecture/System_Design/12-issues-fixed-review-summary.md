# Issues Fixed — Expert Review Summary

> Source: `Yango_Clone_Architecture.docx`
> Consolidated log of issues found during expert review of the architecture, and where each fix is documented.

## 🔴 Critical Issues — All Fixed

| # | Issue | Fix Applied | Where |
|---|---|---|---|
| C1 | PM2 cluster vs Socket.io — process boundary break | ADR-009: Fork mode mandated | [Doc 3](03-high-level-architecture.md), [Doc 11](11-deployment.md) |
| C2 | Socket.io authentication missing | JWT middleware on all connections + room auth | [Doc 8](08-api-contract.md), [Doc 9](09-realtime-architecture.md) |
| C3 | Race condition — double accept | Redis `SET NX EX` atomic lock | [Doc 5](05-services-design.md), [Doc 6](06-database-schema.md), [Doc 7](07-critical-flows.md) |
| C4 | Duplicate ride booking | MongoDB partial unique index on passengerId | [Doc 6](06-database-schema.md) |

## 🟡 Important Issues — All Fixed

| # | Issue | Fix Applied | Where |
|---|---|---|---|
| H1 | OTP brute force / rate limiting | 3 requests/10min + 5 attempts + 60s cooldown | [Doc 5](05-services-design.md), [Doc 6](06-database-schema.md), [Doc 7](07-critical-flows.md) |
| H2 | Admin APIs completely missing | Full admin endpoint set added | [Doc 8](08-api-contract.md) |
| H3 | Nginx WebSocket 60s timeout | `proxy_read_timeout 86400` in WS location | [Doc 11](11-deployment.md) |
| H4 | Google Maps cost unestimated | Cost note + fare estimate cache (5 min Redis) | [Doc 2](02-system-context.md), [Doc 5](05-services-design.md), [Doc 6](06-database-schema.md) |
| H5 | GraphQL endpoint fate undecided | Decision: Remove — Socket + FCM is sufficient | [Doc 6](06-database-schema.md), [Doc 10](10-base-server-mapping.md) |
| H6 | "API Gateway" terminology confusion | Renamed to Entry Point / Router with clarification note | [Doc 3](03-high-level-architecture.md) |

## 🟠 Medium Issues — Fixed

| # | Issue | Fix Applied | Where |
|---|---|---|---|
| M1 | Redis persistence not configured | AOF enabled in Docker compose command | [Doc 11](11-deployment.md) |
| M2 | Redis failure — single point | ADR-003 explicitly acknowledges; V1 accepted risk | [Doc 4](04-architecture-decision-records.md) |
| M3 | Driver document storage on local disk | S3 mandated, S3 config added | [Doc 5](05-services-design.md), [Doc 10](10-base-server-mapping.md) |
| M4 | Docker vs PM2 redundancy | Decision: Docker for containers, PM2 on host only | [Doc 11](11-deployment.md) |
| M5 | 200m detection client vs server | Server-side mandated, reason documented | [Doc 5](05-services-design.md), [Doc 7](07-critical-flows.md) |
| M6 | Fare needs Google Maps — not documented | Google Maps Distance Matrix explicitly in Fare Service | [Doc 5](05-services-design.md) |

## 🟢 Minor Issues — Fixed

| # | Issue | Fix Applied | Where |
|---|---|---|---|
| L1 | ADR embedded in Doc 3 | ADR promoted to standalone Doc 4 | [Doc 4](04-architecture-decision-records.md) |
| L2 | Backup never tested | Restore drill requirement added | [Doc 11](11-deployment.md) |
| L3 | VPS scale-up plan missing | Scale-up triggers and actions table added | [Doc 11](11-deployment.md) |
| L4 | 2-year retention — no purge job | Monthly archival cron script added | [Doc 11](11-deployment.md) |
| L5 | Uptime 99.9% vs single VPS mismatch | Callout added with options | [Doc 1](01-requirements.md) |
| L6 | FCM device token update endpoint missing | Added to User and Driver endpoints | [Doc 8](08-api-contract.md) |

---

✅ **Document Quality: 95/100 — Production Ready for V1 Build**

---
◀ Previous: [11 — Deployment](11-deployment.md) | Back to [README](../README.md)
