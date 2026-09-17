# Document 9 — Real-time Architecture

> Source: `Yango_Clone_Architecture.docx`

## 9.1 Technology Stack

| Component | Technology | Purpose |
|---|---|---|
| Foreground Real-time | Socket.io | Live events while app is open |
| Background Push | FCM | Notifications when app is background/closed |
| Location Store | Redis GEO | Driver positions with auto-expire |
| Socket Auth | JWT Middleware | All connections authenticated |

## 9.2 Connection Flow

```
App opens
    ↓
Socket.io connect with auth:
  socket.auth = { token: accessToken }
    ↓
Server Socket Middleware:
  verify JWT token
  attach user to socket: socket.user = { id, role }
  Valid?   → join personal room: user:{userId} or driver:{driverId}
  Invalid? → socket.disconnect() immediately
    ↓
Connection established
  Driver → emit driver:online
  Passenger → listen for ride events
```

## 9.3 Rooms Strategy

| Room | Members | Events | Lifecycle |
|---|---|---|---|
| user:{userId} | Passenger only | ride:accepted, ride:cancelled | Lifetime of socket connection |
| driver:{driverId} | Driver only | ride:request, ride:cancelled | Lifetime of socket connection |
| ride:{rideId} | Passenger + Driver | ride:location, ride:arriving, ride:started, ride:completed | ACCEPTED → COMPLETED/CANCELLED |

```js
// Server: Join ride room after acceptance
io.to(`user:${passengerId}`).emit("ride:accepted", rideData)
socket.join(`ride:${rideId}`)              // driver joins
passengerSocket.join(`ride:${rideId}`)     // passenger joins

// Room join authorization check
socket.on("ride:join", async ({ rideId }) => {
  const ride = await Ride.findById(rideId)
  const userId = socket.user.id
  const isAuthorized = ride.passengerId == userId || ride.driverId == userId
  if (!isAuthorized) return socket.emit("error", "Unauthorized")
  socket.join(`ride:${rideId}`)
})
```

## 9.4 Location Update Flow

```
Driver emits every 5 seconds:
  driver:location:update { driverId, lat, lng, heading }
       ↓
Location Handler (server):
  1. GEOADD drivers:locations {lng} {lat} {driverId}
  2. Active ride check:
     rides.findOne({ driverId, status: { $in: [ACCEPTED, DRIVER_ARRIVING, STARTED] }})
  3. If active ride:
     a. Calculate distance to pickup (if status = ACCEPTED)
        distance ≤ 200m → update status → DRIVER_ARRIVING
                        → notify passenger (Socket + FCM)
     b. Broadcast to ride room:
        io.to(`ride:${rideId}`).emit("ride:location", { lat, lng, heading })
  4. If no active ride:
     Redis update only — for matching pool
```

## 9.5 Disconnect Handling

```
Driver socket disconnects (network drop / app close):
    ↓
Socket: "disconnect" event fires
    ↓
Location Service:
  Redis TTL 30s already set — auto-expires
  Driver exits matching pool automatically
    ↓
Was driver in active ride?
  YES:
    Notify passenger: "Driver connection lost"
    Start 30s reconnect timer
    ┌─ Reconnects within 30s → Resume ride normally
    └─ Does not reconnect:
         Ride Service decides:
         - Driver CANCELLED → restart matching
         - Or admin manual resolution
  NO:
    Nothing to do — driver exits pool via TTL
```

## 9.6 Scaling Path

| Phase | Setup | Concurrent Connections |
|---|---|---|
| V1 (Now) | Single Socket.io instance, PM2 fork mode | Up to ~1500 |
| V1.5 | Redis Adapter + PM2 cluster, sticky sessions on Nginx | 2000-10000 |
| V2 | Dedicated Socket.io cluster, horizontal scaling | 10000+ |

> ⚠️ **Nginx WebSocket Timeout** — Default Nginx `proxy_read_timeout` is 60 seconds. WebSocket connections will be cut every 60s silently. Add to Nginx WS location: `proxy_read_timeout 86400;` — keeps connections alive up to 24 hours.

---
◀ Previous: [08 — API Contract](08-api-contract.md) | Back to [README](../README.md) | Next → [10 — Base Server Mapping](10-base-server-mapping.md)
