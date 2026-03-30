# Advanced Production Scheduling

Constraint-based finite capacity scheduling with resource optimization, what-if simulation, and Gantt chart visualization, managed by the Manufacturing Service.

## Overview

Advanced Production Scheduling provides intelligent scheduling capabilities that go beyond basic work order sequencing. The module implements constraint-based scheduling algorithms that account for finite machine capacity, tooling availability, labor skills, material constraints, and changeover times. It supports what-if simulation scenarios for capacity planning, Gantt chart visualization for schedule communication, and simultaneous scheduling of multiple resources including machines, tools, and operators.

## Data Flow Diagram

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  Work Orders     │────▶│  Constraint      │────▶│  Scheduling Engine  │
│  + BOM/Routing   │     │  Collector       │     │  ┌───────────────┐  │
└──────────────────┘     │  ┌────────────┐  │     │  │ Finite        │  │
                         │  │ Material   │  │     │  │ Capacity      │  │
┌──────────────────┐     │  │ Machine    │  │     │  │ Scheduler     │  │
│  Resource        │────▶│  │ Tooling    │  │     │  └───────┬───────┘  │
│  Definitions     │     │  │ Labor      │  │     │  ┌───────▼───────┐  │
└──────────────────┘     │  └────────────┘  │     │  │ Optimizer     │  │
                         └──────────────────┘     │  │ (multi-obj)   │  │
┌──────────────────┐     ┌──────────────────┐     │  └───────┬───────┘  │
│  Simulation      │────▶│  What-If Engine  │────▶│          │          │
│  Parameters      │     └──────────────────┘     └──────────┼──────────┘
                                                              │
                                                   ┌──────────▼──────────┐
                                                   │  Schedule Output     │
                                                   │  ┌────────────────┐  │
                                                   │  │ Gantt Entries  │  │
                                                   │  │ Resource Alloc │  │
                                                   │  │ KPI Metrics    │  │
                                                   │  └────────────────┘  │
                                                   └─────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Finite Capacity Scheduling** | Manufacturing Service (Rust) | Schedule generation constrained by actual resource capacities rather than infinite capacity assumptions |
| **Constraint Engine** | Manufacturing Service | Multi-constraint evaluation including material availability, machine capability, tooling, labor skills, and changeover rules |
| **What-If Simulation** | Manufacturing Service | Forked schedule scenarios with parameter variations for capacity planning, impact analysis, and schedule comparison |
| **Gantt Chart Engine** | Manufacturing Service | Time-bar representation of scheduled operations with resource rows, dependency links, and drag-to-reschedule support |
| **Multi-Resource Scheduling** | Manufacturing Service | Simultaneous allocation of machines, operators, tools, and fixtures with conflict detection and resolution |
| **Optimization Engine** | Manufacturing Service | Multi-objective optimization balancing on-time delivery, resource utilization, changeover minimization, and cost |
| **Schedule Firming** | Manufacturing Service + Workflow Service | Freeze window management with approval workflow for schedule changes inside the frozen zone |
| **Storage** | PostgreSQL | Schedule data, simulation results, and Gantt entries in Manufacturing database |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Scheduling Engine | Finite capacity schedule generation and regeneration | Manufacturing Service |
| Constraint Collector | Aggregate material, machine, tooling, and labor constraints | Manufacturing Service |
| What-If Simulator | Forked scenario creation, execution, and comparison | Manufacturing Service |
| Gantt Chart Renderer | Schedule visualization data generation for UI rendering | Manufacturing Service |
| Resource Allocator | Multi-resource assignment with conflict detection | Manufacturing Service |
| Schedule Optimizer | Multi-objective schedule optimization | Manufacturing Service |
| Firming Manager | Freeze window control and change approval workflows | Manufacturing Service |
| Schedule Publisher | Firm schedule dissemination to MES dispatch lists | Manufacturing Service |

## Constraint Types

| Constraint | Category | Impact | Example |
|-----------|----------|--------|---------|
| Machine Capacity | Resource | Limits operations per time window | CNC mill: 1 operation at a time |
| Material Availability | Material | Delays start until material ready | Raw steel: available in 3 days |
| Tooling Requirement | Resource | Requires specific tool per operation | Fixture A required for op 20 |
| Labor Skill | Resource | Operator must have required certification | Welding: certified welder only |
| Changeover Time | Time | Adds setup time between different operations | Color change: 45 min cleanup |
| Predecessor | Dependency | Operation cannot start before predecessor ends | Op 20 must follow op 10 |
| Delivery Priority | Business | Elevates schedule position for priority orders | Customer A: must ship by Friday |
| Frozen Zone | Time | Prevents changes within time fence | No changes within 48 hours of now |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Work Orders | Manufacturing Service | Internal API | Unscheduled orders as scheduling input |
| BOM & Routing | Manufacturing Service | Internal API | Operation sequences, resource requirements, standard times |
| MES | Manufacturing Service | Event-driven | Firmed schedule published to dispatch lists |
| Material Requirements | Commerce Service | Event-driven | Material availability constraints for scheduling |
| IoT Telemetry | Platform Service | Event-driven | Real-time machine status for capacity updates |
| Digital Twin | Manufacturing Service | Internal API | Machine state and capability data |
| Workflow Engine | Workflow Service | Event-driven | Schedule change approval inside frozen zone |
| Report Service | Report Service | Internal API | Schedule adherence and utilization analytics |
| Calendar Management | HCM Service | Internal API | Shift patterns, holidays, and crew availability |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `manufacturing.schedule.created` | `{schedule_id, plant_id, horizon_start, horizon_end, order_count, generated_at}` | Schedule generation completes | MES, Report Service, Notification Service |
| `manufacturing.schedule.simulation.completed` | `{simulation_id, schedule_id, variant_count, best_scenario_id, compared_at}` | What-if simulation run completes | Report Service, Dashboard |
| `manufacturing.schedule.firmed` | `{schedule_id, firmed_by, frozen_until, operation_count}` | Schedule approved and frozen | MES (dispatch lists), Notification Service |
| `manufacturing.schedule.resource-conflict` | `{schedule_id, resource_id, conflict_type, conflicting_operations}` | Resource double-booking detected | Notification Service, Planner Dashboard |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `manufacturing.work-order.created` | Manufacturing Service | Add to scheduling queue |
| `manufacturing.work-order.priority-changed` | Manufacturing Service | Reschedule affected operations |
| `manufacturing.mes.production-completed` | Manufacturing Service | Update actuals and reschedule remaining |
| `manufacturing.mes.downtime-recorded` | Manufacturing Service | Adjust capacity and reschedule |
| `commerce.inventory.atp-calculated` | Commerce Service | Update material constraint availability |
| `platform.iot.telemetry.received` | Platform Service | Update machine status for capacity |
| `config.changed` | Config Service | Reload scheduling parameters |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `production_schedules` | `id, tenant_id, plant_id, status, horizon_start, horizon_end, frozen_until, generated_at, firmed_by` | Has many `schedule_gantt_entries`, `schedule_resources` |
| `schedule_resources` | `id, tenant_id, schedule_id, resource_id, resource_type, capacity_hours, allocated_hours, available_from` | Belongs to `production_schedules` |
| `schedule_constraints` | `id, tenant_id, schedule_id, constraint_type, resource_id, parameters, is_active` | Belongs to `production_schedules` |
| `schedule_simulations` | `id, tenant_id, base_schedule_id, name, parameters, status, best_scenario_id, created_at` | Has many `schedule_gantt_entries` (scenario) |
| `schedule_gantt_entries` | `id, tenant_id, schedule_id, work_order_id, operation_id, resource_id, start_time, end_time, sequence, status` | Belongs to `production_schedules`, `work_order` |

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `manufacturing.scheduling.horizon_days` | 30 | Default scheduling horizon in days |
| `manufacturing.scheduling.frozen_zone_hours` | 48 | Hours from now that are frozen |
| `manufacturing.scheduling.optimization_timeout_seconds` | 300 | Max time for optimization run |
| `manufacturing.scheduling.changeover_matrix_enabled` | true | Use changeover time matrix between products |
| `manufacturing.scheduling.simulation_max_variants` | 10 | Maximum what-if scenarios per simulation |
| `manufacturing.scheduling.auto_reschedule_on_deviation` | true | Auto-reschedule when actuals deviate beyond threshold |

## Cross-References

- [MES](mes.md) — Firmed schedules drive MES dispatch lists
- [Work Orders](../06-services/manufacturing.md) — Production orders as scheduling input
- [Digital Twin](../06-services/manufacturing.md) — Machine state for capacity planning
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
- [Manufacturing Service](../06-services/manufacturing.md) — Manufacturing service specification
- [NFR: Performance](../10-planning/nfr.md) — Schedule generation < 60s for 1000 operations
