---
title: "Frontend Naming Conventions"
folder: "06-frontend"
document_type: "frontend-implementation-guideline"
status: "approved-draft"
source_of_truth:
  - "Unified Commerce Scope V1"
  - "Unified Commerce Database Design final V2"
  - "Frontend architecture V1"
  - "Backend Architecture final"
technology_stack:
  - "React"
  - "TypeScript"
  - "TanStack Query"
  - "Zustand"
  - "Tailwind CSS"
  - "IndexedDB for offline POS storage"
last_updated: "2026-05-10"
ownership: "Frontend architecture and SaaS tenant UI design"
description: "Consistent frontend naming rules."
---

# Frontend Naming Conventions

## Purpose
- Defines file, function, hook, store, and route naming standards.
- Applies to the approved React + TypeScript + TanStack Query + Zustand + Tailwind CSS frontend.
- Must support tenant-specific feature access and configurable permissions.
- Must stay consistent with backend Clean Architecture API boundaries.

## Naming Purpose
- Names must reveal business meaning.
- Names must align with approved modules and database concepts.
- Names must avoid vague generic labels where domain language exists.
- Names must distinguish platform, tenant, outlet, POS, and customer concepts.

## File Naming Table
| Item | Convention | Example |
|---|---|---|
| component | PascalCase | `ProductSearchPanel.tsx` |
| hook | camelCase with `use` | `useProductSearchQuery.ts` |
| store | kebab/camel plus store | `cart.store.ts` |
| type file | PascalCase or domain name | `ProductTypes.ts` |
| API file | domain api | `productApi.ts` |
| page | PascalCase + Page | `TillOpenPage.tsx` |
| layout | PascalCase + Layout | `POSLayout.tsx` |

## Hook Names
- Query hooks start with `use` and end with `Query`.
- Mutation hooks start with `use` and end with `Mutation`.
- UI hooks describe the UI behavior.
- Access hooks use `useCan` or domain-specific wrapper.

## Hook Examples
```ts
useProductSearchQuery
useCompleteSaleMutation
useOpenTillSessionMutation
useReceiptReprintMutation
useOfflineSyncStatusQuery
useTenantAccessContextQuery
```

## Store Naming
| Store | Action naming examples |
|---|---|
| `cart.store.ts` | `addItem`, `removeLine`, `changeQty`, `clearCart` |
| `till.store.ts` | `setActiveSession`, `clearSession`, `markClosing` |
| `offline.store.ts` | `setOnline`, `setQueueSummary`, `markSyncing` |
| `ui.store.ts` | `openDrawer`, `closeModal`, `setActivePanel` |

## Permission Names
- Use permission codes from backend permission catalog.
- Do not invent frontend-only permission aliases for sensitive actions.
- Feature keys must match platform feature catalog.
- Use constants to avoid typo risk.

## Permission Constant Example
```ts
export const POS_PERMISSIONS = {
  CREATE_SALE: "pos.sale.create",
  VOID_SALE: "pos.sale.void",
  PRICE_OVERRIDE: "pos.price.override",
} as const;
```

## Route Naming
| Route | Purpose |
|---|---|
| `/platform/tenants` | Super Admin tenant management |
| `/tenant/catalog/products` | Tenant catalog management |
| `/tenant/access/roles` | Tenant role configuration |
| `/pos` | POS terminal checkout |
| `/pos/till/open` | Till open |
| `/pos/payment` | Payment workflow |

## Naming Diagram
```mermaid
flowchart TD
  Domain[Business Domain] --> Feature[Feature Folder Name]
  Feature --> Hook[useDomainActionQuery]
  Feature --> Component[DomainPanel]
  Feature --> Store[domain.store.ts when shared]
  Feature --> Route[/tenant/domain/action]
```

## Avoid Names
- Avoid `data`, `item`, `thing`, `managerStuff`, `adminAction` in public interfaces.
- Avoid role-fixed names like `CashierOnlyButton` when permission-based access is required.
- Avoid tenant-specific business names in reusable components.
- Avoid `common` folders that become dumping grounds.

## Related Documents

- [[frontend-coding-standards]]
- [[feature-access-ui-rules]]
- [[frontend-folder-structure]]

-