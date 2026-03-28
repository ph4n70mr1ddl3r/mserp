# Event Mesh

Event backbone for cross-system event routing and integration.

| Aspect | Implementation |
|--------|---------------|
| Event Backbone | Centralized event routing infrastructure built on RabbitMQ with topic-based routing |
| Event Gateway | HTTP-to-event bridge for external systems to publish and subscribe to MSERP events |
| Topic Hierarchy | Structured topic namespace (`{domain}.{entity}.{action}`) with wildcard subscriptions |
| Event Filtering | Content-based filtering allowing consumers to subscribe to specific event payloads |
| Dead Letter Management | Centralized DLQ monitoring and replay across all services |
| Schema Registry | Event schema registry with compatibility checking and version management |

*See [API Marketplace](api-marketplace.md) for API catalog. See [Architecture Overview](../architecture/overview.md) for communication patterns.*
