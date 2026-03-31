# Non-Functional Requirements

Performance targets are defined in SPEC.md §17.1.

Availability targets are defined in SPEC.md §17.2.

Scalability targets are defined in SPEC.md §17.3.

Rate limiting is defined in SPEC.md §10.3.

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

Data retention policies are defined in SPEC.md §9.8. The table below provides storage tier and disposal method implementation details.

| Data Category | Storage Tier | Disposal Method |
|---------------|-------------|-----------------|
| Business data (orders, invoices) | PostgreSQL (hot) → S3 (cold) | N/A |
| Audit logs | Loki (hot, 90 days) → S3 (cold) | N/A |
| ESG / emissions | PostgreSQL → Data Lake | N/A |
| User activity logs | Loki (hot, 30 days) → S3 (cold) | Secure deletion |
| PII (customer/employee) | PostgreSQL | Anonymization or deletion per GDPR/contract |
| System metrics | Prometheus (hot, 90 days) → Thanos/S3 (cold) | Automatic expiry |
| Application logs | Loki → S3 | Automatic expiry |
| Sessions, tokens | Redis TTL | Automatic expiry |
| Data Lake raw zone | MinIO | Lifecycle policies |
| ML training data | MinIO | Secure deletion |

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

Quality gates per phase are defined in SPEC.md §18.3.

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
