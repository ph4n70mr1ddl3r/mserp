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
| IoT Telemetry Ingestion | < 100ms (event to storage) | Platform Service metrics |
| Digital Twin State Sync | < 500ms (telemetry to twin update) | Manufacturing Service metrics |
| RPA Bot Execution Startup | < 3 seconds | Platform Service metrics |
| Process Mining Analysis | < 60 seconds (standard process) | Report Service metrics |
| CDP Identity Resolution | < 2 seconds (single profile) | CRM Service metrics |
| B2B Portal Page Load | < 3 seconds (product catalog) | Commerce Service metrics |
| Reconciliation Auto-Match | < 30 seconds (per account) | Finance Service metrics |
| Profitability Calculation | < 15 seconds (full dimension scan) | Finance Service metrics |

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
| Per-tenant API | 1,000 requests/second (configurable per plan) | API Gateway (Traefik/Kong) |
| Per-user API | 100 requests/second | API Gateway |
| Authentication attempts | 10 failed attempts per minute per IP | Auth Service |
| Report generation | 5 concurrent reports per tenant | Report Service semaphore |
| File uploads | 50 MB per file, 500 MB per tenant per day | Platform Service |
| Search queries | 30 requests/second per tenant | Elasticsearch throttling |
| Webhook deliveries | 100 deliveries/second per tenant (outbound) | Integration Service |
| Bulk import | 10,000 rows per request, 1 concurrent import per tenant | Integration Service |

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
| Financial transactions | 7 years (regulatory) | PostgreSQL (hot) → S3 (cold) | Secure deletion after retention period |
| Audit logs | 7 years (SOC 2 / SOX) | Loki (hot, 90 days) → S3 (cold) | Secure deletion |
| User activity logs | 1 year | Loki (hot, 30 days) → S3 (cold) | Secure deletion |
| PII (customer/employee) | Duration of contract + 30 days | PostgreSQL | Anonymization or deletion per GDPR/contract |
| Emissions data | 10 years (regulatory) | PostgreSQL → Data Lake | Anonymization |
| System metrics | 13 months | Prometheus (hot, 90 days) → Thanos/S3 (cold) | Automatic expiry |
| Application logs | 90 days (hot), 1 year (cold) | Loki → S3 | Automatic expiry |
| Session data | 24 hours | Redis TTL | Automatic expiry |
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

## 10. Monitoring & Observability

### 10.1 Metrics (Prometheus)

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

### 10.2 Standard Service Metrics

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
| `scheduler_job_duration_seconds` | Histogram | job_type | Scheduled job execution duration |
| `scheduler_job_active_count` | Gauge | job_type | Currently running scheduled jobs |
| `subscription_billing_cycle_total` | Counter | status | Subscription billing cycle results |
| `credit_check_total` | Counter | result | Credit check outcomes (pass/hold/release) |
| `trade_compliance_screening_total` | Counter | result | Trade compliance screening outcomes |
| `knowledge_search_total` | Counter | result | Knowledge base search results (hit/miss) |
| `iot_telemetry_ingested_total` | Counter | device_id, telemetry_type | IoT telemetry events ingested |
| `iot_device_alert_total` | Counter | device_id, alert_type | IoT alert rule triggers |
| `digital_twin_sync_duration_seconds` | Histogram | asset_id | Digital twin state sync latency |
| `rpa_bot_execution_total` | Counter | bot_id, status | RPA bot execution outcomes |
| `rpa_bot_execution_duration_seconds` | Histogram | bot_id | RPA bot execution duration |
| `process_mining_analysis_duration_seconds` | Histogram | process_type | Process mining analysis duration |
| `cdp_identity_resolution_total` | Counter | result | CDP identity resolution outcomes |
| `collaboration_message_total` | Counter | channel_id | Collaboration messages posted |
| `logistics_tracking_update_total` | Counter | carrier, status | Logistics tracking updates received |
| `rate_limit_exceeded_total` | Counter | tenant_id, endpoint | Rate limit violations |

### 10.3 Dashboards (Grafana)

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
| Subscription Health | Product, Finance | MRR/ARR, churn rate, renewal rate, billing success rate |
| Credit & Collections | Finance, Sales | Credit exposure, aging buckets, collection effectiveness index |
| Trade Compliance | Legal, Operations | Screening volumes, match rates, license status, shipment holds |
| Knowledge Base | Support, Operations | Article views, search effectiveness, resolution rate, content gaps |
| SLI/SLO Dashboard | SRE, Engineering | Error budget burn rate, SLO compliance, alert history |
| IoT & Digital Twin | Operations, Manufacturing | Device connectivity, telemetry throughput, alert rates, twin sync latency |
| RPA Operations | Operations, IT | Bot execution rates, success rates, execution duration, error distribution |
| Process Performance | Process owners, Operations | Process cycle times, conformance rates, bottleneck heatmaps |
| Supplier Risk | Procurement, Finance | Risk score distribution, alert volumes, mitigation plan status |
| Customer Intelligence | Marketing, Sales | Profile completeness, segment distribution, journey completion, engagement scores |
| B2B Portal | Commerce, Sales | Portal adoption, order volume, reorder rates, customer satisfaction |
| Collaboration | IT, Operations | Message volume, active channels, task completion rates |
| Financial Close | Finance | Close task progress, reconciliation match rates, close cycle duration |

### 10.4 Structured Logging

All services MUST emit structured JSON logs with these fields:

```json
{
  "timestamp": "2026-03-28T10:30:00Z",
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

**Log Levels Policy:**
- `TRACE`: Verbose internal state (disabled in production)
- `DEBUG`: Diagnostic information (disabled in production by default)
- `INFO`: Business events (order created, invoice generated)
- `WARN`: Recoverable issues (cache miss fallback, retry triggered)
- `ERROR`: Failures requiring attention (database error, external service failure)

### 10.5 Distributed Tracing

All services participate in distributed tracing via OpenTelemetry + Jaeger:

- Every HTTP request creates a root span.
- Service-to-service HTTP calls propagate `trace_id` via `traceparent` header.
- Event publishing creates a child span with event metadata.
- Event consumption creates a linked span using `correlation_id`.
- Database queries are instrumented as child spans.
- Cache operations are instrumented as child spans.
- ML inference calls are instrumented as child spans with model metadata.

**Trace Sampling:**
- Production: 10% default sampling rate; 100% for error traces and slow requests (> p99 latency).
- Staging: 100% sampling.
- Trace retention: 7 days (hot), 30 days (cold storage).

### 10.6 Alerts

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
| Trade Compliance Screening Failure | Critical | < 5 minutes | Halt affected orders, investigate screening service |
| Subscription Billing Failure | Critical | < 15 minutes | Investigate, manual billing fallback |
| Revenue Recognition Discrepancy | Warning | < 1 hour | Finance team review, check recognition rules |
| Scheduler Job Overdue | Warning | < 30 minutes | Investigate scheduler service, check job queue |
| Error Budget Exhausted | Warning | < 4 hours | Freeze feature work, prioritize reliability |
| Backup Verification Failed | Critical | < 1 hour | Investigate backup infrastructure, re-run verification |
| IoT Device Offline (>10 devices) | Warning | < 30 minutes | Investigate device connectivity, check gateway |
| RPA Bot Failure Rate > 10% | Warning | < 30 minutes | Investigate failing bots, check target systems |
| Process Conformance < 90% | Warning | < 4 hours | Review process deviations, update workflow definitions |
| CDP Identity Resolution Backlog > 1000 | Warning | < 1 hour | Scale CDP processing, check data source connectivity |
| Supplier Risk Score Critical | Critical | < 1 hour | Review supplier risk detail, notify procurement, initiate mitigation |
| Digital Twin Sync Lag > 5s | Warning | < 15 minutes | Check telemetry pipeline, investigate device health |
| Financial Close Task Overdue | Warning | < 1 hour | Notify task owner, escalate to close manager |

### 10.7 Alert Routing

| Severity | Channel | Escalation |
|----------|---------|-----------|
| Critical | PagerDuty + Slack #incidents | On-call engineer → Engineering lead → CTO |
| Warning | Slack #alerts | On-call engineer acknowledges |
| Info | Slack #notifications | No action required |

## 11. Sustainability / Green IT

| Metric | Target | Measurement |
|--------|--------|-------------|
| Carbon efficiency | Track CO2 per API request | Cloud provider carbon metrics + estimated compute carbon |
| Resource utilization | > 60% average CPU utilization across cluster | Prometheus metrics |
| Idle resource cleanup | Automatic scale-to-zero for non-production | K8s HPA minReplicas=0 for dev/staging |
| Efficient languages | Rust for minimal memory/CPU footprint | Compared to equivalent JVM/Node workloads |
| Image size | < 50MB per service (compressed) | Container registry metrics |
| Cold start time | < 5 seconds for auto-scaled pods | Pod startup duration metric |

---

*See [Deployment](deployment.md) for Kubernetes and infrastructure monitoring.*
*See [Data Architecture](data-architecture.md) for database observability.*
