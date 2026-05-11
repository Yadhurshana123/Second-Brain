---
title: Request Response Standard
folder: 04-api
document_type: api-contract-standard
order: 5
status: approved-draft
last_updated: 2026-05-10
source_of_truth:
  - Unified Commerce Scope V1
  - Unified Commerce Database Design Final V2/V4
  - Frontend Architecture V1
  - Backend Architecture Final
core_architecture_rule: tenant-level features require configurable RBAC, feature assignment, permissions, and user-right customization
---

# Request Response Standard

## Purpose
Define common request and response structure for APIs so frontend, backend, tests, and AI IDE implementation remain consistent.

## Standard Success Envelope
```json
{
  "success": true,
  "data": { "id": "uuid", "status": "active" },
  "meta": {
    "requestId": "req_01",
    "timestamp": "2026-05-10T10:00:00Z"
  }
}
```

## Standard Error Envelope
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "One or more validation errors occurred.",
    "details": [
      { "field": "sku", "message": "SKU already exists for this tenant." }
    ]
  },
  "meta": { "requestId": "req_01" }
}
```

## Request DTO Rules
- Request DTOs must not expose database entities directly.
- Tenant id should not be accepted in body for normal tenant APIs.
- Server-side actor, tenant, outlet, and device context must come from auth/context resolution.
- Write DTOs must include only fields the actor is allowed to set.
- Sensitive computed fields such as totals must be recalculated by backend.

## Response DTO Rules
| Response Type | Rule |
|---|---|
| Detail response | Return business-safe fields only |
| List response | Include paging metadata |
| Command response | Return id, status, business number, and summary |
| Audit-related response | Do not expose secrets or raw sensitive payloads |
| Payment response | Hide private provider config and card data |

## Pagination Contract
```http
GET /api/v1/catalog/products?page=1&pageSize=25&sort=name:asc&status=active
```

```json
{
  "success": true,
  "data": [],
  "meta": {
    "page": 1,
    "pageSize": 25,
    "totalItems": 143,
    "totalPages": 6
  }
}
```

## Filtering and Sorting
- Filters must be explicit and documented per endpoint.
- Sorting should use whitelisted fields only.
- Search must be tenant-scoped.
- Date filters must specify timezone behavior for business date vs timestamp.
- Reports should use business date, outlet, channel, and tenant filters.

## Command Request Example
```json
{
  "items": [
    { "variantId": "uuid", "qty": 2 }
  ],
  "payments": [
    { "methodType": "cash", "amount": 1500.00 }
  ],
  "idempotencyKey": "tenant-device-sale-001"
}
```

## Related Documents
- [[error-contract]]
- [[endpoint-design]]
- [[idempotency-rules]]
- [[module-endpoint-map]]

## Implementation Checklist
- Confirm whether the endpoint is platform-level or tenant-level.
- Resolve authenticated actor from JWT claims before business logic.
- Resolve tenant context from route/header/subdomain according to the approved rule.
- Reject requests where target records do not belong to the resolved tenant.
- Validate platform feature entitlement when the action is feature-gated.
- Validate runtime feature flag when a tenant/outlet/user override exists.
- Validate role permissions and role-feature assignments.
- Validate request DTO with module-specific validators.
- Use application service orchestration for business workflows.
- Use repository and Unit of Work for transactional writes.
- Recalculate sensitive totals server-side.
- Record audit logs for sensitive actions and configuration changes.
- Return standard response envelope and standard error contract.
- Add tests for allowed, denied, invalid, duplicate, and cross-tenant cases.
- Confirm whether the endpoint is platform-level or tenant-level.
- Resolve authenticated actor from JWT claims before business logic.
- Resolve tenant context from route/header/subdomain according to the approved rule.
- Reject requests where target records do not belong to the resolved tenant.
- Validate platform feature entitlement when the action is feature-gated.
- Validate runtime feature flag when a tenant/outlet/user override exists.
- Validate role permissions and role-feature assignments.
- Validate request DTO with module-specific validators.
- Use application service orchestration for business workflows.
- Use repository and Unit of Work for transactional writes.
- Recalculate sensitive totals server-side.
- Record audit logs for sensitive actions and configuration changes.
- Return standard response envelope and standard error contract.
- Add tests for allowed, denied, invalid, duplicate, and cross-tenant cases.
- Confirm whether the endpoint is platform-level or tenant-level.
- Resolve authenticated actor from JWT claims before business logic.
- Resolve tenant context from route/header/subdomain according to the approved rule.
- Reject requests where target records do not belong to the resolved tenant.
- Validate platform feature entitlement when the action is feature-gated.
- Validate runtime feature flag when a tenant/outlet/user override exists.
- Validate role permissions and role-feature assignments.
- Validate request DTO with module-specific validators.
- Use application service orchestration for business workflows.
- Use repository and Unit of Work for transactional writes.
- Recalculate sensitive totals server-side.
- Record audit logs for sensitive actions and configuration changes.
- Return standard response envelope and standard error contract.
- Add tests for allowed, denied, invalid, duplicate, and cross-tenant cases.
- Confirm whether the endpoint is platform-level or tenant-level.
- Resolve authenticated actor from JWT claims before business logic.
- Resolve tenant context from route/header/subdomain according to the approved rule.
- Reject requests where target records do not belong to the resolved tenant.
- Validate platform feature entitlement when the action is feature-gated.
- Validate runtime feature flag when a tenant/outlet/user override exists.
- Validate role permissions and role-feature assignments.
- Validate request DTO with module-specific validators.
- Use application service orchestration for business workflows.
- Use repository and Unit of Work for transactional writes.
- Recalculate sensitive totals server-side.
- Record audit logs for sensitive actions and configuration changes.
- Return standard response envelope and standard error contract.
- Add tests for allowed, denied, invalid, duplicate, and cross-tenant cases.
- Confirm whether the endpoint is platform-level or tenant-level.
