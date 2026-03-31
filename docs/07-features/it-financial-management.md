# IT Financial Management

IT cost transparency, chargebacks, and investment analysis, managed by the Finance Service.

## Overview

IT Financial Management (ITFM) provides comprehensive visibility into technology costs across the organization, enabling informed decision-making about IT investments, cost optimization, and value delivery. The system supports multiple cost allocation models (direct, indirect, shared services), generates chargeback and showback reports to business units, tracks IT investment portfolios with TCO analysis, manages cloud cost optimization, and provides budget vs. actual tracking with variance analysis. It integrates with general ledger, procurement, project management, and cloud billing systems to create a unified view of technology spend.

## Data Flow Diagram

```
┌──────────────┐     ┌──────────────────┐     ┌─────────────────────────┐
│  GL / AP /   │────▶│  Cost Collection │────▶│  Allocation Engine       │
│  Cloud Bills │     │  & Normalization │     │  ┌───────────────────┐  │
│  / Contracts │     │  ┌─────────────┐  │     │  │ Direct Allocation │  │
│  / Projects  │     │  │ Classify &  │  │     │  │ (invoice-level)   │  │
│              │     │  │ Normalize   │  │     │  └───────┬───────────┘  │
└──────────────┘     │  └─────────────┘  │     │  ┌───────▼───────────┐  │
                     └──────────────────┘     │  │ Indirect / Shared │  │
                                                │  │ (cost pool dist.) │  │
                                                │  └───────┬───────────┘  │
                                                └──────────┼─────────────┘
                                                           │
                              ┌────────────────────────────┼───────────────┐
                              │                            │               │
                     ┌────────▼────────┐        ┌─────────▼──────┐  ┌────▼──────┐
                     │  Chargeback /   │        │  TCO & ROI     │  │  Budget vs │
                     │  Showback       │        │  Analysis      │  │  Actual    │
                     │  Reporting      │        │  Engine        │  │  Tracking  │
                     └────────┬────────┘        └─────────┬──────┘  └────┬──────┘
                              │                           │              │
                              ▼                           ▼              ▼
                     ┌────────────────────────────────────────────────────────┐
                     │        IT Financial Dashboards & Reporting              │
                     │  Cost transparency → Investment decisions → Optimization│
                     └─────────────────────────────────────────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Cost Pool Management** | Finance Service (Rust) | Defines and manages IT cost pools (infrastructure, applications, personnel, cloud, licenses, projects). Costs are collected from GL, AP, cloud billing APIs, and vendor contracts into unified cost pools |
| **Cost Allocation Engine** | Finance Service (Rust) | Multi-model allocation: direct allocation (traceable to a BU), indirect allocation (overhead distributed by allocation keys), and shared services allocation (usage-based distribution). Supports configurable allocation keys: headcount, revenue, usage metrics, FTE, custom weights |
| **Chargeback / Showback** | Finance Service (Rust) | Generates chargeback (actual cost recovery via journal entries) or showback (informational reporting) to business units. Supports tiered pricing models, negotiated rates, and markup/markdown for shared services |
| **IT Investment Portfolio Tracking** | Finance Service + Workflow Service | Tracks IT investments through lifecycle: proposal, approval, execution, benefits realization. Portfolio view with status, spend, timeline, and ROI projections |
| **TCO Analysis** | Finance Service (Rust) | Total Cost of Ownership analysis for technology assets: acquisition costs, operational costs, maintenance, support, and end-of-life. Compares build vs. buy, on-prem vs. cloud scenarios |
| **Cloud Cost Optimization** | Finance Service (Rust) | Cloud billing ingestion, cost anomaly detection, rightsizing recommendations, reserved instance coverage analysis, unused resource identification, and cost trend forecasting |
| **Budget vs. Actual Tracking** | Finance Service (Rust) | IT budget entry at cost pool, project, and vendor level. Actuals collected from GL and AP. Variance analysis with configurable thresholds and automated alerts |
| **Project-Based IT Costing** | Finance Service (Rust) | Tracks all IT costs (labor, infrastructure, licenses, vendor) at the project level. Capitalization rules determine which costs are capitalized vs. expensed per accounting standards |
| **Vendor Contract Cost Management** | Finance Service (Rust) | Tracks vendor contract values, committed spend, actual spend, renewal dates, and contract utilization. Alerts for over/under-utilization and upcoming renewals |
| **ROI Analysis** | Finance Service (Rust) | Calculates return on IT investments: NPV, IRR, payback period, benefit-cost ratio. Tracks projected vs. realized benefits with variance analysis |
| **Storage** | PostgreSQL + MinIO | IT financial data in PostgreSQL; supporting documents (contracts, invoices, reports) in MinIO |

## Cost Allocation Model

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    IT Cost Allocation Model                              │
│                                                                          │
│  Direct Costs (Traceable to a specific BU):                              │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  Dedicated infrastructure, BU-specific licenses, project costs  │    │
│  │  Allocation: Direct charge to consuming BU                      │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                          │
│  Indirect Costs (Shared across BUs):                                     │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  Network, security, IT governance, helpdesk, shared services   │    │
│  │  Allocation: Distributed by allocation key (headcount, revenue, │    │
│  │  usage metrics, FTE, custom weights)                            │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                          │
│  Shared Services (Usage-based):                                          │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  Cloud compute, storage, database, SaaS subscriptions           │    │
│  │  Allocation: Usage-based (CPU-hours, GB-months, API calls)     │    │
│  │  with tiered pricing and volume discounts                       │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                          │
│  Chargeback vs. Showback:                                                │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  Chargeback: Actual journal entry to BU cost center (hard $)   │    │
│  │  Showback: Informational report only (BU sees cost, no entry)  │    │
│  │  Hybrid: Direct costs charged back, indirect shown              │    │
│  └─────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────┘
```

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| General Ledger | Finance Service | Internal API | GL transactions for IT cost collection, chargeback journal posting |
| Accounts Payable | Finance Service | Internal API | Vendor invoices for IT spend classification |
| Budget Management | Finance Service | Internal API | IT budget definitions, allocations, and revisions |
| Project Management | Project Service | Event-driven | Project costs, labor tracking, milestone data |
| Subscription Billing | Commerce Service | Event-driven | SaaS subscription costs, usage metrics |
| Cloud Billing APIs | Integration Service | Connector | AWS, Azure, GCP billing data ingestion |
| Workflow Engine | Workflow Service | Event-driven | IT investment approval workflows |
| Report Service | Report Service | Internal API | IT financial dashboards, analytics, forecasting |
| Notification Service | Platform Service | Event-driven | Budget variance alerts, contract renewal reminders |
| Config Service | Platform Service | Event-driven | Allocation rules, cost pool definitions, thresholds |
| Audit Trail | Platform Service | Event-driven | Full audit log of IT financial calculations and chargebacks |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `finance.itfm.cost-allocated` | `{allocation_id, period, cost_pool_id, source_entity, target_entities[], allocation_method, total_amount, allocations[]}` | Cost pool allocation run completes | GL, Dashboard, BU Reporting |
| `finance.itfm.chargeback.executed` | `{chargeback_id, period, business_unit, total_chargeback, journal_entry_ids[], line_items[]}` | Chargeback journal entries posted | GL, Dashboard, BU Reporting |
| `finance.itfm.investment.approved` | `{investment_id, name, category, total_budget, projected_roi, approved_by, approved_at}` | IT investment approved through workflow | Dashboard, Budget, Project Service |
| `finance.itfm.tco-calculated` | `{tco_id, asset_or_project, years, acquisition_cost, operational_cost, total_tco, npv, irr}` | TCO analysis completes | Dashboard, Investment Review |
| `finance.itfm.budget-variance.detected` | `{variance_id, period, cost_pool_id, budget_amount, actual_amount, variance_pct, threshold_exceeded}` | Variance exceeds configured threshold | Notification Service, Dashboard |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `finance.journal.posted` | Finance Service | Collect GL transactions for IT cost pools |
| `finance.invoice.approved` | Finance Service | Classify vendor invoices for IT spend |
| `finance.budget.allocated` | Finance Service | Load IT budget allocations for comparison |
| `platform.scheduler.job-executed` | Platform Service | Trigger periodic cost allocation and chargeback runs |
| `project.invoice.generated` | Project Service | Collect project costs for IT investment tracking |
| `commerce.subscription.billing.completed` | Commerce Service | Collect subscription costs for allocation |
| `workflow.step.approved` | Workflow Service | Process IT investment approval decisions |

## Data Model Reference

### `it_cost_pools`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `name` | VARCHAR(200) | Cost pool name |
| `description` | TEXT | Cost pool description |
| `category` | VARCHAR(50) | `infrastructure`, `applications`, `cloud`, `licenses`, `personnel`, `projects`, `security`, `network`, `support`, `other` |
| `allocation_type` | VARCHAR(20) | `direct`, `indirect`, `shared_services` |
| `allocation_key` | VARCHAR(50) | Allocation method: `headcount`, `revenue`, `usage`, `fte`, `equal`, `custom` |
| `custom_weights` | JSONB | Custom allocation weights per BU (nullable) |
| `gl_account_ids` | UUID[] | GL accounts feeding into this cost pool |
| `chargeback_mode` | VARCHAR(20) | `chargeback`, `showback`, `hybrid` |
| `markup_pct` | DECIMAL(5,2) | Chargeback markup percentage (default 0) |
| `is_active` | BOOLEAN | Whether cost pool is active |
| `effective_from` | DATE | Pool effective start date |
| `effective_to` | DATE | Pool effective end date (nullable) |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

### `it_allocation_rules`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `cost_pool_id` | UUID | Reference to `it_cost_pools` |
| `name` | VARCHAR(200) | Rule name |
| `description` | TEXT | Rule description |
| `priority` | INTEGER | Execution priority (lower = higher priority) |
| `source_filter` | JSONB | Filter criteria for costs to allocate (GL account, vendor, project, etc.) |
| `allocation_method` | VARCHAR(30) | `proportional`, `fixed_amount`, `tiered`, `usage_based`, `percentage` |
| `allocation_basis` | JSONB | Allocation basis definition (weights, tiers, usage metrics) |
| `target_business_units` | JSONB | Target BU distribution `[{bu_id, weight_or_pct}]` |
| `is_active` | BOOLEAN | Whether rule is active |
| `effective_from` | DATE | Rule effective start date |
| `effective_to` | DATE | Rule effective end date (nullable) |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

### `it_chargebacks`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `period_start` | DATE | Chargeback period start |
| `period_end` | DATE | Chargeback period end |
| `business_unit` | VARCHAR(100) | Target business unit |
| `entity_id` | UUID | Legal entity |
| `cost_pool_id` | UUID | Source cost pool |
| `chargeback_mode` | VARCHAR(20) | `chargeback`, `showback` |
| `direct_costs` | DECIMAL(18,2) | Direct cost amount |
| `indirect_costs` | DECIMAL(18,2) | Allocated indirect costs |
| `shared_service_costs` | DECIMAL(18,2) | Allocated shared service costs |
| `markup_amount` | DECIMAL(18,2) | Applied markup |
| `total_chargeback` | DECIMAL(18,2) | Total chargeback amount |
| `journal_entry_ids` | UUID[] | Posted journal entries (chargeback mode) |
| `status` | VARCHAR(20) | `calculated`, `reviewed`, `posted`, `disputed`, `adjusted` |
| `dispute_notes` | TEXT | Dispute resolution notes (nullable) |
| `line_items` | JSONB | Detailed breakdown by cost category |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

### `it_investments`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `name` | VARCHAR(300) | Investment name |
| `description` | TEXT | Investment description |
| `category` | VARCHAR(50) | `infrastructure`, `application`, `cloud_migration`, `security`, `compliance`, `innovation`, `modernization`, `other` |
| `status` | VARCHAR(20) | `proposed`, `under_review`, `approved`, `in_progress`, `completed`, `cancelled`, `on_hold` |
| `priority` | VARCHAR(10) | `critical`, `high`, `medium`, `low` |
| `business_sponsor` | UUID | Business unit sponsor |
| `it_owner` | UUID | IT project owner |
| `total_budget` | DECIMAL(18,2) | Total approved budget |
| `spent_to_date` | DECIMAL(18,2) | Actual spend to date |
| `committed_cost` | DECIMAL(18,2) | Committed (not yet invoiced) |
| `remaining_budget` | DECIMAL(18,2) | Available budget |
| `start_date` | DATE | Investment start date |
| `end_date` | DATE | Planned end date |
| `actual_end_date` | DATE | Actual completion date (nullable) |
| `projected_roi_pct` | DECIMAL(5,2) | Projected ROI percentage |
| `projected_npv` | DECIMAL(18,2) | Projected net present value |
| `projected_irr` | DECIMAL(5,2) | Projected internal rate of return |
| `projected_payback_months` | INTEGER | Projected payback period |
| `realized_benefits` | DECIMAL(18,2) | Actual benefits realized to date |
| `cost_center_id` | UUID | Cost center for investment |
| `project_id` | UUID | Linked project (nullable) |
| `approval_chain` | JSONB | Approval workflow history |
| `business_case` | JSONB | Business case details |
| `risk_assessment` | JSONB | Risk factors and mitigation |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

### `it_tco_analyses`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `name` | VARCHAR(300) | Analysis name |
| `asset_or_project` | VARCHAR(300) | Asset or project being analyzed |
| `analysis_type` | VARCHAR(30) | `build_vs_buy`, `onprem_vs_cloud`, `vendor_comparison`, `replacement`, `new_acquisition` |
| `analysis_horizon_years` | INTEGER | Number of years in analysis |
| `scenario` | VARCHAR(30) | `current`, `proposed`, `alternative` |
| `acquisition_cost` | DECIMAL(18,2) | One-time acquisition/implementation cost |
| `annual_operational_cost` | DECIMAL(18,2) | Annual operating cost |
| `annual_maintenance_cost` | DECIMAL(18,2) | Annual maintenance/support cost |
| `annual_personnel_cost` | DECIMAL(18,2) | Annual staffing cost |
| `annual_cloud_cost` | DECIMAL(18,2) | Annual cloud service cost |
| `end_of_life_cost` | DECIMAL(18,2) | Decommission/disposal cost |
| `total_tco` | DECIMAL(18,2) | Total cost of ownership over horizon |
| `npv` | DECIMAL(18,2) | Net present value |
| `irr` | DECIMAL(5,2) | Internal rate of return |
| `payback_months` | INTEGER | Payback period in months |
| `discount_rate` | DECIMAL(5,4) | Discount rate used for NPV |
| `annual_breakdown` | JSONB | Year-by-year cost breakdown |
| `assumptions` | JSONB | Analysis assumptions and notes |
| `status` | VARCHAR(20) | `draft`, `reviewed`, `approved`, `superseded` |
| `investment_id` | UUID | Linked investment (nullable) |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

### `it_vendor_contracts`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `vendor_id` | UUID | Vendor reference |
| `contract_name` | VARCHAR(300) | Contract name/description |
| `contract_type` | VARCHAR(30) | `saas`, `iaas`, `paas`, `license`, `managed_service`, `consulting`, `hardware`, `other` |
| `contract_number` | VARCHAR(100) | Contract reference number |
| `start_date` | DATE | Contract start date |
| `end_date` | DATE | Contract end date |
| `total_contract_value` | DECIMAL(18,2) | Total contract value |
| `annual_value` | DECIMAL(18,2) | Annual contract value |
| `committed_spend` | DECIMAL(18,2) | Committed spend (minimum) |
| `actual_spend_to_date` | DECIMAL(18,2) | Actual spend to date |
| `utilization_pct` | DECIMAL(5,2) | Contract utilization percentage |
| `auto_renewal` | BOOLEAN | Whether contract auto-renews |
| `renewal_notice_days` | INTEGER | Days before end date for renewal notice |
| `payment_terms` | VARCHAR(100) | Payment terms description |
| `cost_pool_id` | UUID | Assigned cost pool |
| `business_units` | JSONB | Business units consuming this contract |
| `sla_terms` | JSONB | Contract SLA terms |
| `status` | VARCHAR(20) | `active`, `pending_renewal`, `expired`, `terminated`, `renegotiating` |
| `attachments` | JSONB | Contract document references |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

### `it_budget_items`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `fiscal_year` | INTEGER | Fiscal year |
| `period` | VARCHAR(20) | Period (`annual`, `Q1`, `Q2`, `Q3`, `Q4`, `M01`-`M12`) |
| `cost_pool_id` | UUID | Cost pool reference |
| `category` | VARCHAR(50) | Budget category |
| `business_unit` | VARCHAR(100) | Business unit (nullable for IT-wide) |
| `budget_amount` | DECIMAL(18,2) | Budgeted amount |
| `actual_amount` | DECIMAL(18,2) | Actual amount |
| `committed_amount` | DECIMAL(18,2) | Committed but not yet spent |
| `remaining_budget` | DECIMAL(18,2) | Available budget |
| `variance_amount` | DECIMAL(18,2) | Budget - Actual variance |
| `variance_pct` | DECIMAL(5,2) | Variance percentage |
| `forecast_amount` | DECIMAL(18,2) | Forecasted full-period amount |
| `status` | VARCHAR(20) | `draft`, `approved`, `in_progress`, `closed` |
| `notes` | TEXT | Budget notes and explanations |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

## Business Rules

| Rule | Description |
|------|-------------|
| **BR-ITFM-001** | All IT costs must be classified into exactly one cost pool. Unassigned costs trigger an alert and are placed in an `unclassified` pool for review. |
| **BR-ITFM-002** | Direct costs are allocated to the consuming business unit without modification. Indirect costs are distributed based on the allocation key defined for the cost pool. |
| **BR-ITFM-003** | Chargeback journal entries are generated only for chargeback-mode cost pools. Showback items generate reports but no GL entries. Hybrid mode charges back direct costs and shows indirect costs. |
| **BR-ITFM-004** | Chargeback calculations must be reviewed and approved before journal posting. Auto-posting is permitted only if the chargeback variance from the prior period is < 10%. |
| **BR-ITFM-005** | IT investments exceeding the configurable approval threshold (default: $50,000) must go through the Workflow Service approval chain. Investments below threshold require only the IT owner's approval. |
| **BR-ITFM-006** | TCO analyses must span a minimum of 3 years and a maximum of 10 years. The discount rate defaults to the company's WACC but can be overridden per analysis. |
| **BR-ITFM-007** | Budget variance alerts are triggered when actual spend exceeds the configurable threshold (default: 10% of budget) for any cost pool period combination. Critical alerts (default: > 25%) notify the CFO. |
| **BR-ITFM-008** | Vendor contract utilization below 50% or above 95% triggers a review notification. Contracts with auto-renewal generate renewal review notices at the configured notice period (default: 90 days before expiry). |
| **BR-ITFM-009** | Cloud cost optimization recommendations are generated weekly. Anomalies exceeding 20% of the trailing 30-day average cost trigger immediate alerts. |
| **BR-ITFM-010** | Cost allocation runs are idempotent. Re-running allocation for a period in `calculated` status replaces the prior result. Posted allocations require reversal before recalculation. |
| **BR-ITFM-011** | ROI projections must be updated quarterly for all active investments. Investments with realized benefits < 50% of projected benefits at the midpoint trigger a review workflow. |
| **BR-ITFM-012** | All IT financial calculations, allocation changes, and chargeback postings are recorded with full audit trail: who, when, what changed, and why. |

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `finance.itfm.default_allocation_key` | `headcount` | Default allocation key for indirect cost pools |
| `finance.itfm.chargeback_autopost_variance_pct` | 10 | Max period-over-period variance for auto-posting (%) |
| `finance.itfm.investment_approval_threshold` | 50000 | Amount above which workflow approval is required |
| `finance.itfm.tco_min_years` | 3 | Minimum TCO analysis horizon |
| `finance.itfm.tco_max_years` | 10 | Maximum TCO analysis horizon |
| `finance.itfm.tco_default_discount_rate` | 0.08 | Default discount rate for NPV (8%) |
| `finance.itfm.budget_variance_warning_pct` | 10 | Budget variance % to trigger warning alert |
| `finance.itfm.budget_variance_critical_pct` | 25 | Budget variance % to trigger critical alert |
| `finance.itfm.contract_renewal_notice_days` | 90 | Days before contract expiry for renewal notice |
| `finance.itfm.cloud_cost_anomaly_threshold_pct` | 20 | Cloud cost anomaly detection threshold (%) |
| `finance.itfm.roi_review_trigger_pct` | 50 | Realized vs. projected benefits ratio triggering review |
| `finance.itfm.allocation_schedule` | `monthly` | Cost allocation run frequency |

## Cross-References

- [Finance Service](../06-services/finance.md) — Finance service specification
- [Budget Management](../06-services/finance.md) — IT budget integration and allocations
- [Project Service](../06-services/project.md) — Project costs and investment tracking
- [Commerce Service](../06-services/commerce.md) — Subscription billing and usage metrics
- [Workflow Engine](../06-services/workflow.md) — IT investment approval workflows
- [Report Service](../06-services/report.md) — IT financial dashboards and analytics
- [Architecture Overview](../01-architecture/overview.md) — System communication patterns
- [NFR: Performance](../10-planning/nfr.md) — Cost allocation processing requirements
