# Architecture

## 1. High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              CLIENT LAYER                                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   Web App    │  │  Mobile App  │  │  Desktop App │  │  API Clients │     │
│  │  (React/Ang) │  │  (iOS/Andr)  │  │  (Electron)  │  │  (3rd Party) │     │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘     │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              GATEWAY LAYER                                   │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                    API Gateway (Traefik primary; Kong alt)             │   │
│  │  • Rate Limiting  • Auth  • Routing  • Load Balancing  • SSL         │   │
│  │  • Feature Flags  • CORS  • Request Compression  • Circuit Breaker   │   │
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
│  │  (:8010)    │ │  (:8011)    │ │             │ │             │            │
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
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐                            │
│  │  Grafana    │ │Alertmanager │ │  DuckDB     │                            │
│  │(Dashboards) │ │  (Alerts)   │ │ (Analytics) │                            │
│  └─────────────┘ └─────────────┘ └─────────────┘                            │
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
| Extensibility | Plugin architecture for integrations; feature flags for progressive delivery |

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
| Commerce (Sales + Inventory) | 8010 | Sales operations, customer management, stock, warehousing, pricing engine |
| Finance (Finance + Procurement) | 8011 | Financial accounting, purchasing, supplier management, multi-currency, budgeting |
| HR | 8012 | Human resources, payroll, workforce management, recruitment, performance |
| Manufacturing | 8013 | Production, manufacturing operations, cost accounting, quality management |
| Report | 8014 | Analytics, reporting, dashboards, BI, AI-driven insights |
| Workflow | 8015 | Business process automation, approvals, BPMN engine, SLA management |
| CRM / Marketing | 8016 | Customer relationship management, leads, campaigns, marketing automation |
| Project Management | 8017 | Project planning, resource allocation, time & expense, project billing |

### 3.3 Supporting Services

| Service | Port | Purpose |
|---------|------|---------|
| Platform (Notif + File + Audit) | 8020 | Notifications, file storage, document management, audit logging |
| Integration | 8021 | External integrations, API management, connector framework, EDI |

## 4. Service Consolidation Rationale

Consolidation decisions are made to minimize cross-service transaction complexity while maintaining bounded context integrity. Detailed rationale is documented here only; the service tables in [Services](services.md) reference this section.

### 4.1 Commerce Service (Sales + Inventory)

Sales and inventory are tightly coupled — orders affect stock, stock affects quotations, pricing depends on inventory levels. Kept as a single service to avoid cross-service transaction complexity and enable atomic stock reservation during order creation.

### 4.2 Finance Service (Finance + Procurement)

Procurement directly drives accounts payable and financial transactions. A purchase order receipt automatically creates journal entries and AP records. Unified service ensures consistent financial records and eliminates saga overhead for procurement-to-payment flows.

### 4.3 Platform Service (Notification + File + Audit)

Notifications, file storage, and audit logging are cross-cutting concerns with no complex domain logic of their own. They share common infrastructure patterns (delivery queues, storage backends, append-only writes) and consolidating reduces operational overhead without creating coupling.

## 5. Communication Patterns

| Pattern | When to Use | Technology | Example |
|---------|-------------|------------|---------|
| Synchronous Request/Response | Read operations, queries requiring immediate response, service-to-service lookups | REST over HTTP (JSON) | `GET /api/v1/commerce/products/{id}` |
| Asynchronous Events | State changes, cross-service side effects, eventual consistency | RabbitMQ (AMQP) | `commerce.order.created` → Finance, Platform, Report |
| Saga (Choreography) | Multi-service distributed transactions with compensating actions | RabbitMQ events with compensating actions | Order fulfillment: Reserve stock → Invoice → Notify |
| Saga (Orchestration) | Multi-step flows requiring central coordination or involving services without inbox queues | Originating service as orchestrator, HTTP for non-event services | Employee onboarding: HR orchestrates → Identity (HTTP) → Platform (event) |
| Bulk / Batch | Large dataset import/export, mass updates | REST bulk endpoints (≤100 items), file-based for larger sets | `POST /api/v1/commerce/products/bulk` |

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

### 6.2 Multi-Currency

All financial amounts are stored in a base currency per tenant and converted using exchange rates maintained by the Finance Service.

| Aspect | Implementation |
|--------|---------------|
| Base currency | Configured per tenant (e.g., `USD`, `EUR`) |
| Exchange rates | Daily automated feed from ECB or configurable provider; manual override supported |
| Storage | Amounts stored as `(amount DECIMAL(19,4), currency_code CHAR(3), exchange_rate DECIMAL(19,10))` |
| Triangulation | EUR-based triangulation for non-EUR conversions per ECB convention |
| Realized gains/losses | Automatically calculated on settlement (payment vs. invoice rate difference) |

### 6.3 Data Import/Export Framework

A standardized framework for importing and exporting data across all services, managed by the Integration Service.

| Format | Import | Export | Max Size |
|--------|--------|--------|----------|
| CSV | Yes | Yes | 100MB |
| JSON | Yes | Yes | 50MB |
| Excel (.xlsx) | Yes | Yes | 50MB |
| PDF | No | Yes (reports only) | N/A |

Import operations are asynchronous — the client submits a file, receives a job ID, and polls for status. Validation errors are returned per-row with line numbers.

---

*See [Services](services.md) for detailed per-service specifications.*
*See [Events](events.md) for event schemas and cross-domain communication.*
