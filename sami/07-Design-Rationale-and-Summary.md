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

### Revision Note

An architecture review identified that the original document was internally consistent but deferred several decisions the architecture actually depends on to be implementable, securable, and operable: form/response versioning, form lifecycle rules, authorization policy, concrete API contracts, operational/deployment design, sensitive-data lifecycle, availability mechanisms, respondent-submission reliability, and a verification plan. Each of these is now a first-class part of this document:

- Form lifecycle and mutation rules — §2.4
- Form/response versioning — §5.3
- API contract (schemas, errors, pagination, idempotency, versioning) — §5.1.1
- Authorization matrix — §6.1
- Sensitive data classification, retention, and deletion — §6.2
- Operational architecture (deployment, observability, recovery objectives) — §6.5
- Availability mechanisms and SLO — §6.6
- Testability and verification plan — §6.7
- Respondent failure and duplicate-submission handling — §3.3

What remains legitimately deferred to detailed design is narrower and more concrete: exact infrastructure provider/tooling choices, fine-tuned SLA/SLO numbers, database schema-level optimization, and framework-specific component design.

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
        B2[Validation + Auth]
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

The architecture establishes the high-level structure, boundaries, lifecycle, authorization policy, data-versioning guarantees, and operational model the service depends on. Remaining decisions — framework-specific component design, database schema optimization, infrastructure provider selection, and individual field-level API contracts beyond what §5.1.1 defines — are appropriately left to detailed design.
