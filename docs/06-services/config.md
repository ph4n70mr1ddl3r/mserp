# Config Service

| Aspect | Details |
|--------|---------|
| Port | 8004 |
| Database | `config_db` |
| Responsibilities | System configuration, settings management |

## Features

Dynamic config with hot reload, environment variable overrides (internal only, not API-exposed), CORS origin management, localization string management, scheduled maintenance windows

## Configuration Schema

Configuration is hierarchical, typed, and scoped per tenant with fallback to global defaults.

**Configuration Hierarchy (highest to lowest precedence):**

```
1. Environment variables (MSERP_{SECTION}_{KEY}) — internal overrides, not exposed via API
2. Tenant-specific overrides (config_db)
3. Global defaults (config_db)
4. Compiled-in defaults (Rust code)
```

**Configuration Categories:**

| Category | Scope | Examples |
|----------|-------|---------|
| System | Global | `system.max_upload_size_mb`, `system.default_locale`, `system.maintenance_mode` |
| Service | Global | `commerce.order.auto_cancel_hours`, `finance.fiscal_year_start` |
| Tenant | Per-tenant | Overrides of service/system config per tenant |
| Localization | Per-tenant | `i18n.date_format`, `i18n.number_format`, `i18n.timezone` |
| ESG | Per-tenant | `esg.reporting_framework`, `esg.carbon_offset_enabled`, `esg.emission_factors_source` |

## Config Propagation

- Services poll Config Service every 30 seconds for changes (config key: `config.poll_interval_seconds`).
- Critical config changes (e.g., `system.maintenance_mode`) are pushed immediately via RabbitMQ event `config.changed`.
- Services cache config in-memory with a TTL matching the poll interval.
- Config changes are immutable — every change is append-only in the audit log.

## Configuration API

**CRUD Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/config` | List all config entries (query params: `scope`, `category`, `tenant_id`) |
| `GET` | `/config/{key}` | Get single config entry with resolved value for the requesting tenant |
| `PUT` | `/config/{key}` | Create or update a config entry (body: `{ value, type, scope, tenant_id?, reason }`) |
| `DELETE` | `/config/{key}` | Delete a tenant-level override (global keys cannot be deleted via API) |
| `GET` | `/config/audit` | List audit trail entries (query params: `key`, `changed_by`, `from`, `to`) |
| `POST` | `/config/cache/flush` | Invalidate server-side config cache and notify all services via `config.changed` |

**CORS Origin Management:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/config/cors/origins` | List allowed CORS origins for a tenant (query param: `tenant_id`) |
| `POST` | `/config/cors/origins` | Add a CORS origin (body: `{ tenant_id, origin }`); validates origin is a valid URL |
| `DELETE` | `/config/cors/origins` | Remove a CORS origin (body: `{ tenant_id, origin }`) |

**Localization Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/config/locales` | List all supported locales with display names and RTL flag |
| `GET` | `/config/locales/{locale}/translations` | Get all translation strings for a locale (query param: `tenant_id` for overrides) |
| `PUT` | `/config/locales/{locale}/translations/{key}` | Create or update a translation string (body: `{ value, tenant_id? }`) |

## Validation Rules

| Rule | Description |
|------|-------------|
| Type validation | Values must match the declared type: `string`, `integer`, `boolean`, `json`, `url` |
| Range validation | Numeric values may define `min` and `max` bounds in `config_schema` |
| Key naming | Keys must be dot-separated lowercase with alphanumeric segments (regex: `[a-z][a-z0-9]*(\.[a-z][a-z0-9]*)*`) |
| Value size limit | String values: max 10,000 characters; JSON values: max 100 KB |
| Required fields | Schema may mark a key as `required` — deletion of required keys is rejected |
| URL validation | Type `url` values must parse as valid `http` or `https` URLs |
| Immutable keys | Keys marked `immutable: true` in schema cannot be updated after creation |

## Audit Trail

- Every config change (create, update, delete) is recorded as an immutable append-only entry.
- Audit entries cannot be modified or deleted, even by admins.

| Audit Field | Type | Description |
|-------------|------|-------------|
| `id` | UUID | Unique audit entry ID |
| `key` | String | The config key that was changed |
| `old_value` | Text | Previous value (null for creates) |
| `new_value` | Text | New value (null for deletes) |
| `scope` | Enum | `global` or `tenant` |
| `tenant_id` | UUID | Tenant ID if tenant-scoped, null if global |
| `changed_by` | UUID | User ID of the actor |
| `changed_at` | Timestamp | ISO 8601 UTC timestamp |
| `reason` | Text | Mandatory reason for the change (min 10 characters) |
| `source` | Enum | `api`, `env_override`, `migration` |

## Localization Management

| Aspect | Details |
|---------|---------|
| Translation key format | Dot-separated lowercase: `module.section.label` (e.g., `commerce.order.confirm_button`) |
| Supported locales | `en`, `es`, `fr`, `de`, `pt`, `ja`, `zh-CN`, `zh-TW`, `ko`, `ar`, `hi`, `ru`, `it`, `nl` |
| Fallback chain | Tenant locale → `en` (English) — if a key is missing in the tenant locale, the English value is returned |
| RTL support | `ar` and `he` locales are flagged as RTL; frontend reads `direction` from locale metadata |
| Plural rules | ICU MessageFormat plural syntax supported (e.g., `{count, plural, one {# item} other {# items}}`) |
| Interpolation | Variables in translation values use `{variable_name}` syntax; validated at save time |

- Tenant-specific translation overrides are stored with `tenant_id` and take precedence over the base locale strings.
- Missing translation keys are logged as warnings and surfaced in a translation coverage dashboard.

## Maintenance Windows

| Attribute | Type | Description |
|-----------|------|-------------|
| `enabled` | Boolean | Whether maintenance mode is active |
| `message` | String | Banner text displayed to all users (supports localization keys) |
| `scheduled_start` | Timestamp | Planned maintenance start time (ISO 8601) |
| `scheduled_end` | Timestamp | Planned maintenance end time (ISO 8601) |
| `allowed_ips` | String[] | IP allowlist for admin access during maintenance |
| `retry_after_seconds` | Integer | Value for `Retry-After` header during maintenance |

**API Behavior During Maintenance:**

- All non-admin API requests receive `503 Service Unavailable` with `Retry-After` header.
- Requests from IPs in `allowed_ips` are served normally.
- The maintenance banner is served from `GET /public/maintenance` (unauthenticated, used by frontend).
- When `system.maintenance_mode` is set to `true`, a `config.changed` event is published immediately so all services activate maintenance behavior without waiting for the poll interval.

## Database Tables

| Table | Key Columns | Description |
|-------|-------------|-------------|
| `config_entries` | `key`, `value`, `type`, `scope`, `tenant_id`, `updated_at` | Current config values with scope resolution |
| `config_audit` | `id`, `key`, `old_value`, `new_value`, `scope`, `tenant_id`, `changed_by`, `changed_at`, `reason`, `source` | Immutable append-only change history |
| `config_schema` | `key`, `type`, `required`, `immutable`, `min`, `max`, `pattern`, `default_value` | Validation rules and metadata per config key |
| `localization_strings` | `locale`, `key`, `value`, `tenant_id`, `updated_at` | Translation strings (base + tenant overrides) |
| `cors_origins` | `id`, `tenant_id`, `origin`, `created_at`, `created_by` | Allowed CORS origins per tenant |

## Events Published

| Event | Payload | Trigger |
|-------|---------|---------|
| `config.changed` | `{ key, old_value, new_value, scope, tenant_id?, changed_by }` | Any config entry is created, updated, or deleted |
| `config.cache.flushed` | `{ triggered_by }` | Manual cache flush via API |

## See Also

- [Tenant Service](tenant.md)
- [Architecture Overview](../01-architecture/overview.md)
