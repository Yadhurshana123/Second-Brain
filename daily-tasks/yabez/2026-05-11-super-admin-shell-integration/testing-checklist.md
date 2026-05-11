# Testing checklist — Super Admin shell & integration

**Date:** 2026-05-11

Use **local** credentials only; never paste real secrets into tickets or chat logs.

---

## Platform auth

- [ ] `POST /api/v1/platform/auth/login` returns success with valid seeded platform user (local only).
- [ ] Invalid email/password returns controlled error; UI shows safe message (no stack traces to end users).
- [ ] Valid login persists session per existing frontend pattern (reload, new tab behavior as designed).

## Routing & guards

- [ ] Navigating to `/super-admin/dashboard` **without** session redirects to `/super-admin/login` (or shows honest unauthenticated state per spec).
- [ ] After login, user reaches dashboard without being dropped on **Staff** `/login`.
- [ ] **403** from platform APIs shows forbidden / not authorized UI, not tenant POS.

## Layout & navigation

- [ ] Super Admin sidebar/topbar render; links go to dashboard, tenants, audit-logs as implemented.
- [ ] Global `src/components/Sidebar.tsx` is **unchanged** for Super Admin routes (regression: staff app still works).

## Dashboard & data honesty

- [ ] With **backend up**: widgets show **API-driven** data or documented empty states.
- [ ] With **tenant API down**: tenants section shows **empty/error**, not mock tenant names.
- [ ] With **revenue API down**: copy shows **Unavailable** (or agreed equivalent).
- [ ] With **activity API down**: **No recent tenant activity** (or empty list).
- [ ] With **support/attention API down**: **Unavailable** or **No platform attention items available**.

## Tenant management (in progress)

- [ ] Page layout aligns with reference where implemented.
- [ ] No hardcoded “fake Fortune 500” style tenant rows.

## Tenant / audit logs UI

- [ ] Audit / activity view does not fabricate production-like events when API is missing.
- [ ] Loading and error states use shared **LoadingState** / **ErrorState** components where applicable.

## Shared UI

- [ ] Button, Input, Table, Modal, Card, Badge behave consistently (focus, disabled, loading).
- [ ] EmptyState used where lists can be legitimately empty.

## Styling

- [ ] Super Admin pages pick up global/Tailwind styles (no raw unstyled form disaster on primary flows).

## Backend seed (local dev only)

- [ ] Seed runs only when Development + enabled flag.
- [ ] Row appears in `platform_users` with `status` = `active` (verify with placeholder SQL in `backend-dev-seed.md`).
- [ ] Logs contain **no** password or hash.

## Regression — staff app

- [ ] Staff login at `/login` (if kept) still works for tenant flows.
- [ ] No accidental sharing of platform token with tenant API clients unless explicitly architected (should not be).
