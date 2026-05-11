---
title: Naming Conventions
folder: 03-data
document_type: data-standard
status: architecture-aligned-draft
source_of_truth:
  - Unified Commerce Scope V1
  - Unified Commerce Database Design V4
  - Frontend Architecture V1
  - Backend Architecture Final
access_model: tenant-configurable-rbac-and-feature-permissions
last_updated: 2026-05-10
---
# Naming Conventions

## Purpose

Naming must stay consistent across database, backend modules, API contracts, and frontend models.

## Database names

| Item | Convention | Example |
|---|---|---|
| Tables | snake_case plural | `product_variants` |
| Columns | snake_case | `tenant_id` |
| Primary key | `id` unless reference table uses smallint | `id uuid PK` |
| Foreign key | referenced entity name + `_id` | `product_id` |
| Status column | clear domain suffix when needed | `payment_status`, `fulfillment_status` |
| Business number | document name + `_number` | `sale_number`, `order_number` |

## Code mapping

| Layer | Preferred style |
|---|---|
| C# entity | PascalCase class and properties |
| DTO | PascalCase class, clear request/response suffix |
| TypeScript type | PascalCase type, camelCase fields after API mapping |
| Permission code | dot-separated module action | `catalog.product.create` |

## Notes

- Do not invent alternative names when a table or column already exists in the approved schema.
- Enum-like values must match the documented database check values.
- API names may be friendly, but persistence mapping must remain exact.
