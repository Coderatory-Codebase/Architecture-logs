# 4. Logical and Process View

## 4.1 Frontend Logical View

```mermaid
flowchart TB
    A[Presentation]
    A --> B[Pages]
    A --> C[Components]
    A --> D[Form Builder]

    D --> E[Application]
    E --> F[Form State]
    E --> G[Question State]
    E --> H[Response State]

    H --> I[API Layer]
    I --> J[Form API]
    I --> K[Question API]
    I --> L[Response API]
```

### Frontend Layer Responsibilities

#### Presentation Layer

Responsible for:

- Rendering forms
- User interaction
- Form builder UI
- Validation feedback
- Loading/error states

It should not contain backend business rules.

#### Feature/Application Layer

Responsible for:

- Managing UI state
- Coordinating user actions
- Preparing API requests
- Handling API results

Examples:

- Form Management
- Form Builder
- Question Management
- Response Management

#### API Layer

Provides a single boundary between frontend features and backend APIs.

Example:

- formApi.create()
- formApi.update()
- formApi.publish()
- questionApi.create()
- questionApi.update()
- responseApi.submit()
- responseApi.getAll()

The UI should not directly construct HTTP requests throughout individual components.

## 4.2 Backend Logical View

```mermaid
flowchart TB
    A[Backend] --> B[Form Module]
    A --> C[Response Module]
    B --> D[Question Module]
    D --> E[Service Layer]
    E --> F[Repository]
    F --> G[Model]
    G --> H[(Database)]
```

## 4.3 Process View

```mermaid
flowchart LR
    A[Web Browser] -->|HTTPS / REST| B[Form Service API]
    B --> C[Repository]
    C --> D[(Database)]
```

The browser and backend communicate through request/reply interactions.

## 4.4 Runtime Request Flow

```mermaid
sequenceDiagram
    participant User as User
    participant UI as Frontend Component
    participant API as Frontend API Layer
    participant Route as Backend Route
    participant Ctrl as Controller
    participant Valid as Validation
    participant Svc as Service
    participant Repo as Repository
    participant DB as Database

    User->>UI: Initiates action
    UI->>API: API call
    API->>Route: HTTP request
    Route->>Ctrl: Route to controller
    Ctrl->>Valid: Validate input
    Valid->>Svc: Pass business request
    Svc->>Repo: Persistence operation
    Repo->>DB: Query / write
    DB-->>Repo: Result
    Repo-->>Svc: Data
    Svc-->>Ctrl: Business response
    Ctrl-->>API: HTTP response
    API-->>UI: UI update
    UI-->>User: Final state
```
