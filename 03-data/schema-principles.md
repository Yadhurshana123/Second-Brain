---
title: Schema Principles
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
# Schema Principles

## Purpose

This document defines the rules developers must follow when implementing the approved database design.

## Core principles

| Principle | Required behavior |
|---|---|
| Tenant isolation | Tenant-owned data must be filtered by authenticated tenant context. |
| Configurable access | Tenant-level features must not use fixed access assumptions. |
| Normalized writes | Transactional source-of-truth data must stay normalized. |
| Ledgers over overwrites | Stock, loyalty, gateway events, and audit history use append-style records. |
| Read models are projections | Reporting summaries are rebuildable summaries, not source ledgers. |
| Same-tenant references | FK values must belong to the same tenant unless they reference platform catalog data. |

## Write validation sequence

```mermaid
sequenceDiagram
    participant UI as Frontend
    participant API as API Controller
    participant App as Application Service
    participant DB as Database
    UI->>API: Request with ids and payload
    API->>App: Authenticated context + DTO
    App->>App: Validate tenant, feature, permission
    App->>App: Validate FK ownership and business state
    App->>DB: Persist inside transaction
    DB-->>App: Constraints enforce final safety
    App-->>UI: Standard response
```

## Do not create

| Wrong pattern | Reason |
|---|---|
| Generic cache tables | Not in approved schema and can conflict with source-of-truth rules. |
| Hardcoded cashier/manager rights | Tenant-specific RBAC and feature assignment are required. |
| Stock quantity on products | Stock belongs to outlet/variant inventory tables. |
| Plain secrets in JSON config | Payment secrets must use secret references. |

## Related documents

- [[database-overview]]
- [[tenant-consistency-rules]]
- [[indexing-strategy]]
