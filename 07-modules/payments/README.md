---
title: "Payments Module Overview"
folder: "07-modules/payments"
document_type: "module-readme"
module: "payments"
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

# Payments Module Overview

## Module Purpose
The `payments` module groups related Unified Commerce capabilities and documents how they are implemented across database, backend, API, frontend, security, and cache/storage layers.
All tenant-operational capabilities in this module must support tenant-specific configuration through feature entitlements, role permissions, role-feature assignment, and runtime feature flags.

## Feature Folders
| Feature | Spec | API | History |
|---|---|---|---|
| `payment-allocations` | [[features/payment-allocations/feature-spec|Spec]] | [[features/payment-allocations/api-spec|API]] | [[features/payment-allocations/feature-history|History]] |
| `payment-method-types` | [[features/payment-method-types/feature-spec|Spec]] | [[features/payment-method-types/api-spec|API]] | [[features/payment-method-types/feature-history|History]] |
| `payment-provider-configs` | [[features/payment-provider-configs/feature-spec|Spec]] | [[features/payment-provider-configs/api-spec|API]] | [[features/payment-provider-configs/feature-history|History]] |
| `payment-transactions` | [[features/payment-transactions/feature-spec|Spec]] | [[features/payment-transactions/api-spec|API]] | [[features/payment-transactions/feature-history|History]] |
| `payments` | [[features/payments/feature-spec|Spec]] | [[features/payments/api-spec|API]] | [[features/payments/feature-history|History]] |
| `refunds` | [[features/refunds/feature-spec|Spec]] | [[features/refunds/api-spec|API]] | [[features/refunds/feature-history|History]] |
| `tenant-payment-methods` | [[features/tenant-payment-methods/feature-spec|Spec]] | [[features/tenant-payment-methods/api-spec|API]] | [[features/tenant-payment-methods/feature-history|History]] |

## Database Tables Commonly Used
| Table | Purpose in module |
|---|---|
| `payment_method_types` | Supports `payments` module workflows and tenant-scoped persistence. |
| `tenant_payment_methods` | Supports `payments` module workflows and tenant-scoped persistence. |
| `payment_provider_configs` | Supports `payments` module workflows and tenant-scoped persistence. |
| `payments` | Supports `payments` module workflows and tenant-scoped persistence. |
| `payment_transactions` | Supports `payments` module workflows and tenant-scoped persistence. |
| `sale_payment_allocations` | Supports `payments` module workflows and tenant-scoped persistence. |
| `order_payment_allocations` | Supports `payments` module workflows and tenant-scoped persistence. |
| `refunds` | Supports `payments` module workflows and tenant-scoped persistence. |

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
| API | `POS.API/Modules/Payments` | Controllers, request models, response models. |
| Application | `POS.Application/Modules/Payments` | Services, validators, interfaces, and `Dtos/`. |
| Domain | `POS.Domain/Modules/Payments` | Pure rules and entities where needed. |
| Infrastructure | `POS.Infrastructure/Repositories/Payments` | EF Core repositories and persistence logic. |

## Frontend Placement
- Feature UI belongs under `src/features/` using React with TypeScript.
- Page composition belongs under `src/pages/` and layout shell folders.
- Server state uses TanStack Query from `core/api/queryClient.ts`.
- Workflow state uses Zustand stores from `src/state/`.
- Offline POS storage uses `core/offline/syncQueue.ts` and IndexedDB.

## Caching and Storage Placement
| Layer | Placement | Rule |
|---|---|---|
| Backend PostgreSQL | Transaction tables with idempotency keys, status history, and summary read models | Transactions are durable records, not cache; no Redis. |
| Frontend TanStack Query | Entity detail, status history, payment/refund/order lookup | Invalidate aggressively after state-changing operations. |
| Zustand | Multi-step form progress, selected lines, approval modal state | Do not keep final totals as authoritative state. |
| IndexedDB | Only POS/offline transaction queues where the workflow is cashier-side | Online admin/e-commerce flows should not use IndexedDB as a system cache. |

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
- Offline consideration 2: Use IndexedDB only when this feature participates in POS offline continuity or sync recovery.
- Offline consideration 3: Use IndexedDB only when this feature participates in POS offline continuity or sync recovery.
- Offline consideration 4: Use IndexedDB only when this feature participates in POS offline continuity or sync recovery.
- Offline consideration 5: Use IndexedDB only when this feature participates in POS offline continuity or sync recovery.
- Offline consideration 6: Use IndexedDB only when this feature participates in POS offline continuity or sync recovery.
- Offline consideration 7: Use IndexedDB only when this feature participates in POS offline continuity or sync recovery.
- Offline consideration 8: Use IndexedDB only when this feature participates in POS offline continuity or sync recovery.
- Offline consideration 9: Use IndexedDB only when this feature participates in POS offline continuity or sync recovery.
- Offline consideration 10: Use IndexedDB only when this feature participates in POS offline continuity or sync recovery.
- Offline consideration 11: Use IndexedDB only when this feature participates in POS offline continuity or sync recovery.
- Offline consideration 12: Use IndexedDB only when this feature participates in POS offline continuity or sync recovery.
- Offline consideration 13: Use IndexedDB only when this feature participates in POS offline continuity or sync recovery.
- Implementation consideration 14: Keep `module` tenant-scoped and never resolve records without `tenant_id` or a tenant-owned parent reference.
- Implementation consideration 15: Keep `module` tenant-scoped and never resolve records without `tenant_id` or a tenant-owned parent reference.
- Implementation consideration 16: Keep `module` tenant-scoped and never resolve records without `tenant_id` or a tenant-owned parent reference.
- Implementation consideration 17: Keep `module` tenant-scoped and never resolve records without `tenant_id` or a tenant-owned parent reference.
- Implementation consideration 18: Keep `module` tenant-scoped and never resolve records without `tenant_id` or a tenant-owned parent reference.
- Implementation consideration 19: Keep `module` tenant-scoped and never resolve records without `tenant_id` or a tenant-owned parent reference.
- Implementation consideration 20: Keep `module` tenant-scoped and never resolve records without `tenant_id` or a tenant-owned parent reference.
- Implementation consideration 21: Keep `module` tenant-scoped and never resolve records without `tenant_id` or a tenant-owned parent reference.
- Implementation consideration 22: Keep `module` tenant-scoped and never resolve records without `tenant_id` or a tenant-owned parent reference.
- Implementation consideration 23: Keep `module` tenant-scoped and never resolve records without `tenant_id` or a tenant-owned parent reference.
- Implementation consideration 24: Keep `module` tenant-scoped and never resolve records without `tenant_id` or a tenant-owned parent reference.
- Implementation consideration 25: Keep `module` tenant-scoped and never resolve records without `tenant_id` or a tenant-owned parent reference.
- Access consideration 26: Permission `payments.module.read` may show data, while command permissions must be separately configured per role.
- Access consideration 27: Permission `payments.module.read` may show data, while command permissions must be separately configured per role.
- Access consideration 28: Permission `payments.module.read` may show data, while command permissions must be separately configured per role.
- Access consideration 29: Permission `payments.module.read` may show data, while command permissions must be separately configured per role.
- Access consideration 30: Permission `payments.module.read` may show data, while command permissions must be separately configured per role.
- Access consideration 31: Permission `payments.module.read` may show data, while command permissions must be separately configured per role.
- Access consideration 32: Permission `payments.module.read` may show data, while command permissions must be separately configured per role.
- Access consideration 33: Permission `payments.module.read` may show data, while command permissions must be separately configured per role.
- Access consideration 34: Permission `payments.module.read` may show data, while command permissions must be separately configured per role.
- Access consideration 35: Permission `payments.module.read` may show data, while command permissions must be separately configured per role.
- Access consideration 36: Permission `payments.module.read` may show data, while command permissions must be separately configured per role.
- Access consideration 37: Permission `payments.module.read` may show data, while command permissions must be separately configured per role.
- Data consideration 38: Use PostgreSQL indexes/read models for repeated reads; do not create Redis dependency or generic cache tables.
- Data consideration 39: Use PostgreSQL indexes/read models for repeated reads; do not create Redis dependency or generic cache tables.
- Data consideration 40: Use PostgreSQL indexes/read models for repeated reads; do not create Redis dependency or generic cache tables.
- Data consideration 41: Use PostgreSQL indexes/read models for repeated reads; do not create Redis dependency or generic cache tables.
- Data consideration 42: Use PostgreSQL indexes/read models for repeated reads; do not create Redis dependency or generic cache tables.
- Data consideration 43: Use PostgreSQL indexes/read models for repeated reads; do not create Redis dependency or generic cache tables.
- Data consideration 44: Use PostgreSQL indexes/read models for repeated reads; do not create Redis dependency or generic cache tables.
- Data consideration 45: Use PostgreSQL indexes/read models for repeated reads; do not create Redis dependency or generic cache tables.
- Data consideration 46: Use PostgreSQL indexes/read models for repeated reads; do not create Redis dependency or generic cache tables.
- Data consideration 47: Use PostgreSQL indexes/read models for repeated reads; do not create Redis dependency or generic cache tables.
- Data consideration 48: Use PostgreSQL indexes/read models for repeated reads; do not create Redis dependency or generic cache tables.
- Data consideration 49: Use PostgreSQL indexes/read models for repeated reads; do not create Redis dependency or generic cache tables.
- Frontend consideration 50: Use TanStack Query for server data and Zustand only for local interaction state.
- Frontend consideration 51: Use TanStack Query for server data and Zustand only for local interaction state.
- Frontend consideration 52: Use TanStack Query for server data and Zustand only for local interaction state.
- Frontend consideration 53: Use TanStack Query for server data and Zustand only for local interaction state.
- Frontend consideration 54: Use TanStack Query for server data and Zustand only for local interaction state.
- Frontend consideration 55: Use TanStack Query for server data and Zustand only for local interaction state.
- Frontend consideration 56: Use TanStack Query for server data and Zustand only for local interaction state.
- Frontend consideration 57: Use TanStack Query for server data and Zustand only for local interaction state.
