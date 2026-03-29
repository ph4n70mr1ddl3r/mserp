# Tenant Service

| Aspect | Details |
|--------|---------|
| Port | 8003 |
| Database | `tenant_db` |
| Responsibilities | Multi-tenancy, tenant provisioning, feature flags |

## Features

Tenant lifecycle management (provisioning, active, suspended, decommissioned), subscription tier management, feature toggles with rollout strategies, tenant-specific branding, tenant quota enforcement (users, storage, API calls), data residency controls (region pinning per tenant)

## Tenant Lifecycle

| State | Description |
|-------|-------------|
| Provisioning | Initial state after signup; resources (database schema, storage bucket, default config) are being allocated |
| Active | Tenant is fully operational; all API endpoints available |
| Suspended | Tenant is read-only; login blocked for non-admin users; triggered by billing failure or admin action |
| Decommissioned | Tenant data is queued for deletion after retention period; all API calls return 410 Gone |

**State Transitions:**

| From | To | Trigger |
|------|----|---------|
| Provisioning | Active | All resources allocated and health checks pass |
| Active | Suspended | Billing payment failure after grace period (7 days), or admin manual suspension |
| Suspended | Active | Billing payment resolved, or admin reactivation |
| Suspended | Decommissioned | Retention period expires (default 90 days) with no reactivation |
| Active | Decommissioned | Admin-initiated termination (requires confirmation flow) |

- Provisioning timeout: if resources are not ready within 10 minutes, the tenant is moved to `decommissioned` and the signup is retried.
- Decommissioned data is hard-deleted after the retention window via a daily cron job.
- All state transitions emit events (see Events Published below).

## Subscription Tiers

| Feature | Standard | Professional | Enterprise |
|---------|----------|-------------|------------|
| Users | 50 | 500 | Unlimited |
| Storage | 10 GB | 100 GB | 1 TB |
| API calls/min | 1,000 | 10,000 | 100,000 |
| Custom integrations | 3 | 25 | Unlimited |
| Custom applications (low-code) | 1 | 10 | Unlimited |
| Data warehouse storage | 5 GB | 50 GB | 500 GB |
| SSO providers | 1 | 3 | Unlimited |
| Audit log retention | 30 days | 1 year | Unlimited |
| Custom branding | — | Logo + colors | Full (logo, colors, domain, email templates) |
| Data residency control | Default region only | Configurable region | Multi-region with replication |
| SLA | 99.5% | 99.9% | 99.99% |
| Support | Email (48h response) | Email + chat (8h response) | Dedicated account manager (1h response) |

## Feature Flags

| Flag Type | Description |
|-----------|-------------|
| Boolean | On/off toggle for a feature (e.g., `dark_mode`) |
| Percentage Rollout | Gradual rollout to a percentage of users (e.g., `new_ui` at 25%) |
| User Segment | Enabled for specific user segments or individual user IDs |

**Evaluation Flow:**

1. Request arrives with `X-Tenant-Id` and `X-User-Id` headers.
2. Flag evaluator checks overrides in order: user-level → tenant-level → global default.
3. For percentage rollouts, a deterministic hash of `(flag_key, user_id)` determines inclusion.
4. Result is cached per flag key for the duration of the config poll interval (30s).

**Example Flags:**

| Flag Key | Type | Default | Description |
|----------|------|---------|-------------|
| `dark_mode` | Boolean | `false` | Enable dark mode UI |
| `new_ui` | Percentage | `0%` | New navigation layout rollout |
| `ai_features` | User Segment | — | AI-powered suggestions (Enterprise only) |
| `advanced_analytics` | User Segment | — | Advanced analytics dashboard (Professional+) |

## Quota Enforcement

- Quota checks are performed in a shared middleware layer that all services include.
- The middleware calls Tenant Service (`GET /internal/tenants/{id}/quotas`) on a cache-through basis (TTL = 60s).
- Current usage is incremented atomically via `POST /internal/tenants/{id}/quotas/{resource}/consume`.

**Response Headers (on every API response):**

| Header | Example | Description |
|--------|---------|-------------|
| `X-Quota-Resource` | `api_calls` | The quota resource being measured |
| `X-Quota-Usage` | `842` | Current usage for the period |
| `X-Quota-Limit` | `1000` | Maximum allowed for the period |
| `X-Quota-Remaining` | `158` | Remaining quota |

**Overage Behavior:**

| Resource | Overage Policy |
|----------|---------------|
| API calls/min | **Reject** — return 429 Too Many Requests with `Retry-After` header |
| Users | **Reject** — return 403 Forbidden on user creation |
| Storage | **Warn** — accept writes, emit `tenant.quota.warning` event at 80% and 95%, admin notified |
| Data warehouse storage | **Reject** — return 507 Insufficient Storage |

## Data Residency

- Each tenant is pinned to a region at provisioning time (stored in `tenants.region`).
- The API gateway reads `X-Tenant-Id`, resolves the tenant's region, and routes the request to the correct regional service instance.
- Cross-region data transfer is prohibited: a tenant pinned to `eu-west-1` cannot have data stored or processed in `us-east-1`.
- Compliance frameworks are tracked per tenant to enforce additional residency constraints.

| Supported Region | Compliance Frameworks |
|------------------|-----------------------|
| `us-east-1` | SOC 2, HIPAA |
| `eu-west-1` | GDPR, SOC 2 |
| `ap-southeast-1` | PDPA, SOC 2 |
| `me-south-1` | PDPL |

- Region changes require a migration request processed by the Platform Service job scheduler. Migration copies all tenant data and updates the routing table. The tenant is in read-only mode during migration.

## Tenant Branding

| Attribute | Type | Description |
|-----------|------|-------------|
| `logo_url` | URL | Tenant logo (max 2 MB, SVG or PNG) |
| `primary_color` | String | Hex color for UI accent (#RRGGBB) |
| `secondary_color` | String | Hex color for UI secondary elements |
| `favicon_url` | URL | Custom favicon |
| `custom_domain` | String | CNAME target (e.g., `app.customer.com`); requires DNS verification |
| `email_from_name` | String | Display name for outgoing transactional emails |
| `email_from_address` | String | Reply-to address for outgoing emails (must be verified) |

- Custom domain provisioning: tenant adds a CNAME record → Tenant Service verifies ownership via DNS TXT challenge → TLS certificate is provisioned automatically via ACME.
- Branding values are served at `GET /public/tenants/{slug}/branding` (unauthenticated, used by frontend shell).
- If a branding attribute is not set, the global platform default is used.

## Database Tables

| Table | Key Columns | Description |
|-------|-------------|-------------|
| `tenants` | `id`, `name`, `slug`, `tier`, `status`, `region`, `branding_json`, `created_at`, `updated_at` | Core tenant record |
| `subscriptions` | `id`, `tenant_id`, `tier`, `billing_provider`, `billing_customer_id`, `current_period_start`, `current_period_end`, `renewal_auto` | Active subscription and billing details |
| `feature_flags` | `id`, `key`, `type`, `value`, `rollout_config_json`, `scope` (global/tenant/user), `enabled` | Feature flag definitions |
| `feature_flag_overrides` | `id`, `flag_id`, `tenant_id`, `user_id`, `value`, `updated_by` | Per-tenant and per-user flag overrides |
| `tenant_quotas` | `id`, `tenant_id`, `resource`, `current_usage`, `limit`, `period_start` | Quota tracking per resource per billing period |
| `data_residency_rules` | `id`, `tenant_id`, `region`, `compliance_frameworks`, `enforced` | Data residency and compliance constraints |
| `tenant_branding` | `id`, `tenant_id`, `logo_url`, `primary_color`, `secondary_color`, `custom_domain`, `email_from_name`, `email_from_address` | Branding configuration |

## Events Published

| Event | Payload | Trigger |
|-------|---------|---------|
| `tenant.created` | `{ tenant_id, name, tier, region }` | Tenant finishes provisioning and becomes active |
| `tenant.updated` | `{ tenant_id, changed_fields[] }` | Any tenant attribute is modified |
| `tenant.suspended` | `{ tenant_id, reason }` | Tenant enters suspended state |
| `tenant.reactivated` | `{ tenant_id }` | Tenant returns to active from suspended |
| `tenant.decommissioned` | `{ tenant_id, data_deletion_date }` | Tenant enters decommissioned state |
| `tenant.feature.changed` | `{ tenant_id, flag_key, old_value, new_value, scope }` | Feature flag override is added or modified |
| `tenant.quota.warning` | `{ tenant_id, resource, current_usage, limit, percentage }` | Quota usage exceeds 80% or 95% |

## See Also

- [Auth Service](auth.md)
- [Identity Service](identity.md)
- [Config Service](config.md)
- [Architecture Overview](../01-architecture/overview.md)
