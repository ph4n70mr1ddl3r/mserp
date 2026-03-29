# Intelligent Document Processing (IDP)

AI-powered document classification, data extraction, and business document integration with self-learning capabilities, managed by the Platform Service with ML inference by the Report Service.

## Overview

Intelligent Document Processing automates the ingestion, classification, extraction, and integration of business documents into MSERP workflows. The system uses ML models to classify incoming documents, extract structured data fields, learn from user corrections to improve accuracy over time, and automatically create business documents (invoices, expense reports, purchase orders) from extracted data. It supports high-volume batch processing with parallel extraction, confidence-based human review queues, and full audit trail for compliance.

## Architecture Diagram

```
┌──────────────────────────────────────────────────────────────────────┐
│                     Intelligent Document Processing                  │
│                                                                       │
│  ┌────────────┐   ┌──────────────┐   ┌────────────────────────────┐ │
│  │ Document   │──▶│ Document     │──▶│ Classification Pipeline    │ │
│  │ Ingestion  │   │ Preprocessing│   │ ┌──────────┐ ┌──────────┐ │ │
│  │            │   │ (OCR, skew   │   │ │ Doc Type │ │ Language │ │ │
│  │ • Upload   │   │  correction, │   │ │ Model    │ │ Detection│ │ │
│  │ • Email    │   │  noise       │   │ └─────┬────┘ └────┬─────┘ │ │
│  │ • Scanner  │   │  removal)    │   │       └──────┬─────┘       │ │
│  │ • API      │   └──────────────┘   └─────────────┼─────────────┘ │
│  └────────────┘                                      │               │
│                                                      ▼               │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │              Data Extraction Engine                           │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐   │   │
│  │  │ Field        │  │ Table        │  │ Key-Value        │   │   │
│  │  │ Extraction   │  │ Extraction   │  │ Pair Extraction  │   │   │
│  │  │ (invoices,   │  │ (line items, │  │ (header fields)  │   │   │
│  │  │  POs, etc.)  │  │  schedules)  │  │                  │   │   │
│  │  └──────┬───────┘  └──────┬───────┘  └────────┬─────────┘   │   │
│  │         └─────────────────┼───────────────────┘              │   │
│  │                           ▼                                   │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐   │   │
│  │  │ Validation   │  │ Confidence   │  │ Template         │   │   │
│  │  │ Engine       │  │ Scoring      │  │ Learning Store   │   │   │
│  │  └──────┬───────┘  └──────┬───────┘  └──────────────────┘   │   │
│  └─────────┼─────────────────┼──────────────────────────────────┘   │
│            │                 │                                       │
│            ▼                 ▼                                       │
│  ┌─────────────────┐  ┌───────────────┐  ┌─────────────────────┐   │
│  │ Human Review    │  │ Business Doc  │  │ Audit Trail &       │   │
│  │ Queue           │  │ Auto-Creation │  │ Document Store      │   │
│  │ (low confidence)│  │ (invoices,    │  │                     │   │
│  └─────────────────┘  │  expenses,    │  └─────────────────────┘   │
│                        │  POs, etc.)  │                             │
│                        └─────────────┘                              │
└──────────────────────────────────────────────────────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Document Classification Pipeline** | Platform Service + Report Service (ONNX) | Multi-stage pipeline: language detection → document type classification (invoice, PO, contract, receipt, shipping doc, tax form, bank statement, other) with confidence scoring |
| **Data Extraction Models** | Report Service (ONNX + Candle) | Per-document-type extraction models: field extraction, table extraction, key-value pair extraction with layout-aware processing |
| **Template Learning System** | Platform Service (Rust) | Self-learning system that captures user corrections and feedback; updates extraction templates; retrains models on schedule |
| **Batch Processing Architecture** | Platform Service (Rust) | High-volume parallel document processing with configurable batch sizes, priority queues, and retry policies |
| **Validation Engine** | Platform Service (Rust) | Cross-field validation (date formats, amounts match line items, tax calculations), business rule validation, and duplicate detection |
| **Business Document Integration** | Platform Service → Finance/Commerce/HCM | Auto-create invoices, expense reports, purchase orders, receipts, and vendor records from extracted data with mapping rules |
| **Confidence Scoring** | Report Service (ONNX) | Per-field confidence score (0-100); fields below threshold flagged for human review; overall document confidence aggregated |
| **Human Review Queue** | Platform Service + Workflow Service | Prioritized review queue with assignment rules, SLA tracking, and feedback capture for model improvement |
| **OCR Integration** | Platform Service (Tesseract/ONNX) | Multi-language OCR with layout preservation for scanned documents and image-based PDFs |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Document Ingestion | Multi-channel document intake (upload, email, API, scanner) | Platform Service |
| Preprocessing Engine | OCR, deskew, denoise, page segmentation, layout analysis | Platform Service |
| Classification Model | Document type classification with confidence | Report Service |
| Extraction Engine | Field, table, and key-value extraction | Report Service |
| Validation Engine | Business rules, cross-field validation, duplicate check | Platform Service |
| Template Manager | Extraction template CRUD, versioning, learning | Platform Service |
| Review Queue | Human review assignment, prioritization, feedback | Platform Service |
| Business Doc Creator | Auto-create business documents from extracted data | Platform Service |
| Batch Processor | Parallel document processing orchestration | Platform Service |
| Audit Logger | Full processing audit trail | Platform Service |

## Model Training Workflow

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  Annotated   │───▶│  Training    │───▶│  Validation  │───▶│  Model       │
│  Dataset     │    │  Pipeline    │    │  (accuracy,  │    │  Registry    │
│  (corrections│    │  (Candle +   │    │   bias,      │    │  (versioned) │
│   + feedback)│    │   ONNX)      │    │   fairness)  │    │              │
│  └──────────┘    └──────────────┘    └──────────────┘    └──────┬───────┘
                                                               │
                                                               ▼
                                                        ┌──────────────┐
                                                        │  Blue-Green  │
                                                        │  Deployment  │
                                                        │  (A/B test)  │
                                                        └──────────────┘
```

| Stage | Trigger | Frequency | Validation Criteria |
|-------|---------|-----------|-------------------|
| Feedback Collection | User correction or feedback on extraction | Continuous | N/A |
| Template Update | Template modification by admin | On demand | N/A |
| Model Retraining | Scheduled (weekly) or accuracy threshold breach | Weekly or on trigger | Accuracy > 95%, no regression > 2% |
| Model Deployment | Model passes validation | On model approval | A/B test with 5% traffic, monitor for 48h |

## Document Types & Extraction Fields

| Document Type | Key Fields Extracted | Table Extraction | Typical Confidence |
|--------------|---------------------|-----------------|-------------------|
| Invoice | vendor, invoice_no, date, due_date, total, tax, currency | line_items (description, qty, unit_price, amount) | 90-95% |
| Purchase Order | supplier, po_no, date, delivery_date, total | line_items (item, qty, price) | 92-97% |
| Receipt | vendor, date, total, payment_method, category | line_items (item, price) | 85-92% |
| Shipping Document | shipper, tracking_no, origin, destination, weight | packages (dimensions, weight) | 88-94% |
| Contract | parties, effective_date, expiry_date, value, type | clauses, amendments | 80-88% |
| Bank Statement | bank, account, period, opening_balance, closing_balance | transactions (date, desc, amount) | 90-96% |
| Tax Form | form_type, period, taxpayer_id, amounts_by_box | supporting schedules | 85-92% |
| Expense Report | employee, period, total, project_code | expense_lines (date, category, amount) | 88-94% |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Accounts Payable | Finance Service | Internal API | Auto-create vendor invoices from extracted invoice data |
| Expense Management | Finance Service | Internal API | Auto-create expense reports from extracted receipt data |
| Purchase Orders | Commerce/Manufacturing Service | Internal API | Match extracted invoices against POs |
| Vendor Management | Commerce/Manufacturing Service | Internal API | Auto-create/update vendor records |
| Document Management | Platform Service | Internal API | Store original and processed documents |
| Workflow Engine | Workflow Service | Event-driven | Human review routing, approval workflows |
| Notification Service | Platform Service | Event-driven | Review assignments, processing completion alerts |
| ML Inference | Report Service | gRPC | Classification and extraction model inference |
| Audit Trail | Platform Service | Event-driven | Full processing audit log |
| Data Import/Export | Platform Service | Internal API | Bulk document import/export |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `platform.idp.document.classified` | `{document_id, doc_type, confidence, classified_at}` | Classification completes | Extraction Engine, Dashboard |
| `platform.idp.extraction.completed` | `{document_id, doc_type, fields, field_count, avg_confidence}` | Extraction completes | Validation Engine, Dashboard |
| `platform.idp.document.validated` | `{document_id, validation_result, issues[]}` | Validation completes | Review Queue, Business Doc Creator |
| `platform.idp.document.review-required` | `{document_id, reason, priority, assigned_to}` | Low confidence or validation failure | Notification Service, Review Queue |
| `platform.idp.document.review-completed` | `{document_id, reviewer, corrections[], feedback}` | Human review completes | Template Learning, Audit Trail |
| `platform.idp.business-doc.created` | `{source_doc_id, business_doc_type, business_doc_id}` | Business document auto-created | Finance/Commerce/HCM Service |
| `platform.idp.batch.completed` | `{batch_id, total, succeeded, failed, review_required}` | Batch processing completes | Dashboard, Notification Service |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `platform.file.uploaded` | Platform Service | Trigger classification pipeline |
| `platform.email.received` | Platform Service | Trigger classification pipeline |
| `report.ml.inference.complete` | Report Service | Apply classification/extraction results |
| `workflow.step.approved` | Workflow Service | Update review status |
| `finance.supplier.created` | Finance Service | Update vendor extraction reference data |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `idp_document` | `id, tenant_id, source_type, original_file_ref, status, doc_type, confidence` | Has many `idp_extracted_field`, `idp_review` |
| `idp_extracted_field` | `id, document_id, field_name, field_value, confidence, bounding_box, corrected_value` | Belongs to `idp_document` |
| `idp_extracted_table` | `id, document_id, table_index, rows, confidence` | Belongs to `idp_document`, has many `idp_extracted_field` |
| `idp_review` | `id, document_id, reviewer, status, started_at, completed_at, feedback` | Belongs to `idp_document` |
| `idp_template` | `id, doc_type, version, field_definitions, extraction_rules, is_active` | Used by extraction engine |
| `idp_batch` | `id, status, total_docs, processed, succeeded, failed, created_at, completed_at` | Has many `idp_document` |

## Cross-References

- [Intelligent Process Automation](intelligent-process-automation.md) and [RPA](../06-services/platform.md) — Bot-based document handling automation (RPA module)
- [Data Import/Export Framework](../06-services/integration.md) — Bulk data operations (Data Import/Export module)
- [Adaptive Intelligence](adaptive-intelligence.md) — ML model lifecycle management
- [Content Management](content-management.md) — Document storage and lifecycle
- [Digital Signatures](../06-services/platform.md) — Signed document verification
- [Architecture Overview](../01-architecture/overview.md) — System context and communication patterns
- [Platform Service](../06-services/platform.md) — Platform service specification
- [NFR: Performance](../10-planning/nfr.md) — IDP document extraction < 10s per page
