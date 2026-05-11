---
title: Session Rules
folder: 09-security-and-compliance
document_type: security-control-specification
status: approved-draft
source_of_truth:
  - Unified Commerce Scope V1
  - Unified Commerce Database Design V4
  - Frontend Architecture V1
  - Backend Architecture final
platform_rule: tenant features are configurable through RBAC, feature entitlement, permissions, and user rights
document_id: SEC-SESSION
last_updated: 2026-05-11
tags: [session, jwt, till]
---

# Session Rules

## Purpose
This document defines authentication session, JWT lifetime, refresh behavior, tenant context, outlet selection, till session dependency, inactivity handling, and forced logout behavior.

## Source-of-truth alignment
| Area | Required alignment |
|---|---|
| Scope | Unified Commerce must support POS, e-commerce, offline POS, payments, returns, inventory, reporting, and tenant configuration. |
| Database | Security controls must map to approved tables instead of invented storage. |
| Backend | .NET Clean Architecture uses service pattern and repository pattern, not CQRS or MediatR. |
| Frontend | React TypeScript uses route guards, TanStack Query, Zustand, Tailwind CSS, and IndexedDB for offline POS only. |
| Access | Tenant features are configurable through entitlements, feature flags, roles, permissions, and user rights. |

## Related documents
- [[../01-product/project-scope|Project Scope]]
- [[../02-architecture/architecture-overview|Architecture Overview]]
- [[../03-data/data-model-overview|Data Model Overview]]
- [[../04-api/auth-and-authorization|API Auth and Authorization]]
- [[../05-backend/backend-architecture|Backend Architecture]]
- [[../06-frontend/frontend-architecture|Frontend Architecture]]
- [[../07-modules|Module Specifications]]

## Tables and data ownership
| Table / entity | Security relevance |
|---|---|
| `users` | Must be tenant-aware, access-controlled, audited, or protected according to this document. |
| `platform_users` | Must be tenant-aware, access-controlled, audited, or protected according to this document. |
| `tenants` | Must be tenant-aware, access-controlled, audited, or protected according to this document. |
| `outlets` | Must be tenant-aware, access-controlled, audited, or protected according to this document. |
| `pos_devices` | Must be tenant-aware, access-controlled, audited, or protected according to this document. |
| `tills` | Must be tenant-aware, access-controlled, audited, or protected according to this document. |
| `till_sessions` | Must be tenant-aware, access-controlled, audited, or protected according to this document. |
| `audit_logs` | Must be tenant-aware, access-controlled, audited, or protected according to this document. |

## Required permissions and feature gates
| Permission / gate | Usage rule |
|---|---|
| `session.create` | Must be checked by backend services before the corresponding operation is executed. |
| `session.refresh` | Must be checked by backend services before the corresponding operation is executed. |
| `session.revoke` | Must be checked by backend services before the corresponding operation is executed. |
| `till.session.open` | Must be checked by backend services before the corresponding operation is executed. |
| `till.session.close` | Must be checked by backend services before the corresponding operation is executed. |
| Tenant feature entitlement | Required for non-platform features before tenant users can configure or use the feature. |
| Runtime feature flag | May disable or narrow behavior by tenant, outlet, or user scope. |

## API surface examples
| Method | Endpoint | Purpose |
|---|---|---|
| `POST` | `/api/v1/auth/refresh` | Refresh user session. |
| `POST` | `/api/v1/sessions/revoke` | Revoke active session. |

```http
POST /api/v1/auth/refresh HTTP/1.1
Authorization: Bearer <jwt>
X-Tenant-Id: <tenant-id>
X-Outlet-Id: <outlet-id-if-required>
Idempotency-Key: <key-for-write-operations>
Content-Type: application/json
```

## Mermaid control flow
```mermaid
flowchart TD
  A[Login] --> B[JWT session]
  B --> C[Outlet/device selection]
  C --> D[Till session guard]
  D --> E[POS allowed]
  B --> F[Admin layout allowed by role]
```

## Backend implementation rules
- Validate JWT signature, expiry, issuer, audience, and token purpose before building actor context.
- Resolve tenant context from trusted token claims or validated request context, not from arbitrary body fields.
- Use Application services for workflow orchestration and repositories for database access.
- Do not implement CQRS, MediatR, or generic security bypass handlers.
- DTOs must live in module `Dtos/` folders with one DTO per `.cs` file.
- Every write operation must validate tenant, feature entitlement, role permission, runtime flag, and business status.
- Repository queries must include tenant filters for tenant-owned data.
- Sensitive operations must append `audit_logs` or `offline_sync_audit_logs` where applicable.
- Payment secrets must use `secret_ref`; do not store private keys or card data in JSON payloads.
- PostgreSQL may be used for indexed read optimization; do not add Redis or generic cache tables now.

## Frontend implementation rules
- Use `AuthGuard` for authenticated areas and `RoleGuard` for permission-sensitive pages.
- Use `TillSessionGuard` before POS billing, payment, return, or cash drawer workflows that require an open till.
- Use TanStack Query for server-state caching and invalidation after mutations.
- Use Zustand only for local workflow state such as cart, UI, till, session, and offline indicators.
- Use IndexedDB through `core/offline` only for offline POS data, sync queue data, and cached POS reference data.
- Do not treat hidden buttons as security; backend must reject unauthorized calls.
- Tenant-specific layout must render actions from effective permissions and feature flags.

## Validation rules
- `tenant_id` must match the authenticated tenant context for tenant-owned actions.
- `outlet_id` must be required for outlet-scoped POS, inventory, device, and till actions.
- Actor status must be active before any authenticated operation continues.
- Tenant status must allow the requested operation; suspended tenants cannot process new sales/orders unless explicitly permitted.
- Permission code must be mapped to an active role assignment for the actor.
- Feature must be entitled to the tenant before runtime flags or role feature assignments can allow usage.
- Request payloads must not override server-derived actor, tenant, outlet, or device context.
- Idempotency must be enforced for payments, refunds, orders, offline sync, and duplicate-sensitive writes.
- Invalid status transitions must be blocked before persistence.

## Tenant-specific behavior
- Platform admin controls platform-level features and tenant entitlements.
- Tenant admin configures tenant roles, role permissions, feature assignment, and runtime behavior where entitled.
- Outlet users receive outlet-specific roles when workflows depend on physical outlet context.
- The same feature may be enabled for one tenant and disabled for another.
- The same role name can have different permissions across tenants; code must not hardcode role behavior.
- Backend authorization must compute effective access from database state, not from frontend assumptions.

## Security failure behavior
| Failure | Required response |
|---|---|
| Missing JWT | Return `401 Unauthorized`. |
| Invalid tenant context | Return `403 Forbidden` or `404 Not Found` without leaking cross-tenant data. |
| Feature not entitled | Return `403 FeatureDisabled`. |
| Permission missing | Return `403 PermissionDenied`. |
| Invalid status transition | Return `409 Conflict`. |
| Duplicate idempotency key | Return original accepted result or controlled duplicate response. |

## Audit and logging requirements
- Record actor type, actor id, tenant id, outlet/device id where relevant, entity type, entity id, action, and timestamp.
- For configuration changes, capture before and after values when safe.
- For failed sensitive actions, log enough diagnostic context without exposing secrets.
- Audit records must not be updated by normal application users.
- Offline sync technical lifecycle belongs in `offline_sync_audit_logs`; business actions still belong in `audit_logs`.

## Example service guard pattern
```csharp
public async Task EnsureAllowedAsync(SecurityContext ctx, string featureKey, string permissionCode)
{
    await _tenantAccessService.EnsureTenantActiveAsync(ctx.TenantId);
    await _featureService.EnsureTenantFeatureEnabledAsync(ctx.TenantId, featureKey);
    await _permissionService.EnsureUserPermissionAsync(ctx.UserId, ctx.TenantId, ctx.OutletId, permissionCode);
}
```

## Data flow references
- Authentication context flows from JWT into API middleware and Application service guards.
- Authorization context flows from roles, permissions, feature entitlements, and feature flags.
- POS device context flows from registered device and till session into sales, payments, receipts, and offline sync.
- Payment and refund context flows through payment rows, allocation tables, provider transactions, and audit logs.
- Offline POS context flows from IndexedDB to sync queues and then into authoritative server tables.

## Non-negotiable constraints
- Do not create generic cache tables such as `backend_cache`, `tenant_cache`, or `query_cache`.
- Do not introduce Redis in the current documented design.
- Do not store plain passwords, plain OTPs, gateway private keys, or card data.
- Do not bypass backend authorization because the frontend already hid an action.
- Do not allow cross-tenant joins without tenant ownership validation.

## Implementation notes
- Keep security logic reusable, but avoid a single uncontrolled god-service.
- Keep permission names stable because they are stored and assigned to tenant roles.
- Add database indexes for tenant-scoped lookup paths used by security-critical APIs.
- Security errors should be clear to authorized admins but should not leak protected data.
- Documentation in this folder applies across all module specifications and user flows.

