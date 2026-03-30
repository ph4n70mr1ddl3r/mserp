# Advanced Demand Management

ML-powered demand sensing, statistical forecasting, and advanced demand management for demand-driven supply chain planning, managed by the Commerce Service. Report Service provides ML inference models via gRPC; Manufacturing Service provides capacity and production data via events; Finance Service provides budget and procurement data via events; CRM Service provides pipeline and opportunity data via events.

> **Ownership Note**: Advanced Demand Management is owned by Commerce Service. Report Service provides ML inference for demand sensing and forecast refinement via gRPC. Manufacturing Service provides capacity and production plan data via events (`manufacturing.plan.*`, `manufacturing.work-order.*`). Finance Service provides budget allocations and procurement data via events (`finance.budget.*`, `finance.procurement.*`). CRM Service provides pipeline and opportunity data via events (`crm.opportunity.*`). Commerce owns all demand management events under the `commerce.demand.*` namespace.

## Overview

Advanced Demand Management provides a comprehensive, ML-augmented demand planning platform that combines statistical forecasting methods, real-time demand sensing, collaborative forecasting workflows, and demand shaping intelligence. The module ingests historical sales data, external demand signals, and cross-functional inputs to generate accurate demand forecasts at configurable granularity levels (product, product family, location, channel, customer segment). Demand sensing uses ML models to detect short-term demand shifts from POS data, web traffic, social signals, and weather patterns, providing near-real-time demand adjustments. Statistical forecasting supports multiple methods including ARIMA, exponential smoothing (Holt-Winters), and moving averages, with automatic model selection based on data characteristics. Consensus forecasting enables cross-functional collaboration between sales, marketing, finance, and operations. Demand shaping quantifies the impact of promotions, pricing changes, and marketing activities on demand patterns. ABC/XYZ segmentation classifies items by revenue contribution and demand variability to drive differentiated planning strategies. Safety stock optimization calculates optimal buffer levels using service level targets and demand variability. Forecast accuracy tracking measures bias, MAPE, and WAPE across all methods and time horizons to continuously improve forecast quality.

## Data Flow Diagram

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  Historical      │────▶│  Forecast Engine │────▶│  Statistical        │
│  Sales Data      │     │  ┌─────────────┐ │     │  Forecasting        │
└──────────────────┘     │  │ Model       │ │     │  ┌───────────────┐  │
                         │  │ Selection   │ │     │  │ ARIMA /       │  │
┌──────────────────┐     │  └─────────────┘ │     │  │ Holt-Winters  │  │
│  External        │────▶│  ┌─────────────┐ │     │  │ / Moving Avg  │  │
│  Demand Signals  │     │  │ Signal      │ │     │  └───────┬───────┘  │
│  (POS, weather,  │     │  │ Aggregator  │ │     └──────────┼──────────┘
│   web, social)   │     │  └─────────────┘ │                │
└──────────────────┘     └──────────────────┘     ┌──────────▼──────────┐
                                                  │  ML Demand Sensing   │
┌──────────────────┐     ┌──────────────────┐     │  ┌───────────────┐  │
│  CRM Pipeline    │────▶│  Demand          │────▶│  │ Short-Term   │  │
│  (Opportunities) │     │  Collaboration   │     │  │ Adjustment   │  │
└──────────────────┘     │  ┌─────────────┐ │     │  └───────────────┘  │
                         │  │ Consensus   │ │     └──────────┬──────────┘
┌──────────────────┐     │  │ Workflow    │ │                │
│  Finance Budget  │────▶│  └─────────────┘ │     ┌──────────▼──────────┐
│  Allocations     │     └──────────────────┘     │  Demand Shaping      │
└──────────────────┘                                │  ┌───────────────┐  │
                                                    │  │ Promo Impact  │  │
┌──────────────────┐     ┌──────────────────┐      │  │ Price Elastic │  │
│  Promotion       │────▶│  Segmentation    │      │  └───────────────┘  │
│  Calendar        │     │  (ABC/XYZ)       │─────▶└──────────┬──────────┘
└──────────────────┘     └──────────────────┘                  │
                                                               │
                         ┌──────────────────┐       ┌─────────▼──────────┐
                         │  Forecast         │◀─────│  Safety Stock       │
                         │  Accuracy         │      │  Optimization       │
                         │  Tracker          │      └────────────────────┘
                         └──────────────────┘
                                │
                         ┌──────▼──────┐
                         │  Published   │
                         │  Demand Plan │
                         └─────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Forecast Engine** | Commerce Service (Rust) | Core engine that orchestrates forecast generation: model selection, parameter optimization, forecast aggregation, and multi-level reconciliation (top-down, bottom-up, middle-out) |
| **Statistical Forecasting** | Commerce Service (Rust) | Native implementations of ARIMA(p,d,q), Holt-Winters (additive/multiplicative), and weighted moving average; automatic parameter tuning via AIC/BIC minimization |
| **ML Demand Sensing** | Report Service (ONNX) + Commerce Service (gRPC client) | ML models for short-term demand sensing (1-4 weeks); ingests POS data, web traffic, weather forecasts, and social signals; models served via Report Service gRPC inference endpoint |
| **Demand Collaboration** | Commerce Service (Rust) + Workflow Service | Consensus forecasting workflow: contributor input collection → variance analysis → exception-based resolution → consensus lock; supports sales, marketing, finance, and operations contributors |
| **Demand Shaping** | Commerce Service (Rust) | Promotion lift modeling, price elasticity estimation, and cannibalization analysis; quantifies demand impact of marketing activities and pricing changes |
| **Demand Segmentation** | Commerce Service (Rust) | ABC analysis by revenue contribution, XYZ analysis by demand variability (coefficient of variation); combined ABC/XYZ matrix for differentiated planning strategies |
| **Safety Stock Optimization** | Commerce Service (Rust) | Calculates optimal safety stock using demand variability, supply variability, target service level, and review period; supports both normal and non-normal demand distributions |
| **Forecast Accuracy Tracking** | Commerce Service (Rust) | Continuous measurement of forecast quality: MAPE, WAPE, bias, tracking signal; accuracy by method, horizon, segment, and product; automated model performance comparison |
| **Storage** | PostgreSQL + Redis | Forecast results, demand signals, accuracy metrics, and safety stock recommendations in PostgreSQL; forecast cache and signal buffer in Redis for sub-second access |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Forecast Engine | Orchestrate statistical and ML forecast generation with model selection | Commerce Service |
| ARIMA Module | Auto-regressive integrated moving average forecasting with automatic parameter optimization | Commerce Service |
| Holt-Winters Module | Triple exponential smoothing with additive and multiplicative seasonality | Commerce Service |
| Moving Average Module | Weighted and simple moving average forecasting for stable demand patterns | Commerce Service |
| Model Selector | Automatic best-fit model selection based on AIC/BIC, cross-validation, and data characteristics | Commerce Service |
| ML Demand Sensor | Short-term demand adjustment using external signals and ML inference | Report Service (model) + Commerce Service (invocation) |
| Signal Aggregator | Ingest and normalize external demand signals (POS, web, weather, social) | Commerce Service |
| Consensus Forecaster | Multi-contributor forecast input, variance analysis, and consensus workflow | Commerce Service + Workflow Service |
| Demand Shaping Engine | Promotion impact analysis, price elasticity calculation, cannibalization modeling | Commerce Service |
| ABC/XYZ Segmenter | Item classification by revenue contribution and demand variability | Commerce Service |
| Safety Stock Optimizer | Optimal buffer calculation considering demand/supply variability and service level targets | Commerce Service |
| Accuracy Tracker | Forecast accuracy measurement, bias detection, and model performance comparison | Commerce Service |
| Forecast Reconciliation | Multi-level forecast reconciliation (top-down, bottom-up, middle-out) across product/location hierarchies | Commerce Service |

## Forecasting Methods

| Method | Best For | Horizon | Seasonality | Parameters | Auto-Tuning |
|--------|----------|---------|-------------|------------|-------------|
| **ARIMA(p,d,q)** | Trending, non-seasonal or seasonal data | Medium (1-12 months) | Via SARIMA extension | p, d, q (seasonal: P, D, Q, m) | AIC/BIC minimization |
| **Holt-Winters** | Data with trend and seasonality | Short-Medium (1-6 months) | Additive or multiplicative | α, β, γ smoothing parameters | Error minimization |
| **Weighted Moving Average** | Stable, low-variability demand | Short (1-3 months) | None | Weights, window size | Variance minimization |
| **ML Demand Sensing** | Short-term demand shifts, external signals | Very Short (1-4 weeks) | Implicit | Model-specific | Report Service training pipeline |
| **Composite (Auto-Select)** | Mixed patterns, uncertain data characteristics | Any | Automatic | Best-fit selection | AIC/BIC + cross-validation |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Sales Orders / History | Commerce Service | Internal API | Historical sales data for forecast training; order pipeline for near-term demand |
| Inventory Planning | Commerce Service | Internal API | Safety stock recommendations feed inventory planning; current positions inform demand |
| Manufacturing Capacity | Manufacturing Service | Event-driven | Production capacity and plan data for supply-constrained demand planning |
| Production Schedules | Manufacturing Service | Event-driven | Planned production for demand-supply matching and feasibility validation |
| Report Service (ML) | Report Service | gRPC | ML model inference for demand sensing, promotion lift, and forecast refinement |
| Budget Allocations | Finance Service | Event-driven | Budget constraints and financial targets for top-down demand planning |
| Procurement Plans | Finance Service | Event-driven | Supplier and procurement data for supply-informed demand adjustments |
| Sales Pipeline | CRM Service | Event-driven | Opportunity pipeline and win probability for near-term demand sensing |
| Customer Segments | CRM Service | Event-driven | Customer segment data for differentiated demand analysis |
| Promotion Management | Commerce Service | Internal API | Promotion calendar and pricing events for demand shaping analysis |
| Supply Chain Collaboration | Commerce Service | Event-driven | Consensus forecast sharing with suppliers via collaboration network |
| Connected Planning | Report Service | Event-driven | Demand plan feeds connected planning models; planning assumptions constrain forecasts |
| Workflow Engine | Workflow Service | Event-driven | Consensus forecasting approval and exception resolution workflows |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `commerce.demand.forecast.generated` | `{forecast_id, method, granularity, period_range, items[{item_id, location_id, periods[{date, qty, confidence}]}], generated_at}` | Forecast generation completes | Connected Planning, Manufacturing, Supply Collaboration |
| `commerce.demand.forecast.adjusted` | `{forecast_id, adjusted_by, reason, adjustment_type, old_values, new_values, approved_by}` | Manual or consensus adjustment applied | Report Service, Connected Planning |
| `commerce.demand.forecast.published` | `{forecast_id, version, status, published_by, published_at}` | Forecast version locked and published | Supply Collaboration, Manufacturing, Finance |
| `commerce.demand.sensing.signal-received` | `{signal_id, source_type, signal_data, confidence, received_at}` | External demand signal ingested | Demand Sensor, Dashboard |
| `commerce.demand.sensing.adjustment-made` | `{adjustment_id, forecast_id, signal_sources[], baseline_qty, adjusted_qty, confidence_delta}` | ML demand sensing adjusts short-term forecast | Forecast Engine, Dashboard |
| `commerce.demand.consensus.contribution-submitted` | `{contribution_id, forecast_id, contributor_id, contributor_role, values[], notes, submitted_at}` | Contributor submits forecast input | Consensus Forecaster, Notification |
| `commerce.demand.consensus.resolved` | `{consensus_id, forecast_id, contributors[], variance_pct, resolution_method, resolved_at}` | Consensus forecast finalized | Connected Planning, Notification |
| `commerce.demand.shaping.impact-calculated` | `{analysis_id, promotion_id, item_id, baseline_qty, lift_qty, lift_pct, cannibalization[], elasticity}` | Promotion or price impact modeled | Demand Shaping Dashboard, Marketing |
| `commerce.demand.segmentation.updated` | `{segmentation_id, period, items[{item_id, abc_class, xyz_class, combined_class, revenue_pct, cv}]}` | ABC/XYZ segmentation recalculated | Planning Strategy, Safety Stock |
| `commerce.demand.safety-stock.recommendation` | `{recommendation_id, item_id, location_id, current_qty, recommended_qty, service_level, demand_variability, supply_lead_time}` | Safety stock optimization run completes | Inventory Planning, Dashboard |
| `commerce.demand.accuracy.measured` | `{measurement_id, forecast_id, method, horizon, mape, wape, bias, tracking_signal, measured_at}` | Forecast accuracy calculated against actuals | Model Selector, Dashboard |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `commerce.order.completed` | Commerce Service | Ingest completed order for historical demand data |
| `commerce.order.submitted` | Commerce Service | Near-term demand signal from open orders |
| `commerce.inventory.atp-updated` | Commerce Service | Trigger safety stock recalculation on inventory changes |
| `manufacturing.plan.updated` | Manufacturing Service | Adjust demand forecast for supply-constrained scenarios |
| `manufacturing.capacity.updated` | Manufacturing Service | Validate demand plan feasibility against capacity |
| `report.ml.inference.complete` | Report Service | Apply ML demand sensing adjustments |
| `finance.budget.allocated` | Finance Service | Apply budget constraints to top-down demand planning |
| `finance.procurement.po-confirmed` | Finance Service | Validate supply availability for demand plan |
| `crm.opportunity.stage-changed` | CRM Service | Adjust near-term demand based on pipeline changes |
| `crm.opportunity.created` | CRM Service | New pipeline opportunity for demand sensing |
| `commerce.promotion.created` | Commerce Service | Trigger demand shaping analysis for promotion |
| `commerce.promotion.price-changed` | Commerce Service | Recalculate price elasticity impact |
| `workflow.step.approved` | Workflow Service | Approve consensus forecast contribution |
| `integration.external-signal.received` | Integration Service | Ingest external demand signal (POS, weather, social) |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `demand_forecasts` | `id, tenant_id, method, granularity, period_start, period_end, status, version, generated_by, generated_at, published_at` | Has many `demand_forecast_items` |
| `demand_forecast_items` | `id, tenant_id, forecast_id, item_id, location_id, channel_id, periods{date, baseline_qty, adjusted_qty, confidence, method}` | Belongs to `demand_forecasts` |
| `demand_signals` | `id, tenant_id, source_type, source_ref, signal_data, confidence, processed, received_at, processed_at` | Ingested by demand sensor |
| `demand_sensing_adjustments` | `id, tenant_id, forecast_id, signal_ids[], baseline_qty, adjusted_qty, confidence_delta, model_version, adjusted_at` | References `demand_forecasts`, `demand_signals` |
| `demand_consensus_contributions` | `id, tenant_id, forecast_id, contributor_id, contributor_role, values{period, qty, notes}, status, submitted_at` | Belongs to `demand_forecasts` |
| `demand_consensus_sessions` | `id, tenant_id, forecast_id, status, deadline, contributors[], variance_threshold, resolved_at` | Has many `demand_consensus_contributions` |
| `demand_shaping_analyses` | `id, tenant_id, promotion_id, item_id, baseline_qty, lift_qty, lift_pct, price_elasticity, cannibalization{item_id, impact_pct}, model_version` | References promotion and item |
| `demand_segmentation` | `id, tenant_id, period, item_id, location_id, abc_class, xyz_class, combined_class, revenue_contribution_pct, coefficient_of_variation` | Per item per period |
| `demand_safety_stock` | `id, tenant_id, item_id, location_id, recommended_qty, current_qty, service_level_target, demand_std_dev, supply_lead_time_avg, supply_lead_time_std_dev, review_period, last_recalculated` | Per item per location |
| `demand_accuracy_metrics` | `id, tenant_id, forecast_id, method, horizon_days, mape, wape, bias, tracking_signal, measured_period, measured_at` | References `demand_forecasts` |
| `demand_forecast_audit` | `id, tenant_id, forecast_id, event_type, old_values, new_values, reason, changed_by, changed_at` | Audit trail for forecast changes |

## Business Rules

### Forecast Generation

1. Forecasts are generated at configurable granularity: SKU-location, SKU-location-channel, or product-family-region.
2. The model selector evaluates all applicable methods and selects the best-fit based on lowest AIC/BIC with cross-validation on the most recent 20% of history.
3. Minimum historical data requirements: ARIMA (24 periods), Holt-Winters (24 periods with at least 2 seasonal cycles), Moving Average (6 periods).
4. If no method meets minimum accuracy thresholds, the system falls back to naive forecast (last period actual) with a low confidence flag.
5. Forecasts are regenerated on a configurable schedule (default: weekly) or on-demand when significant demand signals are detected.

### Demand Sensing

1. ML demand sensing adjusts only the 1-4 week horizon; longer horizons rely on statistical methods.
2. Signal confidence threshold: signals below the configured `min_signal_confidence` (default: 0.5) are logged but not applied.
3. Maximum adjustment cap: sensing adjustments are capped at ±30% of the baseline forecast (configurable) to prevent overcorrection.
4. All sensing adjustments are tracked with before/after values and contributing signal sources for auditability.
5. The ML model is retrained monthly by Report Service; model version is recorded on each adjustment.

### Consensus Forecasting

1. A consensus session is created for each forecast cycle with a configurable contributor list and deadline.
2. Variance threshold (default: 10%): if all contributor inputs fall within the threshold of the statistical baseline, auto-approve without escalation.
3. If variance exceeds threshold, an exception workflow is triggered via Workflow Service for resolution.
4. Consensus forecast is calculated as a weighted average of contributor inputs, with weights configurable per contributor role.
5. Once locked, the consensus forecast becomes the official demand plan and is published to downstream consumers.

### Demand Shaping

1. Promotion lift is modeled using historical promotion response data, adjusted for seasonality and trend.
2. Price elasticity is estimated using log-linear regression on historical price-quantity pairs; minimum 10 data points required.
3. Cannibalization analysis identifies cross-product demand transfer within the same product category.
4. Shaping adjustments are applied as overlay layers on the baseline forecast, not replacing it, preserving the ability to isolate promotion-driven demand.

### ABC/XYZ Segmentation

1. **ABC classification**: A (top 80% cumulative revenue), B (next 15%), C (bottom 5%). Thresholds configurable per tenant.
2. **XYZ classification**: X (CV ≤ 0.5, stable demand), Y (0.5 < CV ≤ 1.0, variable), Z (CV > 1.0, erratic). Coefficient of variation calculated over trailing 12 periods.
3. Combined matrix drives planning strategy: AX items (high value, stable) get sophisticated forecasting; CZ items (low value, erratic) may use simple methods or reorder points.
4. Segmentation is recalculated monthly; changes exceeding two classes trigger a review notification.

### Safety Stock Optimization

1. Safety stock formula: `SS = Z × √(L × σ²_d + D²_avg × σ²_L)` where Z = service level Z-score, L = lead time, σ_d = demand std dev, D_avg = average demand, σ_L = lead time std dev.
2. Service level targets are configurable per ABC/XYZ segment (default: AX=99%, AY=97%, AZ=95%, BX=95%, BY=90%, BZ=85%, CX=85%, CY=80%, CZ=75%).
3. Recalculation is triggered by: forecast regeneration, significant demand signal, inventory policy change, or on a configurable schedule (default: weekly).
4. Recommendations are advisory until explicitly approved by the inventory planner.

### Forecast Accuracy Tracking

1. Accuracy is measured at four horizons: 1-week, 1-month, 3-month, 6-month.
2. Metrics: MAPE (Mean Absolute Percentage Error), WAPE (Weighted Absolute Percentage Error), Bias (systematic over/under-forecasting), Tracking Signal (cumulative bias / MAD).
3. Tracking signal outside ±4 triggers an automatic alert for forecast review.
4. Model performance is compared across methods; if the current method underperforms by >5% WAPE versus the next-best for three consecutive periods, the model selector is triggered.
5. Accuracy data feeds back into the model selector weighting to improve future model selection.

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `commerce.demand.forecast.schedule` | `weekly` | Forecast regeneration frequency (daily, weekly, monthly) |
| `commerce.demand.forecast.granularity` | `sku_location` | Default forecast granularity (sku_location, sku_location_channel, family_region) |
| `commerce.demand.forecast.horizon_weeks` | 52 | Forecast horizon in weeks |
| `commerce.demand.sensing.enabled` | true | Enable ML demand sensing adjustments |
| `commerce.demand.sensing.horizon_weeks` | 4 | Demand sensing adjustment horizon |
| `commerce.demand.sensing.min_signal_confidence` | 0.5 | Minimum confidence to apply a demand signal |
| `commerce.demand.sensing.max_adjustment_pct` | 30 | Maximum sensing adjustment as percentage of baseline |
| `commerce.demand.consensus.variance_threshold_pct` | 10 | Variance threshold for auto-approval |
| `commerce.demand.consensus.default_deadline_days` | 5 | Default days for contributor input deadline |
| `commerce.demand.shaping.min_data_points` | 10 | Minimum historical data points for elasticity estimation |
| `commerce.demand.segmentation.abc_thresholds` | `A:80,B:15,C:5` | Cumulative revenue percentage thresholds for ABC classes |
| `commerce.demand.segmentation.xyz_thresholds` | `X:0.5,Y:1.0,Z:>1.0` | Coefficient of variation thresholds for XYZ classes |
| `commerce.demand.safety_stock.service_levels` | `AX:99,AY:97,AZ:95,BX:95,BY:90,BZ:85,CX:85,CY:80,CZ:75` | Default service level targets per segment |
| `commerce.demand.safety_stock.recalc_schedule` | `weekly` | Safety stock recalculation schedule |
| `commerce.demand.accuracy.tracking_signal_limit` | 4 | Tracking signal threshold for alert generation |
| `commerce.demand.accuracy.model_switch_threshold_pct` | 5 | WAPE percentage threshold to trigger model reselection |
| `commerce.demand.history.min_periods.arima` | 24 | Minimum historical periods required for ARIMA |
| `commerce.demand.history.min_periods.holt_winters` | 24 | Minimum historical periods required for Holt-Winters |
| `commerce.demand.history.min_periods.moving_avg` | 6 | Minimum historical periods required for moving average |

## Cross-References

- [Connected Planning](connected-planning.md) — Demand plan feeds unified financial, workforce, and supply chain planning
- [Supply Chain Collaboration](supply-chain-collaboration.md) — Consensus forecasts shared with suppliers via CPFR workflows
- [Global Order Promising](global-order-promising.md) — Demand forecasts inform inventory planning and promise calculations
- [Order Orchestration](order-orchestration.md) — Demand-driven fulfillment prioritization and backorder management
- [Report Service](../06-services/report.md) — ML model inference, training pipeline, and forecast analytics
- [Manufacturing Service](../06-services/manufacturing.md) — Production capacity and schedule data for demand-supply matching
- [Commerce Service](../06-services/commerce.md) — Commerce service specification and order management
- [SPEC.md §7.1](../SPEC.md) — CRM → Commerce handoff for pipeline data
- [SPEC.md §7.2](../SPEC.md) — Commerce → Finance handoff for demand-driven procurement
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
- [NFR: Performance](../10-planning/nfr.md) — Forecast generation < 5 min per tenant; sensing adjustment < 500ms
