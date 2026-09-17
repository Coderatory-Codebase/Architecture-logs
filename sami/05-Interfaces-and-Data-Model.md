# 5. Interfaces and Data Model

## 5.1 Interface Definition

The primary interface between frontend and backend is REST/HTTP.

Example:

```http
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

## 5.2 Data Flow

### Form Creation

```mermaid
flowchart TD
    A[Frontend] -->|POST /forms| B[Controller]
    B --> C[Validation]
    C --> D[Form Service]
    D --> E[Form Repository]
    E --> F[(Database)]
    F --> G[Response]
    G --> H[Frontend]
```

### Response Submission

```mermaid
flowchart TD
    A[Respondent] --> B[Frontend]
    B -->|POST /forms/:formId/responses| C[Controller]
    C --> D[Validation]
    D --> E[Response Service]
    E --> F[Form/Question Repository]
    E --> G[Response Repository]
    F --> H[(Database)]
    G --> H
    H --> I[Response]
    I --> J[Frontend]
```

## 5.3 Data Model

```mermaid
classDiagram
    class User
    class Form
    class Question
    class Response
    class Answer

    User "1" --> "N" Form
    Form "1" --> "N" Question
    Form "1" --> "N" Response
    Response "1" --> "N" Answer
```

The Form Service owns form-related data while user identity remains owned by the authentication/user system.

## 5.4 Architectural Patterns

### Client-Server

The frontend acts as the client and the backend acts as the server.

The client requests services through HTTP APIs, while the server owns business logic and data management.

### Layered Architecture

The backend uses:

```mermaid
flowchart TD
    A[Route] --> B[Controller]
    B --> C[Validation]
    C --> D[Service]
    D --> E[Repository]
    E --> F[Model]
    F --> G[(Database)]
```

This separates HTTP handling, business logic, and persistence.

### Repository / Data Mapper Boundary

Persistence is isolated behind repositories.

```mermaid
flowchart TD
    A[Service] --> B[Repository Interface]
    B --> C[Database Implementation]
```

The domain/service layer therefore does not need to depend directly on database-specific operations.

### Modular Service Pattern

Form functionality is organized into feature boundaries:

```mermaid
flowchart LR
    A[Form] --> B[Question]
    A --> C[Response]
```

Each module maintains a focused responsibility.
