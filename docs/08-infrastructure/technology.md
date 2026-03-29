# Technology Stack

## 1. Core Technologies

| Layer | Technology | Version | Purpose |
|-------|------------|---------|---------|
| Language | Rust | 1.83+ | Primary development language (Edition 2021) |
| Runtime | Tokio | 1.x | Async runtime (multi-threaded scheduler) |
| Web Framework | Axum | 0.7+ | HTTP server/framework (Tower-based) |
| ORM | SeaORM | 0.12+ | Database operations (async, dynamic queries) |
| Migration | sea-orm-migration | 0.12+ | Schema migrations (part of SeaORM ecosystem) |

## 2. Infrastructure

| Component | Technology | Version | Purpose |
|-----------|------------|---------|---------|
| Database | PostgreSQL | 16 | Primary data store (all services) |
| Connection Pooler | PgBouncer | 1.22+ | Connection pooling for PostgreSQL |
| Cache | Redis | 7.x | Caching, sessions, rate limiting, pub/sub |
| Message Broker | RabbitMQ | 3.12 | Event bus, async messaging, DLQ |
| Search | Elasticsearch | 8.x | Full-text search, log analytics, product search |
| Object Storage | MinIO | RELEASE.2024-01+ | File/document storage (S3-compatible) |
| API Gateway | Traefik (primary) / Kong (alternative) | 3.x | Routing, rate limiting, auth, feature flags |
| Container Orchestration | Kubernetes | 1.28+ | Container orchestration and scheduling |
| Log Aggregation | Loki | Latest | Log aggregation and querying |
| Analytics Engine | DuckDB | 0.10+ | Embedded analytical database for Report Service |
| Vector Engine | qdrant (optional) | 1.8+ | Vector search for AI/ML similarity queries |
| ML/AI Runtime | ONNX Runtime | 1.17+ | Embedded ML model inference |
| Secrets Management | HashiCorp Vault | 1.15+ | Key management, secret storage, dynamic credentials |
| Data Lake Storage | MinIO (S3-compatible) | RELEASE.2024-01+ | Raw and curated data lake zones |
| IaC | Terraform | 1.7+ | Cloud infrastructure provisioning |
| GitOps | ArgoCD | 2.10+ | GitOps continuous delivery |
| Mobile Framework | React Native | 0.73+ | iOS and Android mobile application |
| Web Frontend | React + TypeScript | 18.x / 5.x | Admin dashboard and web application |

## 3. Observability

| Component | Technology | Version | Purpose |
|-----------|------------|---------|---------|
| Metrics | Prometheus + Grafana | Latest | System metrics, dashboards, alerting |
| Logging | Loki + Grafana | Latest | Log aggregation, structured log queries |
| Tracing | Jaeger | Latest | Distributed tracing, request flow visualization |
| Alerting | Alertmanager | Latest | Incident management, alert routing |
| Uptime | Blackbox Exporter | Latest | External endpoint monitoring |
| Synthetics | k6 (cloud) | Latest | Synthetic transaction monitoring |
| Long-term Metrics | Thanos (optional) | Latest | Long-term Prometheus storage with S3 backend |

## 4. Rust Crate Selection

```toml
[dependencies]
# Web
axum = "0.7"
axum-extra = { version = "0.9", features = ["typed-header", "query"] }
tower = "0.4"
tower-http = { version = "0.5", features = ["cors", "trace", "compression-gzip", "request-id", "propagate-header", "ws"] }
hyper = "1.0"

# Async
tokio = { version = "1", features = ["full"] }

# Database
sea-orm = { version = "0.12", features = ["sqlx-postgres", "runtime-tokio-rustls"] }

# HTTP Client
reqwest = { version = "0.12", features = ["json", "rustls-tls"], default-features = false }

# Serialization
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"

# Validation
validator = "0.16"

# Auth
jsonwebtoken = "9.2"
argon2 = "0.5"
totp-rs = "5.0"

# Messaging
lapin = "2.3"
deadpool-lapin = "0.10"

# Cache
redis = { version = "0.24", features = ["tokio-comp"] }

# Config
config = "0.14"
dotenvy = "0.15"

# Logging
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter", "json"] }

# OpenAPI
utoipa = "4.2"
utoipa-swagger-ui = "4.0"

# Utils
uuid = { version = "1.6", features = ["v4", "v7"] }
chrono = { version = "0.4", features = ["serde"] }
chrono-tz = "0.8"
thiserror = "1.0"
anyhow = "1.0"
rust_decimal = { version = "1.33", features = ["serde"] }

# Observability
opentelemetry = "0.27"
opentelemetry-semantic-conventions = "0.27"
tracing-opentelemetry = "0.28"
opentelemetry-otlp = "0.27"
prometheus = "0.13"

# ML/AI (optional, Report + Platform services)
ort = { version = "2.0", optional = true }
candle-core = { version = "0.4", optional = true }
candle-nn = { version = "0.4", optional = true }
tokenizers = { version = "0.15", optional = true }

# NLP (Platform Service — Digital Assistant)
rust-bert = { version = "0.22", optional = true }

# Analytics (Report Service only)
duckdb = { version = "1.0", optional = true }
arrow = { version = "50.0", optional = true }

# CSV/Excel (Integration Service)
csv = "1.3"
calamine = "0.22"
rust_xlsxwriter = "0.60"

# Scheduling (Platform Service — Enterprise Job Scheduler)
cron = "0.12"

# Regular Expressions (Trade Compliance screening, Configurator rules)
regex = "1.10"

# GraphQL (Report Service — optional)
async-graphql = { version = "7.0", optional = true }
async-graphql-axum = { version = "7.0", optional = true }

# Privacy / PII Detection (Platform Service — optional)
regex-fancy = { version = "0.5", optional = true }

# Intelligent Document Processing (Platform Service — optional)
tract-onnx = { version = "0.21", optional = true }

# Testing
tokio-test = "0.4"
fake = { version = "2.9", features = ["derive"] }
```

### Dev Dependencies

```toml
[dev-dependencies]
mockall = "0.11"
testcontainers = "0.15"
testcontainers-modules = { version = "0.3", features = ["postgres", "redis", "rabbitmq"] }
pact_consumer = "1.2"
pact_provider = "1.2"
proptest = "1.4"
criterion = { version = "0.5", features = ["html_reports"] }
```

## 5. Workspace Dependency Management

All workspace crates share dependency versions via Cargo workspace inheritance to prevent version drift:

```toml
# Root Cargo.toml
[workspace]
members = ["crates/*", "services/*"]
resolver = "2"

[workspace.package]
edition = "2021"
rust-version = "1.83"

[workspace.dependencies]
serde = { version = "1.0", features = ["derive"] }
sea-orm = { version = "0.12", features = ["sqlx-postgres", "runtime-tokio-rustls"] }
# ... all shared dependencies listed here with exact versions
```

Each crate references workspace dependencies:

```toml
# crates/my-crate/Cargo.toml
[dependencies]
serde = { workspace = true }
sea-orm = { workspace = true }
```

**Rules:**
- All shared dependencies MUST be declared in the workspace `[workspace.dependencies]` section.
- Individual crates MUST NOT pin different versions of workspace-managed dependencies.
- New dependency additions require workspace-level review (dependabot PR).

## 6. CI / Security Tooling

| Tool | Purpose | When |
|------|---------|------|
| Trivy | Container and filesystem security scanning | Every PR |
| Snyk | Dependency vulnerability scanning | Every PR |
| `cargo audit` | Rust crate vulnerability database check | Weekly (CI) + Every PR |
| `cargo clippy` | Linting (`-- -D warnings`) | Every commit |
| `cargo fmt` | Code formatting check | Every commit |
| `cargo tarpaulin` | Code coverage reporting | Every PR |
| `k6` | Load testing | Weekly + pre-release |
| `cargo deny` | License compliance and ban list checking | Every PR |
| Cosign | Container image signing for supply chain verification | Every build |
| SAST | Static application security testing (Semgrep) | Every PR |
| DAST | Dynamic application security testing (OWASP ZAP) | Weekly + pre-release |
| Syft | Software Bill of Materials (SBOM) generation | Every build |

## 7. Version Policy

| Category | Policy |
|----------|--------|
| Rust Edition | 2021 |
| MSRV (Minimum Supported Rust Version) | 1.83 |
| Dependency Updates | Weekly automated PRs via Dependabot; critical security patches within 24 hours |
| Breaking Dependency Changes | Require team review and migration plan |
| Crate Compatibility | All workspace crates share the same Rust edition and MSRV |
| ML Model Versioning | Models versioned separately; backward-compatible inference via ONNX |
| Frontend (React/TypeScript) | Node.js 20 LTS; React 18.x; TypeScript strict mode enabled |

## 8. Container Images

| Aspect | Policy |
|--------|--------|
| Base Image | `rust:1.83-slim-bookworm` (build), `debian:bookworm-slim` (runtime) |
| Alternative Runtime | `gcr.io/distroless/cc-debian12` (for minimal attack surface; no shell) |
| Image Size Target | < 50MB per service (compressed) |
| Multi-stage Build | Yes — builder stage compiles, runtime stage copies binary only |
| Registry | GitHub Container Registry (ghcr.io) |
| Image Scanning | Trivy scan on every build; block deployment on Critical/High CVEs |
| Image Signing | Cosign signatures for supply chain verification |
| SBOM | Software Bill of Materials generated per image (Syft) |
| Non-root User | All containers run as non-root user (`USER 1000:1000`) |
| Read-only Root FS | Root filesystem mounted read-only; writable emptyDir for `/tmp` |

### Multi-stage Dockerfile Template

```dockerfile
FROM rust:1.83-slim-bookworm AS builder
WORKDIR /app
COPY . .
RUN cargo build --release

FROM debian:bookworm-slim AS runtime
RUN apt-get update && apt-get install -y ca-certificates && rm -rf /var/lib/apt/lists/*
COPY --from=builder /app/target/release/my-service /usr/local/bin/
USER 1000:1000
EXPOSE 8080
ENTRYPOINT ["my-service"]
```

## 9. Performance Benchmarking

| Component | Tool | Benchmark Target |
|-----------|------|-----------------|
| API Handlers | Criterion | p99 < 5ms per handler (no I/O) |
| Database Queries | Criterion + testcontainers | p99 < 100ms for complex queries |
| Event Processing | Custom harness | p99 < 100ms end-to-end |
| Cache Operations | Criterion | p99 < 1ms |
| ML Inference | Custom harness | p99 < 500ms per prediction |
| IDP Extraction | Custom harness | < 10 seconds per page |

Benchmarks run automatically in CI on every PR to the `main` branch.

---

*See [Deployment](deployment.md) for container orchestration and CI/CD pipeline details.*
