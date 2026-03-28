# Local Development Setup

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
