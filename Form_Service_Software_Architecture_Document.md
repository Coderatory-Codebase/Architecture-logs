# Form Service

## Software Architecture Document

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Background](#2-background)
3. [Functional Requirements](#3-functional-requirements)
4. [Quality Attributes](#4-quality-attributes)
5. [Architecture Overview](#5-architecture-overview)
6. [System Context](#6-system-context)
7. [User Interaction](#7-user-interaction)
8. [Data Flow](#8-data-flow)
9. [Architectural Patterns](#9-architectural-patterns)
10. [Architectural Tactics](#10-architectural-tactics)
11. [Logical View](#11-logical-view)
12. [Frontend Element Catalog](#12-frontend-element-catalog)
13. [Backend Logical View](#13-backend-logical-view)
14. [Backend Element Catalog](#14-backend-element-catalog)
15. [Relations](#15-relations)
16. [Process View](#16-process-view)
17. [Runtime Request](#17-runtime-request)
18. [Interface Definition](#18-interface-definition)
19. [Data Model](#19-data-model)
20. [Security Boundary](#20-security-boundary)
21. [Error Handling](#21-error-handling)
22. [Extensibility](#22-extensibility)
23. [Design Rationale](#23-design-rationale)
24. [Architectural Summary](#24-architectural-summary)

---

## 1. Introduction

The Form Service is a web-based system for creating, configuring, publishing, and submitting forms.

The system consists of two primary subsystems:

- **Frontend** — user-facing form builder, form management, and response interfaces.
- **Backend** — form domain, validation, authorization, persistence, and API services.

The architecture separates presentation concerns from application and data concerns so that either side can evolve independently.

---

## 2. Background

The system provides functionality comparable to a simplified Google Forms platform.

The primary users are:

- **Form Owner** — creates and manages forms.
- **Respondent** — accesses published forms and submits responses.

The existing Base Server architecture provides the backend conventions that this service should follow, including controller/service/repository separation and shared authentication, error handling, and response mechanisms.

---

## 3. Functional Requirements

The architecture must support the following capabilities:

### Form Management

- Create form
- Update form
- Delete form
- Retrieve form
- List forms
- Publish form
- Close form

### Question Management

- Add questions
- Update questions
- Delete questions
- Reorder questions
- Configure question types and options

### Response Management

- Submit response
- Validate response against form definition
- Retrieve responses
- Paginate responses

### Frontend

- Form dashboard
- Form builder
- Question editor
- Form preview
- Published form
- Response interface
- Response viewer

---

## 4. Quality Attributes

The primary architectural drivers are:

### 4.1 Usability

The form builder should allow users to create and configure forms without unnecessary navigation.

Frontend components should provide immediate validation feedback and clear form states.

### 4.2 Maintainability

Frontend and backend responsibilities must remain separated.

Backend features should be modular and follow the existing Base Server structure.

Frontend components should be organized by feature rather than allowing business logic to accumulate inside page components.

### 4.3 Modifiability

The system should allow:

- UI changes without changing backend business logic.
- Database implementation changes without changing business services.
- New question types without restructuring the complete system.
- External integrations to be introduced behind defined interfaces.

### 4.4 Testability

Business logic should be independently testable.

Repositories should be replaceable with mocks/stubs during service testing.

Frontend components should be testable independently from API implementations.

These requirements favor high cohesion and low coupling.

---

## 5. Architecture Overview

### 5.1 Big Picture

```text
                         FORM SERVICE
┌──────────────────────────────────────────────────────┐
│                                                      │
│   ┌──────────────┐          ┌───────────────────┐   │
│   │   Frontend   │  HTTP    │      Backend      │   │
│   │              │◄────────►│                   │   │
│   │ Form Builder │          │ Form API          │   │
│   │ Forms        │          │ Question API      │   │
│   │ Responses    │          │ Response API      │   │
│   └──────────────┘          └─────────┬─────────┘   │
│                                      │              │
│                                      ▼              │
│                              ┌───────────────┐      │
│                              │   Database    │      │
│                              └───────────────┘      │
│                                                      │
└──────────────────────────────────────────────────────┘
```

The frontend communicates with the backend through HTTP APIs.

The backend owns business rules and persistence.

The frontend does not directly access the database.

---

## 6. System Context

```text
                    ┌──────────────┐
                    │ Form Owner   │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │  Frontend    │
                    └──────┬───────┘
                           │
                         HTTPS
                           │
                           ▼
                    ┌──────────────┐
                    │ Form Service │
                    │   Backend    │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │   Database   │
                    └──────────────┘
                           ▲
                           │
                         HTTPS
                           │
                    ┌──────┴───────┐
                    │ Respondent   │
                    └──────────────┘
```

Authentication is provided by the existing authentication/user infrastructure.

---

## 7. User Interaction

### Form Owner

```text
Login
  ↓
Forms Dashboard
  ↓
Create Form
  ↓
Configure Questions
  ↓
Save Draft
  ↓
Publish
  ↓
Share Form
  ↓
View Responses
```

### Respondent

```text
Open Form
  ↓
Load Published Form
  ↓
Enter Answers
  ↓
Submit
  ↓
Submission Result
```

---

## 8. Data Flow

### Form Creation

```text
Frontend
   ↓
POST /forms
   ↓
Controller
   ↓
Validation
   ↓
Form Service
   ↓
Form Repository
   ↓
Database
   ↓
Response
   ↓
Frontend
```

### Response Submission

```text
Respondent
   ↓
Frontend
   ↓
POST /forms/:formId/responses
   ↓
Controller
   ↓
Validation
   ↓
Response Service
   ↓
Form/Question Repository
   ↓
Response Repository
   ↓
Database
   ↓
Response
   ↓
Frontend
```

---

## 9. Architectural Patterns

### 9.1 Client-Server

The frontend acts as the client and the backend acts as the server.

The client requests services through HTTP APIs, while the server owns business logic and data management.

### 9.2 Layered Architecture

The backend uses:

```text
Route
  ↓
Controller
  ↓
Validation
  ↓
Service
  ↓
Repository
  ↓
Model
  ↓
Database
```

This separates HTTP handling, business logic, and persistence.

### 9.3 MVC / Presentation Separation

The frontend is responsible for presentation and user interaction.

The backend is responsible for application/domain behavior.

Changing the frontend implementation should not require changes to Form Service business rules.

### 9.4 Repository / Data Mapper Boundary

Persistence is isolated behind repositories.

```text
Service
   ↓
Repository Interface
   ↓
Database Implementation
```

The domain/service layer therefore does not need to depend directly on database-specific operations.

### 9.5 Modular Service Pattern

Form functionality is organized into feature boundaries:

```text
Form
Question
Response
```

Each module maintains a focused responsibility.

---

## 10. Architectural Tactics

### Maintainability

- Feature-based modularization
- Clear layer boundaries
- Shared infrastructure
- Dependency inversion
- Centralized error handling

### Modifiability

- Frontend/backend separation
- Repository abstraction
- API contracts
- Extensible question configuration

### Testability

- Services independent of HTTP
- Repositories mockable
- Frontend API layer separated from UI components
- Component-level testing

### Availability

- Centralized exception handling
- Database error handling
- Stateless API design
- Pagination for large response collections

---

## 11. Logical View

The Logical View describes the major functional modules and layers of the system.

### 11.1 Frontend

```text
Presentation
    │
    ├── Pages
    ├── Components
    └── Form Builder
          │
          ▼
Application
    │
    ├── Form State
    ├── Question State
    └── Response State
          │
          ▼
API Layer
    │
    ├── Form API
    ├── Question API
    └── Response API
```

---

## 12. Frontend Element Catalog

### Presentation Layer

Responsible for:

- Rendering forms
- User interaction
- Form builder UI
- Validation feedback
- Loading/error states

It should not contain backend business rules.

### Feature/Application Layer

Responsible for:

- Managing UI state
- Coordinating user actions
- Preparing API requests
- Handling API results

Examples:

```text
Form Management
Form Builder
Question Management
Response Management
```

### API Layer

Provides a single boundary between frontend features and backend APIs.

Example:

```text
formApi.create()
formApi.update()
formApi.publish()

questionApi.create()
questionApi.update()

responseApi.submit()
responseApi.getAll()
```

The UI should not directly construct HTTP requests throughout individual components.

---

## 13. Backend Logical View

```text
                  Backend
                     │
          ┌──────────┴──────────┐
          │                     │
     Form Module          Response Module
          │                     │
     Question Module            │
          │                     │
          └──────────┬──────────┘
                     │
                Service Layer
                     │
                Repository
                     │
                  Model
                     │
                 Database
```

---

## 14. Backend Element Catalog

### Route Layer

Defines public API endpoints and maps requests to controllers.

### Controller Layer

Responsible for HTTP-level orchestration.

It should remain independent of database implementation.

### Validation Layer

Defines request contracts and validates external input.

### Service Layer

Contains domain/application rules.

Examples:

```text
createForm
publishForm
updateQuestion
submitResponse
getResponses
```

### Repository Layer

Provides persistence operations.

Examples:

```text
FormRepository
QuestionRepository
ResponseRepository
```

### Model Layer

Defines persistence representation for:

```text
Form
Question
Response
```

---

## 15. Relations

### Frontend → Backend

Frontend communicates through REST APIs over HTTPS.

### Controller → Service

Controllers invoke application services.

### Service → Repository

Services request persistence operations through repository abstractions.

### Repository → Database

Repositories perform database operations.

### Authentication → Form Service

Form Service consumes authenticated user context and applies resource-level authorization.

---

## 16. Process View

The Process View represents runtime components and their communication.

```text
┌───────────────┐
│ Web Browser   │
│               │
│ React Client  │
└───────┬───────┘
        │
        │ HTTPS / REST
        ▼
┌──────────────────────┐
│ Form Service API     │
│                      │
│ Express / Node.js    │
└──────────┬───────────┘
           │
           │ Repository
           ▼
┌──────────────────────┐
│ Database             │
└──────────────────────┘
```

The browser and backend communicate through request/reply interactions.

---

## 17. Runtime Request

```text
User Action
    ↓
Frontend Component
    ↓
Frontend API Layer
    ↓
HTTP Request
    ↓
Backend Route
    ↓
Controller
    ↓
Validation
    ↓
Service
    ↓
Repository
    ↓
Database
    ↓
Repository
    ↓
Service
    ↓
Controller
    ↓
HTTP Response
    ↓
Frontend API Layer
    ↓
UI Update
```

---

## 18. Interface Definition

The primary interface between frontend and backend is REST/HTTP.

Example:

```text
POST /forms
GET /forms
GET /forms/:id
PATCH /forms/:id
DELETE /forms/:id

POST /forms/:id/publish

POST /forms/:formId/questions
PATCH /forms/:formId/questions/:questionId
DELETE /forms/:formId/questions/:questionId

POST /forms/:formId/responses
GET /forms/:formId/responses
```

The API contract defines:

- Request structure
- Response structure
- HTTP status codes
- Validation errors
- Authorization behavior

---

## 19. Data Model

```text
User
 │
 │ 1:N
 ▼
Form
 │
 │ 1:N
 ▼
Question

Form
 │
 │ 1:N
 ▼
Response
 │
 │ 1:N
 ▼
Answer
```

The Form Service owns form-related data while user identity remains owned by the authentication/user system.

---

## 20. Security Boundary

```text
Frontend
   │
   │ Authentication Context
   ▼
Backend
   │
   ├── Authentication
   ├── Authorization
   ├── Validation
   └── Resource Ownership
```

The frontend may hide unavailable actions for usability, but authorization must always be enforced by the backend.

---

## 21. Error Handling

Communication failures use standard HTTP semantics.

Examples:

| Status Code | Meaning |
|---|---|
| 400 | Invalid Request |
| 401 | Unauthenticated |
| 403 | Forbidden |
| 404 | Resource Not Found |
| 409 | Conflict |
| 422 | Validation Failure |
| 500 | Internal Server Error |

Backend errors should be centrally handled and logged.

Frontend should translate API errors into appropriate user-facing feedback.

---

## 22. Extensibility

The architecture should support future functionality without restructuring existing layers.

Potential extensions:

- Anonymous Responses
- File Upload
- Conditional Questions
- Form Templates
- Analytics
- Export
- Notifications
- Collaboration
- Versioning
- External Integrations

External integrations should be introduced through interfaces rather than embedding provider-specific logic into the Form domain.

---

## 23. Design Rationale

The architecture separates the system into client-side presentation, backend application/domain logic, and persistence.

This provides:

- Independent frontend evolution
- Clear backend responsibilities
- Replaceable persistence implementation
- Testable business logic
- Controlled dependencies
- Feature-level modularity

The design intentionally avoids coupling the UI directly to the database or allowing controllers to become the location of business logic.

---

## 24. Architectural Summary

```text
                         FORM SERVICE
                              │
              ┌───────────────┴───────────────┐
              │                               │
          FRONTEND                         BACKEND
              │                               │
       Presentation                    Route / Controller
              │                               │
       Feature State                      Validation
              │                               │
          API Layer                        Service
              │                               │
              └──────── HTTPS ────────────────┤
                                              │
                                         Repository
                                              │
                                            Model
                                              │
                                          Database
```

### Primary Architectural Principles

- Separation of Concerns
- High Cohesion
- Low Coupling
- Dependency Inversion
- Single Responsibility
- Extensibility
- Testability

The architecture establishes the high-level structure and boundaries. Detailed implementation decisions such as framework-specific component design, database schema optimization, deployment topology, and individual API contracts can be defined during detailed design.
