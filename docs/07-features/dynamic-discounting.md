# Dynamic Discounting & Early Payment Optimization

Dynamic early payment discount management, supplier discount optimization, and working capital analytics, managed by the Finance Service (treasury and AP).

## Overview

Dynamic Discounting transforms the traditional static discount model ("2/10 Net 30") into a sliding-scale system where discount rates are dynamically calculated based on actual payment timing. Suppliers can offer or accept early payment discounts at any point during the payment term, and buyers can optimize working capital by choosing which invoices to pay early based on available cash, cost of capital, and APR comparison. The system supports supplier discount program enrollment, automated discount matching against PO terms, ML-optimized discount negotiation, cash flow impact simulation for early payment decisions, and comprehensive working capital analytics dashboards.

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Dynamic Discount Calculation** | Finance Service (Rust) | Sliding-scale discount engine: APR-equivalent calculation for any early payment date; configurable discount curves (linear, tiered, custom); real-time discount amount computation at payment selection time |
| **Supplier Discount Program Management** | Finance Service (Rust) | Supplier enrollment in discount programs; configurable discount terms per supplier, supplier group, or commodity category; program lifecycle management (enrollment, activation, suspension, renewal) |
| **Early Payment Offer Workflows** | Finance Service + Workflow Service | Buyer-initiated or supplier-initiated early payment offers; offer creation, negotiation, acceptance/rejection workflow; batch offer processing for high-volume suppliers |
| **Working Capital Optimization** | Finance Service (Rust) | Cash position-aware discount optimization: ranks available discount opportunities by effective APR; recommends payment timing based on cost of capital vs. discount return; DSO/DPO/DIO analytics |
| **Discount Capture Rate Tracking** | Report Service + Finance Service | Tracks discount capture rate: discounts captured vs. discounts available; missed discount analysis with root cause; supplier-level and category-level capture rate reporting |
| **APR Calculation** | Finance Service (Rust) | Annualized Percentage Rate calculation for early payment discounts: `APR = (discount_pct / (1 - discount_pct)) × (365 / (term_days - discount_days))`; comparison against cost of capital for decision support |
| **Supplier Portal Integration** | Commerce Service + Finance Service | Supplier-facing portal for discount program enrollment, early payment offer submission, discount acceptance/rejection, payment status tracking with discount details |
| **Automated Discount Matching** | Finance Service (Rust) | Auto-matching of invoice discount terms against PO terms and supplier discount programs; validation of discount eligibility; exception handling for term mismatches |
| **Cash Flow Impact Simulation** | Finance Service + Report Service | Simulate early payment scenarios: impact on cash position, working capital, and P&L over configurable horizon; compare strategies (pay all early, selective, none) |
| **Discount Negotiation Optimization (ML)** | Report Service (ONNX) + Finance Service | ML model trained on historical discount acceptance data predicts optimal discount rate for each supplier; considers supplier's historical acceptance patterns, commodity seasonality, and market conditions |

## Discount Calculation Model

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Dynamic Discount Calculation                          │
│                                                                          │
│  Static Discount (Traditional):                                          │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  "2/10 Net 30" → 2% discount if paid by day 10, full by day 30│    │
│  │  APR = (0.02/0.98) × (365/20) = 37.2%                        │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                          │
│  Dynamic Discount (Sliding Scale):                                       │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  Day 1-5:    2.5% discount  (APR: 67.8% - 18.6%)              │    │
│  │  Day 6-10:   2.0% discount  (APR: 18.6% - 14.9%)              │    │
│  │  Day 11-15:  1.5% discount  (APR: 14.9% - 9.3%)               │    │
│  │  Day 16-20:  1.0% discount  (APR: 9.3% - 6.1%)                │    │
│  │  Day 21-30:  0.5% discount  (APR: 6.1% - 2.1%)                │    │
│  │  Day 31+:    Net due (no discount)                             │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                          │
│  Decision Matrix (Buyer):                                                │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  IF discount APR > cost of capital → PAY EARLY (capture value)│    │
│  │  IF discount APR < cost of capital → PAY AT TERM (use capital)│    │
│  │  IF cash constrained → Prioritize highest APR discounts       │    │
│  └─────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────┘
```

## Discount Program Configuration

| Parameter | Description | Default | Range |
|-----------|-------------|---------|-------|
| **Discount Curve Type** | Linear, tiered, or custom sliding scale | Tiered | Linear / Tiered / Custom |
| **Maximum Discount Rate** | Cap on discount percentage | 3.0% | 0.5% - 10.0% |
| **Early Payment Window** | Days from invoice date eligible for early pay | 30 days | 7 - 120 days |
| **Minimum Invoice Amount** | Threshold below which discount processing is skipped | $100 | $0 - $10,000 |
| **APR Threshold** | Minimum APR to recommend early payment | 8% | 1% - 50% |
| **Cost of Capital** | Buyer's weighted average cost of capital for comparison | 6% | 1% - 25% |
| **Auto-Pay Threshold** | APR above which payment is auto-approved | 15% | 5% - 50% |
| **Supplier Approval Required** | Whether supplier must accept early payment offer | Yes | Yes / No / Per supplier |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Accounts Payable | Finance Service | Internal API | Invoice data, payment terms, vendor master for discount eligibility |
| Treasury / Cash Management | Finance Service | Internal API | Cash position, cost of capital, liquidity forecast for payment decisions |
| Supplier Portal | Commerce Service | Internal API | Supplier-facing discount enrollment, offer management, acceptance |
| Purchase Orders | Commerce Service | Internal API | PO terms for discount validation and matching |
| Report Service (ML) | Report Service | gRPC | Discount optimization ML model, cash flow simulation engine |
| Platform Workflow | Workflow Service | Event-driven | Early payment approval workflows, discount exception routing |
| Notification Service | Platform Service | Event-driven | Discount offer notifications, acceptance confirmations, payment reminders |
| General Ledger | Finance Service | Internal API | Discount expense/income GL posting, realized discount accounting |
| Payment Gateway | Finance Service | Internal API | Payment execution with discount-adjusted amounts |
| Audit Trail | Platform Service | Event-driven | Full audit of discount calculations, approvals, and payments |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `finance.discount.offer.created` | `{offer_id, supplier_id, invoice_ids[], discount_rate, early_pay_date, apr}` | Early payment offer generated | Notification Service, Supplier Portal |
| `finance.discount.offer.accepted` | `{offer_id, supplier_id, accepted_rate, accepted_date, accepted_by}` | Supplier accepts early payment offer | Payment Engine, GL, Dashboard |
| `finance.discount.offer.rejected` | `{offer_id, supplier_id, rejection_reason}` | Supplier rejects early payment offer | Dashboard, AP Team |
| `finance.discount.payment.executed` | `{payment_id, invoice_id, original_amount, discount_amount, payment_amount, apr_captured}` | Early payment with discount executed | GL, Treasury, Dashboard |
| `finance.discount.capture.updated` | `{period, total_available, total_captured, capture_rate, missed_value}` | Discount capture rate recalculated | Dashboard, AP Manager |
| `finance.discount.optimization.recommendation` | `{recommendation_id, invoices[], projected_savings, cash_impact, recommended_strategy}` | ML optimization model generates payment recommendations | AP Team, Treasury |
| `finance.discount.cashflow.simulated` | `{simulation_id, strategy, horizon_days, cash_impact, discount_savings, net_benefit}` | Cash flow simulation completes | Dashboard, Treasury |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `finance.invoice.approved` | Finance Service | Evaluate invoice for early payment discount eligibility |
| `finance.payment.due-approaching` | Finance Service | Trigger discount opportunity analysis for approaching invoices |
| `finance.cash-position.updated` | Finance Service | Recalculate optimal payment timing based on current cash |
| `finance.purchase-order.created` | Commerce Service | Check PO terms against supplier discount program |
| `report.ml.inference.complete` | Report Service | Apply discount optimization model predictions |
| `workflow.step.approved` | Platform Service | Process early payment approval decision |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `discount_program` | `id, supplier_id, commodity_category, curve_type, tiers[], max_rate, effective_from, effective_to, status` | Belongs to supplier; has many `discount_offer` |
| `discount_offer` | `id, program_id, supplier_id, invoice_ids[], discount_rate, early_pay_date, apr, status, created_at` | Belongs to `discount_program` |
| `discount_payment` | `id, offer_id, invoice_id, original_amount, discount_amount, payment_amount, apr_captured, paid_at` | Belongs to `discount_offer` |
| `discount_capture_metric` | `id, period, supplier_id, category, available_discounts, captured_discounts, capture_rate, missed_value` | Periodic aggregate |
| `discount_simulation` | `id, strategy, horizon_days, invoices_included, projected_savings, cash_impact, net_benefit, simulated_at` | Simulation result |
| `discount_curve` | `id, program_id, day_range_start, day_range_end, discount_rate, implied_apr` | Belongs to `discount_program` |

## Working Capital Analytics

| Metric | Calculation | Description | Target |
|--------|-------------|-------------|--------|
| Days Sales Outstanding (DSO) | `(AR Balance / Revenue) × Days` | Average days to collect payment | Decreasing trend |
| Days Payable Outstanding (DPO) | `(AP Balance / COGS) × Days` | Average days to pay suppliers | Optimized (not maximized) |
| Days Inventory Outstanding (DIO) | `(Inventory / COGS) × Days` | Average days inventory held | Decreasing trend |
| Cash Conversion Cycle | `DSO + DIO - DPO` | Days from cash out to cash in | Minimize |
| Working Capital Ratio | `Current Assets / Current Liabilities` | Liquidity health indicator | 1.5 - 2.0 |
| Discount Capture Rate | `Captured Discounts / Available Discounts × 100` | % of discounts successfully captured | > 85% |
| Weighted Average APR Captured | `Σ(APR × discount_amount) / Σ(discount_amount)` | Blended return on early payments | > Cost of capital |
| Annualized Discount Savings | `Σ(discount_amount) annualized` | Total discount value captured per year | Maximize |

## Discount Dashboard Components

| Component | Data Source | Refresh Rate | Description |
|-----------|-----------|--------------|-------------|
| Discount Opportunity Queue | Finance Service | Real-time | Invoices eligible for early payment sorted by APR |
| Cash Position Widget | Finance Service | Hourly | Current cash vs. early payment commitments |
| Discount Capture Trend | Report Service | Daily | Capture rate trend over time with missed discount analysis |
| APR Comparison Chart | Finance Service | On demand | Discount APR vs. cost of capital across supplier programs |
| Supplier Discount Summary | Finance Service | On demand | Per-supplier discount program status, rates, and capture |
| Working Capital Dashboard | Finance Service | Daily | DSO, DPO, DIO, CCC trends with target lines |
| Savings Realization Tracker | Finance Service | Monthly | Projected vs. realized discount savings by period |
| Payment Timing Optimizer | Report Service (ML) | On demand | AI-recommended payment timing for open invoices |

## Supplier Portal Discount Experience

```
┌─────────────────────────────────────────────────────────────────────┐
│                   Supplier Discount Portal View                      │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  Discount Program Status: ACTIVE ✓                            │  │
│  │  Enrolled: Jan 2024  │  Curve: Dynamic (Tiered)              │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  Early Payment Offers                                          │  │
│  │  ┌──────────┬──────────┬──────────┬──────────┬──────────┐    │  │
│  │  │ Invoice  │ Original │ Discount │ Net Pay  │ Pay By   │    │  │
│  │  │ INV-1234 │ $50,000  │ $1,000   │ $49,000  │ Apr 5    │    │  │
│  │  │ INV-1235 │ $32,000  │ $640     │ $31,360  │ Apr 8    │    │  │
│  │  │ INV-1237 │ $18,500  │ $370     │ $18,130  │ Apr 12   │    │  │
│  │  └──────────┴──────────┴──────────┴──────────┴──────────┘    │  │
│  │  [Accept All]  [Accept Selected]  [Counter-Offer]            │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  Discount History: YTD                                        │  │
│  │  Offers Received: 48  │  Accepted: 42  │  Savings: $24,300   │  │
│  └───────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

## Cross-References

- [Strategic Sourcing](../06-services/finance.md) — Supplier management and procurement optimization (Strategic Sourcing module)
- [Commodity Management](../06-services/finance.md) — Commodity pricing and market data (Commodity Management module)
- [Finance Service](../06-services/finance.md) — Treasury, AP, and cash management operations
- [Architecture Overview](../01-architecture/overview.md) — System context
