# Project Management Service

| Aspect | Details |
|--------|---------|
| Port | 8017 |
| Database | `project_db` |
| Responsibilities | Project planning and execution, resource allocation, time and expense tracking, project billing, budgeting and costing, risk management, earned value management, program management, portfolio analysis |

## Features

| Feature | Description |
|---------|-------------|
| Project Planning | WBS, Gantt chart, milestones, task dependencies, Critical Path Method |
| Resource Allocation | Resource requests, assignments, utilization tracking, skills matching |
| Time & Expense (Project) | Project timesheets, expense reports, approval workflows |
| Project Billing | Time & material billing, fixed-price billing, milestone billing |
| Budget & Costing | Project budgets, cost tracking, variance analysis, burdened cost calculations |
| Risk Management | Risk identification, assessment matrix, mitigation tracking |
| Project Templates | Standard project templates, copy-from-project |
| Program Management | Program definition, project grouping, cross-project dependencies |
| Project Analytics | Project health, budget vs actual, earned value |
| Earned Value Management | PV, EV, AC, CPI, SPI, EAC, ETC |
| Project Portfolio Analysis | Portfolio balancing, investment analysis, benefit realization, interdependency mapping |

## Project Lifecycle

```
                    submit                start              hold
  ┌───────┐    ┌──────────┐    ┌──────────────┐    ┌───────────┐
  │ Draft │───>│ Planned  │───>│ In Progress  │───>│ On Hold   │
  └───────┘    └──────────┘    └──────────────┘    └───────────┘
                                     │                   │
                              complete│            resume│
                                     v                   v
                               ┌───────────┐    ┌──────────────┐
                               │ Completed │<───│ In Progress  │
                               └───────────┘    └──────────────┘
                                     │
                                close│
                                     v
                               ┌───────────┐
                               │  Closed   │
                               └───────────┘
```

| From | To | Trigger | Side Effect |
|------|----|---------|-------------|
| Draft | Planned | Submit plan | Validates WBS, budget, resource assignments |
| Planned | In Progress | Start | Publishes `project.updated`, schedules checkpoints |
| In Progress | On Hold | Hold | Records reason, freezes timesheets |
| On Hold | In Progress | Resume | Recalculates dates, publishes `project.updated` |
| In Progress | Completed | Complete | Validates tasks closed, triggers billing review |
| Completed | Closed | Close | Publishes `project.updated`, archives to Finance |

## Project Planning

### Work Breakdown Structure (WBS)

Projects decompose into a hierarchical task tree. The WBS defines scope and is the foundation for scheduling, costing, and resource allocation.

| Property | Description |
|----------|-------------|
| Task hierarchy | Parent-child with unlimited depth |
| Task types | Summary (roll-up), work (leaf), milestone (zero-duration marker) |
| Effort estimation | Estimated hours per leaf task; summaries roll up |
| Weighting | Configurable weight per task for earned value |

### Scheduling

| Capability | Description |
|------------|-------------|
| Gantt chart | Visual timeline from task dates and dependencies |
| Dependencies | FS, SS, FF, SF with configurable lead/lag |
| Critical Path Method | Forward/backward pass to identify critical path and total float |
| Milestones | Zero-duration markers with customer sign-off |

## Resource Allocation

| Capability | Description |
|------------|-------------|
| Resource requests | Submit by role, skill set, and time period |
| Assignments | Named resources to tasks with allocation % and date range |
| Skills matching | Match requests against HCM employee skill profiles |
| Utilization tracking | Allocated vs available hours across all projects |
| Overallocation detection | Flag resources exceeding 100% in any period |

## Time & Expense

Project Service owns project time/expense for billing and costing. HCM owns attendance/time-off for payroll. Finance owns expense management for reimbursement.

| Capability | Description |
|------------|-------------|
| Project timesheets | Daily time entries per task: hours, billable flag, notes |
| Expense reports | Project expenses: amount, category, receipt attachment |
| Approval workflows | Multi-level approval via Workflow Service |
| Rate cards | Role-based billing rates per project |

Timesheets follow Draft → Submitted → Approved/Rejected lifecycle. Approval publishes `project.timesheet.approved` and updates actual cost. HCM Service provides availability and skill profiles via `hr.employee.#` events. Finance Service receives approved entries for billing. Workflow Service handles approval routing.

## Project Billing

| Method | Description |
|--------|-------------|
| Time & Material | Invoice on billable hours × rate card |
| Fixed-Price | Invoice on project/milestone completion vs contract value |
| Milestone Billing | Invoice on milestone sign-off at configurable % |

1. Approved timesheets accumulate billable hours per task.
2. On billing cycle end (weekly/bi-weekly/monthly), system generates billing batch.
3. Batch calculates amounts using rate card; publishes `project.invoice.generated`.
4. Finance Service creates the accounts receivable invoice from the event.

## Budget & Costing

| Cost Element | Source | Description |
|--------------|--------|-------------|
| Labor | Approved timesheets | Hours × cost rate per resource |
| Material | Manual or commerce integration | Direct material charges |
| Expense | Approved expense reports | Travel, subsistence, other |
| Overhead (burden) | Configurable rate | % on top of labor cost |

Burdened Cost = Raw Labor Cost + (Raw Labor Cost × Burden Rate %)

| Metric | Formula |
|--------|---------|
| Cost Variance (CV) | BCWP − ACWP |
| Schedule Variance (SV) | BCWP − BCWS |
| Budget at Completion (BAC) | Sum of all budgeted costs |

## Risk Management

### Assessment Matrix

| | Low Impact | Medium Impact | High Impact |
|---|---|---|---|
| **High Probability** | Medium | High | Critical |
| **Medium Probability** | Low | Medium | High |
| **Low Probability** | Negligible | Low | Medium |

Risk score = probability (1-5) × impact (1-5). Each risk tracks strategy (Avoid/Mitigate/Transfer/Accept), mitigation actions, owner, status (Open/Mitigating/Closed/Accepted), and residual score.

## Project Templates

| Capability | Description |
|------------|-------------|
| Standard templates | Pre-built for implementation, consulting, internal project types |
| Template structure | WBS tasks, milestones, dependencies, role-based placeholders |
| Copy-from-project | Create template from existing project, stripping actuals and dates |
| Instantiation | New project from template, shifting schedule to target start date |

## Program Management

A program groups related projects under shared objectives, budgets, and executive dashboards. Cross-project dependencies between tasks in different projects within the same program are tracked and included in program-level critical path analysis.

## Project Analytics

| Dashboard | Metrics |
|-----------|---------|
| Project Health | SPI, CPI, risk heat map |
| Budget vs Actual | BAC, EAC, ETC, VAC, % complete |
| Resource Utilization | Allocated vs available hours, bench time |
| Milestone Status | Planned vs actual dates, overdue count |
| Billing Summary | Billed to date, pending, write-offs |

## Earned Value Management

| Metric | Abbr | Formula | Meaning |
|--------|------|---------|---------|
| Planned Value | PV | BAC × Planned % Complete | Budgeted cost of work scheduled |
| Earned Value | EV | BAC × Actual % Complete | Budgeted cost of work performed |
| Actual Cost | AC | Sum of actual costs | Actual cost of work performed |
| Cost Performance Index | CPI | EV / AC | < 1 = over budget |
| Schedule Performance Index | SPI | EV / PV | < 1 = behind schedule |
| Estimate at Completion | EAC | BAC / CPI (typical); AC + (BAC − EV) (atypical) | Forecasted total cost |
| Estimate to Complete | ETC | EAC − AC | Remaining cost |
| Variance at Completion | VAC | BAC − EAC | Budget surplus or deficit |
| To-Complete Performance Index | TCPI | (BAC − EV) / (BAC − AC) | Required CPI to meet BAC |

## Project Portfolio Analysis

| Capability | Description |
|------------|-------------|
| Portfolio definition | Strategic grouping with business alignment and investment categories |
| Project scoring | Multi-criteria: strategic fit, ROI, risk, resources, urgency |
| Portfolio balancing | Risk/reward, short/long-term, strategic dimension analysis |
| Resource capacity planning | Aggregate demand with capacity gap analysis |
| Investment analysis | NPV, IRR, payback period, benefit-cost ratio |
| What-if modeling | Scenario planning for allocation and budget changes |
| Benefit realization | Post-implementation tracking vs business case |
| Interdependency mapping | Cross-project dependencies and portfolio critical path |

| Metric | Formula | Description |
|--------|---------|-------------|
| NPV | Discounted cash flows − investment | Positive = value creation |
| IRR | Discount rate where NPV = 0 | Compare against hurdle rate |
| Payback Period | Investment / annual inflow | Time to recover |
| Benefit-Cost Ratio | Discounted benefits / discounted costs | > 1 = worthwhile |

## Database Tables

| Table | Purpose | Key Columns |
|-------|---------|-------------|
| `projects` | Project definitions and state | `id`, `tenant_id`, `name`, `status`, `program_id`, `customer_id`, `start_date`, `end_date`, `budget`, `billing_method`, `burden_rate`, `project_manager_id`, `template_id`, +standard |
| `project_tasks` | WBS task hierarchy | `id`, `tenant_id`, `project_id`, `parent_task_id`, `name`, `task_type`, `status`, `start_date`, `end_date`, `estimated_hours`, `actual_hours`, `percent_complete`, `weight`, +standard |
| `project_milestones` | Milestone markers with sign-off | `id`, `tenant_id`, `project_id`, `task_id`, `name`, `planned_date`, `actual_date`, `sign_off_by`, `sign_off_date`, `billing_pct`, `status`, +standard |
| `task_dependencies` | Dependency edges | `id`, `tenant_id`, `predecessor_task_id`, `successor_task_id`, `dependency_type`, `lag_days`, +standard |
| `project_resources` | Resource assignments | `id`, `tenant_id`, `project_id`, `task_id`, `resource_id`, `role`, `allocation_pct`, `start_date`, `end_date`, `cost_rate`, `bill_rate`, +standard |
| `project_timesheets` | Time entries | `id`, `tenant_id`, `project_id`, `task_id`, `resource_id`, `date`, `hours`, `billable`, `description`, `status`, `approved_by`, `approved_at`, +standard |
| `project_expenses` | Expense entries | `id`, `tenant_id`, `project_id`, `task_id`, `resource_id`, `expense_date`, `amount`, `currency`, `category`, `receipt_file_id`, `status`, `approved_by`, `approved_at`, +standard |
| `project_billings` | Billing records | `id`, `tenant_id`, `project_id`, `billing_date`, `billing_method`, `amount`, `currency`, `status`, `finance_invoice_id`, +standard |
| `project_risks` | Risk register | `id`, `tenant_id`, `project_id`, `task_id`, `title`, `category`, `probability`, `impact`, `risk_score`, `residual_score`, `strategy`, `mitigation_actions`, `owner_id`, `status`, +standard |
| `project_budgets` | Budget allocations | `id`, `tenant_id`, `project_id`, `task_id`, `cost_category`, `budgeted_amount`, `actual_amount`, `committed_amount`, `currency`, +standard |
| `rate_cards` | Role-based rates | `id`, `tenant_id`, `project_id`, `role`, `cost_rate`, `bill_rate`, `currency`, `effective_from`, `effective_to`, +standard |
| `project_templates` | Reusable templates | `id`, `tenant_id`, `name`, `project_type`, `template_data` (JSONB), `source_project_id`, +standard |
| `programs` | Program definitions | `id`, `tenant_id`, `name`, `objectives`, `budget`, `program_manager_id`, `status`, `start_date`, `end_date`, +standard |
| `program_projects` | Program membership | `id`, `tenant_id`, `program_id`, `project_id`, +standard |

> All tables include standard columns per SPEC.md §9.1. The `+standard` shorthand above indicates these columns are present.

## Events Published

| Event | Payload | Description |
|-------|---------|-------------|
| `project.created` | `{ project_id, tenant_id, name, billing_method, project_manager_id }` | New project created |
| `project.updated` | `{ project_id, tenant_id, status, changed_fields }` | Project changed |
| `project.task.created` | `{ task_id, project_id, tenant_id, task_type, parent_task_id }` | Task added to WBS |
| `project.task.completed` | `{ task_id, project_id, tenant_id, actual_hours, completed_by }` | Task completed |
| `project.milestone.reached` | `{ milestone_id, project_id, tenant_id, name, actual_date, sign_off_by }` | Milestone achieved |
| `project.timesheet.submitted` | `{ timesheet_id, project_id, tenant_id, resource_id, hours, date }` | Timesheet submitted |
| `project.timesheet.approved` | `{ timesheet_id, project_id, tenant_id, resource_id, hours, approved_by }` | Timesheet approved |
| `project.expense.submitted` | `{ expense_id, project_id, tenant_id, resource_id, amount, currency }` | Expense submitted |
| `project.invoice.generated` | `{ invoice_id, project_id, tenant_id, amount, currency, billing_method }` | Invoice generated |
| `project.risk.identified` | `{ risk_id, project_id, tenant_id, category, probability, impact, risk_score }` | Risk registered |

## Events Consumed

| Binding Pattern | Events Consumed |
|----------------|-----------------|
| `project.#` | Self-binding for saga compensation |
| `hr.employee.#` | Employee data for resource allocation |
| `commerce.order.#` | Order data for project billing |
| `config.changed` | Configuration updates |

## See Also

- [HCM Service](hr.md)
- [Finance Service](finance.md)
- [Commerce Service](commerce.md)
- [Workflow Service](workflow.md)
- [Config Service](config.md)
- [Architecture Overview](../01-architecture/overview.md)
- [Event Catalog](../04-events/catalog.md)
- [API Endpoints](../05-api/endpoints.md)
