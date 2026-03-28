# Intelligent Close

AI-driven financial close automation with anomaly detection, predictive analytics, and continuous close capabilities, managed by the Finance Service with ML models from the Report Service.

## Overview

Intelligent Close transforms the traditional batch-oriented financial close into an AI-assisted, continuous process. It leverages machine learning to detect anomalies in real-time as transactions post, auto-assigns close tasks based on historical patterns, predicts close bottlenecks before they occur, and provides a live dashboard of close progress. The system enables a shift from reactive period-end scrambling to proactive, continuous close certification.

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Close Automation Engine** | Finance Service (Rust) | Orchestrates the entire close process: task creation, dependency management, parallel execution, and certification |
| **Anomaly Detection Rules** | Finance Service + Report Service (ONNX) | Configurable anomaly rules (balance threshold, variance percentage, missing entries) plus ML-based outlier detection |
| **Auto-Reconciliation with ML** | Finance Service + Report Service | Integrates with Account Reconciliation to auto-match transactions and flag exceptions; ML models improve over time |
| **Continuous Close Architecture** | Finance Service | Real-time period-end processing that continuously reconciles and certifies as transactions post, rather than batch close |
| **Close Dashboard Components** | Report Service + Platform Service | Real-time dashboard with close progress, task status, bottleneck identification, and predictive ETA |
| **Predictive Close Analytics** | Report Service (ONNX) | ML models predict close completion time, identify at-risk tasks, and suggest task reassignment |
| **Close Task Prioritization** | Finance Service | AI-driven task prioritization based on criticality path, dependencies, and historical duration |
| **Period-End Checklist Automation** | Finance Service + Workflow Engine | Automated checklist generation from templates with task assignment, dependencies, and deadline tracking |

## Close Process Flow

```
┌──────────────────────────────────────────────────────────────────┐
│                    Period-End Close Process                       │
│                                                                   │
│  ┌─────────┐    ┌──────────┐    ┌──────────┐    ┌─────────────┐ │
│  │ Period   │───▶│ Pre-Close│───▶│ Execute  │───▶│ Post-Close  │ │
│  │ Trigger  │    │ Checks   │    │ Tasks    │    │ Review      │ │
│  └─────────┘    └──────────┘    └──────────┘    └─────────────┘ │
│       │              │               │                 │          │
│       │         ┌────▼────┐    ┌────▼────┐     ┌────▼────┐     │
│       │         │ Anomaly │    │ Reconcil│     │ Final   │     │
│       │         │ Scan    │    │ + Match │     │ Certify │     │
│       │         └─────────┘    └─────────┘     └─────────┘     │
│       │              │               │                 │          │
│       ▼              ▼               ▼                 ▼          │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              Continuous Close Monitor                     │   │
│  │  ┌────────────┐  ┌────────────┐  ┌──────────────────┐   │   │
│  │  │ Anomaly    │  │ Reconcil.  │  │ Predictive ETA   │   │   │
│  │  │ Detection  │  │ Progress   │  │ & Risk Score     │   │   │
│  │  └────────────┘  └────────────┘  └──────────────────┘   │   │
│  └──────────────────────────────────────────────────────────┘   │
│                              │                                   │
│                              ▼                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              Close Dashboard (Real-Time)                  │   │
│  │  Progress Bar │ Task List │ Anomalies │ ETA │ Bottlenecks│   │
│  └──────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────┘
```

## Dashboard Components

| Component | Data Source | Refresh Rate | Description |
|-----------|-----------|--------------|-------------|
| Close Progress Bar | Finance Service | Real-time | Overall % complete across all close tasks |
| Task Status Grid | Finance Service | Real-time | Per-task status: pending, in-progress, blocked, complete |
| Anomaly Alert Feed | Report Service | Real-time | Live anomaly alerts with severity and suggested actions |
| Reconciliation Status | Finance Service | On match event | Auto-match rate, exception count, resolution status |
| Predictive ETA | Report Service (ML) | 5 minutes | Predicted close completion time with confidence interval |
| Bottleneck Identifier | Finance Service | 5 minutes | Tasks blocking the critical path with duration prediction |
| Period Comparison | Report Service | On demand | Current vs. prior period close metrics (duration, exceptions) |
| Risk Heat Map | Report Service (ML) | 5 minutes | At-risk tasks color-coded by predicted delay probability |

## Anomaly Detection Rules

| Rule Type | Detection Method | Example | Action |
|-----------|-----------------|---------|--------|
| Balance Threshold | Static rule | Account balance exceeds configurable threshold | Alert + hold for review |
| Variance Analysis | Statistical | Balance deviates > N standard deviations from historical | Alert with explanation |
| Missing Entries | Pattern detection | Expected recurring entries not present | Alert + create follow-up task |
| Unusual Activity | ML classification | Transaction pattern classified as anomalous (score > threshold) | Alert + quarantine |
| Intercompany Imbalance | Rule-based | IC entries don't net to zero across entities | Block close + create exception |
| Subledger Mismatch | Reconciliation | Subledger total ≠ GL control account | Block close + create exception |
| Zero-Balance Alert | Rule-based | Account with expected activity shows zero balance | Alert for investigation |
| Duplicate Detection | ML + rule | Potential duplicate journal entries detected | Alert + hold for review |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| General Ledger | Finance Service | Internal API | GL balances, journal entries for close processing |
| Account Reconciliation | Finance Service | Internal API | Reconciliation status feeds into close progress |
| Subledgers (AP/AR/FA) | Finance Service | Internal API | Subledger balance verification |
| Intercompany | Finance Service | Internal API | IC elimination verification |
| Workflow Engine | Platform Service | Event-driven | Close task routing, approval workflows, escalations |
| Notification Service | Platform Service | Event-driven | Close task assignments, anomaly alerts, deadline reminders |
| Report Service | Report Service | gRPC | ML inference (anomaly detection, ETA prediction, risk scoring) |
| Dashboard | Report Service | WebSocket | Real-time close progress updates to UI |
| Audit Trail | Platform Service | Event-driven | Full audit log of close actions and certifications |
| Period Management | Finance Service | Internal API | Period open/close state, soft/hard close |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `close.period.initiated` | `{period_id, close_type, initiated_by}` | Period close initiated | All close participants, Dashboard |
| `close.task.created` | `{task_id, period_id, task_type, assignee, due_date}` | Close task auto-generated | Notification Service, Dashboard |
| `close.anomaly.detected` | `{anomaly_id, period_id, account_id, rule_type, severity, details}` | Anomaly detection rule fires | Notification Service, Dashboard, Workflow |
| `close.task.completed` | `{task_id, completed_by, completed_at, notes}` | User completes close task | Close Engine, Dashboard |
| `close.bottleneck.identified` | `{period_id, task_ids, predicted_delay_hours}` | ML model predicts delay | Dashboard, Notification Service |
| `close.period.certified` | `{period_id, certified_by, certified_at, close_duration}` | All close tasks complete and certified | Audit Trail, Report Service, Dashboard |
| `close.reconciliation.status` | `{period_id, auto_match_rate, exceptions_open, exceptions_resolved}` | Reconciliation status update | Dashboard, Close Engine |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `finance.transaction.posted` | Finance Service | Trigger real-time anomaly check |
| `reconciliation.match.completed` | Finance Service | Update close reconciliation progress |
| `reconciliation.exception.created` | Finance Service | Create close follow-up task |
| `reconciliation.close.certified` | Finance Service | Mark reconciliation task complete |
| `platform.workflow.step-completed` | Platform Service | Update close task status |
| `report.ml.inference.complete` | Report Service | Apply anomaly scores, update ETA predictions |
| `finance.period.closing` | Finance Service | Initiate close process |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `close_period` | `id, period_id, close_type, status, initiated_at, certified_at, certified_by` | Has many `close_task`, `close_anomaly` |
| `close_task` | `id, period_id, task_type, assignee, status, priority, due_date, dependency_ids, completed_at` | Belongs to `close_period` |
| `close_anomaly` | `id, period_id, account_id, rule_type, severity, status, resolution, resolved_by` | Belongs to `close_period` |
| `close_template` | `id, name, task_definitions, dependency_graph, auto_assign_rules` | Template for `close_period` |
| `close_metric` | `id, period_id, metric_type, value, recorded_at` | Belongs to `close_period` |

## Cross-References

- [Account Reconciliation](reconciliation.md) — Auto-matching and reconciliation features
- [Adaptive Intelligence](adaptive-intelligence.md) — ML model training and inference pipeline
- [Workflow Engine](workflow.md) — Close task workflow definitions and escalation rules
- [Revenue Recognition](revenue-recognition.md) — Revenue recognition close tasks
- [Finance Service](../services/finance.md) — Period management and GL operations
- [Report Service](../services/report.md) — Dashboard and ML inference
- [NFR: Performance](../planning/nfr.md) — Anomaly detection < 30s per period scan
