# Deployment Architecture

## 1. Kubernetes Deployment

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          Kubernetes Cluster                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                            Ingress Controller                          │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                      │                                       │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                         Namespace: mserp-system                        │  │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐          │  │
│  │  │ Traefik │ │ Cert-   │ │Prometheus│ │ Grafana │ │ Jaeger │          │  │
│  │  │         │ │ Manager │ │          │ │         │ │        │          │  │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘          │  │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐                                    │  │
│  │  │ Alert-  │ │  Loki   │ │ Vault / │                                    │  │
│  │  │ manager │ │         │ │  KMS    │                                    │  │
│  │  └─────────┘ └─────────┘ └─────────┘                                    │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                      │                                       │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                        Namespace: mserp-services                       │  │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐          │  │
│  │  │  Auth   │ │Identity │ │ Tenant  │ │ Config  │ │Commerce │          │  │
│  │  │(3 pods) │ │(3 pods) │ │(2 pods) │ │(2 pods) │ │(5 pods) │          │  │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘          │  │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐          │  │
│  │  │ Finance │ │   HR    │ │Manufact-│ │ Report  │ │Workflow │          │  │
│  │  │(5 pods) │ │(3 pods) │ │(3 pods) │ │(3 pods) │ │(3 pods) │          │  │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘          │  │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐                       │  │
│  │  │Platform │ │Integrat-│ │CRM/Mktg │ │ Project │                       │  │
│  │  │(3 pods) │ │(3 pods) │ │(3 pods) │ │(3 pods) │                       │  │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘                       │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                      │                                       │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                       Namespace: mserp-infra                           │  │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐          │  │
│  │  │PostgreSQL│ │RabbitMQ │ │  Redis  │ │Elastic  │ │  MinIO  │          │  │
│  │  │ Cluster │ │ Cluster │ │ Cluster │ │ Search  │ │         │          │  │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘          │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 2. Resource Requirements (Per Service)

| Service | CPU Request | CPU Limit | Memory Request | Memory Limit | Replicas |
|---------|-------------|-----------|----------------|--------------|----------|
| Auth | 100m | 500m | 128Mi | 512Mi | 3 |
| Identity | 100m | 500m | 128Mi | 512Mi | 3 |
| Tenant | 50m | 200m | 64Mi | 256Mi | 2 |
| Config | 50m | 200m | 64Mi | 256Mi | 2 |
| Commerce (Sales + Inventory) | 250m | 1000m | 256Mi | 1Gi | 5 |
| Finance (Finance + Procurement) | 250m | 1000m | 256Mi | 1Gi | 5 |
| HR | 100m | 500m | 128Mi | 512Mi | 3 |
| Manufacturing | 100m | 500m | 128Mi | 512Mi | 3 |
| Report | 500m | 2000m | 512Mi | 2Gi | 3 |
| Workflow | 100m | 500m | 128Mi | 512Mi | 3 |
| CRM / Marketing | 100m | 500m | 128Mi | 512Mi | 3 |
| Project Management | 100m | 500m | 128Mi | 512Mi | 3 |
| Platform (Notif + File + Audit) | 200m | 1000m | 256Mi | 1Gi | 3 |
| Integration | 100m | 500m | 128Mi | 512Mi | 3 |

**Total minimum resources (all services at minimum replicas):**
- CPU: ~3.3 cores requested, ~12.3 cores limit
- Memory: ~3.3 GiB requested, ~13.5 GiB limit
- Pods: 44

## 3. CI/CD Pipeline

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           CI/CD Pipeline                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐  │
│   │  Code   │───▶│  Build  │───▶│  Test   │───▶│ Security│───▶│  Deploy │  │
│   │  Push   │    │         │    │         │    │  Scan   │    │         │  │
│   └─────────┘    └─────────┘    └─────────┘    └─────────┘    └─────────┘  │
│        │              │              │              │              │        │
│        ▼              ▼              ▼              ▼              ▼        │
│   GitHub        Cargo build    Unit tests    Trivy          ArgoCD         │
│   Actions       Docker build   Integration   Clippy         Helm           │
│                               E2E tests     cargo-audit    Kustomize       │
│                               Contract      Snyk                          │
│                               tests (Pact)  cargo-deny                    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.1 Pipeline Stages

| Stage | Actions | Failure Policy |
|-------|---------|---------------|
| Build | `cargo build --release`, Docker image build | Block merge |
| Test | Unit, integration, contract (Pact), saga tests | Block merge |
| Security | Trivy scan, `cargo audit`, `cargo deny`, Snyk, clippy | Block merge on Critical/High |
| Deploy (Staging) | ArgoCD auto-sync to staging | Alert on failure |
| Deploy (Production) | Manual approval + ArgoCD sync | Rollback on health check failure |

### 3.2 Contract Testing

| Aspect | Details |
|--------|---------|
| Framework | Pact (Rust: `pact_consumer` + `pact_provider`) |
| Consumer Tests | Each service publishes its expected provider contract |
| Provider Verification | Provider services verify all consumer contracts on every PR |
| Event Contracts | Event schemas are versioned; breaking changes bump `event_version` |
| Enforcement | PR cannot merge if provider verification fails |

### 3.3 Branch Strategy

| Branch | Purpose | Deploy Target |
|--------|---------|--------------|
| `main` | Production-ready code | Production (via tag) |
| `develop` | Integration branch | Staging (auto-deploy) |
| `feature/*` | Feature branches | None (CI only) |
| `hotfix/*` | Emergency fixes | Production (expedited) |
| `release/*` | Release candidates | Staging + Production (manual) |

## 4. Health Probes (Kubernetes Configuration)

### 4.1 Liveness Probe

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: http
  initialDelaySeconds: 10
  periodSeconds: 15
  timeoutSeconds: 5
  failureThreshold: 3
```

### 4.2 Readiness Probe

```yaml
readinessProbe:
  httpGet:
    path: /readyz
    port: http
  initialDelaySeconds: 5
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3
```

### 4.3 Startup Probe (for services with slow initialization)

```yaml
startupProbe:
  httpGet:
    path: /readyz
    port: http
  initialDelaySeconds: 0
  periodSeconds: 5
  timeoutSeconds: 5
  failureThreshold: 30  # Up to 150 seconds to start
```

## 5. Graceful Shutdown

### 5.1 Shutdown Sequence

All services MUST handle `SIGTERM` gracefully to ensure no in-flight requests or events are lost.

```
1. Receive SIGTERM
2. Set "draining" state (stop accepting new connections from K8s)
3. Wait for in-flight HTTP requests to complete (max 30 seconds)
4. Stop consuming from RabbitMQ queues
5. Wait for in-flight event handlers to complete (max 15 seconds)
6. Flush outbox poller (publish any pending outbox events, max 10 seconds)
7. Close database connections
8. Close Redis connections
9. Close RabbitMQ connections
10. Exit with code 0
```

### 5.2 Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `shutdown.http_drain_timeout` | 30s | Max time to wait for in-flight HTTP requests |
| `shutdown.event_drain_timeout` | 15s | Max time to wait for in-flight event handlers |
| `shutdown.outbox_flush_timeout` | 10s | Max time to flush pending outbox events |
| `shutdown.total_timeout` | 60s | Hard deadline — process is killed after this |

### 5.3 Kubernetes Integration

```yaml
spec:
  terminationGracePeriodSeconds: 90  # Must exceed shutdown.total_timeout
  containers:
    - name: service
      lifecycle:
        preStop:
          exec:
            command: ["/bin/sh", "-c", "sleep 5"]  # Wait for pod removal from endpoints
```

### 5.4 Implementation Requirements

- Use `tokio::signal::unix::signal(SIGTERM)` for signal handling.
- Use `axum::serve()` with graceful shutdown via `tokio::select!`.
- Track in-flight requests with an `AtomicUsize` counter.
- Outbox poller MUST check the draining state and exit its loop.
- Connection pools (database, Redis, RabbitMQ) MUST be closed explicitly.

## 6. Multi-Region Deployment

| Aspect | Specification |
|--------|---------------|
| Strategy | Active-passive failover with warm standby |
| DNS | Primary region receives 100% of traffic; secondary receives no traffic during normal operation |
| Failover DNS | On primary failure, DNS updated to secondary (automated via health checks); RTO < 1 hour |
| Data Replication | PostgreSQL logical replication with synchronous standby |
| RabbitMQ | Federation plugin for cross-region queue mirroring |
| Redis | Redis Enterprise with active-passive replication |
| File Storage | MinIO site replication |

### 6.1 Disaster Recovery

| Scenario | Recovery Procedure | RTO | RPO |
|----------|-------------------|-----|-----|
| Single pod failure | K8s auto-restart | < 30s | 0 |
| Node failure | K8s reschedule pods | < 2 min | 0 |
| Database failover | PostgreSQL streaming replication failover | < 5 min | 0 (sync) |
| Region failure | DNS failover to secondary region | < 1 hour | < 5 min |
| Data corruption | Point-in-time recovery from WAL archive | < 4 hours | < 5 min |

### 6.2 Backup Strategy

| Component | Backup Method | Frequency | Retention |
|-----------|--------------|-----------|-----------|
| PostgreSQL | pg_basebackup + WAL archiving | Continuous | 30 days |
| Redis | RDB snapshots | Hourly | 7 days |
| RabbitMQ | Definitions export + message backup | Daily | 7 days |
| MinIO | Versioning + cross-region replication | Continuous | 90 days |
| Elasticsearch | Snapshot to MinIO | Daily | 14 days |
| K8s manifests | GitOps (ArgoCD source of truth) | Continuous | Indefinite |

## 7. Autoscaling

### 7.1 Horizontal Pod Autoscaler (HPA)

All services have HPA configured:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: commerce-service
spec:
  minReplicas: 3
  maxReplicas: 15
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
```

### 7.2 Autoscaling Thresholds

| Service | Min Replicas | Max Replicas | CPU Target | Memory Target |
|---------|-------------|-------------|------------|---------------|
| Auth | 3 | 10 | 70% | 80% |
| Commerce | 5 | 20 | 70% | 80% |
| Finance | 5 | 20 | 70% | 80% |
| Report | 3 | 10 | 60% | 70% |
| All others | 2 | 10 | 70% | 80% |

---

*See [Project Structure](project-structure.md) for local development setup.*
*See [Security](security.md) for mTLS and certificate management.*
