# Document 10 — Base Server Mapping

> Source: `Yango_Clone_Architecture.docx`
> Maps this architecture onto the existing "Base Server" starter codebase — what to keep, extend, remove, or add.

## 10.1 What Base Server Already Has

| Feature | Status | Action |
|---|---|---|
| Express + TypeScript | ✅ Ready | Keep as-is |
| MongoDB connection | ✅ Ready | Keep as-is |
| PostgreSQL connection | ✅ Ready | Keep as-is (notifications) |
| Socket.io | ✅ Ready | Extend — add auth middleware |
| JWT (access + refresh) | ✅ Ready | Keep logic — change auth method |
| Rate limiting | ✅ Ready | Keep + add OTP-specific limits |
| Error handling | ✅ Ready | Keep as-is |
| Standard response format | ✅ Ready | Keep as-is |
| Role-based auth | ✅ Ready | Keep + extend for DRIVER role |
| Driver CRUD | ✅ Ready | Extend significantly |
| GraphQL notifications | ❌ Remove | Replace with Socket.io + FCM only |
| Email auth flow | ❌ Remove | Replace entirely with OTP flow |

## 10.2 New Work Required

| Feature | Effort | Priority |
|---|---|---|
| Phone OTP auth (Twilio) | Medium | P1 — blocks everything |
| Socket.io JWT middleware | Low | P1 — security |
| Redis integration | Medium | P1 — location + OTP |
| Ride Service + lifecycle | High | P1 — core feature |
| Matching Service + atomic lock | High | P1 — core feature |
| Location Service + 200m detection | Medium | P1 — core feature |
| Admin APIs | Medium | P1 — driver approval |
| S3 document upload | Medium | P2 — before driver onboarding |
| FCM integration | Medium | P2 |
| Notification triggers | Medium | P2 |
| Fare Service + Google Maps | Low | P2 |
| Rating Service | Low | P3 |

## 10.3 Folder Structure

```
src/
├── APIs/
│   ├── index.ts
│   └── v1/
│       ├── auth.routes.ts          ← REWORK (OTP)
│       ├── user.routes.ts          ← EXTEND
│       ├── driver.routes.ts        ← EXTEND
│       ├── ride.routes.ts          ← NEW
│       ├── location.routes.ts      ← NEW
│       ├── fare.routes.ts          ← NEW
│       ├── rating.routes.ts        ← NEW
│       └── admin.routes.ts         ← NEW
│
├── services/
│   ├── auth.service.ts             ← REWORK
│   ├── user.service.ts             ← EXTEND
│   ├── driver.service.ts           ← EXTEND
│   ├── ride.service.ts             ← NEW
│   ├── matching.service.ts         ← NEW
│   ├── location.service.ts        ← NEW
│   ├── fare.service.ts             ← NEW
│   ├── rating.service.ts           ← NEW
│   └── notification.service.ts     ← EXTEND (add FCM)
│
├── repositories/
│   ├── user.repository.ts
│   ├── driver.repository.ts        ← EXTEND
│   ├── ride.repository.ts          ← NEW
│   └── rating.repository.ts        ← NEW
│
├── models/
│   ├── user.model.ts               ← UPDATE
│   ├── driver.model.ts             ← UPDATE
│   ├── ride.model.ts               ← NEW
│   └── rating.model.ts             ← NEW
│
├── socket/
│   ├── index.ts                    ← EXTEND
│   ├── middleware/
│   │   └── auth.middleware.ts      ← NEW (JWT verify)
│   └── handlers/
│       ├── location.handler.ts     ← NEW
│       └── ride.handler.ts         ← NEW
│
├── config/
│   ├── redis.config.ts             ← NEW
│   ├── fcm.config.ts               ← NEW
│   ├── twilio.config.ts            ← NEW
│   └── s3.config.ts                ← NEW
│
└── utils/
    ├── geo.utils.ts                 ← NEW
    └── fare.utils.ts                ← NEW
```

## 10.4 Auth Rework Plan

| Action | Detail |
|---|---|
| Remove | Email verification endpoints (/auth/email/verify, etc.) |
| Remove | Email service module and all email calls |
| Remove | Email-related environment variables |
| Remove | GraphQL notification endpoint |
| Add | POST /auth/send-otp → Twilio integration |
| Add | POST /auth/verify-otp → Redis check + JWT issue |
| Add | OTP rate limiting (attempts + cooldown Redis keys) |
| Add | Twilio SDK and config |
| Keep | JWT issue/refresh/logout logic — same flow, different trigger |

## 10.5 Environment Variables

```
# Redis
REDIS_HOST=
REDIS_PORT=6379
REDIS_PASSWORD=

# Twilio
TWILIO_ACCOUNT_SID=
TWILIO_AUTH_TOKEN=
TWILIO_PHONE_NUMBER=

# FCM
FCM_PROJECT_ID=
FCM_PRIVATE_KEY=
FCM_CLIENT_EMAIL=

# AWS S3 (document uploads)
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_REGION=
AWS_S3_BUCKET=

# Google Maps
GOOGLE_MAPS_API_KEY=
```

---
◀ Previous: [09 — Real-time Architecture](09-realtime-architecture.md) | Back to [README](../README.md) | Next → [11 — Deployment](11-deployment.md)
