# Order Orchestration

Fulfillment orchestration engine with routing rules, split/merge logic, backorder management, and wave planning for multi-source order fulfillment, managed by the Commerce Service.

## Overview

Order Orchestration provides the intelligent middleware between order capture and fulfillment execution. The module evaluates routing rules to determine optimal fulfillment sources, applies assignment logic to distribute work across warehouses and channels, and manages split and merge decisions for complex orders. It includes a backorder management queue with automated fulfillment triggers when inventory becomes available, and supports wave-based fulfillment planning for batch processing efficiency. The engine handles multi-source fulfillment scenarios where a single order may be partially shipped from multiple locations.

## Data Flow Diagram

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  Sales Order     │────▶│  Orchestration    │────▶│  Routing Engine     │
│  Created         │     │  Process Manager  │     │  ┌───────────────┐  │
└──────────────────┘     │  ┌─────────────┐ │     │  │ Rule Evaluator│  │
                         │  │ Process     │ │     │  │ (priority     │  │
┌──────────────────┐     │  │ State       │ │     │  │  ordered)     │  │
│  Inventory ATP   │────▶│  │ Machine     │ │     │  └───────┬───────┘  │
│  Data            │     │  └─────────────┘ │     └──────────┼──────────┘
└──────────────────┘     └──────────────────┘                │
                                   │              ┌─────────▼──────────┐
                    ┌──────────────▼──────────────│  Split/Merge Engine │
                    │                             │  ┌───────────────┐  │
                    │                             │  │ Assignment    │  │
                    │                             │  │ Rules         │  │
                    │                             │  └───────────────┘  │
                    │                             └─────────┬──────────┘
                    │                                       │
           ┌────────▼──────┐  ┌──────────────┐  ┌─────────▼──────────┐
           │  Fulfillment  │  │  Backorder   │  │  Wave Planner      │
           │  Assignment   │  │  Queue       │  │  (batch scheduling)│
           └───────┬───────┘  └──────┬───────┘  └────────────────────┘
                   │                 │
                   ▼                 ▼
           ┌──────────────────────────────────────────────┐
           │  Warehouse / Logistics / Drop-Ship Execution  │
           └──────────────────────────────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Orchestration Engine** | Commerce Service (Rust) | State machine-driven process manager that orchestrates order lines through routing, assignment, and fulfillment stages |
| **Routing Rules** | Commerce Service | Priority-ordered rule evaluation considering source proximity, inventory availability, cost, and delivery SLA |
| **Assignment Engine** | Commerce Service | Distributes fulfillment tasks to warehouses, stores, or drop-ship suppliers based on assignment rules and capacity |
| **Split/Merge Logic** | Commerce Service | Automatic order splitting by fulfillment source with merge support for consolidated shipments |
| **Backorder Management** | Commerce Service | Priority queue with automated fulfillment triggers when inventory replenished, with expiration and cancellation rules |
| **Wave Planning** | Commerce Service + Workflow Service | Batch grouping of fulfillment assignments by criteria (carrier, zone, priority) for warehouse efficiency |
| **Multi-Source Fulfillment** | Commerce Service | Orchestration of partial shipments from multiple sources with consolidated tracking and customer communication |
| **Storage** | PostgreSQL | Orchestration process state, rules, assignments, and backorder data in Commerce database |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Orchestration Process Manager | State machine for order fulfillment lifecycle | Commerce Service |
| Routing Engine | Rule evaluation and fulfillment source selection | Commerce Service |
| Assignment Engine | Warehouse and supplier task distribution | Commerce Service |
| Split/Merge Processor | Order line split and shipment consolidation | Commerce Service |
| Backorder Queue Manager | Backorder prioritization, expiration, and auto-fulfill | Commerce Service |
| Wave Planner | Batch fulfillment scheduling and release | Commerce Service |
| Fulfillment Tracker | Multi-source shipment tracking and status aggregation | Commerce Service |
| Orchestration Analytics | Routing effectiveness, SLA adherence, cost analysis | Commerce Service |

## Orchestration Process States

| State | Description | Next States |
|-------|-------------|-------------|
| Pending | Order received, awaiting routing | Routing |
| Routing | Evaluating fulfillment source rules | Assigned, Split, Backordered |
| Assigned | Fulfillment source selected, awaiting execution | In Progress, Reassigned |
| Split | Multiple fulfillment sources required | Partially Fulfilled |
| In Progress | Fulfillment execution underway | Fulfilled, Partially Fulfilled, Exception |
| Partially Fulfilled | Some lines shipped, others pending | Fulfilled, Exception |
| Fulfilled | All lines shipped and confirmed | Completed |
| Backordered | Lines queued awaiting inventory | Routing (on replenishment) |
| Exception | Fulfillment failure requiring intervention | Assigned, Cancelled |
| Completed | All deliveries confirmed | Terminal |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Sales Orders | Commerce Service | Internal API | Order capture triggers orchestration |
| Inventory ATP/CTP | Commerce Service | Internal API | Real-time availability for routing decisions |
| Warehouse Management | Commerce Service | Event-driven | Fulfillment assignment execution |
| Logistics | Commerce Service | Event-driven | Shipment creation and carrier selection |
| Pricing Engine | Commerce Service | Internal API | Fulfillment cost calculation for routing |
| Payment Service | Commerce Service | Internal API | Payment capture and refund for split orders |
| Notification Service | Platform Service | Event-driven | Customer shipment notifications, backorder alerts |
| Report Service | Report Service | Internal API | Fulfillment performance dashboards |
| Workflow Engine | Workflow Service | Event-driven | Wave release approval workflows |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `commerce.orchestration.rule-evaluated` | `{process_id, order_id, rule_id, source_id, result}` | Routing rule evaluation completes | Orchestration Analytics, Dashboard |
| `commerce.orchestration.split` | `{process_id, order_id, split_count, assignment_ids}` | Order split across multiple sources | Notification Service, Fulfillment Tracker |
| `commerce.orchestration.merged` | `{process_id, order_id, merged_assignment_ids, tracking_id}` | Partial shipments consolidated | Notification Service, Sales Orders |
| `commerce.backorder.created` | `{backorder_id, order_id, line_items, priority, expected_date}` | Insufficient inventory for fulfillment | Notification Service, Inventory Planning |
| `commerce.backorder.fulfilled` | `{backorder_id, order_id, fulfilled_by, fulfilled_at}` | Backorder auto-fulfilled on replenishment | Notification Service, Report Service |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `commerce.order.placed` | Commerce Service | Start orchestration process |
| `commerce.order.cancelled` | Commerce Service | Cancel orchestration and release assignments |
| `commerce.inventory.atp-updated` | Commerce Service | Re-evaluate backorder queue for fulfillment |
| `commerce.inventory.stock-received` | Commerce Service | Trigger backorder auto-fulfill check |
| `commerce.warehouse.shipment-confirmed` | Commerce Service | Update assignment status, check for merge |
| `commerce.logistics.tracking-updated` | Commerce Service | Update multi-source tracking aggregation |
| `workflow.step.approved` | Workflow Service | Release wave for fulfillment |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `orchestration_rules` | `id, tenant_id, name, type, priority, conditions, actions, is_active` | Evaluated by routing engine |
| `orchestration_processes` | `id, tenant_id, order_id, status, current_state, started_at, completed_at` | Has many `orchestration_assignments` |
| `fulfillment_waves` | `id, tenant_id, wave_number, criteria, status, planned_release, released_at` | Has many `orchestration_assignments` |
| `backorder_queue` | `id, tenant_id, order_id, line_item_id, requested_qty, fulfilled_qty, priority, status, expires_at` | Belongs to `order`, links to `orchestration_processes` |
| `orchestration_assignments` | `id, tenant_id, process_id, wave_id, source_id, source_type, line_items, status, assigned_at` | Belongs to `orchestration_processes`, `fulfillment_waves` |

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `commerce.orchestration.max_splits_per_order` | 5 | Maximum fulfillment sources per order |
| `commerce.orchestration.routing_timeout_seconds` | 30 | Max time for routing rule evaluation |
| `commerce.orchestration.backorder.expiration_days` | 30 | Days before backorder auto-cancels |
| `commerce.orchestration.wave.default_size` | 100 | Default number of assignments per wave |
| `commerce.orchestration.wave.auto_release` | false | Auto-release waves without approval |
| `commerce.orchestration.merge_window_hours` | 48 | Window for consolidating split shipments |

## Cross-References

- [B2C Commerce](b2c-commerce.md) — Storefront orders trigger orchestration
- [Sales Orders](../06-services/commerce.md) — Order capture as orchestration input
- [Inventory ATP/CTP](../06-services/commerce.md) — Availability data for routing decisions
- [Warehouse Management](../06-services/commerce.md) — Fulfillment execution
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
- [Commerce Service](../06-services/commerce.md) — Commerce service specification
- [NFR: Performance](../10-planning/nfr.md) — Routing evaluation < 500ms per order
