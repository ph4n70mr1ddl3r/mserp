# Touchless Invoicing

AI-powered end-to-end AP invoice automation with zero-touch processing, managed by the Finance Service.

## Overview

Touchless Invoicing eliminates manual intervention in the accounts payable invoice processing lifecycle by combining Intelligent Document Processing (IDP), three-way matching (PO-receipt-invoice), ML-based line-item matching, and automated GL coding. The system ingests invoices from multiple capture channels, extracts and validates data, performs matching against purchase orders and goods receipts, predicts GL account assignments, detects duplicates and anomalies, and routes only exceptions to human processors. Straight-through processing (STP) rate is continuously tracked and improved through machine learning that learns from user corrections.

## Data Flow Diagram

```
┌──────────────┐     ┌──────────────────┐     ┌─────────────────────────┐
│  Invoice     │────▶│  IDP Extraction  │────▶│  Data Validation &      │
│  Capture     │     │  (Platform IDP)  │     │  Enrichment              │
│  (email,     │     │  ┌─────────────┐ │     │  ┌───────────────────┐  │
│   portal,    │     │  │ OCR + NLP   │ │     │  │ Supplier lookup   │  │
│   upload,    │     │  │ field extract│ │     │  │ Tax calculation   │  │
│   EDI)       │     │  └─────────────┘ │     │  │ Line-item parse   │  │
└──────────────┘     └──────────────────┘     │  └───────┬───────────┘  │
                                                └──────────┼─────────────┘
                                                           │
                              ┌────────────────────────────┼───────────────┐
                              │                            │               │
                     ┌────────▼────────┐        ┌─────────▼──────┐  ┌────▼──────┐
                     │  3-Way Match    │        │  Auto-Coding   │  │  Duplicate │
                     │  Engine         │        │  (ML Predict)  │  │  Detection │
                     │  PO ↔ GR ↔ INV │        │  GL Account    │  │  & Anomaly │
                     └────────┬────────┘        └─────────┬──────┘  └────┬──────┘
                              │                           │              │
                              ▼                           ▼              ▼
                     ┌────────────────────────────────────────────────────────┐
                     │           Confidence Scoring & Decision Engine         │
                     │  STP Path: All scores > threshold → Auto-process       │
                     │  Exception Path: Any score < threshold → Route to team │
                     └─────────────────────────┬──────────────────────────────┘
                                               │
                      ┌────────────────────────┼───────────────────────┐
                      │                        │                       │
             ┌────────▼────────┐     ┌─────────▼──────┐     ┌────────▼────────┐
             │  Auto-Processed │     │  Exception      │     │  Metrics &      │
             │  (STP Invoice)  │     │  Routing Queue  │     │  Learning Loop  │
             │  → Post to GL   │     │  → Human Review │     │  → Model Update │
             └─────────────────┘     └────────────────┘     └─────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Invoice Capture** | Finance Service + Platform Service | Multi-channel capture: email ingestion with attachment parsing, supplier portal upload, manual upload, EDI (XML/EDIFACT), drag-and-drop. All channels route to IDP for extraction |
| **IDP Integration** | Platform Service (IDP) → Finance Service | Leverages Platform Service's Intelligent Document Processing for OCR, field extraction, and structured data output. Finance Service subscribes to `platform.idp.extraction.completed` events |
| **3-Way Matching Engine** | Finance Service (Rust) | Matches invoice header and line items against Purchase Order and Goods Receipt: quantity tolerance, price tolerance, and amount tolerance. Supports one-to-one, one-to-many, and many-to-one matching scenarios |
| **ML Line-Item Matching** | Report Service (ONNX) → Finance Service | ML model predicts line-item to PO line matching based on description similarity, historical patterns, and commodity codes. Confidence score drives auto-match vs. exception routing |
| **Auto-Coding (GL Prediction)** | Report Service (ONNX) → Finance Service | ML model predicts GL account assignments based on supplier, commodity, description, amount patterns, and historical coding. Learns from corrections via feedback loop |
| **Duplicate Detection** | Finance Service (Rust) | Multi-signal duplicate detection: supplier + invoice number, supplier + amount + date proximity, line-item fingerprint similarity. Configurable sensitivity thresholds |
| **Anomaly Flagging** | Finance Service (Rust) | Statistical anomaly detection on amount, quantity, pricing, and terms compared against supplier norms and historical patterns |
| **STP Rate Tracking** | Finance Service (Rust) | Real-time and periodic metrics: STP rate, match rates, coding accuracy, exception volumes, processing time. Per-supplier and aggregate views |
| **Exception Routing** | Finance Service + Workflow Service | Rule-based and confidence-score-based routing of exceptions to qualified processors. Queue management with priority, SLA, and escalation |
| **Continuous Learning** | Report Service (ONNX) | User corrections feed back into ML training pipeline. Models retrained on configurable schedule. A/B testing of model versions with accuracy tracking |
| **Supplier Portal Integration** | Commerce Service + Finance Service | Supplier portal for invoice submission, status tracking, and communication. Pre-submission validation reduces exceptions |
| **Early Payment Discount Detection** | Finance Service (Rust) | Parses invoice terms for early payment discount opportunities and integrates with Dynamic Discounting feature |
| **Storage** | PostgreSQL + MinIO | Invoice data and processing results in PostgreSQL; invoice documents and attachments in MinIO |

## Matching Algorithms

| Algorithm | Match Type | Use Case | Typical Accuracy |
|-----------|-----------|----------|-----------------|
| Exact PO Match | Invoice PO number + line number = PO line | Standard PO-backed invoices | 50-60% STP |
| Fuzzy PO Match | Partial PO reference + supplier + amount | Truncated or modified PO references | +10-15% additional |
| Line-Item ML Match | Description + commodity + quantity + price | Complex or split-line invoices | +10-15% additional |
| Receipt Match | PO line + GR quantity received ≥ invoice qty | 3-way quantity validation | +5-10% additional |
| Non-PO Match | Supplier + category + historical patterns | Non-PO invoices (utilities, services) | +5-8% additional |
| Duplicate Fingerprint | Supplier + invoice# + amount + date | Multi-signal duplicate detection | 99%+ detection rate |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Intelligent Document Processing | Platform Service | Event-driven | Invoice OCR, field extraction, structured data output |
| Accounts Payable | Finance Service | Internal API | Invoice creation, approval, payment scheduling |
| Purchase Orders | Commerce Service | Internal API | PO data for matching, goods receipts |
| Order Fulfillment | Commerce Service | Event-driven | Shipment/delivery confirmations for receipt matching |
| General Ledger | Finance Service | Internal API | GL account master, coding validation, journal posting |
| Supplier Portal | Commerce Service | Internal API | Supplier invoice submission, status queries |
| Workflow Engine | Workflow Service | Event-driven | Exception routing, approval workflows |
| ML Inference | Report Service | gRPC | Line-item matching, GL coding prediction, anomaly detection |
| Dynamic Discounting | Finance Service | Internal API | Early payment discount detection and evaluation |
| Notification Service | Platform Service | Event-driven | Exception alerts, processing confirmations |
| Audit Trail | Platform Service | Event-driven | Full audit log of touchless processing decisions |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `finance.touchless-invoice.captured` | `{capture_id, source, supplier_id, invoice_number, document_ref, extracted_fields}` | Invoice captured and extracted from IDP | Matching Engine, Dashboard |
| `finance.touchless-invoice.matched` | `{invoice_id, match_type, po_ids[], gr_ids[], match_confidence, line_match_count, unmatched_lines}` | Matching engine completes for invoice | Auto-Coding Engine, Dashboard |
| `finance.touchless-invoice.auto-coded` | `{invoice_id, gl_assignments[], coding_confidence, predicted_by_model_version}` | GL coding prediction completes | Validation, Dashboard |
| `finance.touchless-invoice.exception-created` | `{exception_id, invoice_id, exception_type, confidence_score, assigned_to, priority}` | Processing confidence below STP threshold | Workflow Engine, Notification Service |
| `finance.touchless-invoice.processed` | `{invoice_id, processing_path, processing_time_ms, stp_flag, journal_entry_id}` | Invoice fully processed (auto or manual) | GL, Metrics, Dashboard |
| `finance.touchless-invoice.metrics.updated` | `{period, total_invoices, stp_count, stp_rate, match_rate, coding_accuracy, avg_processing_time}` | Periodic metrics snapshot computed | Dashboard, Report Service |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `platform.idp.extraction.completed` | Platform Service | Ingest extracted invoice data, begin matching and validation |
| `finance.invoice.created` | Finance Service | Evaluate invoice for touchless processing eligibility |
| `commerce.order.shipped` | Commerce Service | Record goods receipt data for 3-way matching |
| `finance.purchase-order.confirmed` | Finance Service | Update PO data available for matching |
| `report.ml.inference.complete` | Report Service | Apply ML model predictions for matching and coding |
| `workflow.step.approved` | Workflow Service | Process exception resolution and continue invoice flow |

## Data Model Reference

### `invoice_automation_rules`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `name` | VARCHAR(200) | Rule name |
| `description` | TEXT | Rule description |
| `rule_type` | VARCHAR(50) | `matching`, `coding`, `routing`, `duplicate_detection`, `anomaly`, `stp_threshold` |
| `priority` | INTEGER | Execution priority (lower = higher priority) |
| `conditions` | JSONB | Rule conditions (supplier, category, amount range, etc.) |
| `actions` | JSONB | Rule actions (auto-match, auto-code, route to, flag, etc.) |
| `is_active` | BOOLEAN | Whether the rule is active |
| `effective_from` | DATE | Rule effective start date |
| `effective_to` | DATE | Rule effective end date (nullable) |
| `applied_count` | INTEGER | Number of times rule has been applied |
| `last_applied_at` | TIMESTAMPTZ | Last application timestamp |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

### `invoice_matching_results`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `invoice_id` | UUID | Reference to the AP invoice |
| `match_type` | VARCHAR(30) | `exact_po`, `fuzzy_po`, `ml_line_item`, `receipt`, `non_po`, `no_match` |
| `po_id` | UUID | Matched purchase order (nullable) |
| `po_line_id` | UUID | Matched PO line (nullable) |
| `gr_id` | UUID | Matched goods receipt (nullable) |
| `invoice_line_id` | UUID | Invoice line being matched (nullable) |
| `confidence_score` | DECIMAL(5,2) | Match confidence (0-100) |
| `match_details` | JSONB | Detailed match breakdown (field-level scores) |
| `is_auto_accepted` | BOOLEAN | Whether match was auto-accepted (STP) |
| `reviewed_by` | UUID | Manual reviewer (nullable) |
| `reviewed_at` | TIMESTAMPTZ | Manual review timestamp (nullable) |
| `status` | VARCHAR(20) | `pending`, `auto_matched`, `manually_matched`, `rejected`, `escalated` |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

### `invoice_coding_suggestions`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `invoice_id` | UUID | Reference to the AP invoice |
| `invoice_line_id` | UUID | Invoice line being coded (nullable) |
| `gl_account_id` | UUID | Suggested GL account |
| `cost_center_id` | UUID | Suggested cost center (nullable) |
| `dimension_values` | JSONB | Additional dimension suggestions (project, department, etc.) |
| `confidence_score` | DECIMAL(5,2) | Coding confidence (0-100) |
| `prediction_source` | VARCHAR(30) | `ml_model`, `rule`, `supplier_default`, `historical`, `manual` |
| `model_version` | VARCHAR(50) | ML model version used (nullable) |
| `is_accepted` | BOOLEAN | Whether suggestion was accepted |
| `corrected_gl_account_id` | UUID | User-corrected GL account (nullable, for learning) |
| `corrected_by` | UUID | User who corrected (nullable) |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

### `invoice_exceptions`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `invoice_id` | UUID | Reference to the AP invoice |
| `exception_type` | VARCHAR(50) | `match_failure`, `coding_failure`, `duplicate_suspected`, `anomaly_detected`, `price_variance`, `quantity_variance`, `missing_po`, `missing_gr`, `validation_error` |
| `severity` | VARCHAR(10) | `low`, `medium`, `high`, `critical` |
| `description` | TEXT | Exception description |
| `confidence_score` | DECIMAL(5,2) | Overall processing confidence score |
| `details` | JSONB | Structured exception details |
| `status` | VARCHAR(20) | `open`, `in_progress`, `resolved`, `escalated`, `dismissed` |
| `assigned_to` | UUID | Processor assigned |
| `assigned_team` | VARCHAR(100) | Team queue assigned to |
| `priority` | INTEGER | Queue priority (lower = higher priority) |
| `due_at` | TIMESTAMPTZ | SLA deadline |
| `resolved_by` | UUID | Resolver (nullable) |
| `resolved_at` | TIMESTAMPTZ | Resolution timestamp (nullable) |
| `resolution` | TEXT | Resolution notes |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

### `invoice_automation_metrics`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `period_start` | DATE | Metric period start |
| `period_end` | DATE | Metric period end |
| `supplier_id` | UUID | Supplier (nullable — null = aggregate) |
| `total_invoices` | INTEGER | Total invoices processed |
| `stp_invoices` | INTEGER | Invoices processed without human touch |
| `stp_rate` | DECIMAL(5,2) | STP percentage |
| `auto_matched_count` | INTEGER | Invoices auto-matched |
| `auto_matched_rate` | DECIMAL(5,2) | Auto-match percentage |
| `auto_coded_count` | INTEGER | Invoices auto-coded |
| `auto_coded_rate` | DECIMAL(5,2) | Auto-code percentage |
| `coding_accuracy` | DECIMAL(5,2) | Percentage of auto-codes accepted without correction |
| `exception_count` | INTEGER | Total exceptions raised |
| `duplicate_detected_count` | INTEGER | Duplicates detected |
| `anomaly_detected_count` | INTEGER | Anomalies detected |
| `avg_processing_time_ms` | INTEGER | Average end-to-end processing time |
| `avg_exception_resolution_time_ms` | INTEGER | Average exception resolution time |
| `median_processing_time_ms` | INTEGER | Median end-to-end processing time |
| `p95_processing_time_ms` | INTEGER | 95th percentile processing time |
| `model_version` | VARCHAR(50) | ML model version used |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

### `supplier_invoice_templates`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `tenant_id` | UUID | Tenant isolation (RLS) |
| `supplier_id` | UUID | Supplier reference |
| `template_name` | VARCHAR(200) | Template name |
| `invoice_format` | VARCHAR(30) | `pdf`, `xml`, `edi`, `csv`, `image` |
| `field_mappings` | JSONB | Mapping of extracted fields to invoice fields |
| `default_gl_account_id` | UUID | Default GL account for this supplier |
| `default_cost_center_id` | UUID | Default cost center (nullable) |
| `default_dimensions` | JSONB | Default dimension values |
| `matching_preferences` | JSONB | Preferred matching strategy and tolerances |
| `auto_approve_threshold` | DECIMAL(18,2) | Amount below which auto-approval is permitted |
| `is_active` | BOOLEAN | Whether template is active |
| `usage_count` | INTEGER | Number of times template has been used |
| `last_used_at` | TIMESTAMPTZ | Last usage timestamp |
| `created_at` | TIMESTAMPTZ | Record creation timestamp |
| `updated_at` | TIMESTAMPTZ | Record update timestamp |
| `created_by` | UUID | User who created |
| `updated_by` | UUID | User who last updated |
| `version` | INTEGER | Optimistic concurrency version |
| `is_deleted` | BOOLEAN | Soft delete flag |

## STP Decision Model

```
┌─────────────────────────────────────────────────────────────────────────┐
│              Touchless Processing Decision Engine                       │
│                                                                          │
│  Invoice → Capture → Extract → Validate → Match → Code → Decide        │
│                                                                          │
│  Decision Matrix:                                                        │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  IF match_confidence >= 90                                      │    │
│  │    AND coding_confidence >= 85                                  │    │
│  │    AND no_duplicate_flag                                        │    │
│  │    AND no_anomaly_flag                                          │    │
│  │    AND amount <= auto_approve_limit                             │    │
│  │  → STP: Auto-process (no human touch)                           │    │
│  │                                                                  │    │
│  │  IF match_confidence >= 70 AND < 90                             │    │
│  │    OR coding_confidence >= 60 AND < 85                          │    │
│  │  → Exception: Route to AP team with confidence scores           │    │
│  │                                                                  │    │
│  │  IF match_confidence < 70                                       │    │
│  │    OR duplicate_flag = true                                     │    │
│  │    OR anomaly_flag = true                                       │    │
│  │  → Exception: Route to senior AP with high priority             │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                          │
│  Continuous Learning:                                                    │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  User corrections → Feedback store → Training pipeline          │    │
│  │  → Model retrain → A/B test → Promote if accuracy improved     │    │
│  └─────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────┘
```

## Business Rules

| Rule | Description |
|------|-------------|
| **BR-TI-001** | An invoice is eligible for touchless processing only if it passes all validation checks: supplier is active, required fields are present, tax calculation is valid, and no business rules are violated. |
| **BR-TI-002** | Three-way matching tolerances are configurable per supplier: quantity tolerance (default 5%), price tolerance (default 2%), and amount tolerance (default 1%). Variances within tolerance are auto-accepted. |
| **BR-TI-003** | Straight-through processing requires all of the following: match confidence >= threshold (default 90), coding confidence >= threshold (default 85), no duplicate flag, no anomaly flag, and invoice amount <= auto-approve limit. |
| **BR-TI-004** | Duplicate detection must check all incoming invoices against invoices from the same supplier within the last 365 days. A duplicate is flagged if supplier + invoice number matches, or if supplier + amount + date is within configurable proximity. |
| **BR-TI-005** | All auto-coding predictions with confidence < 100% are logged for learning. User corrections to auto-coded entries feed back into the ML training pipeline within 24 hours. |
| **BR-TI-006** | Exceptions must be routed to the appropriate team queue within 5 seconds of detection. SLA timers start immediately. Escalation occurs if not resolved within the configured SLA window. |
| **BR-TI-007** | Supplier invoice templates override global automation rules. If a supplier has a template, its field mappings, default coding, and matching preferences take precedence. |
| **BR-TI-008** | The STP rate must be calculated and published daily. A decline of > 5 percentage points from the previous week's average triggers an automatic alert to the AP manager. |
| **BR-TI-009** | Early payment discount terms detected on invoices must be evaluated and surfaced to the Dynamic Discounting feature for APR analysis and early payment decision. |
| **BR-TI-010** | All touchless processing decisions (auto-accept, exception route, manual override) are logged with full context: confidence scores, model version, rule evaluations, and timestamps. |
| **BR-TI-011** | ML model version changes must go through A/B testing with at least 500 invoices before promotion. Promotion requires equal or improved accuracy on all key metrics. |
| **BR-TI-012** | Invoices exceeding the auto-approve threshold amount (configurable per tenant) require at least one human approval even if all confidence scores exceed STP thresholds. |

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `finance.touchless.stp_match_threshold` | 90 | Minimum match confidence for STP (0-100) |
| `finance.touchless.stp_coding_threshold` | 85 | Minimum coding confidence for STP (0-100) |
| `finance.touchless.auto_approve_limit` | 100000 | Maximum invoice amount for full STP (no human) |
| `finance.touchless.match_quantity_tolerance` | 5.0 | Quantity tolerance for 3-way match (%) |
| `finance.touchless.match_price_tolerance` | 2.0 | Price tolerance for 3-way match (%) |
| `finance.touchless.match_amount_tolerance` | 1.0 | Amount tolerance for 3-way match (%) |
| `finance.touchless.duplicate_lookback_days` | 365 | Days to look back for duplicate detection |
| `finance.touchless.exception_sla_hours` | 24 | Hours before exception escalation |
| `finance.touchless.metrics_snapshot_interval` | 3600 | Seconds between metric snapshots |
| `finance.touchless.model_retrain_interval_days` | 7 | Days between ML model retraining |

## Cross-References

- [Dynamic Discounting](dynamic-discounting.md) — Early payment discount detection and APR evaluation
- [Account Reconciliation](reconciliation.md) — Invoice-to-payment reconciliation
- [Shared Services Center](shared-services-center.md) — Centralized AP processing workbench
- [Finance Service](../06-services/finance.md) — Finance service specification
- [Platform Service](../06-services/platform.md) — IDP (Intelligent Document Processing) module
- [Report Service](../06-services/report.md) — ML inference pipeline and model management
- [Architecture Overview](../01-architecture/overview.md) — System communication patterns
- [NFR: Performance](../10-planning/nfr.md) — Touchless processing latency requirements
