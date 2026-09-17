## 12. Business Logic --- Core Domain Rules

These are the rules each Service layer must enforce --- not CRUD
mechanics, but the actual business constraints that make the system
correct.

  -----------------------------------------------------------------------------
  Module                              Rule
  ----------------------------------- -----------------------------------------
  **Employee**                        An employee cannot be their own manager.
                                      `manager` field must reference an
                                      existing, active employee. `department`
                                      and `designation` must exist before an
                                      employee can be assigned to them.

  **Leave**                           Requested dates cannot overlap an
                                      already-approved leave for the same
                                      employee. Requested days cannot exceed
                                      the employee's remaining leave balance
                                      for that leave type. A manager cannot
                                      approve their own leave request (must
                                      escalate to their own manager or
                                      HR/Admin). Approving a leave auto-creates
                                      a corresponding `CalendarEvent` (this is
                                      the integration point between Leave and
                                      Calendar).

  **Calendar**                        *(exists)* An event's `endTime` must be
                                      after `startTime` (already enforced via
                                      Joi `.greater(joi.ref('startTime'))`).
                                      Events are always scoped to the owning
                                      user --- no cross-user read/write,
                                      enforced at the repository layer.

  **Attendance**                      One check-in per employee per calendar
                                      day. Check-out timestamp must be after
                                      check-in. A check-in without a matching
                                      check-out by end of day is flagged, not
                                      silently dropped.

  **Announcements**                   Only Admin/HR can post (enforced by
                                      `authorize.ts`, not just hidden UI). If
                                      `targetDepartment` is set, it must
                                      reference an existing department; if
                                      unset, the announcement broadcasts to
                                      all.

  **Payroll** *(when built)*          `netSalary = grossSalary − deductions`,
                                      computed server-side only --- never
                                      accepted as client input. Payroll can
                                      only be generated for employees with
                                      `status: active` for that pay period, and
                                      only after that period's attendance
                                      records are finalized.

  **RBAC**                            Permission checks are additive down a
                                      fixed hierarchy --- Admin ⊇ HR ⊇ Manager
                                      ⊇ Employee for shared actions --- but
                                      module-specific actions (e.g., "approve
                                      *my team's* leave") are scoped by
                                      relationship (`manager` field), not just
                                      role name.
  -----------------------------------------------------------------------------
