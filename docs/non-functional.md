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

## 2. Availability

| Metric | Target | Measurement |
|--------|--------|-------------|
| Uptime SLA | 99.9% (8.76 hours/year downtime) | Prometheus uptime calculation |
| Data Durability | 99.999% (5 nines) | PostgreSQL synchronous replication + WAL archiving + cross-region backup |
| Recovery Time Objective (RTO) | < 1 hour (region failover) | Disaster recovery drill |
| Recovery Point Objective (RPO) | < 5 minutes | PostgreSQL replication lag monitoring |
| Deployment Downtime | Zero (rolling updates) | K8s rolling deployment strategy |
| Scheduled Maintenance | Zero downtime (rolling) | Blue-green deployment for major upgrades |

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

## 4. Compliance

| Standard | Applicability | Key Controls |
|----------|---------------|-------------|
| SOC 2 Type II | Required for all deployments | Audit logging, access controls, encryption, incident response |
| GDPR | Required for EU customers | Data erasure, data portability, consent management, DPA, PIA |
| HIPAA | Per-tenant (healthcare clients) | PHI encryption, access logging, BAAs, minimum necessary access |
| PCI DSS | Required when payment processing enabled | Tokenization, network segmentation, vulnerability scanning |
| SOX | Required for publicly traded customers | Financial audit trail, change management, access controls, SoD |
| ISO 27001 | Recommended | Information security management system |
| ESG Regulations | Per-tenant (reporting required) | Emissions data accuracy, audit trail, framework compliance (GRI, SASB, TCFD) |

## 5. Data Residency

| Requirement | Specification |
|-------------|---------------|
| Region Pinning | Tenant data can be pinned to a specific geographic region |
| Cross-Region Transfer | Data never leaves pinned region except with explicit tenant consent |
| Data Classification | All data tagged with residency requirement at record level |
| Backup Compliance | Backups stored in same region as primary data |
| Enforcement | Enforced at connection routing layer and RLS policies |

## 6. Testing Strategy

### 6.1 Testing Pyramid

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

### 6.2 Test Categories

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
| AI/ML Tests | Model accuracy, bias, drift | Custom harness | N/A | On model update |

### 6.3 Testing Tools

| Tool | Purpose |
|------|---------|
| `cargo test` | Unit and integration tests |
| `testcontainers-rs` | Spin up PostgreSQL, Redis, RabbitMQ for integration tests |
| `fake` | Generate realistic test data |
| `pact_consumer` / `pact_provider` | Contract testing |
| `tower::ServiceExt` | Test Axum handlers without HTTP server |
| `mockall` | Mock traits for unit testing |
| Docker Compose | E2E test environment |
| `k6` | Load testing |
| Trivy | Container security scanning |
| `cargo audit` | Dependency vulnerability checking |
| OWASP ZAP | Dynamic application security testing |
| axe-core / Pa11y | Accessibility testing |

### 6.4 Test Environment Strategy

| Environment | Purpose | Data | Refresh |
|-------------|---------|------|---------|
| Unit (in-memory) | Fast feedback, no external deps | Mocked | Per test |
| Integration (testcontainers) | Service + real DB/cache/MQ | Seeded per test | Per test |
| E2E (Docker Compose) | Full system validation | Seeded once per run | Per pipeline |
| Staging | Pre-production validation | Anonymized production snapshot | Weekly |
| Production | Smoke tests only | Real data (read-only) | N/A |

### 6.5 Test Data Management

- Unit tests use `fake` crate for random, realistic data.
- Integration tests use idempotent seed scripts (`INSERT ... ON CONFLICT DO NOTHING`).
- E2E tests use a dedicated test tenant with known data.
- Test data MUST NOT contain real PII.
- Each test is responsible for its own setup and teardown (no shared mutable state).
- Staging environment uses anonymized production snapshots (PII replaced with synthetic data via Platform Service data masking).

### 6.6 Quality Gates

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

## 7. Monitoring & Observability

### 7.1 Metrics (Prometheus)

All services MUST expose a `/metrics` endpoint (Prometheus format) with the following metric categories:

| Category | Metrics |
|----------|---------|
| HTTP | Request rate, latency (p50, p95, p99), error rate, active requests |
| Business | Orders created, invoices generated, stock alerts, pipeline conversions, emissions recorded |
| Database | Query latency, connection pool usage, replication lag |
| Messaging | Publish rate, consume rate, consumer lag, DLQ depth |
| Cache | Hit rate, miss rate, eviction rate, memory usage |
| Infrastructure | CPU, memory, disk, network (via node exporter) |
| ML/AI | Inference latency, prediction confidence, model drift, feature store freshness |

### 7.2 Standard Service Metrics

All services MUST expose these Prometheus metrics:

| Metric | Type | Labels | Description |
|--------|------|--------|-------------|
| `http_requests_total` | Counter | method, path, status | Total HTTP requests |
| `http_request_duration_seconds` | Histogram | method, path | Request latency |
| `http_active_requests` | Gauge | — | Currently in-flight requests |
| `db_query_duration_seconds` | Histogram | query_name | Database query latency |
| `db_pool_active_connections` | Gauge | — | Active DB connections |
| `db_pool_idle_connections` | Gauge | — | Idle DB connections |
| `event_published_total` | Counter | event_type, status | Events published |
| `event_consumed_total` | Counter | event_type, status | Events consumed |
| `event_processing_duration_seconds` | Histogram | event_type | Event processing latency |
| `outbox_pending_count` | Gauge | — | Pending outbox entries |
| `cache_hit_total` | Counter | key_pattern | Cache hits |
| `cache_miss_total` | Counter | key_pattern | Cache misses |
| `saga_started_total` | Counter | saga_type | Sagas started |
| `saga_completed_total` | Counter | saga_type, status | Sagas completed (success/failed/compensated) |
| `assistant_intent_total` | Counter | intent, status | Digital assistant intent resolution |
| `assistant_action_duration_seconds` | Histogram | action | Digital assistant action execution time |
| `ml_inference_duration_seconds` | Histogram | model_name | ML inference latency |
| `esg_emissions_recorded_total` | Counter | scope, unit | Emissions data points recorded |

### 7.3 Dashboards (Grafana)

| Dashboard | Audience | Key Panels |
|-----------|----------|-----------|
| System Overview | SRE, DevOps | All services health, throughput, error rates, pod counts |
| Service Health | Developers | Per-service HTTP metrics, DB connections, event lag |
| Business KPIs | Business users | Orders, invoices, revenue, pipeline, headcount |
| Database Performance | DBAs | Query latency, connection pools, replication lag, slow queries |
| Message Queue Status | SRE | Queue depth, consumer lag, DLQ count, publish rate |
| Cache Performance | Developers | Hit/miss ratio, eviction rate, memory, key count |
| Security | Security team | Login failures, auth errors, rate limit hits, audit events, SoD conflicts |
| ESG / Sustainability | ESG analysts | Emissions by scope, carbon intensity, target progress, offset tracking |
| AI/ML Operations | Data scientists | Model accuracy, inference latency, feature drift, prediction volume |
| Cost | Finance | Resource utilization, pod counts, storage usage, data lake size |
| Digital Assistant | Product team | Intent distribution, success rate, response time, user satisfaction |

### 7.4 Structured Logging

All services MUST emit structured JSON logs with these fields:

```json
{
  "timestamp": "2026-03-27T10:30:00Z",
  "level": "INFO",
  "service": "commerce-service",
  "tenant_id": "tenant-uuid",
  "request_id": "request-uuid",
  "trace_id": "trace-uuid",
  "span_id": "span-uuid",
  "message": "Order created successfully",
  "order_id": "order-uuid",
  "customer_id": "customer-uuid"
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `timestamp` | Yes | ISO 8601 with timezone |
| `level` | Yes | TRACE, DEBUG, INFO, WARN, ERROR |
| `service` | Yes | Service name |
| `tenant_id` | Conditional | Tenant context — present for tenant-scoped requests, omitted for system-level operations |
| `request_id` | Conditional | From `X-Request-ID` header — omitted for background tasks and event handlers |
| `trace_id` | Yes | From OpenTelemetry context |
| `span_id` | Yes | From OpenTelemetry context |
| `message` | Yes | Human-readable log message |

**Log Redaction:** PII fields (email, phone, SSN) are automatically redacted in logs using the `mserp-core` redaction utility.

### 7.5 Distributed Tracing

All services participate in distributed tracing via OpenTelemetry + Jaeger:

- Every HTTP request creates a root span.
- Service-to-service HTTP calls propagate `trace_id` via `traceparent` header.
- Event publishing creates a child span with event metadata.
- Event consumption creates a linked span using `correlation_id`.
- Database queries are instrumented as child spans.
- Cache operations are instrumented as child spans.
- ML inference calls are instrumented as child spans with model metadata.

### 7.6 Alerts

| Alert | Severity | Response Time | Response |
|-------|----------|---------------|----------|
| Service Down (all replicas) | Critical | < 5 minutes | Investigate, restart, escalate |
| High Error Rate (>1% of requests) | Critical | < 5 minutes | Investigate, rollback if recent deploy |
| High Latency (p99 > 2s) | Warning | < 15 minutes | Investigate, check DB/cache |
| Disk Usage > 80% | Warning | < 1 hour | Investigate, clean up, expand |
| Disk Usage > 90% | Critical | < 15 minutes | Immediate action to free space |
| Certificate Expiry < 14 days | Warning | < 24 hours | Trigger cert rotation |
| DLQ Depth > 100 | Warning | < 1 hour | Investigate failed messages |
| RabbitMQ Consumer Lag > 1000 | Warning | < 15 minutes | Scale consumers, investigate bottleneck |
| Database Replication Lag > 30s | Warning | < 15 minutes | Check network, primary load |
| Pod Restart Loop | Critical | < 5 minutes | Check logs, resource limits |
| Saga Compensation Failed | Critical | < 5 minutes | Manual intervention, check DLQ |
| Tenant Quota Exceeded | Info | < 24 hours | Notify tenant admin |
| SoD Conflict Detected | Warning | < 1 hour | Review and mitigate |
| ML Model Drift Detected | Warning | < 24 hours | Evaluate retraining, review accuracy |
| ESG Data Anomaly | Info | < 48 hours | Review emissions data, verify calculations |

### 7.7 Alert Routing

| Severity | Channel | Escalation |
|----------|---------|-----------|
| Critical | PagerDuty + Slack #incidents | On-call engineer → Engineering lead → CTO |
| Warning | Slack #alerts | On-call engineer acknowledges |
| Info | Slack #notifications | No action required |

## 8. Sustainability / Green IT

| Metric | Target | Measurement |
|--------|--------|-------------|
| Carbon efficiency | Track CO2 per API request | Cloud provider carbon metrics + estimated compute carbon |
| Resource utilization | > 60% average CPU utilization across cluster | Prometheus metrics |
| Idle resource cleanup | Automatic scale-to-zero for non-production | K8s HPA minReplicas=0 for dev/staging |
| Efficient languages | Rust for minimal memory/CPU footprint | Compared to equivalent JVM/Node workloads |

---

*See [Deployment](deployment.md) for Kubernetes and infrastructure monitoring.*
*See [Data Architecture](data-architecture.md) for database observability.*
