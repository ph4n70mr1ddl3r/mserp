# Architecture Overview

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
| Commerce (Sales + Inventory) | 8010 | Sales operations, customer management, stock, warehousing, pricing engine, PIM, transportation, ATP/CTP, product configurator, credit management, subscription management, drop ship, connected logistics, warranty management, loyalty management, omnichannel management, price optimization |
| Finance (Finance + Procurement) | 8011 | Financial accounting, purchasing, supplier management, multi-currency, budgeting, treasury, EPM, CLM, expenses, revenue recognition (ASC 606/IFRS 15), strategic sourcing, supplier risk management, account reconciliation, profitability analysis, lease accounting (ASC 842/IFRS 16), grant management, joint venture accounting, intelligent close, advanced collections, advanced tax management, commodity management, spend analysis, supplier diversity management |
| HR | 8012 | Human resources, payroll, workforce management, recruitment, performance, talent review, succession, multi-country payroll |
| Manufacturing | 8013 | Production, manufacturing operations, cost accounting, quality management, PLM, EAM, Digital Twin, IoT, Manufacturing Intelligence, Digital Thread, enterprise health & safety (EHS), MRO, product compliance & regulatory management |
| Report | 8014 | Analytics, reporting, dashboards, BI, AI-driven insights, ESG, carbon accounting, embedded ML, narrative reporting, process mining, corporate performance management, augmented analytics |
| Workflow | 8015 | Business process automation, approvals, BPMN engine, SLA management |
| CRM / Marketing | 8016 | Customer relationship management, leads, campaigns, marketing automation, field service management, surveys & feedback, sales territory & quota planning, customer data platform (CDP), enterprise contact center, social selling, CX digital & A/B testing |
| Project Management | 8017 | Project planning, resource allocation, time & expense, project billing, EVM |

### 3.3 Supporting Services

| Service | Port | Purpose |
|---------|------|---------|
| Platform (Notif + File + Audit) | 8020 | Notifications, file storage, document management, audit logging, digital assistant, app builder, GRC, enterprise job scheduler, knowledge management, digital signatures, RPA, collaboration, IoT device registry, content management, privacy management, DLP, intelligent document processing (IDP), ITSM, compliance hub |
| Integration | 8021 | External integrations, API management, API marketplace, connector framework, EDI, MDM, data governance, trade compliance, event mesh, blockchain integration |

## 4. Service Consolidation Rationale

Consolidation decisions are made to minimize cross-service transaction complexity while maintaining bounded context integrity. Detailed rationale is documented here only; the service tables in [Services](../services/overview.md) reference this section.

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

> **Note:** Service-to-service authentication (mTLS + Service JWT) is a security mechanism that applies to all communication patterns — it is not a separate communication pattern. See [Security](../security.md) for details.

*See [Services](../services/overview.md) for detailed per-service specifications.*
