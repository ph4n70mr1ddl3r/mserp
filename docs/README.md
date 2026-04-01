# MSERP Documentation

> **Start here:** Read [SPEC.md](../SPEC.md) for the authoritative master specification and [AGENTS.md](../AGENTS.md) for AI agent development instructions.

## Document Hierarchy

```
SPEC.md (Authoritative single source of truth)
├── AGENTS.md (AI agent development guide and workflow)
└── docs/
    ├── 01-architecture/     (system design)
    ├── 02-security/         (auth, authz, data protection)
    ├── 03-data/             (databases, caching, warehouse)
    ├── 04-events/           (event bus, catalog, sagas)
    ├── 05-api/              (standards, endpoints, errors)
    ├── 06-services/         (per-service specifications)
    ├── 07-features/         (cross-cutting & complex feature specs — 61 feature specifications)
    ├── 08-infrastructure/   (deployment, technology, observability)
    ├── 09-development/      (project structure, conventions, setup)
    └── 10-planning/         (phases, NFRs)
```

> **Note:** Most feature-level detail now lives in the service specs (`06-services/`). The 61 feature specs in `07-features/` cover only cross-cutting or complex capabilities that warrant dedicated treatment beyond their service spec.

---

## 01 — Architecture & Design

| Document | Description |
|----------|-------------|
| [Architecture Overview](01-architecture/overview.md) | High-level architecture, design principles, service topology, communication patterns |

## 02 — Security

| Document | Description |
|----------|-------------|
| [Security Overview](02-security/overview.md) | Authentication flow, JWT, service-to-service auth, CORS |
| [Authorization](02-security/authorization.md) | RBAC, ABAC, permission format, 40 predefined roles |
| [Data Protection](02-security/data-protection.md) | Encryption, PII handling, key management |
| [GRC](02-security/grc.md) | SoD, risk management, compliance, privacy, trade compliance |
| [Threat Model](02-security/threat-model.md) | 36 threat categories, incident response |

## 03 — Data

| Document | Description |
|----------|-------------|
| [Data Architecture](03-data/overview.md) | Database-per-service, standard columns, multi-tenancy (RLS), migrations, soft delete, audit trail |
| [Caching Strategy](03-data/caching.md) | Redis cache-aside, 29 cache policies, invalidation strategies |
| [Data Pipeline](03-data/data-pipeline.md) | Data lake (MinIO zones), ETL pipeline, DuckDB star schema warehouse |
| [Domain Data Models](03-data/domain-models.md) | 33 domain data models (revenue recognition, lease, grants, JV, MDM golden records, CDP, IoT, etc.) |

## 04 — Events

| Document | Description |
|----------|-------------|
| [Event-Driven Architecture](04-events/overview.md) | RabbitMQ exchange, inbox bindings, outbox pattern, event versioning, DLQ |
| [Event Catalog](04-events/catalog.md) | All domain events by service (290+ events), cross-domain event flows |
| [Saga Patterns](04-events/sagas.md) | 11 distributed transaction patterns with compensating actions |

## 05 — API

| Document | Description |
|----------|-------------|
| [API Design Standards](05-api/standards.md) | REST standards, pagination, idempotency, versioning, health checks, bulk operations |
| [API Endpoints](05-api/endpoints.md) | Complete endpoint listings for all 14 services (600+ endpoints) |
| [Error Codes](05-api/error-codes.md) | Complete error code taxonomy (210+ codes) |

## 06 — Services

> For the full service catalog including port assignments and database names, see [SPEC.md §3](../SPEC.md).

| Document | Description |
|----------|-------------|
| [Services Overview](06-services/overview.md) | Service categories, consolidation rationale |
| [Auth Service](06-services/auth.md) | OAuth2/OIDC, MFA, SSO, JWT, sessions |
| [Identity Service](06-services/identity.md) | Users, roles, RBAC/ABAC, org hierarchy |
| [Tenant Service](06-services/tenant.md) | Multi-tenancy, feature flags, quotas |
| [Config Service](06-services/config.md) | Dynamic configuration, localization |
| [Commerce Service](06-services/commerce.md) | Orders, inventory, pricing, PIM, logistics, subscriptions, loyalty |
| [Finance Service](06-services/finance.md) | GL, AP/AR, treasury, revenue recognition, procurement, tax |
| [HCM Service](06-services/hr.md) | Employee lifecycle, payroll, recruitment, talent, self-service |
| [Manufacturing Service](06-services/manufacturing.md) | BOM, work orders, PLM, EAM, quality, IoT, digital twin, EHS |
| [Report Service](06-services/report.md) | Analytics, dashboards, BI, ESG, ML/AI, process mining |
| [Workflow Service](06-services/workflow.md) | BPMN engine, approvals, SLA, business rules |
| [CRM Service](06-services/crm.md) | Pre-sale (leads, deals, campaigns), CDP, field service, contact center |
| [Project Service](06-services/project.md) | Projects, resources, billing, EVM, portfolio |
| [Platform Service](06-services/platform.md) | Notifications, files, GRC, assistant, IDP, RPA/IPA, ITSM |
| [Integration Service](06-services/integration.md) | Connectors, EDI, MDM, data quality, trade compliance |

## 07 — Feature Specifications

> Feature specs that were fully covered by their service spec have been consolidated into `06-services/`. The 55 feature specs below cover cross-cutting or complex features that require dedicated treatment.

### Finance Service

| Feature | Spec |
|---------|------|
| Account Reconciliation | [reconciliation.md](07-features/reconciliation.md) |
| Intelligent Close | [intelligent-close.md](07-features/intelligent-close.md) |
| Dynamic Discounting | [dynamic-discounting.md](07-features/dynamic-discounting.md) |
| Financial Reporting Studio | [financial-reporting-studio.md](07-features/financial-reporting-studio.md) |
| Fixed Asset Lifecycle | [fixed-asset-lifecycle.md](07-features/fixed-asset-lifecycle.md) |
| Subledger Accounting Engine | [subledger-accounting.md](07-features/subledger-accounting.md) |
| Advanced Tax Filing | [advanced-tax-filing.md](07-features/advanced-tax-filing.md) |

### Commerce Service

| Feature | Spec |
|---------|------|
| Supply Chain Collaboration | [supply-chain-collaboration.md](07-features/supply-chain-collaboration.md) |
| Order Orchestration | [order-orchestration.md](07-features/order-orchestration.md) |
| B2C Commerce & Storefront | [b2c-commerce.md](07-features/b2c-commerce.md) |
| Advanced Demand Management | [advanced-demand-management.md](07-features/advanced-demand-management.md) |
| Global Order Promising | [global-order-promising.md](07-features/global-order-promising.md) |
| Advanced Warehouse Management (WMS) | [advanced-warehouse-management.md](07-features/advanced-warehouse-management.md) |
| Advanced Transportation Management (TMS) | [advanced-transportation.md](07-features/advanced-transportation.md) |
| CPQ (Configure-Price-Quote) | [cpq.md](07-features/cpq.md) |
| Retail POS | [retail-pos.md](07-features/retail-pos.md) |
| Property Management | [property-management.md](07-features/property-management.md) |

### HCM Service

| Feature | Spec |
|---------|------|
| Goals & Career Development | [goals-career-development.md](07-features/goals-career-development.md) |
| Structured Onboarding Journey | [onboarding-journey.md](07-features/onboarding-journey.md) |

### Manufacturing Service

| Feature | Spec |
|---------|------|
| MES | [mes.md](07-features/mes.md) |
| Production Scheduling | [production-scheduling.md](07-features/production-scheduling.md) |

### Report Service

| Feature | Spec |
|---------|------|
| Connected Planning | [connected-planning.md](07-features/connected-planning.md) |
| Self-Service ML Studio | [self-service-ml.md](07-features/self-service-ml.md) |

### Platform Service

| Feature | Spec |
|---------|------|
| Adaptive Intelligence | [adaptive-intelligence.md](07-features/adaptive-intelligence.md) |
| Digital Assistant | [digital-assistant.md](07-features/digital-assistant.md) |
| IDP (Intelligent Document Processing) | [idp.md](07-features/idp.md) |
| Intelligent Process Automation (RPA & IPA) | [intelligent-process-automation.md](07-features/intelligent-process-automation.md) |
| Enterprise Collaboration | [collaboration.md](07-features/collaboration.md) |
| Content Management (ECM) | [content-management.md](07-features/content-management.md) |
| Compliance Hub | [compliance-hub.md](07-features/compliance-hub.md) |
| DLP | [dlp.md](07-features/dlp.md) |
| Privacy Management | [privacy.md](07-features/privacy.md) |
| Email Service | [email.md](07-features/email.md) |
| Full-Text Search | [search.md](07-features/search.md) |
| Application Composer | [application-composer.md](07-features/application-composer.md) |
| Access Certification | [access-certification.md](07-features/access-certification.md) |
| Policy Automation | [policy-automation.md](07-features/policy-automation.md) |

### Integration Service

| Feature | Spec |
|---------|------|
| API Marketplace | [api-marketplace.md](07-features/api-marketplace.md) |
| Enterprise Data Quality | [enterprise-data-quality.md](07-features/enterprise-data-quality.md) |
| EDI | [edi.md](07-features/edi.md) |
| Event Mesh | [event-mesh.md](07-features/event-mesh.md) |
| Global Trade Management | [global-trade-management.md](07-features/global-trade-management.md) |

### CRM Service

| Feature | Spec |
|---------|------|
| Marketing Automation | [marketing-automation.md](07-features/marketing-automation.md) |

### Cross-Cutting

| Feature | Spec | Service |
|---------|------|---------|
| Multi-Tenancy | [multi-tenancy.md](07-features/multi-tenancy.md) | Tenant Service |
| Construction & Engineering | [construction-engineering.md](07-features/construction-engineering.md) | Project Service |

## 08 — Infrastructure & Operations

| Document | Description |
|----------|-------------|
| [Technology Stack](08-infrastructure/technology.md) | Rust 1.83+, Axum, PostgreSQL 16, RabbitMQ 3.12, all crate versions |
| [Deployment](08-infrastructure/deployment.md) | Kubernetes, CI/CD (GitHub Actions + ArgoCD), health probes, multi-region, IaC |
| [Monitoring & Observability](08-infrastructure/observability.md) | 46+ Prometheus metrics, 37 dashboards, 35+ alerts, OpenTelemetry tracing |

## 09 — Development

| Document | Description |
|----------|-------------|
| [Project Structure](09-development/project-structure.md) | Monorepo layout, 14 service directories, shared crates, migrations, IaC |
| [Code Conventions](09-development/conventions.md) | Naming, error handling, module structure, async patterns, testing |
| [Local Setup](09-development/local-setup.md) | Prerequisites, Docker Compose, port mapping, database migrations |

## 10 — Planning

| Document | Description |
|----------|-------------|
| [Development Phases](10-planning/phases.md) | 5-phase implementation roadmap (38 weeks) |
| [Non-Functional Requirements](10-planning/nfr.md) | Performance, availability, scalability, compliance, testing strategy |

## Reference

| Document | Description |
|----------|-------------|
| [Glossary](glossary.md) | 200+ terms, abbreviations, references |
