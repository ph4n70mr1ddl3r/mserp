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

## See Also

- [Tenant Service](tenant.md)
- [Architecture Overview](../architecture/overview.md)
