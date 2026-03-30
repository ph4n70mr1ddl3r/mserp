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

> Owned and operated by HCM Service. Platform Service provides the notification and file storage infrastructure that the portal uses. The self-service API endpoints are defined in the HCM Service spec and listed under HCM in `docs/05-api/endpoints.md`.

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

## Application Composer

| Module | Description |
|--------|-------------|
| Custom Fields | Custom field definitions with data types, validation rules, visibility settings, and field-level permissions |
| Custom Objects | Custom business object creation with configurable schemas, relationships, and lifecycle states |
| Serverless Scripts | Serverless script execution for custom business logic with sandboxed runtime and event triggers |
| Page Composer | Visual page composer for building custom UIs with drag-and-drop widgets, layouts, and data bindings |

## Access Certification

| Module | Description |
|--------|-------------|
| Campaign Management | Certification campaign creation with scope definition, scheduling, and multi-level reviewer assignment |
| Reviewer Workflows | Reviewer workflows with attestation decisions, bulk actions, and escalation for overdue reviews |
| Auto-Revoke | Automated revocation of uncertified access with configurable grace periods and exception handling |
| Role Mining | ML-based role mining to identify optimal role definitions from access patterns and usage data |

## IoT Device Registry

Platform Service owns the **global IoT device registry** (device identity, certificates, lifecycle). Manufacturing Service owns **industrial IoT telemetry** (MQTT/OPC-UA ingestion, alert rules, edge processing). The boundary is:
- Device registration and certificate issuance: Platform Service (`platform.iot.device.*` events)
- Telemetry ingestion and processing: Manufacturing Service (`manufacturing.iot.*` events)
- Manufacturing's `iot_devices` table is a local cache of device metadata for industrial devices, not the authoritative registry.

## Database Tables

All tables include standard columns: `id UUID PK`, `tenant_id UUID`, `created_at TIMESTAMPTZ`, `updated_at TIMESTAMPTZ`, `created_by UUID`, `updated_by UUID`, `version INT`, `is_deleted BOOLEAN`.

### platform_db

| Table | Additional Columns |
|-------|-------------------|
| `notifications` | `user_id UUID`, `type VARCHAR(20)`, `channel VARCHAR(20)`, `subject VARCHAR(255)`, `body TEXT`, `status VARCHAR(20)`, `sent_at TIMESTAMPTZ`, `read_at TIMESTAMPTZ`, `reference_type VARCHAR(50)`, `reference_id UUID`, `metadata JSONB` |
| `files` | `filename VARCHAR(255)`, `content_type VARCHAR(100)`, `size_bytes BIGINT`, `storage_key VARCHAR(500)`, `checksum VARCHAR(64)`, `folder_id UUID`, `tags TEXT[]`, `access_control JSONB`, `is_public BOOLEAN` |
| `grc_sod_rules` | `name VARCHAR(255)`, `description TEXT`, `permission_a VARCHAR(100)`, `permission_b VARCHAR(100)`, `severity VARCHAR(10)`, `is_active BOOLEAN` |
| `grc_risk_register` | `title VARCHAR(255)`, `category VARCHAR(50)`, `likelihood VARCHAR(10)`, `impact VARCHAR(10)`, `risk_score INT`, `mitigation_plan TEXT`, `owner_id UUID`, `status VARCHAR(20)`, `review_date DATE` |
| `scheduler_jobs` | `name VARCHAR(255)`, `cron_expression VARCHAR(100)`, `service VARCHAR(50)`, `endpoint VARCHAR(255)`, `payload JSONB`, `status VARCHAR(20)`, `last_run_at TIMESTAMPTZ`, `next_run_at TIMESTAMPTZ`, `max_retries INT`, `timeout_ms INT` |
| `knowledge_articles` | `title VARCHAR(255)`, `content TEXT`, `category_id UUID`, `status VARCHAR(20)`, `tags TEXT[]`, `views INT`, `rating DECIMAL`, `published_at TIMESTAMPTZ` |
| `rpa_bots` | `name VARCHAR(255)`, `description TEXT`, `definition JSONB`, `trigger_type VARCHAR(20)`, `schedule VARCHAR(100)`, `status VARCHAR(20)`, `last_execution_id UUID`, `owner_id UUID` |
| `collaboration_channels` | `name VARCHAR(255)`, `type VARCHAR(20)`, `description TEXT`, `context_type VARCHAR(50)`, `context_id UUID`, `members JSONB`, `is_private BOOLEAN` |
| `idp_document_types` | `name VARCHAR(100)`, `description TEXT`, `extraction_schema JSONB`, `confidence_threshold DECIMAL`, `model_id UUID`, `is_active BOOLEAN` |
| `itsm_incidents` | `incident_number VARCHAR(50)`, `priority VARCHAR(10)`, `category VARCHAR(30)`, `status VARCHAR(20)`, `requester_id UUID`, `assignee_id UUID`, `sla_due_at TIMESTAMPTZ`, `resolution TEXT`, `resolved_at TIMESTAMPTZ` |
| `content_repositories` | `name VARCHAR(255)`, `description TEXT`, `schema_definition JSONB`, `retention_policy JSONB`, `storage_quota_bytes BIGINT`, `is_compliance_archive BOOLEAN`, `status VARCHAR(20)` |
| `iot_devices` | `device_id VARCHAR(100) UNIQUE`, `name VARCHAR(255)`, `type VARCHAR(30)`, `certificate_id UUID`, `certificate_pem TEXT`, `firmware_version VARCHAR(50)`, `status VARCHAR(20)`, `metadata JSONB`, `last_heartbeat_at TIMESTAMPTZ`, `provisioned_at TIMESTAMPTZ` |

| `privacy_consents` | Consent records per processing purpose with versioned consent |
| `privacy_dsar_requests` | Data subject access request tracking with fulfillment workflow |
| `privacy_dpia_records` | Data protection impact assessment records with risk scoring |
| `privacy_breach_records` | Breach detection, assessment, and notification records |
| `dlp_policies` | DLP policy definitions with conditions, actions, and scope |
| `dlp_incidents` | DLP incident records with detection details and resolution |
| `compliance_dashboard_metrics` | Composite compliance health scores per domain |
| `compliance_regulatory_changes` | Regulatory change records with impact assessment |
| `compliance_control_mappings` | Cross-framework control mapping entries |
| `compliance_calendar_events` | Compliance calendar entries with deadlines |
| `assistant_conversations` | Digital assistant conversation persistence with context |
| `assistant_intents` | Intent classification definitions with training data |
| `app_builder_apps` | Low-code application definitions with schema and UI config |
| `app_builder_objects` | Custom business object definitions with field schema |
| `collaboration_messages` | Channel messages with thread structure and reactions |
| `collaboration_tasks` | Collaboration tasks with assignments and due dates |
| `content_documents` | Document metadata with lifecycle state and retention linkage |
| `content_retention_policies` | Retention policy definitions per document class |
| `digital_signature_requests` | Digital signature request tracking with status |
| `idp_extractions` | Document extraction results with per-field confidence scores |

> **Note:** The `iot_devices` table in `platform_db` is the **authoritative** IoT device registry. Platform publishes `platform.iot.device.registered`, `platform.iot.device.certificate.issued`, and `platform.iot.device.decommissioned` events. Manufacturing Service consumes these events to sync its local `iot_devices` cache table.

### audit_db

| Table | Additional Columns |
|-------|-------------------|
| `audit_events` | `actor_id UUID`, `action VARCHAR(50)`, `entity_type VARCHAR(50)`, `entity_id UUID`, `changes JSONB`, `ip_address INET`, `user_agent TEXT`, `severity VARCHAR(10)` |

## Events Published

| Event | Description |
|-------|-------------|
| `platform.notification.sent` | Notification dispatched |
| `platform.notification.failed` | Notification delivery failed |
| `platform.file.uploaded` | File uploaded and stored |
| `platform.file.deleted` | File soft-deleted |
| `platform.audit.logged` | Audit log entry created |
| `platform.digital-assistant.intent.resolved` | Digital assistant resolved user intent |
| `platform.digital-assistant.action.executed` | Digital assistant executed an action |
| `platform.digital-assistant.feedback.recorded` | User feedback on assistant response recorded |
| `platform.app-builder.app.published` | Low-code application published |
| `platform.app-builder.app.updated` | Low-code application updated |
| `platform.grc.compliance.violation.detected` | Compliance violation detected |
| `platform.grc.risk.assessment.completed` | Risk assessment completed |
| `platform.grc.sod.conflict.detected` | Segregation of Duties conflict detected |
| `platform.grc.incident.created` | GRC incident created |
| `platform.data-mask.applied` | Data masking rule applied to non-production environment |
| `platform.scheduler.job.started` | Scheduled job started execution |
| `platform.scheduler.job.completed` | Scheduled job completed |
| `platform.scheduler.job.failed` | Scheduled job failed |
| `platform.knowledge.article.created` | Knowledge base article created |
| `platform.knowledge.article.published` | Knowledge base article published |
| `platform.knowledge.article.updated` | Knowledge base article updated |
| `platform.signature.requested` | Digital signature requested |
| `platform.signature.completed` | Digital signature completed |
| `platform.rpa.bot.created` | RPA bot created |
| `platform.rpa.bot.executed` | RPA bot execution completed |
| `platform.rpa.bot.failed` | RPA bot execution failed |
| `platform.collaboration.message.posted` | Collaboration message posted |
| `platform.collaboration.task.created` | Collaboration task created |
| `platform.collaboration.task.completed` | Collaboration task completed |
| `platform.iot.device.registered` | IoT device registered in device registry |
| `platform.iot.device.certificate.issued` | IoT device certificate issued |
| `platform.iot.device.decommissioned` | IoT device decommissioned |
| `platform.idp.document.classified` | Document classified by intelligent document processing |
| `platform.idp.extraction.completed` | Data extraction from document completed |
| `platform.idp.extraction.failed` | Data extraction from document failed |
| `platform.idp.model.trained` | IDP extraction model training completed |
| `platform.itsm.incident.created` | IT service incident created |
| `platform.itsm.change.approved` | IT change request approved |
| `platform.compliance.alert.triggered` | Compliance hub alert triggered |
| `platform.ipa.cognitive-bot.executed` | Intelligent process automation bot completed cognitive task |
| `platform.ipa.process.discovered` | New automation opportunity discovered from usage analytics |
| `platform.ipa.bot.self-learned` | Bot updated its model from feedback |

## Events Consumed

Inbox binding: `platform.inbox` binds to the following routing keys:

| Binding Pattern | Events Consumed |
|----------------|-----------------|
| `auth.login.#` | `auth.login.succeeded`, `auth.login.failed` |
| `hr.#` | `hr.employee.created`, `hr.employee.hired`, `hr.employee.updated`, `hr.employee.separated`, `hr.talent-review.initiated` |
| `commerce.order.#` | `commerce.order.created`, `commerce.order.submitted`, `commerce.order.fulfilled`, `commerce.order.cancelled` |
| `commerce.credit.#` | `commerce.credit.hold.applied`, `commerce.credit.hold.released` |
| `commerce.shipment.#` | `commerce.shipment.dispatched`, `commerce.shipment.in-transit`, `commerce.shipment.delivered` |
| `commerce.logistics.#` | `commerce.logistics.tracking.updated`, `commerce.logistics.condition.alert`, `commerce.logistics.exception.detected` |
| `finance.lease.#` | `finance.lease.created`, `finance.lease.modified` |
| `finance.supplier-risk.#` | `finance.supplier-risk.score.updated`, `finance.supplier-risk.alert.triggered`, `finance.supplier-risk.mitigation.created` |
| `finance.intelligent-close.#` | `finance.intelligent-close.task.auto-assigned`, `finance.intelligent-close.anomaly.detected`, `finance.intelligent-close.auto-reconciled` |
| `integration.trade-compliance.#` | `integration.trade-compliance.screening.completed`, `integration.trade-compliance.screening.flagged`, `integration.trade-compliance.license.expiring` |
| `crm.cdp.#` | `crm.cdp.profile.created`, `crm.cdp.profile.updated`, `crm.cdp.profile.merged`, `crm.cdp.segment.updated` |
| `report.process.#` | `report.process.discovered`, `report.process.conformance.checked`, `report.process.bottleneck.detected`, `report.process.simulation.completed` |
| `manufacturing.intelligence.#` | `manufacturing.intelligence.oee.threshold-breached`, `manufacturing.intelligence.downtime.categorized`, `manufacturing.intelligence.energy.anomaly`, `manufacturing.intelligence.predictive-maintenance.alert` |
| `tenant.feature.#` | `tenant.feature.changed` |
| `config.changed` | `config.changed` |

## See Also

- [Integration Service](integration.md)
- [Workflow Service](workflow.md)
- [Tenant Service](tenant.md)
- [Architecture Overview](../01-architecture/overview.md)
