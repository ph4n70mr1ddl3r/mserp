# Architecture

## 1. High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              CLIENT LAYER                                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   Web App    │  │  Mobile App  │  │  Desktop App │  │  API Clients │     │
│  │  (React/Ang) │  │(React Native)│  │  (Electron)  │  │  (3rd Party) │     │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘     │
│  ┌──────────────┐  ┌──────────────┐                                         │
│  │   Digital    │  │  Low-code    │                                         │
│  │  Assistant   │  │   Builder    │                                         │
│  └──────────────┘  └──────────────┘                                         │
└─────────────────────────────────────────────────────────────────────────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              GATEWAY LAYER                                   │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                    API Gateway (Traefik primary; Kong alt)             │   │
│  │  • Rate Limiting  • Auth  • Routing  • Load Balancing  • SSL         │   │
│  │  • Feature Flags  • CORS  • Request Compression  • Circuit Breaker   │   │
│  │  • API Versioning  • Request/Response Transformation                  │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           MICROSERVICES LAYER                                │
│                                                                              │
│  ── Core Services (ports 8001-8009) ──────────────────────────────────────   │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐            │
│  │   Auth      │ │  Identity   │ │   Tenant    │ │   Config    │            │
│  │  Service    │ │  Service    │ │  Service    │ │  Service    │            │
│  │  (:8001)    │ │  (:8002)    │ │  (:8003)    │ │  (:8004)    │            │
│  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘            │
│                                                                              │
│  ── Business Services (ports 8010-8019) ──────────────────────────────────   │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐            │
│  │  Commerce   │ │  Finance    │ │     HR      │ │Manufacturing│            │
│  │ (Sales +    │ │ (Finance +  │ │  Service    │ │  Service    │            │
│  │  Inventory) │ │ Procurement)│ │  (:8012)    │ │  (:8013)    │            │
│  │  (:8010)    │ │  (:8011)    │  │             │ │             │            │
│  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘            │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐            │
│  │   Report    │ │  Workflow   │ │   CRM /     │ │  Project    │            │
│  │  Service    │ │  Service    │ │  Marketing  │ │  Mgmt Svc   │            │
│  │  (:8014)    │ │  (:8015)    │ │  (:8016)    │ │  (:8017)    │            │
│  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘            │
│                                                                              │
│  ── Supporting Services (ports 8020-8029) ────────────────────────────────   │
│  ┌─────────────┐ ┌─────────────┐                                            │
│  │  Platform   │ │Integration  │                                            │
│  │ (Notif +    │ │  Service    │                                            │
│  │  File+Audit)│ │  (:8021)    │                                            │
│  │  (:8020)    │ │             │                                            │
│  └─────────────┘ └─────────────┘                                            │
└─────────────────────────────────────────────────────────────────────────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           INFRASTRUCTURE LAYER                               │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐            │
│  │ PostgreSQL  │ │  RabbitMQ   │ │    Redis    │ │ Elasticsearch│           │
│  │  (Primary)  │ │ (Messaging) │ │  (Cache)    │ │  (Search)    │           │
│  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘            │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐            │
│  │ MinIO/S3    │ │ Prometheus  │ │   Jaeger    │ │   Loki      │           │
│  │ (Storage)   │ │ (Metrics)   │ │  (Tracing)  │ │  (Logs)     │           │
│  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘            │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐            │
│  │  Grafana    │ │Alertmanager │ │  DuckDB     │ │  ONNX       │           │
│  │(Dashboards) │ │  (Alerts)   │ │ (Analytics) │ │  Runtime    │           │
│  └─────────────┘ └─────────────┘ └─────────────┘ │ (ML/AI)     │           │
│                                                    └─────────────┘           │
│  ┌─────────────┐ ┌─────────────┐                                             │
│  │  qdrant     │ │ HashiCorp   │                                             │
│  │(Vector Srch)│ │   Vault     │                                             │
│  └─────────────┘ │  (Secrets)  │                                             │
│                   └─────────────┘                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 2. Design Principles

| Principle | Description |
|-----------|-------------|
| Domain-Driven Design | Services organized by business domains with bounded contexts, ubiquitous language per domain |
| Event-Driven Architecture | Async communication via RabbitMQ for loose coupling; choreography-based sagas for distributed transactions |
| Database per Service | Each service owns its data schema exclusively; no cross-service database access |
| API-First | OpenAPI specs defined before implementation; contract-driven development |
| Observable by Default | Structured logging, Prometheus metrics, distributed tracing built into every service |
| Zero Trust Security | mTLS, JWT, RBAC/ABAC at every layer; defense in depth |
| Cloud-Native | Stateless services, externalized configuration, health probes, graceful shutdown |
| Extensibility | Plugin architecture for integrations; feature flags for progressive delivery; low-code builder for customizations |
| AI-First Analytics | Embedded ML models, NLP digital assistant, predictive analytics, anomaly detection integrated into every domain |
| Sustainability by Design | ESG metrics collection built into business processes; carbon tracking as a first-class concern |

## 3. Service Categories

Port ranges are reserved by category: **8001-8009** (core), **8010-8019** (business), **8020-8029** (supporting). Gaps within ranges are reserved for future services.

### 3.1 Core Services (Infrastructure)

| Service | Port | Purpose |
|---------|------|---------|
| Auth | 8001 | Authentication, token management, session handling, OAuth2/OIDC provider |
| Identity | 8002 | User management, roles, permissions, organizations, user preferences |
| Tenant | 8003 | Multi-tenancy, tenant provisioning, feature flags, subscription management |
| Config | 8004 | System configuration, settings management, dynamic runtime configuration |

### 3.2 Business Services

| Service | Port | Purpose |
|---------|------|---------|
| Commerce (Sales + Inventory) | 8010 | Sales operations, customer management, stock, warehousing, pricing engine, PIM, transportation, ATP/CTP, product configurator, credit management, subscription management, drop ship |
| Finance (Finance + Procurement) | 8011 | Financial accounting, purchasing, supplier management, multi-currency, budgeting, treasury, EPM, CLM, expenses, revenue recognition (ASC 606/IFRS 15) |
| HR | 8012 | Human resources, payroll, workforce management, recruitment, performance, talent review, succession, multi-country payroll |
| Manufacturing | 8013 | Production, manufacturing operations, cost accounting, quality management, PLM, EAM |
| Report | 8014 | Analytics, reporting, dashboards, BI, AI-driven insights, ESG, carbon accounting, embedded ML, narrative reporting |
| Workflow | 8015 | Business process automation, approvals, BPMN engine, SLA management |
| CRM / Marketing | 8016 | Customer relationship management, leads, campaigns, marketing automation, field service management, surveys & feedback, sales territory & quota planning |
| Project Management | 8017 | Project planning, resource allocation, time & expense, project billing, EVM |

### 3.3 Supporting Services

| Service | Port | Purpose |
|---------|------|---------|
| Platform (Notif + File + Audit) | 8020 | Notifications, file storage, document management, audit logging, digital assistant, app builder, GRC, enterprise job scheduler, knowledge management, digital signatures |
| Integration | 8021 | External integrations, API management, connector framework, EDI, MDM, data governance, trade compliance |

## 4. Service Consolidation Rationale

Consolidation decisions are made to minimize cross-service transaction complexity while maintaining bounded context integrity. Detailed rationale is documented here only; the service tables in [Services](services.md) reference this section.

### 4.1 Commerce Service (Sales + Inventory)

Sales and inventory are tightly coupled — orders affect stock, stock affects quotations, pricing depends on inventory levels. Kept as a single service to avoid cross-service transaction complexity and enable atomic stock reservation during order creation. Product Information Management (PIM) and Transportation Management are included within Commerce because product data directly feeds pricing and search, and shipment scheduling is tightly coupled with order fulfillment.

### 4.2 Finance Service (Finance + Procurement)

Procurement directly drives accounts payable and financial transactions. A purchase order receipt automatically creates journal entries and AP records. Unified service ensures consistent financial records and eliminates saga overhead for procurement-to-payment flows. Corporate Treasury, Enterprise Expense Management, Enterprise Performance Management (EPM), and Contract Lifecycle Management (CLM) are consolidated here because treasury operations require real-time cash positions from AP/AR, expense management directly feeds the general ledger, EPM builds on budgeting and financial reporting, and CLM automates contract-linked financial transactions.

### 4.3 Platform Service (Notification + File + Audit)

Notifications, file storage, and audit logging are cross-cutting concerns with no complex domain logic of their own. They share common infrastructure patterns (delivery queues, storage backends, append-only writes) and consolidating reduces operational overhead without creating coupling. The Digital Assistant and Application Builder are included here because they are platform-level capabilities that operate across all business domains — the assistant routes to appropriate services, and the app builder composes UIs from existing APIs.

## 5. Communication Patterns

| Pattern | When to Use | Technology | Example |
|---------|-------------|------------|---------|
| Synchronous Request/Response | Read operations, queries requiring immediate response, service-to-service lookups | REST over HTTP (JSON) | `GET /api/v1/commerce/products/{id}` |
| Asynchronous Events | State changes, cross-service side effects, eventual consistency | RabbitMQ (AMQP) | `commerce.order.created` → Finance, Platform, Report |
| Saga (Choreography) | Multi-service distributed transactions with compensating actions | RabbitMQ events with compensating actions | Order fulfillment: Reserve stock → Invoice → Notify |
| Saga (Orchestration) | Multi-step flows requiring central coordination or involving services without inbox queues | Originating service as orchestrator, HTTP for non-event services | Employee onboarding: HR orchestrates → Identity (HTTP) → Platform (event) |
| Bulk / Batch | Large dataset import/export, mass updates | REST bulk endpoints (≤100 items), file-based for larger sets | `POST /api/v1/commerce/products/bulk` |
| Streaming | Real-time data feeds for dashboards, live updates | WebSocket via Platform Service | Real-time order status, live KPI dashboards |
| Digital Assistant | Natural language queries, conversational interactions | NLP intent recognition → service API dispatch | "Show me Q3 revenue by region" → Report Service API |

> **Note:** Service-to-service authentication (mTLS + Service JWT) is a security mechanism that applies to all communication patterns — it is not a separate communication pattern. See [Security](security.md) for details.

## 6. Cross-Cutting Concerns

### 6.1 Internationalization (i18n)

All user-facing text, error messages, date/number formatting, and currency display are localized. The system uses the tenant's locale settings (managed via Config Service) and the `Accept-Language` request header for per-request overrides.

| Aspect | Implementation |
|--------|---------------|
| Locale resolution | `Accept-Language` header → Tenant default → System default (`en-US`) |
| Translation storage | Config Service (tenant-scoped overrides for labels, messages) |
| Date/Time formatting | `chrono` with locale-aware formatters |
| Number/Currency formatting | ICU-compatible formatting via `rust_icu` or equivalent |
| Supported locales | 40+ locales (seeded reference data) |
| RTL support | Right-to-left layout support for Arabic, Hebrew locales |
| Pluralization | Locale-aware plural rules for dynamic content |

### 6.2 Multi-Currency

All financial amounts are stored in a base currency per tenant and converted using exchange rates maintained by the Finance Service.

| Aspect | Implementation |
|--------|---------------|
| Base currency | Configured per tenant (e.g., `USD`, `EUR`) |
| Exchange rates | Daily automated feed from ECB or configurable provider; manual override supported |
| Storage | Amounts stored as `(amount DECIMAL(19,4), currency_code CHAR(3), exchange_rate DECIMAL(19,10))` |
| Triangulation | EUR-based triangulation for non-EUR conversions per ECB convention |
| Realized gains/losses | Automatically calculated on settlement (payment vs. invoice rate difference) |
| Unrealized gains/losses | Calculated during period-end revaluation of open foreign-currency items |
| Corporate rates | Tenant-defined corporate exchange rates overriding market rates for internal transfers |

### 6.3 Data Import/Export Framework

A standardized framework for importing and exporting data across all services, managed by the Integration Service.

| Format | Import | Export | Max Size |
|--------|--------|--------|----------|
| CSV | Yes | Yes | 100MB |
| JSON | Yes | Yes | 50MB |
| Excel (.xlsx) | Yes | Yes | 50MB |
| PDF | No | Yes (reports only) | N/A |
| XML | Yes | Yes | 50MB |
| Parquet | No | Yes (data lake export) | Unlimited |

Import operations are asynchronous — the client submits a file, receives a job ID, and polls for status. Validation errors are returned per-row with line numbers.

### 6.4 Master Data Management (MDM)

Master data is governed centrally by the Integration Service to ensure data quality and consistency across all services.

| Aspect | Implementation |
|--------|---------------|
| Master data domains | Customers, Suppliers, Products, Employees, Chart of Accounts |
| Golden record | Integration Service maintains the authoritative "golden record" per entity |
| Deduplication | Automated matching and merging based on configurable rules (fuzzy + exact) |
| Quality rules | Data quality scorecards with completeness, accuracy, consistency checks |
| Distribution | Master data changes propagated via events to consuming services |
| Data stewardship | Dashboard for manual review of matching conflicts and quality violations |

### 6.5 Mobile Platform

The mobile platform provides native mobile access to all MSERP capabilities.

| Aspect | Implementation |
|--------|---------------|
| Framework | React Native for iOS and Android |
| Offline support | Local SQLite cache with background sync on connectivity |
| Push notifications | FCM (Android), APNs (iOS) via Platform Service |
| Biometric auth | Face ID, Touch ID, fingerprint for mobile MFA |
| Mobile-specific APIs | Optimized endpoints with reduced payload sizes for mobile clients |

### 6.6 Low-Code Application Builder

A visual application builder enabling business users to create custom applications without coding.

| Aspect | Implementation |
|--------|---------------|
| Page designer | Drag-and-drop UI builder with pre-built components |
| Business objects | Custom entities with fields, validations, and relationships |
| Workflow integration | Custom workflows via BPMN designer connected to business objects |
| Data binding | Connect custom UI to any MSERP API endpoint |
| Publishing | Apps published per-tenant with version control |
| Sandbox | Test customizations in sandbox before publishing to production |

### 6.7 ESG / Sustainability

Environmental, Social, and Governance tracking integrated into business operations.

| Aspect | Implementation |
|--------|---------------|
| Carbon emissions | Scope 1, 2, 3 emissions tracking per facility, product, and transaction |
| Energy tracking | Energy consumption monitoring by facility and process |
| ESG scorecards | Configurable ESG KPIs with targets, trends, and benchmarking |
| Regulatory reporting | Frameworks: GRI, SASB, TCFD, EU Taxonomy |
| Carbon offsets | Offset tracking, retirement, and verification |

### 6.8 Enterprise Job Scheduler

Centralized scheduling and execution of background jobs across all services, managed by the Platform Service.

| Aspect | Implementation |
|--------|---------------|
| Job Types | Scheduled reports, data import/export, batch processing, ETL jobs, payroll runs, maintenance scheduling |
| Scheduling | Cron expressions, one-time, recurring, event-triggered |
| Job Dependencies | DAG-based dependency graph with parallel execution support |
| Retry Policy | Configurable retry count, exponential backoff, dead letter for permanently failed jobs |
| Monitoring | Job execution history, duration tracking, failure alerting |
| Concurrency | Per-tenant job concurrency limits to prevent resource exhaustion |

### 6.9 Trade Compliance

Automated trade compliance screening and documentation managed by the Integration Service.

| Aspect | Implementation |
|--------|---------------|
| Restricted Party Screening | Automated screening against government denied party lists (OFAC, EU, UN) |
| Export Controls | License determination, exception management, dual-use goods classification |
| Customs Documentation | Automated generation of commercial invoices, packing lists, certificates of origin |
| Compliance Screening | Real-time screening on order creation, supplier onboarding, shipment dispatch |
| Audit Trail | Complete compliance decision log for regulatory audits |

### 6.10 Revenue Recognition

Automated revenue recognition compliant with ASC 606 and IFRS 15, managed by the Finance Service.

| Aspect | Implementation |
|--------|---------------|
| Standard | ASC 606 (US GAAP) and IFRS 15 compliance |
| Performance Obligations | Automated identification and allocation of performance obligations |
| Revenue Scheduling | Revenue recognition schedules based on satisfaction methods (point-in-time, over-time) |
| Contract Modifications | Handling of contract changes with prospective/retrospective adjustment |
| Allocation | Standalone selling price allocation across multiple performance obligations |
| Variable Consideration | Estimation and constraint of variable consideration (discounts, rebates, contingencies) |

### 6.11 Knowledge Management

Knowledge base authoring and delivery managed by the Platform Service.

| Aspect | Implementation |
|--------|---------------|
| Article Authoring | Rich text editor with version control and approval workflow |
| Categorization | Hierarchical categories with tagging and full-text search |
| Access Control | Role-based access to articles, internal vs. external knowledge bases |
| Integration | Context-sensitive knowledge suggestions in CRM cases and support workflows |
| Analytics | Article view counts, helpfulness ratings, search effectiveness |

### 6.12 Field Service Management

End-to-end field service operations managed by the CRM / Marketing Service.

| Aspect | Implementation |
|--------|---------------|
| Service Orders | Creation from CRM cases, warranty claims, maintenance contracts |
| Technician Scheduling | Skills-based assignment, geographic optimization, calendar integration |
| Parts Logistics | Spare parts reservation from inventory, van stock management |
| SLA Tracking | Response time and resolution time monitoring with escalation |
| Mobile Access | Field technician mobile app with offline capability |

### 6.13 Subscription Management

Full subscription lifecycle management integrated with Commerce and Finance Services.

| Aspect | Implementation |
|--------|---------------|
| Subscription Lifecycle | Create, amend, renew, cancel, suspend subscriptions |
| Billing Models | Recurring, usage-based, tiered, hybrid pricing |
| Billing Schedules | Automated invoice generation on billing cycle dates |
| Amendments | Mid-cycle changes with proration calculations |
| Revenue Recognition | Integration with Finance Service revenue recognition (ASC 606) |
| Churn Management | Automated renewal reminders, cancellation workflows, retention analytics |

### 6.14 Product Configurator

Constraint-based product configuration managed by the Commerce Service.

| Aspect | Implementation |
|--------|---------------|
| Configuration Rules | Constraint-based rules (compatibility, incompatibility, recommendations) |
| Pricing Integration | Dynamic pricing based on selected configuration options |
| BOM Generation | Automatic BOM creation from configured product |
| Validation | Real-time validation of configuration completeness and feasibility |
| Guided Selling | Step-by-step configuration wizard for sales users |

---

*See [Services](services.md) for detailed per-service specifications.*
*See [Events](events.md) for event schemas and cross-domain communication.