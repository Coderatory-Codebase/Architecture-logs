## 13. Functional Components (Building Block View)

Each module is a self-contained **building block** with an explicit
provided interface (what it exposes) and required interface (what it
depends on) --- this is what keeps the monolith modular rather than a
tangle of cross-imports.

  ---------------------------------------------------------------------------------------------------------
  Building Block            Responsibility    Provided Interface                          Required
                                                                                          Interface
                                                                                          (depends on)
  ------------------------- ----------------- ------------------------------------------- -----------------
  **Auth**                  Identity,         `POST /v1/register`, `/login`, `/logout`,   none (root
                            login/session     `GET /v1/user/me`                           module)

  **Departments**           Org units         `GET/POST/PUT/DELETE /v1/hr/departments`    none

  **Designations**          Job titles per    `GET/POST/PUT/DELETE /v1/hr/designations`   Departments
                            department                                                    (`departmentId`
                                                                                          reference)

  **Employees**             HR profile per    `GET/POST/PUT/DELETE /v1/hr/employees`      Auth (`userId`),
                            user                                                          Departments,
                                                                                          Designations

  **OrgChart**              Hierarchical read GraphQL `query orgChart`                    Employees
                            view                                                          (read-only, no
                                                                                          writes here)

  **Calendar** *(exists)*   Personal          `GET/POST/PUT/DELETE /v1/hr/calendar` +     Auth (`userId`)
                            scheduling        Socket.io                                   

  **Leave**                 Leave lifecycle   `GET/POST/PATCH /v1/hr/leaves` + Socket.io  Employees,
                                                                                          Calendar (creates
                                                                                          events on
                                                                                          approval)

  **Attendance**            Check-in/out      `POST /v1/hr/attendance/check-in`,          Employees
                            records           `/check-out`, `GET .../report` + Socket.io  
                                              (conditional)                               

  **Announcements**         Broadcast notices `GET/POST /v1/hr/announcements` + Socket.io Departments
                                                                                          (optional
                                                                                          targeting)

  **Payroll** *(future)*    Salary            `GET/POST /v1/hr/payroll`                   Employees,
                            computation                                                   Attendance

  **NotificationService**   Shared pub-sub    internal `publish(event, room)` used by     Socket.io core
  *(proposed, §"6. Naya     for real-time     Leave/Attendance/Announcements              
  professional addition")*  events                                                        
  ---------------------------------------------------------------------------------------------------------
