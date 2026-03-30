# Advanced Financial Reporting Studio

Enterprise financial report builder with XBRL taxonomy, management reporting, regulatory report templates, and collaborative annotations, managed by the Report Service (report builder) + Finance Service (financial data).

## Overview

Advanced Financial Reporting Studio provides a drag-and-drop report designer purpose-built for financial reporting, combining the structure of regulatory compliance with the flexibility of management reporting. It supports XBRL taxonomy mapping and tagging for electronic filing, pre-built templates for SEC, IFRS, and local GAAP regulatory reports, multi-entity consolidated reporting with currency translation, and collaborative annotation workflows for narrative sections. The studio enables report bursting by entity or department, scheduled distribution with version-controlled approvals, interactive drill-down from summary to transaction level, and export to PDF, Excel, HTML, and XBRL formats — all within a single integrated authoring environment.

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Drag-and-Drop Report Designer** | Report Service (React + Canvas) | Visual report layout editor with financial-specific components: balance sheet grid, income statement format, cash flow waterfall, KPI cards; drag-and-drop placement, alignment guides, responsive sections; pixel-perfect print layout mode |
| **XBRL Taxonomy Mapping & Tagging** | Report Service (Rust) | Import and manage XBRL taxonomies (US GAAP, IFRS, local extensions); drag-to-tag interface mapping report cells to taxonomy elements; automatic dimension and context inference; validation against taxonomy rules |
| **Regulatory Report Templates** | Report Service (Rust) | Pre-built templates: 10-K, 10-Q, 8-K (SEC), IFRS full set, local GAAP statutory reports; template versioning aligned with regulatory updates; template customization with locked regulatory sections |
| **Management Report Builder** | Report Service (React) | Free-form management report builder with commentary sections, embedded charts, KPI scorecards, and variance tables; narrative sections with rich text editing; auto-populated data sections linked to financial models |
| **Multi-Entity Consolidated Reporting** | Finance Service + Report Service | Consolidated report generation across entities: automatic elimination entries, minority interest calculation, currency translation (temporal/current rate method), segment reporting |
| **Currency Translation in Reports** | Finance Service (Rust) | Automatic currency translation using configurable methods (current rate, temporal, weighted average); translation adjustment reporting; dual-currency display support |
| **Report Versioning & Approvals** | Report Service + Workflow Service | Full report version history with diff; multi-level approval workflow: author → reviewer → approver → locked; comment and markup annotations per version; electronic sign-off |
| **Scheduled Report Distribution** | Report Service (Rust) | Configurable report schedules (daily, weekly, monthly, quarterly, annual); distribution lists by entity, department, or role; format selection per recipient (PDF, Excel, HTML, XBRL); report bursting |
| **Interactive Drill-Down Reports** | Report Service (React) | Click-through drill from summary to detail: report total → entity → department → account → subledger → transaction; breadcrumb navigation; persistent drill state for audit |
| **Embedded Charts & Visualizations** | Report Service (D3/Chart.js) | Financial chart library: waterfall charts, bridge charts, trend lines, variance bars, KPI gauges; charts auto-update with data changes; export charts embedded in PDF/HTML |
| **Report Bursting** | Report Service (Rust) | Split a single report template into individual reports per entity, department, or segment; personalized cover pages; automated distribution with security enforcement; burst status tracking |
| **XBRL Validation & Filing** | Report Service (Rust) | Pre-filing XBRL validation: taxonomy compliance, calculation consistency, dimension validation; validation error resolution workflow; direct filing integration with SEC EDGAR, HMRC, and local regulators |
| **Narrative Report Sections** | Report Service (React) | Rich text narrative sections with collaborative annotations: threaded comments, change tracking, reviewer markup; auto-save with version history; approval gating for narrative sections |
| **Export Engine** | Report Service (Rust) | Multi-format export: PDF (pixel-perfect), Excel (data + formatting), HTML (interactive with drill-down), XBRL (tagged instance document); batch export with watermarking |

## Report Types

| Report Type | Templates | XBRL Required | Consolidation | Frequency | Audience |
|------------|-----------|--------------|---------------|-----------|----------|
| **SEC Filing (10-K)** | Balance Sheet, Income Statement, Cash Flow, Equity, Notes | Yes (US GAAP) | Yes | Annual | Regulators, Investors |
| **SEC Filing (10-Q)** | Condensed Financial Statements, Notes | Yes (US GAAP) | Yes | Quarterly | Regulators, Investors |
| **IFRS Full Set** | Statement of FP, P&L, OCI, Cash Flow, Equity, Notes | Yes (IFRS) | Yes | Annual / Interim | Regulators, Investors |
| **Local GAAP Statutory** | Jurisdiction-specific templates | Varies | Per entity | Annual | Local Regulators |
| **Management Pack** | Executive Summary, KPI Dashboard, Variance Analysis | No | Yes | Monthly | C-Suite, Board |
| **Segment Report** | Revenue, Profit, Assets by segment | Optional | Yes | Quarterly | Internal Management |
| **Budget vs. Actual** | Department/Entity level variance report | No | Configurable | Monthly | Department Heads, Finance |
| **Cash Flow Forecast** | Rolling 13-week, direct/indirect method | No | Yes | Weekly | Treasury |
| **Board Report** | Customizable board package with narrative | No | Yes | Quarterly | Board of Directors |
| **Intercompany Netting** | IC balances, elimination summary | No | Yes | Monthly | Finance Team |

## XBRL Integration

```
┌──────────────────────────────────────────────────────────────────────────┐
│                      XBRL Taxonomy Integration                            │
│                                                                          │
│  ┌──────────────────────┐    ┌──────────────────────────────────────┐   │
│  │  Taxonomy Manager     │    │  Tagging Interface                    │   │
│  │                       │    │                                       │   │
│  │  • US GAAP (2024)    │    │  Report Cell ──▶ XBRL Element        │   │
│  │  • IFRS (2024)       │    │  • Drag-to-tag mapping                │   │
│  │  • Local Extensions  │    │  • Auto-suggest from context          │   │
│  │  • Custom Extensions │    │  • Dimension assignment               │   │
│  │  • Version Mgmt      │    │  • Context (entity, period, scenario) │   │
│  └──────────┬───────────┘    └──────────────┬───────────────────────┘   │
│              │                               │                           │
│              └───────────────┬───────────────┘                           │
│                              │                                           │
│  ┌───────────────────────────▼───────────────────────────────────────┐  │
│  │                   Validation Engine                                │  │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐ │  │
│  │  │ Taxonomy   │  │ Calculation│  │ Dimension  │  │ Filing     │ │  │
│  │  │ Compliance │  │ Consistency│  │ Validation │  │ Readiness  │ │  │
│  │  └────────────┘  └────────────┘  └────────────┘  └────────────┘ │  │
│  └───────────────────────────┬───────────────────────────────────────┘  │
│                              │                                           │
│  ┌───────────────────────────▼───────────────────────────────────────┐  │
│  │                   Export & Filing                                  │  │
│  │  • XBRL Instance Document generation                              │  │
│  │  • Inline XBRL (iXBRL) HTML rendering                             │  │
│  │  • SEC EDGAR filing integration                                    │  │
│  │  • Local regulator filing (HMRC, ACRA, etc.)                      │  │
│  │  • Filing status tracking and confirmation                        │  │
│  └───────────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────────┘
```

## Report Designer Components

| Component | Type | Data Binding | Description |
|-----------|------|-------------|-------------|
| **Financial Grid** | Layout | GL Account / Line Item | Multi-column financial grid with automatic totaling, indent levels, and sign conventions |
| **KPI Card** | Display | Measure / Calculation | Single KPI with value, trend indicator, target comparison, and sparkline |
| **Waterfall Chart** | Chart | Bridge Data | Variance waterfall showing bridge from opening to closing balance |
| **Variance Table** | Data Table | Actual vs. Plan/Budget | Side-by-side comparison with variance amount, percentage, and conditional formatting |
| **Trend Line** | Chart | Time-Series Data | Multi-period trend with optional forecast projection |
| **Rich Text Block** | Narrative | Manual / Template | Collaborative text section with comments, track changes, and approval |
| **Consolidation Roll-Up** | Data Table | Entity-Level Data | Entity-by-entity roll-up with eliminations and consolidated total |
| **Segment Breakdown** | Chart / Table | Segment Data | Revenue, profit, or KPI by business segment, geography, or product line |
| **Embedded Sub-Report** | Layout | Report Reference | Nest another report template as a section within the current report |
| **Drill-Down Link** | Interactive | Detail Query | Click-through link to detailed report or transaction list |
| **Image / Logo** | Display | Static | Company logo, signature blocks, certification images |
| **Page Header / Footer** | Layout | Dynamic | Page numbers, report date, confidentiality notice, approval status |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| General Ledger | Finance Service | Internal API | Trial balance, journal detail, account balances for report data |
| Consolidation | Finance Service | Internal API | Consolidated balances, elimination entries, currency translation |
| Revenue Recognition | Finance Service | Internal API | Recognized revenue data for revenue reports and disclosures |
| Budget / Planning | Finance Service | Internal API | Budget and plan data for variance reporting |
| Report Service (Analytics) | Report Service | Internal API | Embedded analytics, charts, and data visualization |
| Platform Workflow | Workflow Service | Event-driven | Report approval workflows, review routing, electronic sign-off |
| Notification Service | Platform Service | Event-driven | Report assignment, review requests, approval confirmations |
| Content Management | Platform Service | Internal API | Report template storage, archive, and retrieval |
| Audit Trail | Platform Service | Event-driven | Full audit of report access, modifications, and approvals |
| Scheduled Jobs | Platform Service | Internal API | Report generation scheduling and distribution |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `finance.report.generated` | `{report_id, template_id, entity_ids[], period, created_by}` | New report instance created | Dashboard, Audit Trail |
| `finance.report.data-refreshed` | `{report_id, data_source, records_updated, refreshed_at}` | Report data source refreshed | Dashboard, Report Author |
| `finance.report.xbrl-tagged` | `{report_id, taxonomy, elements_tagged, validation_errors}` | XBRL tagging completed | Validation Engine, Author |
| `finance.report.xbrl-validated` | `{report_id, taxonomy, errors[], warnings[], filing_ready}` | XBRL validation completes | Author, Filing Team |
| `finance.report.submitted-for-review` | `{report_id, submitted_by, reviewer_ids[], due_date}` | Report submitted to approval workflow | Notification, Reviewers |
| `finance.report.published` | `{report_id, approved_by, approved_at, version}` | Report approved and locked | Audit Trail, Distribution |
| `finance.report.distributed` | `{report_id, distribution_id, recipients[], format, sent_at}` | Report distributed to recipients | Notification, Audit Trail |
| `finance.report.xbrl-filed` | `{report_id, filing_id, regulator, filing_status, confirmation}` | XBRL filing submitted to regulator | Audit Trail, Compliance |
| `finance.report.annotation.added` | `{annotation_id, report_id, section, author, text, thread_id}` | Collaborative annotation added | Reviewers, Author |
| `finance.report.burst.completed` | `{burst_id, report_id, burst_count, entity_count, distribution_count}` | Report bursting finishes | Distribution, Dashboard |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `finance.period.closed` | Finance Service | Trigger period-end report generation |
| `finance.consolidation.completed` | Finance Service | Refresh consolidated report data |
| `finance.revenue.recognized` | Finance Service | Update revenue data in reports |
| `workflow.step.approved` | Workflow Service | Update report approval status |
| `platform.scheduler.job.started` | Platform Service | Execute scheduled report generation and distribution |
| `report.ml.inference.complete` | Report Service | Apply anomaly insights to report commentary suggestions |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `report_template` | `id, name, type, taxonomy_ref, layout_config{}, sections[], version, locked_sections[]` | Has many `report_instance` |
| `report_instance` | `id, template_id, entity_ids[], period_start, period_end, status, version, created_by` | Has many `report_section`, `report_approval` |
| `report_section` | `id, report_id, section_type, data_binding{}, content, xbrl_tags{}, sort_order` | Belongs to `report_instance` |
| `report_xbrl_mapping` | `id, template_id, section_id, cell_ref, taxonomy_element, context{}, dimensions{}` | Belongs to `report_template` |
| `report_approval` | `id, report_id, version, submitted_by, submitted_at, approver_id, status, comments, approved_at` | Belongs to `report_instance` |
| `report_distribution` | `id, report_id, recipients[], format, schedule, burst_config{}, last_distributed_at` | Belongs to `report_instance` |
| `report_annotation` | `id, report_id, section_id, author_id, parent_annotation_id, text, status, created_at` | Belongs to `report_section`, threaded |
| `report_xbrl_filing` | `id, report_id, taxonomy, instance_document_ref, regulator, filing_status, confirmation_ref, filed_at` | Belongs to `report_instance` |
| `report_burst_config` | `id, report_id, burst_by (entity/department/segment), template_overrides{}, distribution_rules[]` | Belongs to `report_instance` |

## Cross-References

- [Corporate Performance Management](../06-services/report.md) — Performance reporting and KPI management (CPM module)
- [Augmented Analytics](../06-services/report.md) — Natural language queries and auto-insights for financial data (Augmented Analytics module)
- [Revenue Recognition](../06-services/finance.md) — Revenue data feeding financial reports (Revenue Recognition module)
- [Report Service](../06-services/report.md) — Report builder, analytics, and distribution engine
- [Finance Service](../06-services/finance.md) — Financial data, consolidation, and period management
