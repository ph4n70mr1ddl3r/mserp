# Innovation Management

Idea capture, evaluation, and pipeline management within PLM with stage-gate process, scoring models, patent tracking, and crowdsourced innovation campaigns, managed by the Manufacturing Service.

## Overview

Innovation Management provides a structured framework for capturing, evaluating, and advancing ideas from conception through implementation. The module supports idea submission from internal teams and external partners, multi-stage evaluation workflows with configurable scoring models, resource allocation for innovation projects, and patent tracking for intellectual property protection. It enables innovation portfolio visualization, trend analysis from competitive intelligence, crowdsourced innovation campaigns, idea-to-project conversion with full traceability, and stage-gate process management with approval workflows. The module operates within the PLM domain, linking innovations to product lifecycle data and manufacturing process improvements.

## Data Flow Diagram

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  Idea Submission  │────▶│  Categorization  │────▶│  Evaluation Queue   │
│  Portal           │     │  & Tagging       │     │  ┌───────────────┐  │
│  (Internal/       │     └──────────────────┘     │  │ Screen        │  │
│   Partner)        │                                │  │ Assess        │  │
└──────────────────┘     ┌──────────────────┐     │  │ Validate      │  │
                          │  Scoring Model   │────▶│  │ Approve       │  │
┌──────────────────┐     │  ┌────────────┐  │     │  └───────┬───────┘  │
│  Campaign        │────▶│  │ Criteria   │  │     └──────────┼──────────┘
│  Manager         │     │  │ Weights    │  │                │
└──────────────────┘     │  └────────────┘  │     ┌──────────▼──────────┐
                          └──────────────────┘     │  Stage-Gate Process  │
┌──────────────────┐     ┌──────────────────┐     │  ┌───────────────┐  │
│  Competitive     │────▶│  Trend Analysis  │────▶│  │ Gate Reviews  │  │
│  Intelligence    │     │  Engine          │     │  │ Approvals     │  │
└──────────────────┘     └──────────────────┘     │  └───────┬───────┘  │
                                                   └──────────┼──────────┘
                                                              │
┌──────────────────┐     ┌──────────────────┐     ┌──────────▼──────────┐
│  Approved Idea   │────▶│  Project         │────▶│  Innovation Project │
│                  │     │  Creation        │     │  ┌───────────────┐  │
└──────────────────┘     └──────────────────┘     │  │ Resources     │  │
                                                   │  │ Patent Filing │  │
┌──────────────────┐                              │  │ Portfolio     │  │
│  Patent Tracking │─────────────────────────────▶│  └───────────────┘  │
└──────────────────┘                              └─────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Idea Submission Portal** | Manufacturing Service (Rust) | Web-based submission interface for internal employees and external partners with configurable form fields, file attachments, and anonymity options |
| **Evaluation Workflow** | Manufacturing Service + Workflow Service | Multi-stage evaluation pipeline (screen, assess, validate, approve) with configurable gate criteria and reviewer assignment |
| **Scoring Engine** | Manufacturing Service | Configurable scoring models with weighted criteria, automated score computation, and threshold-based advancement |
| **Stage-Gate Process** | Manufacturing Service + Workflow Service | Formal stage-gate management with review scheduling, decision recording, and automatic stage advancement or rejection |
| **Campaign Manager** | Manufacturing Service | Time-boxed innovation campaigns with themes, participant management, submission tracking, and campaign analytics |
| **Patent Tracking** | Manufacturing Service | Patent filing status management with docket integration, deadline tracking, and IP portfolio visibility |
| **Portfolio View** | Manufacturing Service | Aggregated innovation pipeline visualization with stage distribution, resource allocation, and ROI projection |
| **Trend Analysis** | Manufacturing Service + Report Service | Competitive intelligence aggregation, idea clustering, trend identification, and opportunity scoring |
| **Storage** | PostgreSQL | All innovation management data in Manufacturing database; RLS-scoped per tenant |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Idea Submission Handler | Idea CRUD, attachment management, submission validation, duplicate detection | Manufacturing Service |
| Categorization Engine | Idea classification, tag management, auto-categorization hints | Manufacturing Service |
| Evaluation Pipeline | Stage progression, reviewer assignment, score aggregation, gate decisions | Manufacturing Service |
| Scoring Model Manager | Scoring model CRUD, criteria configuration, weight management, version control | Manufacturing Service |
| Stage-Gate Manager | Gate definition, review scheduling, decision capture, stage advancement | Manufacturing Service |
| Campaign Manager | Campaign lifecycle, participant management, submission tracking, analytics | Manufacturing Service |
| Project Converter | Idea-to-project conversion with resource allocation and timeline creation | Manufacturing Service |
| Patent Tracker | Filing status management, deadline alerts, IP portfolio tracking | Manufacturing Service |
| Portfolio Aggregator | Pipeline visualization, resource summary, ROI analysis, trend data | Manufacturing Service |
| Trend Analyzer | Competitive intelligence processing, idea clustering, opportunity scoring | Manufacturing Service |

## Stage-Gate Process

| Stage | Gate | Entry Criteria | Review Focus | Possible Decisions |
|-------|------|---------------|-------------|-------------------|
| Screening | Gate 1: Initial Screen | Complete submission form, valid category | Alignment with strategy, feasibility, duplication check | Advance, Reject, Merge with existing |
| Assessment | Gate 2: Technical Assessment | Passed screening, initial score above threshold | Technical feasibility, resource requirements, risk assessment | Advance, Reject, Request more info |
| Validation | Gate 3: Business Validation | Passed assessment, business case complete | Market potential, ROI projection, competitive landscape | Advance, Reject, Defer |
| Approval | Gate 4: Executive Approval | Passed validation, sponsor identified | Strategic fit, budget allocation, priority ranking | Approve, Reject, Defer, Conditionally approve |
| Development | Gate 5: Launch Review | Approved, project plan complete | Project readiness, team assignment, timeline confirmation | Launch project, Return to approval |

## Scoring Model Structure

| Component | Description | Example |
|-----------|-------------|---------|
| Criteria | Individual evaluation dimension | Technical feasibility, Market potential, Strategic alignment, Resource requirement, Sustainability impact |
| Weights | Relative importance per criterion (0.0–1.0, sum to 1.0) | Market potential: 0.30, Technical feasibility: 0.25, Strategic alignment: 0.20 |
| Scale | Numeric range per criterion (1–10) | 1 = Very low, 5 = Moderate, 10 = Very high |
| Threshold | Minimum weighted score to advance stage | Screening: 5.0, Assessment: 6.0, Validation: 7.0 |
| Model Version | Versioned scoring configuration for audit trail | v1.0 (2024-Q1), v1.1 (2024-Q2) |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| PLM / Product Lifecycle | Manufacturing Service | Internal API | Product data for idea context; innovation feeds product development |
| ECO / Change Management | Manufacturing Service | Event-driven | Approved ideas may trigger engineering change orders |
| Workflow Engine | Workflow Service | Event-driven | Stage-gate approvals, evaluation assignments, review scheduling |
| Product Catalog | Commerce Service | Event-driven | New product creation from approved innovation projects |
| ML/AI Inference | Report Service | Event-driven | Trend analysis, idea clustering, market predictions |
| Config Service | Config Service | Event-driven | Scoring model parameters, evaluation criteria, campaign settings |
| Notification Service | Platform Service | Event-driven | Submission confirmations, review assignments, gate decisions |
| Report Service | Report Service | Internal API | Innovation dashboards, portfolio analytics, trend reports |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `manufacturing.innovation.idea-submitted` | `{idea_id, submitter_id, category, title, campaign_id, submitted_at}` | New idea submitted via portal or campaign | Notification Service, Report Service |
| `manufacturing.innovation.idea-evaluated` | `{idea_id, evaluator_id, stage, score, criteria_scores, evaluated_at, recommendation}` | Evaluator completes scoring at a stage gate | Notification Service, Report Service |
| `manufacturing.innovation.idea-approved` | `{idea_id, approver_id, final_score, approved_stage, sponsor_id, approved_at}` | Idea passes final approval gate | Project Converter, Notification Service, Report Service |
| `manufacturing.innovation.project-created` | `{project_id, idea_id, name, sponsor_id, budget_allocated, team_size, start_date}` | Approved idea converted to innovation project | Report Service, Notification Service, Finance Service |
| `manufacturing.innovation.stage-gate.passed` | `{idea_id, gate_number, gate_name, decision, score, decided_by, decided_at}` | Idea passes a stage-gate review | Notification Service, Report Service, Workflow Service |
| `manufacturing.innovation.campaign-launched` | `{campaign_id, name, theme, start_date, end_date, target_participants, launched_at}` | Innovation campaign goes live | Notification Service, Report Service |
| `manufacturing.innovation.patent-filed` | `{patent_id, idea_id, filing_type, jurisdiction, filing_date, attorney_ref, status}` | Patent application filed for an idea | Report Service, Notification Service |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `manufacturing.eco.submitted` | Manufacturing Service | Correlate engineering changes with existing ideas; auto-link related innovations |
| `manufacturing.product-lifecycle.updated` | Manufacturing Service | Update idea context with product lifecycle stage changes; trigger re-evaluation |
| `commerce.product.created` | Commerce Service | Link innovation ideas to newly created products; close completed idea pipeline |
| `report.ml.inference.complete` | Report Service | Incorporate ML-driven trend analysis, market predictions, and idea clustering |
| `workflow.step.approved` | Workflow Service | Advance stage-gate decision; confirm evaluation assignment completion |
| `config.changed` | Config Service | Reload scoring model weights, evaluation thresholds, campaign parameters |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `innovation_ideas` | `id, tenant_id, title, description, category_id, submitter_id, status, current_stage, overall_score, campaign_id, source_type, is_anonymous` | Belongs to `idea_categories`, `innovation_campaigns`; has many `idea_evaluations`, `stage_gate_reviews` |
| `idea_categories` | `id, tenant_id, name, category_type, description, icon_ref, sort_order, is_active` | Has many `innovation_ideas` |
| `idea_evaluations` | `id, tenant_id, idea_id, evaluator_id, stage, criteria_scores, total_score, recommendation, comments, evaluated_at` | Belongs to `innovation_ideas` |
| `scoring_models` | `id, tenant_id, name, description, version, threshold_score, is_active, effective_from, effective_until` | Has many `scoring_criteria` |
| `scoring_criteria` | `id, tenant_id, model_id, criterion_name, weight, min_score, max_score, description, sort_order` | Belongs to `scoring_models` |
| `innovation_projects` | `id, tenant_id, idea_id, name, description, sponsor_id, status, budget_allocated, budget_used, start_date, target_date, completed_date, team_ids` | Belongs to `innovation_ideas` |
| `patent_filings` | `id, tenant_id, idea_id, project_id, filing_type, jurisdiction, application_number, filing_date, status, attorney_ref, next_deadline, notes` | Belongs to `innovation_ideas`, `innovation_projects` |
| `innovation_campaigns` | `id, tenant_id, name, theme, description, start_date, end_date, target_participants, status, owner_id, banner_ref` | Has many `idea_submissions`, `innovation_ideas` |
| `idea_submissions` | `id, tenant_id, idea_id, campaign_id, submitter_id, submitted_at, referral_source, is_valid` | Belongs to `innovation_campaigns`, `innovation_ideas` |
| `stage_gate_reviews` | `id, tenant_id, idea_id, gate_number, gate_name, scheduled_at, completed_at, decision, score, decided_by, attendees, notes, outcome_document_ref` | Belongs to `innovation_ideas` |

All tables include standard columns: `id`, `tenant_id`, `created_at`, `updated_at`, `created_by`, `updated_by`, `version`, `is_deleted`.

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `manufacturing.innovation.scoring.default_threshold` | 6.0 | Default minimum weighted score to advance |
| `manufacturing.innovation.evaluation.max_reviewers_per_stage` | 5 | Maximum assigned reviewers per evaluation stage |
| `manufacturing.innovation.evaluation.reminder_interval_hours` | 48 | Hours between evaluation reminder notifications |
| `manufacturing.innovation.evaluation.timeout_days` | 14 | Days before unreviewed evaluation is escalated |
| `manufacturing.innovation.campaign.max_active_campaigns` | 10 | Maximum concurrent active innovation campaigns |
| `manufacturing.innovation.idea.duplicate_similarity_threshold` | 0.85 | Cosine similarity threshold for duplicate detection |
| `manufacturing.innovation.patent.deadline_warning_days` | 30 | Days before patent deadline to trigger warning |
| `manufacturing.innovation.portfolio.auto_archive_after_days` | 365 | Days after last activity before auto-archiving rejected ideas |
| `manufacturing.innovation.submission.max_attachments` | 10 | Maximum file attachments per idea submission |
| `manufacturing.innovation.submission.max_attachment_size_mb` | 25 | Maximum size per attachment in megabytes |

## Cross-References

- [MES](mes.md) — Process improvement ideas linked to shop floor data
- [Production Scheduling](production-scheduling.md) — Innovation projects may alter production capabilities
- [Product Lifecycle Management](../06-services/manufacturing.md) — PLM integration for idea-to-product pipeline
- [ECO / Change Management](../06-services/manufacturing.md) — Approved ideas may initiate engineering changes
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
- [Manufacturing Service](../06-services/manufacturing.md) — Manufacturing service specification
- [NFR: Performance](../10-planning/nfr.md) — Idea submission < 500ms; scoring computation < 200ms
