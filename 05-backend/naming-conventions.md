---
title: Naming Conventions
folder: 05-backend
document_type: backend-naming-conventions
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

# Naming Conventions

Purpose: Defines backend naming conventions for modules, services, repositories, DTOs, validators, requests, responses and permission codes.

This document is written for the Unified Commerce backend: a multi-tenant SaaS platform combining POS, E-Commerce, inventory, payment, refund, receipt, return, exchange, offline sync, reporting and audit capabilities.

Related reading: [[backend-folder-structure]], [[dto-handling]], [[mapping-rules]]

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
    Module[Products] --> Service[ProductService]
    Module --> Repo[IProductRepository/ProductRepository]
    Module --> Dtos[Dtos/CreateProductDto.cs]
    Module --> Validator[ProductValidator.cs]
    Module --> Controller[ProductsController.cs]
```

## System-Specific Responsibilities

| Artifact | Pattern | Example |
|---|---|---|
| Service interface | `I{Module}Service` | `IProductService` |
| Service class | `{Module}Service` | `ProductService` |
| Repository interface | `I{Entity}Repository` | `IProductRepository` |
| DTO | `{Action}{Entity}Dto` | `CreateProductDto` |
| Permission code | `{module}.{resource}.{action}` | `pos.sale.void` |

## Core Rules

- Names must express business intent and match module responsibility.
- Names should align with approved database and architecture terminology.
- Avoid ambiguous names such as `Manager`, `Processor`, `Helper` unless the business responsibility is explicit.

## 1. Project naming

This area must follow the approved scope, database and backend architecture.
Implementation must stay tenant-scoped and permission-aware.
Avoid adding tables, patterns or modules that are not supported by the approved design.

## 2. Module naming

This area must follow the approved scope, database and backend architecture.
Implementation must stay tenant-scoped and permission-aware.
Avoid adding tables, patterns or modules that are not supported by the approved design.

## 3. DTO naming

API request models represent HTTP input shape.
Application DTOs represent service input and output.
Response DTOs must not expose password hashes, secret references, internal sync payloads or unauthorized tenant data.

## 4. Permission code naming

Access decisions must evaluate tenant entitlement, role feature assignment, permission code and runtime feature flag.
A tenant can customize operational roles such as cashier, outlet manager, inventory user or e-commerce operator.
Hardcoded role behavior is not acceptable for tenant-level features.

## 5. Database-aligned names

Use module names that match business capability names from the scope and database.
Use singular service names and repository names based on aggregate or table ownership.
Permission codes must be descriptive and stable because they are used in configuration and audit.

## Implementation Example

```text
POS.Application/Modules/Products/Dtos/CreateProductDto.cs
POS.Application/Modules/Products/IProductService.cs
POS.Application/Modules/Products/ProductService.cs
POS.Infrastructure/Repositories/Products/ProductRepository.cs
POS.API/Modules/Products/Controllers/ProductsController.cs
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

- [[backend-folder-structure]]
- [[dto-handling]]
- [[mapping-rules]]

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
