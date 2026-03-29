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

> **Full specification:** [docs/01-architecture/overview.md](docs/01-architecture/overview.md)

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

---

## 3. Service Catalog

MSERP consists of 14 microservices organized into 3 tiers. Each service owns its database exclusively. **This section is the PRIMARY reference for service registry.**

### 3.1 Service Registry

| # | Service | Port | Database | Tier | Responsibility |
|---|---------|------|----------|------|---------------|
| 1 | Auth Service | 8001 | `auth_db` | Core | Authentication, JWT, MFA, SSO, session management |
| 2 | Identity Service | 8002 | `identity_db` | Core | Users, roles, permissions, RBAC/ABAC, organizations |
| 3 | Tenant Service | 8003 | `tenant_db` | Core | Multi-tenancy, feature flags, subscriptions, quotas |
| 4 | Config Service | 8004 | `config_db` | Core | Dynamic configuration, localization, ESG settings |
| 5 | Commerce Service | 8010 | `commerce_db` | Business | Order lifecycle, inventory, pricing, PIM, logistics, subscriptions, loyalty, order orchestration, backorder management |
| 6 | Finance Service | 8011 | `finance_db` | Business | GL, AP/AR, treasury, revenue recognition, budgeting, procurement, tax, multi-book accounting, intercompany matching |
| 7 | HCM Service | 8012 | `hr_db` | Business | Employee lifecycle, payroll, recruitment, talent, benefits, self-service, HR help desk, LXP, position control |
| 8 | Manufacturing Service | 8013 | `manufacturing_db` | Business | BOM, work orders, PLM, EAM, quality, digital twin, EHS, process manufacturing, mixed-mode manufacturing |
| 9 | Report Service | 8014 | `report_db` | Business | Analytics, dashboards, BI, ESG reporting, data warehouse, ML platform, semantic layer, report bursting |
| 10 | Workflow Service | 8015 | `workflow_db` | Business | BPMN engine, approvals, SLA management, business rules, SoD enforcement |
| 11 | CRM Service | 8016 | `crm_db` | Business | Pre-sale (leads, deals, campaigns), field service, CDP, contact center, surveys, sales forecasting, partner management |
| 12 | Project Service | 8017 | `project_db` | Business | Projects, resources, billing, EVM, portfolio analysis |
| 13 | Platform Service | 8020 | `platform_db` + `audit_db` | Supporting | Notifications, files, GRC, digital assistant, IDP, RPA/IPA, ITSM, job scheduler, multi-channel assistant, regulatory content |
| 14 | Integration Service | 8021 | `integration_db` | Supporting | Connectors, EDI, MDM, data quality, trade compliance, blockchain, API marketplace, event mesh, flow designer, MFT |

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

### 3.3 Event Bindings

Core services (Auth, Identity, Tenant, Config) do **not** have inbox queues. Auth Service consumes NO events — it uses HTTP to query Identity Service for user/role data during login. Identity Service does not have an inbox queue and communicates with other services via HTTP only.

All 10 services with inbox queues self-bind to `{domain}.#` for saga compensation:

| Service Inbox | Self-Binding | Cross-Domain Bindings |
|--------------|-------------|----------------------|
| `commerce.inbox` | `commerce.#` | `finance.purchase-order.#`, `finance.invoice.#`, `finance.sourcing.#`, `finance.cash-application.#`, `manufacturing.work-order.#`, `manufacturing.digital-twin.#`, `crm.opportunity.won`, `crm.cdp.#` |
| `finance.inbox` | `finance.#` | `commerce.order.#`, `commerce.stock.#`, `commerce.subscription.#`, `commerce.b2b.#`, `commerce.warranty.#`, `hr.payroll.#`, `manufacturing.work-order.#`, `project.invoice.#`, `project.milestone.#`, `platform.idp.#` |
| `hr.inbox` | `hr.#` | `workflow.step.#` |
| `manufacturing.inbox` | `manufacturing.#` | `commerce.stock.#`, `commerce.order.#`, `commerce.warranty.#`, `platform.iot.device.#` |
| `report.inbox` | `report.#` | `commerce.#`, `finance.#`, `hr.#`, `manufacturing.#`, `crm.#`, `project.#`, `workflow.#`, `platform.audit.#`, `platform.rpa.#`, `tenant.feature.#`, `integration.trade-compliance.#` |
| `workflow.inbox` | `workflow.#` | `hr.leave.#`, `commerce.order.#`, `commerce.credit.#`, `finance.purchase-order.#`, `finance.expense.#`, `finance.intelligent-close.#`, `finance.supplier-risk.#`, `manufacturing.eco.#`, `report.process.#`, `platform.rpa.#`, `platform.grc.#`, `integration.trade-compliance.#` |
| `platform.inbox` | `platform.#` | `auth.login.#`, `hr.#`, `commerce.order.#`, `commerce.credit.#`, `commerce.shipment.#`, `commerce.logistics.#`, `finance.lease.#`, `finance.supplier-risk.#`, `finance.intelligent-close.#`, `integration.trade-compliance.#`, `crm.cdp.#`, `report.process.#`, `manufacturing.intelligence.#`, `tenant.feature.#` |
| `integration.inbox` | `integration.#` | `commerce.customer.#`, `commerce.product.#`, `finance.supplier.#`, `hr.employee.#`, `identity.user.#` |
| `crm.inbox` | `crm.#` | `commerce.customer.#`, `commerce.order.#` |
| `project.inbox` | `project.#` | `hr.employee.#`, `commerce.order.#` |

> All services also bind to `config.changed`. Full event catalog: [docs/04-events/catalog.md](docs/04-events/catalog.md)

### 3.4 Key Boundary Rules

| Rule | Detail |
|------|--------|
| **Auth Service** | Consumes NO events. Uses HTTP to query Identity Service for user/role data during login. Identity publishes events consumed by Platform (security audit) and HR (provisioning check). |
| **IoT Device Registry** | Platform Service owns the authoritative device registry and publishes `platform.iot.device.*` events. Manufacturing's `iot_devices` table is a **local cache** synced by consuming `platform.iot.device.*` events. Manufacturing publishes `manufacturing.iot.telemetry.*` events (telemetry only), NOT device registration events. |
| **SoD Rules** | Platform Service owns GRC-level SoD rule definitions (`grc_sod_rules` table). Workflow Service consumes `platform.grc.sod.conflict.detected` events for approval-level enforcement. Workflow queries Platform API for SoD rules — it does NOT have its own `sod_rules` table. |
| **MDM Golden Records** | Integration Service consumes master data events from all services (`commerce.customer.#`, `commerce.product.#`, `finance.supplier.#`, `hr.employee.#`, `identity.user.#`) for MDM golden record management. |
| **Event Mesh** | Owned by Integration Service (infrastructure). Provides RabbitMQ topology management, event gateway, cross-region replication. See [docs/07-features/event-mesh.md](docs/07-features/event-mesh.md). |

---

## 4. Service Domain Ownership

Every feature, entity, API endpoint, database table, and event belongs to EXACTLY ONE service.

### 4.1 Ownership Summary

| Domain | Owning Service | Other Services May |
|--------|---------------|-------------------|
| Pre-sale (leads, opportunities, deals, campaigns) | CRM Service | — |
| Order lifecycle (quotes, orders, deliveries, returns) | Commerce Service | Read order data via events |
| Pricing rules & execution | Commerce Service | — |
| Financial accounting (GL, AP, AR, tax, treasury) | Finance Service | — |
| Budgeting, forecasting, multi-book accounting | Finance Service | — |
| Revenue recognition (ASC 606 / IFRS 15) | Finance Service | Commerce sends contract events |
| Customer profiles & CDP | CRM Service | Commerce provides transactional data |
| Inventory & warehousing | Commerce Service | Manufacturing sends production completions |
| Manufacturing (BOM, work orders, quality, PLM) | Manufacturing Service | Commerce reads ATP data |
| IoT device registry & certificates | Platform Service | Manufacturing caches device metadata locally |
| IoT telemetry, alerts, edge processing | Manufacturing Service | Commerce reads condition data |
| Employee lifecycle, payroll, HR help desk, LXP | HCM Service | — |
| Projects & resources | Project Service | — |
| Reporting platform, semantic layer, dashboards | Report Service | Finance provides report templates |
| Financial report templates (XBRL, regulatory) | Finance Service | Rendered by Report Service |
| Workflow/BPM engine | Workflow Service | All services use workflow |
| SoD rule definitions (GRC-level) | Platform Service | Workflow queries Platform for rules |
| Notifications & alerts | Platform Service | All services publish notification requests |
| File storage & ECM | Platform Service | All services use file APIs |
| RPA & IPA | Platform Service | — |
| Integration, connectors, event mesh, MFT | Integration Service | All services register connectors |
| MDM golden records | Integration Service | All services publish master data events |
| Enterprise data quality | Integration Service | — |
| Authentication & tokens | Auth Service | All services validate tokens |
| Identity & RBAC/ABAC | Identity Service | All services check permissions |
| Tenant management | Tenant Service | All services enforce tenant isolation |
| Configuration | Config Service | All services consume config |
| Digital assistant (NLP/chat, multi-channel) | Platform Service | — |
| IDP (document processing) | Platform Service | — |
| ITSM | Platform Service | — |
| GRC & compliance hub | Platform Service | Integration owns trade compliance screening data |
| Privacy & DLP | Platform Service | — |
| Knowledge management | Platform Service | — |
| Job scheduler | Platform Service | All services register jobs |
| Collaboration | Platform Service | — |
| Email service | Platform Service | — |
| Loyalty programs | Commerce Service | CRM provides customer segments |
| Omnichannel | Commerce Service | — |
| Subscriptions | Commerce Service | Finance handles revenue recognition |
| Credit management | Commerce Service | — |
| ESG & sustainability | Report Service | All services emit emissions data |
| Trade compliance | Integration Service | — |
| Process manufacturing | Manufacturing Service | — |
| Sales forecasting (pipeline-based) | CRM Service | — |
| Partner relationship management | CRM Service | — |

> **Full domain ownership table with modules:** See [AGENTS.md](AGENTS.md) §Service Boundary Rules and individual service specs in [docs/06-services/](docs/06-services/).

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

Report Service owns the analytics platform (dashboard framework, data warehouse, BI tools, semantic layer). Other services do NOT build independent analytics infrastructure. Instead:

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
| **Financials** | GL, AP, AR, Fixed Assets, Cash, Expenses, Collections, Revenue, Lease, Grant, JV, Tax, Multi-Book | Finance Service |
| **SCM** | Inventory, Procurement, Order Management, Logistics, Product Hub, Sourcing, Supplier Management | Commerce Service + Manufacturing Service |
| **HCM** | Core HR, Payroll, Recruiting, Performance, Time & Labor, Learning, Benefits, Talent, Global HR, Help Desk, LXP | HCM Service |
| **CX** | Sales, Service, Marketing, Commerce, CPQ, Field Service, Surveys, Contact Center, Social, Loyalty, CDP, Forecasting, Partners | CRM Service + Commerce Service |
| **ERP** | Projects, Costing, Billing, Manufacturing, Maintenance, Quality | Project Service + Manufacturing Service |
| **SCM Planning** | Demand, Supply, Inventory Optimization, ATP/CTP | Manufacturing Service + Report Service |
| **Manufacturing** | Execution, Quality, Costing, Planning, Digital Twin, IoT, Process Mfg, Mixed-Mode | Manufacturing Service |
| **EPM** | Planning, Budgeting, Forecasting, Consolidation, Reporting, Profitability | Finance Service + Report Service |
| **GRC** | Access Controls, Audit, Risk, Compliance, Privacy, SoD | Platform Service + Auth Service |
| **Integration** | Integration Cloud, B2B, EDI, Connectors, Flow Designer, MFT | Integration Service |
| **Platform** | App Builder, Digital Assistant, Mobile, Search, Workflow, Notifications, Scheduler, MDM, RPA, Content, Collaboration | Platform Service + Workflow Service + Integration Service |

---

## 7. Feature-to-Service Registry

Every feature maps to exactly one service. The 22 features with dedicated specs are:

| Feature | Owning Service | Spec File |
|---------|---------------|-----------|
| Account Reconciliation | Finance Service | [reconciliation.md](docs/07-features/reconciliation.md) |
| Adaptive Intelligence | Platform Service | [adaptive-intelligence.md](docs/07-features/adaptive-intelligence.md) |
| API Marketplace | Integration Service | [api-marketplace.md](docs/07-features/api-marketplace.md) |
| Collaboration | Platform Service | [collaboration.md](docs/07-features/collaboration.md) |
| Compliance Hub | Platform Service | [compliance-hub.md](docs/07-features/compliance-hub.md) |
| Connected Planning | Report Service | [connected-planning.md](docs/07-features/connected-planning.md) |
| Content Management | Platform Service | [content-management.md](docs/07-features/content-management.md) |
| Digital Assistant | Platform Service | [digital-assistant.md](docs/07-features/digital-assistant.md) |
| DLP | Platform Service | [dlp.md](docs/07-features/dlp.md) |
| Dynamic Discounting | Finance Service | [dynamic-discounting.md](docs/07-features/dynamic-discounting.md) |
| EDI | Integration Service | [edi.md](docs/07-features/edi.md) |
| Email Service | Platform Service | [email.md](docs/07-features/email.md) |
| Enterprise Data Quality | Integration Service | [enterprise-data-quality.md](docs/07-features/enterprise-data-quality.md) |
| Event Mesh | Integration Service | [event-mesh.md](docs/07-features/event-mesh.md) |
| Financial Reporting Studio | Finance Service (templates), Report Service (rendering) | [financial-reporting-studio.md](docs/07-features/financial-reporting-studio.md) |
| Full-Text Search | Platform Service | [search.md](docs/07-features/search.md) |
| IDP | Platform Service | [idp.md](docs/07-features/idp.md) |
| Intelligent Close | Finance Service | [intelligent-close.md](docs/07-features/intelligent-close.md) |
| IPA (RPA + AI) | Platform Service | [intelligent-process-automation.md](docs/07-features/intelligent-process-automation.md) |
| Multi-Tenancy | Tenant Service | [multi-tenancy.md](docs/07-features/multi-tenancy.md) |
| Privacy Management | Platform Service | [privacy.md](docs/07-features/privacy.md) |
| Supply Chain Collaboration | Commerce Service | [supply-chain-collaboration.md](docs/07-features/supply-chain-collaboration.md) |

> **Full feature registry with domain modules:** See [AGENTS.md](AGENTS.md) §Feature Spec Registry. All feature specs: [docs/07-features/](docs/07-features/)

---

## 8. Deployment Topology

> **Full specification:** [docs/08-infrastructure/deployment.md](docs/08-infrastructure/deployment.md)

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

> **Full connector list and Connector SDK:** [docs/06-services/integration.md](docs/06-services/integration.md)

The Integration Service provides 50+ pre-built connectors:

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
| Finance | Anomaly detection, cash flow forecasting, payment delay prediction, reconciliation matching, intercompany matching | Time-series, Classification |
| Commerce | Demand forecasting, price optimization, churn prediction, cross-sell/up-sell, fulfillment rule optimization | Regression, Classification, RL |
| HCM | Attrition prediction, resume screening, workforce demand forecasting, learning path recommendations | Classification, NLP, Time-series |
| CRM | Lead scoring, next-best-action, deal risk prediction, sentiment analysis, pipeline forecasting | Classification, Regression, NLP |
| Manufacturing | Predictive maintenance, quality prediction, yield optimization, demand planning, process parameter optimization | Time-series, Classification |
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

All 22 feature specifications are in [docs/07-features/](docs/07-features/). See §7 for the feature-to-service registry.

### 11.7 Security

| # | Document | Path |
|---|----------|------|
| 20 | Security Overview | [docs/02-security/overview.md](docs/02-security/overview.md) |
| 21 | Authorization | [docs/02-security/authorization.md](docs/02-security/authorization.md) |
| 22 | Data Protection | [docs/02-security/data-protection.md](docs/02-security/data-protection.md) |
| 23 | GRC | [docs/02-security/grc.md](docs/02-security/grc.md) |
| 24 | Threat Model | [docs/02-security/threat-model.md](docs/02-security/threat-model.md) |

---

*Document Version: 16.0*
*Last Updated: 2026-03-29*
