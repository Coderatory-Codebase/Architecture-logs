Document 3 — High Level Architecture


3.1 Architecture Style
🏗️ ADR-001 — Modular Monolith V1
Microservices not used. Team is small, launch scale unknown, deploy/debug/maintain must be easy. Single deployable unit with clean internal module boundaries. Extract individual services when: (a) one module becomes a clear bottleneck, or (b) team size reaches 5+.

3.2 Major Components
Client Layer
Component	Technology	Purpose
Passenger App	React Native	Android + iOS — single codebase
Driver App	React Native	Android + iOS — single codebase
Admin Panel	Next.js	SSR, same JS ecosystem, fast

API Layer
Component	Purpose
Entry Point / Router	Request entry, route dispatch (NOT a full API Gateway product)
Rate Limiter	Per-IP throttling — already in Base Server
Auth Middleware	JWT verify on every protected route
Role Middleware	ADMIN vs USER vs DRIVER access control

⚠️ Terminology Note
"API Gateway" in docs refers to the Express Entry Point, not a product like Kong or AWS API Gateway. Team must not confuse the two.

Service Layer
Service	Responsibility
Auth Service	OTP send/verify, JWT issue/refresh
User Service	Passenger profile management
Driver Service	Driver profile, documents, approval state
Location Service	Live driver location — Redis GEO
Ride Service	Full ride lifecycle management
Matching Service	Nearest driver finding + timeout handling
Fare Service	Estimate and final fare calculation
Notification Service	FCM push + Socket.io events
Rating Service	Store ratings, update averages

Data Layer
Store	Technology	What Lives Here
Core Data	MongoDB	Users, drivers, rides, ratings, fare config
Live Location + Cache	Redis	Driver GEO, OTPs, matching state, refresh tokens
Notification Log	PostgreSQL	Notification history, read receipts

3.3 Architecture Diagram
┌─────────────────────────────────────────────────────────┐
│                     CLIENT LAYER                         │
│  [Passenger App]   [Driver App]   [Admin Panel]          │
│  React Native      React Native   Next.js                │
└──────────────────────┬──────────────────────────────────┘
                       │  REST + WebSocket
                       ▼
┌─────────────────────────────────────────────────────────┐
│                      API LAYER                           │
│   Rate Limiter → Auth Middleware → Router                │
└──────────────────────┬──────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────┐
│                   SERVICE LAYER                          │
│  Auth    │ User   │ Driver   │ Ride    │ Matching        │
│  Location│ Fare   │ Rating   │ Notification              │
└──┬───────────────────────────────────┬─────────────────┘
   │                                   │
┌──▼──────────┐  ┌──────────┐  ┌──────▼──────┐
│   MongoDB   │  │  Redis   │  │ PostgreSQL  │
│  (core data)│  │(location)│  │(notif logs) │
└─────────────┘  └──────────┘  └─────────────┘
                       │
         ┌─────────────┼─────────────┐
    ┌────▼────┐  ┌─────▼──┐  ┌──────▼──┐
    │G. Maps  │  │ Twilio │  │  FCM    │
    └─────────┘  └────────┘  └─────────┘

3.4 Technology Decisions & ADRs
ADR	Decision	Choice	Key Reason
ADR-001	Architecture style	Modular Monolith	Small team, unknown scale, easy deploy
ADR-002	Core database	MongoDB	Flexible schema for evolving ride/user/driver data
ADR-003	Live location store	Redis GEO	100 writes/sec — in-memory microsecond reads
ADR-004	Push notifications	FCM	Free, cross-platform, works when socket disconnected
ADR-005	OTP delivery	Twilio	Reliable Pakistan delivery; monitor SMS cost
ADR-006	Mobile apps	React Native	One codebase — Android + iOS
ADR-007	Admin panel	Next.js	SSR, fast, same JS ecosystem
ADR-008	Socket.io scaling	Single instance V1	Add Redis Adapter at 2000+ concurrent connections
ADR-009	Process manager	PM2 fork mode	Cluster mode breaks Socket.io without Redis Adapter

🔴 CRITICAL — ADR-009: PM2 Fork Mode
PM2 cluster mode spawns multiple processes. Socket.io connections are process-specific — without Redis Adapter, sockets break across processes. V1 MUST use PM2 fork mode (single process). Redis Adapter added when scaling beyond single box.
