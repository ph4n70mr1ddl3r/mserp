# Event-Driven Architecture

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
| `commerce.inbox` | `commerce.#`, `finance.purchase-order.#`, `manufacturing.work-order.#`, `crm.opportunity.won`, `config.changed` |
| `finance.inbox` | `finance.#`, `commerce.order.#`, `commerce.stock.#`, `hr.payroll.#`, `manufacturing.work-order.#`, `project.invoice.#`, `project.milestone.#`, `config.changed` |
| `hr.inbox` | `hr.#`, `workflow.step.#`, `config.changed` |
| `manufacturing.inbox` | `manufacturing.#`, `commerce.stock.#`, `commerce.order.#`, `config.changed` |
| `report.inbox` | `commerce.#`, `finance.#`, `hr.#`, `manufacturing.#`, `crm.#`, `project.#`, `workflow.#`, `platform.audit.#`, `platform.esg.#`, `platform.carbon.#`, `config.changed` |
| `workflow.inbox` | `workflow.#`, `hr.leave.#`, `commerce.order.#`, `finance.purchase-order.#`, `finance.expense.#`, `manufacturing.eco.#`, `config.changed` |
| `platform.inbox` | `platform.#`, `auth.login.#`, `tenant.feature.#`, `config.changed` |
| `integration.inbox` | `integration.#`, `config.changed` |
| `crm.inbox` | `crm.#`, `commerce.customer.#`, `config.changed` |
| `project.inbox` | `project.#`, `hr.employee.#`, `commerce.order.#`, `config.changed` |

## 2. Domain Events

### Commerce Events (Sales + Inventory + PIM + Transportation)
| Event | Description |
|-------|-------------|
| `commerce.customer.created` | Customer created |
| `commerce.customer.updated` | Customer updated |
| `commerce.customer.deleted` | Customer soft-deleted |
| `commerce.quotation.created` | Quotation created |
| `commerce.quotation.accepted` | Quotation accepted |
| `commerce.quotation.rejected` | Quotation rejected |
| `commerce.order.created` | Sales order created |
| `commerce.order.submitted` | Sales order submitted for fulfillment |
| `commerce.order.fulfilled` | Order fulfilled |
| `commerce.order.cancelled` | Order cancelled |
| `commerce.order.returned` | Return/RMA created for order |
| `commerce.product.created` | Product created |
| `commerce.product.updated` | Product updated |
| `commerce.product.approved` | Product approved and published (PIM workflow) |
| `commerce.product.published` | Product published to channels |
| `commerce.stock.updated` | Stock level changed |
| `commerce.stock.reserved` | Stock reserved for an order (saga step) |
| `commerce.stock.reservation.failed` | Stock reservation failed (saga compensation trigger) |
| `commerce.stock.reservation.released` | Reserved stock released (saga compensation) |
| `commerce.transfer.initiated` | Stock transfer started |
| `commerce.transfer.completed` | Stock transfer completed |
| `commerce.pricing.calculated` | Price calculated by pricing engine |
| `commerce.delivery.created` | Delivery scheduled |
| `commerce.delivery.completed` | Delivery confirmed |
| `commerce.shipment.dispatched` | Shipment dispatched to carrier |
| `commerce.shipment.in-transit` | Shipment in transit (carrier tracking update) |
| `commerce.shipment.delivered` | Shipment delivered with POD |
| `commerce.carrier.assigned` | Carrier assigned to shipment |

### Finance Events (Finance + Procurement + Treasury + Expenses + CLM + EPM)
| Event | Description |
|-------|-------------|
| `finance.journal.created` | Journal entry created |
| `finance.journal.posted` | Journal entry posted |
| `finance.journal.reversed` | Journal entry reversed |
| `finance.invoice.created` | Invoice created |
| `finance.invoice.creation.failed` | Invoice creation failed (saga compensation trigger) |
| `finance.invoice.paid` | Invoice paid |
| `finance.invoice.credit-memo` | Credit memo issued |
| `finance.payment.received` | Payment received |
| `finance.payment.made` | Payment made to supplier |
| `finance.supplier.created` | Supplier created |
| `finance.supplier.updated` | Supplier updated |
| `finance.purchase-order.created` | Purchase order created |
| `finance.purchase-order.approved` | Purchase order approved |
| `finance.purchase-order.received` | Goods received |
| `finance.account.created` | Account created in chart of accounts |
| `finance.account.updated` | Account modified in chart of accounts |
| `finance.period.closed` | Accounting period closed |
| `finance.budget.created` | Budget created |
| `finance.budget.exceeded` | Budget threshold exceeded (warning) |
| `finance.exchange-rate.updated` | Exchange rate updated |
| `finance.expense.submitted` | Expense report submitted |
| `finance.expense.approved` | Expense report approved |
| `finance.expense.rejected` | Expense report rejected |
| `finance.expense.paid` | Expense reimbursement paid |
| `finance.treasury.cash-position.updated` | Cash position updated across bank accounts |
| `finance.treasury.payment-batch.approved` | Payment batch approved for execution |
| `finance.treasury.payment-batch.executed` | Payment batch executed |
| `finance.contract.created` | Contract created |
| `finance.contract.approved` | Contract approved |
| `finance.contract.renewed` | Contract renewed |
| `finance.contract.expired` | Contract expired |
| `finance.plan.created` | Financial plan created (EPM) |
| `finance.plan.forecast.updated` | Forecast updated in financial plan |

### HR Events
| Event | Description |
|-------|-------------|
| `hr.employee.created` | Employee record created |
| `hr.employee.hired` | Employee onboarded |
| `hr.employee.updated` | Employee profile updated |
| `hr.employee.separated` | Employee offboarded |
| `hr.attendance.recorded` | Attendance check-in/out |
| `hr.leave.requested` | Leave request submitted |
| `hr.leave.approved` | Leave request approved |
| `hr.leave.rejected` | Leave request rejected |
| `hr.payroll.processed` | Payroll run completed |
| `hr.payroll.multi-country.completed` | Multi-country payroll cycle completed |
| `hr.requisition.created` | Job requisition created |
| `hr.applicant.submitted` | Job application submitted |
| `hr.review.started` | Performance review cycle started |
| `hr.review.completed` | Performance review completed |
| `hr.training.completed` | Training course completed |
| `hr.talent-review.initiated` | Talent review process initiated |
| `hr.succession.plan-created` | Succession plan created for a key position |
| `hr.workforce.simulation-completed` | Workforce modeling simulation completed |

### Manufacturing Events (Production + PLM + EAM)
| Event | Description |
|-------|-------------|
| `manufacturing.bom.created` | BOM created |
| `manufacturing.bom.updated` | BOM updated |
| `manufacturing.work-order.created` | Work order created |
| `manufacturing.work-order.started` | Work order started (production began) |
| `manufacturing.work-order.completed` | Work order completed |
| `manufacturing.quality.checked` | Quality check result recorded |
| `manufacturing.quality.failed` | Quality check failed (NCR created) |
| `manufacturing.maintenance.due` | Scheduled maintenance due |
| `manufacturing.eco.submitted` | Engineering Change Order submitted |
| `manufacturing.eco.approved` | Engineering Change Order approved |
| `manufacturing.eco.implemented` | Engineering Change Order implemented |
| `manufacturing.product-lifecycle.revision.created` | Product revision created in PLM |
| `manufacturing.product-lifecycle.phase-in` | Product phase-in initiated |
| `manufacturing.product-lifecycle.phase-out` | Product phase-out initiated |
| `manufacturing.asset.registered` | Enterprise asset registered |
| `manufacturing.asset.maintenance-completed` | Asset maintenance completed |
| `manufacturing.asset.decommissioned` | Asset decommissioned |
| `manufacturing.plan.firmed` | Production plan firmed |
| `manufacturing.plan.simulation.completed` | ASCP simulation completed |

### Platform Events (Notification + File + Audit + Digital Assistant + GRC)
| Event | Description |
|-------|-------------|
| `platform.notification.sent` | Notification dispatched |
| `platform.notification.failed` | Notification delivery failed |
| `platform.file.uploaded` | File uploaded and stored |
| `platform.file.deleted` | File soft-deleted |
| `platform.audit.logged` | Audit log entry created |
| `platform.digital-assistant.intent.resolved` | Digital assistant resolved user intent |
| `platform.digital-assistant.action.executed` | Digital assistant executed an action |
| `platform.digital-assistant.feedback.recorded` | User feedback on assistant response recorded |
| `platform.app-builder.app.published` | Low-code application published |
| `platform.app-builder.app.updated` | Low-code application updated |
| `platform.grc.compliance.violation.detected` | Compliance violation detected |
| `platform.grc.risk.assessment.completed` | Risk assessment completed |
| `platform.grc.sod.conflict.detected` | Segregation of Duties conflict detected |
| `platform.grc.incident.created` | GRC incident created |
| `platform.data-mask.applied` | Data masking rule applied to non-production environment |

> **Note:** `platform.audit.logged` is an internal event published for observability. Report Service subscribes for compliance dashboards. The authoritative audit log is stored directly in `audit_db` at write time (not event-sourced).

### Tenant Events
| Event | Description |
|-------|-------------|
| `tenant.created` | New tenant provisioned |
| `tenant.updated` | Tenant settings updated |
| `tenant.suspended` | Tenant suspended (billing or admin action) |
| `tenant.feature.changed` | Feature flag value changed for a tenant |

### Auth Events
| Event | Description |
|-------|-------------|
| `auth.login.succeeded` | User login succeeded |
| `auth.login.failed` | User login failed |
| `auth.token.refreshed` | Access token refreshed |
| `auth.mfa.enabled` | MFA enabled for a user |
| `auth.password.changed` | Password changed |
| `auth.session.revoked` | Session revoked |
| `auth.step-up.requested` | Step-up authentication requested (elevated privilege) |

> **Note:** Auth events are consumed by the Platform Service for security audit logging and alerting (e.g., brute-force detection on repeated `auth.login.failed`). Auth Service does not consume events from other services.

### Config Events
| Event | Description |
|-------|-------------|
| `config.changed` | System or tenant configuration value changed |

### Workflow Events
| Event | Description |
|-------|-------------|
| `workflow.instance.started` | Workflow instance started |
| `workflow.instance.completed` | Workflow instance completed |
| `workflow.instance.failed` | Workflow instance failed |
| `workflow.step.approved` | Workflow step approved |
| `workflow.step.rejected` | Workflow step rejected |
| `workflow.step.escalated` | Workflow step escalated (timeout or threshold) |
| `workflow.step.delegated` | Workflow step delegated to another user |
| `workflow.sod.conflict.flagged` | SoD conflict flagged during approval workflow |

### CRM / Marketing Events
| Event | Description |
|-------|-------------|
| `crm.contact.created` | Contact created |
| `crm.contact.updated` | Contact updated |
| `crm.lead.created` | Lead created |
| `crm.lead.converted` | Lead converted to opportunity |
| `crm.opportunity.created` | Opportunity created |
| `crm.opportunity.stage-changed` | Opportunity moved to new stage |
| `crm.opportunity.won` | Deal won |
| `crm.opportunity.lost` | Deal lost |
| `crm.campaign.launched` | Campaign launched |
| `crm.campaign.completed` | Campaign completed |
| `crm.activity.logged` | Activity (call, email, meeting) logged |
| `crm.case.created` | Service case created |
| `crm.case.resolved` | Service case resolved |

### Project Events
| Event | Description |
|-------|-------------|
| `project.created` | Project created |
| `project.updated` | Project updated |
| `project.task.created` | Task created |
| `project.task.completed` | Task completed |
| `project.milestone.reached` | Milestone reached |
| `project.timesheet.submitted` | Timesheet entry submitted |
| `project.timesheet.approved` | Timesheet approved |
| `project.expense.submitted` | Expense report submitted |
| `project.invoice.generated` | Project billing invoice generated |
| `project.risk.identified` | Risk identified |

### Integration Events
| Event | Description |
|-------|-------------|
| `integration.sync.started` | External system sync initiated |
| `integration.sync.completed` | External system sync completed |
| `integration.sync.failed` | External system sync failed |
| `integration.import.completed` | Data import completed |
| `integration.import.failed` | Data import failed |
| `integration.master-data.matched` | Master data matching result produced |
| `integration.master-data.merged` | Master data records merged into golden record |
| `integration.master-data.quality.violation` | Data quality rule violated |

> **Note:** Integration Service primarily produces outbound events (sync/import status). External system notifications are handled via direct outbound HTTP calls or webhooks. MDM events are consumed by Report Service for data quality dashboards.

### Cross-Domain Events
| Event | Publisher | Subscribers |
|-------|-----------|-------------|
| `commerce.order.created` | Commerce | Commerce (saga), Finance, Platform, Report, Workflow, CRM |
| `commerce.order.fulfilled` | Commerce | Finance, Platform, Report, CRM |
| `commerce.order.cancelled` | Commerce | Finance, Platform, Report, CRM |
| `commerce.quotation.accepted` | Commerce | Finance, Report, Workflow, CRM |
| `commerce.stock.updated` | Commerce | Manufacturing, Finance, Report |
| `commerce.customer.created` | Commerce | CRM, Report, Platform |
| `commerce.pricing.calculated` | Commerce | Report |
| `commerce.product.approved` | Commerce | Report, Integration (MDM) |
| `commerce.shipment.dispatched` | Commerce | Report, Platform |
| `finance.purchase-order.received` | Finance | Commerce, Platform, Report |
| `finance.invoice.created` | Finance | Commerce (saga), Platform, Report |
| `finance.invoice.paid` | Finance | Report, CRM |
| `finance.exchange-rate.updated` | Finance | Commerce, Report |
| `finance.period.closed` | Finance | Report |
| `finance.expense.submitted` | Finance | Workflow, Report |
| `finance.treasury.cash-position.updated` | Finance | Report |
| `finance.contract.approved` | Finance | Report, Platform |
| `finance.plan.forecast.updated` | Finance | Report |
| `manufacturing.work-order.completed` | Manufacturing | Commerce, Finance, Report |
| `manufacturing.eco.approved` | Manufacturing | Commerce, Report |
| `manufacturing.product-lifecycle.phase-out` | Manufacturing | Commerce, Report |
| `hr.employee.hired` | HR | Platform, Report, Identity (HTTP) |
| `hr.employee.separated` | HR | Platform, Report, Identity (HTTP) |
| `hr.leave.requested` | HR | Workflow, Report |
| `hr.payroll.processed` | HR | Finance, Report |
| `hr.review.completed` | HR | Report |
| `hr.talent-review.initiated` | HR | Report, Platform |
| `crm.opportunity.won` | CRM | Commerce, Report |
| `crm.opportunity.lost` | CRM | Report |
| `project.milestone.reached` | Project | Platform, Report, Finance |
| `project.invoice.generated` | Project | Finance, Report |
| `platform.audit.logged` | Platform | Report |
| `platform.grc.compliance.violation.detected` | Platform | Report, Workflow |
| `platform.grc.sod.conflict.detected` | Platform | Report, Workflow |
| `integration.master-data.merged` | Integration | Report |
| `tenant.feature.changed` | Tenant | Platform (flushes Redis), Report |
| `auth.login.failed` | Auth | Platform (security audit) |
| `config.changed` | Config | All services with inboxes |

> **Note:** Report Service subscribes to key business events for analytics aggregation. Workflow Service subscribes to events that trigger approval workflows. Core services (Auth, Identity, Tenant, Config) do not have inbox queues. Identity receives updates via direct service-to-service HTTP calls.

## 3. Event Schema

```json
{
  "event_id": "01234567-89ab-cdef-0123-456789abcdef",
  "event_type": "commerce.order.created",
  "event_version": "1.0",
  "timestamp": "2026-03-27T10:30:00Z",
  "tenant_id": "tenant-uuid",
  "correlation_id": "correlation-uuid",
  "causation_id": "causation-uuid",
  "aggregate": {
    "type": "SalesOrder",
    "id": "order-uuid"
  },
  "payload": {
    "order_number": "SO-2026-001234",
    "customer_id": "customer-uuid",
    "total_amount": 10000.00,
    "currency": "USD"
  },
  "metadata": {
    "user_id": "user-uuid",
    "source": "commerce-service",
    "version": 1
  }
}
```

> **Note on `tenant_id`:** Tenant-scoped events always include a `tenant_id`. System-level events (e.g., `config.changed` for global config, `auth.login.failed`, `tenant.created`) use `null` for `tenant_id`. Consumers MUST handle `tenant_id: null` gracefully.

## 4. Event Versioning Strategy

### 4.1 Versioning Rules

Event versions follow **semantic versioning** with `MAJOR.MINOR` format (no patch).

| Change Type | Version Bump | Backward Compatible? | Example |
|-------------|-------------|---------------------|---------|
| Add new optional field to payload | Minor: `1.0` -> `1.1` | Yes | Adding `discount_code` to order payload |
| Add new field to metadata | Minor: `1.0` -> `1.1` | Yes | Adding `source_ip` to metadata |
| Rename existing field | Major: `1.1` -> `2.0` | No | Renaming `total` to `total_amount` |
| Remove existing field | Major: `1.1` -> `2.0` | No | Removing deprecated `legacy_id` |
| Change field type | Major: `1.1` -> `2.0` | No | Changing `amount` from string to number |
| Restructure payload | Major: `1.1` -> `2.0` | No | Nesting fields under new structure |

### 4.2 Backward Compatibility

- **Minor versions** (e.g., `1.0` -> `1.1`) are additive only. Consumers MUST ignore unknown fields.
- **Major versions** (e.g., `1.1` -> `2.0`) are breaking. Consumers MUST be updated before the new version is activated.
- Consumers declare the minimum `event_version` they support via configuration.
- **Version rejection is enforced by consumers, not the broker.** Each consumer's event handler checks the `event_version` field in the event envelope and routes unsupported versions to its DLQ with reason `UNSUPPORTED_VERSION`. RabbitMQ does not perform version filtering.

### 4.3 Version Lifecycle

```
1. Event published at version 1.0
2. Producer adds optional field -> version 1.1 (backward compatible)
3. Producer needs to rename field:
   a. Producer publishes both v1.1 and v2.0 for 90 days (dual-publish period)
   b. All consumers are updated to handle v2.0
   c. After 90 days, producer stops publishing v1.1
   d. v1.x support is removed from consumers in the next major release
```

### 4.4 Consumer Version Configuration

Each service declares its supported event versions in its configuration:

```toml
[events.versions]
"commerce.order.created" = { min = "1.0", max = "2.0" }
"finance.invoice.paid" = { min = "1.0", max = "1.1" }
```

- Events outside the supported range are routed to the DLQ by the consumer's event handler with reason `UNSUPPORTED_VERSION`.
- Version ranges are validated at service startup. Misconfiguration prevents service start.

## 5. Transactional Outbox Pattern

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
│     ├─ Publishes to RabbitMQ                                     │
│     ├─ Marks entry as published                                  │
│     └─ On failure: retries with exponential backoff              │
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

## 6. Dead Letter Queues (DLQ)

Every service inbox queue MUST have an associated dead letter queue.

| Setting | Value |
|---------|-------|
| Max Retries | 5 |
| Backoff Strategy | Exponential (1s, 2s, 4s, 8s, 16s) |
| DLQ Naming | `{service}.dlq` |
| DLQ Retention | 30 days |
| Alert on DLQ entry | Yes (Warning severity) |
| DLQ Replay | Supported via admin API (`POST /api/v1/admin/dlq/{service}/replay`) |

## 7. Saga Pattern — Distributed Transactions

Cross-service operations that require atomicity use the **choreography-based saga** pattern. Each step is an event handler that performs its work and emits a success or failure event.

### Exception: Orchestrated Sagas

When a participating service does not have an inbox queue (e.g., Identity Service), the originating service acts as the **orchestrator** and makes direct HTTP calls instead of relying on events.

### Example: Sales Order Fulfillment Saga (Choreography)

> **Note:** Sales and Inventory are part of the same Commerce Service. Stock reservation is an internal operation.

```
┌──────────┐     ┌───────────┐     ┌───────────┐
│ Commerce │     │  Finance  │     │ Platform  │
│ Service  │     │  Service  │     │  Service  │
└────┬─────┘     └────┬──────┘     └────┬──────┘
     │                │                │
     │ 1. Create order + reserve stock │
     │    (internal)                   │
     │                │                │
     │ commerce.order.created          │
     │───────────────▶│                │
     │                │                │
     │                │ 2. Create invoice
     │                │                │
     │                │ finance.invoice.created
     │                │───────────────▶│
     │                │                │
     │                │                │ 3. Notify customer
     │                │                │
     │ 4. Mark order fulfilled         │
     │◀──────────────│                │

--- FAILURE PATHS ---

Stock insufficient:    Cancel order internally → notify customer
Invoice fails:         Release stock → cancel order → notify customer
```

### Saga Step Registry

| Saga | Pattern | Steps | Compensating Actions |
|------|---------|-------|---------------------|
| Sales Order Fulfillment | Choreography | Reserve stock (internal) → Create invoice (Finance) → Notify (Platform) → Fulfill order | Release stock → Cancel invoice → Notify cancellation |
| Purchase Order Receiving | Choreography | Receive goods → Update stock (Commerce) → Record AP (internal) | Reverse stock receipt → Cancel AP entry |
| Employee Onboarding | Orchestrated (HR → Identity via HTTP) | Create identity (HTTP) → Assign roles (HTTP) → Provision access (event) | Deactivate identity → Remove roles |
| Leave Approval | Choreography | Leave requested → Workflow approval → Update leave balance | Reverse balance update |
| Purchase Requisition | Choreography | Requisition created → Approval workflow → Auto-create PO on approval | Cancel requisition |
| Project Billing | Choreography | Milestone reached → Generate invoice (Finance) → Notify customer | Cancel invoice |
| Expense Report | Choreography | Expense submitted → Workflow approval → Create journal entry (Finance) → Notify reimbursement | Reverse journal entry → Notify rejection |
| Contract Approval | Choreography | Contract created → Legal review (Workflow) → Approve → Notify stakeholders | Cancel contract → notify requester |
| Engineering Change Order | Choreography | ECO submitted → Review (Workflow) → Update BOM (Manufacturing) → Notify affected orders | Revert BOM → notify stakeholders |

**Rules:**
- Each saga step MUST have a compensating (undo) action.
- Sagas MUST have a timeout (default: 30 minutes). Expired sagas trigger compensation.
- Saga state is tracked in the originating service's database.
- All choreography saga events include `saga_id` and `saga_step` in their metadata.
- Orchestrated sagas track state in the orchestrator's database and use `correlation_id` to link related HTTP calls.
- **Compensation failure:** If a compensating action fails, the saga enters a `compensation_failed` state, the event is routed to the DLQ, and a Critical alert is triggered. Manual intervention is required. Saga state is retained for 30 days after any terminal state for debugging.

---

*See [Data Architecture](data-architecture.md) for database and schema details.*
*See [API Design](api-design.md) for synchronous communication patterns.*
