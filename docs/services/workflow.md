# Workflow Service

| Aspect | Details |
|--------|---------|
| Port | 8015 |
| Database | `workflow_db` |
| Responsibilities | Business process automation, approvals, BPM engine, SLA management |

## Modules

| Module | Description |
|--------|-------------|
| Workflow Engine | BPMN 2.0-compatible engine, parallel/sequential gateways, subprocesses, timers |
| Approval Chains | Multi-level approval, sequential/parallel approvers, proxy/delegate approvers |
| Escalations | Time-based escalation, skip-level escalation, auto-approve on timeout |
| Process Templates | Pre-built templates (purchase approval, leave approval, expense approval, pricing approval, ECO approval, contract approval) |
| SLA Management | SLA definitions per workflow type, deadline tracking, breach notifications |
| Business Rules | Rule engine for conditional routing (e.g., amount-based, department-based) |
| Process Monitoring | Real-time process instance tracking, bottleneck analysis, audit trail |
| Email Approvals | Approve/reject via email reply (for simple single-step approvals) |
| Segregation of Duties (SoD) | SoD rule definitions, conflict detection before workflow approval, SoD violation reporting |

## See Also

- [Platform Service](platform.md)
- [HR Service](hr.md)
- [Finance Service](finance.md)
- [Architecture Overview](../architecture/overview.md)
