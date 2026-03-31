# Caching Strategy

## 1. Cache Architecture

MSERP uses **cache-aside** pattern with Redis as the caching layer.

### 1.1 Cache Policies by Data Type

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
| Golden records (MDM) | Cache-aside | 15 minutes | On `integration.master-data.merged` event |
| ESG emissions factors | Cache-aside | 24 hours | On `config.changed` (ESG keys) |
| Loyalty program data | Cache-aside | 15 minutes | On `commerce.loyalty.*` events |
| Tax rate tables | Cache-aside | 1 hour | On `finance.tax.*` events |
| Commodity market prices | Cache-aside | 5 minutes | On `finance.commodity.price.updated` event |
| Compliance control mappings | Cache-aside | 1 hour | On `platform.compliance-hub.*` events |
| Digital twin state | Write-through | 30 seconds | On `manufacturing.digital-twin.state.updated` event |
| Contact center queue status | Cache-aside | 10 seconds | On `crm.contact-center.*` events |
| Supplier diversity classifications | Cache-aside | 30 minutes | On `finance.diversity.*` events |
| Data quality scorecards | Cache-aside | 15 minutes | On `integration.data-quality.*` events |
| Dynamic discount programs | Cache-aside | 30 minutes | On `finance.dynamic-discount.offer-created` event |
| Planning scenarios | Cache-aside | 5 minutes | On `report.planning.scenario.*` events |
| Planning assumptions | Cache-aside | 15 minutes | On `report.planning.driver.updated` event |
| Supply chain collaboration scorecards | Cache-aside | 30 minutes | On `commerce.collaboration.*` events |
| Report templates | Cache-aside | 1 hour | On `finance.report.template-created` event |
| Supplier collaboration connections | Cache-aside | 15 minutes | On `commerce.collaboration.*` events |
| Production schedule cache | Cache-aside | 5 minutes | On `manufacturing.schedule.#` events |
| Orchestration rules cache | Cache-aside | 10 minutes | On `commerce.orchestration.rule-evaluated` events |
| Storefront pages cache | Cache-aside | 15 minutes | On `commerce.storefront.page.published` events |
| ML Studio experiment results | Cache-aside | 30 minutes | On `report.ml-studio.model.trained` events |
| Certification campaign status | Cache-aside | 15 minutes | On `platform.grc.certification.#` events |
| Custom field definitions | Cache-aside | 30 minutes | On `platform.composer.extension.created` events |

Cache key format is defined in SPEC.md §8.1. The `{tenant_id}` segment uses the literal string `global` for system-wide, tenant-agnostic data.

### 1.2 Cache Invalidation

- **Event-driven**: Services subscribe to relevant domain events and invalidate affected cache entries.
- **TTL-based**: All cached entries have a TTL as a safety net against stale data.
- **Manual flush**: Tenant Admins can trigger cache flush for their tenant via `POST /api/v1/config/cache/flush`.

### 1.3 Cold Start Behavior

- On service startup, caches are empty (cold).
- First requests are served directly from the database.
- Caches warm up naturally as requests arrive.
- For critical reference data, services MAY pre-warm caches on startup via a configurable list of pre-warm queries.

---

*See [Data Architecture Overview](overview.md) for database patterns and schema elements.*
*See [Data Pipeline](data-pipeline.md) for the analytical pipeline (data lake + warehouse).*
*See [Domain Data Models](domain-models.md) for domain-specific data models.*
