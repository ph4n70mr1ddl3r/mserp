# Credit Management

Comprehensive customer credit lifecycle management integrated with Commerce Service (order processing) and Finance Service (accounts receivable, dunning).

| Aspect | Implementation |
|--------|---------------|
| Credit Scoring | Automated credit scoring based on payment history, financial statements, trade references, and external credit bureau integration (Dun & Bradstreet, Experian, Equifax) |
| Credit Limits | Per-customer credit limits with multi-currency support, multi-entity rules for shared or independent limits across business units |
| Credit Holds | Automatic order and shipment hold when credit limit exceeded; configurable threshold alerts at 75%, 90%, and 100% of limit |
| Credit Release | Manual release by credit analyst and automated release workflows with multi-level approval chains based on exposure amount |
| Credit Exposure | Real-time exposure calculation: open sales orders + posted invoices + unbilled deliveries + pending returns, refreshed on each order event |
| Collection Scoring | Predictive collection scoring using ML models trained on payment patterns, aging buckets, and customer segment to prioritize collection activities |
| Credit Reports | Customer credit history, aging analysis, payment pattern trends, average days-to-pay, and credit utilization over time |
| Credit Review Scheduling | Periodic automated credit reviews triggered by review date, exposure threshold, or score change with analyst assignment |
| Guarantee Management | Parent company guarantees, letters of credit, and insurance policies as credit limit enhancements with expiry tracking |
| Risk Segmentation | Customer risk classification (low/medium/high/critical) with configurable order processing rules per segment |

### Credit Hold Workflow

```
Order Created → Exposure Check → Limit Exceeded?
  ├─ No → Order Proceeds
  └─ Yes → Credit Hold Applied
        → Analyst Review (manual or auto-scored)
        ├─ Release with Approval → Order Proceeds
        ├─ Partial Release → Adjusted Order Proceeds
        └─ Reject → Order Cancelled / Sent Back to Sales
```

### Credit Exposure Calculation

| Component | Source | Timing |
|-----------|--------|--------|
| Open Sales Orders | Commerce Service | Real-time on order entry |
| Posted Invoices (unpaid) | Finance Service (AR) | Real-time |
| Unbilled Deliveries | Commerce Service | Real-time on shipment |
| Pending Returns | Commerce Service | Real-time on RMA creation |
| Outstanding Payments | Finance Service | Real-time |

### Integration Points

| Integration | Description |
|-------------|-------------|
| Commerce — Order Processing | Real-time credit check on order create and amendment; automatic hold insertion on limit breach |
| Finance — Accounts Receivable | Invoice and payment events update credit exposure; dunning status feeds credit scoring |
| Finance — Collections | Collection prioritization based on predictive collection scores and aging analysis |
| External — Credit Bureaus | Automated credit report pulls and score ingestion via Integration Service connectors |

*See [Commerce Service](../services/commerce.md) for credit management module details. See [Finance Service](../services/finance.md) for accounts receivable. See [Integration Service](../services/integration.md) for external bureau connectors. See [Architecture Overview](../architecture/overview.md) for system context.*
