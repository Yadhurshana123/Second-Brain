# Next steps — Super Admin shell & integration

**Date:** 2026-05-11  
**Owner:** Yabez

---

## Immediate

1. **Finish Tenant Management UI** — align with reference layout using **only** real API data or explicit empty/unavailable states; remove any placeholder rows that resemble production tenants.
2. **Backend verification** — confirm all platform dashboard and tenant list contracts match frontend assumptions (status codes, pagination, error bodies).
3. **Styling pass** — close the gap on Super Admin pages that still look like unstyled HTML (global CSS / Tailwind coverage audit).

## Short term

- Add or extend **automated tests** for platform login and `PlatformSessionGuard` behavior (unit + light e2e if pipeline supports it).
- Document **OpenAPI** or internal API sheet for platform endpoints alongside this worklog when stable.
- Decide fate of **root `package-lock.json`** in backend repo (ignore, delete, or commit intentionally)—keep repo root clean per team convention.

## Medium term

- **SSO / MFA** for platform admins (if in roadmap)—keep Staff and Platform auth separation in design docs.
- **Audit log backend** — wire Tenant Activity Logs UI to real persistence and retention policy.
- **Observability** — platform-side metrics and alerts without fabricating UI demo data.

## Branching / release

- Merge **`super-admin-shell-integration`** into the agreed integration branch when review passes; keep **`main`** protected per team policy if adopted.
- Coordinate **staging** deploy checklist: seed **disabled**; use real secret management (`<JWT_SECRET>`, `<CONNECTION_STRING>`, etc. from vault only).

---

*This file is planning only—no code or schema changes implied until tasks are picked up in the IDE.*
