---
title: Entity Relationship Map
folder: 03-data
document_type: relationship-map
status: architecture-aligned-draft
source_of_truth:
  - Unified Commerce Scope V1
  - Unified Commerce Database Design V4
  - Frontend Architecture V1
  - Backend Architecture Final
access_model: tenant-configurable-rbac-and-feature-permissions
last_updated: 2026-05-10
---
# Entity Relationship Map

## Purpose

This document gives the high-level relationship path between major data areas. Full column-level PK/FK details are maintained in [[entities/README]].

## Core relationship diagram

```mermaid
flowchart TD
    Tenants[tenants] --> Outlets[outlets]
    Tenants --> Users[users]
    Tenants --> Roles[roles]
    Roles --> RolePermissions[role_permissions]
    Tenants --> Products[products]
    Products --> Variants[product_variants]
    Outlets --> Inventory[inventory_balances]
    Variants --> Inventory
    Outlets --> Tills[tills]
    Tills --> TillSessions[till_sessions]
    TillSessions --> Sales[sales]
    Sales --> SaleLines[sale_lines]
    Variants --> SaleLines
    Customers[customers] --> Orders[orders]
    Orders --> OrderItems[order_items]
    Orders --> Deliveries[deliveries]
    Sales --> Payments[payments]
    Orders --> Payments
    Sales --> Returns[returns]
    Orders --> Returns
```

## Interpretation rules

| Relationship | Rule |
|---|---|
| Tenant to child | Child must never cross tenant boundaries. |
| Product to variant | Variants are sellable SKU/barcode units. |
| Stock to variant/outlet | Stock is stored by outlet and variant, not on product. |
| Sale/order to payments | Payments are allocated through allocation tables. |
| Return/exchange | Must reference original sale/order or return according to table rules. |

## Related documents

- [[data-dictionary-index]]
- [[entities/catalog-entities]]
- [[entities/pos-device-sales-entities]]
