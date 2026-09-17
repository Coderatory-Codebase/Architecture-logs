## 15. Modular Monolith --- Module Boundary Rules

This system is a **modular monolith**: one deployable Node.js process,
but internally partitioned into independent modules with enforced
boundaries --- deliberately chosen over microservices at this stage
because the team is small and the operational overhead of separate
services (networking, deployment, distributed transactions) isn't
justified yet. The module boundaries are drawn so that **splitting any
module into its own service later is a lift, not a rewrite.**

Rules enforced across every `src/APIs/hr/<module>/` folder:

1.  **A module owns its own Model exclusively.** `leave.repository.ts`
    never imports `employee.model.ts` directly --- if Leave needs
    employee data, it calls `employeeService.getEmployeeById()`, not the
    Employee model.
2.  **Cross-module calls happen only through a Service's exported
    functions**, never through another module's Repository. Repositories
    are private to their own module; Services are the module's public
    interface.
3.  **No shared "god" repository or god service.** The temptation to
    make one `hrRepository.ts` that touches every collection is
    explicitly rejected --- it's exactly what turns a modular monolith
    back into a tangled one.
4.  **Route registration stays centralized** (`src/APIs/index.ts` mounts
    each module's router under its own prefix) but route *logic* stays
    inside the module --- the central index file never contains business
    logic, only wiring.
5.  **The database can be shared (MongoDB Atlas) without the code being
    shared** --- multiple modules may read/write the same physical
    database, but only ever through their *own* collection and their
    *own* Repository. This is what makes future extraction possible:
    pulling Attendance into its own service later means giving it its
    own database connection, not rewriting its internal logic.

This directly extends the Architecture Principles already stated in §5
(Separation of Concerns, Loose Coupling) --- §15 is the enforcement
mechanism, §5 was the intent.
