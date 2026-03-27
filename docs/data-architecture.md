# Data Architecture

## 1. Database per Service Pattern

Each service owns its database exclusively. No service may query another service's database directly — all cross-service data access is via REST APIs or event subscriptions.

```
┌─────────────────────────────────────────────────────────────┐
│                     PostgreSQL Cluster                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ── Core Service Databases ────────────────────────────────  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │  auth_db    │  │ identity_db │  │  tenant_db  │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
│  ┌─────────────┐                                             │
│  │  config_db  │                                             │
│  └─────────────┘                                             │
│                                                              │
│  ── Business Service Databases ────────────────────────────  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │ commerce_db │  │ finance_db  │  │   hr_db     │          │
│  │ (Sales +    │  │ (Finance +  │  │             │          │
│  │  Inventory) │  │ Procurement)│  │             │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │manufacturing│  │ report_db   │  │ workflow_db │          │
│  │    _db      │  │             │  │             │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
│  ┌─────────────┐  ┌─────────────┐                             │
│  │  crm_db     │  │ project_db  │                             │
│  └─────────────┘  └─────────────┘                             │
│                                                              │
│  ── Supporting Service Databases ──────────────────────────  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │ platform_db │  │  audit_db   │  │integration_ │          │
│  │ (Notif +    │  │ (time-series│  │     db      │          │
│  │  File)      │  │  optimized) │  │             │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
└─────────────────────────────────────────────────────────────┘
```

Total: **14 databases** (4 core + 8 business + 2 supporting).

## 2. Common Schema Elements

### Standard Columns (all tables)

Every table in every service database MUST include these standard columns:

```sql
id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
tenant_id       UUID NOT NULL,
created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
created_by      UUID,
updated_by      UUID,
version         INTEGER NOT NULL DEFAULT 1,
is_deleted      BOOLEAN NOT NULL DEFAULT FALSE
```

### Standard Indexes

```sql
CREATE INDEX idx_{table}_tenant_id ON {table} (tenant_id);
CREATE INDEX idx_{table}_created_at ON {table} (created_at);
CREATE INDEX idx_{table}_is_deleted ON {table} (is_deleted);
```

> **Note:** Event and outbox tables use `tenant_id UUID` (nullable) instead of `NOT NULL` — system-level events (e.g., `config.changed`, `auth.login.failed`) are not scoped to a specific tenant.

## 3. Multi-Tenancy Strategy

| Strategy | Implementation |
|----------|----------------|
| Isolation Level | Logical (shared database, `tenant_id` column on every table) |
| Row-Level Security | PostgreSQL RLS policies enforced at connection level |
| Data Partitioning | Partition large tables by `tenant_id` using declarative partitioning |
| Connection Pooling | Shared connection pool; RLS `tenant_id` set per transaction via `app.current_tenant` |
| Quota Enforcement | Tenant Service tracks usage; API Gateway enforces limits |

### RLS Implementation

```sql
-- Per-table RLS policy (applied by migration)
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON orders
    USING (tenant_id = current_setting('app.current_tenant')::UUID);
```

The `app.current_tenant` session variable is set at the beginning of each request by the service's database middleware, extracted from the JWT's `tenant_id` claim.

## 4. Database Migration Strategy

### 4.1 Migration Tool

All migrations use **sea-orm-migration** (managed via `mserp-infrastructure` crate). Each service has its own migration directory under `migrations/{service}/`.

### 4.2 Migration Rules

| Rule | Description |
|------|-------------|
| Forward-Only | Migrations MUST be forward-only. No `down()` migrations in production. |
| Backward Compatibility | Every migration MUST be backward-compatible with the previous application version. Old code MUST work with the new schema for at least one deployment cycle. |
| Zero-Downtime | Migrations MUST NOT lock tables for more than 100ms. Use `CREATE INDEX CONCURRENTLY`, `ALTER TABLE ... ADD COLUMN` (with defaults), and phased column drops. |
| Idempotent | Each migration script MUST be idempotent — safe to re-run if partially applied. |
| Ordered | Migrations are applied in lexicographic order. Use timestamp prefix: `{YYYYMMDDHHMMSS}_{description}.rs` |

### 4.3 Schema Change Patterns

| Change Type | Safe Pattern |
|-------------|-------------|
| Add column | `ALTER TABLE ADD COLUMN` with a default value. Non-nullable columns must have defaults. |
| Remove column | 3-phase: (1) Stop reading column in app, (2) Deploy, (3) Drop column in next migration. |
| Rename column | Add new column + copy data in app layer + drop old column (never `ALTER TABLE RENAME`). |
| Add index | `CREATE INDEX CONCURRENTLY` (never blocks writes). |
| Remove index | `DROP INDEX CONCURRENTLY`. |
| Modify column type | Add new column + migrate data + swap in app + drop old column. |

### 4.4 Rollback Procedure

1. Revert the application deployment to the previous version.
2. The previous version MUST still work with the new schema (backward compatibility guarantee).
3. If the schema change is truly incompatible, create a compensating migration and deploy both together.

### 4.5 Cross-Database Coordination

When a schema change affects multiple services (e.g., adding a field referenced by events):
1. Deploy consumer services first (they ignore the new field).
2. Apply schema migration to the producer's database.
3. Deploy the producer service (now includes the new field in events).
4. Deploy updated consumer services (now handle the new field).

## 5. Soft Delete Strategy

### 5.1 Behavior

- All tables include `is_deleted BOOLEAN NOT NULL DEFAULT FALSE`.
- Soft delete sets `is_deleted = TRUE` and `updated_at = NOW()`.
- Queries MUST include `WHERE is_deleted = FALSE` by default (enforced by SeaORM global filter).

### 5.2 Soft Delete and Events

| Operation | Event Published? | Event Type |
|-----------|-----------------|------------|
| Create | Yes | `{domain}.{entity}.created` |
| Update | Yes | `{domain}.{entity}.updated` |
| Soft Delete | Yes | `{domain}.{entity}.deleted` |
| Hard Delete | No | N/A (hard deletes are admin-only, audited separately) |

- Soft deletes ARE published as events so downstream services can react (e.g., update search indexes).
- Hard deletes are NOT published. They are reserved for GDPR data erasure and require Super Admin + audit log entry.

### 5.3 Unique Constraints with Soft Deletes

Soft-deleted rows can cause unique constraint violations on natural keys (e.g., email, SKU). Use partial unique indexes:

```sql
CREATE UNIQUE INDEX idx_users_email_active
    ON users (tenant_id, email)
    WHERE is_deleted = FALSE;
```

### 5.4 Data Retention and Purge

| Data Category | Soft Delete Retention | Hard Delete Policy |
|--------------|----------------------|-------------------|
| Business data (orders, invoices) | Indefinite | Manual / GDPR request |
| Audit logs | Indefinite | Never (compliance requirement) |
| Temporary data (sessions, tokens) | 90 days | Automated nightly purge |
| Event outbox | 30 days | Automated nightly purge |
| File metadata | Follows linked entity | Cascade with entity |
| Import/export job data | 90 days | Automated nightly purge |
| Report execution history | 1 year | Automated nightly purge |

## 6. Audit Trail (Event Log)

> **Note:** This is an immutable audit log for compliance and data lineage — not event sourcing. The system does not use event sourcing as its primary persistence pattern. Business state is stored in domain tables; this table is a supplementary write-only log.

```
┌──────────────────────────────────────────────────────────┐
│                    Event Store                            │
├──────────────────────────────────────────────────────────┤
│ event_id        UUID PRIMARY KEY                          │
│ aggregate_id    UUID NOT NULL                             │
│ aggregate_type  VARCHAR(100) NOT NULL                     │
│ event_type      VARCHAR(100) NOT NULL                     │
│ event_data      JSONB NOT NULL                            │
│ metadata        JSONB                                     │
│ tenant_id       UUID          (NULL for system events)    │
│ user_id         UUID                                      │
│ occurred_at     TIMESTAMPTZ NOT NULL                      │
│ version         INTEGER NOT NULL                          │
└──────────────────────────────────────────────────────────┘
```

The `audit_db` database is time-series optimized with automatic partitioning by `occurred_at` (monthly partitions). Old partitions are archived to object storage after 2 years but remain queryable.

## 7. Caching Strategy

### 7.1 Cache Architecture

MSERP uses **cache-aside** pattern with Redis as the caching layer.

### 7.2 Cache Policies by Data Type

| Data Type | Cache Strategy | TTL | Invalidation |
|-----------|---------------|-----|-------------|
| User sessions | Write-through | 24 hours | On logout / token refresh |
| Feature flags | Cache-aside (API Gateway + service in-memory) | 30 seconds | On `tenant.feature.changed` event |
| System config | Cache-aside | 30 seconds | On `config.changed` event |
| Product catalog (read-heavy) | Cache-aside | 5 minutes | On `commerce.product.*` / `commerce.stock.*` events |
| Customer data | Cache-aside | 10 minutes | On `commerce.customer.*` events |
| Chart of accounts | Cache-aside | 1 hour | On `finance.account.*` events |
| Exchange rates | Cache-aside | 1 hour | On daily rate refresh |
| Rate limit counters | Sliding window | 1 minute | Automatic expiry |
| Reference data (currencies, countries) | Cache-aside | 1 hour | On deploy / manual flush |
| Localization strings | Cache-aside | 1 hour | On `config.changed` (i18n keys) |

### 7.3 Cache Key Format

```
mserp:{tenant_id}:{service}:{entity}:{id}
```

The `{tenant_id}` segment uses the literal string `global` for system-wide, tenant-agnostic data.

Examples:
- `mserp:tenant-123:commerce:product:prod-456`
- `mserp:tenant-123:finance:account:acc-789`
- `mserp:global:tenant:feature_flags` (global defaults)
- `mserp:tenant-123:tenant:feature_flags` (per-tenant overrides)
- `mserp:global:config:system` (system configuration)
- `mserp:global:finance:exchange_rates:USD_EUR` (exchange rates)

### 7.4 Cache Invalidation

- **Event-driven**: Services subscribe to relevant domain events and invalidate affected cache entries.
- **TTL-based**: All cached entries have a TTL as a safety net against stale data.
- **Manual flush**: Tenant Admins can trigger cache flush for their tenant via `POST /api/v1/config/cache/flush`.

### 7.5 Cold Start Behavior

- On service startup, caches are empty (cold).
- First requests are served directly from the database.
- Caches warm up naturally as requests arrive.
- For critical reference data, services MAY pre-warm caches on startup via a configurable list of pre-warm queries.

## 8. Reference Data and Seeding

### 8.1 Reference Data

Reference data is seeded during initial deployment and managed via migrations.

| Category | Source | Update Frequency |
|----------|--------|-----------------|
| Currencies (ISO 4217) | Static seed | On deploy |
| Countries (ISO 3166) | Static seed | On deploy |
| Tax Codes | Tenant-configurable | Via API |
| Units of Measure | Static seed + tenant-customizable | On deploy + API |
| Payment Terms | Tenant-configurable | Via API |
| Document Templates | Static seed | On deploy |
| Locales / Languages | Static seed | On deploy |
| Exchange Rates | ECB API (automated) + manual override | Daily |
| Industry Classifications (NAICS/SIC) | Static seed | On deploy |

### 8.2 Seeding Process

```bash
# Seed reference data (run once per environment)
make seed

# Seed test data (development only)
make seed-test
```

- Seed scripts are idempotent — safe to re-run.
- Production seeds only reference data (no fake business data).
- Development seeds include sample tenants, users, and business data for testing.
- Seeds use SeaORM migrations with `INSERT ... ON CONFLICT DO NOTHING` for idempotency.

## 9. Data Warehouse (Report Service)

The Report Service maintains a DuckDB-based embedded data warehouse for analytical queries, separate from the operational PostgreSQL databases.

### 9.1 ETL Pipeline

```
Operational DBs (PostgreSQL) ──▶ ETL Jobs (scheduled) ──▶ DuckDB (Star Schema)
                                        │
                                        ├── Extract: Read-only queries on operational data
                                        ├── Transform: Denormalize, aggregate, compute measures
                                        └── Load: Upsert into fact/dimension tables
```

### 9.2 Schema Design

| Table Type | Examples | Refresh Frequency |
|-----------|----------|-------------------|
| Dimension tables | `dim_customer`, `dim_product`, `dim_date`, `dim_employee` | Hourly |
| Fact tables | `fact_sales`, `fact_inventory_movement`, `fact_payroll` | Hourly |
| Aggregate tables | `agg_daily_sales`, `agg_monthly_revenue` | Daily |

### 9.3 Data Freshness

- Real-time dashboards use event stream aggregation (not warehouse).
- Historical reports and trend analysis use the data warehouse.
- Warehouse data is at most 1 hour stale for dimension data and fact data.
- Aggregate tables are refreshed daily during off-peak hours.

---

*See [Events](events.md) for the outbox table schema and event patterns.*
*See [Deployment](deployment.md) for database infrastructure details.*
