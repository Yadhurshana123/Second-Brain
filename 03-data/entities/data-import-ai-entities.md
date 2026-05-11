---
title: Data Import Ai Entities
folder: 03-data/entities
document_type: entity-reference
status: architecture-aligned-draft
source_of_truth:
  - Unified Commerce Scope V1
  - Unified Commerce Database Design V4
  - Frontend Architecture V1
  - Backend Architecture Final
access_model: tenant-configurable-rbac-and-feature-permissions
last_updated: 2026-05-10
---

# Data Import Ai Entities

## Purpose

This document is a module-wise entity reference generated from the approved database design. It uses table-level column definitions so developers can see primary keys, foreign keys, constraints, and implementation notes without depending on old Markdown content.

## Control rule

| Concern | Required behavior |
|---|---|
| Tenant access | Every tenant-level feature must be configurable by tenant role, user right, permission, and feature assignment. |
| Backend authority | API/application services must validate tenant, feature entitlement, runtime flag, role permission, and same-tenant foreign-key ownership. |
| Frontend behavior | UI may hide unavailable actions, but backend rejection is mandatory for unauthorized writes. |
| Platform exception | Platform-admin-only catalog and tenant-control features remain platform controlled. |

## Entity index

| Entity | Purpose | PK | FK count |
|---|---|---:|---:|
| Entity | Purpose | PK | FK count |
|---|---|---:|---:|
| No dedicated import source-of-truth table in approved database | CSV/Excel import writes into target domain tables after validation. AI-assisted extraction is later-phase workflow and must not create unapproved source-of-truth rows. | 0 | 0 |

## Approved data behavior

| Import target | Target tables | Validation authority |
|---|---|---|
| Product import | `products`, `product_variants`, `product_attributes`, `attribute_values`, `price_list_items`, `inventory_balances` where applicable | Catalog, pricing, inventory, and tenant access services |
| Customer import | `customers`, `customer_addresses`, `customer_auth_accounts` when account creation is requested | Customer service and tenant duplicate rules |
| Supplier import | `suppliers`, `supplier_addresses`, `product_suppliers` | Catalog/supplier service |
| AI extraction | Same target tables only after user review | Import review workflow, confidence threshold, and backend validation |

## Implementation notes

- Do not add generic `import_cache`, `ai_cache`, or staging tables unless approved by database design.
- Imported rows must pass the same service validation as manually created rows.
- Low-confidence AI extraction must remain review-only and must not auto-save business records.
- Sensitive import actions must be auditable through `audit_logs`.

## Related documents

- [[catalog-entities]]
- [[customer-ecommerce-entities]]
- [[../schema-principles]]