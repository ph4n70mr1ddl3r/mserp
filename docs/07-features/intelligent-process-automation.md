# Intelligent Process Automation (IPA)

Combines RPA bots with AI/ML for cognitive automation, intelligent document processing, and self-learning task automation, managed by the Platform Service (RPA engine) + Report Service (ML inference) + Integration Service (connector automation).

## Overview

Intelligent Process Automation (IPA) goes beyond traditional rule-based RPA by infusing AI/ML capabilities into every stage of the automation lifecycle. Bots can read unstructured documents, classify exceptions using ML, learn from operator corrections to improve over time, and orchestrate complex workflows that require human judgment at decision points. The system provides process discovery analytics that identify automation candidates from user behavior data, a bot orchestration engine supporting both attended (human-triggered) and unattended (scheduled/event-driven) automation, and ROI dashboards that quantify automation value in terms of time saved, error reduction, and cost avoidance.

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Cognitive RPA (AI-Enhanced Bots)** | Platform Service (Rust) + Report Service (ONNX) | Bots with embedded ML capabilities: visual recognition (screen element detection), natural language understanding (email/invoice parsing), decision trees with ML classification nodes; bots call Report Service for inference at runtime |
| **Self-Learning Bots** | Platform Service + Report Service | Bots that capture operator corrections during attended sessions; corrections fed to retraining pipeline; model updates deployed without bot downtime; confidence thresholds trigger learning mode vs. auto-execute mode |
| **Intelligent Document Processing** | Platform Service + Report Service | Integration with IDP for end-to-end document automation: bot triggers document ingestion, IDP classifies and extracts, bot processes extracted data into target systems; handles invoices, POs, contracts, claims |
| **Attended Automation** | Platform Service (Rust) | Human-triggered bots that run on the user's desktop; side-by-side operation with human override; recording mode captures steps for bot creation; hotkey and context-menu triggers |
| **Unattended Automation** | Platform Service (Rust) | Headless bots running on schedule, event trigger, or API call; queue-based workload distribution across bot workers; priority scheduling and SLA-aware execution; retry and checkpoint logic |
| **Process Discovery** | Report Service (ONNX) | User behavior analytics: captures UI interaction patterns, identifies repetitive task sequences, ranks automation candidates by ROI (frequency × duration × error rate × complexity) |
| **Bot Orchestration (Human-in-the-Loop)** | Platform Service + Platform Workflow | Orchestration engine manages bot-to-human handoffs: bot pauses at decision points, routes to human reviewer, resumes on approval; supports parallel bot execution with merge logic |
| **Exception Handling (ML Classification)** | Platform Service + Report Service | ML-based exception classifier: analyzes exception context (error type, data state, system response) and routes to resolution path (retry, skip, escalate, self-heal); learns from resolution outcomes |
| **Automation ROI Dashboards** | Report Service + Platform Service | Real-time ROI tracking: time saved (bot duration vs. manual benchmark), error reduction rate, cost avoidance (FTE equivalent), bot utilization rate, exception rate trend, ROI by process |

## IPA Pipeline

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Intelligent Process Automation Pipeline                │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │  Phase 1: Discover                                                  │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐     │ │
│  │  │ User Behavior│  │ Process Map  │  │ ROI Ranking          │     │ │
│  │  │ Analytics    │──▶│ Generation   │──▶│ (Automation         │     │ │
│  │  │ Capture      │  │ (Auto)       │  │  Candidates)         │     │ │
│  │  └──────────────┘  └──────────────┘  └──────────────────────┘     │ │
│  └────────────────────────────┬───────────────────────────────────────┘ │
│                               │                                          │
│  ┌────────────────────────────▼───────────────────────────────────────┐ │
│  │  Phase 2: Design & Build                                            │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐     │ │
│  │  │ Bot Designer │  │ ML Node      │  │ Test &               │     │ │
│  │  │ (Visual/Code)│──▶│ Integration  │──▶│ Validate             │     │ │
│  │  │              │  │ (Cognitive)  │  │                      │     │ │
│  │  └──────────────┘  └──────────────┘  └──────────────────────┘     │ │
│  └────────────────────────────┬───────────────────────────────────────┘ │
│                               │                                          │
│  ┌────────────────────────────▼───────────────────────────────────────┐ │
│  │  Phase 3: Deploy & Orchestrate                                      │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐     │ │
│  │  │ Attended     │  │ Unattended   │  │ Human-in-the-Loop    │     │ │
│  │  │ Bots         │  │ Bot Workers  │  │ Decision Points      │     │ │
│  │  └──────┬───────┘  └──────┬───────┘  └──────────┬───────────┘     │ │
│  └─────────┼─────────────────┼──────────────────────┼─────────────────┘ │
│            │                 │                      │                    │
│  ┌─────────▼─────────────────▼──────────────────────▼─────────────────┐ │
│  │  Phase 4: Monitor & Learn                                           │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐     │ │
│  │  │ ROI          │  │ Exception    │  │ Self-Learning        │     │ │
│  │  │ Dashboard    │  │ Classifier   │  │ Feedback Loop        │     │ │
│  │  └──────────────┘  └──────────────┘  └──────────────────────┘     │ │
│  └────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
```

## Bot Types

| Bot Type | Execution Mode | Trigger | Use Cases | ML Integration |
|----------|---------------|---------|-----------|----------------|
| **Attended Desktop** | User's machine, side-by-side | Hotkey, context menu, manual | Data entry assistance, form filling, copy-paste automation | Visual recognition, NLP for field mapping |
| **Unattended Headless** | Server worker, no UI | Schedule, event, API call | Batch processing, report generation, data migration, reconciliation | Document classification, data extraction |
| **Hybrid** | Starts attended, continues unattended | User initiates, bot completes | Complex workflows requiring initial human context | Exception classification, human handoff |
| **Document Processing** | Unattended + IDP pipeline | Document arrival (email, upload, folder watch) | Invoice processing, PO matching, claims adjudication | OCR, classification, extraction, validation |
| **Orchestration** | Bot coordinator | Workflow event | Multi-bot coordination, human-in-the-loop routing | Priority classification, SLA prediction |
| **Self-Learning** | Attended (learning) → Unattended (auto) | Initially manual, then automatic | Processes that improve with feedback (data entry, categorization) | Continuous learning from corrections |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| RPA Engine | Platform Service | Internal API | Bot execution runtime, scheduling, queue management |
| IDP Service | Platform Service / Report Service | Internal API | Document ingestion, classification, extraction pipeline |
| ML Inference | Report Service | gRPC | Real-time ML inference for cognitive bot capabilities |
| Integration Connectors | Integration Service | Internal API | Pre-built connectors for ERP screens, web apps, email, file systems |
| Workflow Engine | Platform Service | Event-driven | Human-in-the-loop task routing, approval workflows, escalation |
| Finance Service | Finance Service | Internal API | Journal entry bots, reconciliation bots, invoice processing bots |
| Commerce Service | Commerce Service | Internal API | Order entry bots, pricing update bots, vendor master bots |
| HCM Service | HCM Service | Internal API | Employee onboarding bots, payroll data entry bots, timesheet bots |
| Notification Service | Platform Service | Event-driven | Bot status alerts, exception notifications, completion summaries |
| Audit Trail | Platform Service | Event-driven | Full bot action logging, before/after snapshots, compliance audit |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `platform.ipa.bot.started` | `{bot_id, execution_id, bot_type, triggered_by, process_name}` | Bot execution begins | Dashboard, Audit Trail |
| `platform.ipa.bot.completed` | `{bot_id, execution_id, status, duration_ms, records_processed, errors[]}` | Bot execution finishes | ROI Dashboard, Audit Trail |
| `platform.ipa.bot.exception` | `{bot_id, execution_id, exception_type, context{}, ml_classification}` | Bot encounters exception | Exception Classifier, Notification |
| `platform.ipa.bot.human-handoff` | `{bot_id, execution_id, handoff_reason, assigned_to, context{}}` | Bot requires human decision | Workflow Engine, Notification |
| `platform.ipa.bot.learning-feedback` | `{bot_id, execution_id, correction_type, before, after, field}` | Operator corrects bot action | ML Retraining Pipeline |
| `platform.ipa.process-discovery.candidate` | `{candidate_id, process_name, frequency, avg_duration, roi_score}` | Process discovery identifies candidate | Dashboard, Automation Manager |
| `platform.ipa.exception.classified` | `{exception_id, classification, confidence, resolution_path}` | ML classifies an exception | Orchestration Engine, Dashboard |
| `platform.ipa.roi.snapshot` | `{snapshot_id, period, total_time_saved_hours, error_reduction_pct, cost_avoidance}` | ROI calculation runs | ROI Dashboard, Management Reports |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `finance.invoice.created` | Finance Service | Trigger invoice processing bot |
| `finance.reconciliation.exception.created` | Finance Service | Trigger reconciliation bot |
| `hr.employee.hired` | HCM Service | Trigger onboarding automation bot |
| `commerce.order.created` | Commerce Service | Trigger order fulfillment bot |
| `platform.idp.document.classified` | Platform Service | Trigger document processing bot with extraction results |
| `workflow.step.approved` | Workflow Service | Resume paused bot after human completes decision task |
| `integration.connector.available` | Integration Service | Enable bot connector for target system |
| `report.ml.inference.complete` | Report Service | Apply ML classification to bot decision nodes |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `ipa_bot` | `id, name, type, process_name, status, version, schedule_config, ml_enabled` | Has many `ipa_bot_execution` |
| `ipa_bot_execution` | `id, bot_id, triggered_by, started_at, completed_at, status, records_processed, duration_ms` | Has many `ipa_bot_action`, `ipa_bot_exception` |
| `ipa_bot_action` | `id, execution_id, step, action_type, target_system, target_object, before_snapshot, after_snapshot` | Belongs to `ipa_bot_execution` |
| `ipa_bot_exception` | `id, execution_id, step, exception_type, context{}, ml_classification, resolution_path, resolved_by` | Belongs to `ipa_bot_execution` |
| `ipa_process_candidate` | `id, process_name, discovery_method, frequency, avg_duration_sec, roi_score, automation_complexity` | Process discovery result |
| `ipa_learning_feedback` | `id, bot_id, execution_id, correction_type, field, before_value, after_value, operator_id` | Belongs to `ipa_bot_execution` |
| `ipa_roi_metric` | `id, bot_id, period, executions, time_saved_hours, errors_prevented, cost_avoidance, utilization_pct` | Belongs to `ipa_bot` |
| `ipa_queue_item` | `id, queue_name, bot_id, priority, payload{}, status, scheduled_at, started_at, completed_at` | Work queue for unattended bots |

## Automation ROI Metrics

| Metric | Calculation | Target | Reporting Frequency |
|--------|-------------|--------|-------------------|
| Time Saved (hours) | `(manual_duration - bot_duration) × executions` | Track and report | Daily |
| Error Reduction Rate | `1 - (bot_error_rate / manual_error_rate) × 100` | > 90% reduction | Weekly |
| Cost Avoidance (FTE) | `time_saved_hours / annual_fte_hours × avg_fte_cost` | Track and report | Monthly |
| Bot Utilization Rate | `active_execution_time / available_time × 100` | > 80% | Daily |
| Exception Rate | `exceptions / total_executions × 100` | < 5% | Daily |
| Self-Learning Improvement | `correction_rate(current) / correction_rate(initial)` | > 50% improvement | Monthly |
| Mean Time to Resolution | `avg(exception_resolved_at - exception_created_at)` | < 4 hours | Weekly |
| Process Coverage | `automated_processes / eligible_processes × 100` | > 60% | Quarterly |

## Exception Classification Paths

| Exception Type | ML Classification | Auto-Resolution | Escalation Path | SLA |
|---------------|-------------------|----------------|-----------------|-----|
| Data Validation Error | Field-level rule violation | Retry with corrected format | Data steward | 2 hours |
| System Unavailable | Connection timeout, auth failure | Retry with exponential backoff | IT support | 1 hour |
| Missing Reference Data | Lookup value not found | Queue for manual enrichment | Process owner | 4 hours |
| Business Rule Violation | Threshold or policy breach | Flag and hold for review | Business approver | 8 hours |
| Document Extraction Failure | OCR confidence < threshold | Route to IDP re-processing | Document specialist | 4 hours |
| Ambiguous Match | Multiple candidates, score < threshold | Present options to operator | Human-in-the-loop | 2 hours |
| Process Change Detected | UI element not found, workflow deviation | Switch to learning mode | RPA developer | 24 hours |

## Cross-References

- [RPA](../06-services/platform.md) — Core RPA engine and bot runtime (RPA module)
- [Intelligent Document Processing](idp.md) — Document classification and data extraction
- [Adaptive Intelligence](adaptive-intelligence.md) — ML model training and inference pipeline
- [Process Mining](../06-services/report.md) — Process discovery and optimization analytics (Process Mining module)
- [Platform Service](../06-services/platform.md) — Bot execution runtime and workflow engine
- [Report Service](../06-services/report.md) — ML inference and ROI analytics
