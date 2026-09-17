Document 8 — API Contract


8.1 Base Configuration
Field	Value
Base URL	https://api.yango-clone.com/v1
Auth Header	Authorization: Bearer {accessToken}
Content-Type	application/json
Versioning	/v1/ prefix — breaking changes get /v2/ route

8.2 Standard Response Shapes
// Success
{
  "success": true,
  "message": "Ride booked successfully",
  "data": { ... }
}

// Error
{
  "success": false,
  "message": "OTP expired",
  "statusCode": 400
}

// Paginated
{
  "success": true,
  "data": { "items": [...], "total": 100, "page": 1, "limit": 20 }
}

8.3 Endpoints
Auth
Method	Endpoint	Auth	Who	Description
POST	/auth/send-otp	No	Anyone	Send OTP to phone
POST	/auth/verify-otp	No	Anyone	Verify OTP, get tokens
POST	/auth/refresh	No	Anyone	Refresh access token
POST	/auth/logout	Yes	Anyone	Invalidate refresh token

User (Passenger)
Method	Endpoint	Auth	Who	Description
GET	/user/profile	Yes	Passenger	Get own profile
PATCH	/user/profile	Yes	Passenger	Update name, photo
GET	/user/ride-history	Yes	Passenger	Paginated ride list
PATCH	/user/device-token	Yes	Passenger	Update FCM token

Driver
Method	Endpoint	Auth	Who	Description
GET	/driver/profile	Yes	Driver	Get own profile
PATCH	/driver/profile	Yes	Driver	Update profile
POST	/driver/documents	Yes	Driver	Upload docs to S3
PATCH	/driver/status	Yes	Driver	Toggle online/offline
GET	/driver/earnings	Yes	Driver	Earnings summary
GET	/driver/ride-history	Yes	Driver	Paginated ride list
PATCH	/driver/device-token	Yes	Driver	Update FCM token

Ride
Method	Endpoint	Auth	Who	Description
POST	/ride/estimate	Yes	Passenger	Fare estimate (cached 5min)
POST	/ride/book	Yes	Passenger	Book ride
GET	/ride/:id	Yes	Both	Ride details
PATCH	/ride/:id/accept	Yes	Driver only	Accept ride (atomic lock)
PATCH	/ride/:id/start	Yes	Driver only	Start ride
PATCH	/ride/:id/complete	Yes	Driver only	Complete ride
PATCH	/ride/:id/cancel	Yes	Both	Cancel — REQUESTED/ACCEPTED only

Location
Method	Endpoint	Auth	Who	Description
GET	/location/nearby-drivers	Yes	Passenger	Drivers within radius on map

GET /location/nearby-drivers?lat=24.8105&lng=67.0711&radius=5

Response:
{
  "success": true,
  "data": {
    "drivers": [
      {
        "driverId": "xyz",
        "location": { "lat": 24.812, "lng": 67.073 },
        "distance": 0.8,
        "vehicle": { "type": "ECONOMY", "make": "Toyota", "color": "White" }
      }
    ]
  }
}

Admin
Method	Endpoint	Auth	Who	Description
GET	/admin/drivers/pending	Yes	Admin	Drivers awaiting approval
PATCH	/admin/drivers/:id/approve	Yes	Admin	Approve driver
PATCH	/admin/drivers/:id/reject	Yes	Admin	Reject with reason
PATCH	/admin/drivers/:id/suspend	Yes	Admin	Suspend driver
GET	/admin/rides	Yes	Admin	All rides — filterable
GET	/admin/dashboard	Yes	Admin	Active rides, drivers, revenue
GET	/admin/fare-config	Yes	Admin	Current fare config
PUT	/admin/fare-config	Yes	Admin	Update fare config

Rating
Method	Endpoint	Auth	Who	Description
POST	/rating/driver	Yes	Passenger only	Rate driver after ride
POST	/rating/passenger	Yes	Driver only	Rate passenger after ride

8.4 WebSocket Events
Client → Server (Emit)
Event	Payload	Who	When
driver:location:update	{ lat, lng, heading }	Driver	Every 5s while online
driver:online	{ driverId }	Driver	Toggle online
driver:offline	{ driverId }	Driver	Toggle offline
ride:join	{ rideId }	Both	After ride accepted

Server → Client (Broadcast)
Event	Payload	To	Channel
ride:request	{ rideId, pickup, dropoff, fare, passenger }	Driver	driver:{driverId}
ride:accepted	{ driverId, name, photo, vehicle, rating }	Passenger	user:{userId}
ride:location	{ lat, lng, heading }	Passenger	ride:{rideId}
ride:arriving	{ eta }	Passenger	ride:{rideId}
ride:started	{ rideId, startedAt }	Passenger	ride:{rideId}
ride:completed	{ fare, duration, distance }	Both	ride:{rideId}
ride:cancelled	{ reason, cancelledBy }	Both	ride:{rideId}

🔴 Socket Authentication Required
All Socket.io connections must be authenticated. On connection: verify JWT from socket.handshake.auth.token. On room join (ride:{rideId}): verify user is the passenger or assigned driver of that ride. Unauthenticated sockets must be disconnected immediately.
