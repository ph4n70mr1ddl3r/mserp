# Contract Manufacturing

Outsourced production management and contract manufacturer collaboration with IP protection, milestone tracking, quality inspection, and cost variance analysis, managed by the Manufacturing Service.

## Overview

Contract Manufacturing provides end-to-end management of outsourced production operations. The module handles contract manufacturer (CM) onboarding and qualification, outsourced BOM management with intellectual property protection rules, production order dispatch to external partners, and milestone-based progress tracking. It supports incoming quality inspection of CM deliveries, yield tracking and reporting, cost variance analysis between contracted and actual costs, lead time management, capacity reservation, material consignment tracking, CM performance scorecards, compliance and audit management, and dual-source risk management for supply chain resilience.

## Data Flow Diagram

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  CM Onboarding   │────▶│  Qualification   │────▶│  Approved CM        │
│  & Registration  │     │  Workflow        │     │  Registry           │
└──────────────────┘     └──────────────────┘     └──────────┬──────────┘
                                                              │
┌──────────────────┐     ┌──────────────────┐     ┌──────────▼──────────┐
│  Production      │────▶│  Order Dispatch  │────▶│  Outsourced Order   │
│  Demand          │     │  Engine          │     │  Management         │
└──────────────────┘     └──────────────────┘     └──────────┬──────────┘
                                                              │
┌──────────────────┐     ┌──────────────────┐     ┌──────────▼──────────┐
│  BOM (Filtered)  │────▶│  IP-Protected    │────▶│  CM BOM View        │
│                  │     │  BOM Builder      │     │  Dispatch           │
└──────────────────┘     └──────────────────┘     └──────────┬──────────┘
                                                              │
┌──────────────────┐     ┌──────────────────┐     ┌──────────▼──────────┐
│  Milestone       │────▶│  Progress        │────▶│  Incoming           │
│  Reports (CM)    │     │  Tracker         │     │  Inspection         │
└──────────────────┘     └──────────────────┘     └──────────┬──────────┘
                                                              │
                                               ┌─────────────▼─────────────┐
                                               │  Scorecard & Variance     │
                                               │  ┌──────────┐ ┌────────┐ │
                                               │  │ Yield    │ │ Cost   │ │
                                               │  │ Report   │ │ Compare│ │
                                               │  └──────────┘ └────────┘ │
                                               └───────────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **CM Onboarding** | Manufacturing Service (Rust) | Multi-step qualification workflow with document collection, capability assessment, site audit scheduling, and approval via Workflow Service |
| **Outsourced BOM** | Manufacturing Service | IP-protected BOM views that strip proprietary process details before dispatching to contract manufacturers |
| **Order Dispatch** | Manufacturing Service | Production order creation and dispatch to qualified CMs with contracted pricing, delivery milestones, and quality requirements |
| **Milestone Tracking** | Manufacturing Service | Configurable milestone-based progress tracking with acceptance gates, notifications, and escalation rules |
| **Incoming Inspection** | Manufacturing Service | Quality inspection workflows for CM deliveries with sampling plans, acceptance criteria, and disposition handling |
| **Yield & Cost Tracking** | Manufacturing Service | Real-time yield analysis against contracted targets with cost variance computation and alerting |
| **Capacity Reservation** | Manufacturing Service | Time-phased capacity booking with CMs including commitment tracking and overrun handling |
| **Consignment Inventory** | Manufacturing Service + Commerce Service | Material consignment tracking for components shipped to CM facilities with ownership and consumption reconciliation |
| **CM Scorecard** | Manufacturing Service | Weighted performance scoring across quality, delivery, cost, and responsiveness dimensions |
| **Storage** | PostgreSQL | All contract manufacturing data in Manufacturing database; RLS-scoped per tenant |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| CM Registry | Contract manufacturer CRUD, qualification status, contact management | Manufacturing Service |
| Qualification Engine | Multi-step qualification workflow, document management, audit scheduling | Manufacturing Service |
| Outsourced Order Manager | Order creation, dispatch, milestone definition, status tracking | Manufacturing Service |
| IP-Protected BOM Builder | BOM filtering, IP rule enforcement, CM-specific view generation | Manufacturing Service |
| Milestone Tracker | Progress capture, acceptance workflows, delay detection, escalation | Manufacturing Service |
| Incoming Inspection Handler | Inspection plan creation, result capture, disposition, NCR generation | Manufacturing Service |
| Yield Report Engine | Yield data collection, target comparison, trend analysis, alerting | Manufacturing Service |
| Cost Variance Analyzer | Contracted vs actual cost comparison, variance categorization, reporting | Manufacturing Service |
| Capacity Reservation Manager | Time-phased capacity booking, commitment tracking, availability checking | Manufacturing Service |
| Consignment Tracker | Material shipment logging, consumption reconciliation, ownership tracking | Manufacturing Service |
| Scorecard Calculator | Weighted performance computation, trend analysis, rating updates | Manufacturing Service |
| Dual-Source Risk Analyzer | Supplier concentration analysis, risk scoring, mitigation recommendations | Manufacturing Service |

## CM Qualification Workflow

| Stage | Gate | Actions | Approval Required |
|-------|------|---------|-------------------|
| Application | Document Submission | Company profile, certifications, capability matrix | No |
| Document Review | Compliance Check | Certifications validated, financial health assessed | Quality Manager |
| Technical Assessment | Capability Audit | Equipment list, process capabilities, capacity verification | Engineering Lead |
| Site Audit | On-Site Inspection | Facility tour, process observation, quality system review | Audit Team |
| Sample Production | First Article Inspection | Sample run, dimensional/functional testing, yield assessment | Quality Manager |
| Trial Order | Pilot Run | Small production run, full quality and delivery evaluation | Operations Manager |
| Approved | Full Qualification | Added to approved CM list, capacity and terms agreed | Supply Chain Director |

## IP Protection Levels

| Level | BOM Visibility | Detail Included | Use Case |
|-------|---------------|-----------------|----------|
| Full | Complete BOM | All components, materials, and specifications | Trusted strategic partner |
| Partial | Assembly-level only | Sub-assemblies as black boxes, final assembly details | Standard CM relationship |
| Restricted | Component list only | Part numbers and quantities, no sources or specs | New or low-trust CM |
| Minimal | Form-fit-function | Only interface specifications and test requirements | Prototype or one-time order |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Work Orders | Manufacturing Service | Internal API | Internal production orders converted to outsourced orders |
| BOM & Routing | Manufacturing Service | Internal API | BOM data filtered through IP protection rules for CM views |
| Quality Management | Manufacturing Service | Internal API | Inspection results feed quality module and scorecards |
| Inventory | Commerce Service | Event-driven | Material availability for consignment; stock updates from CM deliveries |
| Purchase Orders | Finance Service | Event-driven | PO confirmation triggers order dispatch readiness |
| Workflow Engine | Workflow Service | Event-driven | Qualification approvals, milestone acceptances, order approvals |
| Config Service | Config Service | Event-driven | IP protection rules, qualification criteria, scorecard weights |
| Report Service | Report Service | Internal API | CM performance dashboards, cost variance reports, yield analytics |
| Notification Service | Platform Service | Event-driven | Milestone alerts, delay warnings, inspection results |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `manufacturing.cm.order-dispatched` | `{order_id, cm_id, bom_view_id, milestone_count, contracted_cost, due_date, dispatched_at}` | Outsourced order dispatched to CM | Report Service, Notification Service, Finance Service |
| `manufacturing.cm.milestone-reached` | `{order_id, cm_id, milestone_id, milestone_type, completed_at, notes}` | CM reports milestone completion | Report Service, Notification Service, Workflow Service |
| `manufacturing.cm.inspection-completed` | `{inspection_id, order_id, cm_id, result, good_qty, reject_qty, disposition, inspected_at}` | Incoming quality inspection finished | Quality Management, Report Service, Commerce Service |
| `manufacturing.cm.yield-reported` | `{report_id, order_id, cm_id, contracted_yield, actual_yield, variance_pct, reported_at}` | Yield data recorded for CM production run | Report Service, Scorecard Calculator |
| `manufacturing.cm.cost-variance.detected` | `{order_id, cm_id, contracted_cost, actual_cost, variance_amount, variance_pct, currency}` | Actual cost deviates beyond threshold from contracted cost | Finance Service, Report Service, Notification Service |
| `manufacturing.cm.capacity-committed` | `{reservation_id, cm_id, period_start, period_end, capacity_hours, product_family, committed_at}` | CM confirms capacity reservation | Production Scheduling, Report Service |
| `manufacturing.cm.scorecard.updated` | `{cm_id, scorecard_id, overall_score, quality_score, delivery_score, cost_score, responsiveness_score, period}` | CM performance scorecard recalculated | Report Service, Notification Service |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `commerce.order.created` | Commerce Service | Evaluate if order requires outsourced production capacity |
| `commerce.stock.updated` | Commerce Service | Update consignment inventory and material availability for CM orders |
| `manufacturing.work-order.created` | Manufacturing Service | Assess internal capacity; convert to outsourced order if overflow |
| `manufacturing.quality.inspection-completed` | Manufacturing Service | Correlate with CM deliveries for incoming inspection workflows |
| `finance.purchase-order.confirmed` | Finance Service | Trigger order dispatch readiness when PO is confirmed |
| `workflow.step.approved` | Workflow Service | Advance CM qualification stage; accept milestone completion |
| `config.changed` | Config Service | Reload IP protection rules, scorecard weights, variance thresholds |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `contract_manufacturers` | `id, tenant_id, company_name, tax_id, status, ip_protection_level, lead_time_days, payment_terms, rating, qualification_date` | Has many `cm_qualifications`, `outsourced_orders`, `cm_scorecards` |
| `cm_qualifications` | `id, tenant_id, cm_id, stage, status, documents, audit_results, assessed_by, assessed_at, notes` | Belongs to `contract_manufacturers` |
| `outsourced_orders` | `id, tenant_id, cm_id, internal_order_id, bom_view_id, status, contracted_cost, actual_cost, currency, due_date, dispatched_at, completed_at` | Belongs to `contract_manufacturers`; has many `production_milestones`, `incoming_inspections` |
| `cm_bom_views` | `id, tenant_id, cm_id, order_id, bom_id, ip_protection_level, filtered_components, version` | Belongs to `contract_manufacturers`, `outsourced_orders` |
| `production_milestones` | `id, tenant_id, order_id, cm_id, milestone_type, sequence, target_date, actual_date, status, acceptance_status, notes` | Belongs to `outsourced_orders` |
| `incoming_inspections` | `id, tenant_id, order_id, cm_id, inspection_plan_id, lot_qty, sample_qty, good_qty, reject_qty, result, disposition, inspected_by, inspected_at` | Belongs to `outsourced_orders` |
| `cm_yield_reports` | `id, tenant_id, order_id, cm_id, contracted_yield, actual_yield, variance_pct, start_qty, good_qty, scrap_qty, rework_qty, reported_at` | Belongs to `outsourced_orders` |
| `cm_cost_variances` | `id, tenant_id, order_id, cm_id, cost_category, contracted_amount, actual_amount, variance_amount, variance_pct, currency, detected_at` | Belongs to `outsourced_orders` |
| `cm_capacity_reservations` | `id, tenant_id, cm_id, product_family, period_start, period_end, reserved_hours, committed_hours, status, committed_at` | Belongs to `contract_manufacturers` |
| `consignment_inventory` | `id, tenant_id, cm_id, material_id, qty_shipped, qty_consumed, qty_returned, qty_on_hand, location, last_reconciled_at` | Belongs to `contract_manufacturers` |
| `cm_scorecards` | `id, tenant_id, cm_id, period_start, period_end, overall_score, quality_score, delivery_score, cost_score, responsiveness_score, order_count, defect_rate_pct` | Belongs to `contract_manufacturers` |

All tables include standard columns: `id`, `tenant_id`, `created_at`, `updated_at`, `created_by`, `updated_by`, `version`, `is_deleted`.

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `manufacturing.cm.scorecard.quality_weight` | 0.35 | Weight for quality dimension in scorecard |
| `manufacturing.cm.scorecard.delivery_weight` | 0.30 | Weight for on-time delivery dimension |
| `manufacturing.cm.scorecard.cost_weight` | 0.20 | Weight for cost performance dimension |
| `manufacturing.cm.scorecard.responsiveness_weight` | 0.15 | Weight for responsiveness dimension |
| `manufacturing.cm.cost_variance.threshold_pct` | 10.0 | Percentage threshold for cost variance alerts |
| `manufacturing.cm.yield_variance.threshold_pct` | 5.0 | Percentage threshold for yield variance alerts |
| `manufacturing.cm.inspection.default_sample_pct` | 10.0 | Default sampling percentage for incoming inspection |
| `manufacturing.cm.milestone.overdue_grace_hours` | 24 | Hours before milestone overdue triggers escalation |
| `manufacturing.cm.consignment.reconciliation_interval_days` | 7 | Days between consignment inventory reconciliations |
| `manufacturing.cm.dual_source.max_concentration_pct` | 70.0 | Maximum percentage allocated to single CM |

## Cross-References

- [MES](mes.md) — Internal production data informs outsource decisions
- [Production Scheduling](production-scheduling.md) — Schedule capacity gaps trigger CM order dispatch
- [Quality Management](../06-services/manufacturing.md) — Incoming inspection integrates with quality module
- [Commerce Service](../06-services/commerce.md) — Inventory and order events for CM coordination
- [Finance Service](../06-services/finance.md) — Purchase order and cost variance integration
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
- [Manufacturing Service](../06-services/manufacturing.md) — Manufacturing service specification
- [NFR: Performance](../10-planning/nfr.md) — Order dispatch < 2s; scorecard calculation < 5s
