# Subledger Accounting Engine

Centralized journal generation rules, configurable accounting transformations, multi-source subledger aggregation, accounting event model, journal entry templates, and business transaction-to-journal mapping, managed by the Finance Service.

## Overview

The Subledger Accounting Engine provides a configurable, rules-driven layer between business transactions and the general ledger. Rather than hard-coding journal entry logic into each business module (AP, AR, Revenue, Inventory, Payroll), the engine defines a centralized accounting event model where any business transaction produces an accounting event that is transformed into one or more journal entries through configurable rules and templates.

The engine supports multi-source subledger aggregation — each business module (Commerce, Manufacturing, HR, Finance) publishes accounting events that are consumed, validated, transformed, and posted through a unified pipeline. Journal generation rules define the mapping between source event types and journal entry templates, including account derivation logic, amount calculations, and dimension allocations. Accounting transformations enable complex scenarios like revenue recognition scheduling, multi-currency revaluation, and intercompany elimination entries. The result is a single, auditable path from business event to posted journal with full traceability.

## Data Flow Diagram

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  Commerce Events │────▶│  Accounting       │────▶│  Journal Generation │
│  (Order, Delivery)│    │  Event Pipeline   │     │  Rule Engine        │
└──────────────────┘     │  ┌─────────────┐ │     │  ┌───────────────┐  │
                         │  │ Validate    │ │     │  │ Rule Match    │  │
┌──────────────────┐     │  │ Enrich      │ │     │  │ Template      │  │
│  Finance Events  │────▶│  │ Dedup       │ │     │  │ Apply         │  │
│  (Invoice, Pay)  │     │  └─────────────┘ │     │  └───────┬───────┘  │
└──────────────────┘     └──────────────────┘              │          │
                                                           └──────────┼──┘
┌──────────────────┐     ┌──────────────────┐                         │
│  HR Events       │────▶│  Subledger       │◀────────────────────────┘
│  (Payroll)       │     │  Aggregation     │     ┌──────────────────────┐
└──────────────────┘     │  ┌─────────────┐ │     │  Accounting          │
                         │  │ Multi-      │ │     │  Transformations     │
┌──────────────────┐     │  │ Source      │ │     │  ┌───────────────┐   │
│  Config Changes  │────▶│  │ Merger     │ │     │  │ Rev Recog     │   │
│  (Rules Update)  │     │  └─────────────┘ │     │  │ FX Reval      │   │
└──────────────────┘     └──────────────────┘     │  │ IC Eliminate  │   │
                                                   │  └───────┬───────┘   │
                                                   └──────────┼──────────┘
                                                              │
                                                   ┌──────────▼──────────┐
                                                   │  General Ledger      │
                                                   │  Post & Validate     │
                                                   └─────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Accounting Event Model** | Finance Service (Rust) | Canonical event schema for all business transactions entering the accounting pipeline. Events carry source type, source ID, line-level details, amounts, dates, and reference data. Events are immutable once created |
| **Journal Generation Rules** | Finance Service (Rust) | Rule engine that matches incoming accounting events to journal templates based on source type, transaction category, and configurable conditions. Rules define debit/credit account derivation, amount extraction, and dimension mapping |
| **Accounting Transformations** | Finance Service (Rust) | Post-generation transformation layer for complex accounting scenarios: revenue recognition scheduling, multi-currency revaluation, intercompany elimination, and allocation rules. Transformations execute as pipeline stages |
| **Subledger Aggregation** | Finance Service (Rust) | Aggregation of accounting events from multiple business modules (Commerce, HR, Manufacturing) into unified subledger views per account, period, and dimension. Supports real-time and batch aggregation modes |
| **Journal Entry Templates** | Finance Service (Rust) | Versioned templates defining journal entry structure: line patterns, account placeholders, amount formulas, and dimension defaults. Templates support conditional lines and multi-line generation from single events |
| **Business Transaction Mapping** | Finance Service (Rust) | Configurable mapping registry linking business transaction types (order placed, invoice created, payroll processed) to accounting event types and journal generation rules. Mapping changes are versioned and auditable |
| **Validation Engine** | Finance Service (Rust) | Pre-posting validation: balanced entry check, account status verification, period openness check, dimension validation, and business rule compliance. Validation errors block posting with structured error details |
| **Storage** | PostgreSQL | All accounting rules, templates, events, and journal entries in Finance database. Immutable event log append-only. Journal entries versioned for audit |

## Business Rules

| ID | Rule | Description |
|----|------|-------------|
| BR-SLA-001 | Event Immutability | Accounting events are immutable once created. Corrections produce reversal events referencing the original event ID. The full event history is retained for the configured audit retention period (minimum 7 years) |
| BR-SLA-002 | Balanced Entry Requirement | Every generated journal entry must balance (total debits = total credits) within a configurable tolerance (default $0.01). Unbalanced entries are rejected and routed to the exception queue with full diagnostic detail |
| BR-SLA-003 | Period Control | Journal entries may only be posted to open accounting periods. Entries targeting closed periods are auto-routed to the earliest open period or held pending period reopening, depending on configuration |
| BR-SLA-004 | Account Derivation Fallback | Account derivation follows a priority chain: transaction-specific rule → category default → business unit default → company default. If no rule resolves an account, journal generation fails and an exception is raised |
| BR-SLA-005 | Multi-Currency Conversion | Accounting events in foreign currency are converted to the base currency using the exchange rate effective on the accounting date. Conversion uses the configured rate type (daily, monthly average, spot). Unrealized gains/losses are computed on revaluation |
| BR-SLA-006 | Revenue Recognition Scheduling | Revenue recognition transformations generate scheduled journal entries based on the configured recognition method (point-in-time, over-time, milestone-based). Scheduled entries are created in a deferred state and auto-posted on the recognition date |
| BR-SLA-007 | Intercompany Elimination | Accounting events tagged with intercompany indicators generate paired entries in both the source and destination entity ledgers. Consolidated reporting applies elimination rules to zero out intercompany balances |
| BR-SLA-008 | Dimension Allocation | Journal entry lines may carry up to 10 configurable analytical dimensions (cost center, department, project, product line, etc.). Allocation rules distribute amounts across dimensions using fixed percentages, statistical drivers, or derived formulas |
| BR-SLA-009 | Rule Versioning | Journal generation rules are versioned. Active rules are the latest published version. Historical rules are retained for audit. Rule changes require a new version; in-place modification of published rules is not permitted |
| BR-SLA-010 | Processing Idempotency | Each accounting event is processed exactly once. Duplicate event detection uses the event's unique ID with a 24-hour deduplication window. Redelivery attempts for already-processed events return the original journal entry reference |

## Data Model

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `sla_accounting_event_models` | `id, tenant_id, event_type, source_module, source_entity, version, schema_definition, is_active` | Referenced by `sla_journal_generation_rules` |
| `sla_journal_generation_rules` | `id, tenant_id, name, event_type, conditions, template_id, priority, effective_from, effective_to, version, is_active` | Belongs to `sla_accounting_event_models`, `sla_journal_templates` |
| `sla_accounting_transformations` | `id, tenant_id, name, transformation_type, input_schema, transformation_logic, output_schema, execution_order, is_active` | Applied as pipeline stages after journal generation |
| `sla_subledger_sources` | `id, tenant_id, source_module, source_type, connection_config, event_types, last_processed_event_id, status` | Registered sources for multi-source aggregation |
| `sla_journal_templates` | `id, tenant_id, name, line_patterns, account_derivation_rules, amount_formulas, dimension_defaults, version, is_active` | Used by `sla_journal_generation_rules` |

## Events

### Events Produced

| Event Type | Payload | Description |
|------------|---------|-------------|
| `finance.sla.accounting-event-created` | `{event_id, source_module, source_type, source_id, event_type, amount, currency, accounting_date, created_at}` | A new accounting event has been created from a business transaction and entered the processing pipeline |
| `finance.sla.journal-generated` | `{journal_id, event_id, entry_count, total_debits, total_credits, currency, generated_at}` | A journal entry has been generated from an accounting event through rule evaluation |
| `finance.sla.rule-executed` | `{rule_id, rule_version, event_id, journal_id, execution_time_ms, matched_conditions, executed_at}` | A journal generation rule has been evaluated and executed for an accounting event |
| `finance.sla.source-processed` | `{source_id, source_module, events_processed, journals_generated, errors, processed_at}` | A subledger source batch has been processed with summary statistics |

### Events Consumed

| Event Type | Source | Action |
|------------|--------|--------|
| `commerce.order.submitted` | Commerce Service | Create revenue recognition accounting event; generate deferred revenue journal entries |
| `commerce.delivery.completed` | Commerce Service | Create cost-of-goods-sold accounting event; generate inventory relief and COGS journal entries |
| `finance.invoice.created` | Finance Service | Create receivable or payable accounting event; generate AR/AP journal entries |
| `hr.payroll.processed` | HR Service | Create payroll expense accounting event; generate salary, tax, and benefit journal entries |
| `config.changed` | Config Service | Reload journal generation rules, templates, account derivation logic, and transformation definitions |

## Integration Points

| Integration | Service | Protocol | Description |
|-------------|---------|----------|-------------|
| General Ledger | Finance Service | Internal API | Journal posting, account master, period management, and GL reporting |
| Order Management | Commerce Service | Event-driven | Order and delivery events produce revenue and COGS accounting events |
| Accounts Payable | Finance Service | Internal API | Invoice and payment events produce AP and cash accounting events |
| Payroll | HR Service | Event-driven | Payroll processing events produce expense and liability accounting events |
| Platform Config | Config Service | Event-driven | Dynamic reload of accounting rules, templates, and transformation logic |

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `finance.sla.balanced_entry_tolerance` | 0.01 | Maximum debit-credit variance for balanced entry validation |
| `finance.sla.max_dimensions_per_line` | 10 | Maximum analytical dimensions per journal entry line |
| `finance.sla.rule_cache_ttl_seconds` | 300 | Seconds to cache active journal generation rules before refresh |
| `finance.sla.event_retention_years` | 7 | Minimum years to retain accounting event history |
| `finance.sla.batch_size` | 500 | Maximum events per subledger source processing batch |
| `finance.sla.dedup_window_hours` | 24 | Hours to retain processed event IDs for idempotency |

## Implementation Notes

- The accounting event pipeline is implemented as a staged processing model: ingestion → validation → rule matching → journal generation → transformation → posting. Each stage is independently retryable with its own error handling.
- Journal generation rule matching uses a priority-ordered evaluation: rules are sorted by priority (lower number = higher priority), and the first rule whose conditions match the event is selected. Conditions are expressed as JSON predicates evaluated against the event payload.
- Account derivation supports three modes: static (fixed account from template), lookup (query account mapping table by criteria), and formula (compute account from event attributes using an expression language). All three modes may be combined in a single template.
- Revenue recognition scheduling creates future-dated journal entries in a `deferred` state. A scheduled job runs daily to promote deferred entries to `posted` when their recognition date is reached. The job is idempotent — re-processing a date promotes only unprocessed entries.
- Multi-currency handling converts all foreign currency events to the base currency at event processing time using the configured rate type. Subsequent revaluation runs compute unrealized gains/losses on open balances and generate adjustment entries.
- Subledger source tracking maintains a high-water mark (`last_processed_event_id`) per source module. Batch processing resumes from the high-water mark, ensuring no events are missed during downtime. Event ordering is guaranteed by monotonic event IDs per source.

## Cross-References

- [Account Reconciliation](reconciliation.md) — Subledger-to-GL reconciliation and matching
- [Intelligent Close](intelligent-close.md) — Financial close automation consuming subledger data
- [Finance Service](../06-services/finance.md) — GL, AP, AR, and financial operations
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
- [NFR: Performance](../10-planning/nfr.md) — Journal generation < 500ms per event, batch processing > 1000 events/s
