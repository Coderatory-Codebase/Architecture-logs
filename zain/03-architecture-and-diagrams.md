# POS System Architecture and Diagrams

[Back to Documentation Index](README.md)

## Architecture Style

The system uses a layered modular monolith. Each business module contains its own controller, service, repository, and data models.

## Overall Architecture

```mermaid
flowchart LR
    User[Cashier / Manager / Admin] --> UI[React Frontend]
    UI --> API[REST API]
    API --> Security[Authentication and RBAC]
    Security --> Module[Business Modules]
    Module --> Service[Application Service]
    Service --> Repository[Repository]
    Repository --> DB[(MongoDB)]
    Service --> Gateway[Payment Gateway]
    DB --> Response[API Response]
    Gateway --> Response
    Response --> UI
```

## Checkout Flow

```mermaid
flowchart TD
    Start([Start]) --> Login[User logs in]
    Login --> Valid{Credentials valid?}
    Valid -->|No| LoginError[Show login error]
    LoginError --> Login
    Valid -->|Yes| Products[Load products]
    Products --> Cart[Add products to cart]
    Cart --> Confirm{Confirm checkout?}
    Confirm -->|No| Cart
    Confirm -->|Yes| Validate[Validate cart and prices]
    Validate --> Stock{Stock available?}
    Stock -->|No| StockError[Show insufficient stock]
    StockError --> Cart
    Stock -->|Yes| Reserve[Reserve stock atomically]
    Reserve --> Payment[Process payment]
    Payment --> Approved{Payment approved?}
    Approved -->|No| Release[Release reserved stock]
    Release --> PaymentError[Show payment error]
    PaymentError --> Cart
    Approved -->|Yes| Save[Save sale, payment, and stock movement]
    Save --> Receipt[Generate receipt]
    Receipt --> End([Checkout complete])
```

## Backend Layer Flow

```mermaid
flowchart LR
    Request[HTTP Request] --> Controller[Controller]
    Controller --> Service[Service]
    Service --> Repository[Repository]
    Repository --> Database[(MongoDB)]
    Database --> Repository
    Repository --> Service
    Service --> Controller
    Controller --> Response[HTTP Response]
```

## Modules

- Auth: login and JWT authentication
- Users: user accounts and roles
- Inventory: products, categories, and stock
- Sales: cart, checkout, and receipts
- Payments: payment status and methods
- Customers: customer records and history
- Reports: sales and inventory summaries

## Folder Structure

### Backend

```text
backend/
└── src/
    ├── modules/
    │   ├── auth/
    │   │   ├── controller/
    │   │   ├── service/
    │   │   ├── repository/
    │   │   └── routes.ts
    │   ├── users/
    │   ├── inventory/
    │   ├── sales/
    │   ├── payments/
    │   ├── customers/
    │   └── reports/
    ├── config/
    │   ├── database.ts
    │   └── env.ts
    ├── middleware/
    │   ├── authentication.ts
    │   ├── authorization.ts
    │   ├── validation.ts
    │   └── error-handler.ts
    ├── infrastructure/
    │   ├── logger/
    │   └── payment-gateway/
    └── app.ts
```

### Frontend

```text
frontend/
└── src/
    ├── pages/
    │   ├── Login/
    │   ├── POS/
    │   ├── Inventory/
    │   ├── Sales/
    │   ├── Reports/
    │   └── Customers/
    ├── components/
    │   ├── common/
    │   └── pos/
    ├── services/
    │   ├── auth.api.ts
    │   ├── inventory.api.ts
    │   └── sales.api.ts
    ├── hooks/
    ├── store/
    ├── types/
    └── utils/
```

## Layer Responsibilities

| Layer | Responsibility |
|---|---|
| Controller | Handles HTTP requests and responses |
| Service | Contains business logic |
| Repository | Performs database operations |
| Model | Defines MongoDB document structure |
| Middleware | Handles authentication, authorization, and validation |
