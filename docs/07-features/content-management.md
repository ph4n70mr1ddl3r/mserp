# Enterprise Content Management

Enterprise content repository with document lifecycle management, records management, and compliance archive, managed by the Platform Service.

## Overview

Enterprise Content Management provides a comprehensive system for creating, organizing, storing, and managing business content throughout its lifecycle. It features a hierarchical content repository supporting all media types, a configurable document lifecycle state machine (Draft → Review → Approved → Published → Archived), records management with retention policies and legal hold, full-text search with faceted navigation, a compliance-grade immutable archive, disposition scheduling, and deep integration with digital signatures and workflow. The system ensures that content is properly governed from creation through disposal, meeting regulatory requirements for document retention and legal discovery.

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Content Repository Architecture** | Platform Service (Rust + MinIO + PostgreSQL) | Hierarchical content repository supporting documents, images, videos, rich media, and templates; metadata stored in PostgreSQL, binary content in MinIO; folder hierarchy with configurable taxonomy |
| **Document Lifecycle State Machine** | Platform Service (Rust) | Configurable state machine: Draft → Review → Approved → Published → Superseded → Archived → Retired; state transitions governed by workflow rules; audit trail for every transition |
| **Records Management** | Platform Service (Rust) | Retention policies linked to document classification; legal hold preventing disposition; disposition schedules with approval workflows; records classification per regulatory requirements |
| **Content Search with Faceted Navigation** | Platform Service + Elasticsearch | Full-text search across all content with faceted navigation (type, author, date, classification, tags, status); relevance ranking with business context boosting |
| **Compliance Archive** | Platform Service + MinIO | Immutable WORM (Write Once Read Many) archive for regulatory-compliant document retention; tamper-proof storage with hash verification; audit log for archive access |
| **Legal Hold** | Platform Service | Legal hold flag prevents any modification or disposition of held documents; hold management with reason, case reference, and authorized holder; release workflow with approval |
| **Disposition Schedules** | Platform Service | Automated disposition scheduling based on retention policies; disposition approval workflow for sensitive records; certificate of destruction generation |
| **Digital Signatures Integration** | Platform Service | Integration with digital signature service for signing documents during lifecycle transitions; signature verification at document access; multi-signature workflows |

## Document Lifecycle State Machine

```
                    ┌──────────────────────────────────────────┐
                    │          Document Lifecycle               │
                    │                                           │
    ┌────────┐     │   ┌────────┐    ┌────────┐    ┌────────┐ │
    │ Create │─────┼──▶│ Draft  │───▶│ Review │───▶│Approved│ │
    │        │     │   │        │    │        │    │        │ │
    └────────┘     │   │ Edit   │    │ Assign │    │ Sign   │ │
                   │   │ Delete │    │ Reject │    │ Publish│ │
                   │   └───┬────┘    └───┬────┘    └───┬────┘ │
                   │       │             │              │       │
                   │       │◀────────────┘              │       │
                   │       │             ┌──────────────┘       │
                   │       │             ▼                       │
                   │       │     ┌────────────┐    ┌─────────┐ │
                   │       │     │ Published  │───▶│Archived │ │
                   │       │     │            │    │         │ │
                   │       │     │ Supersede  │    │ Retain  │ │
                   │       │     │ Withdraw   │    │ Dispose │ │
                   │       │     └─────┬──────┘    └────┬────┘ │
                   │       │           │                │       │
                   │       │           ▼                ▼       │
                   │       │     ┌──────────┐   ┌──────────┐  │
                   │       │     │Superseded│   │ Retired  │  │
                   │       │     │ (by new  │   │ (disposed│  │
                   │       │     │  version)│   │  /deleted│  │
                   │       │     └──────────┘   └──────────┘  │
                   │       │                                   │
                   │       ▼                                   │
                   │  ┌──────────┐                             │
                   │  │ Deleted  │ (only from Draft)           │
                   │  └──────────┘                             │
                   └──────────────────────────────────────────┘

    ┌──────────────────────────────────────────────────────┐
    │  Legal Hold Overlay (can be applied at any state)     │
    │  • Prevents modification, disposition, and deletion   │
    │  • Applied by authorized legal/compliance users       │
    │  • Released with approval and audit trail             │
    └──────────────────────────────────────────────────────┘
```

## Retention Policy Framework

| Document Class | Retention Period | Trigger Event | Disposition | Regulatory Basis |
|---------------|-----------------|---------------|-------------|-----------------|
| Financial Records | 7 years | Period close | Secure delete | SOX, SOC 2 |
| Tax Records | 7 years | Filing date | Secure delete | Tax regulations |
| Employee Records | 7 years post-termination | Termination date | Anonymize | Labor law |
| Contracts | 10 years post-expiry | Contract end | Review then delete | Commercial law |
| IP / Trade Secrets | Indefinite | Classification change | Review | IP law |
| Compliance Records | 7 years | Audit close | Secure delete | SOC 2, ISO 27001 |
| Customer Communications | 3 years | Last interaction | Anonymize | GDPR |
| Health & Safety Records | 30 years | Incident date | Secure delete | OSHA |
| Marketing Materials | 2 years | Publication date | Archive | Internal policy |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Workflow Engine | Platform Service | Event-driven | Review and approval workflows for lifecycle transitions |
| Digital Signatures | Platform Service | Internal API | Document signing at lifecycle transitions |
| Collaboration | Platform Service | Internal API | Document sharing and co-editing in channels |
| IDP | Platform Service | Internal API | Documents ingested via IDP stored in content repository |
| Knowledge Management | Platform Service | Internal API | Knowledge articles managed through content lifecycle |
| Notification Service | Platform Service | Event-driven | Review requests, approval notifications, disposition alerts |
| Report Service | Report Service | Internal API | Content analytics, storage reports, compliance reports |
| Audit Trail | Platform Service | Event-driven | Full content access and modification audit log |
| Search (Elasticsearch) | Platform Service | Internal API | Content indexing for full-text search |
| Privacy Management | Platform Service | Internal API | Retention policy enforcement, erasure requests |
| Compliance Hub | Platform Service | Internal API | Compliance archive status, retention policy compliance |
| Object Storage (MinIO) | Infrastructure | S3 API | Binary content storage with lifecycle policies |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `content.document.created` | `{document_id, title, classification, author, folder_id}` | New document created | Search Indexer, Audit Trail |
| `content.document.state-changed` | `{document_id, old_state, new_state, changed_by, reason}` | Lifecycle state transition | Workflow Engine, Notification, Audit Trail |
| `content.document.version-added` | `{document_id, version, author, change_summary}` | New version created | Search Indexer, Collaboration |
| `content.document.published` | `{document_id, version, published_by, classification}` | Document published | Search Indexer, Notification |
| `content.document.archived` | `{document_id, archived_by, archive_location, retention_until}` | Document archived | Compliance Hub, Audit Trail |
| `content.document.hold-applied` | `{document_id, hold_reason, case_ref, applied_by}` | Legal hold applied | Audit Trail, Compliance Hub |
| `content.document.hold-released` | `{document_id, released_by, release_reason}` | Legal hold released | Audit Trail, Disposition Scheduler |
| `content.document.disposition-complete` | `{document_id, disposition_type, certificate_ref}` | Disposition completed | Audit Trail, Compliance Hub |
| `content.folder.created` | `{folder_id, name, parent_id, classification}` | New folder created | Search Indexer |
| `content.search.executed` | `{query, facets[], result_count, user_id}` | Search executed | Analytics, Search Optimization |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `platform.workflow.step-completed` | Platform Service | Update document lifecycle state |
| `collab.document.shared` | Platform Service | Index shared document |
| `idp.document.classified` | Platform Service | Store classified document |
| `privacy.erasure.completed` | Platform Service | Erase/anonymize content documents |
| `digital-signatures.signature-completed` | Platform Service | Advance document lifecycle state |
| `compliance.hold-applied` | Platform Service | Apply legal hold to documents |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `content_document` | `id, title, classification, state, current_version, folder_id, author_id, created_at, retention_until` | Has many `content_version`, `content_signature` |
| `content_version` | `id, document_id, version_number, file_ref, file_hash, size_bytes, mime_type, change_summary, created_by` | Belongs to `content_document` |
| `content_folder` | `id, name, parent_id, classification, taxonomy_tags[]` | Has many `content_document`, child `content_folder` |
| `content_retention_policy` | `id, classification, retention_period, disposition_action, trigger_event, regulatory_basis` | Applied to `content_document` by classification |
| `content_legal_hold` | `id, document_id, reason, case_ref, applied_by, applied_at, released_by, released_at` | Belongs to `content_document` |
| `content_disposition` | `id, document_id, scheduled_date, action, status, certificate_ref, completed_at` | Belongs to `content_document` |
| `content_metadata` | `id, document_id, key, value` (EAV pattern) | Belongs to `content_document` |
| `content_signature` | `id, document_id, version, signer_id, signature_ref, signed_at, verified` | Belongs to `content_document` |

## Cross-References

- [Knowledge Management](knowledge-management.md) — Knowledge base features
- [Privacy Management](privacy.md) — Data retention and erasure policies
- [Digital Signatures](../06-services/platform.md) — Document signing workflows
- [Collaboration](collaboration.md) — Document sharing and co-editing
- [Intelligent Document Processing](idp.md) — Automated document ingestion
- [Compliance Hub](compliance-hub.md) — Compliance archive and retention monitoring
- [Architecture Overview](../01-architecture/overview.md) — System context
- [Platform Service](../06-services/platform.md) — Platform service specification
