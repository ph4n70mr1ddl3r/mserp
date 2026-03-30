# Microservices Overview

The platform is composed of 14 microservices organized into 3 categories: Core, Business, and Supporting. Each service owns its database and communicates via asynchronous events (RabbitMQ) and synchronous REST calls through an API Gateway.

## Architectural Rules

| Rule | Detail |
|------|--------|
| Database per Service | No service may query another service's database. All cross-service data access via REST or events. |
| Core Services Publish Only | Auth, Identity, Tenant, Config do NOT have inbox queues. They publish events but do not consume from the event bus. |
| Saga Self-Binding | Every service with an inbox MUST bind to `{domain}.#` (its own domain wildcard) for saga compensation. |
| Config Subscription | All services with inbox queues consume `config.changed` for configuration hot-reload. |
| No Circular Dependencies | Workflow Service is used by all services for approvals — it MUST NOT depend on them. |

## Service Categories

| Category | Port Range | Services |
|----------|-----------|----------|
| Core | 8001–8009 | Auth, Identity, Tenant, Config |
| Business | 8010–8019 | Commerce, Finance, HCM, Manufacturing, Report, Workflow, CRM, Project |
| Supporting | 8020–8029 | Platform, Integration |

> Port ranges include gaps (e.g., 8005–8009, 8018–8019) reserved for future services.

## Inter-Service Dependency Graph

```
                        ┌─────────┐
                        │  Auth   │ (publishes auth.* events)
                        └────┬────┘
                             │ validates credentials via HTTP
                             ▼
                        ┌─────────┐
                        │Identity │ (publishes identity.* events)
                        └────┬────┘
                             │ user/role data via HTTP
                ┌────────────┼────────────┐
                ▼            ▼            ▼
          ┌─────────┐  ┌─────────┐  ┌─────────┐
          │ Tenant  │  │ Config  │  │  All    │
          │         │  │         │  │ Services│
          └────┬────┘  └────┬────┘  └─────────┘
               │            │
               │ tenant events, config events consumed by all inbox services
               ▼            ▼
    ┌──────────────────────────────────────────────────────┐
    │              Business & Supporting Services           │
    │                                                       │
    │  Commerce ←──── Finance  (order→receivable, PO→stock) │
    │      ↕            ↕                                   │
    │  Manufacturing  HR       (work orders, payroll)       │
    │      ↕            ↕                                   │
    │  Report ←────── ALL       (analytics from all events) │
    │                                                       │
    │  CRM ──────→ Commerce    (opportunity→quote→order)    │
    │  Project ──→ Finance     (milestone→invoice→revenue)  │
    │  Workflow ←─ ALL         (approval workflows)         │
    │  Platform ←─ ALL         (notifications, audit, GRC)  │
    │  Integration ← ALL       (MDM, connectors, EDI)      │
    └──────────────────────────────────────────────────────┘
```

### Synchronous HTTP Dependencies (non-event)

| Caller | Callee | Purpose |
|--------|--------|---------|
| Auth Service | Identity Service | User credential validation, role/permission lookup |
| HR Service | Identity Service | User provisioning during onboarding |
| Platform Service | Identity Service | Access revocation from certification campaigns |
| Workflow Service | Platform Service | SoD rule queries for approval enforcement |
| Report Service | All services | Embedded analytics widget data (optional) |

## Service Consolidation Rationale

Multiple functional areas were consolidated into single services to reduce operational complexity, simplify inter-service communication, and enable cohesive transactional boundaries. See [Architecture — Service Consolidation](../01-architecture/overview.md#4-service-consolidation-rationale) for the detailed rationale.

## All Services

| Service | Port | Database | Key Responsibilities | File |
|---------|------|----------|---------------------|------|
| Auth | 8001 | `auth_db` | Authentication, token management, session handling | [auth.md](auth.md) |
| Identity | 8002 | `identity_db` | User management, roles, permissions, organizations | [identity.md](identity.md) |
| Tenant | 8003 | `tenant_db` | Multi-tenancy, tenant provisioning, feature flags | [tenant.md](tenant.md) |
| Config | 8004 | `config_db` | System configuration, settings management | [config.md](config.md) |
| Commerce | 8010 | `commerce_db` | Sales, inventory, pricing, PIM, transportation, ATP/CTP, product configurator, credit management, subscriptions, B2B portal, loyalty, omnichannel, price optimization, order orchestration, global order promising, B2C commerce & storefront | [commerce.md](commerce.md) |
| Finance | 8011 | `finance_db` | Financial accounting, procurement, treasury, EPM, CLM, revenue recognition, strategic sourcing, tax management, commodity management, spend analysis, supplier diversity | [finance.md](finance.md) |
| HCM | 8012 | `hr_db` | Human resources, payroll, workforce management, talent management, goals & objectives, career development, structured onboarding | [hr.md](hr.md) |
| Manufacturing | 8013 | `manufacturing_db` | Production, quality, PLM, EAM, manufacturing intelligence, digital thread, IoT, digital twin, EHS, MRO, product compliance, MES, advanced production scheduling | [manufacturing.md](manufacturing.md) |
| Report | 8014 | `report_db` | Analytics, reporting, dashboards, BI, AI/ML, ESG, narrative reporting, process mining, CPM, self-service ML studio | [report.md](report.md) |
| Workflow | 8015 | `workflow_db` | Business process automation, approvals, BPM engine, SLA management | [workflow.md](workflow.md) |
| CRM | 8016 | `crm_db` | Customer relationship management, marketing automation, field service, CDP, adaptive intelligence, contact center, social selling, A/B testing | [crm.md](crm.md) |
| Project | 8017 | `project_db` | Project planning, execution, resource management, project accounting | [project.md](project.md) |
| Platform | 8020 | `platform_db`, `audit_db` | Notifications, file storage, audit logging, digital assistant, app builder, GRC, RPA, ECM, ITSM, compliance hub, application composer, access certification | [platform.md](platform.md) |
| Integration | 8021 | `integration_db` | External integrations, API management, connectors, MDM, data governance, event mesh, blockchain integration | [integration.md](integration.md) |
