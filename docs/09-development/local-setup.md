# Local Development Setup

## 0. Prerequisites

| Tool | Minimum Version | Purpose |
|------|----------------|---------|
| Rust | 1.83+ | Language toolchain |
| Docker | 24+ | Container runtime |
| Docker Compose | 2.20+ | Multi-container orchestration |
| cargo-watch | latest | Hot-reload during development |
| sea-orm-cli | latest | Database migration management |

Install Rust toolchain:
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup default stable
```

Install cargo-watch and sea-orm-cli:
```bash
cargo install cargo-watch sea-orm-cli
```

## 0.1 Port Mapping

| Service | Port |
|---------|------|
| Auth | 8001 |
| Identity | 8002 |
| Tenant | 8003 |
| Config | 8004 |
| Commerce | 8010 |
| Finance | 8011 |
| HCM | 8012 |
| Manufacturing | 8013 |
| Report | 8014 |
| Workflow | 8015 |
| CRM | 8016 |
| Project | 8017 |
| Platform | 8020 |
| Integration | 8021 |

## 0.2 Database Migrations

Run migrations for all services:
```bash
cargo run -p mserp-migrations -- migrate
```

Run migrations for a specific service:
```bash
cargo run -p mserp-migrations -- migrate --service commerce
```

## 1. Local Development

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

## 2. Service Discovery

Services communicate via **Kubernetes DNS** in production and **Docker Compose service names** in development. No separate service registry is needed.

| Environment | Discovery | Example |
|-------------|-----------|---------|
| Production | K8s DNS (`<service>.mserp-services.svc.cluster.local`) | `http://finance-service:8011` |
| Development | Docker DNS (`http://<service>:<port>`) | `http://finance-service:8011` |
| Unit Tests | Mock adapters via trait injection | N/A |

## 3. Troubleshooting

| Issue | Solution |
|-------|----------|
| `cargo build` fails with linker errors | Ensure `pkg-config` and `libssl-dev` (or `openssl-devel`) are installed |
| Database connection refused | Verify PostgreSQL is running: `docker compose -f docker-compose.dev.yml ps postgres` |
| RabbitMQ connection refused | Check RabbitMQ status: `docker compose -f docker-compose.dev.yml logs rabbitmq` |
| Port already in use | Find and kill the process: `lsof -i :<port>` then `kill <PID>` |
| Migration fails | Ensure the database exists: `cargo run -p mserp-migrations -- migrate` |
| Hot-reload not working | Verify `cargo-watch` is installed: `cargo install cargo-watch` |
| Permission denied on Docker | Add user to docker group: `sudo usermod -aG docker $USER` then log out and back in |
| Service cannot reach another service | Verify both services are on the same Docker network: `docker network ls` |
| Outbox events not publishing | Check outbox poller logs and verify RabbitMQ exchange `mserp.events` exists |
