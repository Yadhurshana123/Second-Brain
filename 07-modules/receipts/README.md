---
title: "Receipts Module Overview"
folder: "07-modules/receipts"
document_type: "module-readme"
module: "receipts"
feature: "module-overview"
status: "approved-draft"
source_of_truth:
  - "Unified Commerce Scope V1"
  - "Unified Commerce Database Design V4"
  - "Frontend Architecture V1"
  - "Backend Architecture Final"
architecture_rules:
  - "Multi-tenant SaaS"
  - "Configurable tenant RBAC and feature access"
  - "Service Pattern + Repository Pattern"
  - "No CQRS and no MediatR"
  - "PostgreSQL caching/read optimization only; no Redis now"
  - "IndexedDB only for offline POS storage"
---

# Receipts Module Overview

## Module Purpose
The `receipts` module groups related Unified Commerce capabilities and documents how they are implemented across database, backend, API, frontend, security, and cache/storage layers.
All tenant-operational capabilities in this module must support tenant-specific configuration through feature entitlements, role permissions, role-feature assignment, and runtime feature flags.

## Feature Folders
| Feature | Spec | API | History |
|---|---|---|---|
| `receipt-print-logs` | [[features/receipt-print-logs/feature-spec|Spec]] | [[features/receipt-print-logs/api-spec|API]] | [[features/receipt-print-logs/feature-history|History]] |
| `receipt-templates` | [[features/receipt-templates/feature-spec|Spec]] | [[features/receipt-templates/api-spec|API]] | [[features/receipt-templates/feature-history|History]] |
| `receipts` | [[features/receipts/feature-spec|Spec]] | [[features/receipts/api-spec|API]] | [[features/receipts/feature-history|History]] |

## Database Tables Commonly Used
| Table | Purpose in module |
|---|---|
| `receipt_templates` | Supports `receipts` module workflows and tenant-scoped persistence. |
| `receipts` | Supports `receipts` module workflows and tenant-scoped persistence. |
| `receipt_print_logs` | Supports `receipts` module workflows and tenant-scoped persistence. |
| `document_sequences` | Supports `receipts` module workflows and tenant-scoped persistence. |

## Access Control Position
- Platform-admin-only configuration remains platform-controlled.
- Tenant-facing features must be configurable for each customer tenant.
- Feature availability starts with `platform_features` and `tenant_feature_entitlements`.
- Tenant admins configure roles, role permissions, and role-feature access where allowed.
- Backend checks cannot be replaced by menu hiding or disabled buttons.

## Architecture Flow
```mermaid
flowchart TD
    A[User action from UI or POS] --> B[JWT and tenant context]
    B --> C[Tenant feature entitlement check]
    C --> D[Role permission and user-right check]
    D --> E[Business validation]
    E --> F[Service + Repository transaction]
    F --> G[PostgreSQL source of truth]
    G --> H[Audit and query invalidation]
```

## Backend Placement
| Layer | Folder | Responsibility |
|---|---|---|
| API | `POS.API/Modules/Receipts` | Controllers, request models, response models. |
| Application | `POS.Application/Modules/Receipts` | Services, validators, interfaces, and `Dtos/`. |
| Domain | `POS.Domain/Modules/Receipts` | Pure rules and entities where needed. |
| Infrastructure | `POS.Infrastructure/Repositories/Receipts` | EF Core repositories and persistence logic. |

## Frontend Placement
- Feature UI belongs under `src/features/` using React with TypeScript.
- Page composition belongs under `src/pages/` and layout shell folders.
- Server state uses TanStack Query from `core/api/queryClient.ts`.
- Workflow state uses Zustand stores from `src/state/`.
- Offline POS storage uses `core/offline/syncQueue.ts` and IndexedDB.

## Caching and Storage Placement
| Layer | Placement | Rule |
|---|---|---|
| Backend PostgreSQL | Read-optimized queries over till, device, product, price, stock, and receipt tables | Use indexes and existing read models; no Redis in current architecture. |
| Frontend TanStack Query | Terminal settings, active till session, product search, receipt templates | Cache server state per tenant, outlet, device, and user. |
| Zustand | Cart, selected customer, payment step, tendered amounts, modal state | Keep transient POS workflow fast without pretending it is final authority. |
| IndexedDB | Offline POS cart snapshots, queued sales/payments/receipts, last synced catalog version | Use only when offline mode is enabled for the tenant/outlet/device. |

## Related Knowledge Base Links
- Product scope: [[../../01-product/project-scope|Project Scope]]
- Architecture: [[../../02-architecture/README|Architecture]]
- Data: [[../../03-data/README|Data Design]]
- API: [[../../04-api/README|API Standards]]
- Backend: [[../../05-backend/README|Backend]]
- Frontend: [[../../06-frontend/README|Frontend]]
- Security: [[../../09-security-and-compliance/README|Security and Compliance]]

## Implementation Considerations
- Do not add new tables unless approved by database design or a formal schema change.
- Do not create generic cache tables such as tenant cache, product cache, POS cache, or query cache.
- Use PostgreSQL indexes, deterministic queries, and existing read models for repeated backend reads.
- Use invalidation rather than stale UI assumptions after create/update/delete/approval actions.
- Keep tenant boundaries visible in API, repository queries, tests, and audit logs.
- Frontend consideration 1: Use TanStack Query for server data and Zustand only for local interaction state.
- Frontend consideration 2: Use TanStack Query for server data and Zustand only for local interaction state.
- Frontend consideration 3: Use TanStack Query for server data and Zustand only for local interaction state.
- Frontend consideration 4: Use TanStack Query for server data and Zustand only for local interaction state.
- Frontend consideration 5: Use TanStack Query for server data and Zustand only for local interaction state.
- Frontend consideration 6: Use TanStack Query for server data and Zustand only for local interaction state.
- Frontend consideration 7: Use TanStack Query for server data and Zustand only for local interaction state.
- Frontend consideration 8: Use TanStack Query for server data and Zustand only for local interaction state.
- Frontend consideration 9: Use TanStack Query for server data and Zustand only for local interaction state.
- Offline consideration 10: Use IndexedDB only when this feature participates in POS offline continuity or sync recovery.
- Offline consideration 11: Use IndexedDB only when this feature participates in POS offline continuity or sync recovery.
- Offline consideration 12: Use IndexedDB only when this feature participates in POS offline continuity or sync recovery.
- Offline consideration 13: Use IndexedDB only when this feature participates in POS offline continuity or sync recovery.
- Offline consideration 14: Use IndexedDB only when this feature participates in POS offline continuity or sync recovery.
- Offline consideration 15: Use IndexedDB only when this feature participates in POS offline continuity or sync recovery.
- Offline consideration 16: Use IndexedDB only when this feature participates in POS offline continuity or sync recovery.
- Offline consideration 17: Use IndexedDB only when this feature participates in POS offline continuity or sync recovery.
- Offline consideration 18: Use IndexedDB only when this feature participates in POS offline continuity or sync recovery.
- Offline consideration 19: Use IndexedDB only when this feature participates in POS offline continuity or sync recovery.
- Offline consideration 20: Use IndexedDB only when this feature participates in POS offline continuity or sync recovery.
- Offline consideration 21: Use IndexedDB only when this feature participates in POS offline continuity or sync recovery.
- Implementation consideration 22: Keep `module` tenant-scoped and never resolve records without `tenant_id` or a tenant-owned parent reference.
- Implementation consideration 23: Keep `module` tenant-scoped and never resolve records without `tenant_id` or a tenant-owned parent reference.
- Implementation consideration 24: Keep `module` tenant-scoped and never resolve records without `tenant_id` or a tenant-owned parent reference.
- Implementation consideration 25: Keep `module` tenant-scoped and never resolve records without `tenant_id` or a tenant-owned parent reference.
- Implementation consideration 26: Keep `module` tenant-scoped and never resolve records without `tenant_id` or a tenant-owned parent reference.
- Implementation consideration 27: Keep `module` tenant-scoped and never resolve records without `tenant_id` or a tenant-owned parent reference.
- Implementation consideration 28: Keep `module` tenant-scoped and never resolve records without `tenant_id` or a tenant-owned parent reference.
- Implementation consideration 29: Keep `module` tenant-scoped and never resolve records without `tenant_id` or a tenant-owned parent reference.
- Implementation consideration 30: Keep `module` tenant-scoped and never resolve records without `tenant_id` or a tenant-owned parent reference.
- Implementation consideration 31: Keep `module` tenant-scoped and never resolve records without `tenant_id` or a tenant-owned parent reference.
- Implementation consideration 32: Keep `module` tenant-scoped and never resolve records without `tenant_id` or a tenant-owned parent reference.
- Implementation consideration 33: Keep `module` tenant-scoped and never resolve records without `tenant_id` or a tenant-owned parent reference.
- Access consideration 34: Permission `receipts.module.read` may show data, while command permissions must be separately configured per role.
- Access consideration 35: Permission `receipts.module.read` may show data, while command permissions must be separately configured per role.
- Access consideration 36: Permission `receipts.module.read` may show data, while command permissions must be separately configured per role.
- Access consideration 37: Permission `receipts.module.read` may show data, while command permissions must be separately configured per role.
- Access consideration 38: Permission `receipts.module.read` may show data, while command permissions must be separately configured per role.
- Access consideration 39: Permission `receipts.module.read` may show data, while command permissions must be separately configured per role.
- Access consideration 40: Permission `receipts.module.read` may show data, while command permissions must be separately configured per role.
- Access consideration 41: Permission `receipts.module.read` may show data, while command permissions must be separately configured per role.
- Access consideration 42: Permission `receipts.module.read` may show data, while command permissions must be separately configured per role.
- Access consideration 43: Permission `receipts.module.read` may show data, while command permissions must be separately configured per role.
- Access consideration 44: Permission `receipts.module.read` may show data, while command permissions must be separately configured per role.
- Access consideration 45: Permission `receipts.module.read` may show data, while command permissions must be separately configured per role.
- Data consideration 46: Use PostgreSQL indexes/read models for repeated reads; do not create Redis dependency or generic cache tables.
- Data consideration 47: Use PostgreSQL indexes/read models for repeated reads; do not create Redis dependency or generic cache tables.
- Data consideration 48: Use PostgreSQL indexes/read models for repeated reads; do not create Redis dependency or generic cache tables.
- Data consideration 49: Use PostgreSQL indexes/read models for repeated reads; do not create Redis dependency or generic cache tables.
- Data consideration 50: Use PostgreSQL indexes/read models for repeated reads; do not create Redis dependency or generic cache tables.
- Data consideration 51: Use PostgreSQL indexes/read models for repeated reads; do not create Redis dependency or generic cache tables.
- Data consideration 52: Use PostgreSQL indexes/read models for repeated reads; do not create Redis dependency or generic cache tables.
- Data consideration 53: Use PostgreSQL indexes/read models for repeated reads; do not create Redis dependency or generic cache tables.
- Data consideration 54: Use PostgreSQL indexes/read models for repeated reads; do not create Redis dependency or generic cache tables.
- Data consideration 55: Use PostgreSQL indexes/read models for repeated reads; do not create Redis dependency or generic cache tables.
- Data consideration 56: Use PostgreSQL indexes/read models for repeated reads; do not create Redis dependency or generic cache tables.
- Data consideration 57: Use PostgreSQL indexes/read models for repeated reads; do not create Redis dependency or generic cache tables.
- Frontend consideration 58: Use TanStack Query for server data and Zustand only for local interaction state.
- Frontend consideration 59: Use TanStack Query for server data and Zustand only for local interaction state.
- Frontend consideration 60: Use TanStack Query for server data and Zustand only for local interaction state.
- Frontend consideration 61: Use TanStack Query for server data and Zustand only for local interaction state.
- Frontend consideration 62: Use TanStack Query for server data and Zustand only for local interaction state.
- Frontend consideration 63: Use TanStack Query for server data and Zustand only for local interaction state.
- Frontend consideration 64: Use TanStack Query for server data and Zustand only for local interaction state.
- Frontend consideration 65: Use TanStack Query for server data and Zustand only for local interaction state.
