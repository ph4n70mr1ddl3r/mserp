# Workforce Scheduling

AI-optimized shift scheduling and labor demand forecasting with pattern management, compliance checking, and self-service shift management, managed by the HCM Service.

## Overview

Workforce Scheduling provides an intelligent scheduling platform that combines configurable shift patterns with AI-powered optimization to generate compliant, fair, and demand-responsive employee schedules. The module supports fixed, rotating, and flexible shift patterns, AI-driven schedule optimization based on demand forecasts and employee preferences, skill-based assignment ensuring coverage requirements are met, overtime management with authorization workflows, shift swap and drop requests with approval routing, labor law compliance checking including break rules and maximum hours, schedule publishing with change notifications, and mobile self-service for employees. It integrates with Report Service for ML-powered demand forecasting and optimization, Workflow Service for approval workflows, and Config Service for labor rules.

## Data Flow Diagram

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  Shift Patterns  │────▶│  Schedule        │────▶│  AI Optimization    │
│  (Fixed/Rotating/│     │  Builder         │     │  Engine             │
│   Flexible)      │     │                  │     │  ┌───────────────┐  │
└──────────────────┘     └──────────────────┘     │  │ Demand        │  │
                                                   │  │ Forecasting  │  │
┌──────────────────┐     ┌──────────────────┐     │  └───────┬───────┘  │
│  Employee        │────▶│  Constraint      │────▶│          │          │
│  Availability    │     │  Solver          │     └──────────┼──────────┘
│  & Skills        │     │                  │                │
└──────────────────┘     └──────────────────┘     ┌──────────▼──────────┐
                                                   │  Draft Schedule     │
┌──────────────────┐     ┌──────────────────┐     │  & Compliance Check │
│  Labor Rules &   │────▶│  Compliance      │◀────│                     │
│  Break Rules     │     │  Validator       │     └──────────┬──────────┘
└──────────────────┘     └──────────────────┘                │
                                                    ┌────────▼──────────┐
┌──────────────────┐     ┌──────────────────┐     │  Published          │
│  Workflow        │◀────│  Approval &       │◀────│  Schedule           │
│  Service         │     │  Publishing       │     │  & Notifications    │
└──────────────────┘     └──────────────────┘     └─────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Shift Pattern Management** | HCM Service (Rust) | Configurable shift templates supporting fixed schedules, rotating patterns (daily, weekly, custom), and flexible/self-select patterns |
| **AI Schedule Optimization** | HCM Service + Report Service | Constraint-satisfaction solver using ML demand forecasts, employee preferences, skill requirements, and fairness metrics to generate optimal schedules |
| **Skill-Based Assignment** | HCM Service | Coverage requirement validation ensuring assigned employees possess required skills and certifications for each shift |
| **Overtime Management** | HCM Service + Workflow Service | Overtime threshold detection, pre-authorization workflows, cost tracking, and compliance with labor regulations |
| **Shift Swap/Drop** | HCM Service | Employee-initiated shift changes with manager approval routing, skill validation for replacement, and coverage impact assessment |
| **Compliance Engine** | HCM Service | Multi-jurisdiction labor law validation including maximum hours, mandatory rest periods, break rules, and consecutive day limits |
| **Schedule Publishing** | HCM Service + Platform Service | Draft-to-published lifecycle with change notifications, version history, and employee acknowledgment tracking |
| **Storage** | PostgreSQL | All schedule, pattern, assignment, constraint, and rule data in HCM database |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Shift Pattern Manager | CRUD for shift templates and rotation rules | HCM Service |
| Schedule Builder | Period schedule generation from patterns and constraints | HCM Service |
| Optimization Engine | AI-powered schedule optimization via Report Service ML | HCM Service |
| Assignment Manager | Employee-to-shift assignment with skill validation | HCM Service |
| Compliance Validator | Labor law and break rule checking against generated schedules | HCM Service |
| Overtime Manager | Overtime detection, authorization, and cost tracking | HCM Service |
| Swap/Drop Processor | Shift change request handling, skill validation, and approval routing | HCM Service |
| Schedule Publisher | Draft-to-published lifecycle, notification dispatch, acknowledgment tracking | HCM Service |
| Constraint Solver | Availability, preference, and coverage constraint resolution | HCM Service |

## Shift Pattern Types

| Pattern | Schedule Method | Flexibility | Use Case |
|---------|----------------|-------------|----------|
| Fixed | Same shifts every period | Low | Traditional 9-to-5 office roles |
| Rotating | Cyclical shift rotation (e.g., 3-shift rotation) | Medium | Manufacturing, healthcare |
| Flexible | Employee self-select within windows | High | Retail, hospitality |
| On-Call | Scheduled availability with activation | On-demand | IT operations, maintenance |
| Split | Two separate shifts in one day | Low | Coverage-intensive roles |
| Compressed | Longer days, fewer per week | Medium | 4×10 schedules, work-life balance |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Employee Profile | HCM Service | Internal API | Employee skills, contracts, and availability |
| Leave Management | HCM Service | Internal API | Approved leave blocks schedule availability |
| Attendance Tracking | HCM Service | Event-driven | Attendance records validate schedule adherence |
| Dynamic Skills | HCM Service | Internal API | Employee skills for skill-based assignment |
| Workflow Engine | Workflow Service | Event-driven | Overtime and shift swap approval workflows |
| Report Service (ML) | Report Service | Event-driven | Demand forecasts and optimization inference results |
| Notification Service | Platform Service | Event-driven | Schedule publish alerts, swap requests, overtime notices |
| Config Service | Config Service | Event-driven | Labor rules, break policies, compliance thresholds |
| Commerce Order System | Commerce Service | Event-driven | Order volume data for demand-driven staffing |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `hr.scheduling.shift-published` | `{schedule_id, tenant_id, period_start, period_end, assignment_count, published_by, published_at}` | Draft schedule finalized and published | Notification Service, Report Service |
| `hr.scheduling.assignment-created` | `{assignment_id, tenant_id, schedule_id, employee_id, shift_date, shift_start, shift_end, role, skill_requirements (JSONB)}` | Employee assigned to a shift | Notification Service, Employee Dashboard |
| `hr.scheduling.swap-requested` | `{swap_id, tenant_id, requestor_id, current_assignment_id, target_assignment_id, reason, requested_at}` | Employee initiates shift swap or drop | Workflow Service, Notification Service |
| `hr.scheduling.swap-approved` | `{swap_id, tenant_id, approved_by, original_employee_id, new_employee_id, assignment_id, approved_at}` | Shift swap approved by manager | Notification Service, Assignment Manager |
| `hr.scheduling.overtime-authorized` | `{authorization_id, tenant_id, employee_id, schedule_id, hours, reason, approved_by, authorized_at}` | Overtime request approved | Report Service, Notification Service, Payroll |
| `hr.scheduling.optimization.completed` | `{optimization_id, tenant_id, schedule_id, score, constraints_satisfied, constraints_violated, generated_at}` | AI optimization run completes | Schedule Builder, Notification Service, Report Service |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `hr.employee.updated` | HCM Service | Update employee availability, skills, and contract terms in constraint pool |
| `hr.leave.approved` | HCM Service | Block employee availability during approved leave period |
| `hr.leave.rejected` | HCM Service | Restore employee availability for previously requested leave period |
| `hr.attendance.recorded` | HCM Service | Validate schedule adherence and flag deviations |
| `commerce.order.created` | Commerce Service | Feed order volume into demand forecast for staffing optimization |
| `report.ml.inference.complete` | Report Service | Process demand forecast and optimization results |
| `workflow.step.approved` | Workflow Service | Finalize overtime or shift swap approval |
| `config.changed` | Config Service | Reload labor rules, break policies, compliance thresholds |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `shift_patterns` | `id, tenant_id, name, pattern_type, rotation_config (JSONB), shift_definitions (JSONB), effective_from, effective_to, department_id, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | Has many `work_schedules` |
| `work_schedules` | `id, tenant_id, pattern_id, department_id, period_start, period_end, status, published_at, published_by, optimization_score, version_note, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `shift_patterns`; has many `schedule_assignments` |
| `schedule_assignments` | `id, tenant_id, schedule_id, employee_id, shift_date, shift_start, shift_end, role, skill_requirements (JSONB), break_start, break_end, is_overtime, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `work_schedules`; references `employee` |
| `shift_swap_requests` | `id, tenant_id, requestor_id, assignment_id, target_employee_id, target_assignment_id, swap_type, reason, status, approved_by, approved_at, created_at, updated_at, created_by, updated_by, version, is_deleted` | References `schedule_assignments`, two `employee` records |
| `overtime_authorizations` | `id, tenant_id, employee_id, schedule_id, authorized_hours, reason, approved_by, authorized_at, effective_date, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | References `work_schedules`, `employee` |
| `labor_rules` | `id, tenant_id, jurisdiction, rule_type, max_daily_hours, max_weekly_hours, min_rest_hours, max_consecutive_days, break_rules (JSONB), overtime_threshold_hours, effective_from, effective_to, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | Referenced by Compliance Validator |
| `scheduling_constraints` | `id, tenant_id, employee_id, constraint_type, constraint_value (JSONB), priority, effective_from, effective_to, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | References `employee` |

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `hr.scheduling.optimization.run_cadence_hours` | 24 | How often AI optimization runs for draft schedules |
| `hr.scheduling.publish_advance_days` | 14 | Days before period start to publish final schedule |
| `hr.scheduling.swap_approval_required` | true | Whether shift swaps require manager approval |
| `hr.scheduling.overtime_auto_threshold` | 40 | Weekly hours threshold triggering overtime detection |
| `hr.scheduling.compliance.strict_mode` | true | Block schedule publish if compliance violations exist |
| `hr.scheduling.max_future_period_months` | 3 | Maximum scheduling period into the future |
| `hr.scheduling.demand.forecast_window_days` | 28 | Days of historical data for demand forecasting |

## Cross-References

- [Dynamic Skills](./dynamic-skills.md) — Employee skills for skill-based shift assignment
- [HCM Service](../06-services/hcm.md) — HCM service specification
- [Commerce Service](../06-services/commerce.md) — Order data for demand-driven staffing
- [Report Service](../06-services/report.md) — ML demand forecasting and optimization
- [Workflow Engine](../06-services/workflow.md) — Overtime and shift swap approvals
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
- [NFR: Performance](../10-planning/nfr.md) — Schedule generation < 30s for 500 employees; optimization < 5min
