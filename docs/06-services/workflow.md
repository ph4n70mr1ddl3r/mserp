# Workflow Service

| Aspect | Details |
|--------|---------|
| Port | 8015 |
| Database | `workflow_db` |
| Responsibilities | BPMN 2.0 process engine, approval chains, SLA management, business rules, SoD enforcement |

## Features

| Feature | Description |
|---------|-------------|
| BPMN 2.0 Engine | BPMN-conformant process execution with full element support |
| Approval Chains | Multi-level approvals, sequential/parallel approvers, delegation, proxy |
| Escalation Management | Time-based escalation, notification escalation, auto-reassignment |
| Process Templates | Six pre-built workflow templates for common business processes |
| SLA Management | SLA definitions per workflow type, deadline tracking, breach alerts |
| Business Rules Engine | Declarative rule definition, evaluation, versioning for conditional routing |
| Process Monitoring | Real-time instance tracking, bottleneck detection, full audit trail |
| Email Approvals | Approve/reject via email reply for simple single-step approvals |
| SoD Enforcement | Segregation of Duties rule definitions, conflict detection in approvals |

## BPMN 2.0 Engine

### Supported Elements

| Element Type | Elements | Description |
|--------------|----------|-------------|
| Tasks | User Task, Service Task, Script Task, Manual Task | Executable work units; user tasks assign to humans, service tasks invoke APIs, script tasks run inline expressions |
| Gateways | Exclusive (XOR), Inclusive (OR), Parallel (AND), Event-based | Control flow divergence and convergence; event-based gateway triggers on first occurring event |
| Events | Start Event (none, timer, message, signal), End Event (none, terminate, error, escalation), Intermediate Catch/Throw (timer, message, signal, link), Boundary Event (timer, error, escalation, signal) | Lifecycle events attached to processes and activities |
| Subprocesses | Embedded Subprocess, Event Subprocess, Call Activity | Embedded subprocesses share parent scope; call activities invoke reusable process definitions |
| Data | Data Object, Data Store, Data Input/Output | Data flow declarations; runtime values stored in process variables |
| Swimlanes | Pool, Lane | Visual org boundaries; lanes map to `tenant_id` and department for routing |
| Artifacts | Text Annotation, Group | Documentation and visual grouping only (non-executable) |

### Process Execution

| Property | Value |
|----------|-------|
| Persistence | Process state persisted to `process_instances` and `process_steps` after every state transition |
| Variables | Typed process variables (string, number, boolean, date, JSON); scoped to instance, subprocess, or step |
| Expressions | FEEL (Friendly Enough Expression Language) for conditional expressions and gateway routing |
| Timer jobs | Cron-based and duration-based timers stored in job queue; polled at 10-second intervals |
| Error handling | Uncaught errors propagate to parent scope; boundary error events catch and route to recovery paths |
| Versioning | Process definitions are versioned; running instances continue on the version they started on |

## Approval Chains

### Configuration

| Property | Description |
|----------|-------------|
| Chain type | Sequential (one approver at a time) or parallel (all approvers simultaneously) |
| Levels | Unlimited approval levels; each level defines one or more approvers |
| Approvers per level | Single user, role-based, or dynamic (resolved at runtime via business rule) |
| Quorum | For parallel chains: minimum number of approvals required before advancing (default: all) |
| Delegation | Approver may delegate to another user; delegation is recorded in audit trail |
| Proxy | Designated proxy approver activates when primary is absent (vacation, leave) |

- Delegation: ad-hoc via API, scheduled (absence proxy), or chain-of-delegation (if `allow_redelegate: true`).
- Escalation within chains: warning notification at SLA threshold → escalate to manager on breach → auto-approve/reject/notify admin at max depth.

## Escalation Management

### Escalation Rule Definition

| Field | Type | Description |
|-------|------|-------------|
| `trigger_type` | enum | `duration` (time elapsed) or `condition` (FEEL expression evaluates to true) |
| `duration_minutes` | int | Minutes after step activation before escalation fires (for `duration` type) |
| `notification_template` | UUID | Email/notification template sent to escalation target |
| `action` | enum | `notify`, `reassign`, `auto_approve`, `auto_reject` |
| `reassign_to` | UUID | Target user or role for reassignment action |
| `max_retries` | int | Maximum times the escalation rule may fire (default: 1) |

### Escalation Flow

1. Step is created with an associated SLA deadline.
2. A timer job is scheduled based on the escalation rule's `duration_minutes`.
3. If the step is actioned before the timer fires, the job is cancelled.
4. On timer fire, the escalation action executes (notify, reassign, auto-approve, or auto-reject).
5. The `workflow.step.escalated` event is published.
6. If `max_retries > 1`, a new timer is scheduled for the next retry.

## Process Templates

| Template | Source Event | Approval Chain | Typical Routing |
|----------|-------------|----------------|-----------------|
| Purchase Requisition Approval | `finance.purchase-order.created` (from Commerce/Finance) | Sequential by amount: < $5K → manager; < $25K → manager + finance; >= $25K → manager + finance + CFO | Amount-based via business rule |
| Expense Approval | `finance.expense.submitted` | Sequential: employee → manager; amounts > $10K add finance review | Amount and category routing |
| Leave Approval | `hr.leave.submitted` | Sequential: employee → manager; > 5 consecutive days adds HR review | Duration-based routing |
| Contract Approval | `commerce.order.created` (contract type) | Sequential: requester → legal → finance → executive (amount threshold) | Amount, contract type, and department routing |
| Credit Limit Approval | `commerce.credit-limit.requested` | Sequential: sales rep → finance manager → CFO (> $100K) | Amount-based with finance escalation |
| Change Order Approval | `manufacturing.eco.submitted` | Sequential: engineer → engineering manager → quality → production | Department and change-impact routing |

Templates are versioned BPMN definitions in `process_definitions` with configurable parameters (thresholds, approver roles) exposed as process variables. Tenants can clone and customize; originals are read-only. Listed via `GET /api/v1/workflow/templates`.

## SLA Management

### SLA Definition Format

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | SLA definition unique identifier |
| `name` | string | Human-readable SLA name |
| `workflow_type` | string | BPMN process definition key this SLA applies to |
| `response_time_minutes` | int | Maximum time before first action on a step |
| `resolution_time_minutes` | int | Maximum time to complete the entire workflow instance |
| `business_calendar_id` | UUID | Reference to business hours calendar (excludes weekends, holidays) |
| `priority` | enum | `low`, `medium`, `high`, `critical`; affects escalation speed multiplier |
| `notification_channels` | string[] | Channels for breach alerts: `email`, `in_app`, `webhook` |

### Business Hours Calendars

Calendars define working days, start/end times, and holiday lists per tenant. SLA durations are calculated in business hours only; multiple calendars supported per tenant (e.g., regions, departments).

### SLA Tracking

Tracked metrics: time to first action, time to resolution, SLA breach count, and average completion time per workflow type.

## Business Rules Engine

### Rule Definition

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Rule unique identifier |
| `name` | string | Human-readable rule name |
| `condition` | string | FEEL expression evaluated against process variables (e.g., `amount > 50000 and department = "sales"`) |
| `action` | string | Outcome when condition is true: route to approver, set variable, trigger escalation |
| `priority` | int | Evaluation order; lower number evaluated first |
| `status` | enum | `draft`, `active`, `archived` |

### Rule Evaluation

1. Rules are grouped by context (workflow type + step).
2. On step activation, all active rules for that context are loaded, sorted by priority.
3. Rules are evaluated sequentially; first matching rule determines the action.
4. If no rule matches, default routing from the BPMN diagram applies.
5. Evaluation results are logged to `process_steps` for audit.

### Rule Versioning

- Every rule change creates a new version in `rule_versions` (immutable audit trail).
- Running instances use the rule version that was active at instance start time.
- Rule rollback restores a previous version as the new active version.

## Process Monitoring

### Real-Time State

| Endpoint | Description |
|----------|-------------|
| `GET /api/v1/workflow/instances` | List all running instances with state and timing |
| `GET /api/v1/workflow/instances/{id}` | Full instance state: current step, history, variables, SLA status |
| `GET /api/v1/workflow/my-tasks` | Pending tasks for the authenticated user |

### Instance States

| State | Description |
|-------|-------------|
| `pending` | Instance created, awaiting start |
| `running` | Instance actively executing |
| `suspended` | Instance paused (manual or error) |
| `completed` | Instance finished successfully |
| `failed` | Instance terminated by error |
| `terminated` | Instance cancelled by user or admin |

### Bottleneck Detection

Average wait time per step is computed across completed instances. Steps exceeding 2x the mean are flagged as bottlenecks, reportable by workflow type, tenant, and time range.

### Audit Trail

Every state transition is recorded in `process_steps` with timestamp, actor, and action. Entries are immutable (append-only), tenant-scoped, and queryable by instance ID, date range, actor, and action type.

## Email Approvals

### Mechanism

1. When a user task is created and the assignee has email notifications enabled, the Workflow Service publishes a notification request to Platform Service.
2. The email contains a unique approval token (single-use, time-limited, HMAC-signed) embedded in approve/reject action links or reply parsing.
3. The assignee clicks the link or replies with `APPROVE` or `REJECT` in the email body.
4. Platform Service's email webhook forwards the response to Workflow Service's email approval endpoint.
5. Workflow Service validates the token, applies the action, and publishes the corresponding event.
6. The token is invalidated after use.

### Constraints

Email approvals support approve/reject only (delegation requires the API). Tokens expire after 72 hours (configurable per tenant) and are bound to a specific step and instance.

## SoD Enforcement

### SoD Rule Definition

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | SoD rule unique identifier |
| `name` | string | Rule name (e.g., "Purchase requester cannot be approver") |
| `conflicting_permissions` | string[][] | Pairs of permission strings that must not be held by the same user |
| `scope` | enum | `global` (entire tenant) or `workflow_type` (specific process) |
| `severity` | enum | `warning` (allow with notification) or `block` (prevent action) |

### Conflict Detection

1. Before an approval action is recorded, the Workflow Service evaluates all active SoD rules.
2. The engine checks whether the prospective actor holds any conflicting permission pair relative to previous actors in the same instance.
3. If a conflict is detected:
   - `severity: block` — the action is rejected with `WORKFLOW_SOD_CONFLICT` error.
   - `severity: warning` — the action is allowed, but `workflow.sod.conflict.flagged` event is published.
4. All SoD evaluations are logged in the process audit trail.

## Database Tables

All tables include standard columns: `id` (UUID PK), `tenant_id` (UUID), `created_at` (timestamptz), `updated_at` (timestamptz), `created_by` (UUID), `updated_by` (UUID), `version` (INT), `is_deleted` (BOOLEAN).

| Table | Purpose | Key Columns |
|-------|---------|-------------|
| `process_definitions` | BPMN process definitions (versioned) | `key` (unique per tenant + key), `name`, `bpmn_xml`, `version`, `status` (draft/active/archived), `category` |
| `process_instances` | Running and completed workflow instances | `definition_id` (FK), `business_key`, `state`, `variables` (JSONB), `started_at`, `completed_at`, `sla_deadline` |
| `process_steps` | Individual steps within an instance (audit trail) | `instance_id` (FK), `step_key`, `step_type`, `assignee_id`, `state`, `action`, `variables` (JSONB), `acted_at`, `duration_ms` |
| `approval_chains` | Approval chain configurations | `name`, `chain_type` (sequential/parallel), `levels` (JSONB — array of level definitions), `quorum`, `allow_redelegate` |
| `sla_definitions` | SLA policies per workflow type | `workflow_type`, `response_time_minutes`, `resolution_time_minutes`, `business_calendar_id`, `priority`, `notification_channels` (text[]) |
| `business_rules` | Routing and conditional rules | `name`, `workflow_type`, `step_key`, `condition` (FEEL), `action` (JSONB), `priority`, `status` (draft/active/archived) |
| `rule_versions` | Immutable rule version history | `rule_id` (FK), `version`, `condition`, `action`, `status` |
| `escalation_rules` | Escalation policies for steps | `workflow_type`, `step_key`, `trigger_type`, `duration_minutes`, `action`, `reassign_to`, `notification_template_id`, `max_retries` |
| `email_approvals` | Email approval tokens and state | `instance_id` (FK), `step_id` (FK), `token_hash`, `assignee_email`, `expires_at`, `used_at`, `action` |
| `sod_rules` | Segregation of Duties rules | `name`, `conflicting_permissions` (JSONB), `scope`, `severity` (warning/block) |
| `business_calendars` | Business hours and holiday calendars | `name`, `timezone`, `working_days` (int[]), `working_hours_start`, `working_hours_end`, `holidays` (JSONB) |
| `delegation_schedules` | Scheduled proxy/delegation assignments | `delegator_id`, `delegate_id`, `starts_at`, `ends_at`, `chain_id` (nullable — null means all chains) |

## Events Published

| Event | Payload | Description |
|-------|---------|-------------|
| `workflow.instance.started` | `{ instance_id, definition_id, tenant_id, business_key, started_by }` | Workflow instance started |
| `workflow.instance.completed` | `{ instance_id, definition_id, tenant_id, business_key, completed_at, duration_ms }` | Workflow instance completed successfully |
| `workflow.instance.failed` | `{ instance_id, definition_id, tenant_id, business_key, error_code, error_message }` | Workflow instance terminated by error |
| `workflow.step.approved` | `{ instance_id, step_id, step_key, tenant_id, approver_id, action, acted_at }` | Workflow step approved |
| `workflow.step.rejected` | `{ instance_id, step_id, step_key, tenant_id, rejector_id, reason, acted_at }` | Workflow step rejected |
| `workflow.step.escalated` | `{ instance_id, step_id, step_key, tenant_id, escalation_action, escalated_to, triggered_by }` | Workflow step escalated (timeout or threshold) |
| `workflow.step.delegated` | `{ instance_id, step_id, step_key, tenant_id, from_user_id, to_user_id, reason }` | Workflow step delegated to another user |
| `workflow.sod.conflict.flagged` | `{ instance_id, step_id, tenant_id, sod_rule_id, conflict_type, severity, actor_id }` | SoD conflict flagged during approval workflow |

> **Note:** Workflow events are consumed by the originating service (e.g., Finance, Commerce, HCM) to update business document status. Platform Service consumes all workflow events for audit logging and notification delivery.

## Events Consumed

| Event | Source Service | Action |
|-------|---------------|--------|
| `hr.leave.#` | HCM Service | Trigger leave approval template |
| `commerce.order.#` | Commerce Service | Trigger contract approval or order approval template |
| `finance.purchase-order.#` | Finance Service | Trigger purchase requisition approval template |
| `finance.expense.#` | Finance Service | Trigger expense approval template |
| `manufacturing.eco.#` | Manufacturing Service | Trigger change order approval template |
| `config.changed` | Config Service | Reload workflow configuration (SLA thresholds, escalation policies) |

## See Also

- [Auth Service](auth.md)
- [Platform Service](platform.md)
- [Finance Service](finance.md)
- [HCM Service](hr.md)
- [Commerce Service](commerce.md)
- [Manufacturing Service](manufacturing.md)
- [Architecture Overview](../01-architecture/overview.md)
- [Event Catalog](../04-events/catalog.md)
- [API Endpoints](../05-api/endpoints.md)
