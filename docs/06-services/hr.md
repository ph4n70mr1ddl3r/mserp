# HCM Service

| Aspect | Details |
|--------|---------|
| Port | 8012 |
| Database | `hr_db` |
| Responsibilities | Employee lifecycle, payroll, recruitment, talent management, benefits, workforce planning, self-service portal |

## Features

| Feature | Description |
|---------|-------------|
| Core HR | Employee records, org chart, position management, global HR |
| Time & Attendance | Timesheets, time-off management, absence tracking |
| Leave Management | Leave policies, accruals, requests, approvals |
| Payroll | Payroll processing, tax calculations, payslips, multi-country localizations |
| Recruitment | Job postings, applicant tracking, interview scheduling, offer management |
| Onboarding | New hire workflows, document collection, equipment provisioning |
| Performance Management | Goals, reviews, 360 feedback, calibration |
| Talent Review & Succession | Talent pools, succession planning, readiness assessment |
| Workforce Modeling | Workforce planning, demand forecasting, scenario modeling |
| Learning & Development | Training catalog, enrollments, certifications, compliance training |
| Compensation | Salary structures, merit planning, variable pay, benchmarking |
| Benefits Administration | Benefit plans, enrollment, eligibility, carrier integration |
| Employee Self-Service Portal | Personal info, payslips, benefits enrollment, time-off requests, expense submission |
| HR Analytics | Turnover, headcount, diversity, compensation analysis |
| Global HR Localization | Country-specific localizations, regulatory compliance, localized reporting |
| Document Management | Employee document storage, e-signatures |

## Employee Lifecycle State Machine

```
                    ┌────────────┐
                    │  Applicant  │
                    └─────┬──────┘
                          │ hire (offer accepted, onboarding complete)
                          ▼
                    ┌────────────┐
              ┌────▶│   Hired     │◀───┐
              │     └─────┬──────┘    │
              │           │ activate  │ reinstate
              │           ▼           │
              │     ┌────────────┐    │
  terminate ──┤     │   Active    │────┘
  (from any)  │     └──┬───┬─────┘
              │        │   │
              │  leave │   │ terminate
              │ request│   │
              │        ▼   ▼
              │  ┌──────────┐  ┌──────────────┐
              │  │ On Leave  │  │  Terminated   │
              │  └────┬─────┘  └──────────────┘
              │       │
              └───────┘
               return from leave
```

### States and Transitions

| From | To | Trigger | Side Effects |
|------|----|---------|--------------|
| Applicant | Hired | Offer accepted, onboarding checklist complete | Create employee record, assign position, publish `hr.employee.hired` |
| Hired | Active | First day activation (automatic or manual) | Provision accounts via Identity Service, publish `hr.employee.created` |
| Active | On Leave | Leave request approved | Update availability, delegate approvals, publish `hr.leave.approved` |
| On Leave | Active | Leave end date reached or early return | Restore availability, recalculate leave balance |
| Active | Terminated | Separation processed | Revoke access, trigger offboarding workflow, publish `hr.employee.separated` |
| On Leave | Terminated | Separation during leave | Revoke access, settle final pay, publish `hr.employee.separated` |

## Payroll Processing Pipeline

```
┌──────────────┐    ┌──────────────────┐    ┌─────────────────┐    ┌──────────────┐
│ Payroll Run   │───▶│ Earnings &       │───▶│ Tax Calculation  │───▶│ Net Pay       │
│ Initialization│    │ Deductions       │    │ (Multi-Country)  │    │ Calculation   │
└──────────────┘    └──────────────────┘    └─────────────────┘    └──────┬───────┘
                                                                            │
                                            ┌───────────────────────────────┘
                                            ▼
                              ┌──────────────────────────┐
                              │ GL Posting to Finance     │
                              │ (hr.payroll.processed)    │
                              └──────────┬───────────────┘
                                         │
                           ┌─────────────┴─────────────┐
                           ▼                           ▼
                 ┌──────────────────┐        ┌──────────────────┐
                 │ Payslip           │        │ Bank File         │
                 │ Generation        │        │ Generation        │
                 └──────────────────┘        └──────────────────┘
```

### Pipeline Stages

| Stage | Description |
|-------|-------------|
| Payroll Run Initialization | Create payroll run for a period and country, lock attendance and leave data |
| Earnings & Deductions | Calculate gross pay from salary structures, overtime, bonuses, deductions (benefits, loans, garnishments) |
| Tax Calculation | Apply country-specific tax rules, social contributions, statutory deductions; supports multiple jurisdictions |
| Net Pay Calculation | Compute final net pay after all adjustments and pre/post-tax deductions |
| GL Posting | Publish `hr.payroll.processed` event with salary expense, tax liability, and benefit cost breakdowns for Finance Service journal entry creation |
| Payslip Generation | Generate individual payslips as PDF, store in employee documents, notify employees via Platform notification service |
| Bank File Generation | Produce payment files in country-specific formats (SEPA, BACS, ACH, wire transfer) for treasury disbursement |

### Multi-Country Payroll

| Capability | Description |
|------------|-------------|
| Country Localizations | Plug-in tax engines per jurisdiction with country-specific rules for income tax, social security, pension, and statutory deductions |
| Multiple Calendars | Support varying payroll frequencies (monthly, semi-monthly, bi-weekly, weekly) per country or entity |
| Regulatory Compliance | Automatic updates for tax bracket changes, new statutory requirements, and reporting obligations per jurisdiction |
| Currency Handling | Payroll calculated in local currency with exchange rate integration from Finance Service |
| Consolidated Reporting | Aggregated multi-country payroll analytics across all entities |

## Modules

### Core HR

Employee records with full biographical, employment, and contact data. Organization chart with hierarchical reporting structures. Position management including headcount, reporting lines, and job grade mapping. Global HR supports country-specific employment fields and regulatory requirements.

### Time & Attendance

Check-in/out with geolocation tracking and IP validation. Timesheet submission and approval workflows. Overtime calculation based on configurable rules per jurisdiction. Absence tracking with automatic leave balance adjustments.

### Leave Management

Configurable leave types (vacation, sick, parental, unpaid) with accrual rules per policy. Leave balance tracking with carry-forward and encashment rules. Leave request submission, multi-level approval via Workflow Service, and calendar integration for team visibility.

### Payroll

Salary structures with components (basic, allowances, deductions). Payroll run lifecycle: draft → calculated → confirmed → posted. Tax calculation engine with jurisdiction-specific rules. Payslip generation and secure distribution. Bank file export in standard formats.

### Recruitment

Job requisition lifecycle: draft → approved → posted → closed. Applicant tracking with status progression (applied → screened → interviewed → offered → hired). Interview scheduling with panel management. Offer management with approval workflows.

### Onboarding

Configurable onboarding checklists per role or department. Document collection with e-signature support. Equipment and IT provisioning request integration. New hire orientation task tracking.

### Performance Management

Review cycle management (annual, quarterly, mid-year, 360°). Goal setting with cascading alignment to organizational objectives. Competency frameworks with rating scales. Calibration sessions for rating normalization.

### Talent Review & Succession

Talent pool management with nine-box grid classification. Succession plans for key positions with readiness assessment. Development action tracking for succession candidates. What-if scenario modeling for leadership gaps.

### Workforce Modeling

Headcount planning with current state and target scenarios. Demand forecasting based on business growth projections. Cost impact modeling for hiring, restructuring, and attrition. Simulation results with publishable workforce plans.

### Learning & Development

Training course catalog with categories, prerequisites, and capacity. Enrollment with approval workflows and waitlist management. Completion tracking with certificate generation. Skills matrix linking courses to competency development.

### Compensation

Salary structures with grade and step progression. Compa-ratio analysis against market survey data. Merit planning with budget allocation and guidelines. Variable pay plans including bonus and commission structures.

### Benefits Administration

Benefit plan catalog (medical, dental, vision, life, retirement). Open enrollment windows with eligibility rules. Dependent and beneficiary management. Cost sharing calculations (employer vs employee contributions). Carrier integration for enrollment feeds.

### HR Analytics

Turnover analysis with voluntary/involuntary breakdown. Time-to-fill and recruitment funnel metrics. Diversity and inclusion dashboards. Compensation equity analysis. Predictive attrition modeling.

### Global HR Localization

Country-specific employment law compliance modules. Localized reporting for statutory requirements. Jurisdiction-specific employment contract templates. Regulatory change tracking and impact assessment.

### Document Management

Employee document storage linked to personnel files. E-signature workflows for employment contracts and policy acknowledgments. Document templates for standard HR correspondence. Retention policies per document type and jurisdiction.

### Goals & Objectives

| Module | Description |
|--------|-------------|
| Cascading Goals | Hierarchical goal cascading from organizational objectives to department, team, and individual goals |
| Alignment | Goal alignment visualization showing how individual goals contribute to strategic objectives |
| Progress Tracking | Quantitative and qualitative progress tracking with check-ins, milestones, and status updates |
| Goal Templates | Reusable goal templates with configurable metrics, weightings, and completion criteria |

### Career Development

| Module | Description |
|--------|-------------|
| Career Paths | Configurable career paths with role progressions, prerequisites, and competency requirements |
| Competency Frameworks | Multi-level competency frameworks with proficiency scales, behavioral indicators, and assessment criteria |
| Development Plans | Individual development plans linking competency gaps to training, mentoring, and stretch assignments |
| Skill Gap Analysis | Automated skill gap analysis comparing current competencies against target role requirements |

### Structured Onboarding

| Module | Description |
|--------|-------------|
| Journey Builder | Visual onboarding journey builder with configurable phases, tasks, and milestones per role or department |
| Task Management | Onboarding task assignment, tracking, and escalation with deadline management and completion verification |
| Equipment Provisioning | Automated equipment and IT provisioning requests integrated with ITSM and asset management |
| Buddy Assignment | New hire buddy or mentor assignment with structured check-in schedules and feedback collection |

## Employee Self-Service Portal

The Employee Self-Service Portal is owned and operated by the HCM Service. It provides employees with direct access to HR functions without requiring HR staff intermediation.

### Portal Capabilities

| Capability | Description |
|------------|-------------|
| Personal Information | View and request updates to personal details, emergency contacts, bank accounts |
| Payslip Access | View and download current and historical payslips |
| Benefits Enrollment | Enroll in or modify benefit plans during open enrollment or qualifying life events |
| Time-Off Requests | Submit leave requests, view balances, check team calendar |
| Attendance | Self-service check-in/out, timesheet review and submission |
| Expense Submission | Submit expense reports for approval (routed to Finance for reimbursement) |
| Training Enrollment | Browse course catalog, enroll in training, track completions |
| Company Directory | Searchable employee directory with org chart navigation |
| Document Access | View employment documents, tax forms, and company policies |
| Performance Goals | View and update personal goals, check review status |

### Platform Service Integration

The portal relies on Platform Service for cross-cutting capabilities:

| Platform Capability | Usage |
|---------------------|-------|
| Notification Service | Email, push, and in-app notifications for leave approvals, payslip availability, enrollment reminders |
| File Storage Service | Document upload/download, payslip PDF storage, profile photo storage |

## Database Tables

All tables include standard columns: `id` UUID PK, `tenant_id` UUID, `created_at` TIMESTAMPTZ, `updated_at` TIMESTAMPTZ, `created_by` UUID, `updated_by` UUID, `version` INT, `is_deleted` BOOLEAN. RLS policies enforce `tenant_id` isolation on every table.

| Table | Purpose | Key Columns |
|-------|---------|-------------|
| `employees` | Employee master records | `employee_number` (unique per tenant), `first_name`, `last_name`, `email`, `phone`, `hire_date`, `termination_date`, `status` (applicant/hired/active/on_leave/terminated), `position_id` FK, `manager_id` FK (self-ref), `department_id`, `location_id`, `employment_type` (full_time/part_time/contractor/intern) |
| `positions` | Job positions in org hierarchy | `position_code` (unique per tenant), `title`, `department_id`, `reports_to` FK (self-ref), `grade_id`, `headcount`, `filled_count`, `status` (open/filled/closed) |
| `salary_structures` | Compensation components per position/grade | `employee_id` FK, `position_id` FK, `basic_salary`, `allowances` JSONB, `deductions` JSONB, `currency`, `effective_from`, `effective_to` |
| `attendance_records` | Daily check-in/out records | `employee_id` FK, `check_in` TIMESTAMPTZ, `check_out` TIMESTAMPTZ, `hours_worked` DECIMAL, `overtime_hours` DECIMAL, `location_lat`, `location_lng`, `ip_address`, `source` (web/mobile/biometric) |
| `timesheets` | Periodic time entries for approval | `employee_id` FK, `period_start` DATE, `period_end` DATE, `entries` JSONB, `total_hours` DECIMAL, `status` (draft/submitted/approved/rejected), `approved_by` FK |
| `leave_types` | Configurable leave categories | `code` (unique per tenant), `name`, `is_paid` BOOLEAN, `accrual_rule` JSONB, `carry_forward_limit`, `encashment_allowed` BOOLEAN |
| `leave_balances` | Leave accrual and usage per employee | `employee_id` FK, `leave_type_id` FK, `accrued` DECIMAL, `used` DECIMAL, `carried_forward` DECIMAL, `year` INT |
| `leave_requests` | Leave requests and approvals | `employee_id` FK, `leave_type_id` FK, `start_date` DATE, `end_date` DATE, `reason`, `status` (pending/approved/rejected/cancelled), `approved_by` FK, `workflow_instance_id` |
| `payroll_runs` | Payroll processing runs | `run_number` (unique per tenant), `period_start` DATE, `period_end` DATE, `country_code`, `currency`, `status` (draft/calculated/confirmed/posted), `total_gross` DECIMAL, `total_net` DECIMAL, `total_tax` DECIMAL, `posted_at` TIMESTAMPTZ |
| `payroll_items` | Individual payroll line items per employee | `payroll_run_id` FK, `employee_id` FK, `gross_pay` DECIMAL, `tax_amount` DECIMAL, `deductions` JSONB, `net_pay` DECIMAL, `bank_account` (encrypted), `payslip_document_id` |
| `recruitment_requisitions` | Job opening requests | `title`, `department_id`, `position_id` FK, `headcount` INT, `priority`, `status` (draft/approved/posted/closed), `hiring_manager_id` FK, `budget_min` DECIMAL, `budget_max` DECIMAL, `description` TEXT |
| `applicants` | Job application records | `requisition_id` FK, `first_name`, `last_name`, `email`, `phone`, `resume_document_id`, `source`, `status` (applied/screened/interviewed/offered/hired/rejected), `interview_scores` JSONB, `offer_details` JSONB |
| `performance_reviews` | Performance evaluation records | `employee_id` FK, `reviewer_id` FK, `cycle_id` FK, `review_type` (annual/quarterly/mid_year/360), `goals` JSONB, `ratings` JSONB, `overall_rating` DECIMAL, `status` (draft/in_progress/completed/calibrated), `submitted_at` TIMESTAMPTZ |
| `review_cycles` | Performance review cycle definitions | `name`, `type` (annual/quarterly/360), `start_date` DATE, `end_date` DATE, `status` (planned/active/closed), `calibration_status` BOOLEAN |
| `talent_reviews` | Talent assessment and nine-box classification | `employee_id` FK, `cycle_id` FK, `performance_rating` DECIMAL, `potential_rating` DECIMAL, `nine_box_position` VARCHAR, `readiness_level` (ready_now/ready_1yr/ready_2yr/ready_3yr), `notes` TEXT |
| `succession_plans` | Succession planning for key positions | `position_id` FK, `candidate_employee_id` FK, `readiness_level`, `development_actions` JSONB, `target_date` DATE, `status` (active/completed/withdrawn) |
| `training_courses` | Training course catalog | `code` (unique per tenant), `title`, `description` TEXT, `category`, `duration_hours` INT, `delivery_mode` (online/classroom/blended), `max_capacity` INT, `prerequisites` JSONB, `status` (active/archived) |
| `training_enrollments` | Employee course enrollments | `employee_id` FK, `course_id` FK, `enrolled_at` TIMESTAMPTZ, `completed_at` TIMESTAMPTZ, `score` DECIMAL, `certificate_document_id`, `status` (enrolled/in_progress/completed/failed/withdrawn) |
| `benefit_plans` | Available benefit plan definitions | `code` (unique per tenant), `name`, `type` (medical/dental/vision/life/retirement/other), `carrier`, `employee_cost` DECIMAL, `employer_cost` DECIMAL, `currency`, `eligibility_rule` JSONB, `enrollment_window_start` DATE, `enrollment_window_end` DATE, `status` (active/closed) |
| `benefit_enrollments` | Employee benefit selections | `employee_id` FK, `benefit_plan_id` FK, `enrolled_at` TIMESTAMPTZ, `dependents` JSONB, `coverage_level` (individual/family/employee_plus_spouse/employee_plus_child), `status` (active/waived/terminated), `effective_from` DATE, `effective_to` DATE |
| `compensation_plans` | Merit and variable pay plans | `name`, `type` (merit/bonus/commission), `fiscal_year` INT, `budget_pool` DECIMAL, `guidelines` JSONB, `status` (draft/active/closed) |
| `workforce_simulations` | Workforce planning scenarios | `name`, `scenario_type` (growth/restructuring/attrition), `assumptions` JSONB, `results` JSONB, `headcount_impact` JSONB, `cost_impact` JSONB, `status` (running/completed/failed) |
| `employee_documents` | Employee file attachments | `employee_id` FK, `document_type` (contract/payslip/certificate/policy/tax_form/other), `title`, `file_storage_key`, `file_size` BIGINT, `mime_type`, `e_signed` BOOLEAN, `retention_until` DATE |
| `self_service_requests` | Employee-initiated change requests | `employee_id` FK, `request_type` (personal_info/bank_details/benefits/leave/expense/training), `payload` JSONB, `status` (pending/approved/rejected), `approved_by` FK, `workflow_instance_id` |

## Events Published

| Event | Payload | Description |
|-------|---------|-------------|
| `hr.employee.created` | `{ employee_id, tenant_id, employee_number, position_id, department_id }` | Employee record created |
| `hr.employee.hired` | `{ employee_id, tenant_id, hire_date, position_id, manager_id }` | Employee hired and onboarding initiated |
| `hr.employee.updated` | `{ employee_id, tenant_id, changed_fields, old_values, new_values }` | Employee profile updated |
| `hr.employee.separated` | `{ employee_id, tenant_id, separation_date, reason, position_id, department_id }` | Employee offboarded |
| `hr.attendance.recorded` | `{ employee_id, tenant_id, date, check_in, check_out, hours_worked, overtime_hours }` | Attendance check-in/out |
| `hr.leave.requested` | `{ leave_request_id, employee_id, tenant_id, leave_type, start_date, end_date, days }` | Leave request submitted |
| `hr.leave.approved` | `{ leave_request_id, employee_id, tenant_id, approved_by, days }` | Leave request approved |
| `hr.leave.rejected` | `{ leave_request_id, employee_id, tenant_id, rejected_by, reason }` | Leave request rejected |
| `hr.payroll.processed` | `{ payroll_run_id, tenant_id, period_start, period_end, country_code, total_gross, total_tax, total_net, gl_entries }` | Payroll run completed |
| `hr.payroll.multi-country.completed` | `{ cycle_id, tenant_id, countries, total_employees, total_gross, total_net }` | Multi-country payroll cycle completed |
| `hr.requisition.created` | `{ requisition_id, tenant_id, title, department_id, headcount, priority }` | Job requisition created |
| `hr.applicant.submitted` | `{ applicant_id, tenant_id, requisition_id, name, email, source }` | Job application submitted |
| `hr.review.started` | `{ cycle_id, tenant_id, review_type, start_date, end_date, participant_count }` | Performance review cycle started |
| `hr.review.completed` | `{ review_id, employee_id, tenant_id, cycle_id, overall_rating }` | Performance review completed |
| `hr.training.completed` | `{ enrollment_id, employee_id, tenant_id, course_id, score, passed }` | Training course completed |
| `hr.talent-review.initiated` | `{ cycle_id, tenant_id, participant_count }` | Talent review process initiated |
| `hr.succession.plan-created` | `{ plan_id, tenant_id, position_id, candidate_id, readiness_level, target_date }` | Succession plan created for a key position |
| `hr.workforce.simulation-completed` | `{ simulation_id, tenant_id, scenario_type, headcount_impact, cost_impact }` | Workforce modeling simulation completed |

## Events Consumed

| Event | Source | Handler |
|-------|--------|---------|
| `workflow.step.#` | Workflow Service | Approval result processing for leave requests, recruitment requisitions, compensation changes, and performance calibration |
| `config.changed` | Config Service | Reload HR configuration (leave policies, tax rules, payroll parameters, benefit plan settings) |

> **Note:** Per the event-driven architecture, HR Service's inbox queue also binds to `hr.#` for saga compensation patterns.

## Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `HR_EMPLOYEE_EXISTS` | 409 | Employee with this ID already exists |
| `HR_LEAVE_BALANCE_EXCEEDED` | 409 | Not enough leave balance |
| `HR_ATTENDANCE_CONFLICT` | 409 | Overlapping attendance record |
| `HR_PAYROLL_ALREADY_PROCESSED` | 409 | Payroll for this period already processed |
| `HR_REQUISITION_CLOSED` | 409 | Job requisition is closed |
| `HR_SUCCESSION_PLAN_EXISTS` | 409 | Succession plan already exists for this position |

## See Also

- [Finance Service](finance.md) — GL posting for payroll, expense reimbursement
- [Identity Service](identity.md) — User account provisioning for new hires, access revocation on termination
- [Workflow Service](workflow.md) — Approval chains for leave, recruitment, compensation
- [Platform Service](platform.md) — Notification delivery, file storage for documents and payslips
- [API Endpoints](../05-api/endpoints.md) — HR endpoint listing
- [Event Catalog](../04-events/catalog.md) — HR event definitions
- [Architecture Overview](../01-architecture/overview.md)
