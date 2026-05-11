# Issues and fixes — 2026-05-11

**Workstream:** Super Admin shell & platform integration

---

## Git / delivery

| Issue | Impact | Fix / outcome |
| --- | --- | --- |
| **Non–fast-forward push** on backend `main` | Local `main` was behind `origin/main`; push rejected. | **`git pull origin main --rebase`**, resolved **add/add conflict** in `.gitignore` by merging remote and local ignore rules into one file. |
| **Editor during rebase** | `git rebase --continue` failed in non-interactive shell (`EDITOR` unset). | After staging resolved files, **`git commit --no-edit`** then **`git rebase --continue`** completed the rebase. |
| **Interrupted push** | Prior push attempt did not finish. | Subsequent **`git push origin main`** completed successfully (`180dec9` → `5943d81` on remote). |

*No repository URLs, tokens, or credentials are recorded here.*

---

## Product / UI

| Issue | Impact | Fix direction |
| --- | --- | --- |
| **Raw / default HTML styling** on some Super Admin screens | Inconsistent UX vs. production target. | Ensure global styles load on Super Admin routes; apply Tailwind/component classes; prefer shared UI primitives. |
| **Risk of fake dashboard data** | Violates “no fake production data” rule. | Bind widgets to real APIs; map failures to **Unavailable** / **empty** / **error** states per `implementation-summary.md`. |

---

## Frontend / backend boundary

| Issue | Impact | Fix direction |
| --- | --- | --- |
| **Backend verification ongoing** | Some platform endpoints may not match all UI assumptions yet. | Continue contract tests or manual checks; adjust UI empty/error copy to match actual HTTP status and payload shapes. |

---

## Secrets hygiene (process)

| Rule | Status |
| --- | --- |
| Do not commit `.env` or local overrides with real secrets | Frontend `.gitignore` updated in related work so local env files are not committed (see team repo history). |
| Seed password only in user-secrets / env | Documented in `backend-dev-seed.md` with placeholders only. |

If a real secret was ever pasted into a file by mistake, **rotate** the secret and purge history per org policy—not documented here.
