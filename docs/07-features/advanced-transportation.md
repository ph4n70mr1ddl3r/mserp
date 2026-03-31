# Advanced Transportation Management (TMS)

Freight rating, carrier selection algorithms, load consolidation, freight audit/payment, route optimization, shipment tracking, and delivery appointment scheduling, managed by the Commerce Service.

## Overview

Advanced Transportation Management (TMS) provides comprehensive planning, execution, and settlement capabilities for inbound and outbound freight across all transportation modes (parcel, LTL, FTL, intermodal, ocean, air). The module combines freight rating engines with carrier selection algorithms that evaluate cost, transit time, service quality, and capacity to determine optimal carrier assignments. Load consolidation groups multiple shipments into full truckloads or optimized container loads to reduce per-unit freight costs.

Freight audit and payment automation validates carrier invoices against contracted rates, accessorials, and service level agreements, flagging discrepancies for resolution. Route optimization leverages graph-based algorithms to minimize total transportation cost across multi-stop routes while respecting delivery time windows and driver hours-of-service regulations. Real-time shipment tracking aggregates carrier milestone updates (pickup, in-transit scans, delivery) into a unified tracking view. Delivery appointment scheduling coordinates dock door availability with carrier capacity and customer receiving windows.

## Data Flow Diagram

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  Outbound Orders │────▶│  Freight Rating  │────▶│  Carrier Selection  │
│  (WMS Packed)    │     │  Engine          │     │  ┌───────────────┐  │
└──────────────────┘     │  ┌─────────────┐ │     │  │ Multi-Criteria│  │
                         │  │ LTL/FTL/    │ │     │  │ Scoring Model │  │
┌──────────────────┐     │  │ Parcel Rate │ │     │  └───────┬───────┘  │
│  Inbound POs     │────▶│  │ Tables      │ │     └──────────┼──────────┘
│  (Procurement)   │     │  └─────────────┘ │                │
└──────────────────┘     └──────────────────┘     ┌──────────▼──────────┐
                                                   │  Load Consolidation │
┌──────────────────┐     ┌──────────────────┐     │  ┌───────────────┐  │
│  Carrier         │────▶│  Shipment        │────▶│  │ Trailer/      │  │
│  Milestones      │     │  Tracking        │     │  │ Container     │  │
│  (EDI 214/API)   │     │  Aggregator      │     │  │ Optimizer     │  │
└──────────────────┘     └──────────────────┘     │  └───────────────┘  │
                                                   └──────────┬──────────┘
┌──────────────────┐     ┌──────────────────┐                │
│  Carrier         │────▶│  Freight Audit   │     ┌──────────▼──────────┐
│  Invoices        │     │  & Payment       │     │  Route Optimizer    │
│  (EDI 210)       │     │  ┌─────────────┐ │     │  ┌───────────────┐  │
└──────────────────┘     │  │ 3-Way Match │ │     │  │ VRP Solver    │  │
                         │  │ (Rate→Ship  │ │     │  │ + Time Windows│  │
                         │  │  →Invoice)  │ │     │  └───────────────┘  │
                         │  └─────────────┘ │     └─────────────────────┘
                         └──────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Freight Rating Engine** | Commerce Service (Rust) | Multi-carrier rate calculation using contracted tariff tables, density-based pricing, and accessorial matrices. Supports LTL class-based, FTL per-mile, parcel dimensional weight, and ocean TEU-based rating |
| **Carrier Selection Algorithm** | Commerce Service (Rust) | Multi-criteria scoring: cost (40%), transit time (25%), on-time performance (20%), claims ratio (10%), strategic preference (5%). Configurable weights per lane and mode |
| **Load Consolidation** | Commerce Service (Rust) | Bin-packing heuristic for trailer/container utilization optimization. Groups shipments by compatible destination zones, equipment requirements, and delivery windows |
| **Freight Audit Engine** | Commerce Service (Rust) | Automated invoice matching against rated shipments with tolerance thresholds for amount, weight, and accessorial charges. Discrepancy categorization and dispute workflow initiation |
| **Route Optimization** | Commerce Service (Rust) | Vehicle routing problem (VRP) solver with time windows, capacity constraints, and multi-objective optimization (cost, time, carbon). Supports single-depot and multi-depot scenarios |
| **Shipment Tracking** | Commerce Service + Integration Service | Carrier milestone aggregation via EDI 214, API polling, and webhook integrations. Unified tracking timeline with estimated delivery recalculation based on real-time position data |
| **Appointment Scheduling** | Commerce Service (Rust) | Dock door calendar with carrier appointment windows, service time estimates, and conflict detection. Two-way confirmation workflow with carrier acceptance |
| **Storage** | PostgreSQL + Redis | Rate tables, carrier profiles, and shipment records in PostgreSQL; real-time tracking state and rate cache in Redis |

## Business Rules

| ID | Rule | Description |
|----|------|-------------|
| BR-TMS-001 | Carrier Selection Minimum Bids | Every shipment rated above $500 must receive rates from at least three active carriers before assignment. If fewer than three carriers service the lane, document the justification and proceed with available bids |
| BR-TMS-002 | Load Consolidation Window | Consolidation planning considers all shipments with a ready-to-ship date within a configurable window (default 48 hours). Shipments with expedited SLAs may be excluded from consolidation at planning time |
| BR-TMS-003 | Freight Audit Tolerance | Carrier invoices matching the rated amount within 2% variance are auto-approved. Invoices exceeding 2% or containing unmatched accessorials are routed to the dispute queue with full supporting documentation |
| BR-TMS-004 | Route Optimization Constraints | All routes must comply with driver hours-of-service regulations (max 11 hours driving, 14 hours on-duty). Routes exceeding limits require a relay or overnight stop, included in the cost optimization |
| BR-TMS-005 | Carrier Performance Threshold | Carriers with on-time delivery below 90% over a rolling 90-day period are flagged for review. Carriers below 80% are auto-excluded from selection unless explicitly overridden by a logistics manager |
| BR-TMS-006 | Appointment Conflict Resolution | When carrier appointment requests conflict for the same dock door, priority is given by shipment value (highest first), then by SLA urgency (earliest delivery commitment first) |
| BR-TMS-007 | Hazardous Material Segregation | Shipments containing hazardous materials must be rated and routed only by carriers with active HAZMAT certifications. Mixed loads require compliant segregation plans before tender |
| BR-TMS-008 | Proof of Delivery Requirement | All shipments with declared value exceeding $10,000 require signature proof of delivery. Carrier integration must return POD documentation within 24 hours of delivery or trigger an exception |
| BR-TMS-009 | Multi-Stop Route Efficiency | Multi-stop routes must achieve a minimum trailer utilization of 60% by cubic volume. Routes below this threshold are rejected and re-planned as direct shipments or held for consolidation |
| BR-TMS-010 | Carbon Emission Tracking | Every rated shipment includes an estimated CO2 emission calculation based on mode, distance, and weight. Carbon cost is included as an optional scoring criterion in carrier selection |

## Data Model

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `tms_freight_rate_tables` | `id, tenant_id, carrier_id, origin_zone, dest_zone, mode, rate_type, rate_amount, minimum_charge, effective_from, effective_to` | Belongs to `carrier`; used by rating engine |
| `tms_carrier_selection_rules` | `id, tenant_id, lane_id, mode, criteria_weights, min_carriers, excluded_carriers, strategic_preferences, is_active` | Evaluated per shipment for carrier scoring |
| `tms_load_plans` | `id, tenant_id, plan_date, equipment_type, origin, destination, stops, total_weight, total_volume, utilization_pct, status` | Has many `tms_shipment_legs` |
| `tms_route_templates` | `id, tenant_id, origin, destination, mode, avg_transit_days, avg_cost, carriers, is_active` | Used for route optimization initialization |
| `tms_shipment_legs` | `id, tenant_id, shipment_id, load_plan_id, carrier_id, origin, destination, status, tracking_number, picked_up_at, delivered_at` | Belongs to `shipment`, `load_plan`, `carrier` |

## Events

### Events Produced

| Event Type | Payload | Description |
|------------|---------|-------------|
| `commerce.tms.load-planned` | `{load_plan_id, warehouse_id, shipment_count, equipment_type, total_weight, planned_date}` | A load plan has been created consolidating multiple shipments |
| `commerce.tms.carrier-assigned` | `{shipment_id, carrier_id, rate_amount, transit_days, assigned_at, assigned_by}` | A carrier has been selected and assigned to a shipment |
| `commerce.tms.shipment-in-transit` | `{shipment_id, carrier_id, tracking_number, origin, destination, departed_at}` | Shipment has been picked up by carrier and is in transit |
| `commerce.tms.delivery-confirmed` | `{shipment_id, carrier_id, delivered_at, pod_reference, signature_verified}` | Shipment has been delivered and proof of delivery recorded |

### Events Consumed

| Event Type | Source | Action |
|------------|--------|--------|
| `commerce.order.shipped` | Commerce Service | Rate shipment, execute carrier selection, create shipment leg |
| `commerce.wms.wave-released` | Commerce Service | Plan outbound load consolidation for wave shipments |
| `commerce.stock.transferred` | Commerce Service | Rate and plan inter-facility transfer shipments |
| `finance.invoice.received` | Finance Service | Match carrier invoice against rated shipment for freight audit |
| `config.changed` | Config Service | Reload rate tables, carrier selection weights, and routing parameters |

## Integration Points

| Integration | Service | Protocol | Description |
|-------------|---------|----------|-------------|
| Warehouse Management | Commerce Service | Internal API | Pack completion triggers carrier tender and shipment creation |
| Order Management | Commerce Service | Event-driven | Order shipment triggers rating and carrier selection |
| Carrier Networks | Integration Service | EDI / API | Rate requests, tender acceptance, tracking milestone ingestion, POD retrieval |
| Freight Audit & Payable | Finance Service | Internal API | Carrier invoice matching, dispute workflow, and payment scheduling |
| Platform Config | Config Service | Event-driven | Dynamic reload of rate tables, carrier rules, and optimization parameters |

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `commerce.tms.rating.default_carrier_count` | 3 | Minimum carrier rates to request per shipment |
| `commerce.tms.consolidation.window_hours` | 48 | Planning window for load consolidation |
| `commerce.tms.audit.tolerance_pct` | 2.0 | Auto-approve variance threshold for freight invoices |
| `commerce.tms.carrier.performance_window_days` | 90 | Rolling window for carrier on-time performance calculation |
| `commerce.tms.routing.max_stops` | 8 | Maximum stops per multi-stop route |
| `commerce.tms.tracking.poll_interval_minutes` | 15 | Carrier tracking milestone polling frequency |

## Implementation Notes

- The freight rating engine uses a layered evaluation pipeline: base rate lookup → accessorial application → fuel surcharge calculation → discount/contracted rate overlay. Each layer produces a rated line item for full audit transparency.
- Carrier selection scoring is implemented as a weighted sum model with configurable criteria weights stored in `tms_carrier_selection_rules`. Weights are normalized to sum to 1.0 before scoring.
- Route optimization uses a modified Clarke-Wright savings algorithm for initial route construction, followed by local search (2-opt, or-opt) improvements. Time windows are handled via forward time slack computation.
- Load consolidation applies a first-fit decreasing heuristic on shipment volumes against standard equipment profiles (53' dry van, 40' container, etc.). Cubic utilization is tracked per load plan for reporting.
- Freight audit matching correlates carrier invoices to rated shipments using bill of lading number, PRO number, or shipment ID. Three-way match (shipment → rate → invoice) with configurable tolerance per accessorial type.
- Tracking state is maintained in Redis as a sorted set of milestone events per shipment, enabling real-time ETA recalculation. Historical milestones are periodically flushed to PostgreSQL for audit and analytics.

## Cross-References

- [Advanced Warehouse Management](advanced-warehouse-management.md) — Pack completion and dock door handoff
- [Order Orchestration](order-orchestration.md) — Shipment creation triggers rating and carrier assignment
- [Commerce Service](../06-services/commerce.md) — Transportation and logistics operations
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
- [NFR: Performance](../10-planning/nfr.md) — Rate calculation < 2s, carrier selection < 5s per shipment
