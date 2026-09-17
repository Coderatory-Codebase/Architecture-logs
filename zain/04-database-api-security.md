# POS Database, API, and Security

[Back to Documentation Index](README.md)

## Database Collections

| Collection | Main Fields | Purpose |
|---|---|---|
| users | name, role, password_hash | System users |
| products | name, sku, price, stock_qty, category_id | Product and inventory data |
| categories | name | Product grouping |
| sales | cashier_id, customer_id, items, total, createdAt | Completed sales |
| payments | sale_id, method, amount, status | Payment records |
| customers | name, phone | Customer records |

## API Endpoints

| Method | Endpoint | Access |
|---|---|---|
| POST | `/api/auth/login` | Public |
| GET | `/api/inventory/products` | Cashier and above |
| POST | `/api/inventory/products` | Manager and above |
| POST | `/api/sales/checkout` | Cashier and above |
| GET | `/api/sales` | Manager and above |
| GET | `/api/customers` | Cashier and above |
| GET | `/api/reports/sales` | Manager and above |

## Checkout Rules

- Server recalculates prices and totals.
- Stock is updated atomically.
- Payment and sale are saved together.
- The `Idempotency-Key` prevents duplicate checkout.
- Failed payments release reserved stock.

## Security

- JWT authentication
- Role-Based Access Control
- Bcrypt password hashing
- Input validation
- Rate limiting on login and payment requests
- HTTPS for API communication
- Payment card data is not stored
- Secrets remain in environment variables
