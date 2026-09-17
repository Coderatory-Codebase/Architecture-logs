Document 5 — Services Design


5.1 Services Overview
Service	Owns	Does NOT Own
Auth Service	OTP, JWT, refresh tokens	User/driver profile
User Service	User (passenger) document	Ride data, payment data
Driver Service	Driver document, vehicle, docs, approval	Location data, ride data
Location Service	Redis GEO data (live locations)	Driver profile, ride data
Ride Service	Ride document, lifecycle	Matching logic, location, payment
Matching Service	Matching logic, timeout handling	Location data, ride data, driver data
Fare Service	Fare config, calculation logic	Ride data, payment
Notification Service	Notification log, device tokens	Business logic
Rating Service	Rating documents, average updates	User/driver profile (only requests update)

5.2 State Machines
Ride States
REQUESTED
    ↓ (driver accepts)
ACCEPTED
    ↓ (driver within 200m)
DRIVER_ARRIVING
    ↓ (driver taps start)
STARTED
    ↓ (driver taps complete)
COMPLETED

CANCELLED ← from REQUESTED or ACCEPTED only
             (cancelledBy: PASSENGER | DRIVER | SYSTEM)
             STARTED rides cannot be cancelled

Driver States
PENDING    → (Admin approves)    → APPROVED
APPROVED   → (Driver goes online) → ACTIVE
ACTIVE     → (Admin action)       → SUSPENDED
SUSPENDED  → (Admin reinstates)   → APPROVED

5.3 Service Details
Auth Service
Endpoint	Method	Auth	Action
/auth/send-otp	POST	No	Generate OTP → Redis (5 min TTL) → Twilio SMS
/auth/verify-otp	POST	No	Redis check → match → delete → JWT issue
/auth/refresh	POST	No	Refresh token → new access token
/auth/logout	POST	Yes	Invalidate refresh token in Redis

🔴 OTP Security — Rate Limiting Required
Max 3 OTP requests per phone per 10 minutes. Max 5 verify attempts — then OTP invalidated. 60-second resend cooldown. Without this, Twilio bill can be exploited.

Driver Service
Endpoint	Method	Auth	Action
/driver/profile	GET	Driver	Fetch driver profile
/driver/profile	PATCH	Driver	Update profile
/driver/documents	POST	Driver	Upload license, vehicle docs → S3
/driver/status	PATCH	Driver	Toggle online/offline
/driver/earnings	GET	Driver	Earnings summary
/driver/ride-history	GET	Driver	Paginated ride history

⚠️ Document Storage
Driver documents (license, vehicle) MUST be stored in S3 or equivalent object storage — NOT local disk. Local disk files are lost on Docker restart or VPS migration.

Ride Service
Endpoint	Method	Auth	Action
/ride/estimate	POST	Passenger	Fare estimate via Google Maps Distance Matrix
/ride/book	POST	Passenger	Create REQUESTED ride → trigger Matching
/ride/:id	GET	Both	Ride detail
/ride/:id/accept	PATCH	Driver	Accept — Redis atomic lock (SET NX)
/ride/:id/start	PATCH	Driver	ACCEPTED → STARTED
/ride/:id/complete	PATCH	Driver	STARTED → COMPLETED → fare finalize
/ride/:id/cancel	PATCH	Both	Cancel — only REQUESTED or ACCEPTED

Matching Service
Ride booked (REQUESTED)
     ↓
Redis GEORADIUS — 5km radius
     ↓
Filter: isOnline=true, status=APPROVED, no active ride
     ↓
Sort by straight-line distance (V1 — road distance V2)
     ↓
Nearest driver → Socket: ride:request
Redis: SET ride:matching:{rideId} {driverId} NX EX 30
     ↓
┌─────────────────────────────────┐
│  30 second window               │
│  ACCEPT → Ride ACCEPTED         │
│  REJECT/TIMEOUT → Next driver   │
│  Max 3 attempts per radius      │
│  After 3 → expand to 10km      │
│  Still none → CANCELLED         │
└─────────────────────────────────┘

🔴 Race Condition — Double Accept Fix
Two drivers can accept same ride simultaneously. Fix: SET ride:matching:{rideId} {driverId} NX EX 30 — atomic Redis lock. First driver to set wins. Second gets "ride already taken" response.

Location Service
Event / Endpoint	Type	Action
driver:location:update	WebSocket	Redis GEOADD — update driver position (TTL 30s)
/location/nearby-drivers	GET	Redis GEORADIUS — return drivers within radius
200m detection	Server-side	Server checks distance on each location update — triggers DRIVER_ARRIVING

⚠️ 200m Detection — Server-Side Only
DRIVER_ARRIVING trigger must be server-side. Client-side detection can be manipulated. On each location update, server calculates distance to passenger pickup — if ≤200m, trigger state change.

Notification Service
Event	Recipient	Channel
Ride requested	Driver	Socket + FCM
Ride accepted	Passenger	Socket + FCM
Driver arriving	Passenger	Socket + FCM
Ride started	Passenger	Socket
Ride completed	Both	Socket + FCM
Ride cancelled	Both	Socket + FCM

5.4 Fare Formula V1
Fare = Base Fare + (Distance km × Per KM Rate) + (Duration min × Per Min Rate)

Distance source: Google Maps Distance Matrix API
Duration source: Google Maps Distance Matrix API
Config source:   fare_config collection (per ride type)

Cache: Same pickup+dropoff pair → cache estimate 5 minutes (Redis)
This reduces Google Maps API calls significantly.
 
