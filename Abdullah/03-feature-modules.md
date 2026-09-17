## 3. Feature Modules

Each module below is scoped, with a recommended phase (build order) and
a one-line purpose.

  ----------------------------------------------------------------------------------------------
  \#             Module                     Purpose                Depends on     Phase
  -------------- -------------------------- ---------------------- -------------- --------------
  1              **RBAC (Roles &            Admin / HR / Manager / Auth           1
                 Permissions)**             Employee roles,        (existing)     
                                            enforced via                          
                                            middleware                            

  2              **Departments &            Org structure ---      RBAC           1
                 Designations**             teams, job titles                     

  3              **Employee Profiles**      Employee record        Departments    1
                                            (distinct from login                  
                                            `User`) --- joining                   
                                            date, department,                     
                                            designation, manager                  

  4              **Scheduling & Calendar**  Personal + company     Employee       2
                 *(exists)*                 events                                

  5              **Leave Management**       Apply / approve /      Employee,      2
                                            reject leave, balance  Calendar       
                                            tracking                              

  6              **Attendance**             Check-in/out,          Employee       3
                                            daily/monthly reports                 

  7              **Announcements / Notice   Company-wide or        Employee, RBAC 3
                 Board**                    team-wide broadcasts                  

  8              **Documents**              Employee               Employee       3
                                            documents/contracts,                  
                                            file upload                           

  9              **Performance Reviews**    Periodic review        Employee, RBAC 4
                                            cycles, manager                       
                                            feedback                              

  10             **Payroll**                Salary computation,    Employee,      4
                                            payslips (sensitive    Attendance     
                                            --- encrypt at rest)                  

  11             **Onboarding/Offboarding   Checklist-driven task  Employee, RBAC 4
                 Workflows**                flow for new/exiting                  
                                            employees                             
  ----------------------------------------------------------------------------------------------

**Phase 1** is the foundation everything else depends on. **Phase 2**
folds Calendar in and adds Leave (which naturally produces calendar
entries). Phases 3--4 are additive and can ship independently.
