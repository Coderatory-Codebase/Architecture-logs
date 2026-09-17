# Document 2 — System Context

> Source: `Yango_Clone_Architecture.docx`

## 2.1 System Boundary

| Inside System | Outside System (External) |
|---|---|
| Passenger App (React Native) | Google Maps Platform |
| Driver App (React Native) | Twilio (SMS / OTP) |
| Admin Panel (Next.js) | FCM — Firebase Cloud Messaging |
| Backend API Server (Node.js + Express) | Payment Gateway (V2 only) |
| MongoDB — Core Data | — |
| PostgreSQL — Notification Logs | — |
| Redis — Location / Cache / OTP | — |
| Socket.io — Real-time Server | — |
| Notification Service (FCM + Socket) | — |

## 2.2 Actors

| Actor | Role | Primary Actions |
|---|---|---|
| Passenger | End user booking rides | Book, track, pay, rate |
| Driver | Service provider | Accept rides, stream location, complete rides |
| Admin | System operator | Approve drivers, manage fares, monitor rides |

## 2.3 External Services

| Service | Purpose | Note |
|---|---|---|
| Google Maps Platform | Location search, routing, distance/fare calculation | Billing on client account; ~2000 calls/day at 500 rides |
| Twilio | Phone OTP delivery | Per-SMS cost — rate limit to prevent abuse |
| FCM | Background push notifications | Free tier sufficient V1; socket only works foreground |
| Payment Gateway | In-app payments | V2 only — Stripe or local provider |

> 💡 **Google Maps Cost Estimate** — 500 rides/day × 3-4 API calls per ride = 1,500–2,000 calls/day. Monitor billing from Day 1. Add 5-minute cache on fare estimates for same pickup/drop to reduce calls.

## 2.4 High-Level Interaction

```
┌──────────────────────────────────────────────────────┐
│                    CLIENT LAYER                       │
│   Passenger App     Driver App     Admin Panel        │
└────────────┬──────────────┬────────────────┬─────────┘
             │ REST          │ WebSocket       │ REST
             └──────────────▼─────────────────┘
                     ┌────────────────┐
                     │   API Gateway  │
                     │  (Entry Point) │
                     └───────┬────────┘
                             │
          ┌──────────────────▼──────────────────┐
          │           BACKEND SERVICES           │
          └──┬───────────┬──────────┬────────────┘
             │           │          │
        ┌────▼───┐  ┌───▼───┐  ┌──▼──────┐
        │MongoDB │  │ Redis │  │Postgres │
        └────────┘  └───────┘  └─────────┘
             │           │
        ┌────▼──────┐  ┌▼──────┐  ┌────────┐
        │Google Maps│  │Twilio │  │  FCM   │
        └───────────┘  └───────┘  └────────┘
```

## 2.5 Assumptions

- Single city launch
- Internet connection required — no offline mode
- Driver phone supports Android and iOS
- Google Maps billing is on the client account
- Fresh database — no migration from existing system

---
◀ Previous: [01 — Requirements](01-requirements.md) | Back to [README](../README.md) | Next → [03 — High Level Architecture](03-high-level-architecture.md)
