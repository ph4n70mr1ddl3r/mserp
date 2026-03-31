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
| Order Orchestration | Choreography | Evaluate orchestration rules → Reserve inventory across sources → Create fulfillment assignments → Process split/merge logic → Release to warehouse | Release inventory reservations → Cancel assignments → Re-queue order |
| MES Production | Choreography | Dispatch work instructions → Log labor transactions → Record production data → Complete operations → Post production to inventory | Reverse labor logging → Scrap production data → Return to work order queue |
| Global Trade Screening | Choreography | Order/trade submitted → Screen parties (Integration) → Check licenses → If blocked: hold order + notify compliance team → If cleared: release order → Update screening status | Release hold → Re-trigger screening → Notify trade compliance team |
| Warehouse Wave Execution | Choreography | Wave released → Pick tasks assigned → Pick completed → Pack completed → Putaway completed → Update inventory → Generate shipment | Cancel remaining picks → Reverse pack → Release putaway reservations → Re-queue wave |

**Rules:**

Saga pattern rules are defined in SPEC.md §5.9. This document defines the step-by-step choreography for each saga.

---

*See [Event-Driven Architecture Overview](./overview.md) for exchange structure, schema, and versioning.*
*See [Event Catalog](./catalog.md) for the full list of domain events.*
*See [Data Architecture](../03-data/overview.md) for database and schema details.*
*See [API Design](../05-api/standards.md) for synchronous communication patterns.*
