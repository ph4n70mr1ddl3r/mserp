# Domain Data Models

## 12. Revenue Recognition Data Model

| Table / Field | Description |
|--------------|-------------|
| `revenue_contracts` | Customer contracts identified for revenue recognition, sourced from sales orders, subscriptions, and project billing |
| `performance_obligations` | Distinct performance obligations per contract, with standalone selling price allocation |
| `revenue_schedules` | Recognition schedule entries: period, amount, status (pending/recognized/deferred) |
| `contract_modifications` | Contract changes with prospective or retrospective adjustment method |
| `revenue_waterfall` | Aggregate view: bookings → deferred → recognized revenue with rollforward |

### Revenue Recognition Event Flow

| Event | Source | Consumer |
|-------|--------|----------|
| `commerce.subscription.created` | Commerce | Finance (create revenue contract) |
| `commerce.subscription.amended` | Commerce | Finance (adjust revenue schedule) |
| `commerce.subscription.billing.completed` | Commerce | Finance (recognize revenue for period) |
| `finance.revenue.contract.approved` | Finance | Report (update revenue waterfall) |
| `finance.revenue.recognized` | Finance | Report (update recognized revenue) |
| `finance.revenue.deferred` | Finance | Report (update deferred revenue) |
| `finance.revenue.adjusted` | Finance | Report (update schedule adjustments) |

- **Finance Service:** Manages revenue contracts, performance obligations, recognition schedules, and disclosure data.
- **Commerce Service:** Notifies Finance of subscription lifecycle events that trigger revenue recognition.
- **Report Service:** Displays deferred revenue by period and subscription; provides revenue waterfall and disclosure reports.

> **Note:** Revenue Recognition obligations are linked to subscription billing schedules. Revenue recognition events (e.g., `finance.revenue.recognized`) update the revenue waterfall and disclosure reports.

## 13. Lease Accounting Data Model

| Table / Field | Description |
|--------------|-------------|
| `lease_contracts` | Lease agreements with classification (operating/finance), terms, payment schedules, and linked assets |
| `lease_right_of_use_assets` | Right-of-use asset records with initial measurement, amortization schedule, and impairment tracking |
| `lease_liabilities` | Lease liability calculations with present value, incremental borrowing rate, and remeasurement history |
| `lease_payments` | Scheduled and actual lease payments with variable payment tracking |
| `lease_modifications` | Lease modification history with remeasurement details and reclassification entries |

## 14. Grant Management Data Model

| Table / Field | Description |
|--------------|-------------|
| `grants` | Grant registrations with funding source, award period, conditions, and budget |
| `grant_budgets` | Grant budget entries with allowable and unallowable cost categories |
| `grant_cost_allocations` | Cost allocations against grants with indirect cost rate calculations |
| `grant_revenue_schedules` | Revenue recognition schedules for milestone-based and cost-reimbursable grants |
| `grant_compliance_checks` | Compliance monitoring records with reporting deadlines and audit trail |

## 15. Joint Venture Data Model

| Table / Field | Description |
|--------------|-------------|
| `joint_ventures` | Joint venture definitions with partner ownership percentages and venturer codes |
| `jv_partners` | Joint venture partners with ownership percentage, billing preferences, and contact details |
| `jv_cost_pools` | Shared cost pools with allocation rules by partner ownership percentage |
| `jv_allocations` | Cost and revenue allocation records with partner-specific breakdowns |
| `jv_partner_billings` | Partner billing statements with detailed cost breakdowns and settlement tracking |

## 16. Warranty Management Data Model

| Table / Field | Description |
|--------------|-------------|
| `warranty_policies` | Warranty policy definitions with coverage terms, duration, and conditions |
| `warranty_registrations` | Product warranty registrations linked to serial/lot numbers and customers |
| `warranty_claims` | Warranty claim submissions with validation status, resolution, and cost tracking |
| `warranty_claim_items` | Claim line items with defect codes, repair/replacement actions, and costs |

## 17. Advanced Collections Data Model

| Table / Field | Description |
|--------------|-------------|
| `collection_strategies` | Configurable collection strategies with aging thresholds and action rules |
| `collection_activities` | Collection activity records (calls, emails, promises to pay) with outcomes |
| `cash_applications` | Cash receipt applications with ML-based invoice matching confidence scores |
| `collection_disputes` | Dispute tracking linked to collection activities with resolution workflows |
| `collection_scores` | Predictive collection scoring per customer with historical trending |

## 18. Data Masking Strategy

### 18.1 Masking for Non-Production Environments

| Technique | Description | When |
|-----------|-------------|------|
| Static masking | Irreversible masking applied to data copies for non-prod environments | On data subset extraction |
| Dynamic masking | Runtime masking based on user permissions (production) | Per query |
| Subsetting | Extract a representative subset of production data for testing | On demand |

### 18.2 Masking Rules by Data Classification

| Classification | Masking Rule | Example |
|---------------|-------------|---------|
| Public | None | Product names |
| Internal | Partial masking | `j***@company.com` |
| Confidential | Full masking / tokenization | `***-**-1234` (SSN) |
| Restricted | Suppression (replace with placeholder) | `[REDACTED]` |

---

*See [Data Architecture Overview](overview.md) for database patterns and schema elements.*
*See [Caching Strategy](caching.md) for cache architecture and policies.*
*See [Data Warehouse](warehouse.md) for analytical schema design.*
*See [Data Lake](data-lake.md) for raw and curated data zones.*
*See [Master Data Management Schema](mdm.md) for MDM and related schemas.*
