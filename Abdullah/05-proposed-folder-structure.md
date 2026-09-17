## 5. Proposed Folder Structure

Calendar physically moves under the `hr/` namespace to reflect that it's
one HRM feature among several (this is a folder move + import-path
update, not a rewrite --- the working logic doesn't change):

    src/APIs/hr/
      departments/
        department.model.ts
        department.repository.ts
        department.service.ts
        department.controller.ts
        index.ts
        types/department.interface.ts
        validation/department.validation.ts
      employees/
        (same file pattern)
      calendar/                 ← existing module, relocated here
        calendar.model.ts
        calendar.repository.ts
        calendar.service.ts
        calendar.controller.ts
        index.ts
        types/calendar.interface.ts
        validation/calendar.validation.ts
      leaves/
        (same file pattern)
      attendance/               ← phase 3
      announcements/            ← phase 3
      documents/                ← phase 3
      performance/              ← phase 4
      payroll/                  ← phase 4

    src/middlewares/
      authorize.ts              ← NEW — role-check middleware, sits after authenticate.ts

Each module keeps the exact five-file shape already proven in
`authentication` and `calendar`: **Model → Repository → Service →
Controller → Routes**, plus `types/` and `validation/`.
