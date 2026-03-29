# AGENTS.md — AI Agent Development Guide

This document instructs AI agents on how to use this specification to build MSERP. Read this file FIRST before any implementation work.

## Specification Hierarchy

Agents MUST follow documents in this order of authority:

1. **SPEC.md** — Authoritative master specification. If any other document conflicts with SPEC.md, SPEC.md wins.
2. **Service specs** (`docs/06-services/*.md`) — Define what each service owns and implements.
3. **Feature specs** (`docs/07-features/*.md`) — Detailed business rules for features whose implementation detail exceeds what service specs contain. Only 22 feature specs remain (see below). For all other features, the service spec is the authoritative source.
4. **Cross-cutting specs** — Architecture, security, data, events, API standards. These define shared patterns all services must follow.
5. **Infrastructure/dev/planning** — Operational concerns, deployment, development workflow.

### Remaining Feature Specs

The following 22 feature specs contain substantial implementation detail not found in service specs: adaptive-intelligence, api-marketplace, collaboration, compliance-hub, connected-planning, content-management, digital-assistant, dlp, dynamic-discounting, edi, email, enterprise-data-quality, event-mesh, financial-reporting-studio, idp, intelligent-close, intelligent-process-automation, multi-tenancy, privacy, reconciliation, search, supply-chain-collaboration.

## Service Ownership Model

Every feature, API endpoint, database table, and event belongs to EXACTLY ONE service. The ownership table is in SPEC.md §Service Domain Ownership.

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
| Projects & resources | Project Service | — |
| Reporting platform & dashboards | Report Service | Finance provides report templates |
| Financial report templates (XBRL, regulatory) | Finance Service | Rendered by Report Service |
| Workflow/BPM engine | Workflow Service | All services use workflow |
| Notifications & alerts | Platform Service | All services publish notification requests |
| File storage & ECM | Platform Service | All services use file APIs |
| RPA & IPA | Platform Service | — |
| Integration & connectors | Integration Service | All services register connectors |
| MDM golden records | Integration Service | All services publish master data events |
| Enterprise data quality | Integration Service (owns all data profiling, cleansing, matching, enrichment) | — |
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
| IoT device registry | Platform Service (owns device registry and certificates); Manufacturing Service (owns industrial IoT telemetry, alerts, edge processing) | Commerce reads condition data |
| Digital twin simulation | Manufacturing Service (owns asset twins and simulations); Platform Service (provides telemetry ingestion infrastructure) | — |
| EHS | Manufacturing Service | — |
| Field service | CRM Service | — |
| Contact center | CRM Service | — |
| Social selling | CRM Service | — |
| Surveys & feedback | CRM Service | — |
| Territory & quota planning | CRM Service | — |
| A/B testing | CRM Service | — |
| Blockchain | Integration Service | — |
| Employee self-service portal | HCM Service | Platform provides notification/file services |

## How to Build a Service

1. Read `docs/06-services/{service}.md` for the service's modules and responsibilities.
2. Read `docs/06-services/overview.md` for the service catalog and port assignments.
3. Check `docs/07-features/` for relevant feature specs. If a feature spec exists, read it for detailed business rules. Otherwise, the service spec contains the complete feature definition.
4. Read `docs/05-api/standards.md` for API design rules (pagination, idempotency, errors).
5. Read `docs/05-api/endpoints.md` for the service's endpoint listing.
6. Read `docs/03-data/overview.md` for database patterns and standard columns.
7. Read `docs/04-events/overview.md` for event patterns.
8. Read `docs/04-events/catalog.md` for the service's published and consumed events.
9. Read `docs/02-security/overview.md` and `docs/02-security/authorization.md` for auth requirements.
10. Read `docs/09-development/project-structure.md` for directory layout.
11. Read `docs/09-development/conventions.md` for code style.

## How to Add a Feature

1. Confirm which service owns the feature (see SPEC.md §Service Domain Ownership).
2. Check `docs/07-features/` for an existing feature spec. If one exists, read it first. If not, the service spec (`docs/06-services/{service}.md`) contains the complete feature definition.
3. If the feature requires new events, add them to `docs/04-events/catalog.md`.
4. If the feature requires new endpoints, add them to `docs/05-api/endpoints.md`.
5. If the feature requires new error codes, add them to `docs/05-api/error-codes.md`.
6. If the feature requires new data models, add them to the service's database in `docs/03-data/`.

## Event-Driven Integration Pattern

Services communicate asynchronously via RabbitMQ events. The canonical event flow is:

1. Service A performs a business action and writes to its outbox table (in its own database).
2. The outbox poller publishes the event to RabbitMQ exchange `mserp.events`.
3. Service B's inbox queue receives the event based on its binding key.
4. Service B processes the event and may publish its own events in response.

Event bindings are defined in `docs/04-events/overview.md`. Cross-domain event flows are defined in `docs/04-events/catalog.md`.

**IMPORTANT:** The inbox binding table in `docs/04-events/overview.md` is the authoritative source for which events a service can receive. If a cross-domain event in `docs/04-events/catalog.md` references a subscriber, that subscriber MUST have a matching inbox binding.

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

- Unit tests for all business logic.
- Integration tests for all API endpoints.
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
