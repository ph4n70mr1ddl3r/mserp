# Enterprise Data Quality Management

Comprehensive data profiling, cleansing, matching, enrichment, monitoring, and quality dashboards, managed by the Integration Service (data quality engine) with the Report Service (quality dashboards).

## Overview

Enterprise Data Quality Management provides a unified framework for ensuring data accuracy, completeness, consistency, and reliability across all MSERP domains. Rather than point-in-time data cleansing, it implements continuous data quality monitoring with real-time profiling, rule-based and ML-driven cleansing, probabilistic matching and deduplication, automated enrichment pipelines, and stewardship workflows for exception resolution. The system computes quality scorecards per domain and entity, triggers anomaly-based quality alerts when scores degrade, enforces data quality SLAs, and integrates directly with the MDM golden record engine to propagate corrections upstream and downstream.

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Data Profiling Engine** | Integration Service (Rust) | Column-level profiling (min/max, distinct count, null rate, pattern frequency, distribution stats) executed in streaming and batch modes; results stored as profiling snapshots for trend analysis |
| **Cleansing Rules Engine** | Integration Service (Rust) | Configurable rule library: standardization (trim, case, format), validation (regex, lookup, range), transformation (parse, map, enrich); rules compiled to optimized native code for high-throughput cleansing |
| **Matching & Deduplication** | Integration Service (Rust + Report Service ONNX) | Probabilistic matching using Fellegi-Sunter model with configurable match rules, weight thresholds, and blocking strategies; ML-enhanced matching for ambiguous cases; survivorship rules for golden record selection |
| **Enrichment Pipelines** | Integration Service | Declarative enrichment pipelines: address standardization (postal API), company verification (D&B/external), geocoding, industry classification; extensible enrichment provider framework |
| **Quality Scorecards** | Report Service + Integration Service | Per-entity quality scores across six dimensions (completeness, accuracy, consistency, timeliness, uniqueness, validity); aggregated domain-level and enterprise-level scorecards with trend charts |
| **Anomaly-Based Quality Alerts** | Integration Service + Report Service | Statistical monitoring of quality metrics; alerts when score degrades beyond configurable thresholds; root-cause analysis linking alerts to specific records and rules |
| **Stewardship Workflows** | Integration Service + Workflow Service | Exception queue for manual review of matching conflicts, cleansing failures, and enrichment errors; task assignment, escalation, and resolution tracking with full audit trail |
| **Data Quality SLAs** | Integration Service | Configurable SLA targets per entity and dimension; SLA breach detection with automated notifications; SLA compliance reporting for data governance |
| **MDM Golden Record Integration** | Integration Service | Bidirectional sync with MDM: quality scores feed golden record confidence; cleansing and matching results update golden records; golden record changes trigger re-profiling |

## Data Quality Rule Engine

| Rule Category | Rule Types | Configuration | Example |
|--------------|-----------|---------------|---------|
| **Standardization** | Trim, case conversion, format normalization | Regex patterns, format templates | Standardize phone to `+1 (XXX) XXX-XXXX` |
| **Validation** | Regex, lookup, range, referential integrity | Pattern library, reference data sets | Validate email against RFC 5322 pattern |
| **Transformation** | Parse, map, split, concatenate, derive | Mapping tables, expression language | Parse full name into first, middle, last |
| **Matching** | Exact, fuzzy (Levenshtein, Jaro-Winkler), phonetic, ML | Match rules, weight sets, thresholds | Match customers by name+address (fuzzy) + email (exact) |
| **Enrichment** | External lookup, derived field, calculated field | Provider config, API credentials | Enrich with D&B company rating via API |
| **Business Rule** | Cross-field validation, conditional rules, aggregate checks | Expression language, rule templates | Ensure `revenue >= COGS` for all product records |

## Quality Dimensions

| Dimension | Definition | Measurement | Target |
|-----------|-----------|-------------|--------|
| **Completeness** | Percentage of required fields populated | `non-null required fields / total required fields * 100` | > 98% |
| **Accuracy** | Percentage of values matching verified source | `verified values / total verified values * 100` | > 95% |
| **Consistency** | Percentage of records consistent across systems | `consistent records / total cross-checked records * 100` | > 97% |
| **Timeliness** | Percentage of records updated within SLA window | `records within SLA / total records * 100` | > 99% |
| **Uniqueness** | Percentage of records that are non-duplicate | `unique records / total records * 100` | > 99.5% |
| **Validity** | Percentage of values conforming to defined formats/rules | `valid values / total values * 100` | > 98% |

## Data Profiling Pipeline

```
┌────────────────────────────────────────────────────────────────────┐
│                     Data Profiling Pipeline                         │
│                                                                     │
│  ┌───────────┐    ┌───────────┐    ┌──────────┐    ┌───────────┐  │
│  │ Source    │───▶│ Column    │───▶│ Pattern  │───▶│ Statistic │  │
│  │ Ingest   │    │ Sample    │    │ Detect   │    │ Compute   │  │
│  └───────────┘    └───────────┘    └──────────┘    └───────────┘  │
│       │                │                │                │          │
│       │           ┌────▼────┐    ┌────▼────┐    ┌────▼────┐      │
│       │           │ Null %  │    │ Top N   │    │ Min/Max │      │
│       │           │ Analysis│    │ Values  │    │ Avg/Med │      │
│       │           └─────────┘    └─────────┘    └─────────┘      │
│       │                │                │                │          │
│       ▼                ▼                ▼                ▼          │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │              Profiling Results Store                          │ │
│  │  ┌────────────┐  ┌────────────┐  ┌──────────────────────┐   │ │
│  │  │ Column     │  │ Quality    │  │ Trend History        │   │ │
│  │  │ Profiles   │  │ Scores     │  │ (snapshot compare)   │   │ │
│  │  └────────────┘  └────────────┘  └──────────────────────┘   │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                              │                                      │
│                              ▼                                      │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │              Quality Dashboard (Report Service)               │ │
│  │  Scorecards │ Trend Charts │ Alerts │ Stewardship Queue      │ │
│  └──────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────────┘
```

## Matching Strategies

| Strategy | Algorithm | Best For | Blocking Key | Threshold |
|----------|----------|----------|-------------|-----------|
| **Deterministic** | Exact field comparison | High-confidence matches (ID, email, tax ID) | Exact field value | 100% |
| **Probabilistic** | Fellegi-Sunter weighted scoring | Name/address matching with variations | Phonetic (Soundex/Metaphone) + ZIP | Configurable (default 85%) |
| **ML-Enhanced** | Gradient boosted classifier | Ambiguous cases, cross-language matching | Learned blocking from training data | Configurable (default 90%) |
| **Fuzzy** | Jaro-Winkler + Levenshtein | Typo-tolerant matching | N-gram based | Configurable (default 80%) |
| **Phonetic** | Soundex / Double Metaphone | Name matching across transliterations | Phonetic encoding | Exact phonetic match |

## Quality Dashboard Components

| Component | Data Source | Refresh Rate | Description |
|-----------|-----------|--------------|-------------|
| Enterprise Quality Score | Integration Service | Hourly | Aggregate quality score across all domains with trend |
| Domain Scorecard | Integration Service | Hourly | Per-domain (CRM, Finance, Supply Chain) quality breakdown by dimension |
| Entity Quality Grid | Integration Service | On demand | Per-entity quality scores with drill-down to field level |
| Duplicate Heat Map | Integration Service | On profiling run | Duplicate density visualization by entity and source |
| Cleansing Activity Feed | Integration Service | Real-time | Live feed of cleansing actions applied with before/after |
| Stewardship Queue | Integration Service | Real-time | Open exceptions requiring manual review with priority and age |
| SLA Compliance Gauge | Integration Service | Hourly | SLA target vs. actual for each dimension and entity |
| Profiling Trend Chart | Report Service | Daily | Quality dimension trends over time with anomaly markers |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Master Data Management | Integration Service | Internal API | Golden record sync, match result propagation, survivorship rules |
| Data Import/Export | Integration Service | Internal API | Inline profiling and cleansing during import; quality-gated export |
| Customer Data Platform | Commerce Service | Internal API | Customer deduplication, enrichment, and quality scoring |
| Integration Service | Integration Service | Internal API | ETL pipeline hooks for inline quality checks and transformations |
| Report Service | Report Service | gRPC | Quality dashboard data, ML-enhanced matching inference, anomaly detection |
| Platform Workflow | Workflow Service | Event-driven | Stewardship task routing, SLA breach escalation, approval workflows |
| Notification Service | Platform Service | Event-driven | Quality alert delivery, SLA breach notifications, stewardship assignments |
| Audit Trail | Platform Service | Event-driven | Full audit log of all quality operations, cleansing actions, and overrides |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `integration.data-quality.profiling.completed` | `{profile_id, entity_type, records_profiled, scores{}}` | Profiling run finishes | Report Service, Dashboard |
| `integration.data-quality.anomaly.detected` | `{anomaly_id, entity_type, dimension, score, threshold, affected_records}` | Quality score degrades below threshold | Notification Service, Stewardship Queue |
| `integration.data-quality.match.completed` | `{match_job_id, entity_type, total_records, match_pairs, auto_merged, exceptions}` | Matching job finishes | MDM Engine, Dashboard |
| `integration.data-quality.cleansing.applied` | `{cleansing_id, entity_type, rule_id, records_cleansed, fields_corrected}` | Cleansing rules applied | Audit Trail, Dashboard |
| `integration.data-quality.enrichment.completed` | `{enrichment_id, entity_type, provider, records_enriched, fields_added}` | Enrichment pipeline finishes | Dashboard, Entity Owners |
| `integration.data-quality.sla.breached` | `{sla_id, entity_type, dimension, target, actual, breached_at}` | Quality SLA target missed | Notification Service, Escalation Workflow |
| `integration.data-quality.stewardship.created` | `{task_id, entity_type, record_ids, exception_type, priority}` | Manual review required for match/cleansing exception | Notification Service, Stewardship Queue |
| `integration.data-quality.stewardship.resolved` | `{task_id, resolved_by, resolution, records_affected}` | Steward resolves exception | Audit Trail, MDM Engine |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `integration.sync.started` | Integration Service | Trigger inline profiling and cleansing on incoming data |
| `integration.master-data.merged` | Integration Service | Re-profile affected entity, update quality scores |
| `crm.cdp.profile.created` | Commerce Service | Trigger deduplication and matching |
| `workflow.step.approved` | Platform Service | Close stewardship task on workflow completion |
| `report.ml.inference.complete` | Report Service | Apply ML matching scores and anomaly classifications |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `dq_profiling_run` | `id, entity_type, source_system, records_profiled, started_at, completed_at` | Has many `dq_column_profile`, `dq_quality_score` |
| `dq_column_profile` | `id, profiling_run_id, column_name, null_pct, distinct_count, min, max, avg, top_values{}, patterns{}` | Belongs to `dq_profiling_run` |
| `dq_quality_score` | `id, profiling_run_id, entity_type, dimension, score, target, sla_breached` | Belongs to `dq_profiling_run` |
| `dq_rule` | `id, name, category, type, config_json, entity_type, severity, is_active` | Has many `dq_rule_execution` |
| `dq_rule_execution` | `id, rule_id, profiling_run_id, records_evaluated, records_failed, corrected_count` | Belongs to `dq_rule`, `dq_profiling_run` |
| `dq_match_job` | `id, entity_type, strategy, blocking_key, threshold, total_records, match_pairs, auto_merged, exceptions` | Has many `dq_match_result` |
| `dq_match_result` | `id, match_job_id, record_id_1, record_id_2, match_score, match_type, status` | Belongs to `dq_match_job` |
| `dq_stewardship_task` | `id, entity_type, record_ids[], exception_type, priority, assignee_id, status, resolution` | References `dq_match_result` or `dq_rule_execution` |
| `dq_sla` | `id, entity_type, dimension, target_score, evaluation_frequency, breach_notification[]` | Template for quality enforcement |

## Cross-References

- [Master Data Management](../06-services/integration.md) — Golden record management and survivorship (MDM module)
- [Data Import/Export](../06-services/integration.md) — Inline quality checks during data loading (Data Import/Export module)
- [Customer Data Platform](../06-services/crm.md) — Customer unification and deduplication (CDP module)
- [Integration Service](../06-services/integration.md) — ETL pipeline and data integration
- [Architecture Overview](../01-architecture/overview.md) — System context
