# 3. Architecture Overview

## 3.1 Big Picture

```mermaid
flowchart LR
    subgraph UI[Frontend Layer]
        A[Form Builder]
        B[Form Management]
        C[Response Interface]
    end

    subgraph API[Backend Layer]
        D[Form API]
        E[Question API]
        F[Response API]
    end

    subgraph DATA[Data Layer]
        G[(Database)]
    end

    A -->|HTTP| D
    B -->|HTTP| D
    C -->|HTTP| F
    D --> E
    E --> G
    F --> G
```

The frontend communicates with the backend through HTTP APIs.

The backend owns business rules and persistence.

The frontend does not directly access the database.

## 3.2 System Context

```mermaid
flowchart TD
    A[Form Owner] --> B[Frontend]
    B -->|HTTPS| C[Form Service Backend]
    C --> D[(Database)]
    E[Respondent] -->|HTTPS| B
```

Authentication is provided by the existing authentication/user infrastructure.

## 3.3 User Interaction Flow

```mermaid
flowchart TD
    A[Login] --> B[Forms Dashboard]
    B --> C[Create Form]
    C --> D[Configure Questions]
    D --> E[Save Draft]
    E --> F[Publish]
    F --> G[Share Form]
    G --> H[View Responses]
```

### Respondent flow

```mermaid
flowchart TD
    A[Open Form] --> B[Load Published Form]
    B --> C[Enter Answers]
    C --> D[Submit]
    D --> E[Submission Result]
```
