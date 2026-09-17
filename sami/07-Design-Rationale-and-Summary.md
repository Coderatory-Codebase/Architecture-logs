# 7. Design Rationale and Architectural Summary

## 7.1 Design Rationale

The architecture separates the system into client-side presentation, backend application/domain logic, and persistence.

This provides:

- Independent frontend evolution
- Clear backend responsibilities
- Replaceable persistence implementation
- Testable business logic
- Controlled dependencies
- Feature-level modularity

The design intentionally avoids coupling the UI directly to the database or allowing controllers to become the location of business logic.

## 7.2 Architectural Summary

```mermaid
flowchart LR
    subgraph FE[Frontend]
        F1[Presentation]
        F2[Feature State]
        F3[API Layer]
    end

    subgraph BE[Backend]
        B1[Route / Controller]
        B2[Validation]
        B3[Service]
        B4[Repository]
        B5[Model]
    end

    subgraph DB[(Database)]
        D1[Persistent Storage]
    end

    F1 --> F2
    F2 --> F3
    F3 -->|HTTPS| B1
    B1 --> B2
    B2 --> B3
    B3 --> B4
    B4 --> B5
    B5 --> D1
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
