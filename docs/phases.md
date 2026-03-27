# Development Phases

## Phase 1: Foundation (Weeks 1-6)

- [ ] Project scaffolding (workspace, crates, services skeleton)
- [ ] Core libraries (`mserp-core`, `mserp-infrastructure`, `mserp-proto`)
- [ ] Outbox pattern implementation
- [ ] Saga framework (state tracking, compensation)
- [ ] Auth service (authentication, JWT, MFA, SSO)
- [ ] Identity service (users, roles, RBAC, organizations)
- [ ] Tenant service (multi-tenancy, feature flags, RLS, quotas)
- [ ] Config service (dynamic configuration, CORS management)
- [ ] Docker Compose setup (dev + full)
- [ ] CI/CD pipeline (with contract testing)
- [ ] Pact contract testing scaffolding
- [ ] Health check endpoints (`/healthz`, `/readyz`)
- [ ] Graceful shutdown implementation
- [ ] Redis caching infrastructure (cache-aside pattern in `mserp-infrastructure`)
- [ ] Multi-currency types and exchange rate storage
- [ ] i18n framework (locale resolution, translation storage)
- [ ] Structured logging and distributed tracing setup

### Phase 1 Deliverables

| Deliverable | Description |
|-------------|-------------|
| `mserp-core` | Shared error types, config, pagination, i18n, money types |
| `mserp-auth` | JWT validation, auth middleware, permission checks |
| `mserp-infrastructure` | DB (RLS), Redis, RabbitMQ, outbox, saga, tracing, metrics |
| `mserp-proto` | Event type definitions for all services |
| Auth Service | Login, logout, refresh, MFA, SSO bridge, JWT issuance, session management |
| Identity Service | User CRUD, RBAC + ABAC, groups, organizations, permissions |
| Tenant Service | Tenant CRUD, feature flags, RLS, subscription tiers, quotas |
| Config Service | Dynamic config, environment overrides, CORS, localization |
| CI/CD | GitHub Actions + ArgoCD + Pact |
| Dev Environment | `docker-compose.dev.yml` + Makefile |
| Caching Infra | Redis cache-aside pattern in shared library |
| Observability | Prometheus metrics, Jaeger tracing, Loki logging |

---

## Phase 2: Core Business (Weeks 7-14)

- [ ] Commerce service (Sales + Inventory)
  - [ ] Sales domain: Customers, Leads, Quotations, Sales Orders, Deliveries, Invoices
  - [ ] Inventory domain: Products, Warehouses, Stock Management, Serial/Batch Tracking
  - [ ] Advanced Pricing Engine: Price lists, discount rules, promotions, customer-specific pricing
  - [ ] Bulk operations API
- [ ] Finance service (Finance + Procurement)
  - [ ] Finance domain: Chart of Accounts, Journal Entries, General Ledger, AP/AR, Fixed Assets
  - [ ] Procurement domain: Suppliers, Purchase Requisitions, RFQ, Purchase Orders, Goods Receipt
  - [ ] Multi-currency accounting: Exchange rates, triangulation, gains/losses
  - [ ] Budgeting: Budget entry, budget vs. actual, approval workflow
- [ ] Event bus integration (RabbitMQ topology with all bindings)
- [ ] Sales Order Fulfillment saga
- [ ] Purchase Order Receiving saga
- [ ] Caching layer (Redis cache-aside applied to commerce and finance read-heavy endpoints)
- [ ] CORS configuration management
- [ ] Error code taxonomy implementation
- [ ] Service-to-service authentication (mTLS + Service JWTs)
- [ ] Period management (open/close accounting periods)
- [ ] Data import/export framework (CSV, JSON, Excel)

### Phase 2 Deliverables

| Deliverable | Description |
|-------------|-------------|
| Commerce Service | Full sales + inventory CRUD, pricing engine, bulk API |
| Finance Service | Full finance + procurement CRUD, multi-currency, budgeting |
| RabbitMQ Topology | All exchanges, queues, DLQs configured with proper bindings |
| Saga: Order Fulfillment | Reserve stock → Invoice → Notify → Fulfill |
| Saga: PO Receiving | Goods receipt → Stock update → AP |
| Caching | Redis cache-aside for all read-heavy endpoints |
| mTLS + Service JWTs | Secure service-to-service communication |
| Data Import/Export | CSV, JSON, Excel import/export via Integration Service |

---

## Phase 3: Extended Business (Weeks 15-22)

- [ ] HR service (core: Employees, Attendance, Leave, Payroll)
- [ ] Manufacturing service (core: BOM, Work Orders, Routing)
- [ ] Workflow service (BPMN engine, approval chains, process templates, SLAs)
- [ ] Platform service (Notification + File + Audit)
- [ ] CRM / Marketing service (Contacts, Leads, Opportunities, Campaigns)
- [ ] Project Management service (Projects, Tasks, Timeline, Resources)
- [ ] Employee Onboarding saga
- [ ] Leave Approval saga
- [ ] Project Billing saga
- [ ] Email notification templates
- [ ] File storage with document preview

### Phase 3 Deliverables

| Deliverable | Description |
|-------------|-------------|
| HR Service | Employee lifecycle, attendance, leave, payroll, recruitment |
| Manufacturing Service | BOM, work orders, routing, work centers |
| Workflow Service | BPMN engine, approval chains, process templates, SLAs, business rules |
| Platform Service | Email, SMS, push, file storage, audit logs, document preview |
| CRM / Marketing Service | Contact management, lead tracking, deal pipeline, campaigns |
| Project Management Service | Projects, tasks, Gantt timeline, resource allocation, time & expense |
| Saga: Employee Onboarding | Identity → Roles → Access |
| Saga: Leave Approval | Request → Workflow approval → Balance update |
| Saga: Project Billing | Milestone → Invoice → Notify |

---

## Phase 4: Enterprise Features (Weeks 23-30)

- [ ] Report service (dashboards, scheduled reports, BI integration)
- [ ] Data warehouse (DuckDB-based star schema, ETL pipeline)
- [ ] Full HR (performance reviews, training, compensation, benefits)
- [ ] Full Manufacturing (production planning/MRP, quality control, cost accounting)
- [ ] Full CRM (marketing automation, customer analytics, activity tracking)
- [ ] Full Project Management (risk management, project templates, EVM)
- [ ] Advanced workflows (escalations, conditional routing, email approvals)
- [ ] AI-driven insights (anomaly detection, demand forecasting, churn prediction)
- [ ] Full-text search integration (Elasticsearch)
- [ ] Reference data seeding system
- [ ] Integration connectors (Stripe, QuickBooks, shipping carriers, etc.)
- [ ] EDI support (EDIFACT, X12)
- [ ] Supplier collaboration portal

### Phase 4 Deliverables

| Deliverable | Description |
|-------------|-------------|
| Report Service | Dashboards, scheduled reports, data export, AI insights |
| Data Warehouse | DuckDB star schema with ETL pipeline |
| Full HR | Performance reviews, training, compensation, benefits |
| Full Manufacturing | MRP, quality control, cost accounting, shop floor control |
| Full CRM | Marketing automation, customer analytics, campaign tracking |
| Full Project Management | Risk management, templates, EVM, project billing |
| Elasticsearch | Full-text search for commerce, CRM, and document content |
| Integration Connectors | Stripe, QuickBooks, Xero, shipping carriers, Slack, Teams |
| EDI | EDIFACT and X12 support for B2B document exchange |
| Reference Data | Currencies, countries, tax codes, locales, UoMs seeded |

---

## Phase 5: Production Ready (Weeks 31-38)

- [ ] Performance optimization and profiling
- [ ] Security hardening and penetration testing
- [ ] Load testing (target: 10,000 req/s sustained)
- [ ] Disaster recovery procedures and failover testing
- [ ] Documentation (API, architecture, runbooks, operational guides)
- [ ] GDPR compliance (data erasure, data lineage, data subject access)
- [ ] HIPAA readiness (encryption, audit, access logging)
- [ ] PCI DSS compliance (if payment processing enabled)
- [ ] Monitoring dashboards (Grafana: system, business, infrastructure)
- [ ] Alert tuning and runbook validation
- [ ] Multi-region deployment setup and testing
- [ ] Database migration strategy validation (zero-downtime proof)
- [ ] Autoscaling validation and load testing
- [ ] Accessibility compliance (WCAG 2.1 AA for web client)
- [ ] Backup and restore validation

### Phase 5 Deliverables

| Deliverable | Description |
|-------------|-------------|
| Load Test Report | 10,000 req/s validated with autoscaling |
| Security Audit | Penetration test results, threat model, fixes applied |
| Runbooks | Incident response, failover, backup/restore, scaling |
| Grafana Dashboards | System, business, and infrastructure metrics |
| GDPR Compliance | Data erasure API, audit trail, data lineage, data subject access |
| HIPAA Readiness | Encryption, audit logging, access controls documentation |
| Multi-Region | Active-passive failover tested with < 1 hour RTO |
| Migration Runbook | Zero-downtime migration procedures validated |
| Autoscaling | HPA validated for all services under peak load |

---

## Phase Dependencies

```
Phase 1 ──────────▶ Phase 2 ──────────▶ Phase 3 ──────────▶ Phase 4 ──────────▶ Phase 5
(Foundation)        (Core Business)     (Extended)         (Enterprise)        (Production)
  6 weeks             8 weeks             8 weeks             8 weeks             8 weeks
```

- Phase 2 depends on Phase 1 (core libraries, auth, infrastructure, observability).
- Phase 3 depends on Phase 2 (event bus, saga framework proven, pricing engine).
- Phase 4 depends on Phase 3 (workflow engine, platform service, all domains available).
- Phase 5 depends on Phase 4 (all services implemented, features complete).

### Phase Quality Gates

| Gate | Requirement | Phase |
|------|-------------|-------|
| Core tests pass | 80%+ coverage on all core crates | Phase 1 |
| Contract tests | All consumer contracts verified | Phase 2 |
| Saga tests | All saga flows tested with failure scenarios | Phase 3 |
| Load test baseline | 5,000 req/s with < 500ms p99 | Phase 4 |
| Production readiness | 10,000 req/s, security audit passed, runbooks validated | Phase 5 |

---

*See [Non-Functional Requirements](non-functional.md) for quality gates and testing strategy.*
