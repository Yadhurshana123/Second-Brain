---
title: Feature Unit Test Authoring Guide
document_type: ai-ide-rule
folder: 14-ai-ide-rules
purpose: Guide Cursor or AI IDE to write unit tests after implementing any backend or frontend feature
source_of_truth:
  - 01-product
  - 02-architecture
  - 03-data
  - 04-api
  - 05-backend
  - 06-frontend
  - 07-modules
  - 08-user-flows
  - 09-security-and-compliance
  - 12-templates
status: active
---
# Backend Feature Completion Testing Rule

  

## Purpose

  

This rule is **mandatory** for AI IDE tools and developers implementing backend features for the Unified Commerce E-POS + E-Commerce SaaS system.

  

When a backend feature’s implementation for the current task is **complete** (services, repositories, domain rules, API controllers — per task scope), the **same delivery must include** automated tests in **`POS.Tests`** for that feature. Do **not** stop after production code only and leave tests for “later,” unless the user **explicitly** asks for code-only delivery.

  

## Core references

  

| Area | Document / path |

|------|------------------|

| Test project layout & commands | Repository `Pos-BackEnd/POS.Tests/README.md` |

| Testing strategy | [[05-backend/testing-strategy]] (vault) or `Pos-BackEnd/docs/testing-strategy.md` (repo) |

| Backend folder structure | [[05-backend/backend-folder-structure]] |

| DTO rules | [[05-backend/dto-handling]] |

| Feature QA | `07-modules/.../features/.../feature-spec.md` checklist |

  

## When this rule applies

  

- After any backend change that adds or significantly changes: Application services, validators, Infrastructure repositories, Domain behavior, or API controllers **for a feature**.

  

## Required sequence (AI IDE)

  

1. Complete the feature code for the agreed scope.

2. **Immediately** add or update tests under `Pos-BackEnd/POS.Tests/` in the layer folder that matches the change:

   - **`Domain/`** — pure entity / value rules (no EF, no HTTP).

   - **`Application/`** — service orchestration (mock repositories; use **`FakeUnitOfWork`** from `Common/TestDoubles`).

   - **`Infrastructure/`** — repository + EF behavior (use **`TestDbContextFactory`** — SQLite in-memory).

   - **`Api/`** — HTTP contract (**`PosWebApplicationFactory`**).

3. **Mirror** the production module path (e.g. `Application/Modules/PlatformUsers/Services/PlatformAuthServiceTests.cs`).

4. Map the feature **`feature-spec.md` QA checklist** to concrete test methods using **`Method_Scenario_Expected`** naming.

5. Run **`dotnet test`** (full solution or filter by `FullyQualifiedName~POS.Tests.<Layer>`); fix failures before considering the task done.

6. Update **`feature-history.md`** for that feature when tests are added or materially changed.

  

## Minimum expectations by change type

  

| You changed… | You must add at least… |

|--------------|-------------------------|

| Application service | Application tests: happy path + primary failure paths (validation, auth, tenant, status, not-found). |

| Repository method with non-trivial SQL/filters | Infrastructure test using `TestDbContextFactory` when tenant/filter correctness matters. |

| Public API endpoint | At least one API smoke test (`Api/`). |

| Non-trivial domain invariants | `Domain/` tests. |

  

Trivial edits (e.g. rename with no behavior change) may only update affected tests; do **not** skip tests when behavior changed.

  

## Naming

  

| Item | Pattern |

|------|---------|

| Test class | `<SubjectUnderTest>Tests` |

| Test method | `Method_Scenario_Expected` (e.g. `LoginAsync_WhenPasswordWrong_ReturnsUnauthorized`) |

  

## Non-negotiable

  

- Tests live in **`POS.Tests`** only — not inside `POS.API`, `POS.Application`, etc.

- No real PostgreSQL or production secrets in automated tests; use shared helpers and SQLite for EF tests.

- Backend assertions must enforce tenant isolation, RBAC, and validation — not UI-only assumptions.

  

## Related rules

  

- [[14-ai-ide-rules/backend-implement]]

- [[14-ai-ide-rules/fullstack-feature-implementation-rule]]

- [[05-backend/testing-strategy]]
