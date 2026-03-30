# Manufacturing Execution System

Real-time shop floor execution with work instruction delivery, labor tracking, production monitoring, OEE calculation, and machine integration via OPC-UA, managed by the Manufacturing Service.

## Overview

The Manufacturing Execution System (MES) module provides the operational layer between production planning and the physical shop floor. It delivers digital work instructions to stations, tracks labor and production data in real time, monitors production line performance, and calculates Overall Equipment Effectiveness (OEE) at the station level. The module integrates with IoT telemetry for machine data collection via OPC-UA, supports shift management with handover workflows, and provides dispatch lists for work center task prioritization.

## Data Flow Diagram

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  Work Orders     │────▶│  Dispatch List   │────▶│  Station Execution  │
│  (Planning)      │     │  Manager         │     │  ┌───────────────┐  │
└──────────────────┘     └──────────────────┘     │  │ Work          │  │
                                                   │  │ Instructions  │  │
┌──────────────────┐     ┌──────────────────┐     │  └───────┬───────┘  │
│  OPC-UA / IoT    │────▶│  Machine Data    │────▶│          │          │
│  Devices         │     │  Collector       │     │  ┌───────▼───────┐  │
└──────────────────┘     └──────────────────┘     │  │ Labor Tracking│  │
                                                   │  │ Production    │  │
┌──────────────────┐     ┌──────────────────┐     │  │ Data Capture  │  │
│  Shift Schedule  │────▶│  Shift Manager   │────▶│  └───────┬───────┘  │
└──────────────────┘     └──────────────────┘     └──────────┼──────────┘
                                                              │
                                                   ┌──────────▼──────────┐
                                                   │   OEE Calculation   │
                                                   │   Availability x    │
                                                   │   Performance x     │
                                                   │   Quality           │
                                                   └─────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Work Instruction Delivery** | Manufacturing Service (Rust) | Version-controlled digital work instructions with media attachments, pushed to station terminals based on dispatch sequence |
| **Labor Tracking** | Manufacturing Service | Real-time labor transaction capture with clock-in/clock-out, job assignment, idle time categorization, and labor cost allocation |
| **Production Data Collection** | Manufacturing Service + Platform Service (IoT) | High-frequency production event capture including cycle counts, scrap, rework, and downtime with configurable data points per operation |
| **OEE Engine** | Manufacturing Service | Continuous OEE calculation at station, line, and plant levels with availability, performance, and quality components |
| **Shift Management** | Manufacturing Service | Shift scheduling with rotating patterns, skill-based assignment, handover checklists, and break management |
| **Dispatch Lists** | Manufacturing Service | Priority-sorted work queues per work center with real-time status updates and sequence optimization |
| **OPC-UA Integration** | Manufacturing Service + Integration Service | Bi-directional OPC-UA client for machine data collection and setpoint delivery with subscription-based monitoring |
| **Storage** | PostgreSQL + TimescaleDB | Configuration data in PostgreSQL; high-frequency production telemetry in TimescaleDB hypertables |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Work Instruction Manager | Versioned instruction CRUD, media management, station delivery | Manufacturing Service |
| Labor Transaction Engine | Clock events, job labor allocation, idle tracking, cost calculation | Manufacturing Service |
| Production Data Collector | Real-time capture of production events, scrap, rework, cycle times | Manufacturing Service |
| OEE Calculator | Continuous availability, performance, and quality metric computation | Manufacturing Service |
| Shift Scheduler | Shift pattern management, assignment, handover workflows | Manufacturing Service |
| Dispatch List Manager | Work center queue management, priority sorting, status tracking | Manufacturing Service |
| OPC-UA Connector | Machine communication, data subscription, command delivery | Manufacturing Service |
| Station Terminal API | Real-time data feeds for shop floor display terminals | Manufacturing Service |

## OEE Calculation Model

| Component | Formula | Data Source |
|-----------|---------|-------------|
| Availability | Run Time / Planned Production Time | Shift schedule, downtime events |
| Performance | (Ideal Cycle Time x Total Count) / Run Time | Cycle time standards, production counts |
| Quality | Good Count / Total Count | Inspection results, scrap/rework counts |
| OEE | Availability x Performance x Quality | Aggregated from all three components |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Work Orders | Manufacturing Service | Internal API | Production order release to dispatch lists |
| BOM & Routing | Manufacturing Service | Internal API | Routing steps define work instruction sequence |
| IoT Telemetry | Platform Service | Event-driven | Machine sensor data for OEE and monitoring |
| Digital Twin | Manufacturing Service | Internal API | Real-time twin updates from production data |
| Quality Management | Manufacturing Service | Internal API | In-process inspection triggers and results |
| Production Scheduling | Manufacturing Service | Internal API | Schedule drives dispatch list sequencing |
| Inventory | Commerce Service | Event-driven | Material consumption booking against WIP |
| Notification Service | Platform Service | Event-driven | Shift alerts, downtime notifications, anomaly warnings |
| Report Service | Report Service | Internal API | OEE dashboards, production analytics |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `manufacturing.mes.production-started` | `{production_id, work_order_id, station_id, operator_id, started_at}` | Operator starts production at station | Dispatch List, OEE Calculator, Report Service |
| `manufacturing.mes.production-completed` | `{production_id, work_order_id, station_id, good_qty, scrap_qty, cycle_time}` | Production operation completed | Work Order, Inventory, Quality, Report Service |
| `manufacturing.mes.labor-logged` | `{transaction_id, employee_id, station_id, work_order_id, hours, activity_type}` | Labor time booked against operation | Labor Cost, Report Service |
| `manufacturing.mes.shift-started` | `{shift_id, line_id, crew_id, started_at}` | Shift clock-in event recorded | OEE Calculator, Notification Service |
| `manufacturing.mes.downtime-recorded` | `{station_id, downtime_type, started_at, ended_at, reason_code}` | Downtime event logged | OEE Calculator, Notification Service, Report Service |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `manufacturing.work-order.released` | Manufacturing Service | Add operations to dispatch list |
| `manufacturing.work-order.completed` | Manufacturing Service | Remove from dispatch list, update OEE |
| `platform.iot.telemetry.received` | Platform Service | Process machine data for production tracking |
| `manufacturing.schedule.firmed` | Manufacturing Service | Update dispatch list sequence from schedule |
| `manufacturing.quality.inspection-completed` | Manufacturing Service | Update quality OEE component |
| `config.changed` | Config Service | Reload shift patterns or OEE parameters |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `mes_work_instructions` | `id, tenant_id, operation_id, version, title, content, media_refs, is_active` | Belongs to routing operation |
| `mes_labor_transactions` | `id, tenant_id, employee_id, station_id, work_order_id, operation_id, activity_type, hours, booked_at` | Belongs to `employee`, `work_order` |
| `mes_production_data` | `id, tenant_id, work_order_id, operation_id, station_id, operator_id, good_qty, scrap_qty, cycle_time, recorded_at` | Belongs to `work_order`, `station` |
| `mes_shift_schedules` | `id, tenant_id, line_id, crew_id, shift_type, start_time, end_time, effective_date, pattern` | Belongs to `production_line` |
| `mes_dispatch_lists` | `id, tenant_id, work_center_id, work_order_id, operation_id, sequence, priority, status, assigned_at` | Belongs to `work_center`, `work_order` |

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `manufacturing.mes.oee.calculation_interval_seconds` | 60 | OEE recalculation frequency |
| `manufacturing.mes.downtime.auto_threshold_seconds` | 300 | Seconds of inactivity before auto-downtime |
| `manufacturing.mes.shift.handover_grace_minutes` | 15 | Grace period for shift handover completion |
| `manufacturing.mes.dispatch.max_queue_size` | 50 | Maximum operations in a work center dispatch list |
| `manufacturing.mes.labor.idle_threshold_minutes` | 10 | Minutes before idle time auto-categorization |
| `manufacturing.mes.production.cycle_time_tolerance` | 0.2 | Deviation threshold for cycle time alerts |

## Cross-References

- [Production Scheduling](production-scheduling.md) — Schedule drives dispatch list sequencing
- [Quality Management](../06-services/manufacturing.md) — In-process inspection integration
- [IoT Architecture](../06-services/platform.md) — OPC-UA and telemetry infrastructure
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
- [Manufacturing Service](../06-services/manufacturing.md) — Manufacturing service specification
- [NFR: Performance](../10-planning/nfr.md) — Production data capture < 100ms latency
