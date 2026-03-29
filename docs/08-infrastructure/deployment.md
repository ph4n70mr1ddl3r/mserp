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
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐                       │  │
│  │  │ Alert-  │ │  Loki   │ │ Vault / │ │ ArgoCD  │                       │  │
│  │  │ manager │ │         │ │  KMS    │ │         │                       │  │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘                       │  │
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
│  │  │Platform │ │Integrat-│ │CRM │ │ Project │                       │  │
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
| Commerce (Sales + Inventory + PIM + Transportation + Subscriptions + Credit + ATP/CTP + Configurator + Drop Ship + Logistics + Warranty + B2B) | 250m | 1000m | 256Mi | 1Gi | 5 |
| Finance (Finance + Procurement + Treasury + Expenses + CLM + EPM + Revenue Recognition + Sourcing + Supplier Risk + Reconciliation + Profitability + Lease + Grants + JV + Intelligent Close + Collections) | 250m | 1000m | 256Mi | 1Gi | 5 |
| HR | 100m | 500m | 128Mi | 512Mi | 3 |
| Manufacturing | 100m | 500m | 128Mi | 512Mi | 3 |
| Report | 500m | 2000m | 512Mi | 4Gi | 3 |
| Workflow | 100m | 500m | 128Mi | 512Mi | 3 |
| CRM | 100m | 500m | 128Mi | 512Mi | 3 |
| Project Management | 100m | 500m | 128Mi | 512Mi | 3 |
| Platform (Notif + File + Audit + Assistant + AppBuilder + GRC) | 200m | 1000m | 256Mi | 1Gi | 3 |
| Integration | 100m | 500m | 128Mi | 512Mi | 3 |

**Total minimum resources (all services at minimum replicas):**
- CPU: ~6.9 cores requested, ~19.9 cores limit
- Memory: ~5.4 GiB requested, ~13.5 GiB limit
- Pods: 44

## 3. Network Policies

All namespaces enforce network policies to restrict traffic flow:

### 3.1 Default Deny All

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: mserp-services
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

### 3.2 Allowed Traffic Matrix

| From | To | Ports | Protocol |
|------|----|-------|----------|
| Ingress Controller | mserp-services (all) | 8080 | HTTP |
| mserp-services (all) | mserp-infra (PostgreSQL) | 5432 | TCP |
| mserp-services (all) | mserp-infra (Redis) | 6379 | TCP |
| mserp-services (all) | mserp-infra (RabbitMQ) | 5672, 15672 | TCP |
| mserp-services (all) | mserp-infra (Elasticsearch) | 9200 | HTTP |
| mserp-services (all) | mserp-infra (MinIO) | 9000 | HTTP |
| mserp-services (all) | mserp-system (Vault) | 8200 | TCP |
| mserp-services (all) | External (via egress) | 443 | HTTPS |
| mserp-system (Prometheus) | mserp-services (all) | 8080 | HTTP (/metrics) |

### 3.3 DNS Egress

All namespaces allow egress to CoreDNS on port 53 (UDP/TCP).

## 4. Pod Disruption Budgets

All critical services define PDBs to ensure availability during voluntary disruptions (node drains, upgrades):

| Service | Min Available | Strategy |
|---------|---------------|----------|
| Auth | 2 | `minAvailable` |
| Commerce | 3 | `minAvailable` |
| Finance | 3 | `minAvailable` |
| Report | 2 | `minAvailable` |
| Platform | 2 | `minAvailable` |
| PostgreSQL | 1 | `minAvailable` |
| RabbitMQ | 1 | `minAvailable` |
| Redis | 1 | `minAvailable` |

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: commerce-pdb
  namespace: mserp-services
spec:
  minAvailable: 3
  selector:
    matchLabels:
      app: commerce-service
```

## 5. Topology Spread Constraints

Services are spread across failure domains to maximize resilience:

```yaml
topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: DoNotSchedule
    labelSelector:
      matchLabels:
        app: commerce-service
  - maxSkew: 1
    topologyKey: kubernetes.io/hostname
    whenUnsatisfiable: ScheduleAnyway
    labelSelector:
      matchLabels:
        app: commerce-service
```

**Rules:**
- Zone-level spreading uses `DoNotSchedule` (strict) for services with >= 3 replicas.
- Node-level spreading uses `ScheduleAnyway` (soft) to avoid blocking scheduling.

## 6. Resource Quotas

### 6.1 Namespace Quotas

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: mserp-services-quota
  namespace: mserp-services
spec:
  hard:
    requests.cpu: "8"
    requests.memory: 16Gi
    limits.cpu: "22"
    limits.memory: 32Gi
    pods: "150"
    persistentvolumeclaims: "20"
    services: "20"
```

### 6.2 Limit Ranges

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mserp-services-limits
  namespace: mserp-services
spec:
  limits:
    - default:
        cpu: "500m"
        memory: "512Mi"
      defaultRequest:
        cpu: "100m"
        memory: "128Mi"
      max:
        cpu: "4"
        memory: "4Gi"
      type: Container
```

## 7. CI/CD Pipeline

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
│                                             SAST (Semgrep)                 │
│                                             DAST (OWASP ZAP)              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 7.1 Pipeline Stages

| Stage | Actions | Failure Policy |
|-------|---------|---------------|
| Build | `cargo build --release`, Docker image build | Block merge |
| Test | Unit, integration, contract (Pact), saga tests | Block merge |
| Security | Trivy scan, `cargo audit`, `cargo deny`, Snyk, clippy, SAST, DAST | Block merge on Critical/High |
| Deploy (Staging) | ArgoCD auto-sync to staging | Alert on failure |
| Deploy (Production) | Manual approval + ArgoCD sync | Rollback on health check failure |

### 7.2 Artifact Promotion

```
Build (PR) → ghcr.io/mserp/my-service:sha-abc123
            → Trivy scan + SBOM generation
            → Cosign sign

Promote (Staging) → Retag: ghcr.io/mserp/my-service:staging
                  → ArgoCD auto-deploy

Promote (Production) → Retag: ghcr.io/mserp/my-service:v1.2.3
                     → Manual approval gate
                     → ArgoCD sync with manual trigger
```

**The same signed image artifact is promoted through environments — no rebuild between stages.**

### 7.3 Contract Testing

| Aspect | Details |
|--------|---------|
| Framework | Pact (Rust: `pact_consumer` + `pact_provider`) |
| Consumer Tests | Each service publishes its expected provider contract |
| Provider Verification | Provider services verify all consumer contracts on every PR |
| Event Contracts | Event schemas are versioned; breaking changes bump `event_version` |
| Enforcement | PR cannot merge if provider verification fails |

### 7.4 Branch Strategy

| Branch | Purpose | Deploy Target |
|--------|---------|--------------|
| `main` | Production-ready code | Production (via tag) |
| `develop` | Integration branch | Staging (auto-deploy) |
| `feature/*` | Feature branches | None (CI only) |
| `hotfix/*` | Emergency fixes | Production (expedited) |
| `release/*` | Release candidates | Staging + Production (manual) |

## 8. Health Probes (Kubernetes Configuration)

### 8.1 Liveness Probe

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

### 8.2 Readiness Probe

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

### 8.3 Startup Probe (for services with slow initialization)

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

## 9. Graceful Shutdown

### 9.1 Shutdown Sequence

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

### 9.2 Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `shutdown.http_drain_timeout` | 30s | Max time to wait for in-flight HTTP requests |
| `shutdown.event_drain_timeout` | 15s | Max time to wait for in-flight event handlers |
| `shutdown.outbox_flush_timeout` | 10s | Max time to flush pending outbox events |
| `shutdown.total_timeout` | 60s | Hard deadline — process is killed after this |

### 9.3 Kubernetes Integration

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

> **Note:** If using distroless container images (no shell), replace the `exec` preStop with a custom waiting binary or use the `Sleep` action with a minimal wait image like `registry.k8s.io/pause:3.9`.

### 9.4 Implementation Requirements

- Use `tokio::signal::unix::signal(SIGTERM)` for signal handling.
- Use `axum::serve()` with graceful shutdown via `tokio::select!`.
- Track in-flight requests with an `AtomicUsize` counter.
- Outbox poller MUST check the draining state and exit its loop.
- Connection pools (database, Redis, RabbitMQ) MUST be closed explicitly.

## 10. Database Migrations

### 10.1 Migration Strategy

| Aspect | Policy |
|--------|--------|
| Tool | `sea-orm-migration` (part of SeaORM ecosystem) |
| Execution | Kubernetes Init Container runs migrations before service starts |
| Backward Compatibility | Migrations MUST be backward-compatible — both old and new service versions must work |
| Rolling Strategy | Additive changes first (add column), removal in subsequent release |
| Rollback | Forward-only migrations; rollback = deploy previous version + new forward migration |

### 10.2 Init Container Pattern

```yaml
initContainers:
  - name: migrate
    image: ghcr.io/mserp/my-service:v1.2.3
    command: ["./my-service", "migrate"]
    envFrom:
      - secretRef:
          name: db-credentials
```

### 10.3 Migration Rules

- **Add columns**: `NULL` or with `DEFAULT` value only. Never `NOT NULL` without a default in a single step.
- **Rename columns**: Two-release process — add new column, populate via background job, remove old column in next release.
- **Drop columns**: Two-release process — stop reading/writing old column in code first, drop in next release.
- **Index creation**: Use `CONCURRENTLY` to avoid locking tables.
- **Large data migrations**: Run as background jobs, not in the migration init container.

## 11. Multi-Region Deployment

| Aspect | Specification |
|--------|---------------|
| Strategy | Active-passive failover with warm standby |
| DNS | Primary region receives 100% of traffic; secondary receives no traffic during normal operation |
| Failover DNS | On primary failure, DNS updated to secondary (automated via health checks); RTO < 1 hour |
| Data Replication | PostgreSQL uses streaming replication with synchronous standby for the primary region failover (zero data loss), and logical replication for cross-region data synchronization (async, < 5 min lag) |
| RabbitMQ | Federation plugin for cross-region queue mirroring |
| Redis | Redis Enterprise with active-passive replication |
| File Storage | MinIO site replication |
| Data Residency | Region-pinned tenants route to designated cluster; cross-region transfer prohibited without consent |

### 11.1 Disaster Recovery

| Scenario | Recovery Procedure | RTO | RPO |
|----------|-------------------|-----|-----|
| Single pod failure | K8s auto-restart | < 30s | 0 |
| Node failure | K8s reschedule pods | < 2 min | 0 |
| Database failover | PostgreSQL streaming replication failover | < 5 min | 0 (sync) |
| Region failure | DNS failover to secondary region | < 1 hour | < 5 min |
| Data corruption | Point-in-time recovery from WAL archive | < 4 hours | < 5 min |

### 11.2 Backup Strategy

| Component | Backup Method | Frequency | Retention | Verification |
|-----------|--------------|-----------|-----------|-------------|
| PostgreSQL | pg_basebackup + WAL archiving | Continuous | 30 days | Weekly restore test to staging |
| Redis | RDB snapshots | Hourly | 7 days | Monthly restore test |
| RabbitMQ | Definitions export + message backup | Daily | 7 days | Monthly restore test |
| MinIO | Versioning + cross-region replication | Continuous | 90 days | Integrity check on restore |
| Elasticsearch | Snapshot to MinIO | Daily | 14 days | Weekly restore test |
| K8s manifests | GitOps (ArgoCD source of truth) | Continuous | Indefinite | Git diff on sync |
| Vault | Encrypted snapshots | Daily | 30 days | Monthly seal/unseal test |
| Data Lake | Cross-region replication via MinIO | Continuous | Indefinite | Integrity check on replicate |

## 12. Autoscaling

### 12.1 Horizontal Pod Autoscaler (HPA)

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

### 12.2 Autoscaling Thresholds

| Service | Min Replicas | Max Replicas | CPU Target | Memory Target |
|---------|-------------|-------------|------------|---------------|
| Auth | 3 | 10 | 70% | 80% |
| Commerce | 5 | 20 | 70% | 80% |
| Finance | 5 | 20 | 70% | 80% |
| Report | 3 | 10 | 55% | 70% |
| Platform | 3 | 15 | 70% | 80% |
| All others | 2 | 10 | 70% | 80% |

### 12.3 Vertical Pod Autoscaler (VPA)

VPA runs in recommendation-only mode for all services to inform resource right-sizing:

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: commerce-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: commerce-service
  updatePolicy:
    updateMode: "Off"  # Recommendations only — manual application
```

## 13. Deployment Strategies

### 13.1 Rolling Update (Default)

Standard Kubernetes rolling update for all routine deployments. Zero downtime guaranteed by readiness probes.

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
```

### 13.2 Blue-Green Deployment

For major version upgrades or breaking schema changes:

```
1. Deploy new version to "green" environment alongside "blue" (current)
2. Run smoke tests and integration tests against "green"
3. Switch traffic from "blue" to "green" via Kubernetes service selector
4. Monitor for 30 minutes; automatic rollback on error rate > 1%
5. Scale down "blue" after verification period
```

### 13.3 Canary Deployment

For high-risk changes to critical services (Commerce, Finance):

```
1. Deploy new version to 10% of pods
2. Monitor error rate, latency, and business metrics for 30 minutes
3. If metrics are within thresholds, gradually increase to 25% → 50% → 100%
4. Automatic rollback if canary error rate exceeds baseline by 0.5%
```

## 14. Infrastructure as Code (IaC)

| Tool | Purpose |
|------|---------|
| Terraform | Cloud infrastructure provisioning (VPC, databases, load balancers) |
| Helm | Kubernetes application packaging and templating |
| Kustomize | Environment-specific overlay configuration |
| ArgoCD | GitOps continuous delivery — cluster state synced from Git |
| Crossplane (optional) | Infrastructure composition for multi-cloud deployments |

All infrastructure definitions are version-controlled in the same repository under `infra/`.

### 14.1 Directory Structure

```
infra/
├── terraform/           # Cloud infrastructure (VPC, RDS, etc.)
│   ├── modules/
│   └── environments/
│       ├── staging/
│       └── production/
├── helm/                # Helm charts for each service
│   ├── charts/
│   │   ├── auth-service/
│   │   ├── commerce-service/
│   │   └── ...
│   └── values/
│       ├── staging.yaml
│       └── production.yaml
├── k8s/                 # Environment overlays
│   ├── base/
│   └── overlays/
│       ├── staging/
│       └── production/
└── argocd/              # ArgoCD Application manifests
    ├── applications/
    └── app-of-apps.yaml
```

---

*See [Project Structure](../09-development/project-structure.md) for local development setup.*
*See [Security](../02-security/overview.md) for mTLS and certificate management.*
