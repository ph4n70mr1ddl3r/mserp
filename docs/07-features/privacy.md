# Privacy Management

Privacy-by-design framework providing consent management, data subject rights automation, DPIA workflows, and regulatory compliance across all MSERP services, managed by the Platform Service.

## Overview

Privacy Management embeds privacy protection into every layer of the MSERP platform. It provides automated consent lifecycle management, data subject rights fulfillment (access, rectification, erasure, portability, objection), Data Protection Impact Assessment workflows, a GDPR Article 30 processing register, data flow mapping, cookie consent management, 72-hour breach notification, and privacy-by-design principles enforced through default configurations. The module integrates with GRC, DLP, and all business services to ensure continuous privacy compliance across jurisdictions.

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Consent Management Lifecycle** | Platform Service (Rust) | End-to-end consent lifecycle: capture → store → enforce → expire → renew; supports granular per-purpose consent with versioning and audit trail |
| **Data Subject Rights Automation** | Platform Service + All Services | Automated fulfillment of access (SAR), rectification, erasure (right to be forgotten), portability (machine-readable export), and objection requests with cross-service orchestration |
| **DPIA Workflows** | Platform Service + Workflow Service | Data Protection Impact Assessment workflows with risk scoring, mitigation tracking, DPO review, and regulatory filing integration |
| **Processing Register (GDPR Art. 30)** | Platform Service | Automated register of processing activities with data categories, purposes, legal bases, retention periods, recipients, and cross-border transfers |
| **Data Flow Mapping** | Platform Service | Visual mapping of personal data flows across MSERP services and third-party integrations with automated discovery from event catalog |
| **Cookie Consent** | Platform Service (Frontend SDK) | Configurable cookie consent banner with per-category consent, consent persistence, and integration with consent enforcement layer |
| **Breach Notification (72-hour)** | Platform Service + Notification Service | Automated breach detection, severity assessment, notification workflow to DPO and supervisory authorities within 72-hour GDPR window |
| **Privacy by Design Principles** | Platform Service (Rust) | Default privacy-protective settings: data minimization, purpose limitation, storage limitation, pseudonymization, and access controls enforced at platform level |

## Consent Management Lifecycle

```
┌───────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Consent     │───▶│   Consent    │───▶│   Consent    │───▶│   Consent    │
│   Capture     │    │   Storage    │    │   Enforcement│    │   Expiry     │
│               │    │              │    │              │    │              │
│ • Web form    │    │ • Versioned  │    │ • Check on   │    │ • Auto-expire│
│ • Mobile SDK  │    │ • Audit trail│    │   data access│    │ • Renewal    │
│ • API         │    │ • Purpose-   │    │ • Block if   │    │   prompt     │
│ • Import      │    │   specific   │    │   no consent │    │ • Grace      │
│               │    │ • Timestamped│    │ • Log denial │    │   period     │
└───────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
```

| Consent State | Description | Transition Trigger |
|--------------|-------------|-------------------|
| `pending` | Consent requested but not yet given | Initial capture |
| `granted` | Data subject has given consent | User acceptance |
| `withdrawn` | Data subject has withdrawn consent | User withdrawal |
| `expired` | Consent period has elapsed | Time-based expiry |
| `revoked` | System revoked consent (e.g., policy change) | Admin action |

## Data Subject Rights Automation

| Right | GDPR Article | Process | Automation Level | SLA |
|-------|-------------|---------|-----------------|-----|
| **Access** | Art. 15 | Gather all personal data across services → compile report → deliver to data subject | Fully automated | 30 days |
| **Rectification** | Art. 16 | Identify data locations → update records → propagate changes across services | Fully automated | 30 days |
| **Erasure** | Art. 17 | Identify all data locations → delete/anonymize → verify complete → confirm to data subject | Automated with verification | 30 days |
| **Portability** | Art. 20 | Export all personal data in machine-readable format (JSON/CSV) → deliver to data subject | Fully automated | 30 days |
| **Objection** | Art. 21 | Record objection → cease processing for specified purpose → verify compliance | Automated | Immediate |
| **Restriction** | Art. 18 | Flag data for restricted processing → enforce access limitations → maintain audit log | Automated | Immediate |

### Erasure Orchestration Flow

```
┌──────────────┐     ┌─────────────────────────────────────────────────┐
│ Erasure      │───▶ │  Cross-Service Discovery                        │
│ Request      │     │  Identify all services holding data for subject  │
└──────┬───────┘     └─────────────────────┬───────────────────────────┘
       │                                    │
       │              ┌─────────────────────┼───────────────────┐
       │              │                     │                   │
       ▼              ▼                     ▼                   ▼
┌────────────┐ ┌────────────┐     ┌────────────┐     ┌────────────┐
│ Finance Svc│ │ HCM Svc    │     │ CRM Svc    │     │ Platform   │
│ Anonymize  │ │ Anonymize  │     │ Anonymize  │     │ Anonymize  │
│ financial  │ │ employee   │     │ customer   │     │ docs,      │
│ PII fields │ │ records    │     │ records    │     │ messages   │
└──────┬─────┘ └──────┬─────┘     └──────┬─────┘     └──────┬─────┘
       │              │                   │                   │
       └──────────────┴───────────────────┴───────────────────┘
                              │
                              ▼
                     ┌────────────────┐     ┌────────────────┐
                     │ Verification   │────▶│ Confirmation   │
                     │ Scan all svc   │     │ to data subject│
                     │ (zero PII)     │     │ + audit record │
                     └────────────────┘     └────────────────┘
```

## DPIA Workflow

| Stage | Participants | Artifacts | Automation |
|-------|-------------|-----------|------------|
| Screening | Privacy team | DPIA screening questionnaire | Auto-score based on data types/volume |
| Assessment | Privacy team + DPO | Risk assessment, necessity/proportionality test | Risk scoring model |
| Mitigation | Privacy team + Dev | Mitigation plan, technical controls | Control suggestion from GRC library |
| Review | DPO | DPO opinion, approval/rejection | Compliance check against regulatory requirements |
| Implementation | Dev team | Implementation evidence | Automated verification of controls |
| Monitoring | Privacy team | Ongoing compliance metrics | Automated monitoring dashboards |

## Breach Notification Timeline

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│ Breach   │────▶│ Assess   │────▶│ Notify   │────▶│ Notify   │────▶│ Notify   │
│ Detected │     │ Severity │     │ DPO      │     │ Authority│     │ Subjects │
│          │     │ + Scope  │     │ (< 1hr)  │     │ (< 72hr) │     │ (if high)│
└──────────┘     └──────────┘     └──────────┘     └──────────┘     └──────────┘
```

| Milestone | Timeframe | Action | Responsible |
|-----------|-----------|--------|-------------|
| Detection | T+0 | Automated breach detection triggers workflow | DLP + Platform Service |
| Assessment | T+1h | Severity classification, scope determination, containment | Security team |
| DPO Notification | T+1h | Notify Data Protection Officer with initial assessment | Automated |
| Authority Notification | T+72h | Submit breach notification to supervisory authority (GDPR Art. 33) | DPO |
| Subject Notification | T+determined | Notify affected data subjects if high risk (GDPR Art. 34) | DPO + Comms |
| Post-Incident Review | T+7d | Root cause analysis, policy updates, preventive measures | Security + Privacy team |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| DLP Module | Platform Service | Internal API | Sensitive data detection feeds privacy assessments |
| GRC Module | Platform Service | Internal API | Risk assessments, control mapping, compliance evidence |
| Auth Service | Auth Service | Internal API | User identity, consent enforcement at access layer |
| Audit Trail | Platform Service | Event-driven | Privacy actions logged (consent changes, SARs, erasures) |
| Notification Service | Platform Service | Event-driven | Breach notifications, consent expiry reminders, SAR updates |
| Workflow Engine | Workflow Service | Event-driven | DPIA workflow, SAR fulfillment orchestration |
| All Business Services | All Services | Internal API | Data subject rights execution (erasure, rectification, export) |
| Compliance Hub | Platform Service | Internal API | Privacy compliance metrics in unified dashboard |
| Report Service | Report Service | Internal API | Privacy compliance reports, processing register reports |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `platform.privacy.consent.captured` | `{subject_id, purposes[], consent_version, captured_at}` | New consent captured | All Services, Audit Trail |
| `platform.privacy.consent.withdrawn` | `{subject_id, purposes[], withdrawn_at}` | Consent withdrawal | All Services, DLP, Notification |
| `platform.privacy.consent.expired` | `{subject_id, purpose, expired_at}` | Consent period expires | Notification Service, All Services |
| `platform.privacy.sar.created` | `{request_id, subject_id, right_type, received_at, due_date}` | Data subject request received | Workflow Engine, Notification |
| `platform.privacy.sar.completed` | `{request_id, subject_id, right_type, completed_at, delivery_method}` | SAR fulfilled | Audit Trail, Notification |
| `platform.privacy.erasure.completed` | `{subject_id, services_affected, records_erased, verified_at}` | Erasure verified complete | Audit Trail, All Services |
| `platform.privacy.dpia.created` | `{dpia_id, processing_activity, risk_level, status}` | DPIA initiated | Workflow Engine, GRC |
| `platform.privacy.dpia.approved` | `{dpia_id, approved_by, conditions[]}` | DPIA approved by DPO | GRC, Audit Trail |
| `platform.privacy.breach.detected` | `{breach_id, severity, affected_records, detected_at}` | Breach detected | Notification Service, DLP, Workflow |
| `platform.privacy.breach.notified` | `{breach_id, authority, notified_at, notification_ref}` | Authority notified | Audit Trail, Compliance Hub |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `platform.dlp.policy.triggered` | Platform Service (DLP) | Assess privacy impact, potential breach |
| `platform.dlp.incident.created` | Platform Service (DLP) | Evaluate breach notification requirement |
| `identity.user.created` | Identity Service | Capture initial consent |
| `identity.user.deactivated` | Identity Service | Trigger erasure workflow |
| `platform.audit.logged` | Platform Service | Enforce consent-based access control |
| `integration.sync.completed` | Integration Service | Check consent for cross-border transfer |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `privacy_consent` | `id, subject_id, purpose, status, version, captured_at, expires_at` | Belongs to `privacy_subject` |
| `privacy_subject` | `id, external_id_type, external_id, jurisdiction, consent_status` | Has many `privacy_consent`, `privacy_sar` |
| `privacy_sar` | `id, subject_id, right_type, status, received_at, due_date, completed_at` | Belongs to `privacy_subject`, has many `privacy_sar_step` |
| `privacy_sar_step` | `id, sar_id, service, action, status, completed_at` | Belongs to `privacy_sar` |
| `privacy_dpia` | `id, processing_activity, risk_level, status, assessor, dpo_review, approved_at` | Has many `privacy_dpia_risk` |
| `privacy_dpia_risk` | `id, dpia_id, risk_type, likelihood, impact, mitigation, residual_risk` | Belongs to `privacy_dpia` |
| `privacy_processing_register` | `id, activity_name, purpose, legal_basis, data_categories, recipients, retention, transfers` | GDPR Art. 30 register |
| `privacy_breach` | `id, severity, affected_records, detected_at, authority_notified_at, subjects_notified_at` | Has many `privacy_breach_event` |
| `privacy_breach_event` | `id, breach_id, event_type, details, created_at` | Belongs to `privacy_breach` |

## Cross-References

- [Data Loss Prevention](dlp.md) — Sensitive data detection and policy enforcement
- [Compliance Hub](compliance-hub.md) — Unified compliance dashboards
- [Security Architecture](../02-security/overview.md) — Encryption, access controls, authorization
- [Architecture Overview](../01-architecture/overview.md) — System context
- [Platform Service](../06-services/platform.md) — Platform service specification
- [NFR: Compliance](../10-planning/nfr.md) — GDPR, privacy requirements
- [NFR: Data Retention](../10-planning/nfr.md) — Retention and disposal policies
