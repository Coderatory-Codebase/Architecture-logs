## 1. Recap --- What Already Exists

  -----------------------------------------------------------------------------------
  Layer                   Status                  Detail
  ----------------------- ----------------------- -----------------------------------
  Boilerplate             ✅ Built                `base_server` --- Express +
                                                  TypeScript + Mongoose, layered as
                                                  Router → Controller → Service →
                                                  Repository → Model

  Auth module             ✅ Built                Register, email-confirm, Login,
                                                  Logout --- JWT stored as httpOnly
                                                  cookies (`accessToken`,
                                                  `refreshToken`), password hashed
                                                  with bcrypt

  User management         ✅ Built                `/v1/user/me` --- protected route
                                                  returning the authenticated user

  Calendar module         ✅ Built                Full CRUD
                                                  (`create/list/get/update/delete`)
                                                  scoped per user, validated with
                                                  Joi, real-time sync via
                                                  **Socket.io**
                                                  (`event:created/updated/deleted`
                                                  emitted to a per-user room)

  Infra                   ✅ Working              MongoDB Atlas (cloud), tested
                                                  end-to-end via Postman, Next.js
                                                  frontend consuming the API
  -----------------------------------------------------------------------------------

This document treats the Calendar module as **one feature inside the
larger HRM system**, not a separate product --- going forward it is
referred to as **"Scheduling & Calendar."**
