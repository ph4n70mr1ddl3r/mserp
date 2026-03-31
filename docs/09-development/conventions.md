# Code Conventions

## 1. Shared Libraries

| Crate | Purpose | Used By |
|-------|---------|---------|
| `mserp-core` | Error types, Result types, config loading, pagination, i18n, money types, data classification, utilities | All services |
| `mserp-auth` | JWT validation, auth middleware, permission checks, step-up auth | All services |
| `mserp-infrastructure` | Database setup (RLS), Redis client, RabbitMQ client, outbox, saga, tracing, metrics | All services |
| `mserp-proto` | Event type definitions, shared DTOs, contract schemas | All services |
| `mserp-migrations` | Centralized migration runner for development; per-service migration in production | All services |

## 2. Code Conventions

### 2.1 Module Structure (per service)

```
service-name/
├── src/
│   ├── main.rs           # Entry point, server startup, graceful shutdown
│   ├── config.rs         # Service-specific configuration struct
│   ├── routes/           # Axum route definitions
│   │   ├── mod.rs
│   │   └── {entity}.rs   # One file per entity/resource
│   ├── handlers/         # Request handlers
│   ├── models/           # Domain models (SeaORM entities)
│   ├── repository/       # Data access layer (trait + impl)
│   ├── services/         # Business logic
│   └── events/           # Event handlers (RabbitMQ consumers); event types defined in mserp-proto crate
├── Cargo.toml
├── Dockerfile
└── tests/                # Integration tests (testcontainers)
```

### 2.2 Naming Conventions

| Item | Convention | Example |
|------|-----------|---------|
| Service crate | `kebab-case` | `commerce-service` |
| Rust module | `snake_case` | `sales_order` |
| Database table | `snake_case` | `sales_orders` |
| Database column | `snake_case` | `customer_id` |
| API endpoint | `kebab-case` | `/api/v1/commerce/price-lists` |
| Event type | `dot.case` | `commerce.order.created` |
| Environment variable | `UPPER_SNAKE_CASE` | `MSERP_DATABASE_URL` |

### 2.3 Error Handling

All services use the shared `mserp-core::error` types which implement RFC 7807 Problem Details. Services define domain-specific error variants that map to HTTP status codes.

### 2.4 Async/Await Patterns

- Use `tokio::spawn` for fire-and-forget tasks (e.g., publishing events, sending notifications).
- Always handle `JoinError` from `tokio::spawn` results in critical paths.
- Prefer structured concurrency: avoid detaching tasks that must complete for correctness.
- Use `tokio::select!` for cancelling background work on shutdown signals.

### 2.5 Database Query Patterns

- Always use parameterized queries via SeaORM — never interpolate values into raw SQL.
- Use SeaORM entity models for all CRUD operations.
- For complex queries, use `sea_query` builders rather than raw SQL strings.
- Wrap all mutations in transactions (`DatabaseTransaction`) when multiple writes are involved.
- Apply tenant_id via RLS as required by SPEC.md §9.3.

### 2.6 Testing Conventions

Test types and coverage requirements are defined in SPEC.md §18. The table below provides code-level conventions.

| Test Type | Convention |
|-----------|-----------|
| Unit tests | Place in the same file as the code under test using the `#[cfg(test)]` module pattern |
| Integration tests | Place in the service-level `tests/` directory using testcontainers for real database and message broker instances |
| Contract tests | Use Pact consumer-driven contracts for all inter-service boundaries |
| Saga compensation tests | Every saga step must have a test verifying its compensating action |

### 2.7 Commit Message Format

Use [Conventional Commits](https://www.conventionalcommits.org/):

```
type(scope): description

[optional body]

[optional footer(s)]
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`

Examples:
- `feat(commerce): add quote-to-order conversion endpoint`
- `fix(finance): correct tax calculation for intercompany invoices`
- `docs(api): update pagination parameters for order listing`

### 2.8 PR Review Guidelines

- All PRs require at least 1 approval before merging.
- All CI checks must pass (lint, type-check, unit tests, integration tests).
- PRs must include a description summarizing the change and referencing any related issue.
- Breaking API changes must include a migration plan in the PR description.
- No direct commits to `main` — all changes go through pull requests.
