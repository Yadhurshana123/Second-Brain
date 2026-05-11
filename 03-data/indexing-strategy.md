---
title: Indexing Strategy
folder: 03-data
document_type: data-performance
status: architecture-aligned-draft
source_of_truth:
  - Unified Commerce Scope V1
  - Unified Commerce Database Design V4
  - Frontend Architecture V1
  - Backend Architecture Final
access_model: tenant-configurable-rbac-and-feature-permissions
last_updated: 2026-05-10
---
# Indexing Strategy

## Purpose

Indexes must support tenant isolation, uniqueness, lookup speed, offline idempotency, and reporting queries without changing business ownership rules.

## Required index categories

| Category | Examples |
|---|---|
| Tenant uniqueness | `UNIQUE (tenant_id, code)`, `UNIQUE (tenant_id, sku)` |
| Partial uniqueness | email/phone/barcode where not null; outlet-specific nullable overrides |
| Idempotency | payment idempotency key and offline client ids |
| Reporting | tenant/outlet/date/channel summary lookups |
| Status workflows | order, fulfillment, payment, sync, and session statuses |

## Partial index examples

```sql
CREATE UNIQUE INDEX ux_users_email
ON users (tenant_id, normalized_email)
WHERE normalized_email IS NOT NULL;

CREATE UNIQUE INDEX ux_open_till_session
ON till_sessions (tenant_id, till_id)
WHERE status = 'open';
```

## Implementation notes

| Area | Guidance |
|---|---|
| Catalog | SKU, barcode, slug, and category lookups must be tenant-scoped. |
| POS | Sale number and offline transaction ids need uniqueness. |
| Payments | Idempotency keys protect duplicate captures. |
| Offline sync | Device plus client entity id prevents duplicate replay. |
| Reports | Business date and scope columns should be indexed together. |

## Related documents

- [[tenant-consistency-rules]]
- [[entities/receipts-audit-offline-entities]]
