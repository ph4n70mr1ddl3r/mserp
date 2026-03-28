# API Marketplace

API catalog, developer portal, and API lifecycle management for internal and external API consumers, managed by the Integration Service.

## Overview

API Marketplace provides a centralized hub for discovering, consuming, and managing all MSERP APIs. It features a self-service developer portal with API key provisioning, interactive documentation, usage analytics, API monetization with tiered access plans, rate limiting per API, version lifecycle management, sandbox environments for testing, and automated API documentation generation from OpenAPI specs. The marketplace supports both internal microservice-to-microservice APIs and external partner/third-party integrations.

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **API Catalog Structure** | Integration Service (Rust + Elasticsearch) | Hierarchical catalog of all MSERP APIs organized by service domain (Finance, Commerce, HCM, CRM, Manufacturing, Project, Platform, Integration); each API entry includes OpenAPI spec, documentation, examples, changelog, and status |
| **Developer Portal Features** | Integration Service (Rust) | Self-service portal with API key provisioning, interactive API explorer (Try-It), code generation (Rust, Python, TypeScript), webhook tester, SDK downloads, and developer onboarding workflows |
| **API Key Provisioning Lifecycle** | Integration Service + Auth Service | Full lifecycle: create → activate → rotate → deprecate → revoke; keys scoped by tenant, API, and environment; automatic rotation reminders and forced rotation on security events |
| **Usage Analytics Dashboards** | Report Service + Integration Service | Per-consumer usage dashboards with request volume, latency percentiles, error rates, quota utilization, and trend analysis; aggregated analytics for API owners |
| **API Monetization Models** | Integration Service + Finance Service | Usage-based billing (per-call), tiered access plans (free, basic, premium, enterprise), custom contracts, revenue sharing for partner APIs; integrated with Finance Service for invoicing |
| **Rate Limiting per API** | Integration Service + API Gateway | Per-API, per-consumer rate limits configurable in marketplace; enforced at API Gateway with standard `X-RateLimit-*` headers; supports burst and sustained rate patterns |
| **Version Management** | Integration Service | Semantic versioning with lifecycle states (alpha, beta, stable, deprecated, sunset); migration guides between versions; sunset schedules with forced upgrade notifications |
| **Sandbox Environments** | Integration Service | Isolated sandbox with synthetic data for API testing; auto-reset on schedule; request/response logging for debugging; shared and dedicated sandbox options |
| **API Documentation Generation** | Integration Service (Rust) | Automated documentation generation from OpenAPI specs; interactive API explorer; request/response examples; error code documentation; SDK auto-generation |

## API Key Provisioning Lifecycle

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  Create   │───▶│  Activate │───▶│  Active   │───▶│ Deprecate│───▶│  Revoke  │
│  Key      │    │  (verify) │    │  (in use) │    │  (sunset)│    │  (expired)│
│           │    │           │    │           │    │          │    │          │
│ Generate  │    │ Email     │    │ Rate      │    │ Grace    │    │ Hard     │
│ secret +  │    │ verify    │    │ enforced  │    │ period + │    │ block    │
│ scopes    │    │ owner     │    │ Analytics │    │ warnings │    │ all calls│
└──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘
                                       │
                                       ▼
                                ┌──────────┐
                                │  Rotate   │
                                │  (refresh │
                                │   secret) │
                                └──────────┘
```

## Monetization Models

| Plan | Price | Rate Limit | Features | Support |
|------|-------|-----------|----------|---------|
| **Free** | $0/month | 100 req/min | Core APIs only, community support | Forum |
| **Basic** | $99/month | 1,000 req/min | All APIs, usage analytics | Email (48h SLA) |
| **Premium** | $499/month | 5,000 req/min | All APIs, sandbox, webhooks, analytics priority | Email (24h SLA) |
| **Enterprise** | Custom | Custom | All APIs, dedicated sandbox, custom rate limits, SLA | Dedicated (4h SLA) |
| **Pay-per-use** | Per-call pricing | Burst-capable | All APIs, real-time usage tracking | Email (24h SLA) |

## Version Lifecycle States

| State | Description | Consumer Impact | Duration |
|-------|-------------|----------------|----------|
| `alpha` | Early preview, breaking changes possible | Opt-in only, no SLA | Indefinite |
| `beta` | Feature-complete, may change | Recommended for testing only | 3-6 months |
| `stable` | Production-ready, backward-compatible guarantees | Full SLA, recommended for production | 12+ months |
| `deprecated` | Replaced by newer version, still functional | Migration warnings in response headers | 6 months |
| `sunset` | No longer accepting requests | All calls return 410 Gone with migration guide | Permanent |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| API Gateway | Traefik/Kong | Configuration | Rate limit and routing configuration per API |
| Auth Service | Auth Service | Internal API | API key validation, OAuth2 scopes, JWT issuance |
| Finance Service | Finance Service | Internal API | Monetization billing, invoice generation |
| Report Service | Report Service | Internal API | Usage analytics dashboards and reports |
| Event Mesh | Integration Service | Internal API | Event-driven API documentation and discovery |
| Notification Service | Platform Service | Event-driven | Key lifecycle notifications, deprecation alerts |
| All Business Services | All Services | OpenAPI spec | API spec registration from service metadata |
| Audit Trail | Platform Service | Event-driven | API access and key management audit log |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `marketplace.api.registered` | `{api_id, service, version, lifecycle_state}` | New API version registered | Catalog, Developer Portal |
| `marketplace.api.deprecated` | `{api_id, version, replacement_version, sunset_date}` | API version marked deprecated | Notification Service, Consumers |
| `marketplace.api.sunset` | `{api_id, version, sunset_at}` | API version reaches sunset | Audit Trail, Consumers |
| `marketplace.key.created` | `{key_id, consumer_id, scopes[], plan}` | New API key provisioned | Audit Trail, Auth Service |
| `marketplace.key.rotated` | `{key_id, rotated_by, rotated_at}` | API key secret rotated | Audit Trail, Consumer |
| `marketplace.key.revoked` | `{key_id, reason, revoked_at}` | API key revoked | Audit Trail, Auth Service, Consumer |
| `marketplace.usage.threshold` | `{consumer_id, api_id, usage_percent, threshold}` | Usage quota threshold reached | Notification Service, Consumer |
| `marketplace.subscription.created` | `{subscription_id, consumer_id, plan, start_date}` | New subscription created | Finance Service, Notification |
| `marketplace.subscription.cancelled` | `{subscription_id, consumer_id, end_date}` | Subscription cancelled | Finance Service, Notification |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `auth.user.registered` | Auth Service | Create developer portal account |
| `finance.invoice.generated` | Finance Service | Link usage to invoice |
| `integration.connector.registered` | Integration Service | Publish connector API to catalog |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `api_catalog_entry` | `id, service, name, version, lifecycle_state, openapi_spec_ref` | Belongs to service, has many `api_catalog_version` |
| `api_catalog_version` | `id, entry_id, version, changelog, migration_guide, registered_at` | Belongs to `api_catalog_entry` |
| `api_key` | `id, consumer_id, secret_hash, scopes, plan, status, created_at, expires_at` | Belongs to `api_consumer` |
| `api_consumer` | `id, tenant_id, name, email, plan, subscription_id, created_at` | Has many `api_key`, `api_usage_daily` |
| `api_usage_daily` | `id, consumer_id, api_id, date, request_count, error_count, avg_latency_ms` | Belongs to `api_consumer`, `api_catalog_entry` |
| `api_subscription` | `id, consumer_id, plan, status, start_date, end_date, billing_amount` | Belongs to `api_consumer` |
| `api_rate_limit` | `id, api_id, plan, requests_per_minute, burst_size` | Belongs to `api_catalog_entry` |

## Cross-References

- [Event Mesh](event-mesh.md) — Event-driven integration backbone
- [API Design Standards](../api/standards.md) — API design conventions and standards
- [API Endpoints](../api/endpoints.md) — Complete endpoint listings
- [Integration Service](../services/integration.md) — Integration service specification
- [Architecture Overview](../architecture/overview.md) — System context
- [NFR: Rate Limiting](../planning/nfr.md) — Rate limit specifications
