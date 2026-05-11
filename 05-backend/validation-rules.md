---
title: Validation Rules
folder: 05-backend
document_type: backend-validation-rules
status: approved-draft
source_of_truth:
  - Unified Commerce Scope V1
  - Unified Commerce Database Design V4
  - Frontend Architecture V1
  - Backend Architecture final
architecture_pattern: Clean Architecture with Service Pattern and Repository Pattern
cqrs: not used
mediatr: not used
dto_rule: Dtos folder with one DTO per C# file
tenant_access_model: configurable RBAC, feature assignment, permission configuration, and runtime feature flags
last_updated: 2026-05-10
---

# Validation Rules

Purpose: Defines validation responsibilities across API, application service, domain model and database constraints.

This document is written for the Unified Commerce backend: a multi-tenant SaaS platform combining POS, E-Commerce, inventory, payment, refund, receipt, return, exchange, offline sync, reporting and audit capabilities.

Related reading: [[feature-access-handling]], [[exception-handling]], [[service-layer-rules]]

## Architecture Position

- Backend implementation follows Clean Architecture with explicit Application Services and Repositories.
- CQRS is not part of this backend design.
- MediatR is not part of this backend design.
- The backend is the final authority for tenant isolation, RBAC, feature access, stock, tax, payment, refund, sync and audit.
- Frontend visibility is allowed for usability, but it is not security.
- Except platform-admin-only features, tenant-level features must be configurable by tenant roles, permissions and feature assignments.

## Approved Pattern

```mermaid
flowchart TD
    A[API model binding] --> B[DTO validation]
    B --> C[Access validation]
    C --> D[Business state validation]
    D --> E[Domain invariant]
    E --> F[Database constraint]
```

## System-Specific Responsibilities

| Validation type | Layer | Example |
|---|---|---|
| Shape validation | API | Required fields and basic formats |
| Permission validation | Application | `catalog.product.create` |
| Tenant ownership | Application/repository | Product category belongs to tenant |
| Domain invariant | Domain | Quantity > 0 |
| Database constraint | Database | Unique tenant SKU or barcode |

## Core Rules

- Do not introduce CQRS, MediatR, command handlers or query handlers for this backend.
- Use explicit services, repositories, validators, DTOs and UnitOfWork transactions.
- Follow SOLID: services depend on interfaces, domain stays pure, infrastructure is replaceable.

## 1. Validation layers

Validate tenant consistency for every FK referenced by a request.
Validate status transitions using configured workflow rules where the database provides transition tables.
Validate financial totals server-side even if the frontend calculated a preview.

## 2. Tenant consistency validation

Validate tenant consistency for every FK referenced by a request.
Validate status transitions using configured workflow rules where the database provides transition tables.
Validate financial totals server-side even if the frontend calculated a preview.

## 3. Status validation

Validate tenant consistency for every FK referenced by a request.
Validate status transitions using configured workflow rules where the database provides transition tables.
Validate financial totals server-side even if the frontend calculated a preview.

## 4. Financial validation

Validate tenant consistency for every FK referenced by a request.
Validate status transitions using configured workflow rules where the database provides transition tables.
Validate financial totals server-side even if the frontend calculated a preview.

## 5. Validation examples

Validate tenant consistency for every FK referenced by a request.
Validate status transitions using configured workflow rules where the database provides transition tables.
Validate financial totals server-side even if the frontend calculated a preview.

## Implementation Example

```csharp
if (dto.Qty <= 0)
    throw ValidationErrors.Field("qty", "Quantity must be greater than zero.");
if (product.TenantId != context.TenantId)
    throw SecurityErrors.TenantMismatch();
```

## API Behavior Example

```http
POST /api/v1/{module}/{resource}
Authorization: Bearer <jwt>
X-Tenant-Id: <tenant-id>
Content-Type: application/json
```

## Tenant-Specific Behavior

- A platform admin may enable a feature for a tenant through tenant feature entitlements.
- A tenant admin configures roles and assigns permissions only inside that tenant boundary.
- Outlet-scoped users must be validated against `outlet_user_roles` where outlet context is required.
- Tenant-level users must be validated against `tenant_user_roles` for tenant-wide actions.
- Runtime flags may disable a feature for a tenant, outlet or user even when entitlement exists.
- Backend services must check this model on every protected write and sensitive read.

## Data Flow References

- `tenants` controls tenant lifecycle and operating mode.
- `platform_features` defines platform-owned feature catalog.
- `tenant_feature_entitlements` controls which features are available to each tenant.
- `roles` and `permissions` define configurable tenant access behavior.
- `role_permissions` and `role_feature_assignments` connect tenant roles to actions and features.
- `feature_flags` applies runtime tenant/outlet/user-level configuration.
- `audit_logs` records sensitive business or configuration changes.

## Implementation Considerations

- Keep controllers thin and move workflow orchestration into application services.
- Keep repositories persistence-focused and transaction-neutral.
- Keep domain models free from EF Core, HTTP and infrastructure concerns.
- Use UnitOfWork for workflows that span multiple tables or modules.
- Use stable permission codes instead of hardcoded role names.
- Use tenant id in all tenant-owned repository methods.
- Do not add generic cache tables or unsupported database shortcuts.
- Do not store secrets, plain OTP codes, card data or payment private keys in JSON columns.
- Record actor, tenant, outlet/device context and reason for sensitive actions.
- Prefer explicit validators over hidden validation inside controllers.

## Do Not Implement

- Do not implement CQRS handlers for this backend.
- Do not introduce MediatR pipelines.
- Do not hardcode cashier, manager or tenant admin capabilities in service logic.
- Do not trust frontend-calculated totals for final sale, order, refund or tax decisions.
- Do not bypass tenant ownership validation for any FK in request payloads.
- Do not create undocumented tables or generic cache tables.

## Review Questions

- Does this implementation preserve tenant isolation?
- Does the service check tenant entitlement, permission and runtime feature flags?
- Does the repository query include tenant id for tenant-owned records?
- Is the transaction boundary correct for all records written?
- Are DTOs separated from API request and database entity models?
- Are sensitive changes audited with actor and reason where required?

## Related Documents

- [[feature-access-handling]]
- [[exception-handling]]
- [[service-layer-rules]]

- Note: Treat offline device payloads as untrusted until accepted by the server.
- Note: Use deterministic error codes for frontend and support workflows.
- Note: Keep module dependencies visible through interfaces and documented service calls.
- Note: Avoid circular service dependencies by moving shared pure logic into domain services or shared application services.
- Note: Do not mix platform admin operations with tenant staff operations in one authorization path.
- Note: Keep payment, stock and audit writes in the same workflow transaction when business consistency requires it.
- Note: Prefer explicit code over hidden magic for enterprise workflows.
- Note: Use database constraints as safety guarantees, not as the only business validation layer.
- Note: Keep feature configuration readable for support, audit and tenant administration.
- Note: Use structured logs with tenant id, actor id, trace id and module name.
