# Saga Patterns

## 1. Saga Pattern — Distributed Transactions

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

*See [Event-Driven Architecture Overview](./overview.md) for exchange structure, schema, and versioning.*
*See [Event Catalog](./catalog.md) for the full list of domain events.*
*See [Data Architecture](../data/overview.md) for database and schema details.*
*See [API Design](../api/standards.md) for synchronous communication patterns.*
