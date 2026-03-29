# MSERP Documentation

> **Start here:** Read [SPEC.md](../SPEC.md) for the authoritative master specification and [AGENTS.md](../AGENTS.md) for AI agent development instructions.

## Document Hierarchy

```
SPEC.md (authoritative)
├── AGENTS.md (AI agent guide)
└── docs/
    ├── 01-architecture/     (system design)
    ├── 02-security/         (auth, authz, data protection)
    ├── 03-data/             (databases, caching, warehouse)
    ├── 04-events/           (event bus, catalog, sagas)
    ├── 05-api/              (standards, endpoints, errors)
    ├── 06-services/         (per-service specifications)
    ├── 07-features/         (cross-cutting & complex feature specs — 22 remaining)
    ├── 08-infrastructure/   (deployment, technology, observability)
    ├── 09-development/      (project structure, conventions, setup)
    └── 10-planning/         (phases, NFRs)
```

> **Note:** Most feature-level detail now lives in the service specs (`06-services/`). The 22 feature specs in `07-features/` cover only cross-cutting or complex capabilities that warrant dedicated treatment beyond their service spec.

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
| [Data Warehouse](03-data/warehouse.md) | DuckDB star schema, ETL pipeline |
| [Data Lake](03-data/data-lake.md) | MinIO zones (raw/curated/analytics), schema-on-read |
| [Domain Data Models](03-data/domain-models.md) | 27 domain data models (revenue recognition, lease, grants, JV, etc.), MDM golden records, CDP schema, IoT telemetry, data quality rules |

## 04 — Events

| Document | Description |
|----------|-------------|
| [Event-Driven Architecture](04-events/overview.md) | RabbitMQ exchange, inbox bindings, outbox pattern, event versioning, DLQ |
| [Event Catalog](04-events/catalog.md) | All domain events by service (300+ events), cross-domain event flows |
| [Saga Patterns](04-events/sagas.md) | 9 distributed transaction patterns with compensating actions |

## 05 — API

| Document | Description |
|----------|-------------|
| [API Design Standards](05-api/standards.md) | REST standards, pagination, idempotency, versioning, health checks, bulk operations |
| [API Endpoints](05-api/endpoints.md) | Complete endpoint listings for all 14 services (400+ endpoints) |
| [Error Codes](05-api/error-codes.md) | Complete error code taxonomy (180+ codes) |

## 06 — Services

| Document | Port | DB | Description |
|----------|------|----|-------------|
| [Services Overview](06-services/overview.md) | — | — | Service categories, consolidation rationale |
| [Auth Service](06-services/auth.md) | 8001 | `auth_db` | OAuth2/OIDC, MFA, SSO, JWT, sessions |
| [Identity Service](06-services/identity.md) | 8002 | `identity_db` | Users, roles, RBAC/ABAC, org hierarchy |
| [Tenant Service](06-services/tenant.md) | 8003 | `tenant_db` | Multi-tenancy, feature flags, quotas |
| [Config Service](06-services/config.md) | 8004 | `config_db` | Dynamic configuration, localization |
| [Commerce Service](06-services/commerce.md) | 8010 | `commerce_db` | Orders, inventory, pricing, PIM, logistics, subscriptions, loyalty |
| [Finance Service](06-services/finance.md) | 8011 | `finance_db` | GL, AP/AR, treasury, revenue recognition, procurement, tax |
| [HCM Service](06-services/hr.md) | 8012 | `hr_db` | Employee lifecycle, payroll, recruitment, talent, self-service |
| [Manufacturing Service](06-services/manufacturing.md) | 8013 | `manufacturing_db` | BOM, work orders, PLM, EAM, quality, IoT, digital twin, EHS |
| [Report Service](06-services/report.md) | 8014 | `report_db` | Analytics, dashboards, BI, ESG, ML/AI, process mining |
| [Workflow Service](06-services/workflow.md) | 8015 | `workflow_db` | BPMN engine, approvals, SLA, business rules |
| [CRM Service](06-services/crm.md) | 8016 | `crm_db` | Pre-sale (leads, deals, campaigns), CDP, field service, contact center |
| [Project Service](06-services/project.md) | 8017 | `project_db` | Projects, resources, billing, EVM, portfolio |
| [Platform Service](06-services/platform.md) | 8020 | `platform_db` + `audit_db` | Notifications, files, GRC, assistant, IDP, RPA/IPA, ITSM |
| [Integration Service](06-services/integration.md) | 8021 | `integration_db` | Connectors, EDI, MDM, data quality, trade compliance |

## 07 — Feature Specifications

> Feature specs that were fully covered by their service spec have been consolidated into `06-services/`. The 22 remaining specs below cover cross-cutting or complex features that require dedicated treatment.

### Finance Service

| Feature | Spec |
|---------|------|
| Account Reconciliation | [reconciliation.md](07-features/reconciliation.md) |
| Intelligent Close | [intelligent-close.md](07-features/intelligent-close.md) |
| Dynamic Discounting | [dynamic-discounting.md](07-features/dynamic-discounting.md) |
| Financial Reporting Studio | [financial-reporting-studio.md](07-features/financial-reporting-studio.md) |

### Commerce Service

| Feature | Spec |
|---------|------|
| Supply Chain Collaboration | [supply-chain-collaboration.md](07-features/supply-chain-collaboration.md) |

### Report Service

| Feature | Spec |
|---------|------|
| Connected Planning | [connected-planning.md](07-features/connected-planning.md) |

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
| Event Mesh | [event-mesh.md](07-features/event-mesh.md) |
| Email Service | [email.md](07-features/email.md) |
| Full-Text Search | [search.md](07-features/search.md) |

### Integration Service

| Feature | Spec |
|---------|------|
| API Marketplace | [api-marketplace.md](07-features/api-marketplace.md) |
| Enterprise Data Quality | [enterprise-data-quality.md](07-features/enterprise-data-quality.md) |
| EDI | [edi.md](07-features/edi.md) |

### Cross-Cutting

| Feature | Spec | Service |
|---------|------|---------|
| Multi-Tenancy | [multi-tenancy.md](07-features/multi-tenancy.md) | Tenant Service |

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
| [Glossary](glossary.md) | 100+ terms, abbreviations, references |
