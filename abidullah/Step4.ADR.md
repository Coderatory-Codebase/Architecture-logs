Document 4 — Architecture Decision Records

Each ADR documents one significant decision: context, choice, alternatives considered, and trade-offs. These are permanent records — never deleted, only superseded.

ADR-001: Modular Monolith over Microservices
Field	Detail
Status	Accepted
Decision	Build V1 as modular monolith
Context	Small team, launch scale unknown, fast iteration needed
Choice	Single deployable unit with clean internal module separation
Alternatives	Microservices from Day 1
Trade-offs	Will require extraction work when scaling — accepted cost for V1 speed
Trigger to revisit	Single module bottleneck identified, OR team size reaches 5+

ADR-002: MongoDB for Core Data
Field	Detail
Status	Accepted
Decision	MongoDB as primary database for users, drivers, rides, ratings
Context	Schema will evolve from V1 to V2; document model fits nested structures
Choice	MongoDB — flexible schema, JS ecosystem native, good for ride app data
Alternatives	PostgreSQL for everything
Trade-offs	No native joins — handled at application layer. Acceptable for this schema

ADR-003: Redis GEO for Live Location
Field	Detail
Status	Accepted
Decision	Redis GEO commands for driver live location storage and retrieval
Context	500 drivers × 1 update/5s = 100 writes/sec at peak
Choice	Redis GEOADD/GEORADIUS — O(N+log N) complexity, in-memory microsecond reads
Alternatives	MongoDB geospatial collection
Trade-offs	Redis restart = location data loss. Acceptable — location is real-time, not historical
Note	Location TTL set to 30s — driver auto-exits matching pool if disconnected

ADR-004: FCM for Push Notifications
Field	Detail
Status	Accepted
Decision	Firebase Cloud Messaging for background push
Context	Socket.io only works when app is foreground. Background events need separate channel
Choice	FCM — free, cross-platform (Android + iOS), reliable
Alternatives	OneSignal, APNs direct, local FCM alternative
Trade-offs	Google dependency — acceptable for V1

ADR-005: Twilio for OTP
Field	Detail
Status	Accepted
Decision	Twilio as primary SMS/OTP provider
Context	Phone OTP is V1 auth mechanism. Pakistan delivery reliability critical
Choice	Twilio — reliable Pakistan delivery, well-documented SDK
Alternatives	Local providers (Infobip, Netnucleus) — cheaper per SMS
Trade-offs	Higher per-SMS cost vs local. Implement rate limiting to control cost
Action	Monitor monthly SMS cost; evaluate local provider at 1000+ OTPs/day

ADR-006: React Native for Mobile Apps
Field	Detail
Status	Accepted
Decision	React Native for both Passenger and Driver apps
Context	Android + iOS needed; team is JS/TS
Choice	Single codebase — React Native
Alternatives	Flutter, Native Swift/Kotlin
Trade-offs	Slightly lower performance than native — acceptable for ride app use case

ADR-007: Next.js for Admin Panel
Field	Detail
Status	Accepted
Decision	Next.js for Admin Panel
Context	Admin panel needs fast load, SSR beneficial, same JS ecosystem
Choice	Next.js
Alternatives	React SPA, Vue
Trade-offs	Minimal — best fit for this use case

ADR-008: Single Socket.io Instance V1
Field	Detail
Status	Accepted
Decision	Single Socket.io server instance for V1
Context	500 concurrent rides = 1000 sockets — single instance handles this
Choice	Single instance, no Redis Adapter V1
Alternatives	Redis Adapter from Day 1
Trade-offs	Cannot scale horizontally without Redis Adapter. Add it at 2000+ connections
Trigger	Add Redis Adapter + sticky sessions when concurrent connections exceed 2000

ADR-009: PM2 Fork Mode (Not Cluster)
Field	Detail
Status	Accepted — Critical
Decision	PM2 in fork mode for V1 (single process)
Context	PM2 cluster mode spawns multiple Node processes. Socket.io is process-bound — connections break across processes without Redis Adapter
Choice	PM2 fork mode — single process, restart on crash
Alternatives	PM2 cluster mode + Redis Adapter from Day 1
Trade-offs	No multi-core utilization V1 — acceptable. Add cluster + Redis Adapter when needed
Config	mode: 'fork', instances: 1 in ecosystem.config.js
