# Backend — development-only Super Admin seed

**Date:** 2026-05-11  
**Purpose:** Local **Development** environment only: ensure one **platform** user exists for `POST /api/v1/platform/auth/login` testing without touching tenant/staff/RBAC tables.

---

## Summary

| Item | Detail |
| --- | --- |
| **Table** | `platform_users` |
| **Login endpoint** | `POST /api/v1/platform/auth/login` |
| **Environment** | Runs **only** when host app environment is **Development** (or equivalent gate in code). |
| **Enable flag** | Explicit **enabled** flag required (no accidental seed in other environments). |
| **Credentials source** | Local **configuration**, **user-secrets**, or **environment variables**—never committed plaintext. |
| **Password handling** | Stored as **BCrypt** hash; plaintext never logged. |
| **Hash in logs** | Password **hash** must not appear in logs. |
| **Scope** | **Does not** modify tenant, staff, or RBAC tables—platform user row only. |

---

## Configuration keys (names only)

Use **User Secrets** or environment variables in local dev. Example placeholders—**do not** put real values in Git or Obsidian:

| Key | Example placeholder |
| --- | --- |
| Seed toggle | `DEV_SUPER_ADMIN_SEED_ENABLED` → `true` (local only) |
| Email | `DEV_SUPER_ADMIN_EMAIL` → `<PLATFORM_ADMIN_EMAIL>` |
| Password | `DEV_SUPER_ADMIN_PASSWORD` → `<LOCAL_DEV_PASSWORD>` |
| Display name | `DEV_SUPER_ADMIN_FULL_NAME` → e.g. `Local Platform Admin` |

**Connection string** (app configuration, not in this doc):

- `ConnectionStrings:DefaultConnection=<CONNECTION_STRING>`

**Database password** (if embedded in connection string): use `<POSTGRES_PASSWORD>` in docs; real value only in local secrets.

---

## PostgreSQL notes

| Item | Value |
| --- | --- |
| **Database** (typical local name) | `UnifiedCommerceDb` |
| **Table** | `platform_users` |
| **Expected status** | `active` for a usable login row |

Do **not** document or paste real DB passwords, connection strings, or JWT signing keys in Second Brain.

---

## Verification SQL (no secrets, no hash)

Run against your local DB after seeding; replace email placeholder:

```sql
SELECT "Id", email, full_name, status, created_at, updated_at
FROM platform_users
WHERE lower(email) = lower('<PLATFORM_ADMIN_EMAIL>');
```

---

## Security checklist (seed feature)

- [ ] Seed runs **only** in Development (or stricter internal profile).
- [ ] Disabled by default in shared `appsettings` if applicable; enabled only via secrets/env locally.
- [ ] No plaintext password in logs, analytics, or exception messages.
- [ ] No password hash in logs.
- [ ] JWT signing key remains `<JWT_SECRET>` in user-secrets—not in repo or docs.
