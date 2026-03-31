# Sales & Operations Planning

Integrated business planning connecting demand, supply, finance, and executive consensus, managed by the Commerce Service. Report Service provides ML-driven demand forecasts; Manufacturing Service provides capacity and production data via events; Finance Service provides budget and actuals data; Workflow Service manages multi-stage approval.

> **Ownership Note**: Sales & Operations Planning is owned by Commerce Service. Manufacturing Service provides capacity and production data via events (`manufacturing.capacity.*`, `manufacturing.plan.*`). Finance Service provides budget allocations and actuals (`finance.budget.*`, `finance.journal.*`). Report Service provides ML inference results (`report.ml.inference.complete`). Workflow Service manages consensus review stages. Commerce owns the S&OP planning engine, demand-supply matching, scenario comparison, and all S&OP events.

## Overview

Sales & Operations Planning (S&OP) provides an integrated business planning process that aligns demand forecasts, supply plans, financial budgets, and strategic objectives into a single consensus plan. The module supports multi-horizon planning across short-range (0–3 months), medium-range (3–12 months), and long-range (12–36 months) time buckets. It includes a demand-supply balancing engine that identifies gaps, a scenario comparison tool for base/upside/downside modeling, rough-cut capacity planning, and a structured consensus workflow with multiple review stages culminating in executive S&OP review. Rolling forecast integration ensures plans remain current as actuals materialize. KPI scorecards track revenue, margin, inventory turns, and service level against plan targets.

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Multi-Horizon Planning** | Commerce Service (Rust) | Time-bucketed planning across short (0–3 mo), medium (3–12 mo), and long range (12–36 mo) horizons with configurable period definitions |
| **Demand-Supply Balancing** | Commerce Service (Rust) | Matching engine that compares aggregate demand views against aggregate supply views, identifying surpluses and shortages by period and product family |
| **Consensus Planning Workflow** | Commerce Service + Workflow Service | Multi-stage review: Data Gathering → Demand Review → Supply Review → Finance Review → Executive S&OP Meeting → Plan Approval. Each stage is a Workflow Service approval step |
| **Scenario Comparison** | Commerce Service (Rust) | Side-by-side comparison of base, upside, and downside scenarios with variance analysis, impact scoring, and what-if adjustment |
| **Gap-to-Plan Analysis** | Commerce Service (Rust) | Automated gap identification between demand and supply by period, product family, and geography with severity classification and recommended actions |
| **Supply Constraint Identification** | Commerce Service (Rust) | Detection of material, capacity, and logistics constraints that prevent plan fulfillment; constraint propagation across planning horizons |
| **Rough-Cut Capacity Planning** | Commerce Service (Rust) | High-level capacity checks against critical resources (production lines, warehouse throughput, labor) to validate plan feasibility |
| **Executive S&OP Review** | Commerce Service + Report Service | Dashboard aggregating KPIs, gap summaries, scenario comparisons, and action items for executive decision-making |
| **Plan Versioning & Approval** | Commerce Service (Rust) | Full version history with approval chain, change tracking, and rollback capability. Only approved versions become the operating plan |
| **KPI Scorecards** | Commerce Service + Report Service | Revenue vs. plan, gross margin, inventory turns, service level (OTIF), forecast accuracy, plan attainment with trend visualization |
| **Rolling Forecast Integration** | Commerce Service (Rust) | Continuous forecast updates from demand planning, with automatic re-evaluation of gaps and plan adjustments within configurable thresholds |
| **Storage** | PostgreSQL | All S&OP plans, versions, scenarios, demand/supply views, gaps, meetings, and action items in Commerce database |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| S&OP Plan Manager | Plan lifecycle: create, version, approve, archive | Commerce Service |
| Demand View Aggregator | Consolidate demand signals from orders, forecasts, and promotions | Commerce Service |
| Supply View Aggregator | Consolidate supply signals from inventory, production plans, and POs | Commerce Service |
| Demand-Supply Matching Engine | Balance demand against supply, identify gaps and surpluses | Commerce Service |
| Scenario Engine | Create, compare, and score planning scenarios | Commerce Service |
| Rough-Cut Capacity Planner | Validate plan feasibility against critical resource constraints | Commerce Service |
| Consensus Workflow Orchestrator | Manage multi-stage review process via Workflow Service | Commerce Service + Workflow Service |
| Meeting Scheduler | Schedule and manage S&OP review meetings with agenda and minutes | Commerce Service |
| Action Item Tracker | Track follow-up actions from S&OP meetings with assignments and due dates | Commerce Service |
| KPI Scorecard Engine | Calculate and trend S&OP performance metrics | Commerce Service + Report Service |
| Rolling Forecast Processor | Integrate continuous forecast updates and trigger plan re-evaluation | Commerce Service |
| Plan Analytics | Plan accuracy, attainment trends, and process effectiveness | Commerce Service + Report Service |

## S&OP Process States

| State | Description | Next States |
|-------|-------------|-------------|
| Draft | Plan created, data gathering in progress | Demand Review |
| Demand Review | Demand side being reviewed and adjusted | Supply Review, Draft |
| Supply Review | Supply side being reviewed against demand | Finance Review, Demand Review |
| Finance Review | Financial alignment and budget reconciliation | Executive Review, Supply Review |
| Executive Review | Executive S&OP meeting for consensus | Approved, Finance Review |
| Approved | Consensus plan approved as operating plan | Active |
| Active | Operating plan in effect, being tracked against actuals | Closed, Superseded |
| Superseded | Replaced by a newer approved plan version | Active |
| Closed | Plan period ended, final performance locked | Terminal |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Demand Planning | Commerce Service | Internal API | Demand forecasts feed S&OP demand views |
| Sales Orders | Commerce Service | Internal API | Actual order data for demand view and plan attainment |
| Inventory Management | Commerce Service | Internal API | Current inventory positions for supply view |
| Manufacturing Capacity | Manufacturing Service | Event-driven | Capacity data for rough-cut capacity planning |
| Production Plans | Manufacturing Service | Event-driven | Production schedules for supply view |
| Budget Management | Finance Service | Event-driven | Budget allocations and actuals for finance review |
| General Ledger | Finance Service | Event-driven | Posted journals for actual-vs-plan comparison |
| ML Inference | Report Service | Event-driven | ML-driven demand forecast refinements |
| Approval Workflows | Workflow Service | Event-driven | Multi-stage consensus review and plan approval |
| Configuration | Config Service | Event-driven | S&OP parameters, thresholds, and period definitions |
| Notification Service | Platform Service | Event-driven | Meeting reminders, gap alerts, action item notifications |
| Report Service | Report Service | Internal API | KPI dashboards and plan performance analytics |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `commerce.sop.plan-created` | `{plan_id, horizon, period_range, product_families[], status}` | New S&OP plan created | Dashboard, Notification |
| `commerce.sop.plan-version.published` | `{plan_id, version_id, version_number, changes_summary, published_by}` | Plan version published for review | Dashboard, Notification, Workflow |
| `commerce.sop.gap-identified` | `{gap_id, plan_id, period, product_family, demand_qty, supply_qty, gap_qty, gap_type, severity}` | Demand-supply matching identifies a gap | Dashboard, Notification, Action Item Tracker |
| `commerce.sop.scenario.compared` | `{scenario_ids[], plan_id, comparison_metrics{}, variance_summary}` | Scenarios compared side-by-side | Dashboard, Executive Review |
| `commerce.sop.meeting-scheduled` | `{meeting_id, plan_id, scheduled_at, participants[], agenda_items[]}` | S&OP review meeting scheduled | Notification Service, Calendar |
| `commerce.sop.action-item.created` | `{action_item_id, plan_id, meeting_id, assignee_id, description, due_date, priority}` | Follow-up action from S&OP meeting | Notification Service, Dashboard |
| `commerce.sop.plan.approved` | `{plan_id, version_id, approved_by, approved_at, effective_period}` | Executive approves consensus plan | All plan consumers, Dashboard |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `commerce.order.created` | Commerce Service | Update demand view with new order data |
| `commerce.stock.updated` | Commerce Service | Update supply view with current inventory positions |
| `commerce.demand.forecast-generated` | Commerce Service | Integrate demand forecast into S&OP demand view, trigger re-evaluation |
| `manufacturing.capacity.updated` | Manufacturing Service | Update rough-cut capacity planning and supply constraints |
| `manufacturing.plan.updated` | Manufacturing Service | Update supply view with revised production schedules |
| `finance.budget.allocated` | Finance Service | Incorporate budget allocations into finance review stage |
| `finance.journal.posted` | Finance Service | Update actual-vs-plan comparison with posted financials |
| `report.ml.inference.complete` | Report Service | Apply ML-driven forecast refinements to demand view |
| `workflow.step.approved` | Workflow Service | Advance consensus workflow to next review stage |
| `config.changed` | Config Service | Update S&OP parameters, thresholds, and period definitions |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `sop_plans` | `id, tenant_id, name, horizon_type, status, period_start, period_end, product_families[], current_version_id, created_at, updated_at, created_by, updated_by, version, is_deleted` | Has many `sop_plan_versions`, `sop_scenarios`, `sop_meetings` |
| `sop_plan_versions` | `id, tenant_id, plan_id, version_number, demand_snapshot{}, supply_snapshot{}, financial_snapshot{}, changes_summary, status, approved_by, approved_at, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `sop_plans` |
| `sop_demand_views` | `id, tenant_id, plan_id, version_id, period, product_family, region_id, forecast_qty, actual_qty, order_qty, promotion_qty, source, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `sop_plans`, `sop_plan_versions` |
| `sop_supply_views` | `id, tenant_id, plan_id, version_id, period, product_family, region_id, inventory_qty, production_qty, po_qty, transfer_qty, source, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `sop_plans`, `sop_plan_versions` |
| `sop_gaps` | `id, tenant_id, plan_id, version_id, period, product_family, region_id, demand_qty, supply_qty, gap_qty, gap_type, severity, status, resolution_notes, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `sop_plans`, `sop_plan_versions` |
| `sop_scenarios` | `id, tenant_id, plan_id, name, scenario_type, assumptions{}, demand_adjustments{}, supply_adjustments{}, financial_impact{}, comparison_metrics{}, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `sop_plans` |
| `sop_meetings` | `id, tenant_id, plan_id, meeting_type, scheduled_at, duration_minutes, organizer_id, participants[], agenda_items[], minutes, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `sop_plans`, has many `sop_action_items` |
| `sop_action_items` | `id, tenant_id, plan_id, meeting_id, assignee_id, description, due_date, priority, status, resolution_notes, completed_at, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `sop_plans`, `sop_meetings` |

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `commerce.sop.default_horizon_months` | 18 | Default planning horizon in months |
| `commerce.sop.period_type` | monthly | Planning period granularity: weekly, monthly, quarterly |
| `commerce.sop.gap_alert_threshold_pct` | 10 | Percentage threshold for gap severity escalation |
| `commerce.sop.auto_reevaluate_on_forecast` | true | Automatically re-evaluate gaps when forecast updates arrive |
| `commerce.sop.max_scenarios_per_plan` | 5 | Maximum simultaneous scenarios per plan |
| `commerce.sop.meeting_reminder_hours` | 24 | Hours before meeting to send reminder notification |
| `commerce.sop.action_item_overdue_days` | 7 | Days after due date to escalate overdue action items |
| `commerce.sop.rolling_forecast_window_months` | 3 | Window for rolling forecast integration |

## KPI Scorecard Metrics

| Metric | Measurement | Frequency | Target Source |
|--------|-------------|-----------|---------------|
| Revenue vs. Plan | `Actual revenue / Planned revenue × 100` | Monthly | Approved plan |
| Gross Margin | `(Revenue - COGS) / Revenue × 100` | Monthly | Finance target |
| Inventory Turns | `COGS / Average Inventory Value` | Monthly | Industry benchmark |
| Service Level (OTIF) | `% of orders delivered on time in full` | Monthly | Service level target |
| Forecast Accuracy | `1 - MAPE (Mean Absolute Percentage Error)` | Monthly | Historical baseline |
| Plan Attainment | `% of plan goals achieved within horizon` | Quarterly | Approved plan |
| Gap Resolution Rate | `% of identified gaps resolved before period end` | Monthly | Process target |
| Meeting Effectiveness | `% of action items completed on time` | Monthly | Process target |

## Cross-References

- [Advanced Demand Management](advanced-demand-management.md) — Demand forecasts feed S&OP demand views
- [Production Scheduling](production-scheduling.md) — Production plans provide supply view data
- [Financial Reporting Studio](financial-reporting-studio.md) — Actual-vs-plan financial analysis
- [Connected Planning](connected-planning.md) — Strategic planning integration
- [Commerce Service](../06-services/commerce.md) — Commerce service specification
- [Manufacturing Service](../06-services/manufacturing.md) — Capacity and production data
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
- [NFR: Performance](../10-planning/nfr.md) — Demand-supply matching < 30s for full horizon
