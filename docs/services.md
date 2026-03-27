# Microservices Breakdown

## 1. Core Services (Infrastructure)

### 1.1 Auth Service

| Aspect | Details |
|--------|---------|
| Port | 8001 |
| Database | `auth_db` |
| Responsibilities | Authentication, token management, session handling |
| Features | OAuth2/OIDC provider, MFA (TOTP, SMS, email, biometric), SSO (SAML 2.0 bridge), password policies (complexity, rotation, history), JWT issuance and rotation, brute-force detection, account lockout, step-up authentication |

**Session Management:**
- Concurrent session control: configurable max sessions per user (default: 5)
- Session revocation: individual or all sessions (e.g., on password change)
- Idle timeout: configurable per tenant (default: 30 minutes)
- Device binding: optional device fingerprint for session validation

### 1.2 Identity Service

| Aspect | Details |
|--------|---------|
| Port | 8002 |
| Database | `identity_db` |
| Responsibilities | User management, roles, permissions, organizations |
| Features | User CRUD with lifecycle (active, suspended, deactivated), RBAC + ABAC, user groups / teams, organization hierarchy (departments, divisions, business units), user preferences (locale, timezone, notification preferences), delegated administration, API key management for service integrations |

**Organizational Hierarchy:**
```
Organization
  └── Business Unit
       └── Department
            └── Team
                 └── User
```

- Users belong to one or more teams.
- Roles are assigned at the organization, business unit, or department level.
- Row-level security can filter data by the user's organizational scope.

### 1.3 Tenant Service

| Aspect | Details |
|--------|---------|
| Port | 8003 |
| Database | `tenant_db` |
| Responsibilities | Multi-tenancy, tenant provisioning, feature flags |
| Features | Tenant lifecycle (provisioning, active, suspended, decommissioned), subscription tier management, feature toggles with rollout strategies, tenant-specific branding, tenant quota enforcement (users, storage, API calls), data residency controls (region pinning per tenant) |

**Tenant Quotas:**

| Resource | Standard | Professional | Enterprise |
|----------|----------|-------------|------------|
| Users | 50 | 500 | Unlimited |
| Storage | 10 GB | 100 GB | 1 TB |
| API calls/min | 1,000 | 10,000 | 100,000 |
| Custom integrations | 3 | 25 | Unlimited |
| Custom applications (low-code) | 1 | 10 | Unlimited |
| Data warehouse storage | 5 GB | 50 GB | 500 GB |

### 1.4 Config Service

| Aspect | Details |
|--------|---------|
| Port | 8004 |
| Database | `config_db` |
| Responsibilities | System configuration, settings management |
| Features | Dynamic config with hot reload, environment variable overrides (internal only, not API-exposed), CORS origin management, localization string management, scheduled maintenance windows |

#### Configuration Schema

Configuration is hierarchical, typed, and scoped per tenant with fallback to global defaults.

**Configuration Hierarchy (highest to lowest precedence):**

```
1. Environment variables (MSERP_{SECTION}_{KEY}) — internal overrides, not exposed via API
2. Tenant-specific overrides (config_db)
3. Global defaults (config_db)
4. Compiled-in defaults (Rust code)
```

**Configuration Categories:**

| Category | Scope | Examples |
|----------|-------|---------|
| System | Global | `system.max_upload_size_mb`, `system.default_locale`, `system.maintenance_mode` |
| Service | Global | `commerce.order.auto_cancel_hours`, `finance.fiscal_year_start` |
| Tenant | Per-tenant | Overrides of service/system config per tenant |
| Localization | Per-tenant | `i18n.date_format`, `i18n.number_format`, `i18n.timezone` |
| ESG | Per-tenant | `esg.reporting_framework`, `esg.carbon_offset_enabled`, `esg.emission_factors_source` |

**Config Propagation:**
- Services poll Config Service every 30 seconds for changes (config key: `config.poll_interval_seconds`).
- Critical config changes (e.g., `system.maintenance_mode`) are pushed immediately via RabbitMQ event `config.changed`.
- Services cache config in-memory with a TTL matching the poll interval.
- Config changes are immutable — every change is append-only in the audit log.

---

## 2. Business Services

### 2.1 Commerce Service (Sales + Inventory)

| Aspect | Details |
|--------|---------|
| Port | 8010 |
| Database | `commerce_db` |
| Responsibilities | Sales operations, customer management, stock management, warehousing, pricing, PIM, transportation |
| Rationale | See [Architecture — Service Consolidation](architecture.md#41-commerce-service-sales--inventory) |

**Sales Modules:**

| Module | Description |
|--------|-------------|
| Customers | Customer CRUD, credit limits, customer groups, pricing tier assignment, customer 360 view |
| Leads & Opportunities | Lead capture, qualification, opportunity pipeline with stages |
| Quotations | Quote creation, versioning, approval, acceptance/expiry workflow |
| Sales Orders | Order creation from quote or standalone, hold management, backorder handling |
| Deliveries & Shipments | Delivery scheduling, shipment tracking, proof of delivery |
| Sales Invoices | Invoice generation from delivery, credit memos, dunning management |
| Commissions | Commission rules, tiered rates, split commissions, payout calculation |
| Returns & Refunds | Return merchandise authorization (RMA), refund processing, restocking |
| Sales Analytics | Pipeline analysis, win/loss ratios, forecast accuracy, territory performance |

**Inventory Modules:**

| Module | Description |
|--------|-------------|
| Products / SKUs | Product hierarchy (category → subcategory → product → variant), attributes, multi-UoM |
| Stock Levels | Real-time stock across warehouses, safety stock, reorder points |
| Warehouses | Warehouse definition, zones, bins, pick/put strategies |
| Stock Transfers | Inter-warehouse transfers, in-transit tracking, receipt confirmation |
| Stock Valuation | FIFO, LIFO, weighted average, standard cost methods (per product) |
| Serial / Batch Tracking | Full traceability, lot numbers, expiry tracking, recall management |
| Reorder Automation | Min/max, EOQ, demand-driven reorder suggestions |
| Cycle Count | Scheduled and ad-hoc cycle counts, variance analysis |

**Product Information Management (PIM / Product Hub):**

| Module | Description |
|--------|-------------|
| Product Data Model | Flexible attribute schema with configurable attribute groups per product category |
| Category Management | Multi-level product categories with inheritance of attributes |
| Media Management | Product images, videos, documents with multi-format support |
| Channel Publishing | Publish product data to multiple channels (web, mobile, marketplace, EDI) |
| Data Quality | Completeness checks, mandatory attribute validation, enrichment workflows |
| Cross-Reference | Map internal SKUs to supplier, customer, and marketplace product identifiers |

**Transportation & Fleet Management:**

| Module | Description |
|--------|-------------|
| Carrier Management | Carrier registration, service level agreements, rate tables |
| Shipment Planning | Optimal carrier selection based on cost, transit time, and service level |
| Route Optimization | Route planning for multi-stop deliveries, fuel cost estimation |
| Freight Costing | Freight charge calculation, bill of lading generation, freight audit |
| Proof of Delivery | Digital POD capture, signature, photo, GPS coordinates |
| Carrier Performance | On-time delivery tracking, damage rate, carrier scorecards |

**Advanced Pricing Engine:**

| Pricing Component | Description |
|-------------------|-------------|
| Price Lists | Multiple price lists by currency, customer group, date range |
| Discount Rules | Percentage, fixed amount, buy-X-get-Y, volume/tiered discounts |
| Promotions | Time-limited promotions with coupon codes, usage limits |
| Price Formulas | Calculated pricing based on cost + margin, competitor price, or custom formula |
| Customer-Specific Pricing | Override price lists per customer or customer group |
| Multi-Currency Pricing | Automatic conversion or fixed prices per currency |
| Margin Analysis | Real-time margin calculation at line and order level |

### 2.2 Finance Service (Finance + Procurement)

| Aspect | Details |
|--------|---------|
| Port | 8011 |
| Database | `finance_db` |
| Responsibilities | Financial accounting, purchasing, supplier management, treasury, EPM, CLM, expenses |
| Rationale | See [Architecture — Service Consolidation](architecture.md#42-finance-service-finance--procurement) |

**Finance Modules:**

| Module | Description |
|--------|-------------|
| Chart of Accounts | Multi-level account hierarchy, account segments, consolidation mapping |
| Journal Entries | Manual and automated journal entries, reversal, recurring entries |
| General Ledger | Periodic posting, trial balance, automatic posting from subledgers |
| Accounts Payable | Vendor invoices, payment batches, payment terms, early payment discounts |
| Accounts Receivable | Customer invoices, receipt application, dunning, collections management |
| Fixed Assets | Asset registration, depreciation (straight-line, declining balance, units-of-production), disposal, revaluation |
| Budgeting | Budget entry by account/period, budget vs. actual, budget approval workflow, multi-year budgets, rolling forecasts |
| Tax Management | Tax codes, tax rates, tax jurisdictions, automatic tax calculation, tax reporting (VAT, GST, sales tax) |
| Multi-Currency | Real-time exchange rates, triangulation, realized/unrealized gains/losses, revaluation |
| Cash Management | Cash position, bank reconciliation, cash flow forecasting |
| Intercompany | Intercompany transactions, elimination entries, consolidated reporting |
| Financial Reports | Balance sheet, P&L, cash flow statement, custom financial reports |
| Cost Centers | Cost center hierarchy, profit center allocation, overhead distribution |
| Period Management | Open/close accounting periods, soft-close (prevent modifications), hard-close (audit lock) |

**Corporate Treasury:**

| Module | Description |
|--------|-------------|
| Cash Position | Real-time cash position across all bank accounts and currencies |
| Bank Integration | Bank statement import (ISO 20022, OFX, BAI2), automatic reconciliation |
| Payment Formats | Multiple payment file formats (SEPA, ACH, SWIFT, local formats) |
| Debt Management | Loan tracking, interest calculation, amortization schedules |
| Investment Management | Short-term investment tracking, yield calculation |
| FX Exposure | Foreign currency exposure tracking, hedging recommendations |
| In-House Banking | Intercompany netting, cash pooling, virtual account management |

**Enterprise Expense Management:**

| Module | Description |
|--------|-------------|
| Expense Reports | Expense report creation, itemization, receipt attachment |
| Per Diem | Configurable per diem rates by location and employee grade |
| Mileage Tracking | GPS-based mileage capture, configurable reimbursement rates |
| Corporate Cards | Corporate credit card transaction import, auto-matching to expenses |
| Approval Workflow | Multi-level approval with policy enforcement (amount, category, compliance) |
| Policy Compliance | Automatic policy violation detection, flagging, exception handling |
| Tax Recovery | VAT/GST recovery on eligible expenses |
| Expense Analytics | Spend by category, policy violation trends, reimbursement timing |

**Enterprise Performance Management (EPM):**

| Module | Description |
|--------|-------------|
| Planning | Top-down and bottom-up planning by account, department, project |
| Forecasting | Rolling forecasts with driver-based models, statistical methods |
| What-If Scenarios | Scenario modeling with side-by-side comparison |
| Consolidation | Multi-entity financial consolidation with currency translation adjustments |
| Allocations | Rule-based allocations across dimensions (cost center, product, geography) |
| Variance Analysis | Actual vs. plan/budget/forecast variance reporting with drill-down |
| Scorecards | Financial scorecards with KPIs, targets, and traffic-light indicators |

**Contract Lifecycle Management (CLM):**

| Module | Description |
|--------|-------------|
| Contract Authoring | Template-based contract creation with clause library |
| Clause Library | Reusable contract clauses with version control and approval status |
| Negotiation | Redline editing, version comparison, term tracking |
| Approval Workflow | Multi-level contract approval with legal review routing |
| Electronic Signatures | Integration with e-signature providers (DocuSign, Adobe Sign) |
| Obligation Tracking | Milestone and deliverable tracking against contract terms |
| Renewal Management | Automatic renewal alerts, renegotiation workflows |
| Contract Analytics | Contract value analysis, renewal rates, obligation compliance |

**Procurement Modules:**

| Module | Description |
|--------|-------------|
| Suppliers | Supplier registration, qualification, classification, bank details, tax information |
| Purchase Requisitions | Internal request for goods/services, approval workflow |
| RFQ (Request for Quotation) | Send RFQ to multiple suppliers, compare bids, award |
| Purchase Orders | PO creation from requisition or standalone, blanket orders, release orders |
| Goods Receipt | Receive against PO, inspection, put-away, quality hold |
| Supplier Evaluation | Scorecard (quality, delivery, price, responsiveness), periodic review |
| Contracts | Supplier contracts, terms, renewal tracking, compliance |
| Self-Service Procurement | Catalog-based purchasing for non-procurement users |
| Supplier Collaboration | Supplier portal for RFQ response, PO acknowledgment, invoice submission, delivery scheduling |

### 2.3 HR Service

| Aspect | Details |
|--------|---------|
| Port | 8012 |
| Database | `hr_db` |
| Responsibilities | Human resources, payroll, workforce management, talent management |

| Module | Description |
|--------|-------------|
| Employees | Employee lifecycle (hire, transfer, promote, separate), employment contracts, job grades |
| Attendance | Check-in/out, time tracking, overtime, timesheet approval, geolocation tracking |
| Leave Management | Leave types, accrual rules, leave balances, calendar integration, team visibility |
| Payroll | Payroll runs, salary structures, deductions, benefits, tax withholdings, payslip generation, bank file export |
| Multi-Country Payroll | Country-specific payroll localizations (tax rules, social contributions, statutory deductions), multiple payroll calendars, regulatory compliance per jurisdiction |
| Recruitment | Job requisitions, job postings, applicant tracking, interview scheduling, offer management |
| Onboarding | Onboarding checklists, document collection, IT provisioning requests |
| Performance Reviews | Review cycles (annual, quarterly, 360°), goals/KPIs, competency frameworks, calibration |
| Talent Review & Succession | Talent pools, nine-box grid, succession plans for key positions, readiness assessment |
| Workforce Modeling | Headcount planning simulations, scenario analysis, cost impact modeling |
| Training | Training catalog, enrollment, completion tracking, certifications, skills matrix |
| Org Chart | Organization hierarchy, position management, reporting structures, headcount planning |
| Compensation | Salary surveys, compa-ratio analysis, merit planning, variable pay |
| Benefits Administration | Benefit plans, enrollment windows, dependents, cost sharing |
| HR Analytics | Turnover analysis, time-to-fill, diversity metrics, workforce planning |

### 2.4 Manufacturing Service

| Aspect | Details |
|--------|---------|
| Port | 8013 |
| Database | `manufacturing_db` |
| Responsibilities | Production operations, quality management, cost accounting, PLM, EAM |

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

### 2.5 Report Service

| Aspect | Details |
|--------|---------|
| Port | 8014 |
| Database | `report_db` (materialized views, pre-aggregated data); reads from other services' event streams |
| Responsibilities | Analytics, reporting, dashboards, BI, AI-driven insights, ESG, carbon accounting |

| Module | Description |
|--------|-------------|
| Report Builder | Drag-and-drop report designer, custom SQL queries, parameterized reports |
| Scheduled Reports | Automated report generation and delivery (email, file share), subscription management |
| Dashboards | Real-time dashboards, widget library, drill-down/drill-through, personal and shared dashboards |
| KPIs | KPI definition, target setting, trend analysis, traffic-light alerts |
| Data Export | CSV, Excel, PDF, Parquet export; scheduled exports; API-driven data extraction |
| BI Integration | ODBC/JDBC bridge, embedded analytics, data warehouse sync |
| AI Insights | Anomaly detection (spend, inventory, revenue), demand forecasting, intelligent recommendations |
| Data Warehouse | Periodic ETL from service databases to DuckDB-based embedded warehouse; star schema with fact/dimension tables |
| ESG / Sustainability Reporting | Emissions tracking dashboards (Scope 1, 2, 3), ESG scorecards, regulatory report generation (GRI, SASB, TCFD, EU Taxonomy) |
| Carbon Accounting | Carbon footprint per product/service, emissions intensity metrics, carbon offset tracking, science-based target progress |
| Embedded ML/AI Platform | Model training and deployment pipeline, feature store, model registry, A/B testing for ML models |
| Predictive Planning | Demand forecasting models, revenue prediction, resource utilization forecasting, what-if scenario modeling |
| Data Lake Integration | Raw data ingestion to data lake, curated data layers, schema-on-read for exploratory analytics |

**AI/ML Capabilities:**

| Capability | Description |
|-----------|-------------|
| Anomaly Detection | Statistical detection of outliers in financial transactions, inventory levels, and sales patterns |
| Demand Forecasting | Time-series forecasting using moving averages, exponential smoothing, and ML models; fed back to Commerce for reorder suggestions |
| Spend Analysis | Automatic categorization of expenses, vendor spend concentration, contract compliance |
| Churn Prediction | Customer behavior analysis identifying at-risk accounts (CRM integration) |
| Cash Flow Forecasting | Predictive cash flow based on historical patterns, outstanding receivables, and payables |
| Sentiment Analysis | NLP-based analysis of customer feedback, support tickets, and survey responses |
| Fraud Detection | Pattern-based fraud detection in financial transactions, expense reports, and procurement |
| Yield Optimization | ML-driven production yield optimization for manufacturing |

### 2.6 Workflow Service

| Aspect | Details |
|--------|---------|
| Port | 8015 |
| Database | `workflow_db` |
| Responsibilities | Business process automation, approvals, BPM engine, SLA management |

| Module | Description |
|--------|-------------|
| Workflow Engine | BPMN 2.0-compatible engine, parallel/sequential gateways, subprocesses, timers |
| Approval Chains | Multi-level approval, sequential/parallel approvers, proxy/delegate approvers |
| Escalations | Time-based escalation, skip-level escalation, auto-approve on timeout |
| Process Templates | Pre-built templates (purchase approval, leave approval, expense approval, pricing approval, ECO approval, contract approval) |
| SLA Management | SLA definitions per workflow type, deadline tracking, breach notifications |
| Business Rules | Rule engine for conditional routing (e.g., amount-based, department-based) |
| Process Monitoring | Real-time process instance tracking, bottleneck analysis, audit trail |
| Email Approvals | Approve/reject via email reply (for simple single-step approvals) |
| Segregation of Duties (SoD) | SoD rule definitions, conflict detection before workflow approval, SoD violation reporting |

### 2.7 CRM / Marketing Service

| Aspect | Details |
|--------|---------|
| Port | 8016 |
| Database | `crm_db` |
| Responsibilities | Customer relationship management, marketing automation, customer analytics |

| Module | Description |
|--------|-------------|
| Contact Management | Contact CRUD, segmentation, tagging, activity history |
| Lead Management | Lead capture (web forms, imports, API), scoring, qualification, assignment rules |
| Deal Pipeline | Opportunity stages, probability, expected close date, competitor tracking |
| Campaigns | Email campaigns, audience segmentation, A/B testing, open/click tracking |
| Marketing Automation | Drip sequences, trigger-based actions, lead nurturing workflows |
| Customer Analytics | Customer lifetime value, acquisition cost, retention rate, NPS tracking |
| Account Management | Key account identification, relationship mapping, strategic account plans |
| Activity Tracking | Calls, emails, meetings, tasks, notes linked to contacts/deals |
| Customer Service | Case management, SLA tracking, knowledge base, escalation workflows |
| Social Listening | Social media mention tracking, sentiment analysis integration |

### 2.8 Project Management Service

| Aspect | Details |
|--------|---------|
| Port | 8017 |
| Database | `project_db` |
| Responsibilities | Project planning, execution, resource management, project accounting |

| Module | Description |
|--------|-------------|
| Project Planning | Project creation, WBS (Work Breakdown Structure), task dependencies, Gantt chart |
| Resource Allocation | Resource requests, skill matching, utilization tracking, overallocation alerts |
| Time & Expense | Timesheets, expense reports, approval workflows, receipt attachment |
| Milestones | Milestone definition, tracking, customer sign-off |
| Project Billing | Fixed-price, time-and-materials, milestone-based billing, revenue recognition |
| Budget & Costing | Project budgets, cost tracking (labor, material, overhead), EVM (Earned Value Management) |
| Risk Management | Risk identification, assessment matrix, mitigation plans, issue tracking |
| Project Templates | Pre-built templates for common project types (implementation, consulting, internal) |
| Project Analytics | Project health dashboards, resource utilization, margin analysis, portfolio view |
| Program Management | Program-level oversight, cross-project dependency tracking, program dashboards |

---

## 3. Supporting Services

### 3.1 Platform Service (Notification + File + Audit + Digital Assistant + App Builder + GRC)

| Aspect | Details |
|--------|---------|
| Port | 8020 |
| Database | `platform_db` (notifications + file metadata), `audit_db` (time-series optimized — see [Data Architecture](data-architecture.md)) |
| Responsibilities | Multi-channel notifications, file storage, document management, audit logging, digital assistant, low-code application builder, GRC |
| Rationale | See [Architecture — Service Consolidation](architecture.md#43-platform-service-notification--file--audit) |

| Module | Description |
|--------|-------------|
| Email Notifications | SMTP integration, templated emails, delivery tracking, bounce handling |
| SMS Notifications | SMS gateway integration (Twilio, etc.), delivery receipts |
| Push Notifications | Mobile push (FCM, APNs), web push |
| In-App Notifications | Real-time notifications via WebSocket, read/unread status, grouping |
| Webhooks | Outbound webhook delivery with retry, HMAC signing, event filtering |
| File Upload/Download | Chunked uploads, virus scanning, image transformation, thumbnail generation |
| Document Metadata | File metadata, tags, versioning, access control |
| Document Preview | In-browser preview for PDF, images, Office documents (via MinIO + LibreOffice conversion) |
| OCR | Optical character recognition for scanned documents, invoice capture |
| Immutable Audit Logs | Append-only audit trail, query by entity/user/action/time range |
| Compliance Reports | SOX audit trail, GDPR data access logs, HIPAA access logs |
| Data Lineage | Track data flow across services, lineage graphs for regulatory reporting |
| Digital Assistant | NLP-based conversational interface, intent recognition, entity extraction, multi-turn dialog, context management, action dispatch to service APIs |
| Application Builder | Visual page designer, custom business objects, drag-and-drop form builder, workflow integration, tenant-scoped publishing with versioning |
| GRC Controls | Segregation of Duties (SoD) rules, compliance policy management, risk register, control assessments, incident management |
| Data Masking | Dynamic and static data masking rules, PII subsetting for non-production environments, masking templates by compliance framework |

### 3.2 Integration Service

| Aspect | Details |
|--------|---------|
| Port | 8021 |
| Database | `integration_db` |
| Responsibilities | External integrations, API management, connector framework, data import/export, MDM, data governance |

| Module | Description |
|--------|-------------|
| Connectors | Pre-built connectors: Stripe, PayPal, QuickBooks, Xero, Salesforce, HubSpot, Slack, Teams, government tax portals, shipping carriers (FedEx, UPS, DHL) |
| Connector SDK | Rust-based plugin framework for building custom connectors |
| Webhooks | Inbound webhook handling with signature verification, retry policy |
| EDI | EDIFACT, X12 support for B2B document exchange (POs, invoices, ASN) |
| API Versioning | External API gateway with version management, deprecation notices |
| Data Import | File-based import (CSV, JSON, Excel, XML), field mapping, validation, error reporting |
| Data Export | Scheduled exports, on-demand exports, FTP/SFTP delivery, Parquet for data lake |
| Integration Monitoring | Connection health, sync status, error rates, data volume tracking |
| OAuth2 Client | OAuth2 client credentials for connecting to external APIs |
| Master Data Management (MDM) | Golden record management, automated deduplication, data quality rules, matching engine, data stewardship workflows |
| Data Governance | Data catalog, lineage tracking, quality scorecards, classification and tagging, retention policies |

---

## 4. Feature Flag Implementation

Feature flags are managed by the Tenant Service and evaluated at the **API Gateway** layer (primary) and **service** layer (secondary).

### 4.1 Flag Types

| Type | Description | Evaluation |
|------|-------------|------------|
| Boolean | Simple on/off toggle | Gateway |
| Variant | A/B testing, gradual rollouts (e.g., `new_ui: { control: 80, treatment: 20 }`) | Gateway |
| Percentage Rollouts | Incremental rollout by percentage | Gateway |
| Tenant Whitelist | Enable for specific tenant IDs | Gateway |
| Schedule | Enable/disable at a specified future date/time | Gateway |

### 4.2 Evaluation Flow

```
1. API Gateway receives request
2. Gateway fetches feature flags for tenant from Tenant Service (cached in Redis, 30s TTL)
3. Gateway evaluates flags and sets X-Feature-Flags header
4. Service reads X-Feature-Flags header for request-time behavior
5. Services MAY fetch flags directly from Tenant Service for internal logic
   (e.g., async event handlers) — cached locally with 30s TTL
```

> **Note:** Feature flags are cached at two layers: (1) the API Gateway caches via Redis, and (2) individual services cache in-memory for internal use. Both layers share the same 30s TTL and are invalidated on flag changes via the `tenant.feature.changed` event (published by Tenant Service). System configuration changes use `config.changed` separately — see [Data Architecture — Caching Strategy](data-architecture.md) for Redis cache key details.

### 4.3 Flag Schema

```json
{
  "flag_key": "new_commerce_workflow",
  "flag_type": "boolean",
  "default_value": false,
  "tenant_override": true,
  "description": "Enable new order processing workflow",
  "created_at": "2026-01-15T10:00:00Z",
  "updated_at": "2026-03-01T14:30:00Z",
  "rollout_percentage": null
}
```

### 4.4 Flag Rules

- Flags MUST have a default value (globally defined).
- Tenant overrides take precedence over defaults.
- Flag evaluation MUST be deterministic — same tenant always gets the same result within the cache TTL window (30 seconds).
- Flag changes are audited (append-only log in `tenant_db`).
- Percentage rollouts use consistent hashing on `tenant_id` to ensure stable assignment.
- Flags MAY have a scheduled activation/deactivation time (evaluated at gateway).

---

*See [Architecture](architecture.md) for the high-level system topology and consolidation rationale.*
*See [API Design](api-design.md) for feature flag and config API endpoints.*
