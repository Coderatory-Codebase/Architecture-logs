## 14. Non-Functional Components (Quality Requirements)

Mapped to arc42's Quality Requirements chapter --- each is a concrete,
testable target, not a vague aspiration.

  -----------------------------------------------------------------------------------------
  Category                            Requirement
  ----------------------------------- -----------------------------------------------------
  **Security**                        JWT access tokens short-lived (existing pattern),
                                      httpOnly + `sameSite: strict` cookies (existing
                                      pattern). Salary fields (`Payslip.netSalary`,
                                      `grossSalary`) encrypted at rest or field-level
                                      hashed where full reversibility isn't needed.
                                      `authorize.ts` enforced on every HR route, not just
                                      hidden in the UI. Rate limiting applied to
                                      write-heavy endpoints (leave apply, check-in) to
                                      prevent abuse.

  **Compliance (GDPR)**               Employee personal data deletable on request (Right to
                                      Erasure) without breaking referential integrity
                                      (soft-delete + anonymize, not hard-delete of
                                      historical payroll/attendance records that must
                                      legally persist). Data retention period defined per
                                      record type (e.g., payroll records retained per local
                                      labor law, not indefinitely).

  **Performance**                     List endpoints (Employees, Leaves, Announcements)
                                      paginated past a defined threshold (e.g., 50 records)
                                      --- never return an unbounded array, the way
                                      Calendar's `findEventsByUser` currently does not
                                      paginate (worth revisiting there too as data grows).
                                      Mongoose indexes on frequently-filtered fields
                                      (`employee`, `department`, `startTime` --- same
                                      pattern already used in
                                      `calendarSchema.index({ user: 1, startTime: 1 })`).

  **Scalability**                     Each module owns its own Model + Repository
                                      exclusively (see §15). This means any single module
                                      (e.g., Attendance, if it becomes write-heavy) can be
                                      extracted into its own service later without
                                      redesigning the rest --- the monolith is modular *by
                                      construction*, not just by folder naming.

  **Availability**                    A `/v1/health` endpoint (already exists as
                                      `/v1/health` per the General router) is the basis for
                                      uptime monitoring / load-balancer health checks in
                                      any future deployment.

  **Maintainability**                 Every module follows the identical five-file shape
                                      (Model → Repository → Service → Controller → Routes)
                                      already proven in Auth and Calendar --- a new
                                      engineer can predict a file's location without being
                                      told.

  **Observability**                   Structured logging via the existing `winston` logger
                                      for every controller response (already implemented
                                      pattern) extended to log role-check failures and
                                      payroll access specifically, given their sensitivity.

  **Usability (API consumer)**        Every endpoint returns the existing uniform
                                      `{ success, statusCode, message, data }` / error
                                      shape --- no module invents its own response format.
  -----------------------------------------------------------------------------------------
