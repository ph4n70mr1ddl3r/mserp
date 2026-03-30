# Global Order Promising (GOP)

ML-powered order promising engine that calculates delivery dates based on inventory availability, production capacity, supplier lead times, and transportation times, providing real-time promising with confidence scores, managed by the Commerce Service.

> **Ownership Note**: Global Order Promising is owned by Commerce Service. Manufacturing Service provides capacity and production data via events (`manufacturing.work-order.*`, `manufacturing.plan.*`). Finance Service provides procurement lead time data via events (`finance.procurement.*`). Transportation and logistics data is sourced from Commerce Service internal modules. Commerce owns all GOP events under the `commerce.gop.*` namespace.

## Overview

Global Order Promising (GOP) provides real-time, accurate delivery date calculations by orchestrating data across inventory positions, production schedules, supplier lead times, and transportation networks. The engine uses ML models to refine lead time predictions based on historical performance, seasonal patterns, and supply chain variability. It supports multiple promising modes — Available-to-Promise (ATP), Capable-to-Promise (CTP), and Profitable-to-Promise (PTP) — with confidence scoring that reflects the reliability of each promise. GOP enables dynamic sourcing rule evaluation to determine the optimal fulfillment source from a global network of warehouses, plants, and suppliers, considering cost, speed, and service level objectives.

## Data Flow Diagram

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  Sales Order     │────▶│  GOP Engine      │────▶│  Promise Calculator │
│  / Quote Request │     │  ┌─────────────┐ │     │  ┌───────────────┐  │
└──────────────────┘     │  │ Sourcing    │ │     │  │ ATP / CTP /   │  │
                         │  │ Rule Eval   │ │     │  │ PTP Logic     │  │
┌──────────────────┐     │  └─────────────┘ │     │  └───────┬───────┘  │
│  Inventory ATP   │────▶│  ┌─────────────┐ │     └──────────┼──────────┘
│  (Commerce)      │     │  │ Lead Time   │ │                │
└──────────────────┘     │  │ Matrix      │ │     ┌──────────▼──────────┐
                         │  └─────────────┘ │     │  ML Confidence      │
┌──────────────────┐     └──────────────────┘     │  Scoring            │
│  Manufacturing   │────▶│                         │  ┌───────────────┐  │
│  Capacity        │     │  ┌─────────────────┐    │  │ Historical   │  │
└──────────────────┘     │  │ Calendar Engine │────│  │ Pattern      │  │
                         │  │ (working days,  │    │  │ Analysis     │  │
┌──────────────────┐     │  │  holidays)      │    │  └───────────────┘  │
│  Supplier Lead   │────▶│  └─────────────────┘    └──────────┬──────────┘
│  Times (Finance) │     └──────────────────┘                 │
└──────────────────┘                                           │
                         ┌──────────────────┐       ┌─────────▼──────────┐
                         │  Transportation  │────▶  │  Promise Result     │
                         │  (Commerce)      │       │  (date, source,     │
                         └──────────────────┘       │   confidence, lines)│
                                                    └────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Sourcing Rule Engine** | Commerce Service (Rust) | Priority-ordered rule evaluation across warehouses, plants, and suppliers considering proximity, cost, capacity, and SLA |
| **ATP Calculator** | Commerce Service | Available-to-Promise logic: checks current inventory + scheduled receipts against demand, netting with safety stock |
| **CTP Calculator** | Commerce Service | Capable-to-Promise logic: factors in production capacity and planned manufacturing orders to promise against future supply |
| **PTP Calculator** | Commerce Service | Profitable-to-Promise logic: evaluates margin impact of each sourcing option, rejects unprofitable promises |
| **ML Confidence Scoring** | Commerce Service + Report Service | ML model (Report Service) scores promise reliability based on historical accuracy, supplier performance, seasonal patterns |
| **Lead Time Matrix** | Commerce Service | Configurable lead time tables by source-destination pair, shipping method, and product category with ML-adjusted overrides |
| **Calendar Engine** | Commerce Service | Working day and holiday calendar per location, region, and carrier for accurate date calculation |
| **Storage** | PostgreSQL + Redis | Sourcing rules, promise results, and configuration in PostgreSQL; inventory and capacity cache in Redis for sub-50ms response |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Sourcing Rule Evaluator | Evaluate and rank fulfillment sources by priority, cost, speed | Commerce Service |
| ATP Calculator | Calculate promise dates from available inventory | Commerce Service |
| CTP Calculator | Calculate promise dates considering production capacity | Commerce Service |
| PTP Calculator | Evaluate profitability of each sourcing option | Commerce Service |
| Lead Time Engine | Aggregate and apply lead times from transport, production, and procurement | Commerce Service |
| Calendar Service | Working day and holiday calendar management per location | Commerce Service |
| ML Confidence Scorer | Generate confidence scores for promises using historical data | Report Service (model) + Commerce Service (invocation) |
| Promise Cache | Cache recent promise results for fast repeat queries | Commerce Service |

## Promise Modes

| Mode | Data Sources | Output | Use Case |
|------|-------------|--------|----------|
| **ATP** (Available-to-Promise) | On-hand inventory, scheduled receipts, allocations, safety stock | Earliest date from available stock | Standard fulfillment from warehouses |
| **CTP** (Capable-to-Promise) | ATP data + production capacity, BOM, production lead times | Earliest date considering manufacturing | Make-to-order and configured products |
| **PTP** (Profitable-to-Promise) | CTP data + cost, margin, price lists, discounts | Promise date with margin validation | High-value or low-margin orders |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Sales Orders | Commerce Service | Internal API | Order and quote lines request promise dates |
| Inventory ATP | Commerce Service | Internal API | Real-time inventory availability and reservations |
| Warehouse Management | Commerce Service | Internal API | Warehouse capacity and processing times |
| Manufacturing Capacity | Manufacturing Service | Event-driven | Production capacity, work order schedules, BOM lead times |
| Procurement Lead Times | Finance Service | Event-driven | Supplier lead times, PO schedules, and delivery performance |
| Transportation | Commerce Service | Internal API | Transit times, carrier schedules, shipping methods |
| Report Service (ML) | Report Service | gRPC | ML model inference for confidence scores and lead time prediction |
| Pricing Engine | Commerce Service | Internal API | Cost and margin data for PTP evaluation |
| Order Orchestration | Commerce Service | Event-driven | Promise result drives fulfillment routing |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `commerce.gop.promise.calculated` | `{promise_id, order_id, mode, lines[{item_id, source_id, source_type, promised_date, confidence}], calculated_at}` | Promise calculation completes | Sales Orders, Order Orchestration, Notification |
| `commerce.gop.promise.broken` | `{promise_id, order_id, original_date, revised_date, reason, affected_lines[]}` | Previously promised date cannot be met | Order Orchestration, Notification, Report Service |
| `commerce.gop.promise.revised` | `{promise_id, order_id, old_date, new_date, reason, confidence_delta}` | Promise date is recalculated with new data | Sales Orders, Notification |
| `commerce.gop.sourcing.evaluated` | `{evaluation_id, order_id, rule_id, candidates_ranked[], selected_source, selection_reason}` | Sourcing rule evaluation completes | Report Service, Analytics |
| `commerce.gop.lead-time.updated` | `{matrix_id, source_id, destination_id, method, old_lead_time, new_lead_time, updated_by}` | Lead time matrix entry is modified | GOP Engine (cache refresh), Report Service |
| `commerce.gop.capacity-snapshot.created` | `{snapshot_id, period, sources[{id, type, available, committed, remaining}]}` | Periodic capacity snapshot for reporting | Report Service, Planning Dashboard |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `commerce.order.submitted` | Commerce Service | Calculate promise for order lines |
| `commerce.inventory.atp-updated` | Commerce Service | Invalidate promise cache, re-evaluate affected promises |
| `manufacturing.capacity.updated` | Manufacturing Service | Update capacity view, re-evaluate CTP promises |
| `manufacturing.work-order.completed` | Manufacturing Service | Adjust capacity view, trigger ATP recalculation |
| `finance.procurement.po-confirmed` | Finance Service | Update supplier lead time data, adjust scheduled receipts |
| `finance.procurement.lead-time-adjusted` | Finance Service | Update lead time matrix with supplier performance data |
| `commerce.logistics.transit-time-updated` | Commerce Service | Update transportation lead times |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `gop_sourcing_rules` | `id, tenant_id, name, priority, conditions{item_category, region, channel, customer_group}, actions{source_preference, split_allowed, max_sources}, is_active` | Evaluated by sourcing engine |
| `gop_promise_results` | `id, tenant_id, order_id, quote_id, mode, status, lines{item_id, source_id, source_type, promised_date, confidence_score, lead_time_breakdown}, expires_at` | References order/quote, links to `gop_sourcing_rules` |
| `gop_capacity_view` | `id, tenant_id, source_id, source_type, period_start, period_end, total_capacity, committed_capacity, available_capacity, unit, updated_at` | References source (warehouse/plant/supplier) |
| `gop_lead_time_matrix` | `id, tenant_id, source_id, destination_id, shipping_method, item_category, base_lead_time_hours, ml_adjusted_hours, confidence, effective_from, effective_to` | References source and destination locations |
| `gop_calendar` | `id, tenant_id, location_id, date, day_type, working_hours, holiday_name, region` | Belongs to location |
| `gop_promise_audit` | `id, tenant_id, promise_id, event_type, old_values, new_values, reason, changed_at` | Audit trail for promise changes |

## Business Rules

### Sourcing Rule Priority

1. Rules are evaluated in strict `priority` order (lowest number = highest priority).
2. First matching rule's actions determine the source selection strategy.
3. If no rule matches, default to the closest warehouse with available inventory.
4. Rules may specify `split_allowed: true` to permit multi-source fulfillment for a single line.
5. `max_sources` caps the number of fulfillment sources per promise (default: 3).
6. Rules support time-based activation (`effective_from` / `effective_to`).

### Lead Time Calculation

1. Total lead time = sourcing lead time + production lead time (CTP only) + transportation lead time + handling time.
2. Each component is looked up from `gop_lead_time_matrix` with ML-adjusted overrides from Report Service.
3. Calendar adjustments: skip non-working days and holidays from `gop_calendar`.
4. Supplier lead times use the 75th percentile of historical delivery performance (configurable percentile).
5. Lead times are cached in Redis with a TTL of 5 minutes; forced refresh on inventory or capacity change events.

### Confidence Scoring

1. Confidence score: 0.0–1.0 scale derived from ML model (Report Service).
2. Model inputs: supplier on-time delivery rate, inventory accuracy, production schedule adherence, seasonal variability, historical promise accuracy.
3. Confidence bands: **High** (≥ 0.85), **Medium** (0.60–0.84), **Low** (< 0.60).
4. Promises below the configured `min_confidence_threshold` are flagged for manual review.
5. Confidence scores are recalculated when any input data changes significantly (> 10% delta).

### Promise Breaking

1. A promise is broken when a previously committed date is no longer achievable.
2. On break: publish `commerce.gop.promise.broken`, notify customer, and attempt to find an alternative source.
3. If an alternative source is found, a revised promise is created automatically.
4. If no alternative exists within the acceptable window, escalate to manual review.
5. Promise break rate is tracked per source and feeds back into confidence scoring.

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `commerce.gop.default_promise_mode` | `atp` | Default promising mode when not specified (atp, ctp, ptp) |
| `commerce.gop.sourcing.max_sources_per_promise` | 3 | Maximum fulfillment sources per promise result |
| `commerce.gop.confidence.min_threshold` | 0.60 | Minimum confidence score before flagging for review |
| `commerce.gop.lead_time.cache_ttl_seconds` | 300 | Redis cache TTL for lead time data |
| `commerce.gop.lead_time.historical_percentile` | 75 | Percentile of historical supplier performance used for lead time |
| `commerce.gop.calendar.default_working_hours` | 8 | Default working hours per day for lead time calculation |

## Cross-References

- [Order Orchestration](order-orchestration.md) — Promise results drive fulfillment routing
- [B2C Commerce](b2c-commerce.md) — Storefront displays promise dates on product pages
- [Manufacturing Service](../06-services/manufacturing.md) — Production capacity data for CTP
- [Finance Service](../06-services/finance.md) — Supplier lead times and procurement data
- [SPEC.md §7.2](../SPEC.md) — Commerce → Finance handoff boundary
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
- [Commerce Service](../06-services/commerce.md) — Commerce service specification
- [NFR: Performance](../10-planning/nfr.md) — Promise calculation < 200ms p99
