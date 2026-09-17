Document 1 — Requirements


1.1 Project Overview
Field	Detail
App Name	Yango Clone
Type	Ride-Hailing Platform
Platforms	Passenger App, Driver App, Admin Panel
Architecture	Modular Monolith V1 → Microservices when needed
Auth Method	Phone + OTP via SMS (Twilio)
Payment V1	Cash Only
Payment V2	In-app Gateway (Stripe / Local)
Launch City	Single city
Currency	PKR

1.2 Functional Requirements
Passenger
▸  Register / Login — phone number + OTP via SMS
▸  Profile management (name, photo)
▸  Pickup & drop location via Google Maps
▸  Ride type selection (Economy, Comfort)
▸  Fare estimate before booking
▸  Ride booking
▸  Real-time nearby drivers on map
▸  Driver details — name, photo, rating, vehicle
▸  Real-time ride tracking
▸  Ride cancellation (policy enforced)
▸  Cash payment V1
▸  Receipt after ride completion
▸  Rate & review driver
▸  Ride history
▸  Push notifications — driver accepted, arriving, started (background + foreground)

Driver
▸  Register / Login — phone number + OTP via SMS
▸  Document upload (license, vehicle docs)
▸  Approval pending state — Admin approves
▸  Online / Offline toggle
▸  GPS location stream every 5 seconds while online
▸  Ride request receive with timeout (30 seconds)
▸  Accept / Reject ride
▸  Navigate to passenger
▸  Start & complete ride
▸  Earnings summary
▸  Rate passenger
▸  Ride history
▸  Push notifications — new request, cancellation (background + foreground)

Admin
▸  Dashboard — active rides, drivers, revenue
▸  Driver approval / rejection with notification
▸  User management
▸  Ride management & monitoring
▸  Fare configuration per ride type
▸  Reports & analytics
▸  Driver document review

1.3 Non-Functional Requirements
Parameter	Target	Notes
Peak Concurrent Rides	500 at launch	2000 at 6 months
Location Writes	~100 writes/sec at peak	500 drivers × 1 update/5s
API Response Time	< 500ms	p95 target
Real-time Latency	< 2 seconds	Location broadcast
Uptime	99.9%	Max 8.7 hrs/year downtime
Location Update Interval	Every 5 seconds	Driver app when online
Data Retention	2 years	Ride history
OTP Expiry	5 minutes	Redis TTL
JWT Access Token	15 minutes	Short-lived
JWT Refresh Token	30 days	Redis stored

⚠️ Uptime vs Single VPS
Current plan is single VPS. 99.9% SLA requires max 8.7hrs downtime/year — provider maintenance alone can exceed this. Either relax NFR to "best effort, single VPS" or plan second instance + load balancer for V1.5.

1.4 Key Decisions
Decision	Choice	Rationale
Auth Method	Phone OTP via Twilio	Pakistan reliable; base server email auth replaced
Payment V1	Cash Only	Simplest; gateway V2 — avoids PCI complexity
OTP Provider	Twilio (primary)	Reliable Pakistan delivery; monitor per-SMS cost

1.5 Out of Scope — V1
Feature	Planned For
Multiple stops	V2
Scheduled rides	V2
Dynamic surge pricing	V2
In-app chat	V2
SOS / Emergency button	V2
Referral system	V2
In-app payment gateway	V2
Email authentication	Removed — replaced by OTP
 
