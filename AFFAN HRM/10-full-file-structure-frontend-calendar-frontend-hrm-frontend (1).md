## 10. Full File Structure --- Frontend (`calendar-frontend` → HRM frontend)

Existing convention (`src/lib/`, `src/hooks/`,
`src/components/<domain>/`, flat `src/__tests__/`) is extended per
domain, not merged into shared generic files.

    src/
      app/
        register/page.tsx                     (existing)
        login/page.tsx                        (existing)
        calendar/page.tsx                     (existing — stays at this route; HRM nav will link to it)
        hr/
          departments/
            page.tsx                          (list view)
            [id]/page.tsx                     (detail/edit view)
          employees/
            page.tsx
            [id]/page.tsx
          leaves/
            page.tsx
          attendance/                          (phase 3)
            page.tsx
          announcements/                       (phase 3)
            page.tsx
          documents/                           (phase 3)
            page.tsx
          performance/                         (phase 4)
            page.tsx
          payroll/                             (phase 4)
            page.tsx

      lib/
        apiClient.ts                          (shared fetch wrapper — renamed from generic api.ts)
        authApi.ts                            (register/login/logout — split out of the old monolithic api.ts)
        calendarApi.ts
        calendarUtils.ts                      (existing)
        departmentsApi.ts
        employeesApi.ts
        leavesApi.ts
        attendanceApi.ts                       (phase 3)
        announcementsApi.ts                    (phase 3)
        documentsApi.ts                        (phase 3)
        performanceApi.ts                      (phase 4)
        payrollApi.ts                          (phase 4)
        socket.ts                             (existing)

      hooks/
        useEvents.ts                          (existing)
        useDepartments.ts
        useEmployees.ts
        useLeaves.ts
        useAttendance.ts                       (phase 3)
        useAnnouncements.ts                    (phase 3)

      components/
        auth/
          RegisterForm.tsx
          LoginForm.tsx
        calendar/
          EventForm.tsx
          EventCard.tsx
          DayGroup.tsx
        departments/
          DepartmentForm.tsx
          DepartmentCard.tsx
          DepartmentList.tsx
        employees/
          EmployeeForm.tsx
          EmployeeCard.tsx
          EmployeeList.tsx
          EmployeeProfileHeader.tsx
        leaves/
          LeaveRequestForm.tsx
          LeaveRequestCard.tsx
          LeaveApprovalPanel.tsx               (manager-only view)
        attendance/                            (phase 3)
          CheckInButton.tsx
          AttendanceTable.tsx
        announcements/                         (phase 3)
          AnnouncementForm.tsx
          AnnouncementBanner.tsx

      __tests__/
        calendarUtils.test.ts                 (existing)
        EventCard.test.tsx                    (existing)
        DepartmentForm.test.tsx
        EmployeeForm.test.tsx
        EmployeeCard.test.tsx
        LeaveRequestForm.test.tsx
        LeaveApprovalPanel.test.tsx

**Naming rule applied here too:** the old shared `api.ts` is split into
one file per domain (`authApi.ts`, `calendarApi.ts`,
`departmentsApi.ts`, ...) --- so backend and frontend never share an
identically-named, differently-scoped file, and each file's purpose is
readable from its name alone.

## Updated Frontend Structure

```
src/
├── __tests__/                          # Centralized tests folder (HR & Auth only)
│   ├── LoginPage.test.tsx              # Authentication test (HR level)
│   ├── RegisterPage.test.tsx           # Authentication test (HR level)
│   ├── Calendar.test.tsx               # <-- Calendar module test file added here
│   ├── OrgChart.test.tsx
│   ├── Employees.test.tsx
│   ├── Leaves.test.tsx
│   ├── Attendance.test.tsx
│   └── Announcements.test.tsx
│
├── app/
│   ├── page.tsx
│   ├── layout.tsx
│   ├── login/                          # HR System Authentication
│   │   └── page.tsx
│   ├── register/                       # HR System Authentication
│   │   └── page.tsx
│   └── hr/                             # Main HR Management Dashboard
│       ├── calendar/
│       │   └── page.tsx                # Calendar feature nested inside HR
│       ├── orgchart/
│       │   └── page.tsx
│       ├── employees/
│       │   └── page.tsx
│       ├── leaves/
│       │   └── page.tsx
│       ├── attendance/
│       │   └── page.tsx
│       └── announcements/
│           └── page.tsx
│
├── components/
│   ├── auth/                           # Global HR System Auth Forms
│   │   ├── LoginForm.tsx
│   │   └── RegisterForm.tsx
│   └── hr/                             # HR module components
│       ├── calendar/
│       │   ├── DayGroup.tsx
│       │   ├── EventCard.tsx
│       │   └── EventForm.tsx
│       ├── orgChart/
│       │   └── HierarchyTree.tsx
│       ├── employees/
│       │   └── EmployeeTable.tsx
│       ├── leaves/
│       │   └── LeaveRequestForm.tsx
│       ├── attendance/
│       │   └── LivePunchWidget.tsx
│       └── announcements/
│           └── AnnouncementTicker.tsx
│
├── hooks/
│   ├── useEvents.ts
│   ├── useOrgChart.ts
│   ├── useEmployees.ts
│   ├── useLeaves.ts
│   ├── useAttendanceSocket.ts
│   └── useAnnouncementsSocket.ts
│
└── lib/
    ├── api.ts
    ├── socket.ts
    ├── calendarApi.ts
    ├── orgChartApi.ts
    ├── employeesApi.ts
    ├── leavesApi.ts
    ├── attendanceApi.ts
    └── announcementsApi.ts
```
