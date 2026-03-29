# Connected Planning

Unified financial, workforce, and supply chain planning with connected models, shared assumptions, and integrated what-if simulation, managed by the Finance Service (financial planning) + HCM Service (workforce planning) + Manufacturing Service (supply chain planning) + Report Service (planning analytics).

## Overview

Connected Planning breaks down traditional planning silos by providing a unified platform where financial, workforce, and supply chain plans share assumptions, drivers, and constraints in real-time. A headcount change in the workforce plan automatically ripples through compensation expense in the financial plan and labor capacity in the supply chain plan. The system supports driver-based planning with cross-domain linkages, integrated what-if simulation across all planning domains, scenario versioning with full audit trails, and plan-vs-actual variance analysis with root-cause drill-down. Rolling forecasts replace static annual budgets, with configurable forecast horizons and automated reconciliation to actuals.

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Unified Planning Workspace** | Report Service (React) + Finance/HR/Manufacturing Services | Single UI for all planning domains with tabbed workspace; shared navigation, assumption library, and scenario selector; role-based views (finance planner, workforce planner, supply planner) |
| **Shared Assumptions Library** | Finance Service (Rust) | Centralized assumption registry: exchange rates, inflation rates, growth rates, salary increase rates, commodity prices; versioned assumptions with effective dates; change impact analysis on linked plans |
| **Financial Planning Models** | Finance Service (Rust) | Revenue planning (top-down, bottom-up, driver-based), expense planning (cost center, project, GL), cash flow planning, capital expenditure planning, balance sheet planning; multi-entity consolidation |
| **Workforce Planning Models** | HCM Service (Rust) | Headcount planning by department/role/grade, compensation planning, benefits modeling, hiring plan, attrition modeling, succession planning, contingent workforce planning |
| **Supply Chain Planning Models** | Manufacturing Service (Rust) | Demand planning with statistical forecasting, supply planning, inventory planning, production scheduling, distribution planning, procurement planning |
| **Cross-Domain Driver Linkage** | Finance Service + HCM Service + Manufacturing Service | Configurable driver links connecting planning models across domains; headcount → compensation expense → cash flow; demand forecast → production plan → COGS; change propagation with dependency ordering |
| **Integrated What-If Simulation** | Report Service (ONNX) + Finance/HR/Manufacturing Services | Multi-scenario simulation engine; modify any assumption or driver and see cascading impact across all domains; side-by-side scenario comparison with variance waterfall charts |
| **Planning Scenarios & Version Control** | Finance Service (Rust) | Scenario management: baseline, best case, worst case, custom; full version history with diff; scenario locking, approval, and promotion to official plan |
| **Planning Workflow Approvals** | Finance Service + Platform Service | Multi-level approval workflows for plan submissions; reviewer assignment, commentary, rejection with rework; approval status dashboard |
| **Plan-vs-Actual Variance** | Finance Service + Report Service | Automated variance calculation at any granularity; favorable/unfavorable classification; root-cause drill-down to contributing drivers; variance trend over time |
| **Rolling Forecasts** | Finance Service | Configurable rolling forecast windows (6/12/18 months); automated actual-to-forecast reconciliation; forecast accuracy tracking; bias detection |
| **Driver-Based Planning** | Finance Service | Define planning drivers (price, volume, headcount, utilization rate, etc.) with formulas; driver values flow through calculation chains to line items; driver sensitivity analysis |

## Connected Planning Architecture

```
┌──────────────────────────────────────────────────────────────────────────┐
│                        Unified Planning Workspace                        │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                   Shared Assumptions Library                         │ │
│  │  FX Rates │ Inflation │ Growth % │ Salary Increase │ Commodity Px  │ │
│  └──────────────────────────┬──────────────────────────────────────────┘ │
│                              │                                           │
│          ┌───────────────────┼───────────────────┐                       │
│          │                   │                   │                       │
│  ┌───────▼───────┐  ┌───────▼───────┐  ┌───────▼───────┐              │
│  │  Financial    │  │  Workforce    │  │  Supply Chain │              │
│  │  Planning     │  │  Planning     │  │  Planning     │              │
│  │               │  │               │  │               │              │
│  │ • Revenue     │  │ • Headcount   │  │ • Demand      │              │
│  │ • Expenses    │  │ • Comp & Ben  │  │ • Supply      │              │
│  │ • Cash Flow   │  │ • Hiring      │  │ • Inventory   │              │
│  │ • CapEx       │  │ • Attrition   │  │ • Production  │              │
│  │ • Balance Sht │  │ • Succession  │  │ • Procurement │              │
│  └───────┬───────┘  └───────┬───────┘  └───────┬───────┘              │
│          │                   │                   │                       │
│          └───────────────────┼───────────────────┘                       │
│                              │                                           │
│  ┌───────────────────────────▼───────────────────────────────────────┐  │
│  │              Cross-Domain Driver Linkage Engine                     │  │
│  │  Headcount ──▶ Comp Expense ──▶ Cash Flow                         │  │
│  │  Demand ──▶ Production ──▶ COGS ──▶ Gross Margin                 │  │
│  │  FX Rate ──▶ Revenue (intl) ──▶ Consolidated P&L                  │  │
│  └───────────────────────────┬───────────────────────────────────────┘  │
│                              │                                           │
│  ┌───────────────────────────▼───────────────────────────────────────┐  │
│  │              What-If Simulation & Scenario Engine                   │  │
│  │  Baseline │ Best Case │ Worst Case │ Custom Scenarios              │  │
│  │  Side-by-Side Compare │ Variance Waterfall │ Sensitivity Analysis  │  │
│  └───────────────────────────┬───────────────────────────────────────┘  │
│                              │                                           │
│  ┌───────────────────────────▼───────────────────────────────────────┐  │
│  │              Planning Analytics & Dashboards                        │  │
│  │  Plan vs Actual │ Rolling Forecast │ Variance │ Approval Status   │  │
│  └────────────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────────────┘
```

## Planning Domains

| Domain | Service | Sub-Models | Key Drivers | Cross-Domain Links |
|--------|---------|-----------|-------------|-------------------|
| **Financial** | Finance Service | Revenue, Expense, Cash Flow, CapEx, Balance Sheet | Growth rate, price, volume, cost inflation | Headcount → Comp Expense; COGS from Supply Chain |
| **Workforce** | HCM Service | Headcount, Compensation, Benefits, Hiring, Attrition | Salary increase %, hiring rate, attrition rate | Headcount → FTE cost; Available labor → Supply capacity |
| **Supply Chain** | Manufacturing Service | Demand, Supply, Inventory, Production, Procurement | Demand forecast, lead time, safety stock, yield | Demand → Revenue; Production cost → COGS |
| **Capital** | Finance Service | Capital projects, asset acquisition, depreciation | Project timeline, cost estimate, useful life | CapEx → Cash Flow; Depreciation → Expense |
| **Project** | Project Service | Resource allocation, project revenue, project cost | Utilization rate, billing rate, cost rate | Project revenue → Revenue; Resources → Workforce |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Finance Service (GL) | Finance Service | Internal API | Actual financial data for plan-vs-actual; GL account structure for planning lines |
| HCM Service | HCM Service | Internal API | Employee master data, compensation bands, org structure for workforce planning |
| Manufacturing Service | Manufacturing Service | Internal API | Production orders, BOMs, lead times, capacity data for supply chain planning |
| Project Service | Project Service | Internal API | Project plans, resource allocations, budgets for project planning |
| Report Service (ML) | Report Service | gRPC | Statistical forecasting models, demand prediction, what-if simulation engine |
| Platform Workflow | Platform Service | Event-driven | Planning approval workflows, plan submission, review, and rejection routing |
| Notification Service | Platform Service | Event-driven | Plan assignment, approval requests, variance alerts, deadline reminders |
| Consolidation | Finance Service | Internal API | Multi-entity consolidation rules, currency translation, elimination entries |
| Budget Management | Finance Service | Internal API | Budget creation from plans, budget control and enforcement |
| Commerce Service | Commerce Service | Internal API | Sales pipeline, order book, and customer demand signals |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `report.planning.scenario.created` | `{scenario_id, name, domain, baseline_scenario_id, created_by}` | New planning scenario created | Dashboard, Stakeholders |
| `report.planning.assumption.updated` | `{assumption_id, name, old_value, new_value, effective_date, impacted_plans[]}` | Shared assumption changed | All linked planning models |
| `report.planning.driver.link.triggered` | `{link_id, source_domain, target_domain, driver_name, value, propagation_chain[]}` | Cross-domain driver change propagated | Target planning models |
| `report.planning.simulation.completed` | `{simulation_id, scenario_id, domains[], duration_ms, result_summary{}}` | What-if simulation finishes | Dashboard, Stakeholders |
| `report.planning.plan.submitted` | `{plan_id, scenario_id, domain, submitted_by, period_range}` | Plan submitted for approval | Approval Workflow, Notification |
| `report.planning.plan.approved` | `{plan_id, approved_by, approved_at, promotion_target}` | Plan approved and promoted | Budget Management, Notification |
| `report.planning.variance.detected` | `{variance_id, plan_id, period, dimension, plan_value, actual_value, variance_pct}` | Plan-vs-actual variance exceeds threshold | Notification Service, Dashboard |
| `report.planning.forecast.updated` | `{forecast_id, domain, period_range, method, accuracy_score}` | Rolling forecast refreshed | Dashboard, Finance Service |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `finance.journal.posted` | Finance Service | Update actuals for plan-vs-actual variance calculation |
| `hr.employee.hired` | HR Service | Update workforce plan actuals and financial driver links |
| `hr.employee.separated` | HR Service | Update workforce plan actuals and financial driver links |
| `manufacturing.work-order.completed` | Manufacturing Service | Update supply chain plan actuals and cost drivers |
| `commerce.order.created` | Commerce Service | Feed demand signals to demand planning model |
| `workflow.step.approved` | Workflow Service | Update plan approval status |
| `report.ml.predict` | Report Service | Apply forecast model predictions to planning scenarios |
| `finance.period.closed` | Finance Service | Trigger rolling forecast refresh for closed period |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `planning_scenario` | `id, name, domain, type (baseline/best/worst/custom), status, locked, created_by` | Has many `planning_version` |
| `planning_version` | `id, scenario_id, version_number, created_at, created_by, change_description` | Has many `planning_line` |
| `planning_line` | `id, version_id, account_id, department_id, period, value, driver_ref, formula` | Belongs to `planning_version` |
| `planning_assumption` | `id, name, category, value, effective_from, effective_to, version` | Referenced by `planning_driver_link` |
| `planning_driver_link` | `id, source_domain, source_driver, target_domain, target_driver, formula, active` | Cross-domain linkage definition |
| `planning_simulation` | `id, scenario_id, assumptions_snapshot{}, results{}, triggered_by, completed_at` | Belongs to `planning_scenario` |
| `planning_approval` | `id, plan_id, scenario_id, submitted_by, submitted_at, approver_id, status, comments` | Belongs to `planning_scenario` |
| `planning_variance` | `id, plan_id, period, dimension, plan_value, actual_value, variance, variance_pct, root_cause` | Belongs to `planning_scenario` |
| `planning_forecast` | `id, domain, period_range, method, values{}, accuracy_score, generated_at` | Rolling forecast record |

## Planning Analytics & Dashboards

| Dashboard Component | Data Source | Refresh Rate | Description |
|---------------------|-----------|--------------|-------------|
| Plan Summary Scorecard | Finance Service | On demand | High-level plan vs. actual across all domains with RAG status |
| Scenario Comparison | Finance Service + Report Service | On simulation | Side-by-side scenario metrics with variance waterfall |
| Driver Sensitivity | Report Service (ML) | On demand | Tornado chart showing which drivers have most impact on plan outcomes |
| Variance Heat Map | Finance Service | Daily | Multi-dimensional variance heat map (entity × account × period) |
| Rolling Forecast Accuracy | Finance Service | Weekly | Forecast vs. actual over rolling window with bias and MAPE tracking |
| Approval Status Board | Platform Service | Real-time | Plan approval pipeline with pending, in-review, and approved status |
| Assumption Change Log | Finance Service | Real-time | Audit trail of all assumption changes with impact assessment |
| Cross-Domain Linkage Map | Finance Service | On demand | Visual graph of driver linkages across financial, workforce, and supply chain |

## Assumption Change Impact Analysis

| Change Scenario | Primary Impact | Secondary Impact (Financial) | Secondary Impact (Workforce) | Secondary Impact (Supply Chain) |
|----------------|---------------|------------------------------|------------------------------|--------------------------------|
| FX Rate +5% | International revenue | Revenue ↑5% (intl), FX gain/loss | Expat comp adjustment | Import material cost ↑ |
| Salary increase +1% | Compensation expense | Operating expense ↑ | Total comp ↑, attrition risk ↓ | Labor cost per unit ↑ |
| Demand forecast -10% | Production plan | Revenue ↓10%, margin pressure | Overtime reduction | Procurement qty ↓, inventory ↓ |
| Attrition rate +2% | Headcount plan | Recruiting cost ↑, productivity ↓ | Open positions ↑, workload ↑ | Capacity risk if skilled labor |
| Commodity price +8% | Material cost | COGS ↑, margin ↓ | No direct impact | Procurement budget ↑, renegotiation |

## Cross-References

- [Corporate Performance Management](../06-services/report.md) — Budget management and performance tracking (CPM module)
- [Profitability Analysis](../06-services/finance.md) — Profitability modeling and margin analysis (Profitability Analysis module)
- [Augmented Analytics](../06-services/report.md) — Natural language planning queries and auto-insights (Augmented Analytics module)
- [Adaptive Intelligence](adaptive-intelligence.md) — ML-powered demand forecasting and predictions
- [Finance Service](../06-services/finance.md) — Financial planning and GL operations
- [HCM Service](../06-services/hr.md) — Workforce and compensation planning
- [Manufacturing Service](../06-services/manufacturing.md) — Supply chain and production planning
