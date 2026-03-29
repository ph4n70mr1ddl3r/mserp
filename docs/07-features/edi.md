# Electronic Data Interchange (EDI)

ANSI X12 and EDIFACT electronic document exchange with partner management, mapping templates, and AS2/SFTP transport, managed by the Integration Service.

## Overview

EDI enables automated business document exchange with trading partners using industry-standard formats (ANSI X12 and EDIFACT). The system supports the full lifecycle of EDI transactions: partner onboarding, mapping template configuration, inbound/outbound processing pipelines, transport via AS2 and SFTP, and acknowledgement handling. Standard transaction sets include Purchase Orders (850), Invoices (810), Advance Shipping Notices (856), Functional Acknowledgements (997), and more. The processing pipeline validates, transforms, and routes documents to the appropriate MSERP service for business processing.

## Architecture Diagram

```
┌──────────────────────────────────────────────────────────────────────┐
│                     EDI Processing Pipeline                           │
│                                                                       │
│  ┌──────────────────── Inbound Pipeline ─────────────────────────┐   │
│  │                                                                │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────┐  │   │
│  │  │ Transport│─▶│ Parse &  │─▶│ Validate │─▶│ Transform    │  │   │
│  │  │ (AS2/    │  │ Envelope │  │ (syntax, │  │ & Map to     │  │   │
│  │  │  SFTP)   │  │          │  │  partner) │  │ canonical    │  │   │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────┬───────┘  │   │
│  │                                                       │        │   │
│  └───────────────────────────────────────────────────────┼────────┘   │
│                                                          │            │
│  ┌───────────────────────────────────────────────────────┼────────┐   │
│  │                 Business Document Router               ▼        │   │
│  │  Routes canonical document to the owning MSERP service          │   │
│  └───────────────────────────────────────────────────────┬────────┘   │
│                                                          │            │
│  ┌──────────────────── Outbound Pipeline ────────────────┼────────┐   │
│  │                                                       ▼        │   │
│  │  ┌──────────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │   │
│  │  │ Transform    │─▶│ Validate │─▶│ Envelope │─▶│ Transport│  │   │
│  │  │ & Map from   │  │ &        │  │ (X12/    │  │ (AS2/    │  │   │
│  │  │ canonical    │  │ Generate │  │  EDIFACT) │  │  SFTP)   │  │   │
│  │  └──────────────┘  └──────────┘  └──────────┘  └──────────┘  │   │
│  │                                                                │   │
│  └────────────────────────────────────────────────────────────────┘   │
│                                                                       │
│  ┌───────────────────── Partner Management ───────────────────────┐   │
│  │  Partner profiles, connection configs, mapping templates, SLAs │   │
│  └────────────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **EDI Parser** | Integration Service (Rust) | Parses ANSI X12 (delimiters, segments, elements) and EDIFACT (UNA, UNB, UNH) envelopes; supports all standard transaction sets; handles interchange, group, and transaction-level Acknowledgements (997/CONTRL) |
| **Validation Engine** | Integration Service (Rust) | Syntax validation (segment order, mandatory elements, code list values), partner-specific validation rules, business-level cross-segment validation |
| **Mapping Templates** | Integration Service (Rust) | Configurable mapping from EDI segments/elements to MSERP canonical JSON; per-partner and per-transaction-type templates; versioned templates with rollback |
| **Transport Layer** | Integration Service (Rust) | AS2 (MIME-based, MDN acknowledgement, digital signatures, encryption) and SFTP (key-based auth, polling and push); configurable retry and dead-letter handling |
| **Partner Management** | Integration Service (Rust + PostgreSQL) | Partner profiles with connection credentials, supported transaction sets, mapping template assignments, SLA thresholds, contact information |
| **Canonical Model** | Integration Service (Rust) | Internal canonical JSON representation for all supported transaction types; service-agnostic format used for routing to business services |
| **Acknowledgement Handler** | Integration Service (Rust) | Automatic 997/CONTRL generation for inbound documents; processing of partner acknowledgements for outbound documents; acknowledgement correlation and SLA tracking |
| **Error & Retry** | Integration Service (Rust) | Structured error classification (syntax, mapping, transport, business); configurable retry policies per partner; dead-letter queue for unprocessable documents |

## Supported Transaction Sets

| Standard | Transaction Set | Code | MSERP Integration |
|----------|----------------|------|-------------------|
| ANSI X12 | Purchase Order | 850 | Commerce Service — create sales order |
| ANSI X12 | Purchase Order Acknowledgement | 855 | Commerce Service — update order status |
| ANSI X12 | Invoice | 810 | Finance Service — create vendor invoice |
| ANSI X12 | Advance Shipping Notice | 856 | Commerce Service — create delivery/shipment |
| ANSI X12 | Purchase Order Change | 860 | Commerce Service — update order |
| ANSI X12 | Invoice Inquiry | 824 | Finance Service — query invoice status |
| ANSI X12 | Functional Acknowledgement | 997 | Integration Service — acknowledgement processing |
| ANSI X12 | Application Advice | 824 | Integration Service — error/notification |
| ANSI X12 | Price/Sales Catalog | 832 | Commerce Service — update product pricing |
| ANSI X12 | Inventory Inquiry/Advice | 846 | Commerce Service — inventory snapshot |
| EDIFACT | Order | ORDERS | Commerce Service — create sales order |
| EDIFACT | Invoice | INVOIC | Finance Service — create vendor invoice |
| EDIFACT | Despatch Advice | DESADV | Commerce Service — create delivery/shipment |
| EDIFACT | Order Response | ORDRSP | Commerce Service — update order status |
| EDIFACT | Functional Acknowledgement | CONTRL | Integration Service — acknowledgement processing |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Transport Manager | AS2 and SFTP connection handling, polling, delivery | Integration Service |
| EDI Parser | X12 and EDIFACT envelope and segment parsing | Integration Service |
| Validation Engine | Syntax, partner rules, and business validation | Integration Service |
| Mapping Engine | EDI-to-canonical and canonical-to-EDI transformation | Integration Service |
| Template Manager | Mapping template CRUD, versioning, assignment | Integration Service |
| Partner Manager | Partner profiles, credentials, transaction set config | Integration Service |
| Acknowledgement Handler | 997/CONTRL generation and processing | Integration Service |
| Document Router | Route canonical documents to owning services | Integration Service |
| Retry & DLQ Manager | Retry policies, dead-letter queue, alerting | Integration Service |
| Audit Logger | Full EDI processing audit trail | Integration Service |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Commerce Service (Orders) | Commerce Service | Internal API + Events | Create/update sales orders from 850/ORDERS; send 855/ORDRSP on order confirmation |
| Commerce Service (Shipments) | Commerce Service | Internal API + Events | Create deliveries from 856/DESADV; send ASN on shipment |
| Finance Service (Invoices) | Finance Service | Internal API + Events | Create vendor invoices from 810/INVOIC; send invoices to partners |
| Commerce Service (Pricing) | Commerce Service | Internal API | Update pricing from 832 catalog feeds |
| Commerce Service (Inventory) | Commerce Service | Internal API | Process 846 inventory snapshots |
| Notification Service | Platform Service | Event-driven | EDI processing errors, SLA breaches |
| Audit Trail | Platform Service | Event-driven | Full EDI document audit log |
| Master Data Management | Integration Service | Internal API | Partner master data synchronisation |
| Workflow Engine | Workflow Service | Event-driven | Exception handling workflows |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `integration.edi.document.received` | `{transaction_id, partner_id, transaction_type, standard, direction, received_at}` | Inbound document parsed and validated | Commerce/Finance Service, Audit Trail |
| `integration.edi.document.sent` | `{transaction_id, partner_id, transaction_type, standard, direction, sent_at}` | Outbound document delivered to partner | Audit Trail, Analytics |
| `integration.edi.document.failed` | `{transaction_id, partner_id, transaction_type, error_code, error_detail, retry_count}` | Document processing fails after retries | Notification Service, Audit Trail |
| `integration.edi.partner.connected` | `{partner_id, transport_type, connection_id, connected_at}` | New partner connection established | Audit Trail, Analytics |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `commerce.order.created` | Commerce Service | Generate outbound 855/ORDRSP to partner |
| `commerce.order.updated` | Commerce Service | Generate outbound 860 change notification |
| `finance.invoice.created` | Finance Service | Generate outbound 810/INVOIC to partner |
| `commerce.shipment.created` | Commerce Service | Generate outbound 856/DESADV to partner |
| `mdm.partner.created` | Integration Service | Create EDI partner profile |
| `mdm.partner.updated` | Integration Service | Update EDI partner configuration |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `edi_partner` | `id, tenant_id, partner_code, name, standard, transport_type, connection_config, is_active` | Has many `edi_transaction`, `edi_mapping` |
| `edi_transaction` | `id, tenant_id, partner_id, direction, standard, transaction_type, status, raw_payload_ref, canonical_payload, ack_status, error_detail` | Belongs to `edi_partner` |
| `edi_mapping` | `id, tenant_id, partner_id, transaction_type, standard, version, mapping_rules, is_active` | Belongs to `edi_partner` |
| `edi_acknowledgement` | `id, transaction_id, ack_type, ack_status, received_at, detail` | Belongs to `edi_transaction` |
| `edi_connection_log` | `id, partner_id, transport_type, direction, status, timestamp, detail` | References `edi_partner` |

## Cross-References

- [Master Data Management](../06-services/integration.md) — Partner master data synchronisation (MDM module)
- [Trade Compliance](../06-services/integration.md) — Export control screening for partner transactions (Trade Compliance module)
- [Data Import/Export Framework](../06-services/integration.md) — Bulk data operations (Data Import/Export module)
- [Notification Service](../06-services/platform.md) — Error and SLA alerting
- [Architecture Overview](../01-architecture/overview.md) — System context
- [Integration Service](../06-services/integration.md) — Integration service specification
