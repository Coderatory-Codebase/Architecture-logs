# POS System Requirements

[Back to Documentation Index](README.md)

## Functional Requirements

- Users can log in securely.
- Admin can create, update, and remove products.
- The system stores product categories and stock quantities.
- Cashier can search products and build a cart.
- The system validates stock before checkout.
- The system supports cash, card, and wallet payments.
- The system generates a receipt after successful checkout.
- The system stores customer records and purchase history.
- Manager can view sales and stock reports.
- The system provides low-stock information.

## Role Permissions

| Feature | Cashier | Manager | Admin |
|---|---:|---:|---:|
| Complete checkout | Yes | Yes | Yes |
| View products | Yes | Yes | Yes |
| Manage inventory | No | Yes | Yes |
| View reports | No | Yes | Yes |
| Manage users and roles | No | No | Yes |

## Non-Functional Requirements

- Stock must never become negative during concurrent sales.
- Passwords and payment credentials must not be stored in plain text.
- Checkout should be fast under normal store load.
- Repeated checkout requests must not create duplicate sales.
- Important actions and errors must be logged.
