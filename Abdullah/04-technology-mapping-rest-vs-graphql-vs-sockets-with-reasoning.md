## 4. Technology Mapping --- REST vs GraphQL vs Sockets (with reasoning)

**Governing rule used below:** REST is the default everywhere --- it's
already the project's proven pattern (Auth + Calendar) and the only
transport a working CRUD app genuinely requires. GraphQL or Sockets get
added to a module **only when its actual access pattern demands it** ---
never by default, never "just in case."

  -------------------------------------------------------------------------------
  Module                   REST              GraphQL           Sockets
  ------------------------ ----------------- ----------------- ------------------
  RBAC                     ---               ---               ---

  Departments &            ✅                Optional          ❌
  Designations                                                 

  Employee Profiles        ✅                Optional          Optional (narrow)

  Scheduling & Calendar    ✅ (exists)       Optional          ✅ (exists)

  Leave Management         ✅                Optional          ✅

  Attendance               ✅                Optional          Conditional

  Announcements            ✅                ❌                ✅ (strongest
                                                               case)

  Documents                ✅                ❌                ❌

  Performance Reviews      ✅                Optional          Optional (weak)

  Payroll                  ✅ only           ❌                ❌ (deliberate)

  Onboarding/Offboarding   ✅                ❌                Optional
                                                               (team-dependent)
  -------------------------------------------------------------------------------

### Reasoning per module

**RBAC** --- Not a resource at all; it's middleware that wraps every
other endpoint. There is nothing to fetch or subscribe to, so none of
the three transports apply to it directly.

**Departments & Designations** --- Low write frequency, no one is
watching this list waiting for a live change. REST CRUD is fully
sufficient. GraphQL only earns its place if an HR dashboard screen
actually needs `department → employees → manager` composed in one query
--- without that specific screen being built, adding GraphQL duplicates
maintenance (two schemas to keep in sync) for zero measured benefit.
Sockets: no case --- a stale department list for a few seconds costs
nothing.

**Employee Profiles** --- Same reasoning as Departments for
REST/GraphQL. Sockets are a narrow, optional case: notifying a manager
the instant a direct report's status changes (e.g., marked inactive).
This is low-frequency and not core to the feature --- can be deferred
indefinitely without hurting the product.

**Scheduling & Calendar** *(already built)* --- Sockets are justified
here for a **proven** reason, not a hypothetical one: the same user in
two tabs/devices needs consistent state, which we demonstrated was a
real problem during development (that's exactly why it was added).
GraphQL stays optional --- useful only if the frontend grows complex
multi-filter queries (date range + search + location combined); the
current single date-range fetch doesn't need it yet.

**Leave Management** --- This is the strongest Socket case besides
Calendar and Announcements, because it's **cross-user and
time-sensitive**: the manager is a different person from the submitter
and needs to know the moment a request lands (often with real time
pressure --- approving before someone travels). Missing this by minutes
has a real cost, unlike Departments. GraphQL stays optional for a "my
team's pending leave this month" reporting view.

**Attendance** --- Sockets are **conditional, not default**: they only
earn their place if a live "who's checked in right now" dashboard screen
is actually built (e.g., for HR/reception). Unlike Leave, where the need
is guaranteed to exist the moment leave requests exist, Attendance's
live-dashboard need depends on a screen that may never get prioritized
--- so it ships without sockets until that screen is committed to.

**Announcements** --- The single strongest Socket case in the entire
system. The whole value of an announcement is timeliness (a company-wide
or emergency notice), and it's a pure **broadcast** pattern --- reusing
the exact same Socket.io room mechanism already built for Calendar, just
broadcasting to "all" instead of a per-user room. Lower implementation
effort than any other socket candidate here, and the highest payoff.
GraphQL adds nothing --- it's a flat list, no resource composition
involved.

**Documents** --- File upload/download is inherently an HTTP/REST
concern (multipart form-data); neither GraphQL nor Sockets handle binary
transfer well, and nobody needs to watch a document appear live. Skip
both.

**Performance Reviews** --- Periodic (quarterly/annual), not urgent ---
a manager writes a review, the employee reads it later. Sockets are a
weak, optional case (a "nice touch" notification when a review is
submitted) with no real cost if missed by a few minutes --- recommend
deferring. GraphQL is optional for a "review history across cycles"
screen, same deferred-until-built logic as elsewhere.

**Payroll** --- REST only, and this is a **deliberate exclusion**, not
an oversight. Financial data benefits from being request/response and
auditable (a clear log of who requested what, when) rather than streamed
live. There is no legitimate case of "someone else needs to know a
payslip changed in real time" --- the opposite is true: minimizing how
salary data moves through the system is the actual security goal.
GraphQL's flexible querying isn't worth the added surface area on
sensitive data either.

**Onboarding/Offboarding** --- Sockets are optional and
**team-dependent**: worth adding only if multiple HR staff coordinate on
the same checklist simultaneously (live task-completion updates prevent
duplicate work --- same underlying reasoning as Calendar's multi-tab
case, but cross-user). If onboarding is handled by one person at a time,
skip sockets entirely.
