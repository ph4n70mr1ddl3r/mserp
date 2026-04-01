# MSERP — Master Specification

> **This document is the authoritative single source of truth for MSERP.** All other documents are subordinate. If any conflict arises, this document takes precedence.

---

## 1. Executive Summary

MSERP is an enterprise-grade, microservices-based ERP system built in Rust, designed for scalability, reliability, and performance. The system provides comprehensive ERP coverage comparable to Oracle Fusion Cloud ERP, spanning Financial Management, Supply Chain Management, Human Capital Management, Customer Experience, Project Management, Manufacturing, Enterprise Performance Management, and Governance, Risk & Compliance — all built on a modern, cloud-native microservices architecture.

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
| Test coverage | >= 80% per crate |

---

## 2. Core Technologies

| Category | Technology | Version |
|----------|-----------|---------|
| Language | Rust | 1.83+ / Edition 2021 |
| Runtime | Tokio | 1.x |
| Web Framework | Axum | 0.7+ |
| ORM | SeaORM | 0.12+ |
| Database | PostgreSQL | 16 |
| Connection Pooler | PgBouncer | 1.22+ |
| Message Broker | RabbitMQ | 3.12 |
| Cache | Redis | 7.x |
| Search | Elasticsearch | 8.x |
| Object Storage | MinIO (S3-compatible) | RELEASE.2024-01+ |
| Data Warehouse | DuckDB | 1.0 |
| Analytics | Apache Arrow | 50.0 |
| ML/AI Runtime | ONNX Runtime (ort) | 2.0 |
| ML Training | Candle | 0.4 |
| Vector Search | qdrant (optional) | 1.8+ |
| Orchestration | Kubernetes | 1.28+ |
| API Gateway | Traefik | 3.x |
| CI/CD | GitHub Actions + ArgoCD | Latest |
| Observability | Prometheus + Grafana + Jaeger + Loki | Latest |
| Secrets | HashiCorp Vault | 1.15+ |
| IaC | Terraform | 1.7+ |
| Frontend | React 18.x + TypeScript 5.x | Node.js 20 LTS |
| Mobile | React Native | 0.73+ |

### 2.1 Crate Dependency Rules

- All workspace crates share dependency versions via Cargo workspace inheritance.
- Individual crates MUST NOT pin different versions of workspace-managed dependencies.
- New dependency additions require workspace-level review.

### 2.2 Container Rules

- Multi-stage Docker builds mandatory (builder: `rust:1.83-slim-bookworm`, runtime: `debian:bookworm-slim` or distroless).
- Images < 50MB compressed, non-root user (1000:1000), read-only root FS.
- Base images signed with Cosign; SBOM generated per image (Syft).
- CI blocks merge on Critical/High CVEs.

---

## 3. Architecture

### 3.1 Topology

```
Client Layer → Gateway Layer (Traefik) → Microservices Layer (14 services) → Infrastructure Layer
```

- **Client Layer**: Web (React), Mobile (React Native), B2B Portal, API Consumers, Admin Console, Digital Assistant
- **Gateway Layer**: Traefik (mTLS termination, rate limiting, circuit breaker, routing, CORS enforcement)
- **Microservices Layer**: 14 services in 3 tiers (see §4)
- **Infrastructure Layer**: PostgreSQL, Redis, RabbitMQ, Elasticsearch, MinIO, DuckDB, observability stack

### 3.2 Communication Patterns

| Pattern | Protocol | Use Case |
|---------|----------|----------|
| Synchronous | REST (HTTP/JSON) | Request/response queries |
| Asynchronous | RabbitMQ (AMQP) | Event propagation, saga choreography |
| Streaming | WebSocket (via Platform Service) | Real-time updates, notifications, collaboration |
| Bulk | REST (multipart/batch, max 100 items) | Data import/export, batch operations |
| File-based | REST + MinIO | Datasets > 100 items (CSV/Excel via Integration Service) |
| Orchestrated | HTTP + events | Cross-service calls when target lacks inbox queue |
| Digital Assistant | NLP intent → service API dispatch | Conversational interface |

### 3.3 Database-per-Service

Each service owns its database exclusively. **No service may query another service's database directly** — all cross-service data access is via REST APIs or event subscriptions. Total: **15 databases** (4 core + 8 business + 3 supporting).

### 3.4 Repository Structure

Monorepo with Cargo workspace. Shared crates: `mserp-core`, `mserp-auth`, `mserp-infrastructure`, `mserp-proto`, `mserp-migrations`.

---

## 4. Service Catalog

### 4.1 Service Registry

| # | Service | Port | Database | Tier | Modules |
|---|---------|------|----------|------|---------|
| 1 | Auth Service | 8001 | `auth_db` | Core | OAuth2/OIDC Provider, MFA (4 factors), SSO/SAML 2.0 Bridge, JWT Lifecycle, Brute-Force Detection, Step-Up Authentication, Session Management |
| 2 | Identity Service | 8002 | `identity_db` | Core | User Lifecycle, RBAC, ABAC, User Groups/Teams, Organizational Hierarchy, API Key Management, Delegated Administration, User Preferences |
| 3 | Tenant Service | 8003 | `tenant_db` | Core | Tenant Lifecycle, Subscription Tiers, Feature Flags, Quota Enforcement, Data Residency, Tenant Branding |
| 4 | Config Service | 8004 | `config_db` | Core | Hierarchical Configuration, Config Propagation, CORS Management, Localization (14 locales), Maintenance Windows, Validation Rules |
| 5 | Commerce Service | 8010 | `commerce_db` | Business | Sales, Inventory, PIM, Transportation, Advanced Pricing Engine, ATP/CTP, Product Configurator, Credit Management, Subscription Management, Intercompany Drop Ship, Connected Logistics, Warranty Management, B2B Portal, Loyalty Management, Omnichannel, AI Price Optimization, Advanced Inventory Optimization, Demand Planning, Supply Allocation, Supply Chain Collaboration, Order Orchestration, Global Order Promising (GOP), B2C Commerce & Storefront, Sales & Operations Planning (S&OP), Channel Revenue Management, Advanced Warehouse Management (WMS), Advanced Transportation Management (TMS), CPQ, Retail POS, Property Management |
| 6 | Finance Service | 8011 | `finance_db` | Business | GL, AP, AR, Treasury, Revenue Recognition (ASC 606/IFRS 15), Budgeting, Procurement, Tax, Multi-Book Accounting, Intercompany Matching, Corporate Treasury, Account Reconciliation, Profitability Analysis, Lease Accounting (ASC 842/IFRS 16), Grant Management, Joint Venture Accounting, Intelligent Close, Advanced Collections, Dynamic Discounting, Expense Management, EPM, Financial Consolidation, Intercompany Netting, Cash Flow Statement, Strategic Sourcing, Supplier Risk Management, CLM, Advanced Tax Management, Commodity Management, Advanced Spend Analysis, Supplier Diversity, Financial Reporting Studio, Fixed Asset Management, Tax Reporting & Provision, Touchless Invoicing, Shared Services Center, IT Financial Management, Subledger Accounting Engine, Advanced Tax Filing |
| 7 | HCM Service | 8012 | `hr_db` | Business | Employee Lifecycle, Payroll Processing, Recruitment, Performance Management, Talent Management, Benefits Administration, Time & Attendance, Leave Management, Employee Self-Service Portal, HR Help Desk, LXP, Position Control, Workforce Planning, Compensation Management, Succession Planning, Training & Development, Employee Documents, Goals & Objectives, Career Development & Paths, Structured Onboarding Journey, Opportunity Marketplace, Dynamic Skills, Workforce Health & Safety, Workforce Scheduling, Employee Experience Platform |
| 8 | Manufacturing Service | 8013 | `manufacturing_db` | Business | BOM, Work Orders, PLM, EAM, Quality Management, Digital Twin, EHS, Process Manufacturing, Mixed-Mode Manufacturing, APS, Manufacturing Intelligence, Lean Manufacturing, Connected Worker, Digital Thread, IoT Telemetry & Alerts, MRO, Product Compliance, Manufacturing Execution System (MES), Advanced Production Scheduling & Simulation, Contract Manufacturing, Innovation Management |
| 9 | Report Service | 8014 | `report_db` | Business | Analytics Platform, Dashboards, BI, ESG Reporting, Data Warehouse (DuckDB), ML Platform, Semantic Layer, Report Bursting, Narrative Reporting, Process Mining, CPM, Embedded Analytics, Predictive Analytics, Self-Service BI, Augmented Analytics, Connected Planning, Carbon Accounting, Self-Service ML Studio |
| 10 | Workflow Service | 8015 | `workflow_db` | Business | BPMN 2.0 Engine, Approval Chains, Escalation Management, SLA Management, Business Rules Engine, Process Monitoring, Email Approvals, SoD Enforcement |
| 11 | CRM Service | 8016 | `crm_db` | Business | Pre-sale (leads, opportunities, campaigns), Field Service, Surveys & Feedback, Sales Territory & Quota, CDP, Enterprise Contact Center, Social Selling, CX Digital & A/B Testing, Sales Forecasting, Partner Management, Sales Performance Management, Depot Repair, Marketing Automation |
| 12 | Project Service | 8017 | `project_db` | Business | WBS, Gantt/CPM Scheduling, Resource Allocation, Time & Expense, Project Billing (T&M/FF/CPFF), Budget & Costing, Risk Management, EVM, Program Management, Portfolio Analysis, Construction & Engineering |
| 13 | Platform Service | 8020 | `platform_db` + `audit_db` | Supporting | Notifications, Files/ECM, GRC, Digital Assistant, IDP, RPA/IPA, ITSM, Job Scheduler, Multi-Channel Assistant, Knowledge Management, Regulatory Content, Full-Text Search, Content Management, Collaboration, Email Service, Privacy Management, DLP, Compliance Hub, Application Composer & Runtime Extensions, Access Certification, Guided Learning, Policy Automation |
| 14 | Integration Service | 8021 | `integration_db` | Supporting | Connectors, EDI (X12/EDIFACT), MDM, Data Quality, Trade Compliance, Blockchain Provenance, API Marketplace, Event Mesh, Flow Designer, MFT, Global Trade Management |

### 4.2 Service Consolidation Rationale

| Service | Consolidates | Rationale |
|---------|-------------|-----------|
| Commerce | Sales + Inventory + PIM + Transportation | Tight coupling between ordering, stock, pricing, and fulfillment |
| Finance | Finance + Procurement + Treasury + Expenses + CLM + EPM | Procurement generates AP; treasury manages cash; expenses flow to GL |
| Platform | Notification + File + Audit + Assistant + App Builder + GRC + many more | Cross-cutting infrastructure used by all services |
| Workflow | BPMN engine only | Separated because all services use it for approvals; must not create circular dependencies |

---

## 5. Event Architecture

### 5.1 Event Naming

All events follow `{domain}.{entity_or_subentity}.{action}` naming convention with **3 or 4 dot-separated segments**. Use **underscores only** for multi-word identifiers — hyphens are not permitted.

```
{domain}.{entity}.{action}              (3 segments — standard)
{domain}.{entity}.{subentity}.{action}  (4 segments — nested entity)

  │        │        └─ lowercase, underscore-separated: created, updated, deleted, submitted, approved, rejected, completed, failed
  │        └────────── lowercase, underscore-separated: order, purchase_order, cash_application, work_order, digital_twin
  └─────────────────── service domain name (exactly one of the 14 below)
```

4-segment form is permitted only for genuinely nested entities (e.g., `manufacturing.digital_twin.state.updated` where `state` is a sub-entity of `digital_twin`). Do not use 4 segments merely to add detail that belongs in the payload.

Domains correspond to service names: `auth`, `identity`, `tenant`, `config`, `commerce`, `finance`, `hr`, `manufacturing`, `report`, `workflow`, `platform`, `integration`, `crm`, `project`.

**Examples**: `commerce.order.created`, `finance.purchase_order.approved`, `manufacturing.digital_twin.state.updated`, `platform.grc.sod.detected`.

### 5.2 RabbitMQ Topology

Single topic exchange `mserp.events`. Each service with an inbox queue has a dedicated `mserp.{service}.events` queue bound with routing keys. **Core services (Auth, Identity, Tenant, Config) do NOT have inbox queues** — they publish events but do not consume from the event bus.

### 5.3 Service Inbox Bindings

| Service Inbox | Self-Binding | Cross-Domain Bindings |
|--------------|-------------|----------------------|
| `commerce.inbox` | `commerce.#` | `finance.purchase_order.#`, `finance.invoice.#`, `finance.sourcing.#`, `finance.cash_application.#`, `manufacturing.work_order.#`, `manufacturing.digital_twin.#`, `crm.opportunity.won`, `crm.cdp.#`, `integration.trade.party_screened`, `config.changed` |
| `finance.inbox` | `finance.#` | `commerce.order.#`, `commerce.stock.#`, `commerce.subscription.#`, `commerce.b2b.#`, `commerce.warranty.#`, `hr.payroll.#`, `manufacturing.work_order.#`, `project.invoice.#`, `project.milestone.#`, `platform.idp.#`, `config.changed` |
| `hr.inbox` | `hr.#` | `workflow.step.#`, `config.changed` |
| `manufacturing.inbox` | `manufacturing.#` | `commerce.stock.#`, `commerce.order.#`, `commerce.warranty.#`, `platform.iot.device.#`, `config.changed` |
| `report.inbox` | `report.#` | `commerce.#`, `finance.#`, `hr.#`, `manufacturing.#`, `crm.#`, `project.#`, `workflow.#`, `platform.audit.#`, `platform.rpa.#`, `platform.grc.#`, `tenant.feature.#`, `integration.trade_compliance.#`, `config.changed` |
| `workflow.inbox` | `workflow.#` | `hr.leave.#`, `commerce.order.#`, `commerce.credit.#`, `commerce.tms.#`, `commerce.wms.#`, `finance.purchase_order.#`, `finance.expense.#`, `finance.intelligent_close.#`, `finance.supplier_risk.#`, `finance.sla.#`, `manufacturing.eco.#`, `report.process.#`, `platform.rpa.#`, `platform.grc.#`, `integration.trade_compliance.#`, `crm.campaign.#`, `config.changed` |
| `platform.inbox` | `platform.#` | `auth.login.#`, `hr.#`, `commerce.order.#`, `commerce.credit.#`, `commerce.shipment.#`, `commerce.logistics.#`, `commerce.tms.delivery_confirmed`, `finance.lease.#`, `finance.supplier_risk.#`, `finance.intelligent_close.#`, `integration.trade_compliance.#`, `crm.cdp.#`, `report.process.#`, `manufacturing.intelligence.#`, `identity.user.#`, `tenant.feature.#`, `tenant.created`, `tenant.decommissioned`, `config.changed` |
| `integration.inbox` | `integration.#` | `commerce.customer.#`, `commerce.product.#`, `finance.supplier.#`, `hr.employee.#`, `identity.user.#`, `config.changed` |
| `crm.inbox` | `crm.#` | `commerce.customer.#`, `commerce.order.#`, `config.changed` |
| `project.inbox` | `project.#` | `hr.employee.#`, `commerce.order.#`, `config.changed` |

### 5.4 Event Envelope

```json
{
  "event_id": "uuid",
  "event_type": "commerce.order.created",
  "event_version": "1.0",
  "timestamp": "2026-03-27T10:30:00Z",
  "tenant_id": "uuid (null for system events)",
  "correlation_id": "uuid",
  "causation_id": "uuid",
  "aggregate": { "type": "SalesOrder", "id": "uuid" },
  "payload": { "..." : "..." },
  "metadata": { "user_id": "uuid", "source": "service-name", "version": 1 }
}
```

### 5.5 Event Versioning

- Semantic `MAJOR.MINOR` (no patch). Minor = additive. Major = breaking.
- Dual-publish period for major versions: 90 days.
- Version rejection by consumers (not broker). Unsupported versions → DLQ.
- Consumers declare supported versions in TOML config.

### 5.6 Transactional Outbox

All services MUST use the transactional outbox pattern. Write business data + event to outbox table in same transaction. Background poller publishes to RabbitMQ. This ensures atomicity without distributed transactions.

### 5.7 Dead Letter Queues

Max 5 retries, exponential backoff (1s, 2s, 4s, 8s, 16s), 30-day retention, replay via admin API.

### 5.8 Event Ordering

- **Per aggregate within a service**: Guaranteed (outbox sequential publish).
- **Across services**: NO guaranteed ordering. Consumers MUST be idempotent.
- **Effectively-once**: At-least-once delivery + idempotent consumers with 24-hour dedup TTL on `event_id`.

### 5.9 Saga Pattern

Default: **choreography-based saga**. Each step emits success/failure events. Exception: orchestrated sagas when target service lacks inbox queue (e.g., Identity Service → HTTP calls).

| Rule | Detail |
|------|--------|
| Compensating action | Every saga step MUST have one |
| Timeout | Default 30 minutes; expired sagas trigger compensation |
| State tracking | In originating service's database |
| Metadata | All saga events include `saga_id` and `saga_step` |
| Compensation failure | DLQ routing, Critical alert, manual intervention required |
| State retention | 30 days after terminal state for debugging |

**Defined Sagas**: Sales Order Fulfillment, Purchase Order Receiving, Employee Onboarding, Leave Approval, Purchase Requisition, Project Billing, Expense Report, Contract Approval, Engineering Change Order, Order Orchestration, MES Production, Global Trade Screening, Warehouse Wave Execution.

> **Full event catalog:** [docs/04-events/catalog.md](docs/04-events/catalog.md) (290+ domain events)
> **Full saga definitions:** [docs/04-events/sagas.md](docs/04-events/sagas.md) (13 sagas)

---

## 6. Service Domain Ownership

Every feature, entity, API endpoint, database table, and event belongs to EXACTLY ONE service.

| Domain | Owning Service |
|--------|---------------|
| Pre-sale (leads, opportunities, deals, campaigns) | CRM Service |
| Order lifecycle (quotes, orders, deliveries, returns) | Commerce Service |
| Pricing rules & execution | Commerce Service |
| Financial accounting (GL, AP, AR, tax, treasury) | Finance Service |
| Budgeting, forecasting, multi-book accounting | Finance Service |
| Revenue recognition (ASC 606 / IFRS 15) | Finance Service |
| Customer profiles & CDP | CRM Service |
| Inventory & warehousing | Commerce Service |
| Manufacturing (BOM, work orders, quality, PLM) | Manufacturing Service |
| IoT device registry & certificates | Platform Service |
| IoT telemetry, alerts, edge processing | Manufacturing Service |
| Employee lifecycle, payroll, HR help desk, LXP | HCM Service |
| Projects & resources | Project Service |
| Reporting platform, semantic layer, dashboards | Report Service |
| Financial report templates (XBRL, regulatory) | Finance Service |
| Workflow/BPM engine | Workflow Service |
| SoD rule definitions (GRC-level) | Platform Service |
| Notifications & alerts | Platform Service |
| File storage & ECM | Platform Service |
| RPA & IPA | Platform Service |
| Integration, connectors, event mesh, MFT | Integration Service |
| MDM golden records | Integration Service |
| Enterprise data quality | Integration Service |
| Authentication & tokens | Auth Service |
| Identity & RBAC/ABAC | Identity Service |
| Tenant management | Tenant Service |
| Configuration | Config Service |
| Digital assistant (NLP/chat, multi-channel) | Platform Service |
| IDP (document processing) | Platform Service |
| ITSM | Platform Service |
| GRC & compliance hub | Platform Service |
| Privacy & DLP | Platform Service |
| Knowledge management | Platform Service |
| Job scheduler | Platform Service |
| Collaboration | Platform Service |
| Email service | Platform Service |
| Loyalty programs | Commerce Service |
| Omnichannel | Commerce Service |
| Subscriptions | Commerce Service |
| ESG & sustainability | Report Service |
| Trade compliance | Integration Service |
| Process manufacturing | Manufacturing Service |
| Sales forecasting (pipeline-based) | CRM Service |
| Partner relationship management | CRM Service |
| Connected planning | Report Service |
| Supply chain collaboration | Commerce Service |
| Order orchestration & fulfillment rules | Commerce Service |
| Global Order Promising (GOP) | Commerce Service |
| B2C commerce & storefront | Commerce Service |
| Goals & career development | HCM Service |
| Onboarding journey builder | HCM Service |
| MES (shop floor execution, labor tracking) | Manufacturing Service |
| Production scheduling & simulation | Manufacturing Service |
| Application composer & runtime extensions | Platform Service |
| Access certification campaigns | Platform Service |
| Advanced demand management (ML-powered demand sensing, statistical forecasting) | Commerce Service |
| Sales & operations planning (S&OP, integrated business planning) | Commerce Service |
| Channel revenue management (partner pricing, rebates, incentives) | Commerce Service |
| Touchless invoicing (AI-powered AP automation) | Finance Service |
| Tax reporting & provision (corporate tax, CbCR) | Finance Service |
| Shared services center (centralized AP/AR processing) | Finance Service |
| IT financial management (IT cost transparency, chargebacks) | Finance Service |
| Opportunity marketplace (internal mobility, gig work) | HCM Service |
| Dynamic skills (AI skills ontology, matching, gap analysis) | HCM Service |
| Workforce health & safety (employee safety, incident management) | HCM Service |
| Workforce scheduling (shift scheduling optimization) | HCM Service |
| Employee experience (recognition, touchpoints, connections) | HCM Service |
| Contract manufacturing (outsourced production management) | Manufacturing Service |
| Innovation management (idea management, PLM innovation) | Manufacturing Service |
| Sales performance management (quotas, commissions, incentives) | CRM Service |
| Depot repair (service logistics, repair management) | CRM Service |
| Guided learning (in-app contextual guidance, walkthroughs) | Platform Service |
| Advanced warehouse management (directed putaway, wave planning, voice picking, cartonization) | Commerce Service |
| Advanced transportation management (freight rating, carrier selection, load consolidation, freight audit) | Commerce Service |
| Global trade management (denied party screening, export licenses, customs documentation, duty management) | Integration Service |
| Marketing automation (multi-step nurture campaigns, behavioral lead scoring, email marketing, campaign attribution) | CRM Service |
| Subledger accounting engine (centralized journal generation rules, configurable accounting transformations) | Finance Service |
| Configure, Price, Quote (CPQ) | Commerce Service |
| Retail Point-of-Sale | Commerce Service |
| Property Management | Commerce Service |
| Advanced Tax Filing | Finance Service |
| Policy Automation | Platform Service |
| Construction & Engineering | Project Service |

---

## 7. Cross-Service Boundaries

### 7.1 CRM → Commerce Handoff

```
CRM: Lead → Qualified Lead → Opportunity
                                    ↓ (opportunity converted to quote)
Commerce: Quote → Order → Delivery → Invoice → Payment (via Finance)
```

CDP (owned by CRM) maintains unified customer profiles. Commerce publishes transactional events (orders, deliveries, returns) that CRM's CDP consumes to enrich profiles.

### 7.2 Commerce → Finance Handoff

| Event | Publisher | Consumer Action |
|-------|-----------|-------------------|
| `commerce.order.submitted` | Commerce | Finance creates receivable |
| `commerce.delivery.completed` | Commerce | Finance recognizes revenue |
| `commerce.subscription.billing.completed` | Commerce | Finance creates performance obligation |
| `commerce.order.returned` | Commerce | Finance creates credit memo |
| `commerce.warranty.claim.submitted` | Commerce | Finance accrues warranty liability |

### 7.3 Analytics Ownership

Report Service owns the analytics platform (dashboard framework, data warehouse, BI tools, semantic layer). Other services do NOT build independent analytics infrastructure. Instead:

- Each service publishes domain events that Report Service consumes for its data warehouse.
- Embedded analytics in any service's UI are served by Report Service's widget framework.

**Exception:** Financial Reporting Studio — Finance owns XBRL templates and regulatory report definitions; Report Service owns the rendering engine.

### 7.4 Employee Self-Service Portal

Owned by HCM Service. Platform Service provides notification and file storage infrastructure.

### 7.5 Project Time & Expense

- **Project Service**: Project timesheets → project billing → revenue recognition (Finance)
- **HCM Service**: Attendance → payroll processing (HCM internal)
- **Finance Service**: Expense reports → reimbursement → payment

### 7.6 IoT Boundary

Platform Service owns the authoritative device registry and publishes `platform.iot.device.*` events. Manufacturing's `iot_devices` table is a **local cache** synced by consuming `platform.iot.device.*` events. Manufacturing publishes `manufacturing.iot.telemetry.*` events (telemetry only), NOT device registration events.

### 7.7 SoD Rule Boundary

Platform Service owns GRC-level SoD rule definitions (`grc_sod_rules` table in `platform_db`). Workflow Service consumes `platform.grc.sod.detected` events and queries Platform API for SoD rules for approval-level enforcement. Workflow does NOT have its own `sod_rules` table.

### 7.8 MDM Golden Records

Integration Service consumes master data events from all services (`commerce.customer.#`, `commerce.product.#`, `finance.supplier.#`, `hr.employee.#`, `identity.user.#`) for MDM golden record management.

---

## 8. Multi-Tenancy

All services enforce tenant isolation via PostgreSQL Row-Level Security (RLS). Every request includes `X-Tenant-Id` header propagated from the JWT.

### 8.1 Isolation Matrix

| Layer | Isolation Strategy |
|-------|-------------------|
| PostgreSQL | Row-Level Security (`tenant_id` column on every table, `app.current_tenant` session variable) |
| Redis | Key prefix: `mserp:{tenant_id}:{service}:{entity}:{id}` |
| Elasticsearch | Per-tenant indices |
| MinIO | Path prefix: `{tenant_id}/` |
| RabbitMQ | `tenant_id` in event envelope (nullable for system events) |
| API | `X-Tenant-Id` header from JWT on every request |

### 8.2 Data Residency

| Region | Compliance Frameworks |
|--------|----------------------|
| US-East | SOC 2, HIPAA, PCI DSS |
| EU-West | GDPR, EU Data Act, SOC 2 |
| APAC | APAC data residency |
| ME | Regional compliance |

Region-pinned tenants route to designated PostgreSQL cluster.

### 8.3 Feature Flags

Three types: Boolean (on/off), Percentage Rollout (0-100%), User Segment (target specific user segments/roles). Evaluation flow: check user segment overrides → evaluate flag → return value.

### 8.4 Quota Enforcement

Five resources tracked: API calls/min, storage, users, custom objects, data warehouse storage. Response headers: `X-Quota-Resource`, `X-Quota-Usage`, `X-Quota-Limit`, `X-Quota-Remaining`.

---

## 9. Data Architecture Rules

### 9.1 Standard Columns (ALL tables)

```sql
id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
tenant_id       UUID NOT NULL,  -- nullable for event/outbox tables
created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
created_by      UUID,
updated_by      UUID,
version         INTEGER NOT NULL DEFAULT 1,
is_deleted      BOOLEAN NOT NULL DEFAULT FALSE
```

### 9.2 Standard Indexes

```sql
CREATE INDEX idx_{table}_tenant_id ON {table} (tenant_id);
CREATE INDEX idx_{table}_created_at ON {table} (created_at);
CREATE INDEX idx_{table}_is_deleted ON {table} (is_deleted);
```

### 9.3 RLS Implementation

```sql
ALTER TABLE {table} ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON {table}
    USING (tenant_id = current_setting('app.current_tenant')::UUID);
```

`app.current_tenant` set per transaction from JWT. Apply `tenant_id` via RLS; never rely on application-level filtering alone.

### 9.4 Migration Rules

| Rule | Description |
|------|-------------|
| Forward-Only | No `down()` migrations in production |
| Backward-Compatible | Old code MUST work with new schema for one deployment cycle |
| Zero-Downtime | Migrations MUST NOT lock tables > 100ms |
| Idempotent | Safe to re-run |
| Ordered | Timestamp prefix: `{YYYYMMDDHHMMSS}_{description}.rs` |

### 9.5 Schema Change Patterns

| Change Type | Safe Pattern |
|-------------|-------------|
| Add column | `ALTER TABLE ADD COLUMN` with default value |
| Remove column | 3-phase: stop reading → deploy → drop column |
| Rename column | Add new + copy data + drop old (never `ALTER TABLE RENAME`) |
| Add index | `CREATE INDEX CONCURRENTLY` |
| Remove index | `DROP INDEX CONCURRENTLY` |
| Modify type | Add new column + migrate data + swap + drop old |

### 9.6 Soft Delete

- Sets `is_deleted = TRUE`. Queries default to `WHERE is_deleted = FALSE` via SeaORM global filter.
- Soft deletes ARE published as events (`{domain}.{entity}.deleted`).
- Hard deletes: admin-only, GDPR data erasure, NOT published, audited separately.
- Partial unique indexes for soft-deleted natural keys: `WHERE is_deleted = FALSE`.

### 9.7 Audit Trail

Supplementary write-only log in `audit_db` (time-series optimized, monthly partitions, 2-year archive). NOT event sourcing — business state lives in domain tables.

### 9.8 Data Retention

| Category | Retention | Hard Delete | Notes |
|----------|-----------|-------------|-------|
| Business data (orders, invoices) | Indefinite | Never (compliance) | Soft-deleted records retained indefinitely |
| Audit logs | Indefinite | Never (compliance) | Archived to cold storage after 2 years |
| Sessions, tokens | 90 days | Automated nightly purge | Redis TTL for active sessions; audit trail 90 days |
| Event outbox | 30 days | Automated nightly purge | Published events purged after 30 days |
| ESG / emissions | Indefinite | Never (regulatory) | Required for GRI/SASB reporting |
| ML model artifacts | 2 years | Automated archive | Archived models retained 2 years |
| Subscription/revenue records | 7 years | Never (compliance) | ASC 606 / IFRS 15 compliance |
| Trade compliance logs | 5 years (regulatory) | Never (regulatory) | Export control and sanctions compliance |
| IDP extraction results | 90 days | Automated nightly purge | PII-containing document extractions |
| Lease/grant/JV records | Duration + 7 years | Never (compliance) | IFRS 16 / ASC 842 requirements |
| Import/export job data | 90 days | Automated nightly purge | Job metadata and temp files |
| User activity logs | 1 year | Automated archive | Archived to cold storage after 1 year |
| ML training data | 2 years | Automated archive | Anonymized training datasets |
| File metadata | Indefinite | Never | File metadata kept; file content per tenant policy |
| File content | Per tenant policy | Per tenant policy | Configurable retention per content type |
| Report execution history | 2 years | Automated archive | Archived to cold storage |
| Custom application data | Per tenant policy | Per tenant policy | Application Composer data follows tenant lifecycle |
| Scheduled job execution history | 90 days | Automated nightly purge | Job execution logs and metadata |
| Knowledge article versions | Indefinite | Never | All historical versions retained |
| Survey responses | Per tenant policy | Per tenant policy | Active surveys retained; deactivated surveys archived |
| Collection activities | 7 years | Never (compliance) | Financial regulatory compliance |
| Data lake (curated zone) | Indefinite | Automated archive after 2 years | Gold zone data kept for analytics |
| Data lake (raw zone) | 2 years | Automated nightly purge | Bronze zone events archived after 2 years |

### 9.9 Data Pipeline

Report Service NEVER reads from operational PostgreSQL directly. Pipeline: Service Events → Outbox → RabbitMQ → Event Archive Worker → MinIO Raw → ETL → MinIO Curated → MinIO Gold → ETL Load → DuckDB (star schema). Real-time dashboards use event stream aggregation, NOT warehouse.

---

## 10. API Design Rules

### 10.1 Standards

| Standard | Specification |
|----------|---------------|
| Style | RESTful with OpenAPI 3.1 |
| Versioning | URL path (`/api/v1/...`) |
| Auth | Bearer JWT |
| Content-Type | `application/json` |
| Error Format | RFC 7807 Problem Details |
| Pagination | Cursor-based (`limit`/`after`, default 50, max 100) |
| Filtering | Query params with operators (`eq`, `ne`, `gt`, `gte`, `lt`, `lte`, `like`, `in`) |
| Sorting | `sort=field:direction` syntax |
| Idempotency | `Idempotency-Key` header on all state-changing requests |
| Compression | `gzip` for responses > 1KB |
| Locale | `Accept-Language` header |
| Timezone | `Time-Zone` header (defaults to tenant timezone) |
| Async | `202 Accepted` with job reference |

### 10.2 Idempotency

Required on all `POST`, `PUT`, `PATCH`, `DELETE`. 24-hour window. Scoped to tenant + user + endpoint. Payload mismatch returns `422 IDEMPOTENCY_CONFLICT`.

### 10.3 Rate Limiting

| Scope | Limit | Window |
|-------|-------|--------|
| Per tenant (global) | 10,000 | 1 min |
| Per user | 1,000 | 1 min |
| Auth endpoints | 10 | 1 min |
| File upload | 50 | 1 min |
| Report generation | 10 | 1 min |
| Bulk operations | 20 | 1 min |
| AI/ML inference | 30 | 1 min |
| Digital assistant | 60 | 1 min |

Responses include `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`. Exceeding returns `429` with `Retry-After`.

### 10.4 API Versioning

Max 2 concurrent versions. 90-day sunset notice via `Warning: 299` header. Breaking changes require version bump.

### 10.5 Response Formats

- **Success**: `{ "data": {...}, "meta": { "request_id", "timestamp" } }`
- **List**: `{ "data": [...], "meta": { "request_id", "timestamp", "pagination": { "limit", "has_more", "cursor", "total_count" } } }`
- **Error**: RFC 7807 with `type`, `title`, `status`, `detail`, `instance`, `errors[]`, `meta`.
- **Async Job**: `{ "data": { "job_id", "status", "status_url" }, "meta": {...} }`

### 10.6 Optimistic Concurrency

Every resource includes `version` field. `PUT`/`PATCH` must include current version. Mismatch returns `409 RESOURCE_CONFLICT`.

### 10.7 Health Checks

| Endpoint | Purpose | K8s Probe |
|----------|---------|-----------|
| `GET /healthz` | Liveness (process running, no dependency checks) | `livenessProbe` |
| `GET /readyz` | Readiness (all critical dependencies reachable) | `readinessProbe` |

Each check has 1-second timeout. Checks run concurrently.

### 10.8 Bulk Operations

Max 100 items. Default: atomic (single transaction). `Prefer: partial-success` header enables non-atomic. `413` for > 100 items.

> **Full endpoint listing:** [docs/05-api/endpoints.md](docs/05-api/endpoints.md) (723 endpoints)
> **Full error codes:** [docs/05-api/error-codes.md](docs/05-api/error-codes.md) (243 error codes)

---

## 11. Security & Authorization

### 11.1 JWT Specification

| Claim | Access Token | Refresh Token |
|-------|-------------|---------------|
| `sub` | User UUID | User UUID |
| `tenant_id` | Tenant UUID | Tenant UUID |
| `roles` | Role strings array | Not included |
| `permissions` | Permission strings array | Not included |
| `org_scope` | Org unit UUID (optional) | Not included |
| `exp` | 15 minutes | 7 days |
| `iat` | Issued at timestamp | Issued at timestamp |
| `jti` | Token UUID (unique) | Token UUID (unique) |
| `type` | `access` | `refresh` |

Algorithm: RS256. Public key cached by services, rotated weekly. Token revocation: Redis blocklist (access), DB deletion (refresh). Refresh token rotation on every refresh.

### 11.2 Service-to-Service Auth

| Method | Detail |
|--------|--------|
| Service JWTs | `svc-{service-name}`, 1-hour expiry, separate signing key, `scope` array |
| mTLS | Internal CA via cert-manager, 90-day validity, TLS 1.3 only |
| Cross-DB access | NOT ALLOWED |

### 11.3 RBAC Permission Format

`{domain}.{entity}.{action}` where domains are the 14 service names, actions are `create`, `read`, `update`, `delete`, `own`, `approve`, `export`, `*`. Legacy mappings: `sales.*` → `commerce.*`, `inventory.*` → `commerce.*`, `procurement.*` → `finance.*`.

**Wildcard Semantics:**
- `finance.*` — all actions on all entities within the finance domain
- `finance.journal.*` — all actions on the journal entity within finance
- `finance.*.read` — read access to all entities within the finance domain
- `*` — super admin; all actions on all entities across all domains

**`own` Action:** Grants access to only the user's own records within that entity. Maps to `WHERE created_by = current_user_id` in addition to standard RLS. Used for self-service scenarios: `hr.attendance.own`, `hr.leave.own`, `hr.expense.own`.

### 11.4 ABAC Attributes

7 attributes evaluated at service layer (not gateway): org unit, resource department, amount threshold, time of day, IP range, data classification, device trust level.

### 11.5 Authorization Flow

1. Gateway extracts `permissions` from JWT (embedded at login).
2. Gateway checks required permission locally (no network call).
3. Service applies row-level checks (RLS, org scope, amount rules).
4. Permission changes take effect on next token refresh (15-minute TTL).

### 11.6 Data Protection

| Aspect | Implementation |
|--------|---------------|
| Encryption at rest | PostgreSQL TDE + pgcrypto AES-256 for PII; MinIO SSE-S3 AES-256; Redis TLS |
| Key management | Master key → DEK → per-table key; rotation every 90 days; Vault/KMS |
| PII handling | 4 classification levels: Public, Internal, Confidential, Restricted |
| GDPR | Right to access, erasure (hard delete with anonymized audit refs), portability |
| Data masking | Static/dynamic/subsetting for non-production environments |

### 11.7 Predefined Roles

Super Admin, Tenant Admin, Finance Manager, Accountant, Treasury Manager, Sales Manager, Sales Rep, Inventory Manager, Procurement Manager, HR Manager, HR Specialist, Employee, Project Manager, CRM User, Manufacturing Manager, Quality Manager, Report Viewer, ESG Analyst, GRC Officer, App Builder, Subscription Manager, Field Service Manager, Trade Compliance Officer, Knowledge Manager, Data Privacy Officer, Content Manager, DLP Analyst, IoT Manager, Collaboration Admin, Logistics Manager, Lease Accountant, Grant Manager, Joint Venture Accountant, Collections Specialist, Warranty Manager, Tax Manager, Commodity Manager, Spend Analyst, Diversity Program Manager, Loyalty Manager, Contact Center Agent, Contact Center Manager, Social Seller, CX Testing Manager, EHS Manager, EHS Specialist, MRO Manager, Product Compliance Manager, IT Service Agent, IT Service Manager, Omnichannel Manager, Price Analyst, Portfolio Manager, Compliance Hub Manager, Career Development Manager, Production Scheduler, Storefront Manager, MES Operator, Extension Developer, ML Studio User, Access Certifier, S&OP Planner, Channel Revenue Manager, Tax Provision Analyst, Shared Services Specialist, IT Financial Manager, Opportunity Marketplace Manager, Skills Administrator, Workforce Safety Officer, Shift Scheduler, Employee Experience Manager, Contract Manufacturing Coordinator, Innovation Manager, Sales Compensation Analyst, Depot Repair Technician, Guided Learning Author, Warehouse Manager, Transportation Manager, Global Trade Compliance Manager, Marketing Automation Manager, Subledger Accountant, CPQ Designer, Retail POS Operator, Property Manager, Tax Filing Specialist, Policy Analyst, Construction Project Manager.

> **Full role definitions with permissions:** [docs/02-security/authorization.md](docs/02-security/authorization.md)

---

## 12. Feature-to-Service Registry

| Feature | Owning Service | Spec File |
|---------|---------------|-----------|
| Account Reconciliation | Finance Service | [reconciliation.md](docs/07-features/reconciliation.md) |
| Adaptive Intelligence | Platform Service (lifecycle) / Report Service (inference) | [adaptive-intelligence.md](docs/07-features/adaptive-intelligence.md) |
| API Marketplace | Integration Service | [api-marketplace.md](docs/07-features/api-marketplace.md) |
| B2C Commerce & Storefront | Commerce Service | [b2c-commerce.md](docs/07-features/b2c-commerce.md) |
| Channel Revenue Management | Commerce Service | [channel-revenue-management.md](docs/07-features/channel-revenue-management.md) |
| Collaboration | Platform Service | [collaboration.md](docs/07-features/collaboration.md) |
| Compliance Hub | Platform Service | [compliance-hub.md](docs/07-features/compliance-hub.md) |
| Connected Planning | Report Service | [connected-planning.md](docs/07-features/connected-planning.md) |
| Contract Manufacturing | Manufacturing Service | [contract-manufacturing.md](docs/07-features/contract-manufacturing.md) |
| Content Management | Platform Service | [content-management.md](docs/07-features/content-management.md) |
| Depot Repair | CRM Service | [depot-repair.md](docs/07-features/depot-repair.md) |
| Digital Assistant | Platform Service | [digital-assistant.md](docs/07-features/digital-assistant.md) |
| DLP | Platform Service | [dlp.md](docs/07-features/dlp.md) |
| Dynamic Discounting | Finance Service | [dynamic-discounting.md](docs/07-features/dynamic-discounting.md) |
| Dynamic Skills | HCM Service | [dynamic-skills.md](docs/07-features/dynamic-skills.md) |
| EDI | Integration Service | [edi.md](docs/07-features/edi.md) |
| Email Service | Platform Service | [email.md](docs/07-features/email.md) |
| Employee Experience | HCM Service | [employee-experience.md](docs/07-features/employee-experience.md) |
| Enterprise Data Quality | Integration Service | [enterprise-data-quality.md](docs/07-features/enterprise-data-quality.md) |
| Event Mesh | Integration Service | [event-mesh.md](docs/07-features/event-mesh.md) |
| Financial Reporting Studio | Finance Service (templates) / Report Service (rendering) | [financial-reporting-studio.md](docs/07-features/financial-reporting-studio.md) |
| Full-Text Search | Platform Service | [search.md](docs/07-features/search.md) |
| Global Order Promising (GOP) | Commerce Service | [global-order-promising.md](docs/07-features/global-order-promising.md) |
| Fixed Asset Lifecycle | Finance Service | [fixed-asset-lifecycle.md](docs/07-features/fixed-asset-lifecycle.md) |
| Advanced Demand Management | Commerce Service | [advanced-demand-management.md](docs/07-features/advanced-demand-management.md) |
| Goals & Career Development | HCM Service | [goals-career-development.md](docs/07-features/goals-career-development.md) |
| Guided Learning | Platform Service | [guided-learning.md](docs/07-features/guided-learning.md) |
| IDP | Platform Service | [idp.md](docs/07-features/idp.md) |
| Innovation Management | Manufacturing Service | [innovation-management.md](docs/07-features/innovation-management.md) |
| Intelligent Close | Finance Service | [intelligent-close.md](docs/07-features/intelligent-close.md) |
| Intelligent Process Automation | Platform Service | [intelligent-process-automation.md](docs/07-features/intelligent-process-automation.md) |
| IT Financial Management | Finance Service | [it-financial-management.md](docs/07-features/it-financial-management.md) |
| MES (Manufacturing Execution System) | Manufacturing Service | [mes.md](docs/07-features/mes.md) |
| Multi-Tenancy | Tenant Service | [multi-tenancy.md](docs/07-features/multi-tenancy.md) |
| Opportunity Marketplace | HCM Service | [opportunity-marketplace.md](docs/07-features/opportunity-marketplace.md) |
| Structured Onboarding Journey | HCM Service | [onboarding-journey.md](docs/07-features/onboarding-journey.md) |
| Order Orchestration | Commerce Service | [order-orchestration.md](docs/07-features/order-orchestration.md) |
| Privacy Management | Platform Service | [privacy.md](docs/07-features/privacy.md) |
| Production Scheduling | Manufacturing Service | [production-scheduling.md](docs/07-features/production-scheduling.md) |
| Sales & Operations Planning | Commerce Service | [sales-operations-planning.md](docs/07-features/sales-operations-planning.md) |
| Sales Performance Management | CRM Service | [sales-performance-management.md](docs/07-features/sales-performance-management.md) |
| Self-Service ML Studio | Report Service | [self-service-ml.md](docs/07-features/self-service-ml.md) |
| Shared Services Center | Finance Service | [shared-services-center.md](docs/07-features/shared-services-center.md) |
| Supply Chain Collaboration | Commerce Service | [supply-chain-collaboration.md](docs/07-features/supply-chain-collaboration.md) |
| Access Certification | Platform Service | [access-certification.md](docs/07-features/access-certification.md) |
| Application Composer | Platform Service | [application-composer.md](docs/07-features/application-composer.md) |
| Tax Reporting & Provision | Finance Service | [tax-reporting.md](docs/07-features/tax-reporting.md) |
| Touchless Invoicing | Finance Service | [touchless-invoicing.md](docs/07-features/touchless-invoicing.md) |
| Workforce Health & Safety | HCM Service | [workforce-health-safety.md](docs/07-features/workforce-health-safety.md) |
| Workforce Scheduling | HCM Service | [workforce-scheduling.md](docs/07-features/workforce-scheduling.md) |
| Advanced Warehouse Management (WMS) | Commerce Service | [advanced-warehouse-management.md](docs/07-features/advanced-warehouse-management.md) |
| Advanced Transportation Management (TMS) | Commerce Service | [advanced-transportation.md](docs/07-features/advanced-transportation.md) |
| Global Trade Management | Integration Service | [global-trade-management.md](docs/07-features/global-trade-management.md) |
| Marketing Automation | CRM Service | [marketing-automation.md](docs/07-features/marketing-automation.md) |
| Subledger Accounting Engine | Finance Service | [subledger-accounting.md](docs/07-features/subledger-accounting.md) |
| Configure, Price, Quote (CPQ) | Commerce Service | [cpq.md](docs/07-features/cpq.md) |
| Retail Point-of-Sale | Commerce Service | [retail-pos.md](docs/07-features/retail-pos.md) |
| Construction & Engineering | Project Service | [construction-engineering.md](docs/07-features/construction-engineering.md) |
| Policy Automation | Platform Service | [policy-automation.md](docs/07-features/policy-automation.md) |
| Advanced Tax Filing | Finance Service | [advanced-tax-filing.md](docs/07-features/advanced-tax-filing.md) |
| Property Management | Commerce Service | [property-management.md](docs/07-features/property-management.md) |

> **All feature specs:** [docs/07-features/](docs/07-features/) (61 feature specifications)

---

## 13. Oracle Fusion Cloud Parity Map

| Oracle Fusion Product Family | Oracle Modules | MSERP Service(s) |
|-----------------------------|---------------|-------------------|
| **Financials** | GL, AP, AR, Fixed Assets, Cash, Expenses, Collections, Revenue, Lease, Grant, JV, Tax, Multi-Book, Financial Consolidation, Accounting Hub, Tax Provision, Touchless Invoicing, Shared Services, Subledger Accounting Engine, Advanced Tax Filing | Finance Service |
| **Procurement** | Supplier Management, Sourcing, Contracts, Purchasing, Self-Service Procurement, Supplier Portal, Procurement Analytics | Finance Service |
| **SCM** | Inventory, Order Management, Logistics, Product Hub, Order Orchestration, GOP, B2C Commerce, Demand Management, S&OP, Channel Revenue Management, Warehouse Management, Transportation Management, CPQ, Retail POS, Property Management | Commerce Service + Manufacturing Service |
| **HCM** | Core HR, Payroll, Recruiting, Performance, Time & Labor, Learning, Benefits, Talent, Global HR, Help Desk, LXP, Goals, Career Development, Onboarding Journeys, Opportunity Marketplace, Dynamic Skills, Workforce Health & Safety, Workforce Scheduling, Employee Experience | HCM Service |
| **CX** | Sales, Service, Marketing, Commerce, CPQ, Field Service, Surveys, Contact Center, Social, Loyalty, CDP, Forecasting, Partners, Sales Performance Management, Depot Repair, Marketing Automation | CRM Service + Commerce Service |
| **ERP** | Projects, Costing, Billing, Manufacturing, Maintenance, Quality, Construction & Engineering | Project Service + Manufacturing Service |
| **SCM Planning** | Demand, Supply, Inventory Optimization, ATP/CTP, GOP, Order Orchestration, Connected Planning | Commerce Service + Manufacturing Service + Report Service |
| **Manufacturing** | Execution (MES), Quality, Costing, Planning, Digital Twin, IoT, Process Mfg, Mixed-Mode, Advanced Production Scheduling, Contract Manufacturing, Innovation Management | Manufacturing Service |
| **EPM** | Planning, Budgeting, Forecasting, Consolidation, Reporting, Profitability, IT Financial Management, Tax Reporting | Finance Service + Report Service |
| **GRC** | Access Controls, Audit, Risk, Compliance, Privacy, SoD, Access Certification, DLP | Platform Service + Auth Service |
| **Integration** | Integration Cloud, B2B, EDI, Connectors, Flow Designer, MFT, Data Quality, Event Mesh, Global Trade Management | Integration Service |
| **Platform** | App Builder, Application Composer, Digital Assistant, Mobile, Search, Workflow, Notifications, Scheduler, MDM, RPA, Content, Collaboration, Access Certification, Guided Learning, Policy Automation | Platform Service + Workflow Service + Integration Service |
| **Analytics** | BI, DV, ML, Augmented Analytics, Self-Service ML Studio, Process Mining, Carbon Accounting | Report Service |

---

## 14. AI/ML Strategy

### 14.1 AI Architecture

```
Business Services Layer (Finance, Commerce, HCM, CRM, Manufacturing, Project, Platform)
    ↕
Adaptive Intelligence Layer (Recommendations, Anomaly Detection, Predictions, NLQ)
    ↕
ML Runtime (ONNX + Candle: Model Serving, Feature Store, A/B Testing, Explain)
    ↕
Model Training Pipeline (Data Prep, Training, Validation, Registry, Deploy)
```

Platform Service owns model lifecycle. Report Service owns inference runtime.

### 14.2 AI Capabilities by Module

| Module | Capability | Model Type |
|--------|-----------|------------|
| Finance | Anomaly detection, cash flow forecasting, payment delay prediction, reconciliation matching | Time-series, Classification |
| Commerce | Demand forecasting, price optimization, churn prediction, cross-sell/up-sell | Regression, Classification, RL |
| HCM | Attrition prediction, resume screening, workforce demand forecasting | Classification, NLP, Time-series |
| CRM | Lead scoring, next-best-action, deal risk prediction, sentiment analysis | Classification, Regression, NLP |
| Manufacturing | Predictive maintenance, quality prediction, yield optimization | Time-series, Classification |
| Project | Risk prediction, timeline estimation, resource optimization | Regression, Classification |
| Platform | Document classification (IDP), NLP assistant, process mining | NLP, CV, Classification |
| Report | NL-to-SQL, auto-insight generation, augmented analytics | NLP, Transformer |

### 14.3 AI Governance

| Principle | Implementation |
|-----------|---------------|
| Explainability | SHAP/LIME scores on all predictions |
| Fairness | Bias testing for protected attributes |
| Privacy | Training data anonymized; PII excluded from features |
| Human Override | All suggestions overridable; low-confidence flagged |
| Audit Trail | All decisions logged with inputs, model version, confidence |

---

## 15. Integration Ecosystem

Integration Service provides 50+ pre-built connectors:

- **Financial**: SAP S/4HANA, NetSuite, Dynamics 365, QuickBooks, Xero, Sage, Stripe, PayPal, Adyen, Square, Plaid, SWIFT, Vertex, Avalara
- **CRM & CX**: Salesforce, HubSpot, Dynamics 365 Sales, Zendesk, Freshdesk, Intercom, Twilio, SendGrid
- **HR & People**: Workday, BambooHR, ADP, Paychex, LinkedIn Recruiter, Greenhouse
- **Supply Chain & Logistics**: FedEx, UPS, DHL, USPS, Amazon Seller Central, Shopify, Magento, WooCommerce
- **Collaboration & Productivity**: Microsoft 365, Google Workspace, Slack, Microsoft Teams, Jira, Confluence, Notion
- **Infrastructure & Developer**: AWS S3, Azure Blob, GCS, SMTP/IMAP, SFTP, Generic REST/SOAP/Webhook, EDI, OPC-UA, MQTT, Snowflake, Power BI, Tableau

> **Full connector details:** [docs/06-services/integration.md](docs/06-services/integration.md)

---

## 16. Deployment Rules

### 16.1 Active-Passive Failover

| Aspect | Specification |
|--------|---------------|
| Failover trigger | Automated: health check failure > 3 probes (30s); Manual: admin-initiated |
| DNS failover | TTL < 60s, automatic switch |
| Database promotion | PostgreSQL synchronous streaming replication |
| Event broker | RabbitMQ federation with queue mirroring |
| Split-brain prevention | Leader election via etcd with fencing tokens |

### 16.2 Graceful Shutdown

Mandatory 10-step sequence: stop accepting new requests (30s HTTP drain) → drain event consumers (15s) → flush outbox (10s) → close DB connections → exit. `terminationGracePeriodSeconds: 90`.

### 16.3 Deployment Strategies

| Strategy | Use Case |
|----------|----------|
| Rolling update (default) | Minor changes |
| Blue-Green | Major version upgrades |
| Canary | High-risk changes (Commerce, Finance) |

Canary rollback: automatic if error rate exceeds baseline by 0.5%. Blue-Green rollback: automatic on error rate > 1%.

### 16.4 Autoscaling

HPA per service with configured min/max replicas. VPA in recommendation-only mode.

### 16.5 Multi-Region

Same signed image artifact promoted through environments. Production deployment requires manual approval.

> **Full deployment spec:** [docs/08-infrastructure/deployment.md](docs/08-infrastructure/deployment.md)

---

## 17. Non-Functional Requirements

### 17.1 Performance

| Metric | Target |
|--------|--------|
| API p50 | < 50ms |
| API p99 | < 500ms |
| API p99.9 | < 2s |
| Throughput | 10,000+ req/s |
| Concurrency | 1,000+ users |

### 17.2 Availability

| Metric | Target |
|--------|--------|
| Uptime SLA | 99.9% (8.76 hours/year) |
| Data durability | 99.999% (5 nines) |
| RTO (automated) | < 15 minutes |
| RTO (region failover) | < 1 hour |
| RPO | < 5 minutes |
| Deployment downtime | Zero |

### 17.3 Scalability

| Metric | Target |
|--------|--------|
| Tenants per cluster | 1,000 |
| Concurrent users | 1,000+ |
| Data per tenant | Up to 100 GB |
| Data per cluster | Up to 10 TB |

> **Full NFR spec:** [docs/10-planning/nfr.md](docs/10-planning/nfr.md)

---

## 18. Testing Requirements

### 18.1 Test Types

| Type | Scope | Tool | Coverage Target |
|------|-------|------|----------------|
| Unit | Logic in same file (`#[cfg(test)]`) | Rust built-in | >= 80% per crate |
| Integration | Service-level with testcontainers | testcontainers-rs | All endpoints |
| Contract | Consumer-driven for ALL inter-service boundaries | Pact | All boundaries |
| Saga compensation | Every saga step has compensating action test | Custom | All sagas |

### 18.2 Quality Rules

- `#![deny(unsafe_code)]` enforced at compiler level.
- `cargo tarpaulin` enforces >= 80% coverage.
- CI blocks merge on: clippy warnings, format errors, test failures, coverage regression.
- No direct commits to `main`. Conventional Commits required.

### 18.3 Quality Gates (per phase)

| Phase | Gate |
|-------|------|
| Foundation | 80%+ coverage on core crates |
| Core Business | All consumer contracts verified |
| Extended Business | All saga flows tested with failure scenarios |
| Enterprise Features | 5,000 req/s with < 500ms p99 |
| Production Ready | 10,000 req/s, security audit passed, runbooks validated |

---

## 19. Document Index

### 19.1 Architecture & Design

| Document | Path |
|----------|------|
| Architecture Overview | [docs/01-architecture/overview.md](docs/01-architecture/overview.md) |

### 19.2 Security

| Document | Path |
|----------|------|
| Security Architecture | [docs/02-security/overview.md](docs/02-security/overview.md) |
| Authorization | [docs/02-security/authorization.md](docs/02-security/authorization.md) |
| Data Protection | [docs/02-security/data-protection.md](docs/02-security/data-protection.md) |
| GRC | [docs/02-security/grc.md](docs/02-security/grc.md) |
| Threat Model | [docs/02-security/threat-model.md](docs/02-security/threat-model.md) |

### 19.3 Data

| Document | Path |
|----------|------|
| Data Architecture | [docs/03-data/overview.md](docs/03-data/overview.md) |
| Caching Strategy | [docs/03-data/caching.md](docs/03-data/caching.md) |
| Data Pipeline | [docs/03-data/data-pipeline.md](docs/03-data/data-pipeline.md) |
| Domain Data Models | [docs/03-data/domain-models.md](docs/03-data/domain-models.md) |

### 19.4 Events

| Document | Path |
|----------|------|
| Event-Driven Architecture | [docs/04-events/overview.md](docs/04-events/overview.md) |
| Event Catalog | [docs/04-events/catalog.md](docs/04-events/catalog.md) |
| Saga Patterns | [docs/04-events/sagas.md](docs/04-events/sagas.md) |

### 19.5 API

| Document | Path |
|----------|------|
| API Design Standards | [docs/05-api/standards.md](docs/05-api/standards.md) |
| API Endpoints | [docs/05-api/endpoints.md](docs/05-api/endpoints.md) |
| Error Codes | [docs/05-api/error-codes.md](docs/05-api/error-codes.md) |

### 19.6 Services

| Document | Path |
|----------|------|
| Services Overview | [docs/06-services/overview.md](docs/06-services/overview.md) |
| Individual service specs (14) | [docs/06-services/](docs/06-services/) |

### 19.7 Features

61 feature specifications in [docs/07-features/](docs/07-features/). See §12 for the registry.

### 19.8 Infrastructure & Development

| Document | Path |
|----------|------|
| Technology Stack | [docs/08-infrastructure/technology.md](docs/08-infrastructure/technology.md) |
| Deployment | [docs/08-infrastructure/deployment.md](docs/08-infrastructure/deployment.md) |
| Observability | [docs/08-infrastructure/observability.md](docs/08-infrastructure/observability.md) |
| Project Structure | [docs/09-development/project-structure.md](docs/09-development/project-structure.md) |
| Code Conventions | [docs/09-development/conventions.md](docs/09-development/conventions.md) |
| Local Setup | [docs/09-development/local-setup.md](docs/09-development/local-setup.md) |

### 19.9 Planning & Reference

| Document | Path |
|----------|------|
| Development Phases | [docs/10-planning/phases.md](docs/10-planning/phases.md) |
| Non-Functional Requirements | [docs/10-planning/nfr.md](docs/10-planning/nfr.md) |
| Glossary | [docs/glossary.md](docs/glossary.md) |

---

*Document Version: 26.0*
*Last updated: 2026-04-01*
