# Configure, Price, Quote (CPQ)

Product configuration, multi-level pricing, and quote-to-order workflow for complex sales, managed by the Commerce Service.

## Overview

CPQ enables sales teams to configure complex products with constraint-based validation, apply multi-level pricing with discount rules, volume tiers, and channel-specific rates, generate versioned quotes, and convert approved quotes to sales orders. The rules engine enforces product compatibility constraints and validates configurations in real time. Non-standard pricing triggers approval workflows via the Workflow Service. Integration with PIM provides product structure data, Inventory enables ATP checks during quoting, and CRM links quotes to opportunities.

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Configuration Engine** | Commerce Service (Rust) | Constraint-based product configuration with compatibility rules, exclusion rules, and feature dependencies. Validates configurations in real time |
| **Pricing Engine** | Commerce Service (Rust) | Multi-level pricing: base price, volume tiers, channel-specific pricing, promotional discounts, and margin guardrails. Supports currency conversion |
| **Quote Management** | Commerce Service (Rust) | Quote generation from configurations with line-item detail, versioning, PDF export, and customer-facing presentation |
| **Quote-to-Order** | Commerce Service (Rust) | One-click conversion of approved quotes to sales orders with inventory reservation and fulfillment triggering |
| **Approval Routing** | Commerce Service + Workflow Service | Non-standard pricing (below margin threshold) triggers approval workflow with configurable escalation |
| **ATP Integration** | Commerce Service (Internal) | Real-time available-to-promise check during quoting with expected ship date calculation |
| **Storage** | PostgreSQL + Redis | Configuration rules, pricing tables, and quote data in PostgreSQL; pricing cache and configuration session state in Redis |

## Business Rules

| Rule | Description |
|------|-------------|
| **BR-CPQ-001** | A configuration is valid only when all mandatory features are selected and no constraint violations exist (incompatibility, exclusion, dependency). |
| **BR-CPQ-002** | Pricing is evaluated in order: base price → volume discount → channel adjustment → promotional discount → manual override. Each layer adjusts the previous result. |
| **BR-CPQ-003** | Quotes below the minimum margin threshold (configurable per tenant) require approval via Workflow Service before customer presentation. |
| **BR-CPQ-004** | Each quote version is immutable. Modifications create a new version with incremented version number and audit trail. |
| **BR-CPQ-005** | ATP checks are performed at quote creation and again at quote-to-order conversion. If availability changed, the user must re-confirm. |
| **BR-CPQ-006** | Expired quotes (past valid-until date) cannot be converted to orders. The quote must be revised and re-presented. |

## Data Model

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `product_config_rules` | `id, tenant_id, product_id, rule_type, constraint_expression, priority, is_active, created_at, updated_at, version, is_deleted` | Belongs to product |
| `pricing_tiers` | `id, tenant_id, product_id, channel, currency, min_qty, max_qty, unit_price, discount_pct, valid_from, valid_to, created_at, updated_at, version, is_deleted` | Belongs to product |
| `quotes` | `id, tenant_id, quote_number, customer_id, opportunity_id, status, valid_until, total_amount, currency, margin_pct, version, created_at, updated_at, created_by, updated_by, is_deleted` | Has many `quote_lines`, belongs to customer/opportunity |
| `quote_lines` | `id, tenant_id, quote_id, product_id, configuration_id, quantity, unit_price, discount_pct, line_total, atp_qty, expected_ship_date, created_at, updated_at, version, is_deleted` | Belongs to quote, product, configuration |
| `configurations` | `id, tenant_id, product_id, selected_features{}, constraint_results{}, is_valid, created_at, updated_at, version, is_deleted` | Has many quote_lines |
| `quote_approvals` | `id, tenant_id, quote_id, approver_id, status, reason, requested_at, decided_at, created_at, updated_at, version, is_deleted` | Belongs to quote |

## Events

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `commerce.quote.created` | `{quote_id, quote_number, customer_id, opportunity_id, total, currency}` | New quote generated | CRM, Report Service |
| `commerce.quote.version-added` | `{quote_id, version, changes_summary, total, currency}` | Quote revision created | CRM, Notification |
| `commerce.quote.submitted-for-approval` | `{quote_id, version, margin_pct, threshold, approver_id}` | Non-standard pricing detected | Workflow Service |
| `commerce.quote.approved` | `{quote_id, version, approved_by, margin_pct}` | Approval granted | Notification |
| `commerce.quote.converted-to-order` | `{quote_id, quote_version, order_id}` | Quote accepted and order created | Order Orchestration, Inventory, CRM |
| `commerce.quote.expired` | `{quote_id, expired_at}` | Quote valid-until date passed | Notification, CRM |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `commerce.product.updated` | Commerce Service | Refresh product configuration rules and pricing |
| `commerce.inventory.stock-changed` | Commerce Service | Update ATP data for open quotes |
| `crm.opportunity.updated` | CRM Service | Link quote to opportunity stage changes |
| `workflow.step.approved` | Workflow Service | Finalize quote approval status |
| `config.changed` | Config Service | Reload pricing thresholds and margin rules |

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/v1/cpq/configurations` | Create and validate a product configuration |
| GET | `/api/v1/cpq/configurations/{id}` | Retrieve configuration with constraint results |
| POST | `/api/v1/cpq/configurations/{id}/price` | Calculate price for a configuration |
| POST | `/api/v1/cpq/quotes` | Generate quote from configuration |
| GET | `/api/v1/cpq/quotes` | List quotes with cursor pagination |
| GET | `/api/v1/cpq/quotes/{id}` | Retrieve quote with lines and versions |
| POST | `/api/v1/cpq/quotes/{id}/versions` | Create a new quote version |
| POST | `/api/v1/cpq/quotes/{id}/submit-approval` | Submit quote for pricing approval |
| POST | `/api/v1/cpq/quotes/{id}/convert` | Convert approved quote to sales order |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| PIM / Catalog | Commerce Service | Internal API | Product structure, features, and compatibility data |
| Inventory (ATP) | Commerce Service | Internal API | Real-time availability and expected ship dates |
| CRM Opportunities | CRM Service | Event-driven | Opportunity linkage and stage synchronization |
| Approval Workflows | Workflow Service | Event-driven | Non-standard pricing approval routing |
| Pricing Configuration | Config Service | Event-driven | Margin thresholds, discount limits, channel rules |
| Notification Service | Platform Service | Event-driven | Quote status and approval notifications |
| Report Service | Report Service | Internal API | Quote conversion analytics and margin reporting |

## Cross-References

- [Order Orchestration](order-orchestration.md) — Quote-to-order conversion triggers fulfillment
- [B2C Commerce](b2c-commerce.md) — Shared pricing engine and promotions
- [Commerce Service](../06-services/commerce.md) — Commerce service specification
- [CRM Service](../06-services/crm.md) — Opportunity and customer data
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
