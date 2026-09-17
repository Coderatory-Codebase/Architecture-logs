## 9. Full File Structure --- Backend (`base_server`)

Existing top-level test convention (`src/__tests__/`, flat, file named
after what it tests --- as already set up in this project) is preserved.
Every service gets a matching test file.

    src/
      constant/
        users.ts                              (existing — extend EUserRoles: ADMIN, HR, MANAGER, EMPLOYEE)

      middlewares/
        authenticate.ts                       (existing)
        authorize.ts                          (NEW — role-check middleware)

      APIs/
        user/                                  (existing — untouched)
          authentication/ ...
          management/ ...

        hr/
          departments/
            department.model.ts
            department.repository.ts
            department.service.ts
            department.controller.ts
            index.ts
            types/department.interface.ts
            validation/department.validation.ts

          designations/
            designation.model.ts
            designation.repository.ts
            designation.service.ts
            designation.controller.ts
            index.ts
            types/designation.interface.ts
            validation/designation.validation.ts

          employees/
            employee.model.ts
            employee.repository.ts
            employee.service.ts
            employee.controller.ts
            index.ts
            types/employee.interface.ts
            validation/employee.validation.ts

          calendar/                            (relocated from src/APIs/calendar/)
            calendar.model.ts
            calendar.repository.ts
            calendar.service.ts
            calendar.controller.ts
            index.ts
            types/calendar.interface.ts
            validation/calendar.validation.ts

          leaves/
            leave.model.ts
            leave.repository.ts
            leave.service.ts
            leave.controller.ts
            index.ts
            types/leave.interface.ts
            validation/leave.validation.ts

          attendance/                          (phase 3)
            attendance.model.ts
            attendance.repository.ts
            attendance.service.ts
            attendance.controller.ts
            index.ts
            types/attendance.interface.ts
            validation/attendance.validation.ts

          announcements/                       (phase 3)
            announcement.model.ts
            announcement.repository.ts
            announcement.service.ts
            announcement.controller.ts
            index.ts
            types/announcement.interface.ts
            validation/announcement.validation.ts

          documents/                           (phase 3)
            document.model.ts
            document.repository.ts
            document.service.ts
            document.controller.ts
            index.ts
            types/document.interface.ts
            validation/document.validation.ts

          performance/                         (phase 4)
            review.model.ts
            review.repository.ts
            review.service.ts
            review.controller.ts
            index.ts
            types/review.interface.ts
            validation/review.validation.ts

          payroll/                             (phase 4)
            payslip.model.ts
            payslip.repository.ts
            payslip.service.ts
            payslip.controller.ts
            index.ts
            types/payslip.interface.ts
            validation/payslip.validation.ts

      services/
        database.ts                           (existing)
        email.ts                              (existing)
        socket.ts                             (existing — extend with announcement:posted, leave:statusChanged events)

      __tests__/
        middlewares/
          authorize.test.ts
        hr/
          department.service.test.ts
          department.validation.test.ts
          designation.service.test.ts
          employee.service.test.ts
          employee.validation.test.ts
          calendar.service.test.ts             (relocated with the module)
          leave.service.test.ts
          leave.validation.test.ts
          attendance.service.test.ts
          announcement.service.test.ts
          document.service.test.ts
          review.service.test.ts
          payslip.service.test.ts

**Naming rule applied:** every model/service/controller/repository is
named after its own domain noun (`department.service.ts`,
`leave.service.ts`) --- never a generic `service.ts` or `controller.ts`
that could belong to any module.

## Updated Backend Structure

```
base_server/
├── src/
│   ├── __tests__/
│   │   ├── auth.test.ts
│   │   ├── calendar.test.ts
│   │   ├── orgChart.test.ts
│   │   ├── employees.test.ts
│   │   ├── leaves.test.ts
│   │   ├── attendance.test.ts
│   │   └── announcements.test.ts
│   │
│   ├── bin/
│   │   └── server.ts
│   │
│   ├── bootstrap/
│   │   └── index.ts
│   │
│   ├── config/
│   │   ├── config.ts
│   │   └── rate-limiter.ts
│   │
│   ├── constant/
│   │   ├── application.ts
│   │   ├── responseMessage.ts
│   │   └── users.ts
│   │
│   ├── APIs/                           # Modular Feature APIs
│   │   ├── auth/                       # [REST APIs] Login, Register, Logout
│   │   │   ├── auth.controller.ts
│   │   │   ├── auth.service.ts
│   │   │   ├── auth.repository.ts
│   │   │   ├── auth.model.ts
│   │   │   └── auth.route.ts
│   │   │
│   │   └── hr/                         # Main HR Management Modules Container
│   │       ├── calendar/               # [GraphQL & REST] Events & Reminders
│   │       │   ├── calendar.controller.ts
│   │       │   ├── calendar.service.ts
│   │       │   ├── calendar.repository.ts
│   │       │   ├── calendar.model.ts
│   │       │   └── calendar.route.ts
│   │       │
│   │       ├── orgchart/               # [GraphQL] Deep Nested Hierarchy Tree Query
│   │       │   ├── orgChart.controller.ts
│   │       │   ├── orgChart.service.ts
│   │       │   ├── orgChart.repository.ts
│   │       │   └── orgChart.route.ts
│   │       │
│   │       ├── employees/              # [REST APIs] CRUD for Employee Management
│   │       │   ├── employees.controller.ts
│   │       │   ├── employees.service.ts
│   │       │   ├── employees.repository.ts
│   │       │   └── employees.route.ts
│   │       │
│   │       ├── leaves/                 # [REST APIs] Leave Applications & Status Updates
│   │       │   ├── leaves.controller.ts
│   │       │   ├── leaves.service.ts
│   │       │   ├── leaves.repository.ts
│   │       │   └── leaves.route.ts
│   │       │
│   │       ├── attendance/             # [REST APIs + WebSockets] Live Punch-in/out
│   │       │   ├── attendance.controller.ts
│   │       │   ├── attendance.service.ts
│   │       │   ├── attendance.repository.ts
│   │       │   └── attendance.route.ts
│   │       │
│   │       └── announcements/          # [REST APIs + WebSockets] Live Announcement Ticker
│   │           ├── announcements.controller.ts
│   │           ├── announcements.service.ts
│   │           ├── announcements.repository.ts
│   │           └── announcements.route.ts
│   │
│   ├── handlers/
│   │   ├── errorHandler/
│   │   │   ├── errorObject.ts
│   │   │   └── httpError.ts
│   │   ├── async.ts
│   │   ├── httpResponse.ts
│   │   ├── logger.ts
│   │   └── notFound.ts
│   │
│   ├── middlewares/
│   │   ├── authenticate.ts             # JWT Token verification for HR routes
│   │   ├── authorize.ts                # Role-based access control
│   │   └── errorHandler.ts
│   │
│   ├── routes/
│   │   └── v1/
│   │       └── index.ts
│   │
│   ├── services/
│   │   ├── database.ts                 # MongoDB Connection
│   │   ├── email.ts
│   │   ├── neonDatabase.ts             # PostgreSQL (Neon) Connection for GraphQL
│   │   └── socket.ts                   # WebSockets Real-time Setup
│   │
│   ├── types/
│   ├── utils/
│   ├── work-arounds/
│   ├── app.ts
│   └── router.ts                       # Central API Router mapping all routes
│
├── .dockerignore
├── .env
├── .env.example
├── .gitignore
├── .prettierrc
├── commitlint.config.js
├── eslint.config.mjs
├── jest.config.js
├── nodemon.json
├── package-lock.json
├── package.json
├── README.md
└── tsconfig.json
```
