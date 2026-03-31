# Sales Performance Management

Quota management, commission calculation, and incentive compensation for sales organizations, managed by the CRM Service. Commerce Service provides order and invoice data; HR Service provides employee data; Workflow Service manages commission approval and dispute resolution.

> **Ownership Note**: Sales Performance Management is owned by CRM Service. Commerce Service provides order completion and invoice payment data via events (`commerce.order.*`, `commerce.invoice.*`). HR Service provides employee hiring and update data (`hr.employee.*`). Workflow Service manages commission approval and dispute resolution workflows. CRM owns quota management, commission calculation, attainment tracking, and all sales performance events.

## Overview

Sales Performance Management provides a comprehensive platform for managing sales compensation, from territory-based quota allocation through commission calculation and payout. The module supports multi-level quota cascading from company-wide targets down through regions, teams, and individual representatives. A flexible commission plan builder with a rule engine supports variable compensation including commissions, bonuses, and SPIFFs (Sales Performance Incentive Fund). Attainment tracking provides real-time visibility into quota progress, with split commission handling for team-based selling scenarios. The system includes dispute management for commission disagreements, quota ramping for new hires, what-if scenario modeling for plan design, and leaderboards with gamification features. Plan effectiveness analytics help optimize compensation programs to drive desired sales behaviors.

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Territory-Based Quota Allocation** | CRM Service (Rust) | Define sales territories by geography, industry, or account assignment; allocate quotas to territories with automatic distribution rules |
| **Multi-Level Quota Cascading** | CRM Service (Rust) | Cascade quotas from company → region → district → team → rep with configurable allocation methods (even split, weighted, custom) |
| **Commission Plan Builder** | CRM Service (Rust) | Visual rule builder for commission plans supporting rate tables, tiers, thresholds, splits, and conditions; plan versioning with effective dates |
| **Attainment Tracking** | CRM Service (Rust) | Real-time attainment calculation against quota for each period, with drill-down to contributing transactions and pipeline contribution |
| **Variable Compensation Calculation** | CRM Service (Rust) | Commission, bonus, and SPIFF calculation engine with support for retroactive adjustments, clawback rules, and holdback logic |
| **Split Commission Handling** | CRM Service (Rust) | Define commission splits across multiple reps for shared deals, with configurable split percentages and credit allocation rules |
| **Payment Scheduling** | CRM Service + Finance Service | Commission payment schedule generation aligned with payroll periods, with holdback and release logic |
| **Dispute Management** | CRM Service + Workflow Service | Commission dispute submission, investigation workflow, resolution tracking, and adjustment processing |
| **Quota Ramping** | CRM Service (Rust) | Graduated quota targets for new hires over configurable ramp period (e.g., 25% in month 1, 50% in month 2, 100% in month 3) |
| **What-If Scenario Modeling** | CRM Service (Rust) | Model proposed commission plans against historical data to project earnings distribution, cost, and behavioral impact |
| **Leaderboards & Gamification** | CRM Service (Rust) | Real-time leaderboards by team/region, achievement badges, streak tracking, and configurable competition periods |
| **Plan Effectiveness Analytics** | CRM Service + Report Service | Compensation cost vs. revenue generated, plan distribution analysis, quota attainment trends, and behavioral correlation |
| **Storage** | PostgreSQL | All quota, commission, attainment, dispute, and plan data in CRM database |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Quota Plan Manager | Create, allocate, adjust, and cascade quota plans | CRM Service |
| Commission Plan Builder | Design commission rules, rate tables, tiers, and conditions | CRM Service |
| Commission Calculation Engine | Calculate commissions, bonuses, and SPIFFs from transactions | CRM Service |
| Attainment Tracker | Track real-time quota attainment with contributing transactions | CRM Service |
| Split Commission Processor | Allocate commissions across multiple contributing reps | CRM Service |
| Payment Schedule Generator | Generate commission payment schedules aligned with payroll | CRM Service |
| Dispute Manager | Handle commission disputes from submission to resolution | CRM Service + Workflow Service |
| Quota Ramp Manager | Manage graduated quota targets for new hires | CRM Service |
| Scenario Modeling Engine | Simulate commission plans against historical data | CRM Service |
| Leaderboard Engine | Calculate rankings, achievements, and competition standings | CRM Service |
| Plan Analytics | Measure compensation plan effectiveness and ROI | CRM Service + Report Service |

## Quota Cascading Levels

| Level | Description | Allocation Method |
|-------|-------------|-------------------|
| Company | Total company revenue/product target | Board-approved annual target |
| Region | Regional allocation of company target | Proportional to territory potential |
| District | District allocation within region | Weighted by historical performance |
| Team | Team allocation within district | Based on team size and coverage |
| Individual | Rep-level quota | Ramp-adjusted for new hires, performance-weighted for existing |

## Commission Plan Types

| Plan Type | Description | Calculation |
|-----------|-------------|-------------|
| Flat Rate | Fixed percentage on all revenue | `Revenue × Rate%` |
| Tiered Rate | Increasing rate at attainment thresholds | `Revenue in Tier × Tier Rate%` |
| Accelerated | Multiplier increases above quota | `Base Rate × Accelerator (attainment > 100%)` |
| Bonus | Fixed amount for achieving milestone | `Fixed Amount IF Attainment ≥ Threshold%` |
| SPIFF | Short-term incentive for specific behavior | `Fixed Amount × Qualifying Events` |
| Matrix | Rate varies by product and volume | `Revenue × Lookup(Product, Volume Tier)` |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Sales Orders | Commerce Service | Event-driven | Order completions trigger commission eligibility evaluation |
| Invoice Payments | Commerce Service | Event-driven | Invoice payments confirm revenue for commission calculation |
| Opportunity Management | CRM Service | Internal API | Opportunity wins trigger commission credit and attainment updates |
| Employee Management | HR Service | Event-driven | New hires trigger quota ramp setup; transfers trigger quota adjustment |
| Commission Approval | Workflow Service | Event-driven | Commission plan approval, dispute resolution, exception handling |
| Configuration | Config Service | Event-driven | Commission parameters, quota rules, payment schedules |
| Notification Service | Platform Service | Event-driven | Attainment milestones, commission statements, dispute updates |
| Report Service | Report Service | Internal API | Plan effectiveness analytics, compensation dashboards |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `crm.quota.allocated` | `{allocation_id, plan_id, assignee_id, level, period, amount, currency, ramp_pct}` | Quota allocated or adjusted for a period | Dashboard, Notification, Attainment Tracker |
| `crm.quota.adjusted` | `{allocation_id, plan_id, assignee_id, old_amount, new_amount, reason, effective_period}` | Quota amount changed after initial allocation | Dashboard, Notification |
| `crm.commission.calculated` | `{transaction_id, plan_id, assignee_id, source_type, source_id, amount, currency, splits[]}` | Commission calculated from qualifying transaction | Payment Scheduler, Dashboard |
| `crm.commission.disputed` | `{dispute_id, transaction_id, assignee_id, disputed_amount, reason, submitted_at}` | Commission dispute filed by rep | Workflow, Notification |
| `crm.commission.approved` | `{transaction_id, plan_id, assignee_id, amount, currency, approved_by, payment_period}` | Commission approved for payment | Payment Scheduler, Finance, Notification |
| `crm.attainment.updated` | `{allocation_id, assignee_id, period, quota_amount, attained_amount, attainment_pct, contributing_transactions[]}` | Attainment recalculated after new transaction | Dashboard, Leaderboard, Notification |
| `crm.plan.scenario-compared` | `{scenario_id, plan_id, projected_cost, projected_revenue, earnings_distribution{}, behavioral_metrics{}}` | What-if scenario simulation completed | Dashboard, Plan Analytics |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `commerce.order.completed` | Commerce Service | Evaluate commission eligibility, calculate commission, update attainment |
| `commerce.invoice.paid` | Commerce Service | Confirm revenue for commission calculation and attainment |
| `crm.opportunity.won` | CRM Service | Trigger commission credit and attainment update |
| `hr.employee.hired` | HR Service | Create quota allocation with ramp schedule for new hire |
| `hr.employee.updated` | HR Service | Adjust quota on transfer, promotion, or team change |
| `workflow.step.approved` | Workflow Service | Finalize commission approval or dispute resolution |
| `config.changed` | Config Service | Update commission parameters, quota rules, payment schedule |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `quota_plans` | `id, tenant_id, name, plan_year, target_amount, currency, status, cascading_method, effective_from, effective_to, created_at, updated_at, created_by, updated_by, version, is_deleted` | Has many `quota_allocations` |
| `quota_allocations` | `id, tenant_id, plan_id, assignee_id, level, parent_allocation_id, period, allocated_amount, currency, ramp_pct, adjustment_reason, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `quota_plans`, optionally has parent `quota_allocations` |
| `commission_plans` | `id, tenant_id, name, plan_type, effective_from, effective_to, status, rules_snapshot{}, total_budget, currency, created_at, updated_at, created_by, updated_by, version, is_deleted` | Has many `commission_rules`, `commission_transactions` |
| `commission_rules` | `id, tenant_id, plan_id, rule_type, priority, conditions{}, rate_table{}, thresholds{}, splits_config{}, is_active, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `commission_plans` |
| `attainment_records` | `id, tenant_id, allocation_id, assignee_id, period, quota_amount, attained_amount, currency, attainment_pct, pipeline_amount, contributing_count, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `quota_allocations` |
| `commission_transactions` | `id, tenant_id, plan_id, rule_id, assignee_id, source_type, source_id, gross_amount, commission_amount, currency, splits[], status, payment_period, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `commission_plans`, `commission_rules` |
| `commission_disputes` | `id, tenant_id, transaction_id, assignee_id, disputed_amount, currency, reason, status, resolution, resolved_by, resolved_at, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `commission_transactions` |
| `plan_scenarios` | `id, tenant_id, plan_id, name, scenario_type, proposed_rules{}, projected_cost, projected_revenue, earnings_distribution{}, behavioral_metrics{}, comparison_baseline_id, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `commission_plans` |

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `crm.spb.commission_calculation_frequency` | daily | How often commission calculations run: real-time, daily, monthly |
| `crm.spb.default_ramp_months` | 3 | Default ramp period in months for new hire quotas |
| `crm.spb.ramp_schedule` | [25, 50, 100] | Percentage of full quota for each ramp month |
| `crm.spb.holdback_pct` | 10 | Percentage of commission held back until deal is fully paid |
| `crm.spb.clawback_window_months` | 6 | Months after payment to allow clawback for cancelled deals |
| `crm.spb.dispute_resolution_days` | 14 | Days allowed for commission dispute resolution |
| `crm.spb.leaderboard_refresh_minutes` | 60 | Minutes between leaderboard recalculation |
| `crm.spb.max_splits_per_transaction` | 5 | Maximum commission splits per transaction |

## Cross-References

- [Order Orchestration](order-orchestration.md) — Completed orders drive commission eligibility
- [Sales Performance](../06-services/crm.md) — CRM service specification
- [Commerce Service](../06-services/commerce.md) — Order and invoice data
- [HR Service](../06-services/hr.md) — Employee data for quota management
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
- [NFR: Performance](../10-planning/nfr.md) — Commission calculation < 15s per batch run
