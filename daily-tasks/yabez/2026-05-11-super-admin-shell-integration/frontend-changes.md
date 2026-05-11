# Frontend changes — Super Admin shell & integration

**Date:** 2026-05-11

## Routes

| Path | Purpose |
| --- | --- |
| `/super-admin/login` | Platform Super Admin login (no Tenant ID). |
| `/super-admin/dashboard` | Platform dashboard (real data or honest empty/unavailable states). |
| `/super-admin/tenants` | Tenant management (in progress; match reference UI with real or empty data only). |
| `/super-admin/audit-logs` | Tenant / platform activity logs UI (no fake production rows). |
| `/login` | **Staff** login if intentionally retained—must stay separate from Super Admin. |

## Super Admin layout

- **`SuperAdminLayout`** (or equivalent): wraps Super Admin routes.
- **Sidebar**: platform navigation only inside Super Admin layout—not the global staff sidebar.
- **Topbar**: platform chrome (user menu, environment indicator as applicable).
- **Protected content area**: outlet for child routes; guarded by platform session rules.

## Guards and errors

| Concern | Handling |
| --- | --- |
| Session | `PlatformSessionGuard` (or equivalent) ensures platform token/session before protected routes. |
| Unauthorized | **401**: redirect to Super Admin login or refresh flow per app pattern. |
| Forbidden | **403**: show unauthorized / forbidden state; do not expose tenant or staff UI. |

## Platform auth

- **Client**: platform auth API module / service (centralized `POST` login, token storage, refresh if applicable).
- **Endpoint**: `POST /api/v1/platform/auth/login`
- **Behavior**: Follow existing frontend session pattern (storage keys, headers, interceptors)—documented here at behavior level only; no tokens or secrets in this vault.

## Shared UI library

Reusable components introduced or consolidated for team use:

- **Button**
- **Input**
- **Table**
- **Modal**
- **Card**
- **Badge**
- **EmptyState**
- **LoadingState**
- **ErrorState**

## Non-negotiable rule: staff `Sidebar.tsx`

- **Do not modify** `src/components/Sidebar.tsx` for Super Admin navigation.
- Super Admin navigation must live inside **`SuperAdminLayout`** or **`src/layouts/super-admin/`** (or equivalent dedicated tree).

## Styling / theming note

- **Issue observed:** Some areas showed **raw/default HTML** appearance (missing or inconsistent utility classes / global styles).
- **Fix direction:**
  - Confirm **global CSS** / **Tailwind** / **scoped CSS** is loaded for Super Admin entry routes.
  - Ensure Super Admin leaf components use the **same design tokens** and utility patterns as the rest of the app.
  - Prefer shared primitives from the UI library over unstyled native controls in platform pages.
