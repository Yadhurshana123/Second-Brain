---
title: Required Schema Extensions
folder: 03-data
document_type: schema-governance
status: architecture-aligned-draft
source_of_truth:
  - Unified Commerce Scope V1
  - Unified Commerce Database Design V4
  - Frontend Architecture V1
  - Backend Architecture Final
access_model: tenant-configurable-rbac-and-feature-permissions
last_updated: 2026-05-10
---
# Required Schema Extensions

## Purpose

This file controls how schema changes should be proposed. It is not permission to add random tables.

## Current position

The approved database design is already broad and includes requested areas such as wishlist, reviews, loyalty, OTP history, attribute templates, attribute presets, and typed offline sale/payment queues.

## Extension rules

| Rule | Required behavior |
|---|---|
| Source check | Confirm the need is not already covered by an approved table. |
| Scope check | Confirm the feature is inside the large Unified Commerce scope. |
| Tenant check | Define whether the table is platform-owned or tenant-owned. |
| Access check | Define feature, role, permission, and audit behavior. |
| Migration check | Include indexes, constraints, status values, and FK ownership. |

## Explicitly blocked without approval

| Table type | Reason |
|---|---|
| Generic cache tables | Conflicts with source-of-truth rules. |
| Duplicate stock tables | Inventory is controlled by balances and movement ledger. |
| Duplicate payment ledgers | Payments and payment_transactions already exist. |
| Unscoped tenant data | Violates tenant isolation. |

## Related documents

- [[schema-principles]]
- [[data-dictionary-index]]
