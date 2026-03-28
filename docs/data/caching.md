# Caching Strategy

## 3. Caching Strategy

### 3.1 Cache Architecture

MSERP uses **cache-aside** pattern with Redis as the caching layer.

### 3.2 Cache Policies by Data Type

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

### 3.3 Cache Key Format

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
- `mserp:tenant-123:integration:golden_record:customer:cust-001`
- `mserp:tenant-123:commerce:loyalty:program:prog-001`
- `mserp:tenant-123:finance:tax_rates:US-CA`
- `mserp:tenant-123:finance:commodity:price:GOLD`
- `mserp:tenant-123:platform:compliance:controls:SOC2`
- `mserp:tenant-123:manufacturing:digital_twin:asset-456`

### 3.4 Cache Invalidation

- **Event-driven**: Services subscribe to relevant domain events and invalidate affected cache entries.
- **TTL-based**: All cached entries have a TTL as a safety net against stale data.
- **Manual flush**: Tenant Admins can trigger cache flush for their tenant via `POST /api/v1/config/cache/flush`.

### 3.5 Cold Start Behavior

- On service startup, caches are empty (cold).
- First requests are served directly from the database.
- Caches warm up naturally as requests arrive.
- For critical reference data, services MAY pre-warm caches on startup via a configurable list of pre-warm queries.

---

*See [Data Architecture Overview](overview.md) for database patterns and schema elements.*
*See [Data Warehouse](warehouse.md) for analytical schema design.*
*See [Data Lake](data-lake.md) for raw and curated data zones.*
*See [Master Data Management Schema](mdm.md) for MDM and related schemas.*
*See [Domain Data Models](domain-models.md) for domain-specific data models.*
