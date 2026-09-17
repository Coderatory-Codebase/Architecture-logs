# Document 6 — Database Schema

> Source: `Yango_Clone_Architecture.docx`

## 6.1 MongoDB Collections

### users

```js
{
  _id:          ObjectId,
  phone:        String,    // unique, indexed
  name:         String,
  photo:        String,    // S3 URL
  rating:       Number,    // default 5.0
  totalRatings: Number,    // default 0
  deviceToken:  String,    // FCM token
  createdAt:    Date,
  updatedAt:    Date
}
```

### drivers

```js
{
  _id:          ObjectId,
  phone:        String,    // unique, indexed
  name:         String,
  photo:        String,    // S3 URL
  status:       Enum[PENDING, APPROVED, SUSPENDED],  // indexed
  isOnline:     Boolean,   // indexed
  rating:       Number,    // default 5.0
  totalRatings: Number,
  vehicle: {
    make:        String,
    model:       String,
    year:        Number,
    plateNumber: String,
    color:       String,
    type:        Enum[ECONOMY, COMFORT]
  },
  documents: {
    licenseUrl:    String,  // S3 URL — NOT local disk
    vehicleDocUrl: String,  // S3 URL — NOT local disk
    verified:      Boolean
  },
  deviceToken:  String,    // FCM token
  createdAt:    Date,
  updatedAt:    Date
}
```

### rides

```js
{
  _id:           ObjectId,
  passengerId:   ObjectId,  // ref: users, indexed
  driverId:      ObjectId,  // ref: drivers, indexed
  status:        Enum[REQUESTED, ACCEPTED, DRIVER_ARRIVING,
                       STARTED, COMPLETED, CANCELLED],
  pickup: {
    address:  String,
    location: { type: "Point", coordinates: [lng, lat] }
  },
  dropoff: {
    address:  String,
    location: { type: "Point", coordinates: [lng, lat] }
  },
  fare: {
    estimated: Number,
    final:     Number,
    currency:  String   // default: PKR
  },
  distance:      Number,    // km
  duration:      Number,    // minutes
  paymentMethod: Enum[CASH],
  cancelledBy:   Enum[PASSENGER, DRIVER, SYSTEM],
  cancelReason:  String,
  createdAt:     Date,      // indexed
  updatedAt:     Date,
  startedAt:     Date,
  completedAt:   Date
}

// Indexes:
{ passengerId: 1, status: 1 }
{ driverId: 1, status: 1 }
{ pickup.location: "2dsphere" }

// Partial unique index — prevents duplicate active rides:
db.rides.createIndex(
  { passengerId: 1 },
  { unique: true,
    partialFilterExpression: {
      status: { $in: ["REQUESTED", "ACCEPTED", "STARTED"] }
    }
  }
)
```

> 🔴 **Duplicate Ride Prevention — Partial Unique Index** — Without this index, double-tap on Book button creates two REQUESTED rides. The partial unique index at database level guarantees one active ride per passenger — cannot be bypassed by app bugs or race conditions.

### ratings

```js
{
  _id:       ObjectId,
  rideId:    ObjectId,   // ref: rides, unique
  raterId:   ObjectId,
  ratedId:   ObjectId,
  raterType: Enum[PASSENGER, DRIVER],
  score:     Number,     // 1-5
  comment:   String,
  createdAt: Date
}
```

### fare_config

```js
{
  _id:        ObjectId,
  rideType:   Enum[ECONOMY, COMFORT],  // unique
  baseFare:   Number,
  perKmRate:  Number,
  perMinRate: Number,
  currency:   String,
  updatedAt:  Date
}
```

## 6.2 Redis Key Design

| Key Pattern | Value | TTL | Purpose |
|---|---|---|---|
| otp:{phone} | 6-digit OTP | 5 minutes | Phone verification |
| otp:attempts:{phone} | attempt count | 10 minutes | Brute force protection |
| otp:cooldown:{phone} | 1 | 60 seconds | Resend cooldown |
| driver:location:{driverId} | GEO point | 30 seconds | Live location (auto-expire on disconnect) |
| ride:matching:{rideId} | driverId (NX lock) | 30 seconds | Atomic accept lock |
| refresh:{userId} | refresh token | 30 days | Token validation + logout support |
| fare:cache:{pickup}:{dropoff} | fare estimate | 5 minutes | Google Maps API cost reduction |

```
// GEO Commands
GEOADD  drivers:locations {lng} {lat} {driverId}
GEORADIUS drivers:locations {lng} {lat} 5 km ASC

// Atomic Matching Lock
SET ride:matching:{rideId} {driverId} NX EX 30
// NX = only set if not exists (atomic)
// EX 30 = auto-expire in 30 seconds

// OTP Rate Limit
INCR otp:attempts:{phone}
EXPIRE otp:attempts:{phone} 600   // 10 min window
```

## 6.3 PostgreSQL — Notifications

```sql
CREATE TABLE notifications (
  id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id    VARCHAR NOT NULL,
  user_type  VARCHAR NOT NULL,   -- PASSENGER | DRIVER
  type       VARCHAR NOT NULL,
  title      VARCHAR NOT NULL,
  body       TEXT,
  data       JSONB,
  sent_via   VARCHAR[],          -- ["socket", "fcm"]
  read_at    TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_notifications_user
  ON notifications(user_id, created_at DESC);
```

## 6.4 GraphQL Decision

> ⚠️ **GraphQL Endpoint — Decision Required** — Base Server has a GraphQL endpoint for notifications. Doc 9 only uses Socket.io + FCM. **Decision: Remove GraphQL endpoint** — it adds complexity with no additional value when Socket + FCM covers all notification needs. Clean up in Base Server mapping phase.

---
◀ Previous: [05 — Services Design](05-services-design.md) | Back to [README](../README.md) | Next → [07 — Critical Flows](07-critical-flows.md)
