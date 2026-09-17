# POS System Overview

[Back to Documentation Index](README.md)

## Project Title

**Role-Based Modular POS Management System**

## Definition

This is a Point of Sale (POS) system for managing store products, inventory, sales, payments, customers, and reports in one system.

## Objectives

- Provide fast checkout for cashiers.
- Maintain accurate inventory.
- Support Cashier, Manager, and Admin roles.
- Record cash, card, and wallet payments.
- Generate receipts and management reports.

## Users

| Role | Main Responsibility |
|---|---|
| Cashier | Sell products, process checkout, and generate receipts |
| Manager | Manage inventory and view sales reports |
| Admin | Manage users, roles, products, and system access |
| Customer | Customer record and purchase history only |

## Overall System Flow

```text
User Login
  -> Product Selection
  -> Cart
  -> Stock Check
  -> Payment
  -> Sale Record
  -> Inventory Update
  -> Receipt
```

## Technology Summary

- Frontend: React
- Backend: Node.js and TypeScript
- Database: MongoDB with Mongoose
- API: REST
- Authentication: JWT and RBAC
