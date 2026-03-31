# Advanced Warehouse Management (WMS)

Directed putaway, wave planning, voice picking, task interleaving, cartonization, zone management, cycle counting, cross-docking, and yard management, managed by the Commerce Service.

## Overview

Advanced Warehouse Management (WMS) extends the Commerce Service's warehouse operations with enterprise-grade capabilities for high-volume, multi-zone distribution centers. The module provides directed putaway logic that optimizes storage location selection based on item velocity, dimensions, and zone affinity. Wave planning groups orders into executable fulfillment batches by carrier, zone, priority, or custom criteria. Voice picking integration enables hands-free operator guidance, while task interleaving maximizes labor utilization by dynamically assigning the next optimal task (pick, replenish, putaway) based on operator location and equipment.

Cartonization algorithms determine optimal carton selection and item packing sequences to minimize shipping costs and material waste. Zone management supports logical and physical warehouse subdivisions with configurable picking strategies (discrete, batch, wave, zone-based). Cycle counting replaces disruptive physical inventories with continuous, probability-weighted count schedules. Cross-docking routes inbound shipments directly to outbound staging, bypassing storage. Yard management tracks trailers, dock door assignments, and carrier appointments across the facility perimeter.

## Data Flow Diagram

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  Sales Orders    │────▶│  Wave Planner    │────▶│  Pick Engine        │
│  (Orchestration) │     │  ┌─────────────┐ │     │  ┌───────────────┐  │
└──────────────────┘     │  │ Wave         │ │     │  │ Voice / RF    │  │
                         │  │ Templates    │ │     │  │ Directed Pick │  │
┌──────────────────┐     │  └─────────────┘ │     │  └───────┬───────┘  │
│  Inbound ASN     │────▶│  ┌─────────────┐ │     └──────────┼──────────┘
│  (Receiving)     │     │  │ Selection    │ │                │
└──────────────────┘     │  │ Criteria     │ │     ┌──────────▼──────────┐
                         │  └─────────────┘ │     │  Task Interleaving   │
┌──────────────────┐     └──────────────────┘     │  ┌───────────────┐  │
│  Inventory       │────▶│  Directed        │────▶│  │ Priority      │  │
│  Replenishment   │     │  Putaway Engine   │     │  │ Scoring Model │  │
└──────────────────┘     │  ┌─────────────┐ │     │  └───────────────┘  │
                         │  │ Zone/Aisle  │ │     └──────────┬──────────┘
┌──────────────────┐     │  │ Affinity    │ │                │
│  Finished Goods  │────▶│  │ Rules       │ │     ┌──────────▼──────────┐
│  (Manufacturing) │     │  └─────────────┘ │     │  Pack & Cartonize    │
└──────────────────┘     └──────────────────┘     │  ┌───────────────┐  │
                                                   │  │ 3D Bin Pack   │  │
┌──────────────────┐     ┌──────────────────┐     │  │ Algorithm     │  │
│  Cycle Count     │────▶│  Zone Manager    │────▶│  └───────┬───────┘  │
│  Schedule        │     │  ┌─────────────┐ │     └──────────┼──────────┘
└──────────────────┘     │  │ Yard & Dock  │ │                │
                         │  │ Door Cal     │ │     ┌──────────▼──────────┐
                         │  └─────────────┘ │     │  Ship & Carrier      │
                         └──────────────────┘     │  Handoff (→ TMS)     │
                                                   └─────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Directed Putaway Engine** | Commerce Service (Rust) | Rule-based location assignment considering item velocity class, dimensions, weight, storage requirements, and zone capacity. Supports ABC velocity analysis with automatic re-classification |
| **Wave Planning** | Commerce Service (Rust) | Configurable wave templates with selection criteria (carrier, zone, priority, SLA), automated wave release scheduling, and manual override capabilities |
| **Voice Picking Integration** | Commerce Service + Integration Service | Voice-directed task delivery via industrial voice terminals. Task commands converted to speech; operator confirmations parsed back. Supports multilingual operations |
| **Task Interleaving** | Commerce Service (Rust) | Real-time task optimization engine that considers operator location, equipment type, skill certification, and task urgency to assign the next highest-value task |
| **Cartonization Engine** | Commerce Service (Rust) | 3D bin-packing algorithm with carton catalog, item dimensions, weight limits, and carrier-specific rules. Outputs carton selection, item placement, and packing instructions |
| **Zone Management** | Commerce Service (Rust) | Logical zone definitions with picking strategies per zone (discrete, batch, wave), zone-to-zone transfer rules, and zone-based labor assignment |
| **Cycle Counting** | Commerce Service (Rust) | Probability-weighted count schedule generation based on ABC class, last count date, adjustment history, and transaction volume. Supports random, triggered, and full-count schedules |
| **Cross-Docking** | Commerce Service (Rust) | Pre-receipt matching of inbound ASN lines against open outbound orders. Automatic allocation and staging for eligible items, bypassing putaway |
| **Yard Management** | Commerce Service (Rust) | Trailer check-in/check-out, dock door scheduling, carrier appointment management, and yard map visualization. Integrates with TMS for appointment confirmation |
| **Storage** | PostgreSQL + Redis | Warehouse configuration and task state in PostgreSQL; real-time task queue and operator session state in Redis |

## Business Rules

| ID | Rule | Description |
|----|------|-------------|
| BR-WMS-001 | Directed Putaway Priority | Location selection evaluates velocity class first, then zone affinity, then available capacity. Fast-moving items (A-class) are assigned to forward pick locations within 20 feet of the nearest pack station |
| BR-WMS-002 | Wave Release Gate | A wave may not be released until all included orders have confirmed inventory allocation and valid carrier assignments. Orders failing either check are auto-removed and re-queued |
| BR-WMS-003 | Task Interleaving Eligibility | Operators are eligible for interleaved task types only if they hold active certifications for each task type and are physically located within or adjacent to the target zone |
| BR-WMS-004 | Cartonization Accuracy | All active item master dimensions and weights must be verified within the last 90 days. Items with expired dimension data are flagged for re-measurement before cartonization |
| BR-WMS-005 | Zone Picking Strategy | Each zone is assigned exactly one active picking strategy (discrete, batch, or wave). Strategy changes require zone quiescence — all active picks must complete before the switch takes effect |
| BR-WMS-006 | Cycle Count Thresholds | A-class items must be counted at least once per 30 days; B-class every 90 days; C-class every 180 days. Items with adjustment variance exceeding 2% of on-hand quantity trigger immediate recount |
| BR-WMS-007 | Cross-Dock Eligibility | An inbound line is eligible for cross-dock only if a matching outbound order exists with a requested ship date within 24 hours and the item does not require quality inspection |
| BR-WMS-008 | Yard Check-In Validation | Trailers cannot be checked into the yard without a valid carrier appointment, ASN reference, or expected delivery order. Unplanned arrivals are routed to a hold area pending manual disposition |
| BR-WMS-009 | Pick Completion Verification | Every pick task requires scan confirmation of both the source location barcode and the item SKU barcode. Quantity must match the pick directive exactly — over-picks and short-picks generate exceptions |
| BR-WMS-010 | Replenishment Trigger | Forward pick locations generate replenishment tasks when on-hand quantity falls below the configured reorder point. Replenishment tasks inherit the priority of the highest-priority open pick dependent on that location |

## Data Model

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `wms_warehouse_zones` | `id, tenant_id, warehouse_id, zone_code, zone_type, picking_strategy, temperature_range, is_active` | Belongs to `warehouse`; has many `wms_locations` |
| `wms_putaway_rules` | `id, tenant_id, warehouse_id, priority, velocity_class, zone_preference, storage_type, conditions, is_active` | Evaluated by putaway engine per item |
| `wms_wave_templates` | `id, tenant_id, warehouse_id, name, selection_criteria, grouping_fields, release_schedule, auto_release, is_active` | Has many `wms_waves` |
| `wms_pick_tasks` | `id, tenant_id, warehouse_id, wave_id, order_id, zone_id, location_id, sku, qty_requested, qty_picked, status, operator_id, assigned_at` | Belongs to `wms_wave`, `zone`, `location` |
| `wms_cartonization_rules` | `id, tenant_id, warehouse_id, carrier_id, carton_catalog, max_weight, dim_weight_factor, algorithm, is_active` | Applied during pack operations |

## Events

### Events Produced

| Event Type | Payload | Description |
|------------|---------|-------------|
| `commerce.wms.wave-released` | `{wave_id, warehouse_id, order_count, line_count, priority, released_at, released_by}` | A wave has been released for picking operations |
| `commerce.wms.pick-completed` | `{pick_task_id, warehouse_id, zone_id, operator_id, order_id, sku, qty_picked, completed_at}` | A pick task has been successfully completed by an operator |
| `commerce.wms.pack-completed` | `{pack_id, warehouse_id, order_id, carton_type, weight, dimensions, tracking_id, packed_at}` | An order has been packed and is ready for carrier handoff |
| `commerce.wms.putaway-completed` | `{putaway_task_id, warehouse_id, location_id, sku, qty, operator_id, completed_at}` | A putaway task has been completed and inventory location updated |

### Events Consumed

| Event Type | Source | Action |
|------------|--------|--------|
| `commerce.order.shipped` | Commerce Service | Update wave and pick task status; close completed order fulfillment cycle |
| `commerce.stock.adjusted` | Commerce Service | Re-evaluate forward pick location levels; trigger replenishment tasks if below reorder point |
| `commerce.delivery.scheduled` | Commerce Service | Create dock door appointment and yard check-in expectation for inbound carrier |
| `manufacturing.work-order.completed` | Manufacturing Service | Generate putaway tasks for finished goods arriving from production |
| `config.changed` | Config Service | Reload wave templates, putaway rules, cartonization parameters, and zone configurations |

## Integration Points

| Integration | Service | Protocol | Description |
|-------------|---------|----------|-------------|
| Order Management | Commerce Service | Internal API | Sales order release triggers wave inclusion and pick task generation |
| Inventory Management | Commerce Service | Internal API | Real-time stock levels for allocation, replenishment, and cycle count variance resolution |
| Transportation Management | Commerce Service | Event-driven | Carrier assignment and shipment confirmation for pack-and-ship handoff |
| Manufacturing Execution | Manufacturing Service | Event-driven | Finished goods receipts generate directed putaway tasks |
| Platform Config | Config Service | Event-driven | Dynamic reload of warehouse rules, templates, and operational parameters |

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `commerce.wms.putaway.velocity_recalc_days` | 7 | Days between ABC velocity class recalculation |
| `commerce.wms.wave.max_orders` | 200 | Maximum orders per wave release |
| `commerce.wms.pick.shortage_threshold_pct` | 5.0 | Percentage threshold for short-pick exception escalation |
| `commerce.wms.cycle_count.a_period_days` | 30 | Maximum days between A-class item counts |
| `commerce.wms.carton.dim_weight_factor` | 166 | Dimensional weight divisor (carrier standard) |
| `commerce.wms.cross_dock.max_hold_hours` | 24 | Maximum hours an item can wait in cross-dock staging |

## Implementation Notes

- Voice picking integration requires a dedicated voice terminal broker that manages persistent WebSocket connections to industrial devices; connection state is maintained in Redis with heartbeat monitoring and automatic reconnection.
- Task interleaving uses a priority-weighted scoring model: `score = urgency_weight * (1 / time_to_sla) + proximity_weight * (1 / distance_meters) + efficiency_weight * equipment_match`. The highest-scoring task is assigned when an operator completes their current task.
- Cartonization uses a modified first-fit decreasing (FFD) 3D bin-packing heuristic. For carriers with dimensional weight pricing, the algorithm minimizes `max(actual_weight, dim_weight)` across all cartons in a shipment.
- Cycle counting schedule generation runs as a daily batch job. The scheduler uses stratified random sampling weighted by adjustment frequency, dollar value, and time since last count to produce daily count assignments per zone.
- Cross-dock matching is performed at ASN receipt time using a pre-built matching index keyed on SKU, warehouse, and requested ship date. The index is rebuilt nightly from open outbound orders.
- Yard management dock door scheduling uses a greedy interval-packing algorithm with carrier appointment windows as constraints. Overlaps are resolved by priority (refrigerated > hazmat > general).

## Cross-References

- [Order Orchestration](order-orchestration.md) — Order release triggers wave planning and pick generation
- [Advanced Transportation](advanced-transportation.md) — Carrier assignment and shipment handoff
- [Commerce Service](../06-services/commerce.md) — Warehouse and inventory operations
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
- [NFR: Performance](../10-planning/nfr.md) — Pick task assignment < 500ms, wave release < 5s
