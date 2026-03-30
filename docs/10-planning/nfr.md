# Non-Functional Requirements

## 1. Performance

| Metric | Target | Measurement |
|--------|--------|-------------|
| API Response Time (p50) | < 50ms | Prometheus histogram `http_request_duration_seconds` |
| API Response Time (p99) | < 500ms | Prometheus histogram `http_request_duration_seconds` |
| API Response Time (p99.9) | < 2s | Prometheus histogram `http_request_duration_seconds` |
| Throughput | 10,000 requests/second | k6 load test |
| Event Processing Latency | < 100ms (publish to consume) | Jaeger trace `publish → consume` span |
| Report Generation | < 30 seconds (standard) | k6 timed request |
| Dashboard Load | < 5 seconds (initial) | Frontend performance monitoring |
| Health Check Response | < 100ms | K8s probe metrics |
| Database Query (simple) | < 10ms | Prometheus `db_query_duration_seconds` p99 |
| Database Query (complex) | < 100ms | Prometheus `db_query_duration_seconds` p99 |
| Cache Hit Rate | > 90% for read-heavy endpoints | `cache_hit_total / (cache_hit_total + cache_miss_total)` |
| Digital Assistant Response | < 3 seconds (intent to answer) | Platform Service metrics |
| ML Inference Latency | < 500ms (single prediction) | ONNX Runtime metrics |
| Mobile API Response (p99) | < 800ms (optimized endpoints) | Mobile performance monitoring |
| Full-Text Search | < 200ms | Elasticsearch query latency |
| ATP Check Response | < 200ms | Commerce Service metrics |
| Product Configuration | < 1 second | Commerce Service metrics |
| Credit Check | < 100ms | Commerce Service metrics |
| Trade Compliance Screening | < 2 seconds per entity | Integration Service metrics |
| Knowledge Search | < 300ms | Platform Service metrics |
| Subscription Billing Cycle | < 5 seconds per subscription | Commerce Service metrics |
| Revenue Recognition Calculation | < 10 seconds per contract | Finance Service metrics |
| Survey Response Processing | < 500ms | CRM Service metrics |
| IoT Telemetry Ingestion | < 100ms (event to storage) | Manufacturing Service metrics |
| Digital Twin State Sync | < 500ms (telemetry to twin update) | Manufacturing Service metrics |
| RPA Bot Execution Startup | < 3 seconds | Platform Service metrics |
| Process Mining Analysis | < 60 seconds (standard process) | Report Service metrics |
| CDP Identity Resolution | < 2 seconds (single profile) | CRM Service metrics |
| B2B Portal Page Load | < 3 seconds (product catalog) | Commerce Service metrics |
| Reconciliation Auto-Match | < 30 seconds (per account) | Finance Service metrics |
| Profitability Calculation | < 15 seconds (full dimension scan) | Finance Service metrics |
| Lease Calculation | < 5 seconds per contract | Finance Service metrics |
| Grant Revenue Recognition | < 10 seconds per grant | Finance Service metrics |
| Joint Venture Allocation | < 10 seconds per venture | Finance Service metrics |
| Intelligent Close Anomaly Detection | < 30 seconds per period scan | Finance Service metrics |
| Cash Application Matching | < 2 seconds per receipt | Finance Service metrics |
| IDP Document Extraction | < 10 seconds per page | Platform Service metrics |
| Warranty Claim Validation | < 500ms | Commerce Service metrics |
| Loyalty Points Accrual/Redemption | < 200ms | Commerce Service metrics |
| Omnichannel Order Routing | < 500ms | Commerce Service metrics |
| Price Optimization Simulation | < 30 seconds | Commerce Service metrics |
| Tax Calculation | < 200ms per transaction | Finance Service metrics |
| Commodity Price Lookup | < 100ms | Finance Service metrics |
| Spend Classification (ML) | < 5 seconds per batch | Finance Service metrics |
| Supplier Diversity Report | < 10 seconds | Finance Service metrics |
| EHS Incident Reporting | < 500ms | Manufacturing Service metrics |
| MRO Repair Order Creation | < 300ms | Manufacturing Service metrics |
| Product Compliance Check | < 1 second | Manufacturing Service metrics |
| Contact Center Interaction | < 500ms (API response) | CRM Service metrics |
| Social Profile Enrichment | < 3 seconds | CRM Service metrics |
| A/B Test Statistical Analysis | < 10 seconds | CRM Service metrics |
| Compliance Hub Dashboard Load | < 3 seconds | Platform Service metrics |
| Blockchain Record Anchoring | < 30 seconds | Integration Service metrics |
| Digital Twin Simulation | < 60 seconds | Manufacturing Service metrics |

## 2. Availability

| Metric | Target | Measurement |
|--------|--------|-------------|
| Uptime SLA | 99.9% (8.76 hours/year downtime) | Prometheus uptime calculation |
| Data Durability | 99.999% (5 nines) | PostgreSQL synchronous replication + WAL archiving + cross-region backup |
| Recovery Time Objective (RTO) | < 1 hour (region failover) | Disaster recovery drill |
| Recovery Point Objective (RPO) | < 5 minutes | PostgreSQL replication lag monitoring |
| Deployment Downtime | Zero (rolling updates) | K8s rolling deployment strategy |
| Scheduled Maintenance | Zero downtime (rolling) | Blue-green deployment for major upgrades |

### 2.1 SLI / SLO Error Budget Framework

| SLI | SLO | Error Budget (30-day window) | Calculation |
|-----|-----|------------------------------|-------------|
| API Availability | 99.9% | 43.2 minutes/month | `1 - (successful_requests / total_requests)` |
| Event Processing Reliability | 99.95% | 21.6 minutes/month | `1 - (successfully_consumed / total_published)` |
| Report Generation Success | 99.5% | 3.6 hours/month | `1 - (completed_reports / total_report_requests)` |
| Search Availability | 99.9% | 43.2 minutes/month | `1 - (successful_searches / total_searches)` |

**Error Budget Policy:**
- When error budget is exhausted for a given SLI, all non-critical feature work stops until the SLO is restored.
- Error budgets reset at the beginning of each calendar month.
- SLO violations trigger a post-incident review within 48 hours.

### 2.2 SLA Calculation Methodology

SLA percentages are calculated monthly based on calendar availability:

```
SLA = ((total_minutes_in_month - downtime_minutes) / total_minutes_in_month) × 100
```

Where:
- `total_minutes_in_month` = number of minutes in the calendar month
- `downtime_minutes` = sum of all unplanned downtime minutes
- Planned maintenance windows are excluded if published via Config Service at least 72 hours in advance
- Downtime is measured at the API Gateway level (5xx error rate > 50% for > 1 consecutive minute)

## 3. Scalability

| Requirement | Specification |
|-------------|---------------|
| Horizontal Scaling | All services auto-scale via HPA (CPU + memory triggers) |
| Database Scaling | Read replicas for read-heavy services, connection pooling (PgBouncer) |
| Multi-Region | Active-passive failover with < 1 hour RTO |
| Tenant Scaling | Support up to 1,000 tenants per cluster |
| Concurrent Users | 1,000+ concurrent active users per cluster |
| Data Volume | Up to 100 GB per tenant, 10 TB per cluster |
| Data Lake Scaling | Unlimited raw zone storage per tenant |
| ML Model Serving | Concurrent inference requests scale horizontally via Report Service HPA |

## 4. Rate Limiting

| Scope | Limit | Enforcement |
|-------|-------|-------------|
| Per-tenant API (global) | 10,000/min | API Gateway (Traefik/Kong) |
| Per-user API | 1,000/min | API Gateway |
| Auth endpoints | 10/min | Auth Service |
| File uploads | 50/min | Platform Service |
| Report generation | 10/min | Report Service semaphore |
| Bulk operations | 20/min | Integration Service |
| AI/ML inference | 30/min | Report Service |
| Digital assistant | 60/min | Platform Service |

**Rate limit responses MUST use HTTP 429 with `Retry-After` header and rate-limit metadata in response headers (`X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`).**

## 5. Compliance

| Standard | Applicability | Key Controls |
|----------|---------------|-------------|
| SOC 2 Type II | Required for all deployments | Audit logging, access controls, encryption, incident response |
| GDPR | Required for EU customers | Data erasure, data portability, consent management, DPA, PIA |
| HIPAA | Per-tenant (healthcare clients) | PHI encryption, access logging, BAAs, minimum necessary access |
| PCI DSS | Required when payment processing enabled | Tokenization, network segmentation, vulnerability scanning |
| SOX | Required for publicly traded customers | Financial audit trail, change management, access controls, SoD |
| ISO 27001 | Recommended | Information security management system |
| ESG Regulations | Per-tenant (reporting required) | Emissions data accuracy, audit trail, framework compliance (GRI, SASB, TCFD) |

## 6. Data Residency

| Requirement | Specification |
|-------------|---------------|
| Region Pinning | Tenant data can be pinned to a specific geographic region |
| Cross-Region Transfer | Data never leaves pinned region except with explicit tenant consent |
| Data Classification | All data tagged with residency requirement at record level |
| Backup Compliance | Backups stored in same region as primary data |
| Enforcement | Enforced at connection routing layer and RLS policies |

## 7. Data Retention

| Data Category | Retention Period | Storage Tier | Disposal Method |
|---------------|-----------------|-------------|-----------------|
| Business data (orders, invoices) | Indefinite | PostgreSQL (hot) → S3 (cold) | N/A |
| Audit logs | Indefinite (compliance) | Loki (hot, 90 days) → S3 (cold) | N/A |
| ESG / emissions | Indefinite (regulatory) | PostgreSQL → Data Lake | N/A |
| User activity logs | 1 year | Loki (hot, 30 days) → S3 (cold) | Secure deletion |
| PII (customer/employee) | Duration of contract + 30 days | PostgreSQL | Anonymization or deletion per GDPR/contract |
| System metrics | 13 months | Prometheus (hot, 90 days) → Thanos/S3 (cold) | Automatic expiry |
| Application logs | 90 days (hot), 1 year (cold) | Loki → S3 | Automatic expiry |
| Sessions, tokens | 90 days | Redis TTL | Automatic expiry |
| Data Lake raw zone | Per-tenant configuration | MinIO | Lifecycle policies |
| ML training data | 2 years | MinIO | Secure deletion |

**Tenant administrators can configure per-category retention periods that exceed (but not fall below) the minimums above.**

## 8. Internationalization & Localization

| Requirement | Specification |
|-------------|---------------|
| Supported Languages | English (default), with framework for additional languages via resource bundles |
| Time Zones | All timestamps stored in UTC; display converted to user's preferred timezone |
| Currency | Multi-currency support with configurable exchange rates; amounts stored as decimal with currency code |
| Date/Time Format | Respects user locale settings (`chrono-tz`) |
| Number Format | Locale-aware formatting (decimal separators, grouping) |
| Character Encoding | UTF-8 everywhere (database, API, logs) |
| Text Direction | Framework supports LTR and RTL layouts for future expansion |
| Translation API | Platform Service exposes translation endpoint for dynamic content |

## 9. Testing Strategy

### 9.1 Testing Pyramid

```
           ┌───────────┐
           │   E2E     │   ← Few, slow, high confidence
           │  Tests    │
          ┌┴───────────┴┐
          │Integration  │   ← Moderate count, moderate speed
          │   Tests     │
         ┌┴─────────────┴┐
         │  Contract      │   ← Per-service, CI-enforced
         │   Tests        │
        ┌┴───────────────┴┐
        │   Unit Tests     │   ← Many, fast, isolated
        └─────────────────┘
```

### 9.2 Test Categories

| Category | Scope | Tooling | Coverage Target | When |
|----------|-------|---------|-----------------|------|
| Unit Tests | Single function/module | `cargo test` | >= 80% | Every commit |
| Integration Tests | Service + database + cache | `testcontainers-rs`, `cargo test` | Key flows | Every PR |
| Contract Tests | API/event contracts | `pact_consumer`, `pact_provider` | All public APIs | Every PR |
| E2E Tests | Full system (multi-service) | Custom harness + Docker Compose | Critical paths | Nightly + pre-release |
| Saga Tests | Distributed transaction flows with failure injection | Custom harness + RabbitMQ | All sagas | Every PR |
| Load Tests | Performance under load | `k6` | N/A | Weekly + pre-release |
| Security Tests | Vulnerability scanning | Trivy, `cargo audit`, Snyk, OWASP ZAP | N/A | Every PR + weekly |
| Chaos Tests | Resilience under failure | Chaos mesh / Litmus | Critical scenarios | Monthly |
| Accessibility Tests | WCAG 2.1 AA compliance | axe-core, Pa11y | All web pages | Every PR |
| AI/ML Tests | Model accuracy, bias, drift, fairness | Custom harness | N/A | On model update + quarterly |

### 9.3 Testing Tools

| Tool | Purpose |
|------|---------|
| `cargo test` | Unit and integration tests |
| `testcontainers-rs` | Spin up PostgreSQL, Redis, RabbitMQ for integration tests |
| `fake` | Generate realistic test data |
| `pact_consumer` / `pact_provider` | Contract testing |
| `tower::ServiceExt` | Test Axum handlers without HTTP server |
| `mockall` | Mock traits for unit testing |
| `proptest` | Property-based testing for edge cases |
| Docker Compose | E2E test environment |
| `k6` | Load testing |
| Trivy | Container security scanning |
| `cargo audit` | Dependency vulnerability checking |
| OWASP ZAP | Dynamic application security testing |
| axe-core / Pa11y | Accessibility testing |

### 9.4 Test Environment Strategy

| Environment | Purpose | Data | Refresh |
|-------------|---------|------|---------|
| Unit (in-memory) | Fast feedback, no external deps | Mocked | Per test |
| Integration (testcontainers) | Service + real DB/cache/MQ | Seeded per test | Per test |
| E2E (Docker Compose) | Full system validation | Seeded once per run | Per pipeline |
| Staging | Pre-production validation | Anonymized production snapshot | Weekly |
| Production | Smoke tests only | Real data (read-only) | N/A |

### 9.5 Test Data Management

- Unit tests use `fake` crate for random, realistic data.
- Integration tests use idempotent seed scripts (`INSERT ... ON CONFLICT DO NOTHING`).
- E2E tests use a dedicated test tenant with known data.
- Test data MUST NOT contain real PII.
- Each test is responsible for its own setup and teardown (no shared mutable state).
- Staging environment uses anonymized production snapshots (PII replaced with synthetic data via Platform Service data masking).

### 9.6 Quality Gates

| Gate | Requirement | Enforced By |
|------|-------------|-------------|
| Unit test coverage | >= 80% per crate | CI (`cargo tarpaulin`) |
| Contract tests pass | All consumer contracts verified | CI (Pact broker) |
| No clippy warnings | `cargo clippy -- -D warnings` | CI |
| No unsafe code | `#![deny(unsafe_code)]` | Rust compiler |
| Format check | `cargo fmt --check` | CI |
| Dependency audit | `cargo audit` passes (no Critical/High) | CI (weekly + PR) |
| License compliance | `cargo deny` passes | CI (every PR) |
| Security scan | Trivy scan passes (no Critical CVEs) | CI (every build) |
| Accessibility | WCAG 2.1 AA compliance on web pages | CI (every PR) |
| SoD compliance | No unresolved SoD conflicts in production config | CI (pre-release) |

---

## 10. Sustainability / Green IT

| Metric | Target | Measurement |
|--------|--------|-------------|
| Carbon efficiency | Track CO2 per API request | Cloud provider carbon metrics + estimated compute carbon |
| Resource utilization | > 60% average CPU utilization across cluster | Prometheus metrics |
| Idle resource cleanup | Automatic scale-to-zero for non-production | K8s HPA minReplicas=0 for dev/staging |
| Efficient languages | Rust for minimal memory/CPU footprint | Compared to equivalent JVM/Node workloads |
| Image size | < 50MB per service (compressed) | Container registry metrics |
| Cold start time | < 5 seconds for auto-scaled pods | Pod startup duration metric |
