# Yango Clone — Ride-Hailing Platform

Ride-hailing platform (Yango clone) built as a **Modular Monolith V1**, targeting a single-city launch in Pakistan (PKR currency, cash-only payments for V1).

| | |
|---|---|
| **Architecture Style** | Modular Monolith V1 → Microservices when needed |
| **Backend** | Node.js + TypeScript + Express (`base_server-master/`) |
| **Passenger / Driver Apps** | React Native (Android + iOS) |
| **Admin Panel** | Next.js (`frontend/`) |
| **Core Database** | MongoDB |
| **Cache / Live Location** | Redis (GEO commands) |
| **Notification Log DB** | PostgreSQL |
| **Real-time** | Socket.io + Firebase Cloud Messaging (FCM) |
| **OTP Provider** | Twilio |
| **Launch Scope** | Single city, PKR, cash-only V1 |

The three apps (Passenger, Driver, Admin) talk to one Express backend over REST + WebSocket. Core entities (users, drivers, rides, ratings, fare config) live in MongoDB; live driver GPS and OTPs live in Redis; notification history lives in PostgreSQL. Matching, ride lifecycle, and location tracking run through Socket.io in a single PM2 fork-mode process for V1.

## System Design Documentation

Full architecture documentation — extracted and organized from `Yango_Clone_Architecture.docx` — lives in [`System_Design/`](System_Design/):

| # | Document | Covers |
|---|---|---|
| 1 | [Requirements](System_Design/01-requirements.md) | Functional & non-functional requirements, key decisions, V1 scope |
| 2 | [System Context](System_Design/02-system-context.md) | System boundary, actors, external services (Google Maps, Twilio, FCM) |
| 3 | [High Level Architecture](System_Design/03-high-level-architecture.md) | Component layers, architecture diagram, ADR summary table |
| 4 | [Architecture Decision Records](System_Design/04-architecture-decision-records.md) | ADR-001 to ADR-009 — full context/choice/trade-off records |
| 5 | [Services Design](System_Design/05-services-design.md) | Service ownership, state machines, endpoints, matching/fare logic |
| 6 | [Database Schema](System_Design/06-database-schema.md) | MongoDB collections, Redis key design, PostgreSQL notifications table |
| 7 | [Critical Flows](System_Design/07-critical-flows.md) | OTP login, ride booking & matching, location tracking, cancellation, rating |
| 8 | [API Contract](System_Design/08-api-contract.md) | REST endpoints (auth, user, driver, ride, admin, rating) + WebSocket events |
| 9 | [Real-time Architecture](System_Design/09-realtime-architecture.md) | Socket.io connection flow, rooms strategy, disconnect handling, scaling path |
| 10 | [Base Server Mapping](System_Design/10-base-server-mapping.md) | What to keep/remove/extend in the existing base server, folder structure, env vars |
| 11 | [Deployment](System_Design/11-deployment.md) | Infrastructure, Docker Compose, Nginx config, CI/CD, backups, scale-up plan |
| 12 | [Issues Fixed — Review Summary](System_Design/12-issues-fixed-review-summary.md) | Critical/important/medium/minor issues found in expert review and their fixes |

### Key things to know before touching the backend

- **PM2 must run in fork mode, not cluster mode** — Socket.io breaks across processes without a Redis Adapter ([ADR-009](System_Design/04-architecture-decision-records.md#adr-009-pm2-fork-mode-not-cluster)).
- **All Socket.io connections must be JWT-authenticated**, and room joins must verify the user belongs to that ride ([Doc 9](System_Design/09-realtime-architecture.md)).
- **Ride acceptance uses an atomic Redis lock** (`SET ride:matching:{rideId} {driverId} NX EX 30`) to prevent two drivers accepting the same ride ([Doc 7](System_Design/07-critical-flows.md)).
- **A MongoDB partial unique index** on `rides.passengerId` prevents duplicate active ride bookings ([Doc 6](System_Design/06-database-schema.md)).
- **Driver documents must go to S3**, never local disk ([Doc 5](System_Design/05-services-design.md)).
- The old **email-auth flow and GraphQL notifications endpoint are being removed**, replaced by phone-OTP auth (Twilio) and Socket.io + FCM only ([Doc 10](System_Design/10-base-server-mapping.md)).

## Repository Layout

```
RideApp/
├── base_server-master/   # Node.js + TypeScript + Express backend
├── frontend/              # Next.js admin panel
└── System_Design/         # Architecture documentation (see index above)
```
