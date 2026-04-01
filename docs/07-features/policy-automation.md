# Policy Automation

Business rule authoring, decision table management, policy simulation, and cross-domain rule enforcement, managed by the Platform Service.

## Overview

Policy Automation provides a centralized business rule management platform enabling non-technical users to author, test, and deploy business policies across all MSERP services. The module includes a natural language rule authoring interface, decision table management with version control, a policy simulation sandbox for testing rules before deployment, A/B rollout capabilities for gradual policy activation, a high-performance rule execution engine with full audit logging, and a cross-domain policy evaluation API consumed by all services. Policies are organized by domain and evaluated in real time via a low-latency API.

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Rule Authoring** | Platform Service (Rust) | Natural language rule editor with guided input, syntax validation, and auto-completion. Rules compiled to deterministic execution plans |
| **Decision Tables** | Platform Service (Rust) | Spreadsheet-style decision table editor with hit policy (first match, all matches, collect), condition columns, and action columns |
| **Version Control** | Platform Service (Rust) | Full version history for rules and decision tables, diff view, rollback capability, and release tagging |
| **Simulation Sandbox** | Platform Service (Rust) | Test policies against sample data sets with expected-vs-actual comparison, performance profiling, and coverage analysis |
| **A/B Rollout** | Platform Service (Rust) | Gradual policy deployment with configurable traffic percentage, automatic rollback on error rate spike, and promotion scheduling |
| **Execution Engine** | Platform Service (Rust) | Rete-optimized forward-chaining rule engine with fact assertion, conflict resolution, and action firing. Sub-millisecond evaluation |
| **Audit Logging** | Platform Service (Rust) | Every rule evaluation logged with input facts, matched rules, actions taken, execution time, and version snapshot |
| **Storage** | PostgreSQL + Redis | Rules, decision tables, versions, and audit logs in PostgreSQL; compiled rule cache and hot evaluation cache in Redis |

## Business Rules

| Rule | Description |
|------|-------------|
| **BR-PA-001** | Published policies are immutable. Modifications create a new version that must go through the simulation and approval process before deployment. |
| **BR-PA-002** | A/B rollout requires at least 1 hour of monitoring before full promotion. Automatic rollback triggers if error rate exceeds 2x baseline. |
| **BR-PA-003** | Every rule evaluation is logged with the policy version, input facts, matched conditions, output actions, and execution latency. |
| **BR-PA-004** | Decision tables with overlapping conditions must define an explicit hit policy (priority order or conflict resolution strategy). |
| **BR-PA-005** | Simulation results must achieve 100% coverage of all rule conditions before a policy can be promoted to production. |
| **BR-PA-006** | Cross-domain policies can read facts from any service domain but can only produce actions within the owning service boundary. |

## Data Model

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `policies` | `id, tenant_id, domain, name, description, status, priority, created_at, updated_at, created_by, updated_by, version, is_deleted` | Has many `policy_versions`, `policy_rules` |
| `policy_versions` | `id, tenant_id, policy_id, version_number, definition{}, change_summary, status, published_at, published_by, created_at, updated_at, version, is_deleted` | Belongs to policy |
| `policy_rules` | `id, tenant_id, policy_id, version_id, rule_name, conditions{}, actions{}, priority, is_active, created_at, updated_at, version, is_deleted` | Belongs to policy, version |
| `decision_tables` | `id, tenant_id, domain, name, hit_policy, columns{}, rows{}, version_number, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | Standalone or referenced by rules |
| `policy_simulations` | `id, tenant_id, policy_id, version_id, test_cases{}, results{}, coverage_pct, passed, created_at, updated_at, created_by, version, is_deleted` | Belongs to policy, version |
| `policy_rollouts` | `id, tenant_id, policy_id, version_id, rollout_pct, status, started_at, completed_at, error_rate, created_at, updated_at, version, is_deleted` | Belongs to policy, version |
| `policy_audit_logs` | `id, tenant_id, policy_id, version_id, facts{}, matched_rules[], actions_taken{}, execution_time_ms, evaluated_at` | Belongs to policy, version |

## Events

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `platform.policy.published` | `{policy_id, version, domain, rule_count}` | Policy version promoted to production | All subscribing services |
| `platform.policy.simulation-completed` | `{simulation_id, policy_id, passed, coverage_pct}` | Simulation run finishes | Notification, Dashboard |
| `platform.policy.rollout-started` | `{rollout_id, policy_id, version, rollout_pct}` | A/B rollout begins | Dashboard |
| `platform.policy.rollout-completed` | `{rollout_id, policy_id, version, final_pct, error_rate}` | Rollout fully promoted | Dashboard, Notification |
| `platform.policy.rollback-triggered` | `{rollout_id, policy_id, reason, error_rate}` | Auto-rollback activated | Notification, Dashboard |
| `platform.policy.evaluated` | `{policy_id, version, domain, matched_count, execution_time_ms}` | Rule evaluation completed (sampled) | Report Service |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `config.changed` | Config Service | Update execution engine parameters |
| `workflow.step.approved` | Workflow Service | Finalize policy publication approval |
| `{domain}.**` | Any service | Ingest domain facts for policy evaluation (via API) |

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/v1/policies` | Create a new policy |
| GET | `/api/v1/policies` | List policies with domain filter |
| GET | `/api/v1/policies/{id}` | Retrieve policy with current version |
| POST | `/api/v1/policies/{id}/versions` | Create a new policy version |
| POST | `/api/v1/policies/{id}/simulate` | Run simulation against test cases |
| POST | `/api/v1/policies/{id}/publish` | Publish a version to production |
| POST | `/api/v1/policies/{id}/rollout` | Start A/B rollout |
| GET | `/api/v1/decision-tables` | List decision tables |
| POST | `/api/v1/decision-tables` | Create or update a decision table |
| POST | `/api/v1/policies/evaluate` | Evaluate policies for given facts (cross-domain API) |
| GET | `/api/v1/policies/{id}/audit` | Retrieve evaluation audit logs |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| All Services | Any | REST API (evaluate) | Policy evaluation API consumed by all services for cross-domain rules |
| Approval Workflows | Workflow Service | Event-driven | Policy publication and rollout approval routing |
| Configuration | Config Service | Event-driven | Engine parameters, cache TTL, sampling rate |
| Notification | Platform Service | Event-driven | Rollout status, simulation results, rollback alerts |
| Report Service | Report Service | Internal API | Policy evaluation analytics, coverage reports |
| Audit Trail | Platform Service | Event-driven | Full audit log of all policy evaluations |

## Cross-References

- [Platform Service](../06-services/platform.md) — Platform service specification
- [Workflow Service](../06-services/workflow.md) — Approval routing for policy changes
- [Compliance Hub](compliance-hub.md) — Compliance policy integration
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
- [API Standards](../05-api/standards.md) — Policy evaluation API design
