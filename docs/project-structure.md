# Project Structure

## 1. Repository Layout

```
mserp/
├── Cargo.toml                    # Workspace root
├── Cargo.lock
├── docker-compose.yml            # Full local development (all services)
├── docker-compose.dev.yml        # Minimal dev (infra + core services only)
├── Makefile                      # Development commands
├── README.md
├── SPEC.md                       # Master spec index
│
├── docs/                         # Documentation
│   ├── architecture.md
│   ├── services.md
│   ├── technology.md
│   ├── api-design.md
│   ├── data-architecture.md
│   ├── events.md
│   ├── security.md
│   ├── deployment.md
│   ├── project-structure.md
│   ├── phases.md
│   ├── non-functional.md
│   ├── glossary.md
│   ├── api/                    # OpenAPI specs (generated)
│   ├── architecture/           # Architecture diagrams (generated)
│   └── runbooks/               # Operational runbooks
│
├── crates/
│   ├── mserp-core/               # Shared core library
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── config.rs         # Configuration management
│   │       ├── error.rs          # Error types (RFC 7807 compatible)
│   │       ├── result.rs         # Result types
│   │       ├── pagination.rs     # Cursor pagination utilities
│   │       ├── i18n.rs           # Internationalization helpers
│   │       ├── money.rs          # Currency/money types (rust_decimal based)
│   │       ├── classification.rs # Data classification types
│   │       └── utils/            # Utilities
│   │
│   ├── mserp-auth/               # Auth domain types
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── domain/
│   │       ├── dto/
│   │       └── events/
│   │
│   ├── mserp-infrastructure/     # Infrastructure concerns
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── database/
│   │       │   ├── mod.rs
│   │       │   ├── pool.rs       # Connection pool with RLS support
│   │       │   ├── outbox.rs     # Transactional outbox
│   │       │   └── migration.rs  # Migration runner
│   │       ├── cache/
│   │       │   ├── mod.rs
│   │       │   └── redis.rs      # Cache-aside implementation
│   │       ├── messaging/
│   │       │   ├── mod.rs
│   │       │   ├── rabbitmq.rs   # RabbitMQ publisher/consumer
│   │       │   └── saga.rs       # Saga state tracking
│   │       └── tracing/
│   │           ├── mod.rs
│   │           ├── jaeger.rs     # OpenTelemetry setup
│   │           └── metrics.rs    # Prometheus metrics setup
│   │
│   └── mserp-proto/              # Protocol definitions
│       ├── Cargo.toml
│       └── src/
│           ├── lib.rs
│           └── events/           # Event type definitions for all services
│
├── services/
│   ├── auth-service/
│   │   ├── Cargo.toml
│   │   ├── Dockerfile
│   │   └── src/
│   │       ├── main.rs
│   │       ├── config.rs
│   │       ├── routes/
│   │       ├── handlers/
│   │       ├── models/
│   │       ├── repository/
│   │       └── services/
│   │
│   ├── identity-service/
│   ├── tenant-service/
│   ├── config-service/
│   ├── commerce-service/         # Sales + Inventory + PIM + Transportation
│   │   ├── src/
│   │   │   ├── main.rs
│   │   │   ├── config.rs
│   │   │   ├── sales/            # Sales domain module
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   ├── repository/
│   │   │   │   └── services/
│   │   │   ├── inventory/        # Inventory domain module
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   ├── repository/
│   │   │   │   └── services/
│   │   │   ├── pricing/          # Pricing engine module
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   └── services/
│   │   │   ├── pim/              # Product Information Management
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── transportation/   # Fleet & Transportation
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── atp/                # Available-to-Promise
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── configurator/       # Product Configurator
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── credit/             # Credit Management
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── subscriptions/      # Subscription Management
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   └── dropship/           # Intercompany Drop Ship
│   │   │       ├── routes/
│   │   │       ├── handlers/
│   │   │       ├── models/
│   │   │       └── services/
│   ├── finance-service/          # Finance + Procurement + Treasury + Expenses + CLM + EPM
│   │   ├── src/
│   │   │   ├── finance/          # Finance domain module
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   ├── repository/
│   │   │   │   └── services/
│   │   │   ├── procurement/      # Procurement domain module
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   ├── repository/
│   │   │   │   └── services/
│   │   │   ├── treasury/         # Corporate Treasury
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── expenses/         # Enterprise Expense Management
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── contracts/        # Contract Lifecycle Management
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── epm/              # Enterprise Performance Management
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── currency/         # Multi-currency module
│   │   │   │   └── services/
│   │   │   └── revenue-recognition/ # Revenue Recognition (ASC 606/IFRS 15)
│   │   │       ├── routes/
│   │   │       ├── handlers/
│   │   │       ├── models/
│   │   │       └── services/
│   ├── hr-service/
│   │   ├── src/
│   │   │   ├── employee/
│   │   │   ├── attendance/
│   │   │   ├── leave/
│   │   │   ├── payroll/
│   │   │   │   ├── domestic/     # Single-country payroll
│   │   │   │   └── multi-country/ # Multi-country payroll localizations
│   │   │   ├── recruitment/
│   │   │   ├── onboarding/
│   │   │   ├── performance/
│   │   │   ├── talent/           # Talent review & succession planning
│   │   │   ├── workforce/        # Workforce modeling
│   │   │   ├── training/
│   │   │   ├── compensation/
│   │   │   └── benefits/
│   ├── manufacturing-service/
│   │   ├── src/
│   │   │   ├── bom/
│   │   │   ├── work-orders/
│   │   │   ├── planning/         # MPS, MRP
│   │   │   ├── ascp/             # Advanced Supply Chain Planning
│   │   │   ├── quality/          # Quality control + AQM + SPC
│   │   │   ├── maintenance/
│   │   │   ├── eam/              # Enterprise Asset Management
│   │   │   ├── work-centers/
│   │   │   ├── routing/
│   │   │   ├── cost-accounting/
│   │   │   ├── shop-floor/
│   │   │   ├── plm/              # Product Lifecycle Management
│   │   │   │   ├── revisions/
│   │   │   │   ├── eco/          # Engineering Change Orders
│   │   │   │   └── phase-management/
│   │   │   └── sustainability/   # Energy, waste, emissions tracking
│   ├── report-service/
│   │   ├── src/
│   │   │   ├── reports/
│   │   │   ├── dashboards/
│   │   │   ├── kpis/
│   │   │   ├── insights/         # AI-driven insights
│   │   │   ├── warehouse/        # DuckDB star schema
│   │   │   ├── data-lake/        # Data lake zone management
│   │   │   ├── esg/              # ESG / sustainability reporting
│   │   │   ├── carbon/           # Carbon accounting
│   │   │   └── ml/               # Embedded ML/AI platform
│   │   │       ├── models/       # Model registry
│   │   │       ├── features/     # Feature store
│   │   │       └── training/     # Model training pipeline
│   ├── workflow-service/
│   │   ├── src/
│   │   │   ├── engine/           # BPMN engine
│   │   │   ├── approvals/
│   │   │   ├── escalations/
│   │   │   ├── templates/
│   │   │   ├── sla/
│   │   │   ├── rules/            # Business rules engine
│   │   │   └── sod/              # Segregation of Duties checks
│   ├── crm-service/
│   │   ├── src/
│   │   │   ├── contacts/
│   │   │   ├── leads/
│   │   │   ├── opportunities/
│   │   │   ├── campaigns/
│   │   │   ├── activities/
│   │   │   ├── cases/            # Customer service cases
│   │   │   ├── analytics/
│   │   │   ├── field-service/      # Field Service Management
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── surveys/            # Survey & Feedback
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   └── territories/        # Sales Territory & Quota
│   │   │       ├── routes/
│   │   │       ├── handlers/
│   │   │       ├── models/
│   │   │       └── services/
│   ├── project-service/
│   │   ├── src/
│   │   │   ├── projects/
│   │   │   ├── tasks/
│   │   │   ├── resources/
│   │   │   ├── timesheets/
│   │   │   ├── expenses/
│   │   │   ├── billing/
│   │   │   ├── risks/
│   │   │   ├── evm/              # Earned Value Management
│   │   │   └── programs/         # Program management
│   ├── platform-service/         # Notification + File + Audit + Digital Assistant + App Builder + GRC
│   │   ├── src/
│   │   │   ├── notification/
│   │   │   │   ├── email/
│   │   │   │   ├── sms/
│   │   │   │   ├── push/
│   │   │   │   ├── websocket/    # Real-time notifications
│   │   │   │   └── webhooks/
│   │   │   ├── file/
│   │   │   │   ├── storage/
│   │   │   │   ├── preview/
│   │   │   │   └── ocr/          # Optical character recognition
│   │   │   ├── audit/
│   │   │   ├── assistant/        # Digital Assistant
│   │   │   │   ├── nlp/          # NLP intent recognition
│   │   │   │   ├── dialog/       # Multi-turn dialog management
│   │   │   │   └── actions/      # Service API dispatch
│   │   │   ├── app-builder/      # Low-code Application Builder
│   │   │   │   ├── designer/     # Page designer
│   │   │   │   ├── objects/      # Custom business objects
│   │   │   │   └── publishing/   # App versioning and publishing
│   │   │   ├── grc/              # Governance, Risk & Compliance
│   │   │   │   ├── sod/          # Segregation of Duties
│   │   │   │   ├── risk/         # Risk register
│   │   │   │   ├── compliance/   # Compliance assessments
│   │   │   │   └── incidents/    # GRC incidents
│   │   │   ├── data-masking/     # Data masking and subsetting
│   │   │   ├── scheduler/          # Enterprise Job Scheduler
│   │   │   │   ├── jobs/
│   │   │   │   ├── triggers/
│   │   │   │   └── monitoring/
│   │   │   ├── knowledge/          # Knowledge Management
│   │   │   │   ├── articles/
│   │   │   │   ├── categories/
│   │   │   │   ├── search/
│   │   │   │   └── analytics/
│   │   │   ├── signatures/         # Digital Signatures
│   │   │   │   ├── workflows/
│   │   │   │   ├── audit/
│   │   │   │   └── providers/
│   │   │   └── self-service/       # Employee Self-Service Portal
│   │   │       ├── portal/
│   │   │       ├── dashboard/
│   │   │       └── preferences/
│   └── integration-service/
│       ├── src/
│       │   ├── connectors/
│       │   ├── webhooks/
│       │   ├── edi/
│       │   ├── import-export/
│       │   ├── mdm/              # Master Data Management
│       │   │   ├── golden-records/
│       │   │   ├── matching/     # Deduplication engine
│       │   │   ├── quality/      # Data quality rules
│       │   │   └── stewardship/  # Data steward workflows
│       │   ├── governance/       # Data governance
│       │   │   ├── catalog/      # Data catalog
│       │   │   ├── lineage/      # Data lineage tracking
│       │   │   └── classification/
│       │   └── trade-compliance/   # Trade Compliance
│       │       ├── screening/      # Denied party screening
│       │       ├── classifications/ # Export control classification
│       │       ├── licenses/       # License management
│       │       └── customs/        # Customs documentation
│
├── migrations/
│   ├── auth/
│   ├── identity/
│   ├── tenant/
│   ├── config/
│   ├── commerce/
│   ├── finance/
│   ├── hr/
│   ├── manufacturing/
│   ├── report/
│   ├── workflow/
│   ├── crm/
│   ├── project/
│   ├── platform/
│   ├── audit/
│   └── integration/
│
├── infra/                          # Infrastructure as Code
│   ├── terraform/                  # Cloud infrastructure
│   │   ├── modules/
│   │   └── environments/
│   │       ├── production/
│   │       └── staging/
│   ├── helm/                       # Helm charts
│   └── k8s/                        # Kubernetes manifests
│       ├── base/
│       │   ├── deployments/
│       │   ├── services/
│       │   ├── configmaps/
│       │   ├── secrets/
│       │   ├── hpa/               # Horizontal Pod Autoscalers
│       │   └── pdb/               # Pod Disruption Budgets
│       └── overlays/
│           ├── development/
│           ├── staging/
│           └── production/
│
├── contracts/                    # Pact contract definitions
│   ├── consumers/
│   └── providers/
│
├── connectors/                   # Integration connector plugins
│   ├── connector-sdk/           # SDK for building connectors
│   ├── stripe/
│   ├── quickbooks/
│   └── ...
│
├── ml/                            # ML model artifacts
│   ├── models/                    # Trained model storage
│   ├── features/                  # Feature definitions
│   └── training/                  # Training scripts and configs
│
├── scripts/
│   ├── setup.sh
│   ├── migrate.sh
│   ├── seed.sh
│   ├── generate-openapi.sh
│   ├── mask-data.sh              # Generate masked data subset
│   └── validate-sod.sh           # Validate SoD rules
│
└── reference-data/               # Static seed data
    ├── currencies.csv
    ├── countries.csv
    ├── locales.csv
    ├── tax-jurisdictions.csv
    ├── uoms.csv
    ├── emission-factors.csv      # EPA/DEFRA emission factors
    ├── esg-frameworks.csv        # GRI, SASB, TCFD mappings
    ├── carrier-service-levels.csv
    ├── payroll-jurisdictions/    # Per-country payroll config
    ├── denied-party-lists/        # Trade compliance screening lists
    ├── export-classifications.csv # ECCN classifications
    ├── revenue-recognition-rules.csv # ASC 606/IFRS 15 standard methods
    ├── subscription-cycles.csv    # Standard billing cycle definitions
    └── knowledge-categories.csv   # Knowledge base category seed data
```

## 2. Local Development

### Minimal Development Setup (`docker-compose.dev.yml`)

For day-to-day development, only infrastructure and the core services needed for the current feature are started:

```bash
# Start infrastructure only (PostgreSQL, Redis, RabbitMQ, MinIO)
make dev-infra

# Start infrastructure + specific services
make dev SERVICES="auth,identity,commerce"

# Run a single service in dev mode (auto-reload)
make run SERVICE=commerce-service

# Run with hot-reload (requires cargo-watch)
make watch SERVICE=commerce-service
```

### `docker-compose.dev.yml` includes:
| Component | Purpose |
|-----------|---------|
| PostgreSQL | Single instance with all databases |
| Redis | Caching, sessions |
| RabbitMQ | Event bus (with management UI on `:15672`) |
| MinIO | Object storage (with console on `:9001`) |
| Jaeger | Distributed tracing UI on `:16686` |
| Mailpit | Email capture for notification testing on `:8025` |
| Elasticsearch | Search (optional, disabled by default) |
| Grafana | Dashboard UI on `:3000` |
| Prometheus | Metrics collection on `:9090` |
| Vault | Secrets management UI on `:8200` |

### Full Stack Setup (`docker-compose.yml`)

For integration testing or demo purposes, all 14 services are started:

```bash
# Start everything
make dev-full

# Run all service tests
make test-all

# Seed databases with test data
make seed

# Run contract tests
make test-contracts

# Run load tests
make load-test
```

## 3. Service Discovery

Services communicate via **Kubernetes DNS** in production and **Docker Compose service names** in development. No separate service registry is needed.

| Environment | Discovery | Example |
|-------------|-----------|---------|
| Production | K8s DNS (`<service>.mserp-services.svc.cluster.local`) | `http://finance-service:8011` |
| Development | Docker DNS (`http://<service>:<port>`) | `http://finance-service:8011` |
| Unit Tests | Mock adapters via trait injection | N/A |

## 4. Shared Libraries

| Crate | Purpose | Used By |
|-------|---------|---------|
| `mserp-core` | Error types, Result types, config loading, pagination, i18n, money types, data classification, utilities | All services |
| `mserp-auth` | JWT validation, auth middleware, permission checks, step-up auth | All services |
| `mserp-infrastructure` | Database setup (RLS), Redis client, RabbitMQ client, outbox, saga, tracing, metrics | All services |
| `mserp-proto` | Event type definitions, shared DTOs, contract schemas | All services |

## 5. Code Conventions

### 5.1 Module Structure (per service)

```
service-name/
├── src/
│   ├── main.rs           # Entry point, server startup, graceful shutdown
│   ├── config.rs         # Service-specific configuration struct
│   ├── routes/           # Axum route definitions
│   │   ├── mod.rs
│   │   └── {entity}.rs   # One file per entity/resource
│   ├── handlers/         # Request handlers
│   ├── models/           # Domain models (SeaORM entities)
│   ├── repository/       # Data access layer (trait + impl)
│   ├── services/         # Business logic
│   └── events/           # Event handlers (RabbitMQ consumers)
├── Cargo.toml
├── Dockerfile
└── tests/                # Integration tests (testcontainers)
```

### 5.2 Naming Conventions

| Item | Convention | Example |
|------|-----------|---------|
| Service crate | `kebab-case` | `commerce-service` |
| Rust module | `snake_case` | `sales_order` |
| Database table | `snake_case` | `sales_orders` |
| Database column | `snake_case` | `customer_id` |
| API endpoint | `kebab-case` | `/api/v1/commerce/price-lists` |
| Event type | `dot.case` | `commerce.order.created` |
| Environment variable | `UPPER_SNAKE_CASE` | `MSERP_DATABASE_URL` |

### 5.3 Error Handling

All services use the shared `mserp-core::error` types which implement RFC 7807 Problem Details. Services define domain-specific error variants that map to HTTP status codes.

---

*See [Deployment](deployment.md) for Kubernetes and CI/CD details.*
*See [Technology Stack](technology.md) for dependency information.*
