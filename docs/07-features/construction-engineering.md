# Construction & Engineering

Commitment management, progress billing, retention, change orders, and field operations for construction projects, managed by the Project Service.

## Overview

Construction & Engineering provides end-to-end project financial and operational management for construction and engineering firms. The module covers commitment management (purchase commitments and change orders), progress billing using percentage-of-completion and milestone-based methods, retention management with holdback and release scheduling, change order management with cost and schedule impact analysis, subcontractor management with prevailing wage compliance and certified payroll, daily field logs and time cards, equipment tracking and utilization, and construction-specific document management (RFIs, submittals, transmittals). Integration with Finance handles AIA billing and job costing, HR manages craft workers, and Commerce supplies materials procurement.

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Commitment Management** | Project Service (Rust) | Purchase commitments with approved value, committed costs, invoiced-to-date, and remaining balance tracking |
| **Progress Billing** | Project Service (Rust) | Percentage-of-completion and milestone-based billing with AIA-style schedule of values, retention holdback calculation |
| **Retention Management** | Project Service (Rust) | Configurable retention percentage, release scheduling based on milestone or calendar triggers, retention receivable tracking |
| **Change Order Management** | Project Service (Rust) | Change order creation with cost/schedule impact analysis, approval routing, commitment adjustment, and version control |
| **Subcontractor Management** | Project Service (Rust) | Subcontractor onboarding, insurance/expiry tracking, prevailing wage compliance, certified payroll reporting |
| **Daily Log & Time Cards** | Project Service (Rust) | Field daily logs (weather, work performed, visitors, incidents), craft worker time cards with cost code allocation |
| **Equipment Tracking** | Project Service (Rust) | Equipment assignment to projects, utilization hours, maintenance scheduling, and cost allocation |
| **Document Management** | Project Service + Platform Service | RFIs, submittals, transmittals with workflow routing, response tracking, and document version control |
| **Storage** | PostgreSQL | All commitments, billings, change orders, daily logs, and construction documents in Project database |

## Business Rules

| Rule | Description |
|------|-------------|
| **BR-CE-001** | A commitment cannot exceed the approved project budget for its cost category without an approved change order. |
| **BR-CE-002** | Progress billing is capped at the percentage of completion verified by field inspection. Override requires project manager approval. |
| **BR-CE-003** | Retention is held at the configured percentage (default 10%) on all progress payments. First half released at substantial completion; second half at final completion. |
| **BR-CE-004** | All change orders impacting schedule > 5 days or cost > $10,000 require project sponsor approval via Workflow Service. |
| **BR-CE-005** | Certified payroll reports must be generated weekly for all craft workers on prevailing wage projects. Non-compliance triggers an alert. |
| **BR-CE-006** | RFIs must be responded to within the configurable SLA (default 10 business days). Overdue RFIs escalate automatically. |

## Data Model

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `commitments` | `id, tenant_id, project_id, vendor_id, commitment_number, description, approved_value, committed_cost, invoiced_to_date, remaining_balance, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to project, has many `change_orders` |
| `progress_billings` | `id, tenant_id, project_id, commitment_id, billing_period, pct_complete, billed_to_date, retention_held, retention_released, this_period_amount, status, created_at, updated_at, version, is_deleted` | Belonds to project, commitment |
| `retention_schedules` | `id, tenant_id, project_id, commitment_id, retention_pct, total_retained, release_triggers[], released_amount, pending_release, created_at, updated_at, version, is_deleted` | Belongs to project, commitment |
| `change_orders` | `id, tenant_id, project_id, commitment_id, co_number, description, cost_impact, schedule_impact_days, status, approved_by, approved_at, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to commitment |
| `daily_logs` | `id, tenant_id, project_id, log_date, weather{}, work_summary, visitors[], incidents[], created_by, created_at, updated_at, version, is_deleted` | Belongs to project |
| `field_time_cards` | `id, tenant_id, project_id, employee_id, date, cost_code, hours_regular, hours_overtime, hours_doubletime, daily_total, created_at, updated_at, version, is_deleted` | Belongs to project, employee |
| `equipment_assignments` | `id, tenant_id, project_id, equipment_id, assigned_from, assigned_to, utilization_hours, cost_allocation, created_at, updated_at, version, is_deleted` | Belongs to project, equipment |
| `construction_documents` | `id, tenant_id, project_id, doc_type, doc_number, subject, status, assigned_to, due_date, response_due, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to project |

## Events

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `project.commitment.created` | `{commitment_id, project_id, vendor_id, approved_value}` | New commitment issued | Finance, Report |
| `project.progress-billing.submitted` | `{billing_id, project_id, period, amount, retention_held}` | Progress billing submitted | Finance, Report |
| `project.change-order.submitted` | `{co_id, project_id, cost_impact, schedule_impact_days}` | Change order created | Workflow, Finance |
| `project.change-order.approved` | `{co_id, project_id, approved_cost, approved_schedule_days}` | Change order approved | Finance, Report |
| `project.retention.released` | `{retention_id, project_id, amount, release_trigger}` | Retention released | Finance, Report |
| `project.daily-log.submitted` | `{log_id, project_id, log_date, weather_summary}` | Daily field log entered | Report, Dashboard |
| `project.rfi.response-required` | `{rfi_id, project_id, subject, due_date}` | RFI issued to design team | Workflow, Notification |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `finance.invoice.approved` | Finance Service | Update commitment invoiced-to-date |
| `hr.employee.time-submitted` | HR Service | Populate field time cards |
| `commerce.order.delivered` | Commerce Service | Track material deliveries against commitments |
| `workflow.step.approved` | Workflow Service | Finalize change order or RFI approval |
| `config.changed` | Config Service | Update retention rates, RFI SLAs, prevailing wage rules |

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/v1/construction/commitments` | Create a purchase commitment |
| GET | `/api/v1/construction/projects/{id}/commitments` | List commitments for a project |
| POST | `/api/v1/construction/billings` | Submit a progress billing |
| POST | `/api/v1/construction/change-orders` | Create a change order |
| POST | `/api/v1/construction/change-orders/{id}/submit-approval` | Submit change order for approval |
| POST | `/api/v1/construction/daily-logs` | Submit a daily field log |
| POST | `/api/v1/construction/time-cards` | Submit field time cards |
| GET | `/api/v1/construction/projects/{id}/retention` | View retention schedule |
| POST | `/api/v1/construction/documents` | Create RFI, submittal, or transmittal |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Job Costing / GL | Finance Service | Event-driven | AIA billing entries, commitment cost postings, retention accounting |
| AP / AR | Finance Service | Internal API | Subcontractor payments, progress billing receivables |
| Craft Workers | HR Service | Event-driven | Employee time submission, certified payroll data |
| Materials / Procurement | Commerce Service | Event-driven | Material deliveries tracked against commitments |
| Approval Workflows | Workflow Service | Event-driven | Change order and RFI approval routing |
| Documents | Platform Service | Internal API | Construction document storage and version control |
| Report Service | Report Service | Internal API | Project cost reports, earned value, WIP reporting |

## Cross-References

- [Project Service](../06-services/project.md) — Project service specification
- [Finance Service](../06-services/finance.md) — AIA billing, job costing, GL integration
- [HR Service](../06-services/hr.md) — Craft worker and payroll data
- [Commerce Service](../06-services/commerce.md) — Materials procurement
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
