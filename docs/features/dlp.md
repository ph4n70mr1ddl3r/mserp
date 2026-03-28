# Data Loss Prevention (DLP)

Sensitive data detection, classification, and policy enforcement across all MSERP services, managed by the Platform Service.

## Overview

Data Loss Prevention provides comprehensive protection for sensitive data across the MSERP platform. It combines automated data classification, a configurable policy engine, real-time access monitoring, and incident response workflows to prevent unauthorized data exposure, exfiltration, and misuse. DLP integrates with the Platform Service audit and notification systems to provide real-time alerting, compliance dashboards, and forensic investigation capabilities. The system supports multiple sensitive data types including PII, PHI, financial data, and intellectual property, with configurable prevention actions ranging from alerts to blocking and encryption.

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Data Classification Taxonomy** | Platform Service (Rust) | Hierarchical classification of sensitive data types with configurable sensitivity levels: PII (names, SSN, email, phone, address), PHI (diagnosis, medications, insurance), Financial (credit cards, bank accounts, salary), IP (trade secrets, source code, formulas) |
| **Policy Engine Architecture** | Platform Service (Rust) | Rule-based policy engine with support for conditions (data type, user role, access pattern, time, location), actions (allow, alert, block, encrypt, quarantine), and policy groups with priority |
| **Access Monitoring Patterns** | Platform Service + Auth Service | Real-time monitoring of data access patterns including volume-based (bulk download), frequency-based (repeated access), time-based (off-hours access), and anomaly-based (deviation from baseline) detection |
| **DLP Incident Workflows** | Platform Service + Workflow Engine | Automated incident creation, investigation workflow, evidence collection, remediation tracking, and resolution with escalation rules |
| **Compliance Reporting Dashboards** | Report Service | Real-time DLP compliance dashboards with incident metrics, policy effectiveness, data exposure trends, and regulatory compliance status |
| **Prevention Actions** | Platform Service | Configurable actions per policy: block (prevent access/transfer), alert (notify admin/user), encrypt (apply additional encryption), quarantine (isolate data), redact (mask in output) |

## Data Classification Taxonomy

```
┌──────────────────────────────────────────────────────────────┐
│                   Data Classification Tree                    │
│                                                               │
│  ┌── PII (Personally Identifiable Information) ──────────┐   │
│  │  ├── Direct Identifiers                               │   │
│  │  │   ├── Full Name, SSN/SIN, Passport Number          │   │
│  │  │   ├── Email Address, Phone Number                   │   │
│  │  │   └── Biometric Data (fingerprint, facial)          │   │
│  │  ├── Quasi-Identifiers                                │   │
│  │  │   ├── Date of Birth, Gender, Zip/Postal Code       │   │
│  │  │   └── Job Title, Department, IP Address             │   │
│  │  └── Sensitive PII                                    │   │
│  │      ├── Financial Account Numbers                     │   │
│  │      └── Government ID Numbers                         │   │
│  └───────────────────────────────────────────────────────┘   │
│                                                               │
│  ┌── PHI (Protected Health Information) ──────────────────┐   │
│  │  ├── Diagnoses, Treatment Plans                       │   │
│  │  ├── Medications, Lab Results                         │   │
│  │  ├── Insurance Information                            │   │
│  │  └── Medical Record Numbers                           │   │
│  └───────────────────────────────────────────────────────┘   │
│                                                               │
│  ┌── Financial Data ──────────────────────────────────────┐   │
│  │  ├── Credit Card Numbers (PCI DSS scope)              │   │
│  │  ├── Bank Account Numbers                             │   │
│  │  ├── Salary and Compensation Data                     │   │
│  │  ├── Tax IDs and Returns                              │   │
│  │  └── Financial Statements (pre-earnings release)      │   │
│  └───────────────────────────────────────────────────────┘   │
│                                                               │
│  ┌── Intellectual Property ───────────────────────────────┐   │
│  │  ├── Trade Secrets and Formulas                       │   │
│  │  ├── Source Code and Algorithms                       │   │
│  │  ├── Product Roadmaps and Pricing Strategies          │   │
│  │  └── Patents and Proprietary Research                 │   │
│  └───────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────┘
```

## Policy Engine Architecture

| Policy Element | Description | Example |
|---------------|-------------|---------|
| **Condition** | Trigger criteria for policy evaluation | `data_type = SSN AND user.role != HR_ADMIN` |
| **Action** | Response when condition matches | `BLOCK access, ALERT security_team` |
| **Scope** | Where policy applies | `ALL services`, `Finance Service`, `API exports` |
| **Priority** | Evaluation order (higher = first) | `100` (evaluated before priority 50) |
| **Effective Period** | Time window for policy enforcement | `2025-01-01 to 2025-12-31` |
| **Exceptions** | Named exclusions from policy | `Service accounts, batch processes` |

### Prevention Actions

| Action | Effect | Use Case |
|--------|--------|----------|
| **BLOCK** | Prevent the data access/transfer entirely | Unauthorized SSN export |
| **ALERT** | Send notification to security team / data owner | Unusual access pattern detected |
| **ENCRYPT** | Apply additional encryption layer on the data | Sensitive report downloaded |
| **QUARANTINE** | Move data to isolated storage pending review | Potentially compromised attachment |
| **REDACT** | Mask sensitive fields in output/reports | PII in support case export |
| **WATERMARK** | Apply tracking watermark to document | Financial report download |

## Access Monitoring Patterns

| Pattern | Detection Method | Threshold | Alert Severity |
|---------|-----------------|-----------|---------------|
| Bulk Download | Volume-based | > 500 records in < 5 minutes | High |
| Abnormal Frequency | Frequency-based | > 10x user's baseline access rate | Medium |
| Off-Hours Access | Time-based | Access outside business hours + timezone deviation | Low |
| Anomalous Location | Geolocation-based | Access from unexpected geography | High |
| Privilege Escalation | Role-based | Access to data outside normal role scope | Critical |
| Data Staging | Pattern-based | Sequential access to sensitive records in systematic order | High |
| API Scraping | Rate-based | Structured sequential API calls suggesting data harvesting | High |
| Report Anomaly | Behavior-based | Unusual report generation (type, frequency, volume) | Medium |

## DLP Incident Workflow

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│ Detection│───▶│ Triage   │───▶│Investigate│───▶│ Remediate│───▶│ Resolve  │
│          │    │          │    │          │    │          │    │          │
│ Policy   │    │ Severity │    │ Evidence │    │ Action   │    │ Close +  │
│ trigger  │    │ assign   │    │ gather   │    │ taken    │    │ report   │
│          │    │ owner    │    │ impact   │    │ prevent  │    │          │
└──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘
                                                                      │
                                                                      ▼
                                                              ┌──────────────┐
                                                              │ Post-Incident│
                                                              │ Review       │
                                                              │ (policy tune)│
                                                              └──────────────┘
```

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Auth Service | Auth Service | Internal API | User role/permissions for policy evaluation |
| Audit Trail | Platform Service | Event-driven | DLP events logged to audit trail |
| Notification Service | Platform Service | Event-driven | DLP alerts and incident notifications |
| Workflow Engine | Platform Service | Event-driven | Incident investigation and remediation workflows |
| Privacy Management | Platform Service | Internal API | Privacy impact assessment integration |
| Compliance Hub | Platform Service | Internal API | DLP compliance metrics and dashboards |
| Report Service | Report Service | Internal API | DLP compliance reports and dashboards |
| All Business Services | All Services | Middleware | DLP policy evaluation on data access/transfer |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `dlp.policy.triggered` | `{policy_id, action, user_id, data_type, severity}` | DLP policy condition matches | Notification Service, Audit Trail |
| `dlp.incident.created` | `{incident_id, severity, policy_id, detected_at}` | Incident auto-created from trigger | Workflow Engine, Notification Service |
| `dlp.incident.resolved` | `{incident_id, resolution, resolved_by, actions_taken}` | Incident resolved | Audit Trail, Compliance Hub |
| `dlp.data.classified` | `{data_id, classification, sensitivity_level, classifier}` | Data classified by scanning engine | Policy Engine, Audit Trail |
| `dlp.access.blocked` | `{user_id, resource, data_type, policy_id, timestamp}` | Access blocked by policy | Notification Service, Audit Trail, Dashboard |
| `dlp.scan.completed` | `{scan_id, records_scanned, violations_found, duration}` | Scheduled scan completes | Dashboard, Compliance Hub |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `auth.user.login` | Auth Service | Update user baseline for anomaly detection |
| `auth.role.assigned` | Auth Service | Update policy scope for user |
| `platform.audit.data-access` | Platform Service | Evaluate DLP policies on access event |
| `platform.document.uploaded` | Platform Service | Scan uploaded document for sensitive data |
| `integration.data.exported` | Integration Service | Evaluate DLP policies on data export |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `dlp_policy` | `id, name, description, conditions, actions, scope, priority, is_active` | Has many `dlp_policy_condition`, `dlp_policy_action` |
| `dlp_policy_condition` | `id, policy_id, field, operator, value` | Belongs to `dlp_policy` |
| `dlp_policy_action` | `id, policy_id, action_type, parameters` | Belongs to `dlp_policy` |
| `dlp_incident` | `id, policy_id, severity, status, detected_at, resolved_at, assigned_to` | Belongs to `dlp_policy`, has many `dlp_incident_event` |
| `dlp_incident_event` | `id, incident_id, event_type, details, created_by, created_at` | Belongs to `dlp_incident` |
| `dlp_data_classification` | `id, data_source, data_id, classification, sensitivity_level, classified_at` | Reference for policy evaluation |
| `dlp_scan_result` | `id, scan_id, data_source, records_scanned, violations, duration` | Aggregated scan metrics |

## Cross-References

- [Privacy Management](privacy.md) — Consent management and data subject rights
- [Compliance Hub](compliance-hub.md) — Unified compliance dashboards
- [Security Architecture](../security/overview.md) — Encryption, mTLS, authorization
- [Architecture Overview](../architecture/overview.md) — System context
- [Platform Service](../services/platform.md) — Platform service specification
- [NFR: Compliance](../planning/nfr.md) — SOC 2, GDPR, HIPAA, PCI DSS compliance requirements
