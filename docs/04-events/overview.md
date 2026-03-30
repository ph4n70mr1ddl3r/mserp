# Event-Driven Architecture Overview

## 1. RabbitMQ Exchange Structure

All domain events are published to a single **topic exchange** (`mserp.events`) and routed to service-specific queues using binding keys.

### Routing Convention

| Pattern | Routes to | Example |
|---------|-----------|---------|
| `{domain}.*` | `{domain}.inbox` queue | `commerce.*` → `commerce.inbox` |
| `{domain}.{entity}.#` | `{domain}.inbox` queue | `commerce.stock.reserved` → `commerce.inbox` |
| `config.changed` | All service inbox queues | `config.changed` → all `.inbox` queues |

Each service's inbox queue binds to the topic exchange with patterns matching the events it consumes. Core services (Auth, Identity, Tenant, Config) do **not** have inbox queues — they publish events but do not consume from the event bus.

```
┌─────────────────────────────────────────────────────────────┐
│                    RabbitMQ Topology                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────┐     ┌─────────────────────────────────┐    │
│  │  mserp.     │────▶│  mserp.commerce.events          │    │
│  │  events     │     ├─────────────────────────────────┤    │
│  │  (Topic)    │     │  mserp.finance.events           │    │
│  └─────────────┘     ├─────────────────────────────────┤    │
│                      │  mserp.hr.events                │    │
│                      ├─────────────────────────────────┤    │
│                      │  mserp.manufacturing.events     │    │
│                      ├─────────────────────────────────┤    │
│                      │  mserp.workflow.events          │    │
│                      ├─────────────────────────────────┤    │
│                      │  mserp.platform.events          │    │
│                      ├─────────────────────────────────┤    │
│                      │  mserp.crm.events               │    │
│                      ├─────────────────────────────────┤    │
│                      │  mserp.project.events           │    │
│                      ├─────────────────────────────────┤    │
│                      │  mserp.report.events            │    │
│                      ├─────────────────────────────────┤    │
│                      │  mserp.integration.events       │    │
│                      └─────────────────────────────────┘    │
│                                      │                       │
│                                      ▼                       │
│                      ┌─────────────────────────────────┐    │
│                      │       Service Queues            │    │
│                      │  • commerce.inbox               │    │
│                      │  • finance.inbox                │    │
│                      │  • platform.inbox               │    │
│                      │  • report.inbox                 │    │
│                      │  • workflow.inbox               │    │
│                      │  • hr.inbox                     │    │
│                      │  • manufacturing.inbox          │    │
│                      │  • integration.inbox            │    │
│                      │  • crm.inbox                    │    │
│                      │  • project.inbox                │    │
│                      │                                  │    │
│                      │       Dead Letter Queues         │    │
│                      │  • {service}.dlq (per queue)    │    │
│                      └─────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

### Service Inbox Bindings

| Service Inbox | Binds To (Routing Keys) |
|--------------|------------------------|
| `commerce.inbox` | `commerce.#`, `commerce.orchestration.#`, `commerce.backorder.#`, `finance.purchase-order.#`, `finance.invoice.#`, `finance.sourcing.#`, `finance.cash-application.#`, `manufacturing.work-order.#`, `manufacturing.digital-twin.#`, `crm.opportunity.won`, `crm.cdp.#`, `config.changed` |
| `finance.inbox` | `finance.#`, `commerce.order.#`, `commerce.stock.#`, `commerce.subscription.#`, `commerce.b2b.#`, `commerce.warranty.#`, `hr.payroll.#`, `manufacturing.work-order.#`, `project.invoice.#`, `project.milestone.#`, `platform.idp.#`, `config.changed` |
| `hr.inbox` | `hr.#`, `hr.goal.#`, `workflow.step.#`, `config.changed` |
| `manufacturing.inbox` | `manufacturing.#`, `manufacturing.mes.#`, `manufacturing.schedule.#`, `commerce.stock.#`, `commerce.order.#`, `commerce.warranty.#`, `platform.iot.device.#`, `config.changed` |
| `report.inbox` | `report.#`, `report.ml-studio.#`, `commerce.#`, `finance.#`, `hr.#`, `manufacturing.#`, `crm.#`, `project.#`, `workflow.#`, `platform.audit.#`, `platform.rpa.#`, `tenant.feature.#`, `integration.trade-compliance.#`, `config.changed` |
| `workflow.inbox` | `workflow.#`, `hr.leave.#`, `commerce.order.#`, `commerce.credit.#`, `finance.purchase-order.#`, `finance.expense.#`, `finance.intelligent-close.#`, `finance.supplier-risk.#`, `manufacturing.eco.#`, `report.process.#`, `platform.rpa.#`, `platform.grc.#`, `integration.trade-compliance.#`, `config.changed` |
| `platform.inbox` | `platform.#`, `platform.composer.#`, `auth.login.#`, `hr.#`, `commerce.order.#`, `commerce.credit.#`, `commerce.shipment.#`, `commerce.logistics.#`, `finance.lease.#`, `finance.supplier-risk.#`, `finance.intelligent-close.#`, `integration.trade-compliance.#`, `crm.cdp.#`, `report.process.#`, `manufacturing.intelligence.#`, `tenant.feature.#`, `tenant.created`, `tenant.decommissioned`, `config.changed` |
| `integration.inbox` | `integration.#`, `commerce.customer.#`, `commerce.product.#`, `finance.supplier.#`, `hr.employee.#`, `identity.user.#`, `config.changed` |
| `crm.inbox` | `crm.#`, `commerce.customer.#`, `commerce.order.#`, `config.changed` |
| `project.inbox` | `project.#`, `hr.employee.#`, `commerce.order.#`, `config.changed` |

## 2. Event Schema

Event envelope schema is defined in **SPEC.md §5.4**.

> **Note on `tenant_id`:** Tenant-scoped events always include a `tenant_id`. System-level events (e.g., `config.changed` for global config, `auth.login.failed`, `tenant.created`) use `null` for `tenant_id`. Consumers MUST handle `tenant_id: null` gracefully.

## 3. Event Versioning Strategy

### 3.1 Versioning Rules

Event versions follow **semantic versioning** with `MAJOR.MINOR` format (no patch).

| Change Type | Version Bump | Backward Compatible? | Example |
|-------------|-------------|---------------------|---------|
| Add new optional field to payload | Minor: `1.0` -> `1.1` | Yes | Adding `discount_code` to order payload |
| Add new field to metadata | Minor: `1.0` -> `1.1` | Yes | Adding `source_ip` to metadata |
| Rename existing field | Major: `1.1` -> `2.0` | No | Renaming `total` to `total_amount` |
| Remove existing field | Major: `1.1` -> `2.0` | No | Removing deprecated `legacy_id` |
| Change field type | Major: `1.1` -> `2.0` | No | Changing `amount` from string to number |
| Restructure payload | Major: `1.1` -> `2.0` | No | Nesting fields under new structure |

### 3.2 Backward Compatibility

- **Minor versions** (e.g., `1.0` -> `1.1`) are additive only. Consumers MUST ignore unknown fields.
- **Major versions** (e.g., `1.1` -> `2.0`) are breaking. Consumers MUST be updated before the new version is activated.
- Consumers declare the minimum `event_version` they support via configuration.
- **Version rejection is enforced by consumers, not the broker.** Each consumer's event handler checks the `event_version` field in the event envelope and routes unsupported versions to its DLQ with reason `UNSUPPORTED_VERSION`. RabbitMQ does not perform version filtering.

### 3.3 Version Lifecycle

```
1. Event published at version 1.0
2. Producer adds optional field -> version 1.1 (backward compatible)
3. Producer needs to rename field:
   a. Producer publishes both v1.1 and v2.0 for 90 days (dual-publish period)
   b. All consumers are updated to handle v2.0
   c. After 90 days, producer stops publishing v1.1
   d. v1.x support is removed from consumers in the next major release
```

### 3.4 Consumer Version Configuration

Each service declares its supported event versions in its configuration:

```toml
[events.versions]
"commerce.order.created" = { min = "1.0", max = "2.0" }
"finance.invoice.paid" = { min = "1.0", max = "1.1" }
```

- Events outside the supported range are routed to the DLQ by the consumer's event handler with reason `UNSUPPORTED_VERSION`.
- Version ranges are validated at service startup. Misconfiguration prevents service start.

## 4. Transactional Outbox Pattern

To prevent the dual-write problem (writing to both the database and RabbitMQ), all services MUST use the transactional outbox pattern.

```
┌──────────────────────────────────────────────────────────────────┐
│                    Transactional Outbox                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  1. Begin DB transaction                                         │
│  2. Write business data to domain tables                          │
│  3. Write event to outbox table (same transaction)                │
│  4. Commit transaction                                           │
│                                                                   │
│  5. Outbox Poller (background task)                              │
│     ├─ Reads unprocessed outbox entries                          │
│     ├─ Publishes to RabbitMQ ONLY                                │
│     ├─ Marks entry as published                                  │
│     └─ On failure: retries with exponential backoff              │
│                                                                   │
│  Note: Data-lake ingestion is a separate process. Archived       │
│  events are read from MinIO (where they are stored after         │
│  publish), not from the outbox poller directly.                  │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

### Outbox Table Schema
```sql
CREATE TABLE outbox_events (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aggregate_type  VARCHAR(100) NOT NULL,
    aggregate_id    UUID NOT NULL,
    event_type      VARCHAR(100) NOT NULL,
    event_version   VARCHAR(10) NOT NULL DEFAULT '1.0',
    payload         JSONB NOT NULL,
    metadata        JSONB,
    tenant_id       UUID,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    published_at    TIMESTAMPTZ,
    status          VARCHAR(20) NOT NULL DEFAULT 'pending',
    retry_count     INTEGER NOT NULL DEFAULT 0,
    next_retry_at   TIMESTAMPTZ
);

CREATE INDEX idx_outbox_status ON outbox_events (status, next_retry_at)
    WHERE status = 'pending';
```

## 5. Dead Letter Queues (DLQ)

DLQ configuration is defined in **SPEC.md §5.7**.

## 6. Event Ordering Guarantees

Ordering guarantees are defined in **SPEC.md §5.8**.

---

*See [Event Catalog](./catalog.md) for the full list of domain events.*
*See [Saga Patterns](./sagas.md) for distributed transaction patterns.*
*See [Data Architecture](../03-data/overview.md) for database and schema details.*
*See [API Design](../05-api/standards.md) for synchronous communication patterns.*
