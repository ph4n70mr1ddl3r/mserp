# Technology Stack

Core technologies are defined in SPEC.md §2. This document provides crate versions, CI tooling, and container policies.

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

Crate dependency rules are defined in SPEC.md §2.1.

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

Container rules are defined in SPEC.md §2.2.

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
