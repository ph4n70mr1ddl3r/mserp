# Platform Service

| Aspect | Details |
|--------|---------|
| Port | 8020 |
| Database | `platform_db` (notifications + file metadata), `audit_db` (time-series optimized) |
| Responsibilities | Multi-channel notifications, file storage, document management, audit logging, digital assistant, low-code application builder, GRC, content management, privacy management, DLP, ITSM, compliance hub |
| Rationale | See [Architecture — Service Consolidation](../01-architecture/overview.md#4-service-consolidation-rationale) |

## Modules

| Module | Description |
|--------|-------------|
| Email Notifications | SMTP integration, templated emails, delivery tracking, bounce handling |
| SMS Notifications | SMS gateway integration (Twilio, etc.), delivery receipts |
| Push Notifications | Mobile push (FCM, APNs), web push |
| In-App Notifications | Real-time notifications via WebSocket, read/unread status, grouping |
| Webhooks | Outbound webhook delivery with retry, HMAC signing, event filtering |
| File Upload/Download | Chunked uploads, virus scanning, image transformation, thumbnail generation |
| Document Metadata | File metadata, tags, versioning, access control |
| Document Preview | In-browser preview for PDF, images, Office documents (via MinIO + LibreOffice conversion) |
| OCR | Optical character recognition for scanned documents, invoice capture |
| Immutable Audit Logs | Append-only audit trail, query by entity/user/action/time range |
| Compliance Reports | SOX audit trail, GDPR data access logs, HIPAA access logs |
| Data Lineage | Track data flow across services, lineage graphs for regulatory reporting |
| Digital Assistant | NLP-based conversational interface, intent recognition, entity extraction, multi-turn dialog, context management, action dispatch to service APIs |
| Application Builder | Visual page designer, custom business objects, drag-and-drop form builder, workflow integration, tenant-scoped publishing with versioning |
| GRC Controls | Segregation of Duties (SoD) rules, compliance policy management, risk register, control assessments, incident management |
| Data Masking | Dynamic and static data masking rules, PII subsetting for non-production environments, masking templates by compliance framework |
| Enterprise Job Scheduler | Cron-based and event-triggered job scheduling, DAG-based job dependencies, retry policies, per-tenant concurrency limits, job execution history and monitoring |
| Knowledge Management | Article authoring with rich text and version control, hierarchical categories, tagging, full-text search, role-based access, context-sensitive knowledge suggestions, article analytics (views, ratings) |
| Digital Signatures | Native digital signature workflows, signature request and tracking, multi-party signing sequences, signature audit trail, integration with e-signature providers (DocuSign, Adobe Sign) for external documents |

> Platform provides notification and file storage infrastructure used by HCM's Employee Self-Service Portal.

## Subdomains

| Subdomain | Modules |
|-----------|---------|
| Communications | Email Notifications, SMS Notifications, Push Notifications, In-App Notifications, Webhooks, Enterprise Collaboration |
| Content | File Upload/Download, Document Metadata, Document Preview, OCR, Enterprise Content Management, Knowledge Management, Digital Signatures |
| Automation (RPA/IPA) | RPA modules, IPA modules |
| Intelligence | Digital Assistant, Intelligent Document Processing (IDP) |
| Platform Services | Application Builder, GRC Controls, ITSM, Compliance Hub, Privacy Management, DLP, Data Masking, Enterprise Job Scheduler, Data Lineage |

## Robotic Process Automation (RPA)

| Module | Description |
|--------|-------------|
| Bot Designer | Visual bot designer with drag-and-drop action sequences and conditional logic |
| Bot Execution | Schedule-based, event-triggered, and manual bot execution with dependency management |
| Action Library | Pre-built actions for data entry, report generation, notifications, API calls, and file operations |
| Credential Vaulting | Secure credential management for bot accounts via HashiCorp Vault integration |
| Bot Monitoring | Execution history, success rates, duration tracking, and error alerting |
| Error Handling | Configurable retry policies, fallback actions, and manual intervention queues |

## Enterprise Collaboration

| Module | Description |
|--------|-------------|
| Team Messaging | Channel-based messaging with business context integration (order, project, case context) |
| Document Collaboration | Real-time document co-editing with version control, commenting, and review workflows |
| Task Management | Personal and team task boards with integration to projects and workflow tasks |
| Presence & Availability | User availability status with calendar integration and notification preferences |
| Unified Search | Search across messages, documents, tasks, and knowledge base articles |

## Enterprise Content Management

| Module | Description |
|--------|-------------|
| Content Repository | Hierarchical content storage with configurable metadata schemas |
| Document Lifecycle | Draft → Review → Approved → Published → Archived lifecycle with configurable workflows |
| Records Management | Retention policies, legal hold, disposition schedules, records classification |
| Content Search | Full-text search with faceted navigation across all content types |
| Compliance Archive | Immutable archive for regulatory-compliant document retention |

## Privacy Management

| Module | Description |
|--------|-------------|
| Consent Management | Granular consent capture, tracking, and enforcement per processing purpose |
| Data Subject Rights | Automated handling of access, rectification, erasure, portability, and objection requests |
| DPIA Workflows | Automated Data Protection Impact Assessment workflows with risk scoring |
| Processing Register | Automated register of data processing activities (GDPR Article 30) |
| Data Flow Mapping | Visual mapping of personal data flows across services and third parties |

## Data Loss Prevention (DLP)

| Module | Description |
|--------|-------------|
| Data Classification | Automated sensitive data classification (PII, PHI, financial, IP) |
| Policy Engine | Configurable DLP policies with detection, blocking, and alerting rules |
| Access Monitoring | Monitoring of data access patterns for anomalous exfiltration behavior |
| DLP Incidents | Automated DLP incident creation, investigation workflows, remediation tracking |
| Compliance Reporting | DLP compliance dashboards, incident metrics, policy effectiveness analysis |

## IT Service Management (ITSM)

| Module | Description |
|--------|-------------|
| Incident Management | IT incident logging, categorization, prioritization, assignment, and resolution tracking with SLA |
| Service Request Catalog | Self-service catalog for IT service requests with approval workflows |
| Problem Management | Root cause analysis, known error database, and proactive problem prevention workflows |
| Change Management | Change request workflows with risk assessment, approval matrices, and change calendars |
| Configuration Management | IT asset configuration item registry with relationship mapping and dependency tracking |
| CMDB Integration | Configuration Management Database with auto-discovery for network, server, and application inventory |
| Release Management | Release planning, deployment scheduling, rollback procedures, and post-deployment verification |

## Compliance Hub

| Module | Description |
|--------|-------------|
| Unified Dashboard | Single compliance dashboard aggregating status from GRC, Trade, Privacy, Tax, ESG, and Product Compliance |
| Regulatory Change Tracker | Automated monitoring of regulatory changes with impact assessment workflows |
| Compliance Calendar | Unified compliance calendar with filing deadlines, audit schedules, and certification renewals |
| Control Mapping | Cross-framework control mapping (SOC 2, GDPR, HIPAA, PCI DSS, SOX, ISO 27001) |
| Compliance Score | Composite compliance health score per domain with trend tracking |
| Regulatory Intelligence | AI-powered regulatory content feed with relevance scoring and automated impact analysis |

## Intelligent Document Processing (IDP)

| Module | Description |
|--------|-------------|
| Document Classification | AI-powered document type classification (invoices, POs, contracts, receipts) |
| Data Extraction | Intelligent field extraction from documents with confidence scoring |
| Template Learning | Self-learning extraction models that improve from user corrections |
| Batch Processing | High-volume document batch processing with parallel extraction |
| Validation | Extracted data validation against business rules and reference data |
| Integration | Auto-create business documents (invoices, expenses) from extracted data |

## Intelligent Process Automation (IPA)

| Module | Description |
|--------|-------------|
| Cognitive Bots | AI-enhanced RPA bots with computer vision, NLP understanding, and adaptive decision-making |
| Self-Learning Bots | Bots that improve from feedback loops, learning from exception handling patterns and user corrections |
| Process Discovery | ML-based process discovery from user interaction analytics to identify automation opportunities |
| Human-in-the-Loop | Configurable human intervention points for exception handling, approval, and validation |
| Bot Orchestration | Multi-bot orchestration with parallel execution, dependency management, and shared state |
| Exception Classification | ML-based classification of bot exceptions with auto-resolution for known patterns |
| Automation ROI | ROI dashboards tracking time saved, error reduction, cost savings, and throughput improvement |
| Attended Automation | Desktop assistant bots that work alongside users, triggered by context and user actions |

## See Also

- [Integration Service](integration.md)
- [Workflow Service](workflow.md)
- [Tenant Service](tenant.md)
- [Architecture Overview](../01-architecture/overview.md)
