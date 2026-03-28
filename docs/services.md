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
| Responsibilities | Sales operations, customer management, stock management, warehousing, pricing, PIM, transportation, adaptive intelligence |
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

**Available-to-Promise (ATP) / Capable-to-Promise (CTP):**

| Module | Description |
|--------|-------------|
| ATP Check | Real-time availability check across warehouses considering open orders, safety stock, and lead times |
| CTP Check | Availability check considering production capacity and procurement lead times for make-to-order items |
| Allocation Rules | Configurable allocation rules by customer priority, channel, and region |
| Promise Date Calculation | Earliest ship date calculation considering inventory, production, and logistics constraints |
| Cross-Region ATP | Aggregate availability across regions with inter-company transfer time |
| ATP Dashboard | Real-time availability views by product, warehouse, and region |

**Product Configurator:**

| Module | Description |
|--------|-------------|
| Configuration Models | Define configurable product models with features, options, and constraints |
| Constraint Rules | Compatibility and incompatibility rules between options (e.g., engine A requires transmission B) |
| Pricing Integration | Dynamic price calculation based on selected configuration |
| BOM Generation | Automatic generation of production BOM from selected configuration |
| Validation Engine | Real-time completeness and feasibility validation |
| Guided Selling | Interactive configuration wizard with recommendations and upsell suggestions |
| Configuration Templates | Pre-defined configurations for common product variants |

**Credit Management:**

| Module | Description |
|--------|-------------|
| Credit Scoring | Automated credit scoring based on payment history, financial data, and external credit bureaus |
| Credit Limits | Customer credit limit management with currency support and multi-entity rules |
| Credit Hold | Automatic order hold when credit limit exceeded, configurable threshold alerts |
| Credit Release | Manual and automated credit release workflows with approval chains |
| Credit Exposure | Real-time credit exposure calculation (open orders + invoices + unbilled) |
| Collection Scoring | Predictive collection scoring for prioritizing collection activities |
| Credit Reports | Customer credit history, aging analysis, payment pattern analysis |

**Subscription Management:**

| Module | Description |
|--------|-------------|
| Subscription Lifecycle | Create, activate, amend, renew, suspend, cancel subscriptions |
| Billing Models | Recurring flat-rate, usage-based, tiered/graduated, per-unit, hybrid pricing |
| Billing Schedules | Automated billing cycle management with proration for mid-cycle changes |
| Usage Metering | Usage tracking and metering for consumption-based billing |
| Amendments | Mid-cycle changes with proration: upgrade, downgrade, add/remove items |
| Renewal Management | Automated renewal with configurable terms, renewal reminders, auto-renewal rules |
| Subscription Analytics | MRR/ARR tracking, churn analysis, cohort analysis, retention rates |
| Revenue Recognition Integration | Automatic performance obligation creation for Finance Service revenue recognition |

**Intercompany Drop Ship:**

| Module | Description |
|--------|-------------|
| Drop Ship Orders | Customer orders fulfilled directly by supplier or intercompany partner |
| Supplier Collaboration | Automated PO generation and dispatch to drop ship supplier |
| Shipment Tracking | Customer-visible tracking from supplier's shipment |
| Invoice Reconciliation | Automated reconciliation of supplier invoice, customer invoice, and PO |
| Multi-Entity Support | Intercompany drop ship across business units with automated intercompany accounting |

**Connected Logistics & Track-and-Trace:**

| Module | Description |
|--------|-------------|
| Real-Time Tracking | GPS-based shipment tracking with carrier integration and geofencing |
| Condition Monitoring | Temperature, humidity, and shock monitoring for sensitive shipments (IoT sensor integration) |
| Predictive ETA | ML-based estimated delivery time considering traffic, weather, and historical performance |
| Geofencing | Automatic alerts on geofence entry/exit for proactive logistics management |
| Exception Management | Automated exception detection and resolution workflows for delayed, damaged, or deviated shipments |
| Last-Mile Optimization | Route optimization for final delivery with delivery time window management |

**B2B Commerce Portal:**

| Module | Description |
|--------|-------------|
| Customer Self-Service | Customer portal for placing and tracking orders with real-time availability checks |
| Reorder Templates | Quick-reorder from order history and configurable saved shopping lists |
| Customer Approval Workflows | Customer-side approval chains for configurable order value thresholds |
| Account Management | Customer-managed users, shipping addresses, payment methods, and preferences |
| Order Tracking | Real-time order and shipment tracking with configurable notification preferences |
| Invoice Portal | Self-service invoice viewing, download, dispute initiation, and payment status |

### 2.2 Finance Service (Finance + Procurement)

| Aspect | Details |
|--------|---------|
| Port | 8011 |
| Database | `finance_db` |
| Responsibilities | Financial accounting, purchasing, supplier management, treasury, EPM, CLM, expenses, adaptive intelligence |
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

**Account Reconciliation:**

| Module | Description |
|--------|-------------|
| Auto-Matching | Rule-based matching of bank statements to open transactions by amount, date, and reference |
| Reconciliation Templates | Configurable reconciliation templates by account type with matching rules |
| Exception Management | Workflow-driven resolution of unmatched and partially matched transactions |
| Close Task Management | Financial close checklist with task assignment, dependencies, and deadline tracking |
| Intercompany Reconciliation | Automated matching of intercompany transactions across business units with elimination entries |

**Profitability Analysis:**

| Module | Description |
|--------|-------------|
| Cost-to-Serve | Full cost allocation to customer level including indirect costs, overhead, and logistics |
| Product Profitability | Gross and net margin analysis by product, category, and variant |
| Customer Profitability | Revenue vs. total cost analysis by customer and customer segment |
| Dimensional Analysis | Profitability by any dimension combination (product, customer, region, channel, business unit) |
| What-If Modeling | Scenario analysis for pricing changes, cost reductions, and product mix shifts |
| Contribution Margin | Variable cost analysis and contribution margin reporting by product line |

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

**Revenue Recognition (ASC 606 / IFRS 15):**

| Module | Description |
|--------|-------------|
| Contract Identification | Automated identification of customer contracts across sales orders, subscriptions, and project billing |
| Performance Obligations | Identification and tracking of distinct performance obligations per contract |
| Transaction Pricing | Allocation of transaction price to performance obligations based on standalone selling prices |
| Recognition Schedules | Automated recognition schedules: point-in-time, over-time (straight-line, proportional) |
| Variable Consideration | Estimation, constraint, and reassessment of variable consideration (discounts, rebates, contingencies) |
| Contract Modifications | Handling of contract changes with prospective and retrospective adjustment methods |
| Disclosure Reports | Required disclosures: remaining performance obligations, contract balances, recognized revenue |
| Revenue Waterfall | Visual waterfall from bookings to recognized revenue with deferral rollforward |
| Multi-Element Arrangements | Allocation across bundled goods, services, and licenses |

**Strategic Sourcing:**

| Module | Description |
|--------|-------------|
| Sourcing Events | Create and manage sourcing events (RFI, RFP, RFQ) with multi-round bidding and supplier collaboration |
| Bid Analysis | Side-by-side bid comparison with configurable scoring criteria and total cost of ownership calculation |
| Award Optimization | Award recommendation engine considering cost, quality, risk, diversity, and strategic criteria |
| Negotiation Tracking | Negotiation round management with version tracking, redline comparison, and term history |
| Sourcing Analytics | Savings tracking, supplier performance post-award, spend under management metrics |

**Supplier Risk Management:**

| Module | Description |
|--------|-------------|
| Risk Scoring | Composite supplier risk score from financial, operational, geographic, and regulatory indicators |
| Financial Health Monitoring | Automated monitoring of supplier financial data (credit ratings, financial statements, payment behavior) |
| Geographic Risk | Country and regional risk assessment including political stability, sanctions, and natural disaster exposure |
| Regulatory Compliance | Supplier compliance monitoring for industry certifications, regulatory requirements, and certifications |
| Risk Dashboards | Supplier risk heat maps, risk trend analysis, concentration risk visualization, alert management |
| Risk Mitigation | Mitigation plan tracking with action items, owners, and deadlines for identified risks |

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
| Responsibilities | Production operations, quality management, cost accounting, PLM, EAM, manufacturing intelligence, digital thread |

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

**Manufacturing Intelligence:**

| Module | Description |
|--------|-------------|
| OEE Tracking | Overall Equipment Effectiveness measurement by work center, line, and plant |
| Downtime Analysis | Downtime categorization, root cause tracking, Pareto analysis |
| Production Analytics | Real-time production rate tracking, yield analysis, scrap monitoring |
| Capacity Utilization | Work center utilization dashboards, bottleneck identification |
| Energy Analytics | Energy consumption per unit produced, cost allocation, efficiency trends |
| Predictive Maintenance Analytics | Maintenance cost analysis, MTBF/MTTR tracking, maintenance effectiveness |

**Digital Thread:**

| Module | Description |
|--------|-------------|
| Traceability Graph | Unified traceability linking design, manufacturing, quality, and service records |
| Serial/Lot Genealogy | Full genealogy from raw material to finished product to customer delivery |
| Change Impact Analysis | Impact analysis for ECOs showing affected orders, inventory, and production plans |
| Product Instance Tracking | Lifecycle tracking of individual product instances from manufacture to disposal |

**IoT Integration:**

| Module | Description |
|--------|-------------|
| Device Registry | IoT device registration with certificate-based authentication and device metadata |
| Telemetry Ingestion | High-throughput telemetry data ingestion from sensors and industrial equipment |
| Alert Rules | Configurable threshold and anomaly-based alerting on real-time telemetry data |
| Edge Processing | Optional edge computing for latency-sensitive processing before cloud ingestion |
| Data Routing | Configurable telemetry routing to manufacturing, logistics, or maintenance modules |
| Device Health Monitoring | Device connectivity status, battery levels, and firmware update management |

**Digital Twin:**

| Module | Description |
|--------|-------------|
| Asset Digital Twins | Virtual representation of physical assets with real-time state synchronization |
| Simulation Engine | What-if simulation for production scenarios, capacity planning, and maintenance impact |
| Predictive Maintenance | ML-based prediction of equipment failure from telemetry and historical maintenance data |
| Real-Time Monitoring | Live dashboard of asset health, performance metrics, and anomaly detection |
| Twin Analytics | Asset performance comparison, degradation trends, and optimization recommendations |

### 2.5 Report Service

| Aspect | Details |
|--------|---------|
| Port | 8014 |
| Database | `report_db` (materialized views, pre-aggregated data); reads from other services' event streams |
| Responsibilities | Analytics, reporting, dashboards, BI, AI-driven insights, ESG, carbon accounting, augmented analytics |

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

**Narrative Reporting:**

| Module | Description |
|--------|-------------|
| Management Reports | Structured management reports with financial and operational data |
| Commentary & Annotations | Collaborative annotations, commentary workflows, and approval chains on report sections |
| Report Packages | Collection of reports organized for board meetings, investor communications |
| Versioning | Report version control with audit trail of commentary changes |
| Distribution | Automated distribution of report packages to stakeholders via email and portal |
| XBRL Tagging | Automated XBRL tagging for regulatory filing requirements |

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

**Process Mining & Optimization:**

| Module | Description |
|--------|-------------|
| Process Discovery | Automated reconstruction of business processes from event logs and audit trails |
| Conformance Checking | Compare actual process execution against defined BPMN workflows |
| Bottleneck Analysis | Identify process steps with highest wait times, lowest throughput, and resource constraints |
| Simulation | What-if simulation of process changes with predicted throughput and cost impact |
| Root Cause Analysis | Drill-down from process KPI deviations to specific events, resources, and transactions |
| Continuous Monitoring | Real-time process performance dashboards with configurable deviation alerts |

**Corporate Performance Management:**

| Module | Description |
|--------|-------------|
| Executive Dashboards | Role-based executive dashboards with KPI summaries and drill-down to operational data |
| Strategy Maps | Visual strategy maps linking corporate objectives, KPIs, and strategic initiatives |
| OKRs | Objective and Key Results tracking with progress visualization and alignment reporting |
| Balanced Scorecard | Four-perspective scorecard: financial, customer, internal process, learning & growth |
| Initiative Tracking | Strategic initiative registration, milestone tracking, budget allocation, and status reporting |

**Augmented Analytics:**

| Module | Description |
|--------|-------------|
| Natural Language Queries | NLQ-to-SQL engine for ad-hoc data exploration without SQL knowledge |
| Auto-Insights | Automated pattern discovery, trend detection, and anomaly flagging |
| Smart Discovery | Automated correlation analysis across dimensions and measures |
| Explainable AI | Human-readable explanations for analytical findings and ML predictions |
| Contextual Narratives | Auto-generated narrative summaries for dashboards and reports |

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
| Responsibilities | Customer relationship management, marketing automation, customer analytics, adaptive intelligence |

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

**Field Service Management:**

| Module | Description |
|--------|-------------|
| Service Orders | Service order creation from CRM cases, warranty claims, and maintenance contracts |
| Technician Scheduling | Skills-based assignment, geographic optimization, real-time calendar management |
| Work Orders | Detailed service work orders with tasks, parts, and labor requirements |
| Parts Logistics | Spare parts reservation and dispatch, van stock management, returns |
| SLA Management | Response time and resolution time SLA tracking with escalation |
| Field Technician Portal | Mobile-optimized portal with schedule, work order details, parts inventory, and completion forms |
| Service Contracts | Service level agreements, warranty tracking, preventive maintenance agreements |
| Service Analytics | Technician utilization, first-time fix rate, customer satisfaction, cost-per-service-call |

**Survey & Feedback Management:**

| Module | Description |
|--------|-------------|
| Survey Builder | Drag-and-drop survey designer with multiple question types (rating, multiple choice, open text, NPS) |
| Distribution Channels | Email, in-app, SMS, website embed, post-interaction trigger |
| NPS / CSAT Collection | Automated Net Promoter Score and Customer Satisfaction surveys at configurable touchpoints |
| Response Analytics | Real-time response dashboards, sentiment analysis, trend analysis, cross-tabulation |
| Trigger Surveys | Event-driven survey dispatch (post-purchase, post-support, post-delivery) |
| Integration | Auto-create CRM cases from negative responses, feed sentiment data to customer analytics |

**Sales Territory & Quota Planning:**

| Module | Description |
|--------|-------------|
| Territory Design | Geographic, industry, and account-based territory modeling and assignment |
| Territory Optimization | Balanced territory allocation based on revenue potential, workload, and coverage metrics |
| Quota Allocation | Top-down and bottom-up quota allocation by territory, product, and sales rep |
| What-If Modeling | Territory and quota scenario planning with revenue impact analysis |
| Historical Analysis | Territory performance trending, quota attainment analysis, overlay comparison |
| Integration | Territory assignments synced to CRM pipeline and opportunity routing |

**Customer Data Platform (CDP):**

| Module | Description |
|--------|-------------|
| Unified Profiles | Single customer view aggregated from Commerce, CRM, Support, and Marketing touchpoints |
| Identity Resolution | Deterministic and probabilistic matching across channels, devices, and identifiers |
| Segmentation | Dynamic segment builder with behavioral, demographic, and transactional criteria |
| Journey Orchestration | Multi-step customer journey designer with trigger-based actions and cross-channel delivery |
| Engagement Scoring | Real-time engagement score computation based on recency, frequency, and monetary value |
| Cross-Channel Analytics | Attribution modeling, channel preference analysis, and journey completion rate reporting |
| Profile Enrichment | Third-party data enrichment integration for enhanced customer attributes |

**Adaptive Intelligence (CRM):**

| Module | Description |
|--------|-------------|
| Lead Scoring | ML-driven lead scoring based on engagement patterns and conversion history |
| Next-Best-Action | Recommended actions for sales reps based on deal context and history |
| Churn Prediction | Real-time churn risk scoring with retention action recommendations |
| Cross-Sell / Up-Sell | Product recommendations based on purchase history and customer profile |
| Deal Risk Assessment | Automated deal risk scoring based on engagement, timeline, and competitive signals |

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
| Responsibilities | Multi-channel notifications, file storage, document management, audit logging, digital assistant, low-code application builder, GRC, content management, privacy management, DLP |
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
| Enterprise Job Scheduler | Cron-based and event-triggered job scheduling, DAG-based job dependencies, retry policies, per-tenant concurrency limits, job execution history and monitoring |
| Knowledge Management | Article authoring with rich text and version control, hierarchical categories, tagging, full-text search, role-based access, context-sensitive knowledge suggestions, article analytics (views, ratings) |
| Digital Signatures | Native digital signature workflows, signature request and tracking, multi-party signing sequences, signature audit trail, integration with e-signature providers (DocuSign, Adobe Sign) for external documents |
| Employee Self-Service Portal | Personal information management, payslip access, benefits enrollment, time-off requests, expense submission, training enrollment, company directory, organizational announcements |

**Robotic Process Automation (RPA):**

| Module | Description |
|--------|-------------|
| Bot Designer | Visual bot designer with drag-and-drop action sequences and conditional logic |
| Bot Execution | Schedule-based, event-triggered, and manual bot execution with dependency management |
| Action Library | Pre-built actions for data entry, report generation, notifications, API calls, and file operations |
| Credential Vaulting | Secure credential management for bot accounts via HashiCorp Vault integration |
| Bot Monitoring | Execution history, success rates, duration tracking, and error alerting |
| Error Handling | Configurable retry policies, fallback actions, and manual intervention queues |

**Enterprise Collaboration:**

| Module | Description |
|--------|-------------|
| Team Messaging | Channel-based messaging with business context integration (order, project, case context) |
| Document Collaboration | Real-time document co-editing with version control, commenting, and review workflows |
| Task Management | Personal and team task boards with integration to projects and workflow tasks |
| Presence & Availability | User availability status with calendar integration and notification preferences |
| Unified Search | Search across messages, documents, tasks, and knowledge base articles |

**Content Management:**

| Module | Description |
|--------|-------------|
| Content Repository | Hierarchical content storage with configurable metadata schemas |
| Document Lifecycle | Draft → Review → Approved → Published → Archived lifecycle with configurable workflows |
| Records Management | Retention policies, legal hold, disposition schedules, records classification |
| Content Search | Full-text search with faceted navigation across all content types |
| Compliance Archive | Immutable archive for regulatory-compliant document retention |

**Privacy Management:**

| Module | Description |
|--------|-------------|
| Consent Management | Granular consent capture, tracking, and enforcement per processing purpose |
| Data Subject Rights | Automated handling of access, rectification, erasure, portability, and objection requests |
| DPIA Workflows | Automated Data Protection Impact Assessment workflows with risk scoring |
| Processing Register | Automated register of data processing activities (GDPR Article 30) |
| Data Flow Mapping | Visual mapping of personal data flows across services and third parties |

**Data Loss Prevention (DLP):**

| Module | Description |
|--------|-------------|
| Data Classification | Automated sensitive data classification (PII, PHI, financial, IP) |
| Policy Engine | Configurable DLP policies with detection, blocking, and alerting rules |
| Access Monitoring | Monitoring of data access patterns for anomalous exfiltration behavior |
| DLP Incidents | Automated DLP incident creation, investigation workflows, remediation tracking |
| Compliance Reporting | DLP compliance dashboards, incident metrics, policy effectiveness analysis |

### 3.2 Integration Service

| Aspect | Details |
|--------|---------|
| Port | 8021 |
| Database | `integration_db` |
| Responsibilities | External integrations, API management, API marketplace, connector framework, data import/export, MDM, data governance, event mesh |

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
| Trade Compliance | Restricted party screening (OFAC, EU, UN denied party lists), export control classification, license management, customs documentation generation, compliance decision audit trail, dual-use goods screening |
| Compliance Screening Engine | Real-time screening on order creation, supplier onboarding, and shipment dispatch with configurable rules and alerting |
| API Marketplace | Centralized API catalog, developer portal, API key provisioning, usage analytics, API monetization |
| Event Mesh | Event gateway (HTTP-to-event bridge), topic hierarchy management, schema registry, event filtering |
| Developer Portal | Self-service portal for API consumers with sandbox access, documentation, and migration guides |

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
