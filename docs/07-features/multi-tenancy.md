# Multi-Tenancy

Tenant lifecycle management, data isolation, context propagation, feature toggles, and data residency enforcement across all MSERP services, managed by the Tenant Service.

## Overview

Multi-Tenancy is the foundational capability that enables MSERP to serve multiple organisations from a single deployment while maintaining strict data isolation and per-tenant customisation. The Tenant Service manages the full tenant lifecycle from provisioning through decommissioning. Data isolation is enforced at every layer: PostgreSQL Row-Level Security (RLS), Redis key-prefix namespacing, Elasticsearch per-tenant indices, and MinIO path-prefix isolation. Tenant context (X-Tenant-Id header) is propagated through all API calls and event processing. Feature toggles allow per-tenant feature enablement, and data residency rules ensure compliance with regional data protection regulations.

## Architecture Diagram

```
┌──────────────────────────────────────────────────────────────────────┐
│                      Multi-Tenancy Architecture                       │
│                                                                       │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │                    Tenant Context Propagation                   │  │
│  │                                                                 │  │
│  │  API Gateway ──▶ X-Tenant-Id Header ──▶ Service Request Context │  │
│  │                         │                                       │  │
│  │                         ▼                                       │  │
│  │  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌─────────────┐ │  │
│  │  │ PostgreSQL│  │ Redis     │  │ Elastic-  │  │ MinIO       │ │  │
│  │  │ RLS      │  │ Key Prefix│  │ search    │  │ Path Prefix │ │  │
│  │  │ (tenant_id│  │ (tenant:  │  │ Per-tenant│  │ (/tenant_id/│ │  │
│  │  │  column) │  │  key)     │  │  indices) │  │  path)      │ │  │
│  │  └───────────┘  └───────────┘  └───────────┘  └─────────────┘ │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                       │
│  ┌─────────────────── Tenant Lifecycle ───────────────────────────┐  │
│  │                                                                 │  │
│  │  Provisioning ──▶ Active ──▶ Suspended ──▶ Reactivated         │  │
│  │                      │                         │                │  │
│  │                      ▼                         │                │  │
│  │                 Decommissioned  ◄──────────────┘                │  │
│  └─────────────────────────────────────────────────────────────────┘  │
│                                                                       │
│  ┌─────────────────── Per-Tenant Configuration ───────────────────┐  │
│  │  Feature Toggles  │  Quotas  │  Branding  │  Data Residency   │  │
│  └────────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Tenant Lifecycle** | Tenant Service (Rust + PostgreSQL) | State machine: `provisioning → active → suspended → decommissioned`; reactivation from `suspended` back to `active`; decommissioning triggers data purge; lifecycle events published to all services for synchronisation |
| **PostgreSQL RLS** | All Services (PostgreSQL) | Every table includes a `tenant_id` column; PostgreSQL Row-Level Security policies enforce `tenant_id = current_setting('app.current_tenant')`; RLS policies applied via migration on tenant provisioning; connection-level tenant context set per request |
| **Redis Isolation** | All Services (Redis) | All Redis keys prefixed with `tenant:{tenant_id}:`; namespaced key patterns for sessions, caches, and rate limiters; Redis ACLs prevent cross-tenant access; tenant key scan and purge on decommission |
| **Elasticsearch Isolation** | Platform Service + Search | Per-tenant index aliases (`orders_{tenant_id}`, `customers_{tenant_id}`); index lifecycle management per tenant; query routing via alias; index deletion on decommission |
| **MinIO Isolation** | Platform Service + File Service | Per-tenant bucket or path prefix (`/tenant_id/...`); bucket policies restrict access to tenant prefix; lifecycle rules for retention per tenant; recursive delete on decommission |
| **Context Propagation** | All Services (Rust) | API Gateway injects `X-Tenant-Id` header from authenticated token; middleware in each service extracts and validates tenant context; tenant context set in DB connection, Redis commands, ES queries, and event headers; event messages include `tenant_id` in envelope |
| **Feature Toggles** | Tenant Service (PostgreSQL) | Per-tenant feature flags (enabled/disabled) with rollout percentage; feature check API consumed by all services; toggle changes published as events; default feature set applied on provisioning |
| **Quotas** | Tenant Service (PostgreSQL) | Per-tenant resource quotas: users, storage, API calls/month, documents; usage tracking with periodic aggregation; quota warning at 80%, hard limit at 100%; quota events for billing integration |
| **Data Residency** | Tenant Service (Rust) | Configurable data residency per tenant (region/country); residency rules enforced at provisioning and runtime; data classification marks sensitive fields; region-scoped infrastructure selection; compliance audit logging |
| **Branding** | Tenant Service (PostgreSQL + MinIO) | Per-tenant visual identity: logo, primary/secondary colours, font, favicon; custom login page text; email footer; stored as tenant branding config; served via CDN |

## Tenant Lifecycle States

| State | Description | Allowed Transitions | Actions on Entry |
|-------|-------------|---------------------|------------------|
| `provisioning` | Tenant being set up; resources allocated | → `active` | Create DB schemas, RLS policies, Redis namespace, ES indices, MinIO path, seed default config |
| `active` | Fully operational tenant | → `suspended` | N/A |
| `suspended` | Temporarily disabled; data preserved | → `reactivated`, → `decommissioned` | Block all API access; stop scheduled jobs; preserve all data |
| `reactivated` | Re-enabling a suspended tenant | → `active` | Restore API access; restart scheduled jobs; publish reactivation event |
| `decommissioned` | Permanently removed | (terminal) | Purge all data (DB, Redis, ES, MinIO); revoke all user access; publish decommission event |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Tenant Manager | Tenant CRUD, lifecycle state machine | Tenant Service |
| Provisioning Engine | Resource allocation on tenant creation | Tenant Service |
| Decommission Engine | Resource cleanup and data purge | Tenant Service |
| Feature Toggle Manager | Per-tenant feature flag CRUD and evaluation | Tenant Service |
| Quota Manager | Quota configuration and usage tracking | Tenant Service |
| Residency Enforcer | Data residency rule evaluation | Tenant Service |
| Branding Manager | Tenant visual identity configuration | Tenant Service |
| Context Middleware | Tenant context extraction and propagation | All Services |
| RLS Manager | Row-Level Security policy management | Tenant Service (policies) + All Services (enforcement) |
| Quota Collector | Usage aggregation across services | Tenant Service |

## Isolation Matrix

| Layer | Isolation Mechanism | Scope | Provisioning | Decommission |
|-------|-------------------|-------|-------------|--------------|
| PostgreSQL | RLS policy + `tenant_id` column | Row-level in shared database | Create RLS policies, seed data | DROP tenant data, remove policies |
| Redis | Key prefix `tenant:{id}:` | Key namespace | N/A (prefix is implicit) | SCAN and DEL tenant keys |
| Elasticsearch | Per-tenant index aliases | Separate indices per tenant | Create indices and aliases | DELETE indices |
| MinIO | Path prefix `/tenant_id/` | Object path isolation | Create tenant folder | Recursive delete |
| RabbitMQ | Tenant ID in event envelope | Message-level filtering | N/A (consumer filters) | N/A (purge on decommission) |
| API | X-Tenant-Id header | Request-level context | N/A | Block tenant API key |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| All Services | All Services | Event-driven | Lifecycle events trigger per-service provisioning/decommission |
| Auth Service | Auth Service | Internal API | Tenant context in JWT; tenant-specific auth config |
| Identity Service | Identity Service | Internal API | Per-tenant RBAC/ABAC policies; user-tenant assignment |
| Config Service | Config Service | Internal API | Per-tenant configuration overrides |
| Platform Service (Search) | Platform Service | Events | ES index provisioning/deletion per tenant |
| Platform Service (File) | Platform Service | Events | MinIO path provisioning/purge per tenant |
| Notification Service | Platform Service | Events | Tenant lifecycle notifications to admins |
| Billing (external) | Integration Service | Events | Quota usage for billing; provisioning/decommission triggers |
| Audit Trail | Platform Service | Event-driven | Full tenant management audit log |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `tenant.created` | `{tenant_id, name, plan, region, admin_email, created_at}` | Tenant provisioning completes | All Services (provision resources) |
| `tenant.suspended` | `{tenant_id, reason, suspended_at}` | Tenant suspended | All Services (block access), Notification Service |
| `tenant.reactivated` | `{tenant_id, reactivated_at}` | Tenant reactivated | All Services (restore access), Notification Service |
| `tenant.decommissioned` | `{tenant_id, decommissioned_at}` | Tenant decommissioning completes | All Services (purge data), Notification Service |
| `tenant.feature.changed` | `{tenant_id, feature_key, old_value, new_value, changed_by}` | Feature toggle updated | All Services (feature check cache invalidation) |
| `tenant.quota.warning` | `{tenant_id, quota_type, current_usage, limit, threshold_pct}` | Usage exceeds threshold (80%, 90%) | Notification Service, Billing |

> **Note**: Tenant Service is a core service with no inbox queue. It publishes events but does not consume from the event bus. Services that need tenant data (e.g., default configuration setup on tenant creation) subscribe to Tenant events via their own inbox queues (e.g., `platform.inbox` subscribes to `tenant.created`).

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `tenant` | `id, name, plan, status, region, admin_email, provisioning_details, created_at, updated_at` | Has many `tenant_feature`, `tenant_quota` |
| `tenant_feature` | `id, tenant_id, feature_key, enabled, rollout_pct, config{}` | Belongs to `tenant` |
| `tenant_quota` | `id, tenant_id, quota_type, limit, current_usage, warning_thresholds[], period` | Belongs to `tenant` |
| `tenant_branding` | `id, tenant_id, logo_ref, primary_color, secondary_color, font_family, favicon_ref, login_text, email_footer` | Belongs to `tenant` (1:1) |
| `tenant_residency_rule` | `id, tenant_id, data_classification, allowed_regions[], enforced` | Belongs to `tenant` |
| `tenant_user` | `id, tenant_id, user_id, role, joined_at` | Belongs to `tenant` and user |

## Cross-References

- [Search](search.md) — Per-tenant Elasticsearch index isolation
- [Email](email.md) — Per-tenant email configuration
- [Content Management](content-management.md) — MinIO tenant path isolation
- [Enterprise Collaboration](collaboration.md) — Tenant-scoped channels
- [Architecture Overview](../01-architecture/overview.md) — System context
- [Data Overview](../03-data/overview.md) — RLS implementation patterns
- [Security Overview](../02-security/overview.md) — Tenant isolation security model
- [Tenant Service](../06-services/tenant.md) — Tenant service specification
