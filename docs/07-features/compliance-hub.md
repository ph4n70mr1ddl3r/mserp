# Compliance Hub

Unified compliance management dashboard providing a single view across all compliance domains with AI-powered regulatory intelligence, managed by the Platform Service (GRC module).

## Overview

Compliance Hub consolidates compliance monitoring, reporting, and management across all regulatory frameworks into a single unified dashboard. It features AI-powered regulatory change tracking with relevance scoring, a compliance calendar with automated reminders, cross-framework control mapping to reduce audit duplication, a composite compliance health score with trend tracking, and regulatory intelligence feeds. The hub aggregates data from all compliance domains — GRC, Trade Compliance, Product Compliance, Privacy, Tax, ESG, SOX, and security — providing a single pane of glass for compliance officers, auditors, and executives.

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Unified Dashboard Architecture** | Platform Service + Report Service | Aggregating dashboard that pulls compliance status from all domains; real-time widget-based layout with customizable views per role (CISO, DPO, CFO, Auditor, Executive) |
| **Regulatory Change Tracker** | Platform Service + Report Service (ONNX) | AI-powered monitoring of regulatory changes across jurisdictions; ML model scores relevance to tenant's industry, geography, and current compliance posture; automated impact assessment workflows |
| **Compliance Calendar** | Platform Service | Unified calendar with filing deadlines, audit schedules, certification renewals, training deadlines, and regulatory milestones; automated reminders and escalation for missed deadlines |
| **Cross-Framework Control Mapping** | Platform Service | Maps controls across frameworks (SOC 2, GDPR, HIPAA, PCI DSS, SOX, ISO 27001, NIST, COBIT) to a unified control library; single evidence artifact satisfies multiple frameworks |
| **Compliance Health Score** | Platform Service + Report Service | Composite score (0-100) calculated from domain-specific sub-scores weighted by business criticality; trend tracking over time; peer benchmarking against industry averages |
| **Regulatory Intelligence Feed** | Integration Service + Report Service (ONNX) | Automated ingestion of regulatory updates from government databases, industry bodies, and legal publishers; AI relevance scoring and summarization; configurable alert thresholds |

## Unified Dashboard Components

| Widget | Data Source | Refresh | Description |
|--------|-----------|---------|-------------|
| Compliance Health Score | All compliance domains | Real-time | Composite 0-100 score with breakdown by domain |
| Domain Status Cards | Per-domain APIs | Real-time | Status per compliance domain (pass/warning/fail) |
| Regulatory Change Feed | Integration Service | Hourly | Latest regulatory changes with relevance scores |
| Upcoming Deadlines | Compliance Calendar | Real-time | Next 30 days of compliance deadlines and actions |
| Control Coverage Map | Control Library | On change | Heat map of control coverage across frameworks |
| Incident Tracker | DLP, Privacy, Security | Real-time | Open compliance incidents with severity and status |
| Audit Readiness | All domains | Daily | Audit readiness assessment with gap analysis |
| Trend Analysis | Report Service | Daily | Compliance score trends over 3/6/12 months |

## Cross-Framework Control Mapping

```
┌──────────────────────────────────────────────────────────────────┐
│                     Unified Control Library                       │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐    │
│  │ Control: Access Logging                                   │    │
│  │                                                           │    │
│  │  SOC 2 CC6.1     ────┐                                   │    │
│  │  ISO 27001 A.12.4 ───┤                                   │    │
│  │  PCI DSS 10.2    ────┤──▶  Unified Evidence Artifact:     │    │
│  │  SOX Section 302 ────┤    Audit log config + sample logs  │    │
│  │  GDPR Art. 30    ────┘                                   │    │
│  │  HIPAA §164.312(b)                                       │    │
│  └──────────────────────────────────────────────────────────┘    │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐    │
│  │ Control: Data Encryption at Rest                          │    │
│  │                                                           │    │
│  │  SOC 2 CC6.7     ────┐                                   │    │
│  │  ISO 27001 A.10.1 ───┤──▶  Unified Evidence Artifact:     │    │
│  │  PCI DSS 3.4     ────┤    Encryption config + key mgmt    │    │
│  │  HIPAA §164.312(a)───┘    audit trail                     │    │
│  │  GDPR Art. 32                                              │    │
│  └──────────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────────┘
```

| Framework | Controls in MSERP | Key Domains |
|-----------|-------------------|-------------|
| **SOC 2 Type II** | 67 mapped controls | Security, Availability, Processing Integrity, Confidentiality, Privacy |
| **GDPR** | 43 mapped controls | Lawful basis, consent, rights, DPO, breach notification, DPIA |
| **HIPAA** | 38 mapped controls | PHI protection, access controls, audit, encryption, BAAs |
| **PCI DSS** | 52 mapped controls | Network security, cardholder data, vulnerability management, monitoring |
| **SOX** | 29 mapped controls | Financial controls, audit trail, change management, SoD |
| **ISO 27001** | 114 mapped controls (Annex A) | Information security management system |
| **NIST CSF** | 98 mapped subcategories | Identify, Protect, Detect, Respond, Recover |

## Compliance Health Score Calculation

| Component | Weight | Calculation |
|-----------|--------|-------------|
| Control Coverage | 30% | `% of mapped controls with passing evidence` |
| Policy Compliance | 20% | `% of active policies with no violations in period` |
| Incident Response | 15% | `% of incidents resolved within SLA + severity weighting` |
| Training Completion | 10% | `% of required compliance training completed on time` |
| Audit Readiness | 15% | `% of audit requirements with current evidence` |
| Regulatory Tracking | 10% | `% of tracked regulatory changes assessed and actioned` |

**Score Thresholds:** 90-100 = Excellent (green), 75-89 = Good (blue), 60-74 = Needs Attention (yellow), < 60 = Critical (red)

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| GRC Module | Platform Service | Internal API | Risk assessments, controls, access certifications, SoD |
| Privacy Management | Platform Service | Internal API | Privacy compliance metrics, consent status, DPIAs, breach status |
| DLP Module | Platform Service | Internal API | DLP policy status, incident metrics, data classification coverage |
| Trade Compliance | Integration Service | Internal API | Restricted party screening status, license compliance |
| Product Compliance | Manufacturing Service | Internal API | Product certifications, material compliance, regulatory status |
| Tax Compliance | Finance Service | Internal API | Tax filing status, nexus compliance, audit status |
| ESG / Sustainability | Report Service | Internal API | ESG metrics, emissions reporting, framework compliance |
| Security Operations | Auth Service | Internal API | Vulnerability scan results, security incident status |
| Report Service | Report Service | Internal API | Compliance reports, dashboards, trend analytics |
| Notification Service | Platform Service | Event-driven | Deadline reminders, compliance alerts, escalation notifications |
| Workflow Engine | Platform Service | Event-driven | Compliance task routing, evidence collection workflows |
| External Regulatory Feeds | Integration Service | Connector | Regulatory change ingestion from external sources |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `platform.compliance.score.updated` | `{domain, old_score, new_score, trend}` | Compliance score recalculated | Dashboard, Notification Service |
| `platform.compliance.deadline.approaching` | `{deadline_id, framework, due_date, assignee}` | Deadline within reminder window | Notification Service, Workflow |
| `platform.compliance.deadline.missed` | `{deadline_id, framework, due_date, assignee}` | Deadline passed without completion | Notification Service, Escalation |
| `platform.compliance.control.failed` | `{control_id, framework, evidence_type, reason}` | Control evidence check fails | Dashboard, Notification, Audit Trail |
| `platform.compliance.control.remediated` | `{control_id, framework, remediated_by, evidence_ref}` | Failed control remediated | Dashboard, Score Calculator |
| `platform.compliance.regulatory-change.detected` | `{change_id, jurisdiction, framework, relevance_score, summary}` | New regulatory change ingested | AI Scorer, Notification Service |
| `platform.compliance.audit.scheduled` | `{audit_id, framework, auditor, start_date, scope}` | Audit scheduled in calendar | Notification Service, All Services |
| `platform.compliance.evidence.requested` | `{request_id, control_id, framework, due_date, requestor}` | Evidence collection initiated | Workflow Engine, Notification |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `platform.dlp.policy.triggered` | Platform Service | Update DLP compliance score |
| `platform.dlp.incident.created` | Platform Service | Create compliance incident |
| `platform.privacy.breach.detected` | Platform Service | Escalate to compliance incident |
| `platform.privacy.consent.withdrawn` | Platform Service | Reassess privacy compliance score |
| `finance.period.closed` | Finance Service | Update SOX compliance score |
| `manufacturing.compliance.certification.updated` | Manufacturing Service | Create compliance calendar entry |
| `integration.trade-compliance.screening.completed` | Integration Service | Update trade compliance score |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `compliance_domain` | `id, name, framework, status, health_score, last_assessed` | Has many `compliance_control`, `compliance_deadline` |
| `compliance_control` | `id, domain_id, unified_control_ref, framework_refs[], status, evidence_refs` | Belongs to `compliance_domain`, has many `compliance_evidence` |
| `compliance_evidence` | `id, control_id, artifact_type, artifact_ref, collected_at, valid_until` | Belongs to `compliance_control` |
| `compliance_deadline` | `id, domain_id, deadline_type, due_date, status, assignee, completed_at` | Belongs to `compliance_domain` |
| `compliance_incident` | `id, domain_id, severity, status, detected_at, resolved_at, assigned_to` | Belongs to `compliance_domain` |
| `compliance_regulatory_change` | `id, source, jurisdiction, change_type, relevance_score, status, assessed_at` | May create `compliance_deadline` |
| `compliance_score_history` | `id, domain_id, score, components, recorded_at` | Belongs to `compliance_domain` |
| `compliance_audit` | `id, framework, auditor, start_date, end_date, status, findings_count` | Has many `compliance_audit_finding` |

## Cross-References

- [Privacy Management](privacy.md) — Privacy compliance and data subject rights
- [Data Loss Prevention](dlp.md) — DLP policy enforcement and incidents
- [Security Architecture](../02-security/overview.md) — Security controls and threat model
- [Architecture Overview](../01-architecture/overview.md) — System context
- [Platform Service](../06-services/platform.md) — Platform service specification
- [NFR: Compliance](../10-planning/nfr.md) — Compliance requirements and standards
