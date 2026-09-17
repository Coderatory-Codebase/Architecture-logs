Document 7 — Critical Flows


Flow 1: Phone OTP Login
Client → POST /auth/send-otp { phone }
     ↓
Auth Service
  Check otp:cooldown:{phone} → if exists → 429 "Wait 60s"
  Check otp:attempts:{phone} → if > 3 per 10min → 429 "Too many requests"
  Generate 6-digit OTP
  Redis SET otp:{phone} = OTP   TTL: 5 min
  Redis SET otp:cooldown:{phone} TTL: 60s
  Twilio → send SMS
     ↓
Client → POST /auth/verify-otp { phone, otp }
     ↓
Auth Service
  INCR otp:attempts:{phone}   // track verify attempts
  If attempts > 5 → DELETE otp:{phone} → 429 "OTP invalidated"
  Redis GET otp:{phone}
  Match? NO  → 400 "Invalid OTP"
  Match? YES → DELETE otp:{phone}
               User exists? → Issue JWT
               New user?    → Create user → Issue JWT
     ↓
Response: { accessToken (15min), refreshToken (30days) }

Flow 2: Ride Booking + Driver Matching
Passenger → POST /ride/book { pickup, dropoff, rideType }
     ↓
Ride Service
  Check partial unique index → passenger already has active ride?
  → YES: 409 "You already have an active ride"
  → NO:  Create ride (status: REQUESTED)
         Calculate fare estimate
         Trigger Matching Service
     ↓
Matching Service
  Attempt 1-3 (5km radius):
    GEORADIUS drivers:locations {pickup} 5 km ASC
    Filter: isOnline=true, status=APPROVED, no active ride
    Sort by distance (straight-line V1)
    → No drivers in 5km? Expand to 10km, repeat
    → Still none? → Ride CANCELLED (SYSTEM)

    Selected driver:
    Socket emit: ride:request → driver room
    SET ride:matching:{rideId} {driverId} NX EX 30

  Driver response (30s window):
  ┌─ ACCEPT: PATCH /ride/:id/accept
  │   SET ride:matching:{rideId} {driverId} NX EX 30
  │   → Returns OK:  ride ACCEPTED, passenger notified
  │   → Returns FAIL: another driver already accepted → 409
  │
  ├─ REJECT: Matching tries next driver (attempt counter++)
  │
  └─ TIMEOUT (30s): Redis key expires → next driver
                    After 3 attempts → CANCELLED

Flow 3: Real-time Location Tracking
Driver App (while online):
  Every 5 seconds:
  Socket emit: driver:location:update { driverId, lat, lng, heading }
     ↓
Location Service (server)
  GEOADD drivers:locations {lng} {lat} {driverId}   // TTL 30s auto-reset
  Calculate distance to passenger pickup (if active ride)
  Distance ≤ 200m? → trigger DRIVER_ARRIVING
     ↓
Active ride?
  YES → broadcast to room ride:{rideId}
        Socket: ride:location { lat, lng, heading }
  NO  → update Redis only (for matching pool)
     ↓
Passenger App
  Receives ride:location event
  Updates driver marker on map

Flow 4: Full Ride Lifecycle
ACCEPTED
  Driver navigates to passenger
  Location updates every 5s → broadcast to ride room
     ↓
DRIVER_ARRIVING  (server-side, triggered at ≤200m)
  Socket + FCM → Passenger: "Driver is arriving"
     ↓
Driver arrives, passenger boards
  Driver → PATCH /ride/:id/start
     ↓
STARTED
  Location tracking continues
  Timer starts (for duration calculation)
     ↓
Driver → PATCH /ride/:id/complete
     ↓
COMPLETED
  Fare Service calculates final fare
  Both notified (Socket + FCM)
  Rating prompt shown in both apps

Flow 5: Cancellation
Passenger or Driver → PATCH /ride/:id/cancel { reason }
     ↓
Ride Service
  Status check:
  REQUESTED or ACCEPTED → allow cancel
  STARTED               → 400 "Cannot cancel ongoing ride"
  COMPLETED/CANCELLED   → 400 "Ride already ended"
     ↓
  Save: cancelledBy, cancelReason
  Status → CANCELLED
     ↓
  cancelledBy = DRIVER?
    → Notify passenger
    → Restart matching (find new driver)
  cancelledBy = PASSENGER?
    → Notify driver
    → End matching loop
     ↓
  Both notified via Socket + FCM

Flow 6: Rating
Ride → COMPLETED
  Both apps show rating prompt
     ↓
Passenger → POST /rating/driver { rideId, score, comment }
Driver    → POST /rating/passenger { rideId, score, comment }
     ↓
Rating Service
  Save rating document
  Recalculate average:
    newRating = ((oldRating × totalRatings) + score) / (totalRatings + 1)
  Update driver/user: { rating, totalRatings }
 
