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

Total: **15 databases** (4 core + 8 business + 3 supporting).

## 2. Common Schema Elements

Standard columns, indexes, and constraints are defined in **SPEC.md §9.1–9.2**. All tables MUST include these.

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

RLS policy implementation is defined in **SPEC.md §9.3**. Connection pooling is configured via PgBouncer (see `docs/08-infrastructure/technology.md`).

## 4. Database Migration Strategy

### 4.1 Migration Tool

All migrations use **sea-orm-migration** (managed via `mserp-infrastructure` crate). Each service has its own migration directory under `migrations/{service}/`.

### 4.2 Migration Rules

Migration rules and safe schema change patterns are defined in **SPEC.md §9.4–9.5**.

### 4.3 Rollback Procedure

1. Revert the application deployment to the previous version.
2. The previous version MUST still work with the new schema (backward compatibility guarantee).
3. If the schema change is truly incompatible, create a compensating migration and deploy both together.

### 4.4 Cross-Database Coordination

When a schema change affects multiple services (e.g., adding a field referenced by events):
1. Deploy consumer services first (they ignore the new field).
2. Apply schema migration to the producer's database.
3. Deploy the producer service (now includes the new field in events).
4. Deploy updated consumer services (now handle the new field).

## 5. Soft Delete Strategy

### 5.1 Behavior

Soft delete rules are defined in **SPEC.md §9.6**.

### 5.2 Unique Constraints with Soft Deletes

Soft-deleted rows can cause unique constraint violations on natural keys (e.g., email, SKU). Use partial unique indexes:

```sql
CREATE UNIQUE INDEX idx_users_email_active
    ON users (tenant_id, email)
    WHERE is_deleted = FALSE;
```

### 5.3 Data Retention and Purge

> **Note:** Data retention policy is authoritatively defined in SPEC.md §9.8. The table below provides supplementary storage-tier implementation details; where retention periods differ from SPEC.md, SPEC.md takes precedence.

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

## 6. Transactional Outbox Tables

Each service database contains its own `outbox_events` table used by the transactional outbox pattern. When a service performs a business action, it writes the event to its local `outbox_events` table within the same database transaction. A separate outbox poller process then publishes these events to RabbitMQ. This ensures atomicity between domain state changes and event publication without distributed transactions.

## 7. Audit Trail (Event Log)

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

### Event Envelope → Audit Trail Field Mapping

The event envelope (defined in `docs/04-events/overview.md`) and the audit trail event store share common fields under different names. The following table maps between them:

| Event Envelope Field | Audit Trail Event Store Field | Notes |
|----------------------|-------------------------------|-------|
| `event_id` | `event_id` | Same name, same UUID |
| `aggregate.id` | `aggregate_id` | Envelope nests under `aggregate` object |
| `aggregate.type` | `aggregate_type` | Envelope nests under `aggregate` object |
| `event_type` | `event_type` | Same name |
| `payload` | `event_data` | Renamed: envelope `payload` → audit `event_data` |
| `metadata` | `metadata` | Same name |
| `tenant_id` | `tenant_id` | Same name; nullable for system events |
| `metadata.user_id` | `user_id` | Promoted from nested metadata to top-level |
| `timestamp` | `occurred_at` | Renamed: envelope `timestamp` → audit `occurred_at` |
| `metadata.version` | `version` | Promoted from nested metadata to top-level |
| _(not present)_ | `data_classification` | Derived from entity type or configured per event type |

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

### 8.2 Seeding Process

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
*See [Data Pipeline](data-pipeline.md) for the analytical pipeline (data lake + warehouse).*
*See [Domain Data Models](domain-models.md) for domain-specific data models.*
