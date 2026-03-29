# Search

Elasticsearch-based full-text search with unified cross-domain querying, faceted navigation, and per-tenant index isolation, managed by the Platform Service.

## Overview

The Search feature provides a unified search infrastructure across all MSERP services. Built on Elasticsearch, it supports full-text search with faceted navigation, search suggestions and autocomplete, per-tenant index isolation for multi-tenancy, and relevance tuning with configurable boosting. Each service publishes indexable content via events or API calls, and the Search service maintains a consolidated index that enables users to find records across domains from a single search interface. Relevance models are tunable per tenant, and search analytics inform continuous improvement.

## Architecture Diagram

```
┌──────────────────────────────────────────────────────────────────────┐
│                        Search Infrastructure                          │
│                                                                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │
│  │ Commerce     │  │ Finance      │  │ CRM          │               │
│  │ Service      │  │ Service      │  │ Service      │               │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘               │
│         │                 │                  │                        │
│         └─────────────────┼──────────────────┘                        │
│                           ▼                                          │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │                   Index Ingestion Pipeline                      │  │
│  │  Event-driven and API-based content ingestion; normalisation,  │  │
│  │  enrichment, and bulk indexing into Elasticsearch               │  │
│  └────────────────────────────┬───────────────────────────────────┘  │
│                               ▼                                      │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │                     Elasticsearch Cluster                       │  │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐               │  │
│  │  │ Tenant A   │  │ Tenant B   │  │ Tenant N   │               │  │
│  │  │ Indices    │  │ Indices    │  │ Indices    │               │  │
│  │  └────────────┘  └────────────┘  └────────────┘               │  │
│  └────────────────────────────┬───────────────────────────────────┘  │
│                               ▼                                      │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │                      Query Layer                                │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐ │  │
│  │  │ Unified      │  │ Faceted      │  │ Suggestions &        │ │  │
│  │  │ Search API   │  │ Navigation   │  │ Autocomplete         │ │  │
│  │  └──────────────┘  └──────────────┘  └──────────────────────┘ │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐ │  │
│  │  │ Relevance    │  │ Tenant       │  │ Analytics &          │ │  │
│  │  │ Tuning       │  │ Scoping      │  │ Logging              │ │  │
│  │  └──────────────┘  └──────────────┘  └──────────────────────┘ │  │
│  └────────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Index Management** | Platform Service (Rust + Elasticsearch) | Per-tenant index isolation using index alias pattern (`{index}_{tenant_id}`); index lifecycle management (ILM) for rollover and retention; mapping templates per domain type (orders, invoices, customers, products, etc.) |
| **Ingestion Pipeline** | Platform Service (Rust) | Dual-mode ingestion: event-driven (consumes domain events from RabbitMQ for real-time updates) and API-based (for bulk reindex and backfill); content normalisation, enrichment (tags, categories), and bulk indexing via Elasticsearch Bulk API |
| **Query Engine** | Platform Service (Rust) | Unified search API accepting free-text queries with optional domain filters; multi-match queries across configured fields; phrase matching, fuzzy matching, and wildcard support; tenant-scoped query execution |
| **Faceted Navigation** | Platform Service (Rust + Elasticsearch) | Aggregation-based facets for domain type, status, date ranges, categories, and custom fields; dynamic facet computation based on query results; configurable facet display per domain |
| **Suggestions & Autocomplete** | Platform Service (Rust + Elasticsearch) | Completion suggester for real-time autocomplete; term and phrase suggesters for did-you-mean functionality; suggestion ranking based on historical click-through data |
| **Relevance Tuning** | Platform Service (Rust) | Configurable field boosting weights per tenant and domain; freshness boosting for recent content; popularity boosting based on click-through rates; function score queries for custom relevance models |
| **Tenant Isolation** | Platform Service (Rust) | All queries scoped to tenant via index alias; no cross-tenant data leakage; per-tenant index settings and analyzers (language-specific) |
| **Analytics** | Platform Service (Rust + PostgreSQL) | Query logging with search terms, result counts, clicked results, and zero-result queries; search effectiveness dashboards; suggestion for synonym and boosting improvements |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Index Manager | Index creation, mapping, lifecycle, aliases | Platform Service |
| Ingestion Pipeline | Content ingestion, normalisation, bulk indexing | Platform Service |
| Event Consumer | Domain event processing for real-time index updates | Platform Service |
| Query Engine | Search query parsing, execution, tenant scoping | Platform Service |
| Facet Engine | Aggregation computation for faceted navigation | Platform Service |
| Suggestion Engine | Autocomplete and did-you-mean suggestions | Platform Service |
| Relevance Manager | Boosting config, scoring models, A/B testing | Platform Service |
| Analytics Collector | Query logging, click tracking, effectiveness metrics | Platform Service |
| Reindex Manager | Full and partial reindex operations | Platform Service |
| Tenant Index Isolation | Per-tenant index routing and access control | Platform Service |

## Searchable Domains

| Domain | Source Service | Key Fields Indexed | Facets |
|--------|---------------|--------------------|--------|
| Orders | Commerce Service | order_no, customer, items, status, total, dates | status, date range, customer |
| Products | Commerce Service | sku, name, description, category, attributes | category, price range, attributes |
| Invoices | Finance Service | invoice_no, vendor/customer, amount, status, dates | status, date range, amount range |
| Customers | CRM Service | name, email, phone, company, tags | industry, region, status |
| Leads/Opportunities | CRM Service | name, company, value, stage, dates | stage, owner, date range |
| Projects | Project Service | name, code, status, manager, dates | status, manager, date range |
| Employees | HCM Service | name, email, department, title | department, title, location |
| Knowledge Articles | Platform Service | title, content, category, tags | category, rating, date range |
| Documents | Platform Service | filename, content (extracted), type, tags | type, date range, owner |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Commerce Service | Commerce Service | Events | Index orders, products, inventory |
| Finance Service | Finance Service | Events | Index invoices, payments, budgets |
| CRM Service | CRM Service | Events | Index customers, leads, opportunities, cases |
| HCM Service | HCM Service | Events | Index employees, candidates |
| Project Service | Project Service | Events | Index projects, tasks, milestones |
| Manufacturing Service | Manufacturing Service | Events | Index work orders, BOMs, quality records |
| Content Management | Platform Service | Events | Index documents, knowledge articles |
| Digital Assistant | Platform Service | Internal API | Search queries from assistant |
| Collaboration | Platform Service | Internal API | Search from collaboration channels |
| Report Service | Report Service | Internal API | Search analytics for dashboards |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `platform.search.index.updated` | `{index_name, tenant_id, document_count, updated_at}` | Index refresh or reindex completes | Analytics, Audit Trail |
| `platform.search.query.executed` | `{tenant_id, query, domains[], result_count, duration_ms, user_id}` | Search query executed | Analytics |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `commerce.order.*` | Commerce Service | Update order index |
| `commerce.product.*` | Commerce Service | Update product index |
| `finance.invoice.*` | Finance Service | Update invoice index |
| `crm.customer.*` | CRM Service | Update customer index |
| `crm.lead.*` / `crm.opportunity.*` | CRM Service | Update lead/opportunity index |
| `project.project.*` | Project Service | Update project index |
| `hr.employee.*` | HCM Service | Update employee index |
| `platform.document.*` | Platform Service | Update document index |
| `platform.article.*` | Platform Service | Update knowledge article index |
| `tenant.created` | Tenant Service | Provision new tenant indices |
| `tenant.decommissioned` | Tenant Service | Delete tenant indices |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `search_index` | `id, tenant_id, domain, index_alias, document_count, last_updated, mapping_version` | Per-tenant, per-domain |
| `search_config` | `id, tenant_id, domain, boost_weights{}, facet_fields[], analyzer, is_active` | Per-tenant, per-domain |
| `search_suggestion` | `id, tenant_id, domain, input[], weight, output, context` | Per-tenant, per-domain |
| `search_query_log` | `id, tenant_id, user_id, query, domains[], result_count, clicked_doc_id, duration_ms, executed_at` | Per-query entry |
| `search_synonym` | `id, tenant_id, domain, terms[], is_active` | Per-tenant, per-domain |

## Cross-References

- [Multi-Tenancy](multi-tenancy.md) — Per-tenant index isolation
- [Digital Assistant](digital-assistant.md) — Search queries from assistant
- [Enterprise Collaboration](collaboration.md) — Unified search in collaboration
- [Knowledge Management](../06-services/platform.md) — Knowledge article indexing (Knowledge Management module)
- [Content Management](content-management.md) — Document content indexing
- [Architecture Overview](../01-architecture/overview.md) — System context
- [Platform Service](../06-services/platform.md) — Platform service specification
