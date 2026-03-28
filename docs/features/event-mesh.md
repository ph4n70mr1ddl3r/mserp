# Event Mesh

Event backbone for cross-system event routing, filtering, and integration, managed by the Integration Service.

## Overview

Event Mesh provides the central nervous system for event-driven communication across MSERP services and external systems. Built on RabbitMQ with topic-based routing, it offers an HTTP-to-event gateway for external publishers/subscribers, a schema registry for contract management, content-based event filtering, cross-region event replication, and comprehensive monitoring. The mesh supports both choreography-based internal events and orchestrated cross-system integration patterns, enabling reliable, scalable, and observable event-driven architectures.

## Mesh Topology Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Event Mesh Topology                         │
│                                                                      │
│  ┌──────────────┐                          ┌──────────────┐        │
│  │ External Sys │                          │ External Sys │        │
│  │ (HTTP)       │                          │ (Webhook)    │        │
│  └──────┬───────┘                          └──────┬───────┘        │
│         │                                         │                 │
│  ┌──────▼─────────────────────────────────────────▼───────┐        │
│  │                   Event Gateway                         │        │
│  │  HTTP-to-Event Bridge │ Authentication │ Rate Limiting │        │
│  │  Request Validation   │ Transform      │ Circuit Breaker│       │
│  └──────────────────────────┬──────────────────────────────┘        │
│                              │                                       │
│  ┌──────────────────────────▼──────────────────────────────┐        │
│  │                 Schema Registry                          │        │
│  │  Schema Versions │ Compatibility Checks │ Evolution Rules│       │
│  └──────────────────────────┬──────────────────────────────┘        │
│                              │                                       │
│  ┌──────────────────────────▼──────────────────────────────┐        │
│  │              Topic Hierarchy Manager                      │        │
│  │  {domain}.{entity}.{action}  │ Wildcards │ Namespace Isolation│   │
│  └──────────────────────────┬──────────────────────────────┘        │
│                              │                                       │
│  ┌──────────────────────────▼──────────────────────────────┐        │
│  │              Event Broker (RabbitMQ Cluster)              │        │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐│       │
│  │  │finance.│ │commerce│ │  hcm.  │ │  crm.  │ │mfg.    ││       │
│  │  │ *      │ │ .*     │ │  *     │ │  *     │ │ *      ││       │
│  │  └───┬────┘ └───┬────┘ └───┬────┘ └───┬────┘ └───┬────┘│       │
│  │      │          │          │          │          │      │       │
│  │  ┌───▼──────────▼──────────▼──────────▼──────────▼───┐ │       │
│  │  │          Event Filter / Router                      │ │       │
│  │  │  Content-based │ Correlation │ Enrichment │ Split  │ │       │
│  │  └────────────────────────────────────────────────────┘ │       │
│  └──────────────────────────────────────────────────────────┘       │
│                              │                                       │
│         ┌────────────────────┼────────────────────┐                 │
│         │                    │                    │                  │
│  ┌──────▼──────┐  ┌─────────▼──────┐  ┌─────────▼──────┐         │
│  │ Finance Svc │  │ Commerce Svc   │  │ All Other Svc  │         │
│  │ Consumer    │  │ Consumer       │  │ Consumers      │         │
│  └─────────────┘  └────────────────┘  └────────────────┘         │
│                                                                     │
│  ┌──────────────────────────────────────────────────────┐          │
│  │              Dead Letter Queue Manager                 │          │
│  │  Poison Detection │ Replay │ Alert │ Archive          │          │
│  └──────────────────────────────────────────────────────┘          │
│                                                                     │
│  ┌──────────────────────────────────────────────────────┐          │
│  │              Cross-Region Replicator                   │          │
│  │  Federation │ Regional Isolation │ Failover Replication│         │
│  └──────────────────────────────────────────────────────┘          │
└─────────────────────────────────────────────────────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Event Gateway (HTTP-to-Event Bridge)** | Integration Service (Rust) | RESTful endpoint for external systems to publish events to and subscribe from the mesh; includes API key authentication, request validation, payload transformation, and circuit breaker protection |
| **Topic Hierarchy Management** | Integration Service + RabbitMQ | Structured namespace `{domain}.{entity}.{action}` (e.g., `finance.invoice.created`) with wildcard subscriptions (`finance.invoice.*`, `finance.#`), namespace isolation per tenant |
| **Schema Registry** | Integration Service (Rust + PostgreSQL) | Event schema storage with versioning, compatibility checking (backward, forward, full), evolution rules, deprecation management, and consumer migration tracking |
| **Event Filtering** | Integration Service (Rust) | Content-based filtering with filter expressions (JSONPath, comparison operators); consumers subscribe to specific payloads without receiving all events on a topic |
| **Cross-System Integration Patterns** | Integration Service | Support for event sourcing, CQRS, saga orchestration, publish-subscribe, request-reply, and dead letter patterns across internal and external systems |
| **Subscription Management** | Integration Service | CRUD for event subscriptions with consumer groups, offset tracking, retry policies, backpressure configuration, and consumer health monitoring |
| **Dead Letter Handling** | Integration Service | Centralized DLQ monitoring, automatic poison message detection, replay capabilities, alert generation, and archival for failed events |
| **Event Transformation** | Integration Service | Configurable event transformation and enrichment at the mesh layer for consumer-specific formats (field mapping, aggregation, splitting, enrichment from reference data) |
| **Event Encryption** | Integration Service | End-to-end event payload encryption for sensitive data with per-tenant key management and key rotation support |
| **Cross-Region Routing** | Integration Service | Event federation across deployment regions with regional isolation, guaranteed delivery, and failover replication |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| All MSERP Services | All Services | Event-driven | Core event publishing and consuming via mesh |
| API Marketplace | Integration Service | Internal API | Event-driven APIs published to API catalog |
| Schema Registry | Integration Service | Internal API | Schema registration and validation |
| External Systems | Via Event Gateway | HTTP/REST | External publishers and subscribers |
| Monitoring | Prometheus + Grafana | Metrics | Throughput, latency, error rate, consumer lag |
| Notification Service | Platform Service | Event-driven | DLQ alerts, consumer failure notifications |
| Audit Trail | Platform Service | Event-driven | Event mesh operations logged |
| Multi-Region | Infrastructure | Federation | Cross-region event replication |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `mesh.subscription.created` | `{subscription_id, consumer, topic_pattern, filter}` | New subscription registered | Schema Registry, Monitoring |
| `mesh.subscription.deleted` | `{subscription_id, consumer, topic_pattern}` | Subscription removed | Monitoring |
| `mesh.event.published` | `{topic, event_id, source, size, published_at}` | Event published to mesh | Monitoring, Audit Trail |
| `mesh.event.delivered` | `{topic, event_id, consumer, delivered_at, latency_ms}` | Event delivered to consumer | Monitoring |
| `mesh.event.dead-lettered` | `{topic, event_id, consumer, reason, error}` | Event moved to DLQ | Notification Service, Monitoring |
| `mesh.event.replayed` | `{topic, event_id, replayed_by, original_error}` | DLQ event replayed | Monitoring, Audit Trail |
| `mesh.schema.registered` | `{topic, schema_version, compatibility}` | New schema version registered | Schema Registry, Audit Trail |
| `mesh.consumer.lag-high` | `{consumer, topic, lag_count, threshold}` | Consumer lag exceeds threshold | Notification Service, Monitoring |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| All domain events | All Services | Route to subscribers based on topic and filter |
| `mesh.event.dead-lettered` | Integration Service | Alert generation, DLQ management |
| `mesh.subscription.created` | Integration Service | Set up queue binding and filtering |
| `mesh.subscription.deleted` | Integration Service | Remove queue binding |

## Subscription Management

| Property | Description | Default |
|----------|-------------|---------|
| `topic_pattern` | Topic to subscribe to (supports wildcards) | Required |
| `filter_expression` | Content-based filter (JSONPath) | None (all events) |
| `consumer_group` | Consumer group for shared subscriptions | Auto-generated |
| `retry_policy` | Retry count and backoff strategy | 3 retries, exponential backoff |
| `batch_size` | Max events per delivery batch | 10 |
| `batch_timeout` | Max wait time for batch assembly | 5 seconds |
| `dead_letter_threshold` | Max retries before DLQ | 5 |
| `backpressure_strategy` | `drop_oldest`, `drop_newest`, `buffer`, `block` | `buffer` (max 10,000) |

## Cross-References

- [API Marketplace](api-marketplace.md) — API catalog and developer portal
- [Architecture Overview](../architecture/overview.md) — Event-driven architecture patterns
- [Event Catalog](../events/catalog.md) — Complete domain event reference
- [Event-Driven Architecture](../events/overview.md) — Event schemas, outbox, saga patterns
- [Integration Service](../services/integration.md) — Integration service specification
- [NFR: Performance](../planning/nfr.md) — Event processing latency < 100ms
