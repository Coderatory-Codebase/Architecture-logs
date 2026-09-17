## 6. Data Model Relationships

    User (existing, login identity)
      └──1:1── Employee (HR profile: department, designation, manager, joining date)
                  ├──N:1── Department ──1:N── Designation
                  ├──1:N── CalendarEvent   (exists — already user-scoped)
                  ├──1:N── LeaveRequest    (approved leave → creates a CalendarEvent)
                  ├──1:N── AttendanceRecord
                  └──1:N── PerformanceReview

    Announcement ──N:M── Employee   (broadcast: all, or targeted by department)

`User` stays the authentication identity (email/password/JWT);
`Employee` is a separate HR profile linked to it --- this keeps auth
concerns and HR concerns cleanly separated, matching the Single
Responsibility principle already used in the codebase.
