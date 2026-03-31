# Workforce Health & Safety

Employee health and safety management with incident tracking, hazard identification, risk assessment, corrective actions, safety inspections, and regulatory compliance, managed by the HCM Service.

## Overview

Workforce Health & Safety provides a comprehensive platform for managing employee health and workplace safety across the organization. The module supports incident reporting with configurable severity classification, structured investigation workflows with root cause analysis, hazard identification and risk assessment matrices, corrective and preventive action tracking, safety inspection scheduling and execution, regulatory compliance tracking for OSHA and local regulations, ergonomic assessment management, wellness program administration, safety training management, and near-miss reporting. It integrates with Manufacturing Service for shop-floor safety events, Workflow Service for investigation and action approvals, and Config Service for regulatory rule sets.

## Data Flow Diagram

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  Incident /      │────▶│  Incident        │────▶│  Investigation      │
│  Near-Miss       │     │  Triage &        │     │  Workflow           │
│  Report          │     │  Classification  │     │  ┌───────────────┐  │
└──────────────────┘     └──────────────────┘     │  │ Root Cause    │  │
                                                   │  │ Analysis      │  │
┌──────────────────┐     ┌──────────────────┐     │  └───────┬───────┘  │
│  Hazard          │────▶│  Risk Assessment │     └──────────┼──────────┘
│  Identification  │     │  Matrix          │                │
└──────────────────┘     └──────────────────┘     ┌──────────▼──────────┐
                                                   │  Corrective &       │
┌──────────────────┐     ┌──────────────────┐     │  Preventive Actions │
│  Safety          │────▶│  Inspection      │────▶│  Tracker            │
│  Inspection      │     │  Engine          │     └──────────┬──────────┘
│  Schedule        │     └──────────────────┘                │
└──────────────────┘                                        │
                          ┌──────────────────┐     ┌────────▼──────────┐
┌──────────────────┐     │  Compliance      │◀────│  Regulatory        │
│  Wellness        │────▶│  Dashboard       │     │  Compliance Engine │
│  Programs        │     │  & Reporting     │     └──────────────────┘
└──────────────────┘     └──────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Incident Management** | HCM Service (Rust) | Configurable incident reporting with severity classification, witness tracking, and automated regulatory threshold detection |
| **Investigation Workflow** | HCM Service + Workflow Service | Structured investigation with evidence collection, root cause analysis (5-Why, fishbone), and findings documentation |
| **Risk Assessment Matrix** | HCM Service | Likelihood × severity risk matrix with configurable thresholds and automatic risk level assignment |
| **Corrective/Preventive Actions** | HCM Service | CAPA tracking with responsible parties, deadlines, verification steps, and effectiveness evaluation |
| **Safety Inspections** | HCM Service | Inspection template management, scheduling, checklist execution, finding documentation, and follow-up tracking |
| **Compliance Engine** | HCM Service | OSHA, local regulation rule sets with automated reportability determination, filing deadline tracking, and audit trail |
| **Wellness Programs** | HCM Service | Program enrollment, activity tracking, incentive management, and participation analytics |
| **Safety Training** | HCM Service | Training requirement assignment based on role and incident history, completion tracking, and certification management |
| **Storage** | PostgreSQL + MinIO | Structured safety data in PostgreSQL; incident evidence (photos, documents) in MinIO |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Incident Reporter | Incident and near-miss submission, classification, and initial triage | HCM Service |
| Investigation Manager | Investigation workflow, evidence collection, root cause analysis | HCM Service |
| Risk Assessor | Hazard risk matrix evaluation, risk level assignment, mitigation priority | HCM Service |
| CAPA Tracker | Corrective and preventive action lifecycle, deadline monitoring, verification | HCM Service |
| Inspection Engine | Template management, scheduling, checklist execution, finding capture | HCM Service |
| Compliance Engine | Regulatory rule evaluation, reportability determination, filing tracking | HCM Service |
| Wellness Program Manager | Program CRUD, enrollment, activity tracking, incentive processing | HCM Service |
| Safety Training Coordinator | Training requirement mapping, assignment, completion tracking | HCM Service |
| Safety Dashboard | Incident trends, compliance status, inspection scores, KPI visualization | HCM Service |

## Incident Severity Classification

| Severity | Level | Response Time | Regulatory Filing | Example |
|----------|-------|---------------|-------------------|---------|
| Critical | 1 | Immediate | OSHA within 8 hours | Fatality, hospitalization of 3+ |
| Severe | 2 | Within 1 hour | OSHA within 24 hours | Serious injury, hospitalization |
| Major | 3 | Within 4 hours | Internal reporting | Injury requiring medical treatment |
| Moderate | 4 | Within 24 hours | Internal tracking | First aid treatment, property damage |
| Minor | 5 | Within 72 hours | Log only | Near miss, minor discomfort |
| Near Miss | N/A | Within 72 hours | Log only | No injury but hazard present |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Employee Profile | HCM Service | Internal API | Employee data, department, role for incident context |
| Manufacturing Safety | Manufacturing Service | Event-driven | Shop-floor incident and inspection event consumption |
| Workflow Engine | Workflow Service | Event-driven | Investigation approval and CAPA review workflows |
| Notification Service | Platform Service | Event-driven | Incident alerts, inspection reminders, CAPA deadlines |
| Report Service | Report Service | Internal API | Safety analytics, compliance dashboards, trend reports |
| Config Service | Config Service | Event-driven | Regulatory rules, severity thresholds, compliance templates |
| Document Management | HCM Service + MinIO | Internal API | Evidence storage and retrieval |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `hr.safety.incident-reported` | `{incident_id, tenant_id, severity, incident_type, location, reporter_id, involved_employee_ids, occurred_at, is_reportable}` | Incident or near-miss submitted | Workflow Service, Notification Service, Report Service, Compliance Engine |
| `hr.safety.investigation-completed` | `{investigation_id, tenant_id, incident_id, root_causes (JSONB), findings, investigator_id, completed_at}` | Investigation workflow finalized | CAPA Tracker, Notification Service, Report Service |
| `hr.safety.corrective-action.created` | `{action_id, tenant_id, incident_id, investigation_id, action_type, description, responsible_id, due_date, priority}` | Corrective or preventive action defined | Notification Service, Workflow Service |
| `hr.safety.inspection.completed` | `{inspection_id, tenant_id, location_id, inspector_id, findings_count, critical_findings, score, completed_at}` | Safety inspection checklist finalized | Report Service, Notification Service, Compliance Engine |
| `hr.safety.hazard.identified` | `{hazard_id, tenant_id, location_id, hazard_type, risk_level, identified_by, mitigation_status}` | New hazard logged during inspection or observation | Risk Assessor, Notification Service, CAPA Tracker |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `hr.employee.updated` | HCM Service | Update incident and training records for role/department changes |
| `manufacturing.safety.incident.created` | Manufacturing Service | Create corresponding HR safety incident record and trigger triage |
| `manufacturing.safety.inspection.completed` | Manufacturing Service | Sync inspection results into HR safety records and compliance tracking |
| `workflow.step.approved` | Workflow Service | Advance investigation or CAPA workflow to next stage |
| `config.changed` | Config Service | Reload regulatory rules, severity thresholds, compliance templates |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `safety_incidents` | `id, tenant_id, incident_type, severity, title, description, location, occurred_at, reporter_id, involved_employee_ids (JSONB), is_reportable, regulatory_filing_status, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | Has one `safety_investigations`; has many `corrective_actions` |
| `safety_investigations` | `id, tenant_id, incident_id, investigator_ids (JSONB), start_date, end_date, root_causes (JSONB), findings (JSONB), methodology, evidence_refs (JSONB), status, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `safety_incidents` |
| `hazard_assessments` | `id, tenant_id, location_id, hazard_type, description, likelihood, severity_score, risk_level, mitigation_measures (JSONB), mitigation_status, identified_by, review_date, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | Has many `corrective_actions` |
| `corrective_actions` | `id, tenant_id, incident_id, investigation_id, hazard_id, action_type, description, responsible_id, due_date, priority, verification_method, effectiveness_rating, completed_at, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `safety_incidents`, `safety_investigations`, `hazard_assessments` |
| `safety_inspections` | `id, tenant_id, template_id, location_id, inspector_id, scheduled_date, completed_date, checklist_results (JSONB), findings_count, critical_findings, score, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | References inspection template and location |
| `wellness_programs` | `id, tenant_id, name, description, program_type, start_date, end_date, enrollment_criteria (JSONB), incentive_rules (JSONB), participant_count, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | Has many participant records |
| `safety_training_records` | `id, tenant_id, employee_id, training_type, course_name, completion_date, expiry_date, certification_ref, provider, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | References `employee` |

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `hr.safety.severity_threshold_reportable` | 2 | Minimum severity requiring regulatory filing |
| `hr.safety.investigation_timeout_days` | 30 | Maximum days to complete investigation before escalation |
| `hr.safety.capa.overdue_alert_hours` | 48 | Hours before CAPA due date to send reminder |
| `hr.safety.inspection.default_cadence_days` | 90 | Default interval between routine safety inspections |
| `hr.safety.near_miss.encouragement_mode` | true | Enable anonymous near-miss reporting |
| `hr.safety.regulatory.jurisdiction` | `OSHA` | Default regulatory framework for compliance tracking |
| `hr.safety.training.auto_assign` | true | Auto-assign safety training based on role and incident history |

## Cross-References

- [Manufacturing Service](../06-services/manufacturing.md) — Shop-floor safety event source
- [HCM Service](../06-services/hcm.md) — HCM service specification
- [Workflow Engine](../06-services/workflow.md) — Investigation and CAPA approval workflows
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
- [NFR: Performance](../10-planning/nfr.md) — Incident creation < 1s; compliance report generation < 10s
