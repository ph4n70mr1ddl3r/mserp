# Manufacturing Service

| Aspect | Details |
|--------|---------|
| Port | 8013 |
| Database | `manufacturing_db` |
| Responsibilities | Production operations, quality management, cost accounting, PLM, EAM, manufacturing intelligence, digital thread, health & safety (EHS), MRO, product compliance |

## Modules

| Module | Description |
|--------|-------------|
| Bill of Materials (BOM) | Multi-level BOM, phantom assemblies, alternative BOMs, engineering change orders |
| Work Orders | Work order creation, scheduling, material issuance, labor reporting, completion |
| Production Planning | MPS (Master Production Schedule), MRP (Material Requirements Planning), capacity planning |
| Advanced Supply Chain Planning (ASCP) | Constraint-based planning, what-if simulation, optimized scheduling, alternate routing selection |
| Quality Control | Inspection plans, sampling procedures, non-conformance reports (NCR), CAPA |
| Advanced Quality Management (AQM) | Statistical Process Control (SPC), Six Sigma tracking, quality cost analysis, supplier quality management |
| Maintenance | Preventive maintenance schedules, work orders, spare parts tracking, equipment downtime |
| Enterprise Asset Management (EAM) | Asset registration and lifecycle tracking, maintenance strategy (preventive, predictive, condition-based), asset hierarchy, spare parts inventory, depreciation integration with Finance |
| Work Centers | Machine and labor resources, capacity, cost rates, scheduling constraints |
| Routing | Operation sequences, setup times, run times, move times, overlapping operations |
| Cost Accounting | Standard cost roll-up, variance analysis (material, labor, overhead), actual costing |
| Shop Floor Control | Real-time production tracking, barcode/RFID scanning, operator assignments |
| Product Lifecycle Management (PLM) | Item concept and revision management, engineering change order (ECO) workflow, BOM versioning, collaboration with suppliers on design, document linking (CAD, specs), product phase-in/phase-out planning |
| Sustainability Tracking | Energy consumption per production run, waste tracking, emissions per product, material recyclability tracking |

## Manufacturing Intelligence

| Module | Description |
|--------|-------------|
| OEE Tracking | Overall Equipment Effectiveness measurement by work center, line, and plant |
| Downtime Analysis | Downtime categorization, root cause tracking, Pareto analysis |
| Production Analytics | Real-time production rate tracking, yield analysis, scrap monitoring |
| Production Rate Tracking | Real-time production rate monitoring against targets with variance alerting |
| Capacity Utilization | Work center utilization dashboards, bottleneck identification |
| Energy Analytics | Energy consumption per unit produced, cost allocation, efficiency trends |
| Predictive Maintenance Analytics | Maintenance cost analysis, MTBF/MTTR tracking, maintenance effectiveness |

## Advanced Planning & Scheduling (APS)

| Module | Description |
|--------|-------------|
| Finite Capacity Scheduling | Finite capacity scheduling across work centers considering machine, labor, and tooling constraints |
| Constraint-Based Optimization | Multi-constraint optimization (material, capacity, tooling) for production scheduling |
| What-If Scheduling | Interactive what-if scheduling scenarios with drag-and-drop Gantt chart manipulation |
| Scheduling Optimization | Automated schedule optimization for throughput, cost, or delivery performance objectives |

## Lean Manufacturing

| Module | Description |
|--------|-------------|
| Kanban Management | Electronic Kanban board management with pull-based replenishment triggers and card circulation tracking |
| Value Stream Mapping | Value stream mapping tools for identifying waste, measuring lead time, and tracking improvement |
| Pull-Based Production | Pull-based production control with demand-driven work order release and WIP limits |

## Connected Worker

| Module | Description |
|--------|-------------|
| Digital Work Instructions | Interactive digital work instructions with multimedia content (video, 3D models) replacing paper-based procedures |
| Real-Time Quality Feedback | In-line quality feedback at the workstation with SPC integration and immediate corrective action alerts |
| AR Guidance | Augmented reality guidance for complex assembly operations with overlay instructions and part verification |

## Digital Thread

| Module | Description |
|--------|-------------|
| Traceability Graph | Unified traceability linking design, manufacturing, quality, and service records |
| Serial/Lot Genealogy | Full genealogy from raw material to finished product to customer delivery |
| Change Impact Analysis | Impact analysis for ECOs showing affected orders, inventory, and production plans |
| Product Instance Tracking | Lifecycle tracking of individual product instances from manufacture to disposal |

## IoT Integration

| Module | Description |
|--------|-------------|
| Device Cache | Local cache of IoT device metadata synced from Platform Service's authoritative device registry |
| Telemetry Ingestion | High-throughput telemetry data ingestion from sensors and industrial equipment |
| Alert Rules | Configurable threshold and anomaly-based alerting on real-time telemetry data |
| Edge Processing | Optional edge computing for latency-sensitive processing before cloud ingestion |
| Data Routing | Configurable telemetry routing to manufacturing, logistics, or maintenance modules |
| Device Health Monitoring | Device connectivity status, battery levels, and firmware update management |

**Note:** Platform Service owns the authoritative IoT device registry and certificate management. Manufacturing Service's `iot_devices` table is a local cache of industrial device metadata. Device registration events originate from Platform (`platform.iot.device.registered`) and Manufacturing subscribes to populate its local cache. Manufacturing publishes telemetry, alert, and operational events under the `manufacturing.iot.*` namespace.

## Digital Twin

| Module | Description |
|--------|-------------|
| Asset Digital Twins | Virtual representation of physical assets with real-time state synchronization |
| Simulation Engine | What-if simulation for production scenarios, capacity planning, and maintenance impact |
| Predictive Maintenance | ML-based prediction of equipment failure from telemetry and historical maintenance data |
| Real-Time Monitoring | Live dashboard of asset health, performance metrics, and anomaly detection |
| Twin Analytics | Asset performance comparison, degradation trends, and optimization recommendations |

## Enterprise Health & Safety (EHS)

| Module | Description |
|--------|-------------|
| Incident Management | Safety incident reporting, investigation workflows, root cause analysis, corrective and preventive actions (CAPA) |
| Risk Assessment | Job hazard analysis (JHA), risk matrices, hierarchy of controls, residual risk tracking |
| Permit-to-Work | Digital work permit management with approval workflows and time-bounded validity |
| Inspection Management | Scheduled and ad-hoc safety inspections with checklists and corrective action assignment |
| Chemical Management | Safety Data Sheet (SDS) management, hazardous material inventory, exposure tracking |
| Occupational Health | Medical surveillance programs, exposure monitoring, health assessments, return-to-work workflows |
| Training & Certification | Safety training requirements tracking, certification expiration alerts, competency verification |
| Regulatory Compliance | OSHA, ISO 45001, and jurisdiction-specific regulatory reporting with automated form generation |

## Maintenance, Repair & Operations (MRO)

| Module | Description |
|--------|-------------|
| MRO Catalog | Centralized catalog of maintenance supplies, spare parts, and consumables |
| MRO Procurement | Automated reorder point-based procurement with blanket purchase agreements |
| Spare Parts Inventory | Dedicated spare parts inventory with criticality classification and obsolescence tracking |
| Repair Order Management | Repair work orders with failure codes, repair estimates, and turnaround time monitoring |
| Overhaul Management | Complex overhaul project management with task breakdown and parts kitting |
| Rotable Management | Rotable component tracking with repair cycle management and exchange pools |

## Product Compliance & Regulatory Management

| Module | Description |
|--------|-------------|
| Regulatory Database | Configurable regulatory requirements database by product category, market, and jurisdiction |
| Compliance Tracking | Product compliance status tracking against applicable regulations with gap identification |
| Certification Management | Product certification lifecycle (UL, CE, FCC, FDA, RoHS, REACH) with renewal tracking |
| Material Compliance | Restricted substance tracking (RoHS, REACH, TSCA, Prop 65) with material declaration management |
| Testing Management | Compliance test plan management with lab integration and result tracking |
| Regulatory Change Management | Regulatory change monitoring with impact analysis on existing products |

## Manufacturing Execution System (MES)

| Module | Description |
|--------|-------------|
| Work Instructions | Digital work instructions with step-by-step procedures, multimedia attachments, and version control |
| Station-Level Tracking | Real-time tracking of production status at each workstation with throughput and cycle time monitoring |
| Labor Transactions | Labor time recording against operations, work orders, and cost centers with overtime and idle time tracking |
| Shift Management | Shift scheduling, handover management, crew assignment, and shift-level production reporting |
| Production Data Collection | Automated and manual production data capture including quantities, scrap, rework, and downtime reasons |

## Advanced Production Scheduling

| Module | Description |
|--------|-------------|
| Constraint-Based Scheduling | Multi-constraint scheduling considering machine capacity, tooling availability, material readiness, and labor skills |
| Gantt Chart | Interactive Gantt chart with drag-and-drop rescheduling, dependency visualization, and critical path highlighting |
| Simulation | What-if scheduling simulation comparing scenarios for throughput, delivery performance, and resource utilization |
| Finite Capacity | Finite capacity planning with overload detection, alternative routing suggestions, and capacity leveling |

## Database Tables

> All tables include standard columns per [SPEC.md §9.1](../SPEC.md).

| Table | Additional Columns |
|-------|-------------------|
| `boms` | `product_id UUID`, `name VARCHAR(255)`, `type VARCHAR(20)`, `quantity DECIMAL`, `uom VARCHAR(20)`, `status VARCHAR(20)`, `revision INT`, `eco_id UUID` |
| `bom_items` | `bom_id UUID`, `component_product_id UUID`, `quantity DECIMAL`, `uom VARCHAR(20)`, `scrap_factor DECIMAL`, `is_phantom BOOLEAN`, `alternative_group VARCHAR(50)`, `sort_order INT` |
| `work_orders` | `order_number VARCHAR(50)`, `bom_id UUID`, `product_id UUID`, `quantity DECIMAL`, `status VARCHAR(20)`, `priority VARCHAR(10)`, `scheduled_start DATE`, `scheduled_end DATE`, `actual_start TIMESTAMPTZ`, `actual_end TIMESTAMPTZ`, `routing_id UUID`, `work_center_id UUID` |
| `work_order_operations` | `work_order_id UUID`, `operation_num INT`, `description TEXT`, `work_center_id UUID`, `setup_time_hrs DECIMAL`, `run_time_hrs DECIMAL`, `status VARCHAR(20)`, `actual_start TIMESTAMPTZ`, `actual_end TIMESTAMPTZ` |
| `quality_inspections` | `inspection_number VARCHAR(50)`, `work_order_id UUID`, `product_id UUID`, `type VARCHAR(20)`, `status VARCHAR(20)`, `result VARCHAR(10)`, `inspector_id UUID`, `inspected_at TIMESTAMPTZ`, `findings JSONB` |
| `maintenance_orders` | `order_number VARCHAR(50)`, `asset_id UUID`, `type VARCHAR(20)`, `status VARCHAR(20)`, `priority VARCHAR(10)`, `scheduled_date DATE`, `completed_date DATE`, `technician_id UUID`, `downtime_hours DECIMAL`, `cost DECIMAL` |
| `assets` | `asset_number VARCHAR(50)`, `name VARCHAR(255)`, `type VARCHAR(30)`, `location VARCHAR(255)`, `status VARCHAR(20)`, `parent_asset_id UUID`, `acquisition_date DATE`, `specifications JSONB` |
| `work_centers` | `code VARCHAR(50)`, `name VARCHAR(255)`, `type VARCHAR(20)`, `capacity DECIMAL`, `capacity_uom VARCHAR(20)`, `cost_rate DECIMAL`, `status VARCHAR(20)`, `location VARCHAR(255)` |
| `routings` | `product_id UUID`, `name VARCHAR(255)`, `status VARCHAR(20)`, `is_default BOOLEAN`, `revision INT` |
| `plm_products` | `product_id UUID`, `revision VARCHAR(20)`, `phase VARCHAR(20)`, `eco_id UUID`, `specifications JSONB`, `documents JSONB`, `status VARCHAR(20)` |
| `iot_devices` | `device_id VARCHAR(100)`, `name VARCHAR(255)`, `type VARCHAR(30)`, `asset_id UUID`, `certificate_id UUID`, `firmware_version VARCHAR(50)`, `status VARCHAR(20)`, `last_telemetry_at TIMESTAMPTZ`, `configuration JSONB` |
| `digital_twins` | `asset_id UUID`, `name VARCHAR(255)`, `model_version VARCHAR(20)`, `state JSONB`, `last_synced_at TIMESTAMPTZ`, `status VARCHAR(20)` |
| `ehs_incidents` | Safety incident records with severity, type, and CAPA linkage |
| `ehs_risk_assessments` | Risk assessment records with risk matrix scoring and controls |
| `ehs_permits` | Digital work permits with approval workflows and validity periods |
| `ehs_inspections` | Safety inspection records with checklists and findings |
| `ehs_chemicals` | Chemical inventory with SDS linkage and hazard classification |
| `mro_catalog_items` | MRO catalog entries with criticality classification and reorder parameters |
| `mro_spare_parts` | Spare parts inventory with criticality and obsolescence tracking |
| `mro_repair_orders` | Repair work orders with failure codes and turnaround time |
| `mro_rotable_components` | Rotable component tracking with repair cycle management |
| `product_regulations` | Regulatory requirements by product category and jurisdiction |
| `product_certifications` | Product certification lifecycle with status and renewal tracking |
| `material_declarations` | Material composition declarations with restricted substance tracking |
| `compliance_test_plans` | Test plan management with lab assignments and result tracking |
| `iot_telemetry_events` | Time-series telemetry events partitioned by month |
| `iot_device_state` | Latest device state snapshots with health scores |
| `digital_thread_events` | Traceability events linking serial/lot numbers to production operations |

> **Note:** The `iot_devices` table is a **local cache** synced by consuming `platform.iot.device.*` events. Manufacturing does NOT publish device registration events — that is Platform Service's domain. Manufacturing only publishes `manufacturing.iot.telemetry.*` events.

## Events Published

| Event | Description |
|-------|-------------|
| `manufacturing.bom.created` | BOM created |
| `manufacturing.bom.updated` | BOM updated |
| `manufacturing.work_order.created` | Work order created |
| `manufacturing.work_order.started` | Work order started (production began) |
| `manufacturing.work_order.completed` | Work order completed |
| `manufacturing.quality.checked` | Quality check result recorded |
| `manufacturing.quality.failed` | Quality check failed (NCR created) |
| `manufacturing.maintenance.due` | Scheduled maintenance due |
| `manufacturing.eco.submitted` | Engineering Change Order submitted |
| `manufacturing.eco.approved` | Engineering Change Order approved |
| `manufacturing.eco.implemented` | Engineering Change Order implemented |
| `manufacturing.product_lifecycle.revision.created` | Product revision created in PLM |
| `manufacturing.product_lifecycle.phase-in` | Product phase-in initiated |
| `manufacturing.product_lifecycle.phase-out` | Product phase-out initiated |
| `manufacturing.asset.registered` | Enterprise asset registered |
| `manufacturing.asset.maintenance_completed` | Asset maintenance completed |
| `manufacturing.asset.decommissioned` | Asset decommissioned |
| `manufacturing.plan.firmed` | Production plan firmed |
| `manufacturing.plan.simulation.completed` | ASCP simulation completed |
| `manufacturing.iot.telemetry.received` | Telemetry data received from IoT device |
| `manufacturing.iot.alert.triggered` | IoT alert rule triggered |
| `manufacturing.iot.device.offline` | IoT device went offline |
| `manufacturing.digital_twin.state.updated` | Digital twin state synchronized with physical asset |
| `manufacturing.digital_twin.simulation.completed` | Digital twin simulation completed |
| `manufacturing.digital_twin.prediction.generated` | Predictive maintenance prediction generated |
| `manufacturing.intelligence.oee.threshold_breached` | OEE dropped below configured threshold |
| `manufacturing.intelligence.downtime.categorized` | Downtime event categorized by root cause |
| `manufacturing.intelligence.energy.anomaly` | Energy consumption anomaly detected |
| `manufacturing.intelligence.predictive_maintenance.alert` | Predictive maintenance alert triggered |
| `manufacturing.safety.incident.created` | Safety incident reported |
| `manufacturing.safety.inspection.completed` | Safety inspection completed |
| `manufacturing.mro.repair.completed` | MRO repair order completed |
| `manufacturing.compliance.certification.updated` | Product certification updated |

## Events Consumed

Inbox binding: `manufacturing.inbox` binds to the following routing keys:

| Binding Pattern | Events Consumed |
|----------------|-----------------|
| `manufacturing.#` | Self-binding for saga compensation |
| `manufacturing.mes.#` | Self-binding for MES saga compensation |
| `manufacturing.schedule.#` | Self-binding for scheduling saga compensation |
| `platform.iot.device.#` | `platform.iot.device.registered`, `platform.iot.device.updated`, `platform.iot.device.decommissioned` — syncs local `iot_devices` cache |
| `commerce.stock.#` | `commerce.stock.updated`, `commerce.stock.reserved`, `commerce.stock.reservation.failed`, `commerce.stock.reservation.released` |
| `commerce.order.#` | `commerce.order.created`, `commerce.order.submitted`, `commerce.order.fulfilled`, `commerce.order.cancelled`, `commerce.order.returned` |
| `commerce.warranty.#` | `commerce.warranty.claim.submitted`, `commerce.warranty.claim.approved` |
| `config.changed` | `config.changed` |

## See Also

- [Commerce Service](commerce.md)
- [Finance Service](finance.md)
- [Report Service](report.md)
- [Architecture Overview](../01-architecture/overview.md)
