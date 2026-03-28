# Data Architecture Overview

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
│  │  Inventory +│  │ Procurement+│  │             │          │
│  │  PIM +      │  │ Treasury +  │  │             │          │
│  │  Transport) │  │ Expenses +  │  │             │          │
│  │             │  │ CLM + EPM)  │  │             │          │
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
│  │  File +     │  │  optimized) │  │ (+ MDM +    │          │
│  │  Assistant +│  │             │  │  Data Gov)  │          │
│  │  AppBuilder)│  │             │  │             │          │
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
| Data Residency | Region-pinned tenants route to designated PostgreSQL cluster; enforced at connection routing layer |

### RLS Implementation

```sql
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
| ESG / emissions data | Indefinite | Never (regulatory requirement) |
| ML model artifacts | 2 years | Automated purge of deprecated models |
| Custom application data (low-code) | Follows app lifecycle | Cascade with application |
| Subscription billing history | 7 years | Automated archive after 7 years (financial record) |
| Revenue recognition schedules | 7 years | Automated archive after 7 years (financial record) |
| Trade compliance screening logs | 5 years | Never (regulatory requirement) |
| Scheduled job execution history | 90 days | Automated nightly purge |
| Knowledge article versions | Follows article lifecycle | Cascade with article deletion |
| Survey responses | Indefinite (while survey active) | Archive after survey deactivation |
| Lease contracts and schedules | 7 years (lease term + regulatory) | Automated archive after retention period |
| Grant records and compliance | Duration of grant + 7 years | Automated archive after retention period |
| Joint venture allocations | 7 years (regulatory) | Automated archive after retention period |
| Warranty claims | Duration of warranty + 7 years | Automated archive after retention period |
| Collection activities | 7 years (financial record) | Automated archive after retention period |
| IDP extraction results | 90 days | Automated nightly purge |
| Data Lake (curated zone) | Schema-on-read for exploratory analytics | Unlimited raw zone storage per tenant |

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
│ data_classification VARCHAR(20) DEFAULT 'Internal'        │
└──────────────────────────────────────────────────────────┘
```

The `audit_db` database is time-series optimized with automatic partitioning by `occurred_at` (monthly partitions). Old partitions are archived to object storage after 2 years but remain queryable. The `data_classification` column enables filtering by data sensitivity level (Public, Internal, Confidential, Restricted).

## 7. Reference Data and Seeding

### 7.1 Reference Data

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
| Emission Factors | EPA / DEFRA / configurable source | Annually |
| ESG Reporting Frameworks | Static seed (GRI, SASB, TCFD, EU Taxonomy) | On deploy |
| Carrier Service Levels | Static seed + tenant-customizable | On deploy + API |
| Payroll Tax Jurisdictions | Per-country seed + configurable | On deploy + API |
| Trade Compliance Denied Party Lists | Government list feeds (OFAC, EU, UN) | Daily (automated) |
| Export Control Classifications (ECCN) | Static seed + tenant-configurable | On deploy + API |
| Subscription Billing Cycles | Tenant-configurable | Via API |
| Revenue Recognition Methods (ASC 606 / IFRS 15) | Static seed (standard recognition methods) | On deploy |
| Knowledge Base Categories | Tenant-configurable | Via API |
| Sales Territory Geographies | Static seed (ISO country/region) | On deploy |
| XBRL Taxonomy Mappings | Static seed + tenant-customizable | On deploy + API |
| IoT Device Types | Static seed + tenant-customizable | On deploy + API |
| Process Mining Activity Classifications | Static seed + tenant-customizable | On deploy + API |
| RPA Bot Templates | Static seed + tenant-customizable | On deploy + API |
| Customer Segments (CDP) | Tenant-configurable | Via API |
| Supplier Risk Indicators | Static seed + tenant-customizable | On deploy + API |
| Lease Classification Types | Static seed (operating, finance) | On deploy |
| Grant Funding Sources | Tenant-configurable | Via API |
| Joint Venture Partner Types | Static seed + tenant-customizable | On deploy + API |
| Collection Strategy Templates | Static seed + tenant-customizable | On deploy + API |
| Warranty Claim Reasons | Tenant-configurable | Via API |
| IDP Document Types | Static seed (invoice, PO, contract, receipt, shipping) | On deploy + API |

### 7.2 Seeding Process

```bash
make seed
make seed-test
```

- Seed scripts are idempotent — safe to re-run.
- Production seeds only reference data (no fake business data).
- Development seeds include sample tenants, users, and business data for testing.
- Seeds use SeaORM migrations with `INSERT ... ON CONFLICT DO NOTHING` for idempotency.

---

*See [Caching Strategy](caching.md) for cache architecture and policies.*
*See [Data Warehouse](warehouse.md) for analytical schema design.*
*See [Data Lake](data-lake.md) for raw and curated data zones.*
*See [Master Data Management Schema](mdm.md) for MDM and related schemas.*
*See [Domain Data Models](domain-models.md) for domain-specific data models.*
