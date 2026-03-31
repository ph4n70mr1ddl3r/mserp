# Shared Services Center

Centralized AP/AR processing across business units with SLA management, managed by the Finance Service.

## Overview

The Shared Services Center (SSC) provides a unified processing hub for accounts payable and accounts receivable operations across multiple business units and legal entities. It consolidates invoice processing, payment execution, and reconciliation into a centralized workbench with intelligent queue management, workload balancing, and automated routing based on complexity and processor skills. SLA tracking per business unit ensures compliance with internal service level agreements, while productivity metrics and escalation management maintain operational efficiency and continuous improvement.

## Data Flow Diagram

```
┌──────────────┐     ┌──────────────────┐     ┌─────────────────────────┐
│  Invoices /  │────▶│  SSC Ingestion   │────▶│  Routing Engine          │
│  Payments /  │     │  & Classification │     │  ┌───────────────────┐  │
│  Exceptions  │     │  ┌─────────────┐  │     │  │ Complexity Score  │  │
│              │     │  │ Entity & BU │  │     │  │ (rules + ML)      │  │
│              │     │  │ assignment   │  │     │  └───────┬───────────┘  │
└──────────────┘     │  └─────────────┘  │     │  ┌───────▼───────────┐  │
                     └──────────────────┘     │  │ Skill Matching    │  │
                                                │  │ (processor fit)   │  │
                                                │  └───────┬───────────┘  │
                                                └──────────┼─────────────┘
                                                           │
                              ┌────────────────────────────┼───────────────┐
                              │                            │               │
                     ┌────────▼────────┐        ┌─────────▼──────┐  ┌────▼──────┐
                     │  Team Workbench │        │  SLA Engine    │  │  Workload  │
                     │  ┌───────────┐  │        │  ┌───────────┐ │  │  Balancer  │
                     │  │ My Queue  │  │        │  │ Track per │ │  │  ┌───────┐ │
                     │  │ Team Queue│  │        │  │ BU & type │ │  │  │Fair   │ │
                     │  │ Escalated │  │        │  └───────────┘ │  │  │dist.  │ │
                     │  └───────────┘  │        └─────────┬──────┘  │  └───────┘ │
                     └────────┬────────┘                  │         └────┬──────┘
                              │                           │              │
                              ▼                           ▼              ▼
                     ┌────────────────────────────────────────────────────────┐
                     │           Payment Execution & Reconciliation            │
                     │  Batch payments → Cross-entity recon → Dashboards       │
                     └─────────────────────────┬──────────────────────────────┘
                                               │
                                               ▼
                     ┌────────────────────────────────────────────────────────┐
                     │           Productivity Metrics & Reporting              │
                     │  Processor KPIs → BU SLA compliance → Improvements     │
                     └────────────────────────────────────────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Multi-Entity Invoice Processing** | Finance Service (Rust) | Unified invoice ingestion across all tenant entities with automatic entity and business unit classification. Supports inter-entity AP/AR with cross-charging |
| **Centralized Payment Execution** | Finance Service (Rust) | Batch payment processing across entities with consolidated payment files, netting of inter-entity payables/receivables, and centralized bank communication |
| **Shared Team Workbench** | Finance Service (Rust) | Web-based workbench with personal queue, team queue, and escalation queue views. Real-time task assignment, reassignment, and status tracking |
| **Queue Management** | Finance Service (Rust) | Priority-based queue with configurable sorting (FIFO, SLA urgency, complexity). Supports pull-based (processor selects) and push-based (auto-assign) distribution |
| **SLA Tracking** | Finance Service (Rust) | Per-business-unit SLA definitions with configurable targets for processing time, resolution time, and accuracy. Real-time SLA compliance dashboards with breach alerting |
| **Workload Balancing** | Finance Service (Rust) | Fair distribution algorithm considering processor capacity, skill set, current workload, and SLA urgency. Prevents overload and ensures equitable distribution |
| **Automated Routing** | Finance Service + Report Service | Rule-based and ML-assisted routing based on document complexity, processor skills, business unit, invoice type, and historical processor performance |
| **Cross-Entity Reconciliation** | Finance Service (Rust) | Automated matching of intercompany AP/AR transactions across entities. Elimination entries and net settlement calculations |
| **Vendor Master Governance** | Finance Service (Rust) | Centralized vendor master management with duplicate detection, change approval workflows, and compliance checks across all entities |
| **Shared Reporting Dashboards** | Finance Service + Report Service | Real-time operational dashboards: queue depth, processing times, SLA compliance, processor productivity, exception aging, and volume trends |
| **Processor Productivity Metrics** | Finance Service (Rust) | Per-processor metrics: items processed, average handling time, accuracy rate, SLA compliance contribution, and throughput trends |
| **Escalation Management** | Finance Service + Workflow Service | Multi-tier escalation: processor → team lead → manager, based on SLA breach proximity and severity. Automated reassignment on escalation |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Accounts Payable | Finance Service | Internal API | Invoice data, approval status, payment scheduling |
| Accounts Receivable | Finance Service | Internal API | AR invoices, payment receipts, dunning |
| Payment Gateway | Finance Service | Internal API | Payment file generation, bank communication |
| Cash Management | Finance Service | Internal API | Cash position, bank account data, liquidity |
| Touchless Invoicing | Finance Service | Internal API | Auto-processed vs. exception invoices from touchless pipeline |
| Reconciliation | Finance Service | Internal API | Cross-entity reconciliation, exception matching |
| Employee Directory | HR Service | Event-driven | Processor availability, skill profiles, organizational changes |
| Workflow Engine | Workflow Service | Event-driven | Approval workflows, escalation rules, SLA management |
| Report Service | Report Service | Internal API | Operational dashboards, analytics, ML routing model |
| Notification Service | Platform Service | Event-driven | Task assignments, SLA breach alerts, escalation notifications |
| Config Service | Platform Service | Event-driven | SLA definitions, routing rules, queue configuration |
| Audit Trail | Platform Service | Event-driven | Full audit log of SSC processing actions |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `finance.ssc.task-routed` | `{task_id, entity_id, business_unit, complexity, assigned_to, assigned_team, queue, routed_at}` | Task assigned to processor or queue | Dashboard, Notification Service |
| `finance.ssc.task-completed` | `{task_id, entity_id, processor_id, processing_time_ms, sla_met, completed_at}` | Processor completes task | SLA Engine, Metrics, Dashboard |
| `finance.ssc.sla-breached` | `{sla_id, entity_id, business_unit, task_id, breach_type, elapsed_time, sla_target}` | SLA threshold exceeded | Notification Service, Escalation Engine, Dashboard |
| `finance.ssc.payment-batch-executed` | `{batch_id, entity_id, payment_count, total_amount, payment_method, executed_at}` | Centralized payment batch submitted | GL, Cash Management, Dashboard |
| `finance.ssc.reconciliation.completed` | `{reconciliation_id, source_entity, dest_entity, matched_count, exception_count, net_settlement}` | Cross-entity reconciliation completes | GL, Dashboard |
| `finance.ssc.metrics.snapshot` | `{period_start, period_end, entity_id, business_unit, tasks_processed, avg_time_ms, sla_compliance_rate, productivity_index}` | Periodic metrics snapshot | Dashboard, Report Service |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `finance.invoice.created` | Finance Service | Route new invoice to SSC work queue for processing |
| `finance.payment.due-approaching` | Finance Service | Prioritize payment processing for approaching due dates |
| `finance.reconciliation.exception-created` | Finance Service | Route reconciliation exceptions to SSC queue |
| `hr.employee.updated` | HR Service | Update processor skill profiles and availability |
| `workflow.step.approved` | Workflow Service | Process approval decisions, advance task status |
| `config.changed` | Platform Service | Reload SLA definitions, routing rules, queue configuration |

## Data Model Reference

### `ssc_work_queues`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `queue_name` | VARCHAR(100) | Queue display name |
| `queue_type` | VARCHAR(30) | `ap_invoice`, `ar_invoice`, `payment`, `reconciliation`, `exception`, `escalation` |
| `entity_id` | UUID | Legal entity scope (nullable for cross-entity queues) |
| `business_unit` | VARCHAR(100) | Business unit scope (nullable for shared queues) |
| `description` | TEXT | Queue description |
| `assignment_mode` | VARCHAR(20) | `push` (auto-assign), `pull` (processor selects), `hybrid` |
| `priority` | INTEGER | Queue priority level (lower = higher priority) |
| `max_queue_depth` | INTEGER | Maximum items allowed in queue before alert |
| `sla_definition_id` | UUID | SLA applied to this queue |
| `is_active` | BOOLEAN | Whether queue is active |
| `config` | JSONB | Additional queue configuration |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

### `ssc_processing_tasks`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `queue_id` | UUID | Reference to `ssc_work_queues` |
| `entity_id` | UUID | Legal entity |
| `business_unit` | VARCHAR(100) | Business unit |
| `task_type` | VARCHAR(30) | `invoice_processing`, `payment_execution`, `reconciliation`, `exception_resolution`, `vendor_maintenance` |
| `source_type` | VARCHAR(30) | `invoice`, `payment`, `reconciliation_exception`, `vendor_change` |
| `source_id` | UUID | Reference to the source document |
| `complexity_score` | DECIMAL(5,2) | Assessed complexity (0-100) |
| `priority` | INTEGER | Task priority (lower = higher priority) |
| `status` | VARCHAR(20) | `queued`, `assigned`, `in_progress`, `pending_review`, `completed`, `escalated`, `returned` |
| `assigned_to` | UUID | Processor assigned (nullable) |
| `assigned_team` | VARCHAR(100) | Team assigned |
| `assigned_at` | TIMESTAMPTZ | Assignment timestamp |
| `started_at` | TIMESTAMPTZ | Processing start timestamp (nullable) |
| `completed_at` | TIMESTAMPTZ | Completion timestamp (nullable) |
| `processing_time_ms` | INTEGER | Actual processing time in milliseconds (nullable) |
| `sla_due_at` | TIMESTAMPTZ | SLA deadline |
| `sla_met` | BOOLEAN | Whether SLA was met (nullable) |
| `escalation_level` | INTEGER | Current escalation level (0 = none) |
| `escalated_to` | UUID | Escalated to user (nullable) |
| `resolution_notes` | TEXT | Resolution details (nullable) |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

### `ssc_sla_definitions`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `name` | VARCHAR(200) | SLA definition name |
| `entity_id` | UUID | Applicable entity (nullable for all entities) |
| `business_unit` | VARCHAR(100) | Applicable business unit (nullable for all BUs) |
| `task_type` | VARCHAR(30) | Task type the SLA applies to |
| `processing_time_target_ms` | INTEGER | Target processing time in milliseconds |
| `resolution_time_target_ms` | INTEGER | Target resolution time from queue entry |
| `accuracy_target_pct` | DECIMAL(5,2) | Target first-pass accuracy rate |
| `volume_target` | INTEGER | Target daily volume per processor (nullable) |
| `is_active` | BOOLEAN | Whether SLA is active |
| `effective_from` | DATE | SLA effective start date |
| `effective_to` | DATE | SLA effective end date (nullable) |
| `escalation_tiers` | JSONB | Escalation configuration: `[{level, breach_pct, notify_users[]}]` |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

### `ssc_sla_metrics`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `sla_definition_id` | UUID | Reference to `ssc_sla_definitions` |
| `entity_id` | UUID | Legal entity |
| `business_unit` | VARCHAR(100) | Business unit |
| `metric_date` | DATE | Date of measurement |
| `total_tasks` | INTEGER | Total tasks in period |
| `tasks_within_sla` | INTEGER | Tasks completed within SLA |
| `sla_compliance_rate` | DECIMAL(5,2) | SLA compliance percentage |
| `avg_processing_time_ms` | INTEGER | Average processing time |
| `avg_resolution_time_ms` | INTEGER | Average resolution time from queue entry |
| `first_pass_accuracy` | DECIMAL(5,2) | First-pass accuracy rate |
| `breach_count` | INTEGER | Number of SLA breaches |
| `escalation_count` | INTEGER | Number of escalations |
| `period` | VARCHAR(20) | `daily`, `weekly`, `monthly` |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

### `ssc_routing_rules`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `name` | VARCHAR(200) | Rule name |
| `description` | TEXT | Rule description |
| `priority` | INTEGER | Execution priority (lower = higher priority) |
| `conditions` | JSONB | Routing conditions (entity, BU, task type, complexity, amount, etc.) |
| `target_queue_id` | UUID | Target work queue |
| `target_team` | VARCHAR(100) | Target team (nullable) |
| `target_processor_id` | UUID | Specific processor (nullable) |
| `use_ml_routing` | BOOLEAN | Whether to use ML-assisted routing |
| `is_active` | BOOLEAN | Whether rule is active |
| `effective_from` | DATE | Rule effective start date |
| `effective_to` | DATE | Rule effective end date (nullable) |
| `applied_count` | INTEGER | Number of times rule has been applied |
| `last_applied_at` | TIMESTAMPTZ | Last application timestamp |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

### `ssc_team_assignments`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `team_name` | VARCHAR(100) | Team name |
| `processor_id` | UUID | Employee/user reference |
| `role` | VARCHAR(30) | `processor`, `reviewer`, `team_lead`, `manager` |
| `skill_set` | JSONB | Processor skills: `[{domain, entities[], task_types[], complexity_max}]` |
| `capacity` | INTEGER | Daily processing capacity (items) |
| `current_workload` | INTEGER | Current active task count |
| `is_active` | BOOLEAN | Whether assignment is active |
| `effective_from` | DATE | Assignment start date |
| `effective_to` | DATE | Assignment end date (nullable) |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

### `ssc_productivity_metrics`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `processor_id` | UUID | Employee/user reference |
| `team_name` | VARCHAR(100) | Team name |
| `metric_date` | DATE | Date of measurement |
| `tasks_processed` | INTEGER | Total tasks processed |
| `avg_handling_time_ms` | INTEGER | Average handling time per task |
| `first_pass_accuracy` | DECIMAL(5,2) | Percentage of tasks completed without rework |
| `sla_compliance_count` | INTEGER | Tasks completed within SLA |
| `sla_compliance_rate` | DECIMAL(5,2) | SLA compliance percentage |
| `escalations_received` | INTEGER | Escalated tasks received |
| `reassignments` | INTEGER | Tasks reassigned to others |
| `overtime_minutes` | INTEGER | Overtime worked (nullable) |
| `utilization_rate` | DECIMAL(5,2) | Productive time / available time |
| `period` | VARCHAR(20) | `daily`, `weekly`, `monthly` |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

## SLA & Escalation Model

```
┌─────────────────────────────────────────────────────────────────────────┐
│              SLA Tracking & Escalation Engine                            │
│                                                                          │
│  SLA Lifecycle:                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  Task Created → SLA Clock Starts → Processing → Completed      │    │
│  │       │              │                    │            │         │    │
│  │       │              ▼                    ▼            ▼         │    │
│  │       │         50% elapsed:        80% elapsed:   SLA Met ✓   │    │
│  │       │         (on track)         Warning alert                │    │
│  │       │                                              SLA Breach ✗│    │
│  │       │                                              → Escalate  │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                          │
│  Escalation Tiers:                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  Level 1 (80% SLA elapsed): Notify processor + team lead       │    │
│  │  Level 2 (SLA breached):      Reassign to team lead, notify mgr│    │
│  │  Level 3 (SLA + 50% grace):   Escalate to manager, route to    │    │
│  │                                overflow processor               │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                          │
│  Workload Balancing:                                                     │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  Score = (capacity - current_load) × skill_match × urgency      │    │
│  │  Assign to processor with highest score                         │    │
│  │  Enforce max capacity limit per processor per day               │    │
│  └─────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────┘
```

## Business Rules

| Rule | Description |
|------|-------------|
| **BR-SSC-001** | All AP/AR documents must enter through the SSC work queue. No direct processing outside the queue is permitted to ensure SLA tracking and audit trail. |
| **BR-SSC-002** | Routing rules are evaluated in priority order. The first matching rule determines the target queue and team. If no rule matches, the task is routed to the default queue. |
| **BR-SSC-003** | Workload balancing must not assign a processor more tasks than their configured daily capacity. Overflow tasks remain in the team queue for the next available processor. |
| **BR-SSC-004** | SLA compliance is measured per business unit. Each BU's SLA target is independent and tracked separately. Aggregate compliance is weighted by volume. |
| **BR-SSC-005** | Escalation tier 1 notification must be sent when 80% of the SLA time has elapsed. Tier 2 escalation (reassignment) occurs at SLA breach. Tier 3 escalation (manager) occurs at SLA + 50% of the SLA duration. |
| **BR-SSC-006** | Payment batches for multiple entities must be consolidated where possible. Inter-entity netting is applied to reduce the total number of external payments. |
| **BR-SSC-007** | Cross-entity reconciliation must be completed within 2 business days of period close. Unmatched intercompany items are escalated to both entity controllers. |
| **BR-SSC-008** | Processor skill profiles must be updated within 24 hours of any organizational change (transfer, role change, leave). Stale skill data triggers a review notification to the team lead. |
| **BR-SSC-009** | Vendor master changes (new vendor, bank detail change, address change) require dual approval independent of the requester. Compliance screening is triggered for all new vendors. |
| **BR-SSC-010** | Productivity metrics are computed daily and aggregated weekly/monthly. Processors with consistently low accuracy (< 95%) or low SLA compliance (< 90%) trigger a coaching workflow. |
| **BR-SSC-011** | All queue operations (assign, reassign, escalate, complete, return) are logged with timestamp, user, and reason code for audit and analytics. |
| **BR-SSC-012** | Metrics snapshots are taken at configurable intervals (default: hourly). Daily roll-ups are immutable once finalized. Monthly roll-ups are available by the 2nd business day. |

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `finance.ssc.default_assignment_mode` | `hybrid` | Default queue assignment mode |
| `finance.ssc.max_processor_capacity` | 50 | Default daily task capacity per processor |
| `finance.sdc.sla_warning_threshold_pct` | 80 | SLA elapsed % to trigger warning |
| `finance.ssc.escalation_tier1_notify` | true | Notify processor + lead at tier 1 |
| `finance.ssc.escalation_tier2_reassign` | true | Reassign to lead at tier 2 |
| `finance.ssc.metrics_snapshot_interval_sec` | 3600 | Seconds between metric snapshots |
| `finance.ssc.vendor_dual_approval` | true | Require dual approval on vendor changes |
| `finance.ssc.intercompany_recon_days` | 2 | Business days for cross-entity reconciliation |
| `finance.ssc.accuracy_coaching_threshold` | 95 | Accuracy % below which coaching is triggered |
| `finance.ssc.sla_coaching_threshold` | 90 | SLA compliance % below which coaching is triggered |

## Cross-References

- [Touchless Invoicing](touchless-invoicing.md) — Automated invoice processing feeding SSC queues
- [Account Reconciliation](reconciliation.md) — Cross-entity reconciliation and matching
- [Dynamic Discounting](dynamic-discounting.md) — Early payment discount optimization
- [Finance Service](../06-services/finance.md) — Finance service specification
- [HR Service](../06-services/hr.md) — Employee data and organizational changes
- [Workflow Engine](../06-services/workflow.md) — Escalation and approval workflows
- [Architecture Overview](../01-architecture/overview.md) — System communication patterns
- [NFR: Performance](../10-planning/nfr.md) — SSC processing latency and SLA requirements
