# Tax Reporting & Provision

Corporate tax provision, compliance, and country-by-country reporting, managed by the Finance Service.

## Overview

Tax Reporting & Provision provides comprehensive automation of the corporate tax lifecycle, from provision calculation through filing, audit, and regulatory reporting. The system automates current and deferred tax provision calculations based on real-time financial data, tracks effective tax rates across jurisdictions, generates tax journal entries, and produces OECD BEPS Action 13 compliant Country-by-Country Reports. It supports multi-jurisdiction tax consolidation, transfer pricing documentation, uncertain tax position tracking (FIN 48), and provides a structured workspace for tax return preparation with full audit trail.

## Data Flow Diagram

```
┌──────────────┐     ┌──────────────────┐     ┌─────────────────────────┐
│  GL / AP /   │────▶│  Tax Data        │────▶│  Tax Provision Engine    │
│  Payroll     │     │  Aggregation     │     │  ┌───────────────────┐  │
│  Transactions│     │  & Normalization │     │  │ Current Tax Calc  │  │
└──────────────┘     └──────────────────┘     │  │ (jurisdiction-    │  │
                                                │  │  specific rates)  │  │
┌──────────────┐     ┌──────────────────┐     │  └───────┬───────────┘  │
│  Revenue /   │────▶│  Temporary       │────▶│  ┌───────▼───────────┐  │
│  Balance     │     │  Difference      │     │  │ Deferred Tax Calc │  │
│  Sheet Data  │     │  Analysis        │     │  │ (asset/liability) │  │
└──────────────┘     └──────────────────┘     │  └───────┬───────────┘  │
                                                └──────────┼─────────────┘
                                                           │
                              ┌────────────────────────────┼───────────────┐
                              │                            │               │
                     ┌────────▼────────┐        ┌─────────▼──────┐  ┌────▼──────┐
                     │  Tax Journal    │        │  CbCR / TP     │  │  Tax Return│
                     │  Entries        │        │  Reports       │  │  Workspace │
                     └────────┬────────┘        └─────────┬──────┘  └────┬──────┘
                              │                           │              │
                              ▼                           ▼              ▼
                     ┌────────────────────────────────────────────────────────┐
                     │              Tax Audit Trail & Compliance               │
                     │  Provision → Journal → Return → Filing → Audit         │
                     └──────────────────────────┬─────────────────────────────┘
                                                │
                                                ▼
                     ┌────────────────────────────────────────────────────────┐
                     │              Tax Calendar & Compliance Tracking         │
                     │  Deadlines → Reminders → Filings → Certifications      │
                     └────────────────────────────────────────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Tax Provision Engine** | Finance Service (Rust) | Calculates current tax expense (jurisdiction-specific rates applied to taxable income) and deferred tax (temporary differences between book and tax basis). Supports multi-rate, multi-jurisdiction calculations with rate-change handling |
| **Effective Tax Rate Tracking** | Finance Service (Rust) | Computes ETR as `Total Tax Expense / Pre-Tax Income` with reconciliation from statutory rate to effective rate. Tracks quarterly and annual ETR with variance analysis |
| **Tax Journal Entry Generation** | Finance Service (Rust) | Automatic creation of tax-related journal entries: current tax expense/payable, deferred tax asset/liability, valuation allowances, tax effect of items in OCI |
| **Country-by-Country Reporting (CbCR)** | Finance Service (Rust) | Generates OECD BEPS Action 13 compliant CbCR: revenue, profit/loss, taxes paid/accrued, capital, earnings, number of employees, tangible assets per tax jurisdiction |
| **Transfer Pricing Documentation** | Finance Service (Rust) | Maintains transfer pricing documentation: master file, local file, entity-level intercompany transaction analysis, benchmarking data, APA tracking |
| **Tax Return Workspace** | Finance Service + Workflow Service | Structured workspace for tax return preparation: data collection, workpapers, adjustments, review/approval workflow, filing status tracking |
| **Tax Audit Trail** | Finance Service (Rust) | Immutable audit log of all tax calculations, assumptions, rate changes, manual adjustments, and approvals with full traceability to source transactions |
| **Multi-Jurisdiction Consolidation** | Finance Service (Rust) | Aggregates tax data across entities and jurisdictions for consolidated tax reporting. Handles currency translation, intercompany eliminations, and jurisdiction-specific rules |
| **Tax Calendar Management** | Finance Service + Platform Service | Configurable tax calendar with filing deadlines, estimated payment dates, extension dates per jurisdiction. Automated reminders and compliance status tracking |
| **Provision-to-Return Reconciliation** | Finance Service (Rust) | Reconciles tax provision estimates to actual tax return amounts. Tracks true-up adjustments and explains variances between provision and return |
| **Uncertain Tax Position Tracking (FIN 48)** | Finance Service (Rust) | Two-step analysis: recognition (more-likely-than-not threshold) and measurement (largest benefit amount with cumulative probability > 50%). Tracks reserves, settlements, and statute of limitations |
| **Storage** | PostgreSQL + MinIO | Tax data in PostgreSQL; supporting documents (tax returns, rulings, APAs) in MinIO |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| General Ledger | Finance Service | Internal API | GL account balances, journal entries, trial balance for tax basis computation |
| Accounts Payable | Finance Service | Internal API | Vendor invoices for input tax credits, withholding tax |
| Accounts Receivable | Finance Service | Internal API | Revenue data, output tax, intercompany transactions |
| Cash Management | Finance Service | Internal API | Tax payments executed, estimated payment scheduling |
| Payroll | HR Service | Event-driven | Payroll data for employment tax calculations, headcount per jurisdiction |
| Order Management | Commerce Service | Event-driven | Revenue data for nexus analysis, transaction-level tax detail |
| Period Close | Finance Service | Event-driven | Provision calculations triggered by period close events |
| Workflow Engine | Workflow Service | Event-driven | Tax return review/approval workflows, audit response workflows |
| Report Service | Report Service | Internal API | Tax analytics dashboards, provision trending, ETR analysis |
| Notification Service | Platform Service | Event-driven | Tax deadline reminders, filing confirmations, escalation alerts |
| Audit Trail | Platform Service | Event-driven | Full audit log of tax calculations and approvals |
| Config Service | Platform Service | Event-driven | Jurisdiction-specific tax rates, rules, and calendar configuration |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `finance.tax.provision.calculated` | `{provision_id, period_id, jurisdiction, current_tax, deferred_tax, effective_rate}` | Tax provision calculation completes for a period | GL, Dashboard, Report Service |
| `finance.tax.provision.posted` | `{provision_id, journal_entry_ids[], total_tax_expense, posted_at}` | Tax journal entries posted to GL | GL, Audit Trail, Dashboard |
| `finance.tax.return.filed` | `{return_id, jurisdiction, period, filing_date, filing_reference, status}` | Tax return filed with tax authority | Dashboard, Audit Trail, Calendar |
| `finance.tax.audit.initiated` | `{audit_id, jurisdiction, tax_years[], authority, initiated_at}` | Tax audit notification received | Workflow Engine, Notification Service |
| `finance.tax.cbc-report.generated` | `{report_id, fiscal_year, jurisdictions_count, generated_at}` | CbCR generation completes for fiscal year | Dashboard, Compliance, Report Service |
| `finance.tax.transfer-pricing.updated` | `{document_id, entity_id, intercompany_transactions[], updated_at}` | Transfer pricing documentation updated | Dashboard, Audit Trail |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `finance.journal.posted` | Finance Service | Ingest GL transactions for tax basis computation |
| `finance.period.closed` | Finance Service | Trigger provision calculation for closed period |
| `finance.invoice.approved` | Finance Service | Capture input tax credit data from vendor invoices |
| `finance.payment.completed` | Finance Service | Record tax payments, update payment schedules |
| `commerce.order.completed` | Commerce Service | Revenue data for transaction tax and nexus analysis |
| `hr.payroll.processed` | HR Service | Payroll data for employment tax and headcount |
| `workflow.step.approved` | Workflow Service | Process tax return review/approval decisions |
| `config.changed` | Platform Service | Reload jurisdiction tax rates, rules, calendar config |

## Data Model Reference

### `tax_provisions`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `period_id` | UUID | Fiscal period reference |
| `entity_id` | UUID | Legal entity |
| `jurisdiction` | VARCHAR(100) | Tax jurisdiction (ISO country code + subdivision) |
| `provision_type` | VARCHAR(20) | `current` or `deferred` |
| `taxable_income` | DECIMAL(18,2) | Taxable income for the period |
| `statutory_rate` | DECIMAL(5,4) | Statutory tax rate for jurisdiction |
| `tax_expense` | DECIMAL(18,2) | Computed tax expense |
| `deferred_tax_asset` | DECIMAL(18,2) | Net deferred tax asset |
| `deferred_tax_liability` | DECIMAL(18,2) | Net deferred tax liability |
| `effective_rate` | DECIMAL(5,4) | Effective tax rate for period |
| `status` | VARCHAR(20) | `draft`, `calculated`, `reviewed`, `posted`, `amended` |
| `journal_entry_ids` | UUID[] | Posted journal entries |
| `assumptions` | JSONB | Calculation assumptions and notes |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

### `tax_provision_details`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `provision_id` | UUID | Reference to `tax_provisions` |
| `line_type` | VARCHAR(50) | `permanent_difference`, `temporary_difference`, `rate_change`, `valuation_allowance`, `credit`, `other` |
| `description` | TEXT | Description of the line item |
| `book_amount` | DECIMAL(18,2) | Amount per accounting books |
| `tax_amount` | DECIMAL(18,2) | Amount per tax basis |
| `difference` | DECIMAL(18,2) | Book-to-tax difference |
| `tax_effect` | DECIMAL(18,2) | Tax effect of the difference |
| `jurisdiction` | VARCHAR(100) | Applicable jurisdiction |
| `sort_order` | INTEGER | Display order in provision workpaper |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

### `tax_jurisdictions`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `country_code` | VARCHAR(3) | ISO 3166-1 alpha-3 country code |
| `subdivision_code` | VARCHAR(10) | ISO 3166-2 subdivision code (nullable) |
| `name` | VARCHAR(200) | Jurisdiction display name |
| `statutory_rate` | DECIMAL(5,4) | Current statutory tax rate |
| `effective_from` | DATE | Rate effective date |
| `effective_to` | DATE | Rate expiry date (nullable) |
| `tax_authority` | VARCHAR(200) | Tax authority name |
| `filing_frequency` | VARCHAR(20) | `monthly`, `quarterly`, `annual` |
| `currency_code` | VARCHAR(3) | Local currency ISO code |
| `has_nexus_threshold` | BOOLEAN | Whether nexus threshold applies |
| `nexus_threshold_amount` | DECIMAL(18,2) | Revenue threshold for nexus |
| `config` | JSONB | Jurisdiction-specific rules and parameters |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

### `tax_returns`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `entity_id` | UUID | Legal entity filing the return |
| `jurisdiction` | VARCHAR(100) | Tax jurisdiction |
| `tax_year` | INTEGER | Tax year |
| `tax_period` | VARCHAR(20) | Period identifier (e.g., `Q1`, `annual`) |
| `return_type` | VARCHAR(50) | Type of tax return (e.g., `corporate_income`, `vat`, `withholding`) |
| `status` | VARCHAR(20) | `preparation`, `review`, `approved`, `filed`, `assessed`, `amended` |
| `taxable_income` | DECIMAL(18,2) | Reported taxable income |
| `tax_liability` | DECIMAL(18,2) | Computed tax liability |
| `payments_made` | DECIMAL(18,2) | Payments/credits applied |
| `balance_due` | DECIMAL(18,2) | Net amount due/refund |
| `filing_date` | DATE | Actual filing date |
| `due_date` | DATE | Filing deadline |
| `extension_date` | DATE | Extended deadline (nullable) |
| `filing_reference` | VARCHAR(100) | Authority-assigned filing reference |
| `assigned_to` | UUID | Tax team member responsible |
| `reviewed_by` | UUID | Reviewer |
| `approved_by` | UUID | Approver |
| `provision_id` | UUID | Linked provision (nullable) |
| `variance_analysis` | JSONB | Provision-to-return variance |
| `attachments` | JSONB | Supporting document references |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

### `tax_calendar_events`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `entity_id` | UUID | Legal entity |
| `jurisdiction` | VARCHAR(100) | Tax jurisdiction |
| `event_type` | VARCHAR(50) | `filing_due`, `estimated_payment`, `extension_due`, `audit_deadline`, `review_deadline`, `custom` |
| `title` | VARCHAR(300) | Event title |
| `description` | TEXT | Event details |
| `due_date` | DATE | Event due date |
| `remind_at` | TIMESTAMPTZ | Reminder trigger timestamp |
| `status` | VARCHAR(20) | `upcoming`, `in_progress`, `completed`, `overdue`, `cancelled` |
| `assigned_to` | UUID | Responsible team member |
| `tax_return_id` | UUID | Linked tax return (nullable) |
| `recurrence_rule` | VARCHAR(200) | iCal RRULE for recurring events (nullable) |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

### `transfer_pricing_documents`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `entity_id` | UUID | Legal entity |
| `document_type` | VARCHAR(30) | `master_file`, `local_file`, `benchmarking`, `apa`, `ruling` |
| `fiscal_year` | INTEGER | Applicable fiscal year |
| `jurisdiction` | VARCHAR(100) | Tax jurisdiction |
| `counterparty_entity_id` | UUID | Intercompany counterparty entity |
| `intercompany_transactions` | JSONB | Summary of intercompany transactions covered |
| `pricing_method` | VARCHAR(50) | Transfer pricing method (CUP, resale price, cost plus, TNMM, profit split) |
| `benchmark_data` | JSONB | Benchmarking analysis data |
| `status` | VARCHAR(20) | `draft`, `under_review`, `approved`, `filed`, `expired` |
| `approved_by` | UUID | Approver |
| `expiry_date` | DATE | Document expiry date (nullable) |
| `attachments` | JSONB | Supporting document references |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

### `tax_audit_trails`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `entity_id` | UUID | Legal entity |
| `audit_type` | VARCHAR(30) | `calculation`, `adjustment`, `rate_change`, `filing`, `authority_audit`, `review`, `approval` |
| `audit_id` | UUID | Reference to tax audit if applicable (nullable) |
| `reference_type` | VARCHAR(30) | `provision`, `return`, `journal_entry`, `cbc_report`, `tp_document` |
| `reference_id` | UUID | ID of the referenced entity |
| `action` | VARCHAR(100) | Action performed |
| `previous_value` | JSONB | Previous state (nullable) |
| `new_value` | JSONB | New state (nullable) |
| `performed_by` | UUID | User who performed the action |
| `performed_at` | TIMESTAMPTZ | When the action occurred |
| `ip_address` | INET | Client IP address |
| `notes` | TEXT | Additional context |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

### `uncertain_tax_positions`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `entity_id` | UUID | Legal entity |
| `jurisdiction` | VARCHAR(100) | Tax jurisdiction |
| `tax_year` | INTEGER | Tax year |
| `description` | TEXT | Description of the uncertain position |
| `position_type` | VARCHAR(50) | `deduction`, `credit`, `income_exclusion`, `nexus`, `entity_classification`, `other` |
| `recognition_threshold_met` | BOOLEAN | More-likely-than-not threshold met |
| `measurement_amount` | DECIMAL(18,2) | Largest benefit with >50% cumulative probability |
| `reserve_amount` | DECIMAL(18,2) | Tax reserve recorded |
| `probability_range` | JSONB | Probability-weighted range of outcomes |
| `status` | VARCHAR(20) | `identified`, `analyzing`, `reserved`, `settled`, `expired`, `reversed` |
| `settlement_amount` | DECIMAL(18,2) | Final settlement amount (nullable) |
| `statute_expiry_date` | DATE | Statute of limitations expiry |
| `assigned_to` | UUID | Tax team member responsible |
| `supporting_analysis` | JSONB | FIN 48 analysis documentation |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

### `cbc_reports`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `ultimate_parent_entity_id` | UUID | Ultimate parent entity |
| `fiscal_year` | INTEGER | Fiscal year covered |
| `jurisdiction` | VARCHAR(100) | Filing jurisdiction |
| `status` | VARCHAR(20) | `draft`, `review`, `approved`, `filed` |
| `entity_count` | INTEGER | Number of constituent entities |
| `total_revenue` | DECIMAL(18,2) | Aggregate unrelated + related revenue |
| `total_profit_loss` | DECIMAL(18,2) | Aggregate profit/(loss) before tax |
| `total_tax_paid` | DECIMAL(18,2) | Total taxes paid |
| `total_tax_accrued` | DECIMAL(18,2) | Total taxes accrued |
| `total_employees` | INTEGER | Aggregate headcount |
| `total_tangible_assets` | DECIMAL(18,2) | Aggregate tangible assets excluding cash |
| `jurisdictional_data` | JSONB | Per-jurisdiction breakdown (revenue, P&L, tax, capital, employees, assets) |
| `entity_data` | JSONB | Per-constituent-entity details (name, jurisdiction, main activities, tax residence) |
| `filing_date` | DATE | Actual filing date |
| `filing_reference` | VARCHAR(100) | Authority-assigned reference |
| `approved_by` | UUID | Approver |
| `attachments` | JSONB | Supporting document references |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

## Provision Calculation Model

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Tax Provision Calculation                             │
│                                                                          │
│  Current Tax Expense:                                                    │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  Taxable Income (per jurisdiction)                              │    │
│  │  × Statutory Rate                                               │    │
│  │  ± Permanent Differences (non-deductible items, tax credits)   │    │
│  │  = Current Tax Expense                                          │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                          │
│  Deferred Tax:                                                           │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  Temporary Differences (book basis ≠ tax basis)                 │    │
│  │  Taxable temp differences × rate = Deferred Tax Liability       │    │
│  │  Deductible temp differences × rate = Deferred Tax Asset        │    │
│  │  ± Valuation Allowance (if realization not more-likely-than-not)│    │
│  │  = Net Deferred Tax (Asset) / (Liability)                       │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                          │
│  Effective Tax Rate Reconciliation:                                      │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  Statutory Rate × Pre-Tax Income = Expected Tax                │    │
│  │  + Permanent Differences (state taxes, nondeductible, credits) │    │
│  │  + Rate Changes (mid-year rate changes, different jurisdictions)│    │
│  │  + Uncertain Tax Positions (FIN 48 reserves)                    │    │
│  │  = Actual Tax Expense → ETR = Actual / Pre-Tax Income           │    │
│  └─────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────┘
```

## Business Rules

| Rule | Description |
|------|-------------|
| **BR-TAX-001** | Tax provision must be calculated for every period before the period can be closed. Provision status must reach `posted` before close is permitted. |
| **BR-TAX-002** | Deferred tax assets must be evaluated for realizability each period. A valuation allowance is required if it is not more-likely-than-not that the DTA will be realized. |
| **BR-TAX-003** | All tax journal entries require dual approval (preparer + reviewer) before posting to the GL. System enforces segregation of duties. |
| **BR-TAX-004** | Tax rates are maintained per jurisdiction with effective dates. Rate changes mid-period are handled using the blended rate method for the transition period. |
| **BR-TAX-005** | CbCR must be generated for all MNE groups with consolidated group revenue exceeding EUR 750M in the prior fiscal year. |
| **BR-TAX-006** | Transfer pricing documentation must be maintained for all intercompany transactions exceeding the materiality threshold (configurable per tenant). |
| **BR-TAX-007** | Uncertain tax positions (FIN 48) must be evaluated using the two-step process: (1) recognition — more-likely-than-not the position will be sustained; (2) measurement — largest cumulative probability amount. |
| **BR-TAX-008** | Tax returns cannot be filed without completing all mandatory workpapers and receiving reviewer and approver sign-off via Workflow Service. |
| **BR-TAX-009** | Provision-to-return reconciliation must explain all variances exceeding the materiality threshold (default 5% of total tax expense). |
| **BR-TAX-010** | Tax calendar events must be generated automatically based on jurisdiction configuration. Overdue filings trigger automatic escalation to the tax director. |
| **BR-TAX-011** | All tax calculations, manual adjustments, and approvals are recorded in the immutable `tax_audit_trails` table with full before/after state capture. |
| **BR-TAX-012** | Intercompany transactions must be eliminated in the consolidated provision. Each entity's provision is calculated separately, then consolidated with eliminations. |
| **BR-TAX-013** | Statute of limitations dates must be tracked for all uncertain tax positions. Automatic notification sent 90 days before expiry. |
| **BR-TAX-014** | Tax provision calculations are idempotent. Recalculating a provision in `draft` status replaces the prior draft. Provisions in `posted` status require an amendment. |

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `finance.tax.provision.auto_calculate_on_close` | true | Trigger provision calculation when period closes |
| `finance.tax.provision.materiality_threshold_pct` | 5.0 | Variance threshold requiring explanation (%) |
| `finance.tax.provision.dual_approval_required` | true | Require preparer + reviewer on tax journal entries |
| `finance.tax.cbcr.revenue_threshold_eur` | 750000000 | MNE revenue threshold for CbCR filing (EUR) |
| `finance.tax.calendar.reminder_days_before` | [30, 14, 7, 1] | Days before deadline to send reminders |
| `finance.tax.calendar.escalation_days_overdue` | 3 | Days after deadline to escalate to tax director |
| `finance.tax.fin48.auto_evaluate_on_period_close` | true | Trigger FIN 48 evaluation on period close |
| `finance.tax.statute_expiry_warning_days` | 90 | Days before statute expiry to send warning |

## Cross-References

- [Intelligent Close](intelligent-close.md) — Tax provision gate in the financial close process
- [Account Reconciliation](reconciliation.md) — Provision-to-return reconciliation
- [Finance Service](../06-services/finance.md) — Finance service specification
- [Workflow Engine](../06-services/workflow.md) — Tax return review and approval workflows
- [Architecture Overview](../01-architecture/overview.md) — System communication patterns
- [NFR: Compliance](../10-planning/nfr.md) — Tax calculation latency and audit requirements
