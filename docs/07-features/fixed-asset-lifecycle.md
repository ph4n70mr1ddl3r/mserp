# Fixed Asset Lifecycle Management

Full lifecycle management of fixed assets from acquisition through depreciation to retirement/disposal, supporting multiple depreciation methods, revaluation, impairment, and capitalization, managed by the Finance Service.

> **Ownership Note**: Fixed Asset Lifecycle is owned by Finance Service. Procurement (Finance) provides acquisition cost data via events (`finance.procurement.*`). GL posting is handled internally within Finance Service. Lease accounting references ASC 842 for leased asset handling and IAS 16 / IFRS for depreciation standards. Manufacturing Service provides asset utilization data for units-of-production depreciation via events (`manufacturing.iot.telemetry.*`).

## Overview

Fixed Asset Lifecycle Management provides comprehensive tracking and accounting for fixed assets from initial acquisition or construction through depreciation, revaluation, potential impairment, and eventual retirement or disposal. The module supports multiple depreciation methods — straight-line, declining balance, units-of-production, and sum-of-years-digits — with concurrent depreciation books for parallel reporting under different accounting standards (e.g., local GAAP and IFRS). It handles asset capitalization with configurable thresholds, automated depreciation scheduling, revaluation adjustments per IAS 16, impairment testing per IAS 36, asset transfers between locations and departments, and retirement/disposal with gain/loss recognition. For leased assets, the module implements ASC 842 / IFRS 16 right-of-use asset recognition and lease liability amortization.

## Data Flow Diagram

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  Procurement     │────▶│  Asset           │────▶│  Depreciation       │
│  (PO Receipt)    │     │  Capitalization  │     │  Engine             │
└──────────────────┘     │  ┌─────────────┐ │     │  ┌───────────────┐  │
                         │  │ Threshold   │ │     │  │ SL / DB /     │  │
┌──────────────────┐     │  │ Check       │ │     │  │ UoP / SYD     │  │
│  Lease Module    │────▶│  └─────────────┘ │     │  └───────┬───────┘  │
│  (ASC 842)       │     └──────────────────┘     └──────────┼──────────┘
└──────────────────┘                                           │
                         ┌──────────────────┐       ┌─────────▼──────────┐
┌──────────────────┐     │  Revaluation     │────▶  │  GL Posting         │
│  Asset Transfer  │────▶│  & Impairment    │       │  (Journal Entries)  │
│  Request         │     │  (IAS 16/36)     │       │  ┌───────────────┐  │
└──────────────────┘     └──────────────────┘       │  │ Depreciation  │  │
                                                     │  │ Disposal      │  │
┌──────────────────┐     ┌──────────────────┐       │  │ Revaluation   │  │
│  Utilization     │────▶│  Units-of-       │────▶  │  │ Transfer      │  │
│  (Manufacturing) │     │  Production      │       │  └───────────────┘  │
└──────────────────┘     │  Depreciation    │       └────────────────────┘
                         └──────────────────┘                │
                                                    ┌────────▼──────────┐
                                                    │  Asset Retirement  │
                                                    │  (Gain/Loss)       │
                                                    └───────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Asset Capitalization** | Finance Service (Rust) | Automated capitalization from procurement receipts or manual entry; threshold-based expense vs. capitalize decision; CIP (Construction in Progress) tracking |
| **Depreciation Engine** | Finance Service | Multi-method depreciation calculator supporting straight-line, declining balance, units-of-production, and sum-of-years-digits; concurrent multi-book depreciation |
| **Revaluation Processing** | Finance Service | Fair value revaluation per IAS 16 with surplus/deficit recognition; revaluation reserve tracking in equity |
| **Impairment Testing** | Finance Service | IAS 36 impairment testing: recoverable amount vs. carrying value; impairment loss recognition and potential reversal |
| **Asset Transfer** | Finance Service | Inter-department and inter-location transfers with GL reclassification; transfer approval workflow via Workflow Service |
| **Retirement & Disposal** | Finance Service | Asset retirement with proceeds, removal costs, and gain/loss calculation; partial and full disposal |
| **Lease Accounting** | Finance Service | ASC 842 / IFRS 16 right-of-use asset and lease liability calculation; operating and finance lease classification |
| **Storage** | PostgreSQL | Asset registers, depreciation schedules, revaluation history, and lease data in Finance database |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Asset Register | Asset master data, classification, location, and custodian management | Finance Service |
| Capitalization Engine | Threshold evaluation, CIP processing, and asset creation | Finance Service |
| Depreciation Scheduler | Periodic depreciation run orchestration and schedule management | Finance Service |
| Depreciation Calculator | Multi-method depreciation computation per asset per book | Finance Service |
| Revaluation Processor | Fair value adjustments and revaluation reserve management | Finance Service |
| Impairment Tester | Recoverable amount calculation and impairment loss posting | Finance Service |
| Transfer Processor | Asset movement between departments, locations, and cost centers | Finance Service |
| Retirement Processor | Disposal, sale, and write-off processing with gain/loss | Finance Service |
| Lease Accounting | ROU asset recognition, lease liability amortization, and lease classification | Finance Service |
| GL Integration | Automated journal entry creation for all asset transactions | Finance Service |

## Depreciation Methods

| Method | Formula | Configuration | Accounting Standard |
|--------|---------|--------------|-------------------|
| **Straight-Line (SL)** | `(Cost - Salvage) / Useful Life` | `cost, salvage_value, useful_life_periods` | IAS 16, GAAP |
| **Declining Balance (DB)** | `Book Value × (Rate)` where Rate = `SL Rate × Accelerator` | `cost, salvage_value, useful_life_periods, accelerator_factor` | GAAP (MACRS), IAS 16 (limited) |
| **Units-of-Production (UoP)** | `(Cost - Salvage) × (Units This Period / Total Estimated Units)` | `cost, salvage_value, total_estimated_units, period_units` | IAS 16 (usage-based) |
| **Sum-of-Years-Digits (SYD)** | `(Cost - Salvage) × (Remaining Life / SYD)` where SYD = `n(n+1)/2` | `cost, salvage_value, useful_life_periods` | GAAP (accelerated) |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Procurement / AP | Finance Service | Event-driven | PO receipt triggers asset capitalization; invoice cost updates asset value |
| General Ledger | Finance Service | Internal API | Automated journal entries for depreciation, revaluation, disposal, and transfer |
| Lease Management | Finance Service | Internal API | Lease data feeds ROU asset creation and lease liability amortization |
| Cost Accounting | Finance Service | Internal API | Asset depreciation allocated to cost centers and projects |
| Manufacturing (EAM) | Manufacturing Service | Event-driven | Asset utilization data for units-of-production depreciation; maintenance events |
| Workflow Engine | Workflow Service | Event-driven | Approval workflows for transfers, revaluations, and retirements above threshold |
| Report Service | Report Service | Internal API | Fixed asset registers, depreciation forecasts, and impairment reports |
| Tax Engine | Finance Service | Internal API | Tax depreciation schedules and asset tax basis tracking |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `finance.asset.created` | `{asset_id, asset_number, category_id, location_id, department_id, acquisition_cost, acquisition_date, depreciation_method, useful_life}` | Asset capitalized or manually created | GL, Cost Accounting, Report Service |
| `finance.asset.depreciation.posted` | `{asset_id, period, book_id, depreciation_amount, accumulated_depreciation, net_book_value}` | Depreciation run completes for period | GL, Cost Accounting, Report Service |
| `finance.asset.revaluated` | `{asset_id, revaluation_id, old_value, new_value, surplus_deficit, reserve_account}` | Asset fair value revaluation processed | GL, Report Service |
| `finance.asset.impairment.identified` | `{asset_id, impairment_id, carrying_value, recoverable_amount, impairment_loss, cash_generating_unit}` | Impairment test indicates loss | GL, Report Service, Notification |
| `finance.asset.transferred` | `{asset_id, transfer_id, from_department, to_department, from_location, to_location, from_cost_center, to_cost_center}` | Asset transfer approved and executed | GL, Cost Accounting, Notification |
| `finance.asset.retired` | `{asset_id, retirement_id, retirement_type, proceeds, removal_costs, net_book_value, gain_loss}` | Asset retired, sold, or written off | GL, Report Service, Notification |
| `finance.asset.lease.rou-recognized` | `{asset_id, lease_id, rou_asset_value, lease_liability, lease_term, discount_rate, classification}` | Right-of-use asset recognized for lease | GL, Lease Management, Report Service |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `finance.procurement.po-received` | Finance Service | Evaluate receipt for asset capitalization (check threshold, create CIP or capitalize) |
| `finance.procurement.invoice-validated` | Finance Service | Update asset acquisition cost with final invoice amount |
| `manufacturing.iot.telemetry.asset-utilization` | Manufacturing Service | Record production units for units-of-production depreciation |
| `manufacturing.eam.maintenance-completed` | Manufacturing Service | Evaluate if maintenance improves asset life or value (capitalization of upgrades) |
| `workflow.step.approved` | Workflow Service | Execute approved transfer, revaluation, or retirement |
| `finance.gl.period-opened` | Finance Service | Trigger depreciation run for newly opened period |
| `config.changed` | Config Service | Reload depreciation parameters and capitalization thresholds |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `fixed_assets` | `id, tenant_id, asset_number, description, category_id, location_id, department_id, cost_center_id, custodian_id, acquisition_cost, acquisition_date, salvage_value, useful_life_periods, depreciation_method, status, parent_asset_id, is_cip` | Has many `asset_depreciation_schedules`, `asset_revaluations`, `asset_retirements` |
| `asset_depreciation_schedules` | `id, tenant_id, asset_id, book_id, period_start, period_end, depreciation_amount, accumulated_depreciation, net_book_value, je_id, status` | Belongs to `fixed_assets` |
| `asset_revaluations` | `id, tenant_id, asset_id, old_gross_value, new_gross_value, revaluation_surplus, revaluation_deficit, appraisal_date, appraised_by, je_id, status` | Belongs to `fixed_assets` |
| `asset_retirements` | `id, tenant_id, asset_id, retirement_type, retirement_date, proceeds, removal_costs, net_book_value_at_retirement, gain_loss_amount, je_id, approved_by` | Belongs to `fixed_assets` |
| `asset_transfers` | `id, tenant_id, asset_id, from_department_id, to_department_id, from_location_id, to_location_id, from_cost_center_id, to_cost_center_id, transfer_date, reason, status, approved_by` | Belongs to `fixed_assets` |
| `asset_leases` | `id, tenant_id, asset_id, lease_number, lease_classification, lease_start, lease_end, lease_term_months, discount_rate, rou_asset_value, lease_liability, periodic_payment, payment_frequency` | Belongs to `fixed_assets` |

## Business Rules

### Depreciation Methods

1. **Straight-Line**: Depreciates evenly over useful life. Default method. Required for IFRS unless another method better reflects consumption pattern.
2. **Declining Balance**: Accelerated depreciation; `accelerator_factor` multiplies the straight-line rate. Switch to straight-line when straight-line depreciation exceeds declining balance.
3. **Units-of-Production**: Depreciation proportional to actual output. Requires `total_estimated_units` and actual `period_units` from Manufacturing telemetry events. Recalculates when total estimates change.
4. **Sum-of-Years-Digits**: Accelerated method using SYD fraction `(remaining_life / sum_of_years)`. Suitable for assets that lose value rapidly in early years.
5. **Multi-book depreciation**: Assets may depreciate concurrently under different methods in separate books (e.g., tax book vs. GAAP book vs. IFRS book). Each book runs independently.

### Capitalization Thresholds

1. Assets below the configured `capitalization_threshold` are expensed immediately rather than capitalized.
2. Thresholds are configurable per asset category and may vary by tenant.
3. Construction in Progress (CIP) assets do not depreciate until capitalized.
4. Capitalization date = date the asset is placed in service, not necessarily the acquisition date.
5. Subsequent expenditure is capitalized only if it increases future economic benefit (IAS 16). Otherwise, it is expensed as maintenance.

### Impairment Testing (IAS 36)

1. Impairment triggers: significant decline in market value, physical damage, adverse changes in economic/legal environment, internal evidence of obsolescence.
2. Recoverable amount = higher of fair value less costs of disposal and value in use.
3. If carrying value > recoverable amount: recognize impairment loss = carrying value − recoverable amount.
4. Impairment reversal permitted under IFRS (not under US GAAP for assets held and used). Reversal limited to original carrying value minus subsequent depreciation.
5. Annual impairment test required for goodwill and intangible assets with indefinite life.

### Revaluation Rules (IAS 16)

1. Revaluation increases: credit to revaluation surplus (other comprehensive income), not profit or loss.
2. Revaluation decreases: charge to profit or loss to the extent it reverses a previous surplus; any excess to revaluation surplus.
3. Revaluation must be applied to the entire class of assets (not individual cherry-picking).
4. Revaluation frequency: sufficient regularity to ensure carrying value does not differ materially from fair value (typically every 1–3 years).
5. On disposal, transfer any remaining revaluation surplus to retained earnings.

### Lease Accounting (ASC 842 / IFRS 16)

1. Lessee recognizes right-of-use (ROU) asset and lease liability for all leases > 12 months (short-term exemption available).
2. Lease classification: finance lease (IFRS 16 eliminates operating/finance distinction for lessees; ASC 842 retains it).
3. Discount rate: rate implicit in the lease; if not determinable, lessee's incremental borrowing rate.
4. ROU asset = initial lease liability + initial direct costs + prepayments − lease incentives.
5. Lease modifications: reassess liability and adjust ROU asset; treat as separate lease if scope increase and standalone price adjustment.

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `finance.asset.capitalization_threshold` | 5000.00 | Minimum cost to capitalize (below this = expense); configurable per category |
| `finance.asset.depreciation.auto_run` | true | Automatically run depreciation when GL period opens |
| `finance.asset.depreciation.default_method` | `straight_line` | Default depreciation method for new assets |
| `finance.asset.retirement.require_approval_above` | 10000.00 | Net book value threshold requiring workflow approval for retirement |
| `finance.asset.revaluation.min_frequency_months` | 12 | Minimum months between revaluations for an asset class |
| `finance.asset.lease.short_term_exempt_months` | 12 | Lease term threshold for short-term lease exemption (ASC 842) |

## Cross-References

- [Procurement](../06-services/finance.md) — Asset acquisition from purchase orders
- [General Ledger](../06-services/finance.md) — Journal entries for all asset transactions
- [Manufacturing EAM](../06-services/manufacturing.md) — Asset utilization telemetry and maintenance events
- [Workflow Service](../06-services/workflow.md) — Approval workflows for transfers and retirements
- [Report Service](../06-services/report.md) — Asset registers, depreciation forecasts, and impairment reports
- [SPEC.md §7.2](../SPEC.md) — Commerce → Finance handoff boundary
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
- [Finance Service](../06-services/finance.md) — Finance service specification
- [NFR: Performance](../10-planning/nfr.md) — Depreciation run < 30s per 10,000 assets
