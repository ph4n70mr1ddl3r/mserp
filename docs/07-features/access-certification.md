# Access Certification

Certification campaigns for periodic access review, auto-revoke rules, risk-based scheduling, role mining, and privileged access reviews, managed by the Platform Service (GRC module).

## Overview

Access Certification provides governance controls for ensuring that user access rights remain appropriate over time. The module supports configurable certification campaigns where managers and application owners review user access across systems, with risk-based scheduling that prioritizes high-risk access for more frequent review. It includes auto-revoke rules for unattested access, role mining capabilities to identify and consolidate redundant entitlements, and specialized workflows for privileged access reviews. The module integrates with Identity Service for role and permission data, GRC SoD rules for risk assessment, and Workflow Service for campaign orchestration.

## Data Flow Diagram

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  Identity Service │────▶│  Access Inventory │────▶│  Campaign Builder   │
│  (Roles/Perms)   │     │  Collector        │     │  ┌───────────────┐  │
└──────────────────┘     │  ┌─────────────┐  │     │  │ Risk Scoring  │  │
                         │  │ Entitlements│  │     │  │ (SoD-aware)   │  │
┌──────────────────┐     │  │ By User     │  │     │  └───────┬───────┘  │
│  GRC SoD Rules   │────▶│  │ By Role     │  │     │  ┌───────▼───────┐  │
│  (Risk Matrix)   │     │  └─────────────┘  │     │  │ Schedule      │  │
└──────────────────┘     └──────────────────┘     │  │ Generator     │  │
                                                   │  └───────────────┘  │
┌──────────────────┐     ┌──────────────────┐     └──────────┬──────────┘
│  Role Mining     │◀────│  Analytics Engine │◀──────────────┘
│  Engine          │     │                  │                │
└──────────────────┘     └──────────────────┘     ┌──────────▼──────────┐
                                                   │  Review Workflow    │
┌──────────────────┐     ┌──────────────────┐     │  ┌───────────────┐  │
│  Auto-Revoke     │◀────│  Decision Engine  │◀────│  │ Certify/     │  │
│  Executor        │     │                  │     │  │ Revoke/       │  │
└──────────────────┘     └──────────────────┘     │  │ Defer         │  │
                                                   │  └───────────────┘  │
                                                   └─────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Campaign Manager** | Platform Service (Rust) | Campaign creation with configurable scope (application, role type, risk level), reviewer assignment, and scheduling |
| **Access Inventory** | Platform Service | Real-time collection of user entitlements from Identity Service with role decomposition and relationship mapping |
| **Risk-Based Scheduling** | Platform Service + Report Service | Campaign frequency driven by access risk score, SoD violation count, and sensitivity classification |
| **Review Workflow** | Platform Service + Workflow Service | Multi-stage review with manager certification, application owner sign-off, and compliance officer approval |
| **Auto-Revoke Engine** | Platform Service | Automated revocation of unattested access after campaign deadline with grace period and exception support |
| **Role Mining** | Platform Service + Report Service | ML-based identification of role patterns, redundant entitlements, and optimization recommendations |
| **Privileged Access Reviews** | Platform Service | Specialized review workflows for admin, service account, and break-glass access with enhanced approval requirements |
| **Storage** | PostgreSQL | Campaign data, review decisions, role mining results, and audit trail in Platform database |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Campaign Manager | Campaign CRUD, scope definition, scheduling, and lifecycle | Platform Service |
| Access Inventory Collector | Entitlement aggregation from Identity Service | Platform Service |
| Risk Scorer | Access risk calculation based on SoD rules and sensitivity | Platform Service |
| Review Assignment Engine | Reviewer assignment based on organizational hierarchy and ownership | Platform Service |
| Decision Processor | Certify/revoke/defer decision processing and downstream actions | Platform Service |
| Auto-Revoke Executor | Timed revocation of unattested access | Platform Service |
| Role Mining Engine | Entitlement pattern analysis and role optimization | Platform Service |
| Campaign Analytics | Completion rates, revocation metrics, and risk trend analysis | Platform Service |

## Certification Campaign Types

| Campaign Type | Scope | Reviewer | Frequency | Example |
|--------------|-------|----------|-----------|---------|
| Application Owner | All access to specific application | Application owner | Quarterly | ERP module access review |
| Manager Review | All access for direct reports | People manager | Semi-annually | Team member access review |
| Role-Based | All users with specific role | Role owner + compliance | Annually | Admin role review |
| Risk-Based | High-risk access across systems | Risk owner | Quarterly | Financial system access |
| Privileged | Admin and service accounts | Security team + CISO | Monthly | Root/admin account review |
| SoD Violation | Users with conflicting roles | Compliance officer | On detection | Payables + vendor creation |
| New Access | Access granted since last campaign | Access sponsor | Quarterly | New entitlement review |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Identity Service | Identity Service | Internal API | User roles, permissions, and organizational hierarchy |
| GRC SoD Engine | Platform Service | Internal API | SoD rule evaluation for risk scoring |
| Workflow Engine | Workflow Service | Event-driven | Review assignment and approval workflows |
| Notification Service | Platform Service | Event-driven | Campaign launch, reminder, and deadline alerts |
| Report Service | Report Service | Internal API | Campaign analytics dashboards |
| Audit Trail | Platform Service | Event-driven | Full audit log of certification decisions |
| HR Service | HCM Service | Internal API | Organizational hierarchy for reviewer assignment |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `platform.grc.certification.campaign-created` | `{campaign_id, type, scope, reviewer_count, deadline}` | New campaign launched | Notification Service, Audit Trail |
| `platform.grc.certification.review-completed` | `{review_id, campaign_id, reviewer_id, decision, items_certified, items_revoked}` | Reviewer submits certification | Audit Trail, Identity Service, Report Service |
| `platform.grc.certification.revoked` | `{campaign_id, user_id, role_ids, revoked_by, reason}` | Access revoked by decision or auto-revoke | Identity Service, Notification Service, Audit Trail |
| `platform.grc.certification.campaign-completed` | `{campaign_id, total_reviews, certified_count, revoked_count, completion_rate}` | Campaign deadline reached and all reviews processed | Report Service, Compliance Dashboard |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `identity.role.assigned` | Identity Service | Update access inventory for upcoming campaigns |
| `identity.role.revoked` | Identity Service | Update access inventory, close open review items |
| `identity.user.deactivated` | Identity Service | Remove user from active campaigns |
| `platform.grc.sod.violation-detected` | Platform Service | Trigger SoD violation campaign |
| `hr.employee.transferred` | HCM Service | Update reviewer assignments |
| `hr.employee.terminated` | HCM Service | Trigger immediate access revocation review |
| `workflow.step.approved` | Workflow Service | Finalize certification decision |
| `workflow.step.rejected` | Workflow Service | Escalate review to next-level reviewer |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `certification_campaigns` | `id, tenant_id, name, type, scope, status, risk_threshold, start_date, deadline, completed_at` | Has many `certification_reviews` |
| `certification_reviews` | `id, tenant_id, campaign_id, reviewer_id, target_user_id, entitlements, status, submitted_at` | Has many `certification_decisions` |
| `certification_decisions` | `id, tenant_id, review_id, entitlement_id, decision, justification, decided_at, decided_by` | Belongs to `certification_reviews` |
| `role_mining_results` | `id, tenant_id, analysis_date, suggested_roles, redundancy_count, optimization_score, status` | Contains suggested role definitions |
| `privileged_access_reviews` | `id, tenant_id, account_id, account_type, current_access, risk_score, reviewer_ids, status, deadline` | Belongs to `certification_campaigns` |

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `platform.grc.certification.default_deadline_days` | 30 | Days to complete certification review |
| `platform.grc.certification.reminder_frequency_days` | 7 | Days between review reminders |
| `platform.grc.certification.auto_revoke_grace_days` | 7 | Days after deadline before auto-revoke |
| `platform.grc.certification.risk_review_frequency` | quarterly | Frequency for high-risk access campaigns |
| `platform.grc.certification.privileged_review_frequency` | monthly | Frequency for privileged access reviews |
| `platform.grc.certification.role_mining.min_sample_size` | 100 | Minimum users for role mining analysis |

## Cross-References

- [Compliance Hub](compliance-hub.md) — Compliance dashboard and policy management
- [Identity Service](../06-services/identity.md) — Role and permission source of truth
- [GRC SoD](../06-services/platform.md) — Segregation of duties rule engine
- [Workflow Engine](../06-services/workflow.md) — Review approval workflows
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
- [Platform Service](../06-services/platform.md) — Platform service specification
- [NFR: Performance](../10-planning/nfr.md) — Access inventory collection < 60s per campaign
