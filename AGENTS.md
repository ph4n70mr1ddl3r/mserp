# AGENTS.md — AI Agent Development Guide

This document instructs AI agents on how to use this specification to build MSERP. Read this file FIRST before any implementation work.

## Specification Hierarchy

Agents MUST follow documents in this order of authority:

1. **SPEC.md** — Authoritative master specification. If any other document conflicts with SPEC.md, SPEC.md wins.
2. **AGENTS.md** — This file. Development workflow, ownership rules, event conventions.
3. **Service specs** (`docs/06-services/*.md`) — Define what each service owns and implements. Each service spec includes: modules, database tables, published/consumed events, and integration points.
4. **Feature specs** (`docs/07-features/*.md`) — Detailed business rules for 22 features whose implementation detail exceeds what service specs contain. For all other features, the service spec is the authoritative source.
5. **Cross-cutting specs** — Architecture, security, data, events, API standards. These define shared patterns all services must follow.
6. **Infrastructure/dev/planning** — Operational concerns, deployment, development workflow.

## Feature Spec Registry (22 specs)

| Feature | Owning Service | Spec File |
|---------|---------------|-----------|
| Account Reconciliation | Finance Service | `docs/07-features/reconciliation.md` |
| Adaptive Intelligence | Platform Service (lifecycle) / Report Service (inference) | `docs/07-features/adaptive-intelligence.md` |
| API Marketplace | Integration Service | `docs/07-features/api-marketplace.md` |
| Collaboration | Platform Service | `docs/07-features/collaboration.md` |
| Compliance Hub | Platform Service | `docs/07-features/compliance-hub.md` |
| Connected Planning | Report Service (analytics) / Finance+HCM+Manufacturing (domain models) | `docs/07-features/connected-planning.md` |
| Content Management | Platform Service | `docs/07-features/content-management.md` |
| Digital Assistant | Platform Service | `docs/07-features/digital-assistant.md` |
| DLP | Platform Service | `docs/07-features/dlp.md` |
| Dynamic Discounting | Finance Service | `docs/07-features/dynamic-discounting.md` |
| EDI | Integration Service | `docs/07-features/edi.md` |
| Email Service | Platform Service | `docs/07-features/email.md` |
| Enterprise Data Quality | Integration Service | `docs/07-features/enterprise-data-quality.md` |
| Event Mesh | Integration Service | `docs/07-features/event-mesh.md` |
| Financial Reporting Studio | Finance Service (templates) / Report Service (rendering) | `docs/07-features/financial-reporting-studio.md` |
| Full-Text Search | Platform Service | `docs/07-features/search.md` |
| IDP | Platform Service (orchestration) / Report Service (ML) | `docs/07-features/idp.md` |
| Intelligent Close | Finance Service | `docs/07-features/intelligent-close.md` |
| IPA (RPA + AI) | Platform Service (runtime) / Report Service (ML) / Integration (connectors) | `docs/07-features/intelligent-process-automation.md` |
| Multi-Tenancy | Tenant Service | `docs/07-features/multi-tenancy.md` |
| Privacy Management | Platform Service | `docs/07-features/privacy.md` |
| Supply Chain Collaboration | Commerce Service (demand) / Manufacturing (capacity) / Integration (network) | `docs/07-features/supply-chain-collaboration.md` |

## Service Ownership Model

Every feature, API endpoint, database table, and event belongs to EXACTLY ONE service. The ownership table is in SPEC.md §4 (Service Domain Ownership).

### Service Boundary Rules

| Domain | Owning Service | Other Services May |
|--------|---------------|-------------------|
| Pre-sale (leads, opportunities, deals, campaigns) | CRM Service | — |
| Order lifecycle (quotes, orders, deliveries, returns) | Commerce Service | Read order data via events |
| Pricing rules & execution | Commerce Service | — |
| Financial accounting (GL, AP, AR, tax, treasury) | Finance Service | — |
| Budgeting & forecasting | Finance Service | — |
| Revenue recognition (ASC 606 / IFRS 15) | Finance Service | Commerce sends contract events |
| Customer profiles & CDP | CRM Service | Commerce provides transactional data |
| Inventory & warehousing | Commerce Service | Manufacturing sends production completions |
| Manufacturing (BOM, work orders, quality, PLM) | Manufacturing Service | Commerce reads ATP data |
| Employee lifecycle & payroll | HCM Service | — |
| Employee self-service portal | HCM Service | Platform provides notification/file services |
| Projects & resources | Project Service | — |
| Reporting platform & dashboards | Report Service | Finance provides report templates |
| Financial report templates (XBRL, regulatory) | Finance Service | Rendered by Report Service |
| Workflow/BPM engine | Workflow Service | All services use workflow |
| Notifications & alerts | Platform Service | All services publish notification requests |
| File storage & ECM | Platform Service | All services use file APIs |
| RPA & IPA | Platform Service | — |
| Integration & connectors | Integration Service | All services register connectors |
| MDM golden records | Integration Service | All services publish master data events |
| Enterprise data quality | Integration Service | — |
| Authentication & tokens | Auth Service | All services validate tokens |
| Identity & RBAC/ABAC | Identity Service | All services check permissions |
| Tenant management | Tenant Service | All services enforce tenant isolation |
| Configuration | Config Service | All services consume config |
| Digital assistant (NLP/chat) | Platform Service | — |
| IDP (document processing) | Platform Service | — |
| ITSM | Platform Service | — |
| GRC & compliance hub | Platform Service (aggregates) | Integration Service owns trade compliance screening data |
| Privacy & DLP | Platform Service | — |
| Knowledge management | Platform Service | — |
| Job scheduler | Platform Service | All services register jobs |
| Collaboration | Platform Service | — |
| Email service | Platform Service (owns all email delivery, templates, suppression) | — |
| Loyalty programs | Commerce Service | CRM provides customer segments |
| Omnichannel | Commerce Service | — |
| Subscriptions | Commerce Service | Finance handles revenue recognition |
| Credit management | Commerce Service | — |
| ESG & sustainability | Report Service | All services emit emissions data |
| Trade compliance | Integration Service | — |
| IoT device registry & certificates | Platform Service | Manufacturing caches device metadata locally |
| IoT telemetry, alerts, edge processing | Manufacturing Service | Commerce reads condition data |
| Digital twin simulation | Manufacturing Service | Platform provides telemetry ingestion infrastructure |
| EHS | Manufacturing Service | — |
| Field service | CRM Service | — |
| Contact center | CRM Service | — |
| Social selling | CRM Service | — |
| Surveys & feedback | CRM Service | — |
| Territory & quota planning | CRM Service | — |
| A/B testing (marketing) | CRM Service | — |
| Blockchain | Integration Service | — |

### Event Namespace Rules

All events MUST follow the `{service-domain}.{entity}.{action}` naming convention:

- Service domains: `auth`, `identity`, `tenant`, `config`, `commerce`, `finance`, `hr`, `manufacturing`, `report`, `workflow`, `crm`, `project`, `platform`, `integration`
- Entities: domain-specific (e.g., `order`, `invoice`, `employee`, `work-order`)
- Actions: `created`, `updated`, `deleted`, `approved`, `completed`, `failed`, etc.

### IoT Boundary Rule

Platform Service owns the **global device registry** (registration, certificates, lifecycle). Manufacturing Service owns **industrial telemetry** (ingestion, alerts, edge processing). The split:

- **Platform publishes** `platform.iot.device.*` events — device registration, certificate provisioning, decommission.
- **Manufacturing consumes** `platform.iot.device.*` events to sync its local `iot_devices` cache table.
- **Manufacturing publishes** `manufacturing.iot.telemetry.*` events — telemetry ingestion, alerts, edge processing results only.
- Manufacturing does **NOT** publish `manufacturing.iot.device.registered` or any device-registry events. Device lifecycle is Platform's responsibility.
- Manufacturing's `iot_devices` table is a **local cache**, not the authoritative registry. Platform's database is the source of truth.

### ML/AI Ownership Split

Report Service owns the **ML/AI platform** (training, serving, feature store, model registry). Platform Service owns **model lifecycle management** (feedback loops, A/B testing, monitoring). Business services own their **domain-specific ML use cases** (define features, consume predictions).

### Critical Integration Points

These are the cross-service integrations that agents must get right. Violating any of these causes architectural inconsistency.

| Integration Point | Description |
|---|---|
| **Auth ↔ Identity (login flow)** | Auth Service uses **HTTP calls** (not events) to query Identity Service for user data during login. Events are too slow for synchronous authentication. Do not refactor this to event-driven. |
| **MDM (master data)** | Integration Service consumes master data events from all services (`{domain}.{entity}.*`) to build golden records. Services MUST publish master data events; they MUST NOT write to Integration's database directly. |
| **SoD (Segregation of Duties)** | Platform Service owns GRC-level SoD rules and risk definitions. Workflow Service queries Platform's API for approval-level SoD checks before routing approvals. Do not duplicate SoD rules in Workflow. |
| **IoT (device registry vs. telemetry)** | Platform owns the device registry (authoritative). Manufacturing caches device data locally by consuming `platform.iot.device.*` events. Manufacturing publishes telemetry only. See "IoT Boundary Rule" above. |

### Common Violations to Avoid

These six inconsistencies have been corrected in the spec. Do not reintroduce them:

1. **Bare domain prefixes in events.** Do NOT use `reconciliation.*` — use `finance.reconciliation.*`. Every event must start with its owning service domain.
2. **Abbreviated domains.** Do NOT use `scc.*` — use `commerce.collaboration.*`. Service domains are never abbreviated.
3. **Nested service domains.** Do NOT use `hr.employee.leave.approved` — use `hr.leave.approved`. The domain is one level; entity nesting follows.
4. **Manufacturing publishing device-registry events.** Manufacturing does NOT publish `manufacturing.iot.device.registered`. Device lifecycle is Platform's domain. Manufacturing publishes `manufacturing.iot.telemetry.*` only.
5. **Event Mesh attributed to "Integration Service (infrastructure)".** Event Mesh is owned by Integration Service — no parenthetical qualifier needed. Do not treat it as a separate infrastructure concern.
6. **Auth querying Identity via events for login.** The login flow requires synchronous HTTP calls from Auth to Identity. Events are async and cannot serve real-time authentication decisions.

## How to Build a Service

1. Read `docs/06-services/{service}.md` for the service's modules, database tables, events, and integration points.
2. Read `docs/06-services/overview.md` for the service catalog and port assignments.
3. Check `docs/07-features/` for relevant feature specs. If a feature spec exists, read it for detailed business rules. Otherwise, the service spec contains the complete feature definition.
4. Read `docs/05-api/standards.md` for API design rules (pagination, idempotency, errors).
5. Read `docs/05-api/endpoints.md` for the service's endpoint listing.
6. Read `docs/03-data/overview.md` for database patterns and standard columns.
7. Read `docs/04-events/overview.md` for event patterns and inbox bindings.
8. Read `docs/04-events/catalog.md` for the service's published and consumed events.
9. Read `docs/02-security/overview.md` and `docs/02-security/authorization.md` for auth requirements.
10. Read `docs/09-development/project-structure.md` for directory layout.
11. Read `docs/09-development/conventions.md` for code style.
12. Read `docs/08-infrastructure/technology.md` for crate versions and dependencies.

## How to Add a Feature

1. Confirm which service owns the feature (see SPEC.md §4 Service Domain Ownership).
2. Check `docs/07-features/` for an existing feature spec. If one exists, read it first. If not, the service spec (`docs/06-services/{service}.md`) contains the complete feature definition.
3. If the feature requires new events, add them to `docs/04-events/catalog.md` using the `{service-domain}.{entity}.{action}` naming convention.
4. If the feature requires new endpoints, add them to `docs/05-api/endpoints.md`.
5. If the feature requires new error codes, add them to `docs/05-api/error-codes.md`.
6. If the feature requires new data models, add them to `docs/03-data/domain-models.md`.
7. If the feature crosses service boundaries, define the integration in both the publisher's and consumer's service specs, and add cross-domain events to `docs/04-events/catalog.md`.

## Event-Driven Integration Pattern

Services communicate asynchronously via RabbitMQ events. The canonical event flow is:

1. Service A performs a business action and writes to its outbox table (in its own database).
2. The outbox poller publishes the event to RabbitMQ exchange `mserp.events`.
3. Service B's inbox queue receives the event based on its binding key.
4. Service B processes the event and may publish its own events in response.

Event bindings are defined in `docs/04-events/overview.md`. Cross-domain event flows are defined in `docs/04-events/catalog.md`.

**IMPORTANT:** The inbox binding table in `docs/04-events/overview.md` is the authoritative source for which events a service can receive. If a cross-domain event in `docs/04-events/catalog.md` references a subscriber, that subscriber MUST have a matching inbox binding.

### Self-Binding for Saga Compensation

All services with inbox queues bind to their own event namespace (`{domain}.#`) for saga compensation patterns. This self-binding is documented in `docs/04-events/overview.md` but may not be repeated in individual service specs. When implementing a saga, always include the self-binding.

## Database Per Service

Each service owns its database exclusively. Other services MUST NOT read or write to another service's database. To access another service's data:
- Use REST APIs for synchronous queries.
- Subscribe to events for asynchronous data replication.

The Report Service is the sole exception: it reads from a DuckDB data warehouse populated by ETL pipelines, not from operational databases.

## Multi-Tenancy

All services MUST enforce tenant isolation via PostgreSQL Row-Level Security (RLS). Every API request includes `X-Tenant-Id` header. Every database query is scoped by `tenant_id`. See `docs/03-data/overview.md` for RLS implementation details.

## Error Handling

All APIs use RFC 7807 problem details. Error codes follow the pattern `{DOMAIN}_{CATEGORY}_{SPECIFIC}`. See `docs/05-api/error-codes.md` for the complete taxonomy. Error codes are additive — never remove or rename an existing code.

## Testing Requirements

- Unit tests for all business logic (`#[cfg(test)]` in same file).
- Integration tests for all API endpoints (`tests/` directory with testcontainers).
- Contract tests (Pact) for all inter-service boundaries.
- Saga compensation tests for all distributed transactions.
- See `docs/10-planning/nfr.md` for the complete testing strategy.

## Key Principles

1. **SPEC.md is law.** If something is unclear, SPEC.md takes precedence.
2. **One owner per feature.** Never implement the same capability in two services.
3. **Events over APIs for cross-service data.** Use async events for data propagation; use sync APIs only for real-time queries.
4. **Backward compatibility always.** APIs and events are versioned. Breaking changes require a migration plan.
5. **Security by default.** Every endpoint checks auth, every query is tenant-scoped, every sensitive field is encrypted.
6. **Observe everything.** All services emit Prometheus metrics, structured logs, and OpenTelemetry traces.
7. **Event namespace discipline.** Always use `{service-domain}.{entity}.{action}`. Never use bare or abbreviated prefixes.
8. **Service specs are the primary reference.** Feature specs supplement service specs for complex features. If a feature has no dedicated spec, the service spec is complete and authoritative.
