<!-- Converted from the original DOCX. Diagram files are stored in the accompanying .assets folder. -->

**BaseBlog**

# Architecture Documentation

Backend and Frontend Design Review

Purpose: explain the project through building blocks, architectural lenses, and core principles.

Prepared for the BaseBlog project

## Document Purpose and Scope

This document describes the current BaseBlog application as two cooperating systems: the Express and TypeScript backend named base_server-master and the Next.js frontend named base_server-frontend. It explains what each part owns, how requests and data move through the system, how security is enforced, and how the project can grow without losing clarity.

The main conclusion is that BaseBlog has a suitable modular architecture for a basic to intermediate CRUD application. REST, MongoDB, cookie-based authentication, and a separate Next.js frontend are appropriate for the current requirements. Technologies such as GraphQL, Socket.IO, and PostgreSQL should be introduced only when a feature creates a clear need.

### Current System at a Glance

| Area | Current implementation | Responsibility |
| --- | --- | --- |
| Frontend | Next.js 16, React 19, TypeScript, Tailwind CSS | Pages, forms, authentication state, user experience |
| Backend | Express, TypeScript, REST API under /v1 | Business rules, validation, authorization, HTTP responses |
| Database | MongoDB through Mongoose | Users, refresh tokens, blogs, view counts |
| Authentication | JWT access and refresh cookies; Bearer fallback | Identify users and protect write operations |
| Client data | Axios and TanStack React Query | HTTP calls, caching, mutations, loading states |
| Quality | Jest, ESLint, TypeScript validation | Unit tests, style checks, safer code changes |

### System Context

```text
Browser  <->  Next.js Frontend  <->  /v1 proxy  <->  Express REST API  <->  MongoDB Atlas
```

The frontend routes browser requests beginning with /v1 to the backend through a Next.js rewrite. Axios sends cookies with requests. The backend authenticates when necessary, executes service and repository logic, and returns a consistent response wrapper containing success, statusCode, message, request, and data.

## Core Architectural Principles

| Principle | Meaning in BaseBlog | Project evidence |
| --- | --- | --- |
| Separation of concerns | Each layer has one primary job. | Backend separates controller, service, repository, model, validation, and middleware. Frontend separates pages, features, components, providers, and API client. |
| Single source of truth | The server decides identity and ownership. | Blog creation derives the author from authenticatedUser rather than accepting an author ID from the browser. |
| Least privilege | Only users with the required permission may modify resources. | Create, update, and delete routes require authentication; update and delete verify author ownership. |
| Explicit contracts | Frontend and backend share stable request and response shapes. | Endpoint constants and typed API functions on the frontend; standard response envelope on the backend. |
| Defence in depth | The user interface is helpful but never the final protection. | Frontend hides Edit and Delete for non-owners; backend still rejects unauthorized PATCH and DELETE requests. |
| Progressive complexity | Technology should match real needs. | REST and MongoDB fit current blog CRUD better than adding GraphQL, realtime infrastructure, or another database now. |

## Building Blocks

The building blocks below show the system through stable concerns. Each concern should remain understandable as new features are added.

### Domain and Data

The core domain is publishing and reading articles. The main entities are User, Token, and Blog. A user registers, confirms an account, logs in, and creates blogs. A blog belongs to one author and contains title, content, views, and timestamps. Tokens support stored refresh-token lifecycle and logout.

| Entity | Key fields | Rules |
| --- | --- | --- |
| User | name, email, phoneNumber, role, consent, confirmation | Created during registration and resolved from the token during protected requests. |
| Token | token | Stores refresh tokens so logout can invalidate stored tokens. |
| Blog | title, content, views, author, createdAt, updatedAt | Author is a User reference. Only the author can update or delete it. |

MongoDB is accessed through Mongoose schemas. The blog model enforces title length, minimum content length, a non-negative view count, and a required author. Joi validates API input before persistence and Mongoose provides schema validation at the data layer.

### Boundaries

| Boundary | Owns | Does not own |
| --- | --- | --- |
| Frontend pages | Route layout and user experience | MongoDB access or authorization decisions |
| Frontend features | API calls, hooks, schemas, types | Page layout and database logic |
| Controllers | HTTP parsing, validation, response shape | Mongoose implementation details |
| Services | Business rules and owner checks | Visual presentation |
| Repositories | Database reads, writes, sorting, population | Permission decisions |
| Infrastructure | Configuration, database connection, logging, Docker | Blog-specific use cases |

### Communication

Communication is request-response REST over HTTP. The frontend Axios client uses credentials and reads the data field from the backend response envelope. React Query manages query caching and invalidates affected data after create, update, delete, login, and logout operations.

| Interaction | Route or mechanism | Notes |
| --- | --- | --- |
| Register | POST /v1/register | Creates user and starts account confirmation. |
| Confirm account | PATCH /v1/registeration/confirm/:token?code=... | Confirms registration code. |
| Login and logout | POST /v1/login; PUT /v1/logout | Backend sets and clears HTTP-only cookies. |
| Current user | GET /v1/user/me | Restores frontend auth state after refresh. |
| Blogs | GET and POST /v1/blogs | GET uses optional authentication; POST requires authentication. |
| Single blog | GET, PATCH, DELETE /v1/blogs/:id | Writes require authentication and author authorization. |

### Security

- Access and refresh tokens are HTTP-only cookies. The frontend does not store them in localStorage.

- authenticate verifies JWT identity, loads the user, and attaches authenticatedUser to the request.

- optionalAuthenticate keeps public GET routes available even without a valid login token.

- The backend enforces author ownership for update and delete. Frontend button visibility is not treated as security.

- Helmet, CORS, cookie parsing, rate limiting, Joi validation, password hashing, and centralized errors provide supporting protections.

Current design note: a logged-in user is currently restricted to opening their own blog details, while a logged-out visitor can view public blogs. This is appropriate for a private author dashboard but is not the normal rule for a public community blog. Decide on a consistent public-reading policy before adding comments.

### Modularity

Backend modules are feature oriented. The blog feature owns routes, controller, service, validation, repository, model, and types. Authentication and user management follow the same pattern. The frontend groups auth and blog logic in feature folders and keeps reusable presentation in components.

```text
Backend: APIs/blog -> controller + service + repository + model + validation + types
```

```text
Frontend: features/blog -> api + hooks + schema + types; components/blog -> reusable presentation
```

### Quality Attributes

| Attribute | Current support | Next improvement when needed |
| --- | --- | --- |
| Maintainability | Layered backend, feature folders, TypeScript, schemas | Give each future feature its own module and tests. |
| Security | JWT cookies, owner checks, validation, centralized errors | Review production CORS, secrets, and security headers before deployment. |
| Reliability | Error middleware, file and console logging, unit tests | Add integration tests and production monitoring. |
| Performance | React Query caching, MongoDB sorting, Next.js | Add pagination, indexes, field selection, and image handling as content grows. |
| Usability | Responsive pages, loading, empty and error states | Perform accessibility review and consistent user feedback pass. |
| Scalability | Separate frontend, backend, and repository layer | Add background work or media storage only after a real requirement appears. |

## Architecture Through the Lenses

### Engineering Lens

The engineering lens asks whether change is predictable and safe. BaseBlog uses TypeScript to reduce mismatch errors, Joi for backend request validation, Zod for frontend form validation, services for business rules, and repositories for data access. A developer can identify where a change belongs: validation rules in schemas, ownership rules in services, and query changes in repositories.

- Keep the backend response wrapper consistent so frontend API functions can reliably return response.data.data.

- Keep author identity server derived. The browser should submit title and content, never an author ID.

- If comments are added later, place them in a separate comment module rather than putting them inside the blog controller.

### Operations Lens

The operations lens asks whether the application can be started, observed, and recovered. The backend bootstraps by connecting to MongoDB and initializing its rate limiter before reporting successful startup. Winston writes operational logs to files and the console. The earlier MongoDB logging issue shows why logs must be size controlled and should not fill the main application database.

- Monitor MongoDB Atlas storage, connection health, API errors, and process restarts.

- Keep logs out of a constrained application database unless retention and size are managed.

- Use the existing health route for availability checks and add external monitoring at deployment time.

- Keep development and production values separate through dotenv-flow environment configuration.

### Developer Experience Lens

The developer experience lens asks whether the team can work safely and quickly. The projects provide scripts for development, linting, testing, and building. The backend has Jest service tests. The frontend has Jest tests for schemas and blog API behavior. The Next.js rewrite prevents local CORS friction while preserving cookie-based authentication.

| Activity | Backend | Frontend |
| --- | --- | --- |
| Start development | npm run start:dev | npm run dev |
| Run tests | npm run test | npm test |
| Lint code | npm run lint | npm run lint |
| Build output | npm run build | npm run build |

### Infrastructure Lens

The infrastructure lens asks how the system runs outside local code. The backend includes separate development and production Dockerfiles. MongoDB Atlas is the managed database dependency. The Next.js frontend can be deployed separately from the Express backend. This one frontend, one backend, one database shape is appropriate for the current project size.

| Component | Infrastructure role | Configuration concern |
| --- | --- | --- |
| Next.js frontend | Serves pages and proxies /v1 calls locally | Set API_BACKEND_URL and NEXT_PUBLIC_API_URL per environment. |
| Express backend | Runs REST endpoints and business rules | Set port, server URL, database URL, and JWT secrets. |
| MongoDB Atlas | Stores application records | Use least-privilege credentials, access rules, backups, and storage alerts. |
| Docker | Repeatable backend image build | Use production image with built output and production dependencies. |

## Frontend Architecture

The frontend uses the Next.js App Router. Pages represent routes. Components provide reusable UI. Feature folders group domain-specific API calls, hooks, types, and validation schemas. Providers establish application-wide query caching and authentication state.

| Area | Purpose | Examples |
| --- | --- | --- |
| app | Route pages and root layout | Home, login, register, blogs, create, detail, edit, confirmation |
| components | Reusable presentation and forms | Navbar, auth card, field, blog card, blog form, blog list |
| features/auth | Authentication contracts and calls | login, register, logout, getCurrentUser, schemas |
| features/blog | Blog API calls and React Query hooks | getBlogs, createBlog, updateBlog, deleteBlog |
| providers | Application-wide client setup | QueryProvider and AuthProvider |
| lib/api | Axios, endpoint map, error helpers | Credentials configuration and standardized API paths |
| tests | Frontend unit tests | Schema validation and blog API request behavior |

AuthProvider restores the current user from GET /v1/user/me after page load. React Query invalidates blog list and detail queries after mutations, so the rendered UI stays aligned with backend state without each page manually coordinating stale data.

## Backend Architecture

The backend is a layered Express application. Requests enter through shared middleware and feature routes. Controllers validate input and produce responses. Services apply rules such as view handling and ownership. Repositories run Mongoose operations. Models define persistence shape. Shared utilities handle JWTs, hashing, errors, logging, and configuration.

| Layer | Responsibility | Example |
| --- | --- | --- |
| Application setup | Configures Express and cross-cutting middleware | Helmet, CORS, cookie parser, JSON parser, static files |
| Routes | Maps URL and HTTP method to controllers | /blogs and /blogs/:id |
| Controllers | Validates HTTP input and sends standard response | blog create, get, update, delete |
| Services | Applies use-case and authorization decisions | Only author may update or delete a blog |
| Repositories | Runs Mongoose persistence operations | findBlogById, increaseViews, updateBlog |
| Models | Defines document schema | Blog title, content, views, author reference |
| Middleware and utilities | Provides shared security and support | JWT, hashing, Joi, errors, rate limiting, logging |

## Recommended Evolution Path

The project should add product value before adding technologies. The current architecture is sufficient for a basic public blog with authenticated author actions. Add small, testable features that preserve the existing boundaries.

1. Decide and document whether blogs remain public for all readers after login or become private dashboard content. Keep routes and UI consistent with the chosen rule.

1. Add pagination and search when the blog list grows.

1. Add tags, draft versus published state, and profile pages if richer publishing workflow is needed.

1. Add comments only after deciding reader visibility, account requirements, and moderation rules. Keep comments in a separate backend module.

1. Add notifications, GraphQL, PostgreSQL, or Socket.IO only when a feature needs them. Realtime notification delivery is a reason for Socket.IO; flexible dashboard data can be a reason for GraphQL.

1. Add integration and end-to-end tests after the core flows are stable: register, confirm, login, create, update, delete, logout, and public viewing.

### Technology Decision Guide

| Question | Recommended answer for current BaseBlog |
| --- | --- |
| Should REST remain? | Yes. It is simple and appropriate for the current CRUD workload. |
| Should GraphQL be added now? | No. Add it only when multiple clients or flexible dashboard queries create a clear need. |
| Should Socket.IO be added now? | No. Add it later for truly realtime behavior such as live notifications. |
| Should PostgreSQL replace MongoDB? | No. MongoDB supports the current domain. Introduce relational storage only for a justified future requirement. |
| What matters most now? | Correct authorization, consistent public and private rules, tests, deployment readiness, and polished UX. |

## Conclusion

BaseBlog is a well-structured basic-to-intermediate application. Its strongest architectural qualities are layered backend design, server-enforced authorization, feature-based frontend organization, typed validation, and test support. The best next decision is not to add every technology. It is to preserve these boundaries while implementing only the features the product genuinely needs.

## System Flow Charts and File Hierarchy

This section is a practical map of the project. It shows both the runtime path of a request and the parent-child folder structure. Generated folders such as node_modules and .next are intentionally excluded because they are dependency or build output rather than authored application structure.

![Diagram](<Blog Architecture Documentation Complete User Flows.assets/image1.png>)

*Figure 1. A blog request enters through the browser, crosses frontend and backend boundaries, reaches MongoDB through the repository, and returns through the same layers.*

![Diagram](<Blog Architecture Documentation Complete User Flows.assets/image2.png>)

*Figure 2. Login creates cookies. Later protected requests are authenticated **before service-level owner checks allow a write operation.*

## Project Parent Child Flow

The root folder contains two independent applications. base_server-master is the backend parent. base_server-frontend is the frontend parent. They communicate through the REST API and do not import each other’s source files.

### Root Structure

```text
blog/
|-- base_server-master/                         Backend application
|-- base_server-frontend/                       Frontend application
|-- Blog Architecture Documentation.docx        Architecture report
`-- package files and local workspace support
```

### Frontend Parent Child Structure

```text
base_server-frontend/
|-- app/                                        Route parent: Next.js App Router
|   |-- layout.tsx                              Root layout -> QueryProvider -> AuthProvider -> Navbar -> pages
|   |-- globals.css                             Global visual tokens, fonts, shared input styles
|   |-- page.tsx                                Home page
|   |-- login/page.tsx                          Login route -> useAuth.login -> features/auth/api
|   |-- register/page.tsx                       Register route -> features/auth/api
|   |-- registration/confirm/[token]/page.tsx   Confirmation route -> confirmRegistration API
|   `-- blogs/
|       |-- page.tsx                            Blog list route -> BlogList -> useBlogs
|       |-- create/page.tsx                     Create route -> BlogForm -> useCreateBlog
|       `-- [id]/
|           |-- page.tsx                        Detail route -> useBlog -> article view
|           `-- edit/page.tsx                   Edit route -> BlogForm -> useUpdateBlog
|-- components/                                 Reusable UI parent
|   |-- auth/auth-card.tsx                      Shared auth page shell
|   |-- auth/field.tsx                          Shared labelled form field
|   |-- blog/blog-card.tsx                      Card -> View, Edit, Delete actions
|   |-- blog/blog-form.tsx                      Shared Create and Edit form
|   |-- blog/blog-list.tsx                      List state -> cards, loading, error, empty
|   `-- layout/navbar.tsx                       Auth-aware navigation and logout
|-- features/                                   Domain logic parent
|   |-- auth/
|   |   |-- api.ts                              login, register, logout, me, confirmation calls
|   |   |-- schema.ts                           Zod login and register validation
|   |   `-- types.ts                            AuthUser, payload and response contracts
|   `-- blog/
|       |-- api.ts                              GET, POST, PATCH, DELETE blog calls
|       |-- hook.ts                             React Query keys, queries, mutations
|       |-- schema.ts                           Zod blog title and content validation
|       `-- types.ts                            Blog, author, create, update contracts
|-- lib/api/                                    Shared API infrastructure parent
|   |-- client.ts                               Axios client with credentials enabled
|   |-- endpoints.ts                            Central REST endpoint map
|   `-- error.ts                                Axios error-message helpers
|-- providers/
|   |-- query-provider.tsx                      React Query client boundary
|   `-- auth-provider.tsx                       Current-user state and login/logout boundary
|-- tests/
|   |-- setup.ts                                Jest DOM test setup
|   |-- schemas.test.ts                         Form schema unit tests
|   `-- blog-api.test.ts                        Blog API request and response tests
|-- public/                                    Static assets parent
|-- next.config.ts                              Rewrites /v1 requests to API_BACKEND_URL
|-- jest.config.js                              Jest configuration and aliases
|-- package.json                                Frontend scripts and dependencies
`-- tsconfig.json                               TypeScript compiler configuration
```

### Backend Parent Child Structure

```text
base_server-master/
|-- src/                                        Backend source parent
|   |-- bin/server.ts                            Process entry -> app.listen -> bootstrap
|   |-- app.ts                                   Express setup -> middleware -> router -> errors
|   |-- bootstrap/index.ts                       Database connect -> rate limiter initialization
|   |-- APIs/                                    Feature and route parent
|   |   |-- index.ts                             Mounts general, auth, user, and blog routes under /v1
|   |   |-- router.ts                            General routes: /self and /health
|   |   |-- controller.ts                        General endpoint handlers
|   |   |-- blog/
|   |   |   |-- index.ts                         /blogs and /blogs/:id route definitions
|   |   |   |-- blog.controller.ts               HTTP validation -> blog service -> standard response
|   |   |   |-- blog.service.ts                  Create, view, ownership, update, delete rules
|   |   |   |-- validation/validation.schema.ts  Joi create and update blog schemas
|   |   |   `-- _shared/
|   |   |       |-- models/blog.model.ts         Mongoose Blog schema
|   |   |       |-- repo/blog.repository.ts      Blog database queries
|   |   |       `-- types/blog.interface.ts      Blog TypeScript interface
|   |   `-- user/
|   |       |-- authentication/
|   |       |   |-- index.ts                     Register, confirm, login, logout routes
|   |       |   |-- authentication.controller.ts HTTP auth flow and cookie handling
|   |       |   |-- authentication.service.ts    Registration, confirmation, login business logic
|   |       |   |-- validation/                  Joi schemas and user validation helpers
|   |       |   `-- types/                       Request contracts
|   |       |-- management/
|   |       |   |-- index.ts                     /user/me route
|   |       |   `-- management.controller.ts     Returns authenticated user
|   |       `-- _shared/
|   |           |-- models/user.model.ts         Mongoose User schema
|   |           |-- models/token.model.ts        Refresh token schema
|   |           |-- repo/user.repository.ts      User queries
|   |           |-- repo/token.repository.ts     Token queries
|   |           `-- types/                       User and token interfaces
|   |-- middlewares/
|   |   |-- authenticate.ts                      Required JWT authentication
|   |   |-- optionalAuthenticate.ts              Public routes may attach a user if token is valid
|   |   |-- rateLimiter.ts                       Request limiting middleware
|   |   `-- errorHandler.ts                      Final Express error response handler
|   |-- handlers/
|   |   |-- async.ts                             Async controller wrapper
|   |   |-- httpResponse.ts                      Standard successful response envelope
|   |   |-- notFound.ts                          Unknown route handler
|   |   |-- logger.ts                            Console and file logging configuration
|   |   `-- errorHandler/                        Error object creation and forwarding helpers
|   |-- services/
|   |   |-- database.ts                          Mongoose connection boundary
|   |   `-- email.ts                             Email delivery boundary
|   |-- config/                                  Environment and rate limiter configuration
|   |-- constant/                                Application, role, and response constants
|   |-- utils/                                   JWT, hashing, parsing, Joi, health helpers
|   |-- types/types.ts                           Shared Express and JWT type definitions
|   `-- __tests__/unit-tests/                    Authentication and blog service tests
|-- docker/                                     Development and production Dockerfiles
|-- docs/                                       Backend notes and API guidance
|-- logs/                                       Runtime log files
|-- scripts/migration.js                         Database migration helper
|-- package.json                                Backend scripts and dependencies
|-- jest.config.js                              Backend Jest configuration
`-- tsconfig.json                               Backend TypeScript configuration
```

## Connection Map Between Files

This flow map explains which parent file calls which child file during the most important user journeys.

### Read a Blog

```text
app/blogs/[id]/page.tsx
  -> features/blog/hook.ts useBlog(id)
    -> features/blog/api.ts getBlog(id)
      -> lib/api/client.ts Axios GET /blogs/:id
        -> next.config.ts rewrite /v1/* to backend
          -> APIs/blog/index.ts GET /blogs/:id
            -> optionalAuthenticate.ts
              -> blog.controller.ts getOne
                -> blog.service.ts getBlogService
                  -> blog.repository.ts findBlogById or increaseViews
                    -> models/blog.model.ts -> MongoDB
```

### Create or Update a Blog

```text
app/blogs/create/page.tsx or app/blogs/[id]/edit/page.tsx
  -> components/blog/blog-form.tsx
    -> features/blog/schema.ts Zod validation
      -> features/blog/hook.ts useCreateBlog or useUpdateBlog
        -> features/blog/api.ts POST or PATCH /blogs
          -> APIs/blog/index.ts authenticate middleware
            -> authenticate.ts resolves authenticatedUser from cookie or Bearer token
              -> blog.controller.ts validates body with Joi
                -> blog.service.ts derives author and checks ownership
                  -> blog.repository.ts writes Mongoose document
                    -> MongoDB
```

### Restore Login State and Logout

```text
app/layout.tsx
  -> providers/query-provider.tsx
    -> providers/auth-provider.tsx on page load
      -> features/auth/api.ts getCurrentUser()
        -> GET /user/me -> user/management route -> authenticate.ts -> controller
          -> AuthProvider stores user and isAuthenticated state
            -> components/layout/navbar.tsx changes visible actions
Navbar logout button
  -> AuthProvider.logout() -> PUT /logout -> backend deletes refresh token and clears cookies
```

## Complete User Journey Flow Charts

These flow charts show what happens after a user clicks a button. Each chart follows the real route through the Next.js frontend, feature API and hooks, Express route, controller, service, repository, and MongoDB. The chart names the most relevant project files at every step.

### 1. Public Browse and View

A logged-out visitor reads a public blog article and creates a counted view.

![Diagram](<Blog Architecture Documentation Complete User Flows.assets/image3.png>)

*Figure 3. Public Browse and View flow.*

### 2. Register and Confirm Account

A new reader creates an account, receives a confirmation link, and activates the account.

![Diagram](<Blog Architecture Documentation Complete User Flows.assets/image4.png>)

*Figure 4. Register and Confirm Account flow.*

### 3. Login and Restore Session

A registered user signs in. Cookies persist the authenticated session across refreshes.

![Diagram](<Blog Architecture Documentation Complete User Flows.assets/image5.png>)

*Figure 5. Login and Restore Session flow.*

### 4. Create Blog

Only a logged-in user can publish. The server assigns ownership from the authenticated identity.

![Diagram](<Blog Architecture Documentation Complete User Flows.assets/image6.png>)

*Figure 6. Create Blog flow.*

### 5. Edit Own Blog

The UI offers edit only to the apparent owner, but the backend independently verifies ownership.

![Diagram](<Blog Architecture Documentation Complete User Flows.assets/image7.png>)

*Figure 7. Edit Own **Blog flow.*

### 6. Delete Own Blog

Deletion requires an explicit browser confirmation and an owner check on the backend.

![Diagram](<Blog Architecture Documentation Complete User Flows.assets/image8.png>)

*Figure 8. Delete Own Blog flow.*

### 7. Logout

Logout clears server-side credentials and removes the frontend session state.

![Diagram](<Blog Architecture Documentation Complete User Flows.assets/image9.png>)

*Figure 9. Lo**gout flow.*

## Architecture Decisions and Operational Rules

This section resolves the important architecture questions identified during review. It distinguishes the current behavior from planned production improvements so that the README does not claim unimplemented features.

### 1. Blog Visibility and Ownership Policy

Published blogs are public. Visitors can browse the blog list and open a published article whether they are logged in or logged out. Logging in does not reduce a reader's access.

Only authenticated users can create a blog. Only the blog's author can edit or delete it. The frontend may hide unavailable actions, but the backend is the authoritative enforcement point and verifies ownership from the authenticated identity.

| Action | Logged-out visitor | Logged-in non-owner | Blog author |
| --- | --- | --- | --- |
| View all published blogs | Allowed | Allowed | Allowed |
| View one published blog | Allowed | Allowed | Allowed |
| Create a blog | Not allowed | Allowed | Allowed |
| Edit a blog | Not allowed | Not allowed | Allowed |
| Delete a blog | Not allowed | Not allowed | Allowed |

### 2. Cookie and CSRF Security Policy

The application uses HTTP-only cookies for authentication. HTTP-only cookies protect token confidentiality in browser JavaScript, but they are not, by themselves, a defense against cross-site request forgery (CSRF).

For a production deployment, state-changing requests such as creating, updating, deleting, and logging out must be protected by an explicit browser trust-boundary control: appropriate `SameSite` and `Secure` cookie settings, plus either `Origin`/`Referer` validation or a CSRF-token mechanism. This control and its negative security tests are planned production work; it is not claimed as implemented by this README.

```text
Trusted BaseBlog frontend ── authenticated write ──► Express API
                                                     │
                                                     ├─ validate cookie/session
                                                     └─ validate request origin or CSRF token

Untrusted third-party site ── forged browser write ─► rejected
```

### 3. Blog View-Count Policy

In the current basic project, a view is counted when the public article endpoint is requested. This is a request counter, not a unique-reader metric: refreshes, bots, retries, and repeat visits can increase it.

The counter should use an atomic database increment so concurrent reads do not overwrite one another. A future version may define unique-view behavior, for example one counted view per authenticated user or visitor session within a selected time interval.

### 4. API Contract Rules

The backend owns the API contract and frontend TypeScript types must follow it. Each endpoint should document:

- HTTP method and route
- Authentication and ownership requirement
- Request body and validation rules
- Success response structure
- Error response structure and status codes
- Pagination and filtering behavior where applicable

The current route inventory is an overview. Publishing a versioned OpenAPI or Swagger specification is a future improvement for independent client development and contract testing.

### 5. Deployment and Recovery Plan

This repository is currently designed for local development and learning. Before a production deployment, the following operational practices must be defined and tested:

- Environment-variable and secret-management process
- Backend health and readiness checks
- MongoDB backup and restore procedure
- Database migration procedure
- Deployment rollback procedure
- Error logging, alerting, and ownership of incident response

These are planned production requirements, not guarantees currently provided by the local development setup.

### 6. Refresh-Token Lifecycle

The current authentication flow stores refresh tokens and invalidates the stored refresh token on logout. Expired or revoked refresh tokens must not issue a new access token.

A production-strength lifecycle should additionally define refresh-token expiry, rotation on refresh, reuse detection, device-specific logout, logout from all devices, compromised-token revocation, and the response returned when refresh fails. These lifecycle details and their security tests are planned improvements unless implemented in the backend configuration and services.
