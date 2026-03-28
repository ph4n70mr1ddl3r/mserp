# Event Mesh

Event backbone for cross-system event routing and integration, managed by the Integration Service.

| Aspect | Implementation |
|--------|---------------|
| Event Backbone | Centralized event routing infrastructure built on RabbitMQ with topic-based routing and guaranteed delivery |
| Event Gateway | HTTP-to-event bridge for external systems to publish and subscribe to MSERP events via REST API |
| Topic Hierarchy | Structured topic namespace (`{domain}.{entity}.{action}`) with wildcard subscriptions and namespace isolation |
| Event Filtering | Content-based filtering allowing consumers to subscribe to specific event payloads with filter expressions |
| Dead Letter Management | Centralized DLQ monitoring, replay, and poison message handling across all services |
| Schema Registry | Event schema registry with compatibility checking (backward, forward, full), version management, and evolution rules |
| Event Versioning | Event schema evolution with backward compatibility, deprecation notices, and consumer migration support |
| Replay & Time-Travel | Event replay capability for recovery, testing, and analytics with configurable retention periods |
| Event Transformation | Configurable event transformation and enrichment at the mesh layer for consumer-specific formats |
| Cross-Region Routing | Event routing across deployment regions for multi-region active-passive with regional isolation |
| Event Encryption | End-to-end event payload encryption for sensitive data with key rotation support |
| Monitoring | Event flow monitoring with throughput, latency, error rate, and consumer lag dashboards |
| Throttling | Per-consumer rate limiting and backpressure handling to prevent consumer overwhelm |

*See [API Marketplace](api-marketplace.md) for API catalog. See [Architecture Overview](../architecture/overview.md) for communication patterns.*
