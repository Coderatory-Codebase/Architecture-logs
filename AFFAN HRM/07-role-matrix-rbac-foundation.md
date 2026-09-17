## 7. Role Matrix (RBAC foundation)

  ------------------------------------------------------------------------------------
  Action                   Admin          HR             Manager        Employee
  ------------------------ -------------- -------------- -------------- --------------
  View own                 ✅             ✅             ✅             ✅
  profile/calendar/leave                                                

  View team's data         ✅             ✅             ✅ (own team   ❌
                                                         only)          

  View company-wide data   ✅             ✅             ❌             ❌

  Create/edit employees    ✅             ✅             ❌             ❌

  Approve leave            ✅             ✅             ✅ (own team)  ❌

  Post announcements       ✅             ✅             ❌             ❌

  Run payroll              ✅             ✅ (if         ❌             ❌
                                          granted)                      
  ------------------------------------------------------------------------------------

This matrix is what `authorize.ts` will enforce per route.
