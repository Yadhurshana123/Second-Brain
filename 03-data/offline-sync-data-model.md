---
title: Offline Sync Data Model
folder: 03-data
document_type: offline-data-model
status: architecture-aligned-draft
source_of_truth:
  - Unified Commerce Scope V1
  - Unified Commerce Database Design V4
  - Frontend Architecture V1
  - Backend Architecture Final
access_model: tenant-configurable-rbac-and-feature-permissions
last_updated: 2026-05-10
---
# Offline Sync Data Model

## Purpose

Offline POS allows billing during connectivity loss, but the server remains the final authority when queued transactions sync.

## Approved sync tables

| Table | Role |
|---|---|
| `offline_sync_batches` | One reconnect/sync attempt from a POS device. |
| `offline_sync_items` | Generic queue item for offline-created records. |
| `offline_sale_sync_queue` | Typed sale staging linked one-to-one with sync item. |
| `offline_payment_sync_queue` | Typed payment staging linked one-to-one with sync item. |
| `offline_sync_conflicts` | Explicit conflict record. |
| `offline_sync_audit_logs` | Technical sync lifecycle audit. |

## Sync acceptance flow

```mermaid
flowchart TD
    Device[POS device] --> Batch[offline_sync_batches]
    Batch --> Item[offline_sync_items]
    Item --> Validate[tenant + device + session + price + stock]
    Validate -->|accepted| Source[Create source-of-truth records]
    Validate -->|conflict| Conflict[offline_sync_conflicts]
    Source --> Audit[offline_sync_audit_logs]
    Conflict --> Audit
```

## Conflict rules

| Conflict | Required behavior |
|---|---|
| Duplicate | Reject or map to existing server entity through idempotency. |
| Stock mismatch | Create conflict; do not silently corrupt inventory. |
| Closed session | Reject or require manager-controlled resolution. |
| Price changed | Follow documented offline pricing policy. |
| Validation failed | Store rejected status and error message. |

## Related documents

- [[entities/receipts-audit-offline-entities]]
- [[tenant-consistency-rules]]
