# Code Conventions

## 4. Shared Libraries

| Crate | Purpose | Used By |
|-------|---------|---------|
| `mserp-core` | Error types, Result types, config loading, pagination, i18n, money types, data classification, utilities | All services |
| `mserp-auth` | JWT validation, auth middleware, permission checks, step-up auth | All services |
| `mserp-infrastructure` | Database setup (RLS), Redis client, RabbitMQ client, outbox, saga, tracing, metrics | All services |
| `mserp-proto` | Event type definitions, shared DTOs, contract schemas | All services |

## 5. Code Conventions

### 5.1 Module Structure (per service)

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
│   └── events/           # Event handlers (RabbitMQ consumers)
├── Cargo.toml
├── Dockerfile
└── tests/                # Integration tests (testcontainers)
```

### 5.2 Naming Conventions

| Item | Convention | Example |
|------|-----------|---------|
| Service crate | `kebab-case` | `commerce-service` |
| Rust module | `snake_case` | `sales_order` |
| Database table | `snake_case` | `sales_orders` |
| Database column | `snake_case` | `customer_id` |
| API endpoint | `kebab-case` | `/api/v1/commerce/price-lists` |
| Event type | `dot.case` | `commerce.order.created` |
| Environment variable | `UPPER_SNAKE_CASE` | `MSERP_DATABASE_URL` |

### 5.3 Error Handling

All services use the shared `mserp-core::error` types which implement RFC 7807 Problem Details. Services define domain-specific error variants that map to HTTP status codes.
