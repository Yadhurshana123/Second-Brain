---
title: Data Dictionary Index
folder: 03-data
document_type: data-index
status: architecture-aligned-draft
source_of_truth:
  - Unified Commerce Scope V1
  - Unified Commerce Database Design V4
  - Frontend Architecture V1
  - Backend Architecture Final
access_model: tenant-configurable-rbac-and-feature-permissions
last_updated: 2026-05-10
---

# Data Dictionary Index

## Purpose

This index lists all approved database tables by documentation file. Column-level PK/FK definitions are in the entity files.

## Complete table inventory

| Entity file | Tables |
|---|---|
| [[entities/platform-tenant-entities]] | `platform_users`, `tenants`, `outlets`, `outlet_addresses`, `document_sequences` |
| [[entities/identity-access-entities]] | `users`, `roles`, `permissions`, `role_permissions`, `tenant_user_roles`, `outlet_user_roles`, `platform_features`, `tenant_feature_entitlements`, `role_feature_assignments` |
| [[entities/configuration-entities]] | `feature_flags`, `tenant_settings`, `ui_themes` |
| [[entities/catalog-entities]] | `brands`, `suppliers`, `supplier_addresses`, `categories`, `return_policies`, `products`, `product_variants`, `product_attributes`, `attribute_values`, `attribute_templates`, `attribute_template_values`, `attribute_presets`, `attribute_preset_items`, `category_attributes`, `variant_attribute_values`, `product_suppliers`, `product_images` |
| [[entities/pricing-tax-entities]] | `tax_classes`, `tax_rates`, `tax_class_rates`, `price_lists`, `price_list_items` |
| [[entities/inventory-entities]] | `inventory_balances`, `inventory_channel_allocations`, `stock_movement_types`, `stock_movements`, `stock_reservations`, `purchase_receipts`, `purchase_receipt_lines`, `stock_adjustments`, `stock_adjustment_lines`, `stock_transfers`, `stock_transfer_lines`, `stocktakes`, `stocktake_lines` |
| [[entities/pos-device-sales-entities]] | `tills`, `pos_devices`, `till_sessions`, `cash_movement_types`, `cash_movements`, `cash_count_denominations`, `sales`, `sale_lines` |
| [[entities/customer-ecommerce-entities]] | `customers`, `customer_auth_accounts`, `customer_auth_identities`, `otp_channels`, `otp_purposes`, `otp_verifications`, `customer_addresses`, `wishlists`, `wishlist_items`, `product_reviews`, `loyalty_programs`, `membership_tiers`, `customer_memberships`, `loyalty_transactions`, `carts`, `cart_items`, `orders`, `order_items`, `order_addresses`, `order_status_transitions`, `order_status_history` |
| [[entities/fulfillment-entities]] | `delivery_methods`, `delivery_zones`, `delivery_zone_rates`, `deliveries`, `delivery_items`, `delivery_tracking` |
| [[entities/payments-entities]] | `payment_method_types`, `tenant_payment_methods`, `payment_provider_configs`, `payments`, `payment_transactions`, `sale_payment_allocations`, `order_payment_allocations`, `refunds` |
| [[entities/discounts-coupons-entities]] | `discount_types`, `discount_scopes`, `discount_policies`, `discount_requests`, `coupons`, `discount_applications`, `coupon_redemptions` |
| [[entities/returns-exchanges-entities]] | `return_reason_codes`, `returns`, `return_lines`, `return_refund_allocations`, `exchanges`, `exchange_lines`, `exchange_payment_allocations`, `exchange_refund_allocations` |
| [[entities/receipts-audit-offline-entities]] | `receipt_templates`, `receipts`, `receipt_print_logs`, `audit_logs`, `offline_sync_batches`, `offline_sync_items`, `offline_sale_sync_queue`, `offline_payment_sync_queue`, `offline_sync_conflicts`, `offline_sync_audit_logs` |
| [[entities/reporting-entities]] | `daily_sales_summaries`, `daily_payment_summaries`, `daily_inventory_summaries`, `daily_discount_return_summaries` |
| [[entities/data-import-ai-entities]] | No dedicated source-of-truth table; writes into approved target entities. |

## Module count summary

| Entity file | Table count |
|---|---:|
| `platform-tenant-entities.md` | 5 |
| `identity-access-entities.md` | 9 |
| `configuration-entities.md` | 3 |
| `catalog-entities.md` | 17 |
| `pricing-tax-entities.md` | 5 |
| `inventory-entities.md` | 13 |
| `pos-device-sales-entities.md` | 8 |
| `customer-ecommerce-entities.md` | 21 |
| `fulfillment-entities.md` | 6 |
| `payments-entities.md` | 8 |
| `discounts-coupons-entities.md` | 7 |
| `returns-exchanges-entities.md` | 8 |
| `receipts-audit-offline-entities.md` | 10 |
| `reporting-entities.md` | 4 |
| `data-import-ai-entities.md` | 0 |

## Access model reminder

Every tenant-level table operation must validate tenant context, feature entitlement, runtime flag where applicable, role permission, and user-right configuration. Platform-owned catalog tables are controlled by platform-admin responsibilities.

## Related documents

- [[database-overview]]
- [[entities/README]]
- [[tenant-consistency-rules]]