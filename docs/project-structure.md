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
│   ├── commerce-service/         # Sales + Inventory (merged)
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
│   │   │   └── pricing/          # Pricing engine module
│   │   │       ├── routes/
│   │   │       ├── handlers/
│   │   │       └── services/
│   ├── finance-service/          # Finance + Procurement (merged)
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
│   │   │   └── currency/         # Multi-currency module
│   │   │       └── services/
│   ├── hr-service/
│   ├── manufacturing-service/
│   ├── report-service/
│   ├── workflow-service/
│   ├── crm-service/
│   ├── project-service/
│   ├── platform-service/         # Notification + File + Audit (merged)
│   │   ├── src/
│   │   │   ├── notification/
│   │   │   ├── file/
│   │   │   └── audit/
│   └── integration-service/
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
├── k8s/
│   ├── base/
│   │   ├── deployments/
│   │   ├── services/
│   │   ├── configmaps/
│   │   ├── secrets/
│   │   ├── hpa/               # Horizontal Pod Autoscalers
│   │   └── pdb/               # Pod Disruption Budgets
│   └── overlays/
│       ├── development/
│       ├── staging/
│       └── production/
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
├── scripts/
│   ├── setup.sh
│   ├── migrate.sh
│   ├── seed.sh
│   └── generate-openapi.sh
│
└── reference-data/               # Static seed data
    ├── currencies.csv
    ├── countries.csv
    ├── locales.csv
    ├── tax-jurisdictions.csv
    └── uoms.csv
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
| `mserp-core` | Error types, Result types, config loading, pagination, i18n, money types, utilities | All services |
| `mserp-auth` | JWT validation, auth middleware, permission checks | All services |
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
