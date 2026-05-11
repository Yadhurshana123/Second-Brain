---
title: "Cache Invalidation And Consistency Feature Specification"
folder: "07-modules/caching-storage-strategy"
document_type: "feature-spec"
module: "caching-storage-strategy"
feature: "cache-invalidation-and-consistency"
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

# Cache Invalidation And Consistency Feature Specification

## Purpose
This document defines the implementation scope for `cache-invalidation-and-consistency` inside the `caching-storage-strategy` module of the Unified Commerce platform.
The feature must support enterprise multi-tenant operation and must never rely on fixed access behavior except for platform-admin-only capabilities.
Every tenant-facing operation is controlled by tenant entitlements, tenant-side feature flags, role permissions, outlet/user role assignment, and backend validation.

## Source Alignment
| Source | Applied decision |
|---|---|
| Scope | Unified POS + E-Commerce business capability with operational auditability. |
| Database | Tenant-owned records, normalized tables, FK consistency, immutable ledgers where required. |
| Backend | Clean Architecture with services, repositories, DTOs, validators, and Unit of Work. |
| Frontend | React TypeScript feature folders, TanStack Query, Zustand, Tailwind, and POS offline core. |

## Database References
| Table | Usage in this feature |
|---|---|
| `daily_sales_summaries` | Source or supporting table for `cache-invalidation-and-consistency` behavior. |
| `daily_payment_summaries` | Source or supporting table for `cache-invalidation-and-consistency` behavior. |
| `daily_inventory_summaries` | Source or supporting table for `cache-invalidation-and-consistency` behavior. |
| `feature_flags` | Source or supporting table for `cache-invalidation-and-consistency` behavior. |
| `tenant_settings` | Source or supporting table for `cache-invalidation-and-consistency` behavior. |
| `offline_sync_items` | Source or supporting table for `cache-invalidation-and-consistency` behavior. |

## Permission Model
| Permission | Meaning | Configurable by tenant? |
|---|---|---|
| `caching-storage-strategy.cache-invalidation-and-consistency.read` | View data and search records. | Yes |
| `caching-storage-strategy.cache-invalidation-and-consistency.create` | Create new records or workflow drafts. | Yes |
| `caching-storage-strategy.cache-invalidation-and-consistency.update` | Modify allowed editable fields. | Yes |
| `caching-storage-strategy.cache-invalidation-and-consistency.approve` | Approve sensitive actions where applicable. | Yes |
| `caching-storage-strategy.cache-invalidation-and-consistency.void` | Void/cancel/reverse where business rules allow. | Yes |

## Tenant-Specific Access Behavior
- Platform admin enables the platform feature for the tenant through `tenant_feature_entitlements`.
- Tenant admin maps enabled features to tenant roles through `role_feature_assignments`.
- Tenant roles receive granular permissions through `role_permissions`.
- Staff users receive tenant-level or outlet-level roles through `tenant_user_roles` or `outlet_user_roles`.
- Backend must evaluate entitlement, feature flag, role, permission, tenant, outlet, and user context before writes.
- Frontend may hide actions, but hidden UI is not security.

## Workflow
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

## Backend Implementation
- Place API request/response contracts in `POS.API/Modules/<Module>/Requests` and `Responses`.
- Place application service, validator, interfaces, and `Dtos/` in `POS.Application/Modules/<Module>`.
- Keep one DTO per `.cs` file inside the module `Dtos/` folder.
- Place pure business rules in `POS.Domain/Modules/<Module>` only when rules are not infrastructure-specific.
- Place EF Core repository implementations in `POS.Infrastructure/Repositories/<Module>`.
- Use service orchestration and repositories; do not introduce CQRS, MediatR, or handler pipelines.

```csharp
public sealed class CacheInvalidationAndConsistencyService : ICacheInvalidationAndConsistencyService
{
    private readonly ICacheInvalidationAndConsistencyRepository _repository;
    private readonly IUnitOfWork _unitOfWork;
    private readonly IAccessDecisionService _access;

    public async Task<ApiResponse<CacheInvalidationAndConsistencyDto>> HandleAsync(CacheInvalidationAndConsistencyRequest request, UserContext actor)
    {
        await _access.RequireAsync(actor, "caching-storage-strategy.cache-invalidation-and-consistency.manage");
        // Validate tenant, outlet, feature entitlement, role permission, and request rules.
        // Query and persist through repositories; do not use CQRS or MediatR.
        await _unitOfWork.SaveChangesAsync();
        return ApiResponse.Success(new CacheInvalidationAndConsistencyDto());
    }
}
```

## Frontend Implementation
- Place API functions in `src/features/<feature>/api` or the closest existing module folder.
- Use TanStack Query for server state and invalidation after mutations.
- Use Zustand for local workflow state such as selected rows, cart steps, modals, and active panels.
- Use Tailwind CSS for consistent enterprise SaaS layouts and POS touch targets.
- Use `core/offline` and IndexedDB only where offline POS continuity is explicitly required.

```ts
export const cacheinvalidationandconsistencyQueryKey = (tenantId: string, outletId?: string) =>
  ["caching-storage-strategy", "cache-invalidation-and-consistency", tenantId, outletId ?? "tenant"] as const;

export function useCacheInvalidationAndConsistency(tenantId: string, outletId?: string) {
  return useQuery({
    queryKey: cacheinvalidationandconsistencyQueryKey(tenantId, outletId),
    queryFn: () => api.get("/api/v1/caching-storage-strategy/cache-invalidation-and-consistency"),
    staleTime: 60_000,
  });
}
```

## Caching and Offline Storage Placement
| Layer | Placement | Rule |
|---|---|---|
| Backend PostgreSQL | Indexed tenant-scoped reads and existing normalized tables | No Redis and no generic cache table in current design. |
| Frontend TanStack Query | Server state returned from API queries | Use structured query keys and mutation invalidation. |
| Zustand | UI-only workflow state | Avoid duplicating server authority. |
| IndexedDB | Not default | Use only for offline POS support approved by feature flag. |

## Validation Rules
- `tenant_id` must be resolved from JWT/context and must match every tenant-owned parent record.
- Outlet-scoped commands must validate `outlet_id` and user outlet role assignment.
- Feature entitlement must exist before tenant-side configuration can enable the feature.
- Sensitive actions must include actor, reason where required, and audit record.
- Commands that may be retried must use idempotency keys or client transaction identifiers.
- Calculated values previewed by frontend must be recalculated by backend before persistence.

## API Example
```http
POST /api/v1/caching-storage-strategy/cache-invalidation-and-consistency
Authorization: Bearer <jwt>
X-Tenant-Id: <tenant-id>
X-Outlet-Id: <outlet-id-if-outlet-scoped>
Idempotency-Key: <required-for-command-operations>
Content-Type: application/json

{
  "featureKey": "caching-storage-strategy.cache-invalidation-and-consistency",
  "scope": "tenant-or-outlet",
  "payload": { "example": true }
}
```

## Data Flow References
- Related module overview: [[../README|Caching Storage Strategy Overview]]
- API standards: [[../../04-api/README|API Standards]]
- Backend rules: [[../../05-backend/README|Backend Architecture]]
- Frontend rules: [[../../06-frontend/README|Frontend Architecture]]
- Data design: [[../../03-data/README|Data Architecture]]

## Implementation Notes
- This document defines feature behavior, not UI decoration only.
- Any cache decision must preserve PostgreSQL as the durable source of truth.
- IndexedDB data must be treated as local pending state until server sync accepts it.
- Permission names shown here are implementation guidance and must be aligned with the platform permission catalog.
- Feature-specific edge cases must be added to the feature history before coding starts.
