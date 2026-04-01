# Advanced Tax Filing

Automated tax return preparation, jurisdictional filing management, tax provision, and country-by-country reporting, managed by the Finance Service.

## Overview

Advanced Tax Filing automates the end-to-end tax compliance lifecycle for multi-jurisdictional enterprises. The module covers automated tax return preparation (federal, state, local) with form pre-population from transaction data, jurisdictional tax filing calendar management with deadline tracking, electronic filing integration with tax authorities, tax provision automation including ASC 740 compliance and deferred tax calculations, country-by-country reporting (CbCR) for multinational enterprises, and comprehensive tax audit trail and documentation management. Transaction tax data flows from AP/AR, sales tax from Commerce, and GL tax accounts provide the accounting backbone.

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Return Preparation** | Finance Service (Rust) | Automated tax form pre-population from GL tax accounts, AP/AR transaction data, and sales tax records. Supports federal, state, and local forms |
| **Filing Calendar** | Finance Service (Rust) | Jurisdiction-aware filing calendar with configurable frequency (monthly, quarterly, annually), deadline tracking, and reminder escalation |
| **E-Filing Integration** | Finance Service + Integration Service | Electronic submission to tax authorities via government connectors (IRS MeF, state portals). Status tracking and acknowledgment handling |
| **Tax Provision** | Finance Service (Rust) | ASC 740 current and deferred tax provision calculations, effective tax rate reconciliation, uncertain tax position (UTP) analysis |
| **CbCR Reporting** | Finance Service (Rust) | Country-by-country report generation per OECD BEPS Action 13 with revenue, profit, tax paid, capital, earnings, employees, and assets by jurisdiction |
| **Audit Documentation** | Finance Service (Rust) | Tax position documentation, work papers, supporting schedules, and audit trail linking transactions to tax line items |
| **Storage** | PostgreSQL + MinIO | Tax returns, provisions, and filing data in PostgreSQL; supporting documents and work papers in MinIO |

## Business Rules

| Rule | Description |
|------|-------------|
| **BR-ATF-001** | Tax returns must be pre-populated from verified GL data. Manual overrides require documentation and manager approval. |
| **BR-ATF-002** | Filing deadlines are tracked per jurisdiction. Returns not submitted 5 business days before deadline trigger escalation to tax director. |
| **BR-ATF-003** | Deferred tax assets and liabilities must reconcile to the GL trial balance. Variance exceeding 0.5% requires investigation. |
| **BR-ATF-004** | CbCR must be filed for multinational groups with consolidated revenue exceeding the configured threshold (default EUR 750M) within 12 months of fiscal year-end. |
| **BR-ATF-005** | All tax positions with uncertainty must be documented with a more-likely-than-not analysis and recorded as UTPs if the threshold is not met. |
| **BR-ATF-006** | E-filing submissions are idempotent. Duplicate submissions are prevented via Idempotency-Key scoped to filing period and jurisdiction. |

## Data Model

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `tax_returns` | `id, tenant_id, jurisdiction, tax_type, filing_period, form_code, data{}, status, filed_at, filing_reference, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to jurisdiction, has `tax_return_lines` |
| `filing_calendars` | `id, tenant_id, jurisdiction, tax_type, frequency, due_date, period_start, period_end, status, assigned_to, created_at, updated_at, created_by, updated_by, version, is_deleted` | Triggers `tax_returns` |
| `tax_provisions` | `id, tenant_id, fiscal_year, jurisdiction, current_tax_expense, deferred_tax_expense, effective_rate, reconciliation{}, utp_entries{}, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | References GL accounts |
| `cbcr_reports` | `id, tenant_id, fiscal_year, country, revenue, profit_before_tax, tax_paid, tax_accrued, capital, retained_earnings, employees, tangible_assets, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | One per country per year |
| `tax_audit_trails` | `id, tenant_id, return_id, line_item, source_transaction_ids[], source_gl_ids{}, amount, tax_treatment, documentation_ref, created_at, updated_at, version, is_deleted` | Links returns to source data |
| `tax_authority_submissions` | `id, tenant_id, return_id, authority_code, submission_ref, status, submitted_at, acknowledged_at, response{}, created_at, updated_at, version, is_deleted` | Belongs to return |

## Events

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `finance.tax-return.prepared` | `{return_id, jurisdiction, filing_period, form_code, total_tax}` | Return pre-population completed | Notification, Dashboard |
| `finance.tax-return.submitted` | `{return_id, jurisdiction, filing_reference, submitted_at}` | E-filing submitted to authority | Notification, Audit |
| `finance.tax-return.acknowledged` | `{return_id, authority_response, acceptance_status}` | Authority acknowledgment received | Notification, Dashboard |
| `finance.tax-provision.calculated` | `{provision_id, fiscal_year, current_tax, deferred_tax, effective_rate}` | Provision calculation completed | GL, Report |
| `finance.cbcr.generated` | `{cbcr_id, fiscal_year, country_count, total_revenue}` | CbCR report generated | Notification, Compliance |
| `finance.filing-deadline.approaching` | `{calendar_id, jurisdiction, due_date, days_remaining}` | Deadline reminder triggered | Notification, Workflow |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `finance.gl-period.closed` | Finance Service | Trigger tax return pre-population for closed period |
| `finance.invoice.posted` | Finance Service | Accumulate transaction tax data for return preparation |
| `finance.payment.completed` | Finance Service | Update tax paid amounts for provision and CbCR |
| `commerce.order.completed` | Commerce Service | Accumulate sales tax data by jurisdiction |
| `integration.tax-authority.response` | Integration Service | Process e-filing acknowledgment |
| `config.changed` | Config Service | Update filing calendars, rates, thresholds |

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/v1/tax/filing-calendar` | List upcoming filing obligations |
| POST | `/api/v1/tax/returns/prepare` | Pre-populate a tax return for a period |
| GET | `/api/v1/tax/returns` | List tax returns with jurisdiction filter |
| GET | `/api/v1/tax/returns/{id}` | Retrieve return with line items |
| POST | `/api/v1/tax/returns/{id}/submit` | E-file a return to tax authority |
| POST | `/api/v1/tax/provisions/calculate` | Run tax provision for a fiscal year |
| GET | `/api/v1/tax/provisions/{id}` | Retrieve provision detail with reconciliation |
| POST | `/api/v1/tax/cbcr/generate` | Generate country-by-country report |
| GET | `/api/v1/tax/audit-trail/{return_id}` | Retrieve audit documentation for a return |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| General Ledger | Finance Service | Internal API | Tax account balances, trial balance data |
| AP / AR | Finance Service | Internal API | Transaction tax data accumulation |
| Sales Tax (Commerce) | Commerce Service | Event-driven | Jurisdictional sales tax data for returns |
| Tax Authority Connectors | Integration Service | Connector | E-filing submission and acknowledgment |
| Report Service | Report Service | Internal API | Tax analytics, effective rate trends |
| Notification | Platform Service | Event-driven | Filing deadline reminders, submission confirmations |
| Compliance Hub | Platform Service | Event-driven | Tax compliance status and audit readiness |

## Cross-References

- [Tax Reporting](tax-reporting.md) — Standard tax reporting and analytics
- [Intelligent Close](intelligent-close.md) — Period close triggers tax return preparation
- [Finance Service](../06-services/finance.md) — Finance service specification
- [Integration Service](../06-services/integration.md) — Tax authority connectors
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
