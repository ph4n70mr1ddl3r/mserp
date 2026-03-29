# MSERP — Master Specification

> **This document is the authoritative single source of truth for MSERP.** All other documents are subordinate. If any conflict arises, this document takes precedence.

---

## 1. Executive Summary

MSERP is an enterprise-grade, microservices-based ERP system built in Rust, designed for scalability, reliability, and performance. The system provides comprehensive ERP coverage comparable to Oracle Fusion Cloud, spanning Financial Management, Supply Chain Management, Human Capital Management, Customer Experience, Project Management, Manufacturing, Enterprise Performance Management, and Governance, Risk & Compliance — all built on a modern, cloud-native microservices architecture.

### 1.1 Key Targets

| Metric | Target |
|--------|--------|
| API response (p50) | < 50ms |
| API response (p99) | < 500ms |
| Peak throughput | 10,000+ req/s |
| Concurrent users | 1,000+ |
| Availability SLA | 99.9% |
| RTO (automated failover) | < 15 minutes |
| RPO (synchronous replication) | < 5 minutes |

### 1.2 Core Technologies

| Category | Technology | Version |
|----------|-----------|---------|
| Language | Rust | 1.83+ |
| Web Framework | Axum | 0.7+ |
| Database | PostgreSQL | 16 |
| Message Broker | RabbitMQ | 3.12 |
| Cache | Redis | 7.x |
| Search | Elasticsearch | 8.x |
| Object Storage | MinIO (S3-compatible) | RELEASE.2024-01+ |
| Orchestration | Kubernetes | 1.28+ |
| API Gateway | Traefik | 3.x |
| CI/CD | GitHub Actions + ArgoCD | Latest |
| Observability | Prometheus + Grafana + Jaeger + Loki | Latest |
| Data Warehouse | Apache Arrow + DuckDB | Latest |
| ML/AI Runtime | ONNX Runtime + Candle | Latest |
| Vector Search | qdrant (optional) | Latest |
| Mobile | React Native | Latest |

---

## 2. Architecture Overview

See [docs/01-architecture/overview.md](docs/01-architecture/overview.md) for the full architecture specification.

### 2.1 Topology

```
Client Layer → Gateway Layer (Traefik) → Microservices Layer (14 services) → Infrastructure Layer
```

- **Client Layer**: Web (React), Mobile (React Native), B2B Portal, API Consumers, Admin Console, Digital Assistant
- **Gateway Layer**: Traefik (mTLS termination, rate limiting, circuit breaker, routing)
- **Microservices Layer**: 14 services in 3 tiers (see §3)
- **Infrastructure Layer**: PostgreSQL, Redis, RabbitMQ, Elasticsearch, MinIO, DuckDB, observability stack

### 2.2 Communication Patterns

| Pattern | Protocol | Use Case |
|---------|----------|----------|
| Synchronous | REST (HTTP/JSON) | Request/response queries |
| Asynchronous | RabbitMQ (AMQP) | Event propagation, saga choreography |
| Streaming | SSE / WebSocket | Real-time updates, notifications |
| Bulk | REST (multipart/batch) | Data import/export, batch operations |

### 2.3 Multi-Tenancy

All services enforce tenant isolation via PostgreSQL Row-Level Security (RLS). Every request includes `X-Tenant-Id` header propagated from the JWT. Cache keys are prefixed with `{tenant_id}:`. Elasticsearch uses per-tenant indices. MinIO paths are prefixed with `{tenant_id}/`.

### 2.4 Deployment

Multi-region active-passive deployment with automated DNS failover. See [docs/08-infrastructure/deployment.md](docs/08-infrastructure/deployment.md) and [SPEC.md §9](#9-deployment-topology).

---

## 3. Service Catalog

MSERP consists of 14 microservices organized into 3 tiers. Each service owns its database exclusively.

### 3.1 Service Registry

| # | Service | Port | Database | Tier | Responsibility |
|---|---------|------|----------|------|---------------|
| 1 | Auth Service | 8001 | `auth_db` | Core | Authentication, JWT, MFA, SSO, session management |
| 2 | Identity Service | 8002 | `identity_db` | Core | Users, roles, permissions, RBAC/ABAC, organizations |
| 3 | Tenant Service | 8003 | `tenant_db` | Core | Multi-tenancy, feature flags, subscriptions, quotas |
| 4 | Config Service | 8004 | `config_db` | Core | Dynamic configuration, localization, ESG settings |
| 5 | Commerce Service | 8010 | `commerce_db` | Business | Order lifecycle, inventory, pricing, PIM, logistics, subscriptions, loyalty |
| 6 | Finance Service | 8011 | `finance_db` | Business | GL, AP/AR, treasury, revenue recognition, budgeting, procurement, tax |
| 7 | HCM Service | 8012 | `hr_db` | Business | Employee lifecycle, payroll, recruitment, talent, benefits, self-service |
| 8 | Manufacturing Service | 8013 | `manufacturing_db` | Business | BOM, work orders, PLM, EAM, quality, IoT, digital twin, EHS |
| 9 | Report Service | 8014 | `report_db` | Business | Analytics, dashboards, BI, ESG reporting, data warehouse, ML platform |
| 10 | Workflow Service | 8015 | `workflow_db` | Business | BPMN engine, approvals, SLA management, business rules |
| 11 | CRM Service | 8016 | `crm_db` | Business | Pre-sale (leads, deals, campaigns), field service, CDP, contact center, surveys |
| 12 | Project Service | 8017 | `project_db` | Business | Projects, resources, billing, EVM, portfolio analysis |
| 13 | Platform Service | 8020 | `platform_db` + `audit_db` | Supporting | Notifications, files, GRC, digital assistant, IDP, RPA/IPA, ITSM, job scheduler |
| 14 | Integration Service | 8021 | `integration_db` | Supporting | Connectors, EDI, MDM, data quality, trade compliance, blockchain, API marketplace |

**Total databases: 15** (4 core + 8 business + 3 supporting including `audit_db`).

### 3.2 Service Specs

Each service has a detailed specification in `docs/06-services/`:

- [Auth Service](docs/06-services/auth.md)
- [Identity Service](docs/06-services/identity.md)
- [Tenant Service](docs/06-services/tenant.md)
- [Config Service](docs/06-services/config.md)
- [Commerce Service](docs/06-services/commerce.md)
- [Finance Service](docs/06-services/finance.md)
- [HCM Service](docs/06-services/hr.md)
- [Manufacturing Service](docs/06-services/manufacturing.md)
- [Report Service](docs/06-services/report.md)
- [Workflow Service](docs/06-services/workflow.md)
- [CRM Service](docs/06-services/crm.md)
- [Project Service](docs/06-services/project.md)
- [Platform Service](docs/06-services/platform.md)
- [Integration Service](docs/06-services/integration.md)

---

## 4. Service Domain Ownership

Every feature, entity, API endpoint, database table, and event belongs to EXACTLY ONE service. This table is the authoritative ownership registry.

### 4.1 CRM Service (Port 8016)

**Owns:** Pre-sale customer relationships, marketing, and post-sale service.

| Domain | Modules |
|--------|---------|
| Lead Management | Lead capture, scoring, qualification, nurture campaigns |
| Deal Pipeline | Opportunities with stages, probability tracking, deal desk |
| Account Management | Customer accounts, hierarchy, relationship mapping |
| Contact Management | Contacts, activity tracking, communication history |
| Campaign Management | Campaign planning, execution, ROI tracking |
| Marketing Automation | Email sequences, drip campaigns, lead nurturing, A/B testing (marketing) |
| Customer Service | Case management, escalation, SLA tracking |
| Social Listening | Brand monitoring, sentiment analysis, competitive intelligence |
| Customer Analytics | Customer lifetime value, acquisition cost, retention rate, churn prediction |
| Field Service | Service orders, technician scheduling, parts logistics, SLA tracking |
| Survey & Feedback | Survey creation, NPS/CSAT collection, response analytics |
| CDP | Unified customer profiles, identity resolution, segmentation, journey orchestration |
| Sales Territory & Quota | Territory management, quota allocation, what-if modeling |
| Contact Center | Omnichannel routing, IVR/voice bot, workforce management, quality management |
| Social Selling | Social enrichment, engagement tracking, relationship mapping, social intelligence |
| Adaptive Intelligence (CRM) | Lead scoring, next-best-action, deal risk prediction, sentiment analysis |

### 4.2 Commerce Service (Port 8010)

**Owns:** The complete order lifecycle from quotation through delivery and returns, including inventory, pricing, and logistics.

| Domain | Modules |
|--------|---------|
| Quotation Management | Quote creation, versioning, approval, conversion to order |
| Order Management | Sales orders, state machine (draft→confirmed→fulfilled→closed), backorders |
| Delivery & Fulfillment | Delivery scheduling, shipment tracking, proof of delivery |
| Returns & Credit Memos | RMA processing, refund/credit workflows |
| Commissions | Sales commission calculation, split commissions, reporting |
| Product Information (PIM) | Product catalog, attributes, categories, cross-references |
| Inventory Management | Stock levels, warehouses, locations, serial/batch tracking |
| Inventory Optimization | Safety stock, demand-driven replenishment, ABC analysis |
| Warehouse Management | Put-away, picking, packing, cycle counting |
| Transfers | Inter-warehouse transfers, in-transit tracking |
| Advanced Pricing Engine | Price lists, discount rules, promotions, price formulas, customer-specific pricing |
| ATP/CTP | Available-to-Promise, Capable-to-Promise, allocation rules |
| Product Configurator | Constraint-based configuration, pricing integration, BOM generation |
| Credit Management | Credit scoring, limits, holds, exposure calculation, bureau integration |
| Subscription Management | Subscription lifecycle, recurring billing, amendments, proration |
| Customer Loyalty | Tiered programs, points engine, reward catalogs, coalition programs |
| B2B Commerce Portal | Self-service ordering, reorder templates, approval workflows |
| Omnichannel | Cross-channel cart, order routing, click-and-collect, ship-from-store |
| AI Price Optimization | Demand elasticity, competitive pricing, dynamic pricing, markdown optimization |
| Intercompany Drop Ship | Cross-entity fulfillment with automated intercompany settlement |
| Connected Logistics | Real-time shipment tracking, condition monitoring, predictive ETA, geofencing |
| Warranty Management | Warranty policies, claims processing, entitlement validation, supplier recovery |
| Demand Planning Integration | Forecast ingestion from Report Service, demand signal processing |
| Supply Allocation | Allocation rules, prioritization, shortage management |

### 4.3 Finance Service (Port 8011)

**Owns:** All financial accounting, treasury, procurement, and financial compliance.

| Domain | Modules |
|--------|---------|
| General Ledger | Chart of accounts, journal entries, posting, period management |
| Accounts Payable | Vendor invoices, three-way match, payment scheduling, vendor management |
| Accounts Receivable | Customer invoices, payment application, dunning, collections |
| Fixed Assets | Asset registration, depreciation, disposal, revaluation |
| Budgeting | Budget creation, approval, variance analysis, budget controls |
| Tax Management | Multi-jurisdiction tax (VAT/GST/sales/use/withholding), exemption management, external engine integration (Vertex/Avalara), nexus tracking |
| Multi-Currency | Exchange rates, triangulation, realized/unrealized gains/losses |
| Cash Management | Cash position, forecasting, bank connectivity, cash pooling |
| Intercompany | Intercompany transactions, elimination, netting |
| Financial Reporting | Standard financial statements, regulatory reports, XBRL tagging |
| Cost Centers & Profit Centers | Organizational accounting, allocation rules |
| Period Management | Open/close periods, soft/hard close, period controls |
| Corporate Treasury | Bank account management, cash positioning, in-house banking, payment netting |
| Account Reconciliation | Auto-matching, reconciliation templates, exception management, close tasks |
| Profitability Analysis | Cost-to-serve, product/customer profitability, margin analysis by dimension |
| Lease Accounting (ASC 842/IFRS 16) | Lease classification, ROU assets, lease liabilities, modifications, disclosures |
| Grant Management | Award tracking, compliance reporting, fund accounting, milestone billing |
| Joint Venture Accounting | JV agreements, ownership splits, automated distribution, partner reporting |
| Intelligent Close | AI-assisted close, anomaly detection, predictive close analytics |
| Advanced Collections | Collection strategies, automated dunning, dispute management, promise-to-pay |
| Dynamic Discounting | Sliding-scale early payment discounts, working capital optimization, discount analytics |
| Enterprise Expense Management | Expense reports, policies, approval workflows, corporate card integration |
| Enterprise Performance Management | Planning, budgeting, forecasting, consolidation, what-if modeling |
| Financial Consolidation | Multi-entity consolidation, currency translation, minority interest |
| Revenue Recognition (ASC 606/IFRS 15) | Performance obligations, revenue scheduling, contract modifications, variable consideration |
| Strategic Sourcing | RFI/RFP/RFQ, bid analysis, award optimization, negotiation tracking |
| Supplier Risk Management | Supplier risk scoring, financial health monitoring, geographic risk |
| Contract Lifecycle Management | Contract authoring, clause library, e-signature integration, renewal tracking |
| Procurement | Purchase requisitions, purchase orders, goods receipt, invoice matching |
| Commodity Management | Market price feeds, hedging, futures tracking, price simulation |
| Advanced Spend Analysis | ML-powered classification, maverick detection, savings identification |
| Supplier Diversity | Diversity classification, spend tracking, tier reporting, compliance reporting |
| Financial Reporting Studio | XBRL report builder, regulatory templates, collaborative annotations (templates owned by Finance, rendered by Report Service) |

### 4.4 HCM Service (Port 8012)

**Owns:** The complete employee lifecycle from recruitment through separation.

| Domain | Modules |
|--------|---------|
| Core HR | Employee records, org chart, position management, global HR |
| Time & Attendance | Timesheets, time-off management, absence tracking |
| Leave Management | Leave policies, accruals, requests, approvals |
| Payroll | Payroll processing, tax calculations, payslips, multi-country localizations |
| Recruitment | Job postings, applicant tracking, interview scheduling, offer management |
| Onboarding | New hire workflows, document collection, equipment provisioning |
| Performance Management | Goals, reviews, 360 feedback, calibration |
| Talent Review & Succession | Talent pools, succession planning, readiness assessment |
| Workforce Modeling | Workforce planning, demand forecasting, scenario modeling |
| Learning & Development | Training catalog, enrollments, certifications, compliance training |
| Compensation | Salary structures, merit planning, variable pay, benchmarking |
| Benefits Administration | Benefit plans, enrollment, eligibility, carrier integration |
| Employee Self-Service Portal | Personal info, payslips, benefits enrollment, time-off requests, expense submission |
| HR Analytics | Turnover, headcount, diversity, compensation analysis |
| Global HR Localization | Country-specific localizations, regulatory compliance, localized reporting |
| Document Management | Employee document storage, e-signatures |

### 4.5 Manufacturing Service (Port 8013)

**Owns:** All manufacturing operations from planning through production, quality, and asset management.

| Domain | Modules |
|--------|---------|
| BOM Management | Bill of materials, versioning, phantom assemblies, engineering BOM vs manufacturing BOM |
| Work Orders | Work order creation, scheduling, execution, completion |
| Production Planning | MPS, MRP, production scheduling, capacity planning |
| ASCP | Advanced supply chain planning, demand-driven MRP |
| Quality Control | Inspection plans, results recording, quality certificates |
| Advanced Quality Management | SPC, quality analytics, CAPA, supplier quality |
| Maintenance | Preventive/predictive maintenance, work orders |
| Enterprise Asset Management (EAM) | Asset registry, lifecycle management, maintenance strategies |
| Work Centers | Work center definition, capacity, cost rates |
| Routing | Operation sequences, time standards, alternative routings |
| Cost Accounting | Standard costing, actual costing, variance analysis |
| Shop Floor Control | RFID/barcode, real-time tracking, operator terminals |
| Product Lifecycle Management (PLM) | Product concepts, revisions, ECO workflow, collaboration |
| Manufacturing Intelligence | Production analytics, OEE tracking, downtime analysis, predictive maintenance analytics |
| Advanced Planning & Scheduling (APS) | Finite scheduling, constraint-based planning, what-if simulation |
| Lean Manufacturing | Kanban, value stream mapping, continuous improvement |
| Connected Worker | Mobile worker, digital work instructions, AR assistance |
| Digital Thread | End-to-end traceability (design→manufacturing→service→disposal) |
| IoT Integration | Device registry, telemetry ingestion (MQTT/OPC-UA), alert rules, edge processing |
| Digital Twin | Asset twins, simulation, predictive maintenance, what-if analysis |
| EHS (Enterprise Health & Safety) | Incident management, risk assessment, chemical management, occupational health, OSHA/ISO 45001 |
| MRO (Maintenance, Repair & Operations) | MRO catalog, spare parts, repair orders, rotable management |
| Product Compliance & Regulatory | Regulatory database, certifications, material compliance (RoHS/REACH), testing management |
| Sustainability | Waste tracking, energy consumption, emissions at manufacturing level |

### 4.6 Report Service (Port 8014)

**Owns:** The analytics and reporting platform. All services embed analytics via Report Service widgets. Report Service is the sole exception to the database-per-service rule: it reads from a DuckDB data warehouse populated by ETL pipelines from event streams, not from operational databases.

| Domain | Modules |
|--------|---------|
| Report Builder | Drag-and-drop report designer, scheduled reports, export (PDF/Excel/HTML) |
| Dashboards | Real-time dashboards, KPI widgets, drill-down analysis |
| Data Warehouse | DuckDB star schema, ETL from event streams (not operational DBs), hourly/daily refresh |
| Data Lake | MinIO zones (raw/curated/analytics), schema-on-read, Parquet storage |
| ESG & Sustainability | Carbon accounting, emissions tracking, ESG scorecards, regulatory frameworks |
| ML/AI Platform | Model training, serving (ONNX), feature store, model registry |
| Predictive Planning | Demand forecasting, what-if simulation, predictive analytics |
| Narrative Reporting | Management commentary, collaborative annotations |
| Process Mining | Process discovery, conformance checking, bottleneck analysis, simulation |
| CPM (Corporate Performance Management) | Executive dashboards, strategy maps, OKRs, balanced scorecard |
| Embedded Analytics | Widget framework for embedding analytics in other services' UIs |
| Augmented Analytics | NL-to-SQL, auto-generated insights, smart data discovery |
| Connected Planning | Unified financial/workforce/supply chain planning with shared assumptions |
| Self-Service BI | Ad-hoc queries, data exploration, visualization builder |

### 4.7 Workflow Service (Port 8015)

**Owns:** The BPM/workflow engine used by all services.

| Domain | Modules |
|--------|---------|
| BPMN 2.0 Engine | Process execution, BPMN conformance (processes, gateways, events, tasks) |
| Approval Chains | Multi-level approvals, delegation, escalation |
| Escalation Management | Time-based escalation, notification escalation, reassignment |
| Process Templates | Pre-built workflow templates (6 standard) |
| SLA Management | SLA definitions per workflow type, deadline tracking |
| Business Rules Engine | Rule definition, evaluation, versioning |
| Process Monitoring | Real-time process state, bottleneck detection, audit trail |
| Email Approvals | Approve/reject via email response |
| SoD Enforcement | SoD rule definitions in approval workflows, conflict detection |

### 4.8 Project Service (Port 8017)

**Owns:** Project and portfolio management.

| Domain | Modules |
|--------|---------|
| Project Planning | WBS, Gantt chart, milestones, dependencies |
| Resource Allocation | Resource requests, assignments, utilization tracking |
| Time & Expense (Project) | Project timesheets, expense reports, approval workflows |
| Project Billing | Time & material billing, fixed-price billing, milestone billing |
| Budget & Costing | Project budgets, cost tracking, variance analysis |
| Risk Management | Risk identification, assessment, mitigation tracking |
| Project Templates | Standard project templates, copy-from-project |
| Program Management | Program definition, project grouping, cross-project dependencies |
| Project Analytics | Project health, budget vs actual, earned value |
| Earned Value Management (EVM) | PV, EV, AC, CPI, SPI, EAC, ETC |
| Project Portfolio Analysis | Portfolio balancing, investment analysis, benefit realization, interdependency mapping |

### 4.9 Platform Service (Port 8020)

**Owns:** Cross-cutting platform capabilities used by all services.

| Domain | Modules |
|--------|---------|
| Notifications | Email, SMS, push, in-app notifications, webhook dispatch |
| File Storage | Upload/download, preview, versioning, MinIO integration |
| Document Metadata | Metadata extraction, tagging, search indexing |
| OCR & Preview | Document OCR, thumbnail generation, format conversion |
| Audit Trail | Event store (in `audit_db`), data lineage, compliance logging |
| GRC | Segregation of Duties (SoD), compliance policy management, risk scoring |
| Data Masking | Field-level masking rules, subsetting, anonymization |
| Job Scheduler | Cron/one-time/recurring/event-triggered jobs, DAG dependencies, retry policies |
| Knowledge Management | Knowledge base, article authoring, categorization, versioning |
| Digital Signatures | Native signing, signature workflows, DocuSign/Adobe Sign integration |
| Digital Assistant | NLP chat interface, intent recognition, action execution, context management |
| Application Builder | Low-code page designer, custom business objects, form builder, workflow integration |
| IDP (Intelligent Document Processing) | AI classification, data extraction, template learning, batch processing |
| RPA & IPA | Bot designer, cognitive automation, self-learning bots, process discovery, attended/unattended |
| Enterprise Collaboration | Channel messaging, document co-editing (CRDT), task management, presence |
| ECM (Enterprise Content Management) | Document lifecycle, records management, retention policies, WORM archive |
| Privacy Management | Consent management, DSAR automation, DPIA workflows, breach notification |
| DLP (Data Loss Prevention) | Data classification, policy engine, access monitoring, incident response |
| ITSM | Incident, problem, change management, CMDB, service catalog, SLA management |
| Compliance Hub | Unified compliance dashboard, regulatory change tracker, cross-framework control mapping |
| Adaptive Intelligence | ML-powered recommendations embedded in business workflows |

### 4.10 Integration Service (Port 8021)

**Owns:** External connectivity, master data management, and data quality.

| Domain | Modules |
|--------|---------|
| Connectors | 50+ pre-built connectors (see §10) |
| Connector SDK | Rust-based SDK for custom connectors |
| Webhooks | Inbound webhook registration, payload mapping, verification |
| EDI (Electronic Data Interchange) | ANSI X12 / EDIFACT processing, AS2/SFTP transport |
| Data Import/Export | CSV/JSON/Excel/XML/PDF/Parquet, field mapping, validation |
| Integration Monitoring | Connection health, throughput metrics, error tracking |
| OAuth2 Client | External OAuth2 flows for connector auth |
| MDM (Master Data Management) | Golden records, deduplication, data stewardship, domain management |
| Data Governance | Data ownership, lineage tracking, policy enforcement |
| Trade Compliance | Restricted party screening (OFAC/EU/UN), export controls, license management, customs documentation |
| Compliance Screening | Denied party screening, sanctions list management |
| API Marketplace (Infrastructure) | API gateway management, rate limiting infrastructure, developer portal backend |
| Event Mesh (Infrastructure) | RabbitMQ topology management, event gateway, cross-region replication |
| Developer Portal | Documentation, sandbox, API key management, usage analytics |
| Blockchain Integration | Distributed ledger adapters (Hyperledger/Ethereum), smart contract management, supply chain provenance |
| Enterprise Data Quality | Data profiling, cleansing, probabilistic matching (Fellegi-Sunter), enrichment, quality scorecards, stewardship workflows |

### 4.11 Core Services

| Service | Port | Key Domains |
|---------|------|------------|
| Auth Service | 8001 | OAuth2/OIDC, MFA (TOTP/SMS/biometric), SSO (SAML 2.0), JWT, session management, brute-force detection |
| Identity Service | 8002 | User lifecycle, RBAC + ABAC, user groups, org hierarchy, API keys, delegated administration |
| Tenant Service | 8003 | Tenant lifecycle, subscription tiers, feature toggles, quotas, data residency |
| Config Service | 8004 | Hierarchical config, localization, ESG settings, config propagation via events |

---

## 5. Cross-Service Boundaries

### 5.1 CRM → Commerce Handoff

CRM owns the pre-sale funnel (lead → opportunity). Commerce owns the order lifecycle (quote → order → delivery → invoice → return). The boundary is at opportunity-to-quote conversion:

```
CRM: Lead → Qualified Lead → Opportunity
                                    ↓ (opportunity converted to quote)
Commerce: Quote → Order → Delivery → Invoice → Payment (via Finance)
```

CDP (owned by CRM) maintains unified customer profiles. Commerce publishes transactional events (orders, deliveries, returns) that CRM's CDP consumes to enrich profiles.

### 5.2 Commerce → Finance Handoff

Commerce manages the order lifecycle. Finance manages financial recording. The boundary:

| Event | Publisher (Commerce) | Consumer (Finance) |
|-------|---------------------|-------------------|
| Order confirmed | `commerce.order.confirmed` | Create receivable |
| Delivery completed | `commerce.delivery.completed` | Recognize revenue |
| Subscription billed | `commerce.subscription.billing.completed` | Create performance obligation |
| Return processed | `commerce.return.completed` | Create credit memo |
| Warranty claim submitted | `commerce.warranty.claim.submitted` | Accrue warranty liability |

### 5.3 Analytics Ownership

Report Service owns the analytics platform (dashboard framework, data warehouse, BI tools). Other services do NOT build independent analytics infrastructure. Instead:

- Each service defines **analytics widgets** that are rendered by the Report Service.
- Each service publishes **domain events** that the Report Service consumes for its data warehouse.
- Embedded analytics in any service's UI are served by the Report Service's widget framework.

**Exception:** Financial Reporting Studio — Finance owns the XBRL templates and regulatory report definitions; Report Service owns the rendering engine.

### 5.4 Employee Self-Service Portal

Owned by HCM Service. Platform Service provides the notification and file storage infrastructure that the portal uses.

### 5.5 Project Time & Expense

Project Service owns project-related time and expense tracking (for project billing and costing). HCM Service owns attendance/time-off tracking (for payroll). Finance Service owns expense management (for reimbursement). The boundary:

- Project Service: Project timesheets → project billing → revenue recognition (Finance)
- HCM Service: Attendance → payroll processing (HCM internal)
- Finance Service: Expense reports → reimbursement → payment

---

## 6. Oracle Fusion Cloud Parity Map

| Oracle Fusion Product Family | Oracle Modules | MSERP Service(s) |
|-----------------------------|---------------|-------------------|
| **Financials** | GL, AP, AR, Fixed Assets, Cash, Expenses, Collections, Revenue, Lease, Grant, JV, Tax | Finance Service |
| **SCM** | Inventory, Procurement, Order Management, Logistics, Product Hub, Sourcing, Supplier Management | Commerce Service (orders, pricing, subscriptions, loyalty) + Manufacturing Service (inventory, procurement, logistics, ATP/CTP) |
| **HCM** | Core HR, Payroll, Recruiting, Performance, Time & Labor, Learning, Benefits, Talent, Global HR | HCM Service |
| **CX** | Sales, Service, Marketing, Commerce, CPQ, Field Service, Surveys, Contact Center, Social, Loyalty, CDP | CRM Service (pre-sale, service, CDP) + Commerce Service (loyalty, B2B portal, omnichannel, A/B testing) |
| **ERP** | Projects, Costing, Billing, Manufacturing, Maintenance, Quality | Project Service + Manufacturing Service |
| **SCM Planning** | Demand, Supply, Inventory Optimization, ATP/CTP | Manufacturing Service (ASCP) + Report Service (forecasting) |
| **Manufacturing** | Execution, Quality, Costing, Planning, Digital Twin, IoT | Manufacturing Service |
| **EPM** | Planning, Budgeting, Forecasting, Consolidation, Reporting, Profitability | Finance Service (budgeting, consolidation, profitability) + Report Service (reporting, CPM, connected planning) |
| **GRC** | Access Controls, Audit, Risk, Compliance, Privacy, SoD | Platform Service (GRC, DLP, privacy, compliance hub) + Auth Service (RBAC, ABAC, SSO, MFA) |
| **Integration** | Integration Cloud, B2B, EDI, Connectors | Integration Service |
| **Platform** | App Builder, Digital Assistant, Mobile, Search, Workflow, Notifications, Scheduler, MDM, RPA, Content, Collaboration | Platform Service + Workflow Service + Integration Service (MDM, connectors) |

---

## 7. Feature-to-Service Registry

Every feature maps to exactly one service. Feature specs are in `docs/07-features/`. Remaining feature domains not listed below are covered by their owning service's spec in `docs/06-services/`.

| Feature | Owning Service | Spec File |
|---------|---------------|-----------|
| Account Reconciliation | Finance Service | `reconciliation.md` |
| Adaptive Intelligence | Platform Service | `adaptive-intelligence.md` |
| API Marketplace | Integration Service | `api-marketplace.md` |
| Collaboration | Platform Service | `collaboration.md` |
| Compliance Hub | Platform Service | `compliance-hub.md` |
| Connected Planning | Report Service | `connected-planning.md` |
| Content Management | Platform Service | `content-management.md` |
| Digital Assistant | Platform Service | `digital-assistant.md` |
| DLP (Data Loss Prevention) | Platform Service | `dlp.md` |
| Dynamic Discounting | Finance Service | `dynamic-discounting.md` |
| EDI (Electronic Data Interchange) | Integration Service | `edi.md` |
| Email Service | Platform Service | `email.md` |
| Enterprise Data Quality | Integration Service | `enterprise-data-quality.md` |
| Event Mesh | Platform Service | `event-mesh.md` |
| Financial Reporting Studio | Finance Service (templates), Report Service (rendering) | `financial-reporting-studio.md` |
| Full-Text Search | Platform Service | `search.md` |
| IDP (Intelligent Document Processing) | Platform Service | `idp.md` |
| Intelligent Close | Finance Service | `intelligent-close.md` |
| Intelligent Process Automation (IPA) | Platform Service | `intelligent-process-automation.md` |
| Multi-Tenancy | Tenant Service | `multi-tenancy.md` |
| Privacy Management | Platform Service | `privacy.md` |
| Supply Chain Collaboration | Commerce Service | `supply-chain-collaboration.md` |

---

## 8. Deployment Topology

See [docs/08-infrastructure/deployment.md](docs/08-infrastructure/deployment.md) for the full deployment specification including multi-region architecture diagrams and Kubernetes topology.

### 8.1 Active-Passive Failover

| Aspect | Specification |
|--------|---------------|
| Failover Trigger | Automated: health check failure > 3 consecutive probes (30s total); Manual: admin-initiated |
| DNS Failover | TTL < 60s, automatic DNS switch |
| Database Promotion | PostgreSQL synchronous streaming replication |
| Event Broker | RabbitMQ federation with queue mirroring |
| RTO | < 1 hour (automated: < 15 minutes) |
| RPO | < 5 minutes (synchronous replication) |
| Split-Brain Prevention | Leader election via etcd with fencing tokens |
| Data Residency | EU tenant data pinned to EU region |

---

## 9. Integration Ecosystem

The Integration Service provides 50+ pre-built connectors. See [docs/06-services/integration.md](docs/06-services/integration.md) for the complete list and Connector SDK specification.

### 9.1 Connector Categories

- **Financial**: SAP S/4HANA, NetSuite, Dynamics 365, QuickBooks, Xero, Sage, Stripe, PayPal, Adyen, Square, Plaid, SWIFT, Vertex, Avalara
- **CRM & CX**: Salesforce, HubSpot, Dynamics 365 Sales, Zendesk, Freshdesk, Intercom, Twilio, SendGrid
- **HR & People**: Workday, BambooHR, ADP, Paychex, LinkedIn Recruiter, Greenhouse
- **Supply Chain & Logistics**: FedEx, UPS, DHL, USPS, Amazon Seller Central, Shopify, Magento, WooCommerce
- **Collaboration & Productivity**: Microsoft 365, Google Workspace, Slack, Microsoft Teams, Jira, Confluence, Notion
- **Infrastructure & Developer**: AWS S3, Azure Blob, GCS, SMTP/IMAP, SFTP, Generic REST/SOAP/Webhook, EDI, OPC-UA, MQTT, Snowflake, Power BI, Tableau

---

## 10. AI/ML Strategy

MSERP embeds AI/ML capabilities across all business modules via a unified ML platform managed by the Report Service (inference) and Platform Service (model lifecycle).

### 10.1 AI Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Business Services Layer                       │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐          │
│  │ Finance  │ │ Commerce │ │   HCM    │ │   CRM    │ ...      │
│  │ Service  │ │ Service  │ │ Service  │ │ Service  │          │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘          │
│       │             │             │             │                │
│  ┌────▼─────────────▼─────────────▼─────────────▼──────────┐   │
│  │              Adaptive Intelligence Layer                  │   │
│  │  Recommendations │ Anomaly Detection │ Predictions │ NLQ │   │
│  └───────────────────────────┬──────────────────────────────┘   │
│                              │                                   │
│  ┌───────────────────────────▼──────────────────────────────┐   │
│  │                    ML Runtime (ONNX + Candle)             │   │
│  │  Model Serving │ Feature Store │ A/B Testing │ Explain   │   │
│  └───────────────────────────┬──────────────────────────────┘   │
│                              │                                   │
│  ┌───────────────────────────▼──────────────────────────────┐   │
│  │                  Model Training Pipeline                   │   │
│  │  Data Prep │ Training │ Validation │ Registry │ Deploy    │   │
│  └───────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 10.2 AI Capabilities by Module

| Module | Capability | Model Type |
|--------|-----------|------------|
| Finance | Anomaly detection, cash flow forecasting, payment delay prediction, reconciliation matching | Time-series, Classification |
| Commerce | Demand forecasting, price optimization, churn prediction, cross-sell/up-sell | Regression, Classification, RL |
| HCM | Attrition prediction, resume screening, workforce demand forecasting | Classification, NLP, Time-series |
| CRM | Lead scoring, next-best-action, deal risk prediction, sentiment analysis | Classification, Regression, NLP |
| Manufacturing | Predictive maintenance, quality prediction, yield optimization, demand planning | Time-series, Classification |
| Project | Risk prediction, timeline estimation, resource optimization | Regression, Classification |
| Platform | Document classification (IDP), data extraction (IDP), NLP assistant, process mining | NLP, CV, Classification |
| Report | NL-to-SQL, auto-insight generation, augmented analytics | NLP, Transformer |

### 10.3 AI Governance

| Principle | Implementation |
|-----------|---------------|
| Explainability | All predictions include SHAP/LIME explainability scores |
| Fairness | Bias testing for protected attributes during validation |
| Privacy | Training data anonymized; PII excluded from feature sets |
| Human Override | All AI suggestions can be overridden; low-confidence flagged for review |
| Audit Trail | All AI decisions logged with input features, model version, confidence score |

---

## 11. Specification Document Index

### 11.1 Architecture & Design

| # | Document | Path |
|---|----------|------|
| 1 | Architecture Overview | [docs/01-architecture/overview.md](docs/01-architecture/overview.md) |
| 2 | Security Architecture | [docs/02-security/overview.md](docs/02-security/overview.md) |
| 3 | Event-Driven Architecture | [docs/04-events/overview.md](docs/04-events/overview.md) |

### 11.2 Services

| # | Document | Path |
|---|----------|------|
| 4 | Services Overview | [docs/06-services/overview.md](docs/06-services/overview.md) |
| 5 | Individual service specs | [docs/06-services/](docs/06-services/) |

### 11.3 API & Data

| # | Document | Path |
|---|----------|------|
| 6 | API Design Standards | [docs/05-api/standards.md](docs/05-api/standards.md) |
| 7 | API Endpoints | [docs/05-api/endpoints.md](docs/05-api/endpoints.md) |
| 8 | Error Codes | [docs/05-api/error-codes.md](docs/05-api/error-codes.md) |
| 9 | Data Architecture | [docs/03-data/overview.md](docs/03-data/overview.md) |
| 10 | Event Catalog | [docs/04-events/catalog.md](docs/04-events/catalog.md) |

### 11.4 Infrastructure & Development

| # | Document | Path |
|---|----------|------|
| 11 | Technology Stack | [docs/08-infrastructure/technology.md](docs/08-infrastructure/technology.md) |
| 12 | Deployment | [docs/08-infrastructure/deployment.md](docs/08-infrastructure/deployment.md) |
| 13 | Observability | [docs/08-infrastructure/observability.md](docs/08-infrastructure/observability.md) |
| 14 | Project Structure | [docs/09-development/project-structure.md](docs/09-development/project-structure.md) |
| 15 | Code Conventions | [docs/09-development/conventions.md](docs/09-development/conventions.md) |
| 16 | Local Setup | [docs/09-development/local-setup.md](docs/09-development/local-setup.md) |

### 11.5 Planning & Reference

| # | Document | Path |
|---|----------|------|
| 17 | Development Phases | [docs/10-planning/phases.md](docs/10-planning/phases.md) |
| 18 | Non-Functional Requirements | [docs/10-planning/nfr.md](docs/10-planning/nfr.md) |
| 19 | Glossary | [docs/glossary.md](docs/glossary.md) |

### 11.6 Feature Specifications

All feature specifications are in [docs/07-features/](docs/07-features/). See §7 for the feature-to-service registry. The 22 feature specs are:

| # | Feature | Owning Service | Spec File |
|---|---------|---------------|-----------|
| 1 | Account Reconciliation | Finance Service | [reconciliation.md](docs/07-features/reconciliation.md) |
| 2 | Adaptive Intelligence | Platform Service | [adaptive-intelligence.md](docs/07-features/adaptive-intelligence.md) |
| 3 | API Marketplace | Integration Service | [api-marketplace.md](docs/07-features/api-marketplace.md) |
| 4 | Collaboration | Platform Service | [collaboration.md](docs/07-features/collaboration.md) |
| 5 | Compliance Hub | Platform Service | [compliance-hub.md](docs/07-features/compliance-hub.md) |
| 6 | Connected Planning | Report Service | [connected-planning.md](docs/07-features/connected-planning.md) |
| 7 | Content Management | Platform Service | [content-management.md](docs/07-features/content-management.md) |
| 8 | Digital Assistant | Platform Service | [digital-assistant.md](docs/07-features/digital-assistant.md) |
| 9 | DLP (Data Loss Prevention) | Platform Service | [dlp.md](docs/07-features/dlp.md) |
| 10 | Dynamic Discounting | Finance Service | [dynamic-discounting.md](docs/07-features/dynamic-discounting.md) |
| 11 | EDI (Electronic Data Interchange) | Integration Service | [edi.md](docs/07-features/edi.md) |
| 12 | Email Service | Platform Service | [email.md](docs/07-features/email.md) |
| 13 | Enterprise Data Quality | Integration Service | [enterprise-data-quality.md](docs/07-features/enterprise-data-quality.md) |
| 14 | Event Mesh | Platform Service | [event-mesh.md](docs/07-features/event-mesh.md) |
| 15 | Financial Reporting Studio | Finance Service (templates), Report Service (rendering) | [financial-reporting-studio.md](docs/07-features/financial-reporting-studio.md) |
| 16 | Full-Text Search | Platform Service | [search.md](docs/07-features/search.md) |
| 17 | IDP (Intelligent Document Processing) | Platform Service | [idp.md](docs/07-features/idp.md) |
| 18 | Intelligent Close | Finance Service | [intelligent-close.md](docs/07-features/intelligent-close.md) |
| 19 | Intelligent Process Automation (IPA) | Platform Service | [intelligent-process-automation.md](docs/07-features/intelligent-process-automation.md) |
| 20 | Multi-Tenancy | Tenant Service | [multi-tenancy.md](docs/07-features/multi-tenancy.md) |
| 21 | Privacy Management | Platform Service | [privacy.md](docs/07-features/privacy.md) |
| 22 | Supply Chain Collaboration | Commerce Service | [supply-chain-collaboration.md](docs/07-features/supply-chain-collaboration.md) |

### 11.7 Security

| # | Document | Path |
|---|----------|------|
| 25 | Security Overview | [docs/02-security/overview.md](docs/02-security/overview.md) |
| 26 | Authorization | [docs/02-security/authorization.md](docs/02-security/authorization.md) |
| 27 | Data Protection | [docs/02-security/data-protection.md](docs/02-security/data-protection.md) |
| 28 | GRC | [docs/02-security/grc.md](docs/02-security/grc.md) |
| 29 | Threat Model | [docs/02-security/threat-model.md](docs/02-security/threat-model.md) |

---

*Document Version: 15.0*
*Last Updated: 2026-03-29*
