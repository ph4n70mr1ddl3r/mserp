# Account Reconciliation

Automated financial reconciliation with intelligent matching, exception management, and close task orchestration, managed by the Finance Service.

## Overview

Account Reconciliation provides end-to-end automation of the financial reconciliation process, from bank statement import through matching, exception resolution, and close certification. The system combines rule-based matching engines with ML-assisted confidence scoring to achieve high auto-match rates while maintaining auditability. It supports all reconciliation types — bank, intercompany, subledger-to-general-ledger, and balance sheet — within a unified framework integrated with the financial close process.

## Data Flow Diagram

```
┌──────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  Bank / ERP  │────▶│  Statement       │────▶│  Matching Engine     │
│  Statements  │     │  Import & Parse  │     │  ┌───────────────┐  │
└──────────────┘     └──────────────────┘     │  │ Rule Matcher  │  │
                                                │  │ (exact/fuzzy) │  │
┌──────────────┐     ┌──────────────────┐     │  └───────┬───────┘  │
│  Subledger   │────▶│  Transaction     │────▶│  ┌───────▼───────┐  │
│  Transactions│     │  Normalization   │     │  │ ML Scorer     │  │
└──────────────┘     └──────────────────┘     │  │ (confidence)  │  │
                                                │  └───────┬───────┘  │
                                                └──────────┼──────────┘
                                                           │
                              ┌────────────────────────────┼───────────────┐
                              │                            │               │
                     ┌────────▼────────┐        ┌─────────▼──────┐  ┌────▼─────┐
                     │  Auto-Matched   │        │  Exceptions    │  │  Unmatched│
                     │  (high conf)    │        │  (low conf)    │  │  Queue    │
                     └────────┬────────┘        └─────────┬──────┘  └────┬─────┘
                              │                           │              │
                              ▼                           ▼              ▼
                     ┌────────────────────────────────────────────────────┐
                     │           Exception Management Workflow            │
                     │  Assign → Investigate → Resolve → Approve → Cert  │
                     └─────────────────────────┬──────────────────────────┘
                                               │
                                               ▼
                     ┌────────────────────────────────────────────────────┐
                     │           Close Task Management                    │
                     │  Reconciliation status → Close checklist → Sign-off│
                     └────────────────────────────────────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Auto-Matching Rules Engine** | Finance Service (Rust) | Configurable multi-criteria matching with support for exact, fuzzy, and ML-assisted matching algorithms |
| **Reconciliation Templates** | Finance Service + Platform Service | Template definitions per account type with customizable matching rules, tolerance thresholds, and approval workflows |
| **Exception Management** | Finance Service + Workflow Engine | Workflow-driven resolution with assignment rules, escalation timers, and SLA tracking |
| **Close Task Management** | Finance Service + Platform Service | Hierarchical close checklist with task dependencies, deadline tracking, and certification workflows |
| **Intercompany Reconciliation** | Finance Service | Automated matching of intercompany transactions across business units with elimination validation |
| **Bank Reconciliation** | Finance Service + Integration Service | Bank statement import (OFX, CSV, ISO 20022, API), automatic matching, and reconciliation reporting |
| **Confidence Scoring** | Report Service (ONNX) | ML model assigns confidence scores (0-100) to each match; low-confidence items routed to exception queue |
| **Storage** | PostgreSQL + MinIO | Transaction data in PostgreSQL; supporting documents (statements, receipts) in MinIO |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Matching Engine | Rule evaluation, fuzzy matching, ML scoring | Finance Service |
| Template Manager | CRUD for reconciliation templates and rules | Finance Service |
| Exception Queue | Prioritized exception management and routing | Finance Service |
| Close Task Manager | Close checklist orchestration with dependencies | Finance Service |
| Statement Importer | Multi-format bank statement parsing and normalization | Finance Service |
| Intercompany Matcher | Cross-entity transaction matching and elimination | Finance Service |
| Reconciliation Store | Persisted reconciliation state and audit trail | Finance Service |
| Confidence Model | ML inference for match confidence scoring | Report Service |

## Matching Algorithms

| Algorithm | Match Type | Use Case | Typical Accuracy |
|-----------|-----------|----------|-----------------|
| Exact Match | Amount + Reference + Date | Standard bank-to-ledger matching | 60-70% auto-match |
| Fuzzy Amount | Amount within tolerance ±threshold | Rounding differences, partial payments | +10-15% additional |
| Reference Match | Fuzzy reference string matching (Levenshtein) | Truncated or modified references | +5-10% additional |
| Many-to-One | Multiple transactions to single statement line | Batch payments, consolidated deposits | +5-8% additional |
| ML-Assisted | Feature-based classification with confidence | Complex patterns, historical learning | +5-10% additional |
| Intercompany | Source/dest entity pair matching with elimination | IC elimination, transfer pricing | 90%+ auto-match |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| General Ledger | Finance Service | Internal API | GL account balances for reconciliation |
| Accounts Payable | Finance Service | Internal API | Open AP items for payment matching |
| Accounts Receivable | Finance Service | Internal API | Open AR items for receipt matching |
| Cash Management | Finance Service | Internal API | Cash position and bank account data |
| Intercompany | Finance Service | Internal API | Cross-entity transaction data |
| Bank Feeds | Integration Service | Connector | Bank statement import via pre-built connectors |
| Workflow Engine | Platform Service | Event-driven | Exception routing, approval workflows |
| Notification Service | Platform Service | Event-driven | Task assignments, escalation alerts |
| Intelligent Close | Finance Service | Internal API | Close automation and anomaly detection |
| Report Service | Report Service | Internal API | Reconciliation reports, dashboards |
| Audit Trail | Platform Service | Event-driven | Full audit log of reconciliation actions |
| ML Inference | Report Service | gRPC | Confidence scoring, ML-assisted matching |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `finance.reconciliation.matched` | `{reconciliation_id, account_id, matched_count, unmatched_count}` | Auto-match run completes | Close Task Manager, Dashboard |
| `finance.reconciliation.exception.created` | `{exception_id, reconciliation_id, transaction_ids, reason}` | Unmatched or low-confidence match detected | Workflow Engine, Notification Service |
| `finance.reconciliation.exception.resolved` | `{exception_id, resolution, resolved_by, resolved_at}` | User resolves exception | Audit Trail, Close Task Manager |
| `finance.reconciliation.template.applied` | `{template_id, account_id, rules_applied}` | Template applied to reconciliation | Audit Trail |
| `finance.reconciliation.completed` | `{period_id, account_id, certified_by, certified_at}` | User certifies reconciliation complete | Close Task Manager, Report Service |
| `finance.reconciliation.intercompany.matched` | `{source_entity, dest_entity, matched_pairs, elimination_entries}` | Intercompany matching completes | Finance Service (elimination) |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `finance.transaction.posted` | Finance Service | New transaction available for matching |
| `finance.bank-statement.imported` | Finance Service | Trigger matching run for account |
| `finance.period.closing` | Finance Service | Initiate reconciliation checklist |
| `finance.intercompany.created` | Finance Service | Queue for intercompany matching |
| `platform.workflow.step-completed` | Platform Service | Update exception resolution status |
| `report.ml.inference.complete` | Report Service | Apply confidence scores to matches |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `reconciliation` | `id, account_id, period_id, status, auto_matched, manual_matched, exceptions_count` | Has many `reconciliation_match`, `reconciliation_exception` |
| `reconciliation_match` | `id, reconciliation_id, statement_line_id, transaction_id, match_type, confidence_score` | Belongs to `reconciliation`, `bank_statement_line`, `gl_transaction` |
| `reconciliation_exception` | `id, reconciliation_id, type, status, assigned_to, priority, resolution, resolved_by` | Belongs to `reconciliation` |
| `reconciliation_template` | `id, name, account_type, matching_rules, tolerance_amount, tolerance_percent` | Has many `matching_rule` |
| `matching_rule` | `id, template_id, sequence, criteria, algorithm, parameters` | Belongs to `reconciliation_template` |
| `close_task` | `id, period_id, account_id, task_type, assignee, status, due_date, completed_at` | Belongs to `period`, linked to `reconciliation` |

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `finance.reconciliation.match.tolerance_amount` | 0.01 | Amount tolerance for fuzzy matching |
| `finance.reconciliation.match.tolerance_percent` | 0.5% | Percentage tolerance for fuzzy matching |
| `finance.reconciliation.match.confidence_threshold` | 80 | Minimum confidence for auto-match (0-100) |
| `finance.reconciliation.match.max_rules_per_template` | 20 | Maximum matching rules per template |
| `finance.reconciliation.exception.escalation_hours` | 24 | Hours before exception escalation |
| `finance.reconciliation.close.auto_certify` | false | Auto-certify when all items matched |

## Cross-References

- [Intelligent Close](intelligent-close.md) — AI-driven close automation and anomaly detection
- [Adaptive Intelligence](adaptive-intelligence.md) — ML model training and inference pipeline
- [Workflow Engine](../06-services/workflow.md) — Exception management workflow definitions
- [Data Import/Export](../06-services/integration.md) — Statement import framework (Data Import/Export module)
- [Architecture Overview](../01-architecture/overview.md) — System communication patterns
- [Finance Service](../06-services/finance.md) — Finance service specification
- [NFR: Performance](../10-planning/nfr.md) — Reconciliation auto-match < 30s per account
