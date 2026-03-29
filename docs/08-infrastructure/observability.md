# Monitoring & Observability

## 1. Metrics (Prometheus)

All services MUST expose a `/metrics` endpoint (Prometheus format) with the following metric categories:

| Category | Metrics |
|----------|---------|
| HTTP | Request rate, latency (p50, p95, p99), error rate, active requests |
| Business | Orders created, invoices generated, stock alerts, pipeline conversions, emissions recorded |
| Database | Query latency, connection pool usage, replication lag |
| Messaging | Publish rate, consume rate, consumer lag, DLQ depth |
| Cache | Hit rate, miss rate, eviction rate, memory usage |
| Infrastructure | CPU, memory, disk, network (via node exporter) |
| ML/AI | Inference latency, prediction confidence, model drift, feature store freshness |

## 1.1 Standard Service Metrics

All services MUST expose applicable metrics from this list based on their domain:

| Metric | Type | Labels | Description |
|--------|------|--------|-------------|
| `http_requests_total` | Counter | method, path, status | Total HTTP requests |
| `http_request_duration_seconds` | Histogram | method, path | Request latency |
| `http_active_requests` | Gauge | — | Currently in-flight requests |
| `db_query_duration_seconds` | Histogram | query_name | Database query latency |
| `db_pool_active_connections` | Gauge | — | Active DB connections |
| `db_pool_idle_connections` | Gauge | — | Idle DB connections |
| `event_published_total` | Counter | event_type, status | Events published |
| `event_consumed_total` | Counter | event_type, status | Events consumed |
| `event_processing_duration_seconds` | Histogram | event_type | Event processing latency |
| `outbox_pending_count` | Gauge | — | Pending outbox entries |
| `cache_hit_total` | Counter | key_pattern | Cache hits |
| `cache_miss_total` | Counter | key_pattern | Cache misses |
| `saga_started_total` | Counter | saga_type | Sagas started |
| `saga_completed_total` | Counter | saga_type, status | Sagas completed (success/failed/compensated) |
| `assistant_intent_total` | Counter | intent, status | Digital assistant intent resolution |
| `assistant_action_duration_seconds` | Histogram | action | Digital assistant action execution time |
| `ml_inference_duration_seconds` | Histogram | model_name | ML inference latency |
| `esg_emissions_recorded_total` | Counter | scope, unit | Emissions data points recorded |
| `scheduler_job_duration_seconds` | Histogram | job_type | Scheduled job execution duration |
| `scheduler_job_active_count` | Gauge | job_type | Currently running scheduled jobs |
| `subscription_billing_cycle_total` | Counter | status | Subscription billing cycle results |
| `credit_check_total` | Counter | result | Credit check outcomes (pass/hold/release) |
| `trade_compliance_screening_total` | Counter | result | Trade compliance screening outcomes |
| `knowledge_search_total` | Counter | result | Knowledge base search results (hit/miss) |
| `iot_telemetry_ingested_total` | Counter | device_id, telemetry_type | IoT telemetry events ingested |
| `iot_device_alert_total` | Counter | device_id, alert_type | IoT alert rule triggers |
| `digital_twin_sync_duration_seconds` | Histogram | asset_id | Digital twin state sync latency |
| `rpa_bot_execution_total` | Counter | bot_id, status | RPA bot execution outcomes |
| `rpa_bot_execution_duration_seconds` | Histogram | bot_id | RPA bot execution duration |
| `process_mining_analysis_duration_seconds` | Histogram | process_type | Process mining analysis duration |
| `cdp_identity_resolution_total` | Counter | result | CDP identity resolution outcomes |
| `collaboration_message_total` | Counter | channel_id | Collaboration messages posted |
| `logistics_tracking_update_total` | Counter | carrier, status | Logistics tracking updates received |
| `rate_limit_exceeded_total` | Counter | tenant_id, endpoint | Rate limit violations |
| `lease_calculation_duration_seconds` | Histogram | classification | Lease accounting calculation latency |
| `grant_recognition_total` | Counter | grant_id, status | Grant revenue recognition outcomes |
| `jv_allocation_duration_seconds` | Histogram | venture_id | Joint venture cost allocation latency |
| `intelligent_close_anomaly_total` | Counter | anomaly_type | Financial close anomaly detections |
| `cash_application_match_total` | Counter | result | Cash application matching outcomes |
| `idp_extraction_duration_seconds` | Histogram | document_type | IDP document extraction latency |
| `idp_extraction_confidence` | Histogram | document_type, field | IDP extraction confidence scores |
| `warranty_claim_total` | Counter | result | Warranty claim processing outcomes |
| `collection_strategy_triggered_total` | Counter | strategy_type | Collection strategy activations |

## 1.2 Dashboards (Grafana)

| Dashboard | Audience | Key Panels |
|-----------|----------|-----------|
| System Overview | SRE, DevOps | All services health, throughput, error rates, pod counts |
| Service Health | Developers | Per-service HTTP metrics, DB connections, event lag |
| Business KPIs | Business users | Orders, invoices, revenue, pipeline, headcount |
| Database Performance | DBAs | Query latency, connection pools, replication lag, slow queries |
| Message Queue Status | SRE | Queue depth, consumer lag, DLQ count, publish rate |
| Cache Performance | Developers | Hit/miss ratio, eviction rate, memory, key count |
| Security | Security team | Login failures, auth errors, rate limit hits, audit events, SoD conflicts |
| ESG / Sustainability | ESG analysts | Emissions by scope, carbon intensity, target progress, offset tracking |
| AI/ML Operations | Data scientists | Model accuracy, inference latency, feature drift, prediction volume |
| Cost | Finance | Resource utilization, pod counts, storage usage, data lake size |
| Digital Assistant | Product team | Intent distribution, success rate, response time, user satisfaction |
| Subscription Health | Product, Finance | MRR/ARR, churn rate, renewal rate, billing success rate |
| Credit & Collections | Finance, Sales | Credit exposure, aging buckets, collection effectiveness index |
| Trade Compliance | Legal, Operations | Screening volumes, match rates, license status, shipment holds |
| Knowledge Base | Support, Operations | Article views, search effectiveness, resolution rate, content gaps |
| SLI/SLO Dashboard | SRE, Engineering | Error budget burn rate, SLO compliance, alert history |
| IoT & Digital Twin | Operations, Manufacturing | Device connectivity, telemetry throughput, alert rates, twin sync latency |
| RPA Operations | Operations, IT | Bot execution rates, success rates, execution duration, error distribution |
| Process Performance | Process owners, Operations | Process cycle times, conformance rates, bottleneck heatmaps |
| Supplier Risk | Procurement, Finance | Risk score distribution, alert volumes, mitigation plan status |
| Customer Intelligence | Marketing, Sales | Profile completeness, segment distribution, journey completion, engagement scores |
| B2B Portal | Commerce, Sales | Portal adoption, order volume, reorder rates, customer satisfaction |
| Collaboration | IT, Operations | Message volume, active channels, task completion rates |
| Financial Close | Finance | Close task progress, reconciliation match rates, close cycle duration |
| Privacy & Compliance | DPO, Legal | Consent rates, DSAR volumes, DPIA status, processing register, breach notifications |
| DLP Monitoring | Security team | DLP policy violations, sensitive data movement, incident resolution |
| Content Management | Content managers | Document lifecycle, records retention, compliance archive status |
| API Marketplace | API product managers | API adoption, consumer usage, rate limit utilization, monetization revenue |
| Digital Thread | Engineering, Quality | Traceability coverage, change impact analysis, genealogy completeness |
| Lease Accounting | Finance, Treasury | Lease portfolio, ROU asset values, liability maturity, payment schedules |
| Grant Management | Finance, Program Managers | Grant utilization, budget vs. actual, compliance deadlines, revenue recognized |
| Joint Venture | Finance, Operations | Partner cost shares, billing status, allocation accuracy, venture performance |
| Intelligent Close | Finance, Controllers | Close progress, anomaly count, auto-reconciliation rates, close cycle duration |
| Collections | Finance, AR | Aging analysis, collection effectiveness, cash application accuracy, dispute resolution |
| Warranty Analytics | Commerce, Manufacturing | Claim rates, cost analysis, product quality correlation, resolution times |
| IDP Operations | Operations, IT | Document volume, extraction accuracy, processing time, model performance |

## 1.3 Structured Logging

All services MUST emit structured JSON logs with these fields:

```json
{
  "timestamp": "2026-03-28T10:30:00Z",
  "level": "INFO",
  "service": "commerce-service",
  "tenant_id": "tenant-uuid",
  "request_id": "request-uuid",
  "trace_id": "trace-uuid",
  "span_id": "span-uuid",
  "message": "Order created successfully",
  "order_id": "order-uuid",
  "customer_id": "customer-uuid"
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `timestamp` | Yes | ISO 8601 with timezone |
| `level` | Yes | TRACE, DEBUG, INFO, WARN, ERROR |
| `service` | Yes | Service name |
| `tenant_id` | Conditional | Tenant context — present for tenant-scoped requests, omitted for system-level operations |
| `request_id` | Conditional | From `X-Request-ID` header — omitted for background tasks and event handlers |
| `trace_id` | Yes | From OpenTelemetry context |
| `span_id` | Yes | From OpenTelemetry context |
| `message` | Yes | Human-readable log message |

**Log Redaction:** PII fields (email, phone, SSN) are automatically redacted in logs using the `mserp-core` redaction utility.

**Log Levels Policy:**
- `TRACE`: Verbose internal state (disabled in production)
- `DEBUG`: Diagnostic information (disabled in production by default)
- `INFO`: Business events (order created, invoice generated)
- `WARN`: Recoverable issues (cache miss fallback, retry triggered)
- `ERROR`: Failures requiring attention (database error, external service failure)

## 1.4 Distributed Tracing

All services participate in distributed tracing via OpenTelemetry + Jaeger:

- Every HTTP request creates a root span.
- Service-to-service HTTP calls propagate `trace_id` via `traceparent` header.
- Event publishing creates a child span with event metadata.
- Event consumption creates a linked span using `correlation_id`.
- Database queries are instrumented as child spans.
- Cache operations are instrumented as child spans.
- ML inference calls are instrumented as child spans with model metadata.

**Trace Sampling:**
- Production: 10% default sampling rate; 100% for error traces and slow requests (> 2 seconds).
- Staging: 100% sampling.
- Trace retention: 7 days (hot), 30 days (cold storage).

## 1.5 Alerts

| Alert | Severity | Response Time | Response |
|-------|----------|---------------|----------|
| Service Down (all replicas) | Critical | < 5 minutes | Investigate, restart, escalate |
| High Error Rate (>1% of requests) | Critical | < 5 minutes | Investigate, rollback if recent deploy |
| High Latency (p99 > 2s) | Warning | < 15 minutes | Investigate, check DB/cache |
| Disk Usage > 80% | Warning | < 1 hour | Investigate, clean up, expand |
| Disk Usage > 90% | Critical | < 15 minutes | Immediate action to free space |
| Certificate Expiry < 14 days | Warning | < 24 hours | Trigger cert rotation |
| DLQ Depth > 100 | Warning | < 1 hour | Investigate failed messages |
| RabbitMQ Consumer Lag > 1000 | Warning | < 15 minutes | Scale consumers, investigate bottleneck |
| Database Replication Lag > 30s | Warning | < 15 minutes | Check network, primary load |
| Pod Restart Loop | Critical | < 5 minutes | Check logs, resource limits |
| Saga Compensation Failed | Critical | < 5 minutes | Manual intervention, check DLQ |
| Tenant Quota Exceeded | Info | < 24 hours | Notify tenant admin |
| SoD Conflict Detected | Warning | < 1 hour | Review and mitigate |
| ML Model Drift Detected | Warning | < 24 hours | Evaluate retraining, review accuracy |
| ESG Data Anomaly | Info | < 48 hours | Review emissions data, verify calculations |
| Trade Compliance Screening Failure | Critical | < 5 minutes | Halt affected orders, investigate screening service |
| Subscription Billing Failure | Critical | < 15 minutes | Investigate, manual billing fallback |
| Revenue Recognition Discrepancy | Warning | < 1 hour | Finance team review, check recognition rules |
| Scheduler Job Overdue | Warning | < 30 minutes | Investigate scheduler service, check job queue |
| Error Budget Exhausted | Warning | < 4 hours | Freeze feature work, prioritize reliability |
| Backup Verification Failed | Critical | < 1 hour | Investigate backup infrastructure, re-run verification |
| IoT Device Offline (>10 devices) | Warning | < 30 minutes | Investigate device connectivity, check gateway |
| RPA Bot Failure Rate > 10% | Warning | < 30 minutes | Investigate failing bots, check target systems |
| Process Conformance < 90% | Warning | < 4 hours | Review process deviations, update workflow definitions |
| CDP Identity Resolution Backlog > 1000 | Warning | < 1 hour | Scale CDP processing, check data source connectivity |
| Supplier Risk Score Critical | Critical | < 1 hour | Review supplier risk detail, notify procurement, initiate mitigation |
| Digital Twin Sync Lag > 5s | Warning | < 15 minutes | Check telemetry pipeline, investigate device health |
| Financial Close Task Overdue | Warning | < 1 hour | Notify task owner, escalate to close manager |
| DLP Policy Violation | Critical | < 5 minutes | Investigate data exfiltration attempt, block if confirmed |
| Privacy Breach Detected | Critical | < 15 minutes | Activate breach notification workflow, assess scope |
| Content Repository Storage > 80% | Warning | < 1 hour | Review retention policies, archive old content |
| Lease Calculation Failure | Critical | < 15 minutes | Investigate lease service, check calculation engine |
| Grant Compliance Deadline Approaching | Warning | < 24 hours | Notify grant manager, prepare compliance report |
| JV Allocation Imbalance | Warning | < 4 hours | Review joint venture allocation rules, check cost pool |
| Intelligent Close Stalled | Warning | < 30 minutes | Check close automation service, review task queue |
| Cash Application Match Rate < 90% | Warning | < 1 hour | Review matching rules, check ML model accuracy |
| IDP Extraction Accuracy < 95% | Warning | < 4 hours | Review extraction models, check document quality |
| Warranty Claim Spike | Warning | < 1 hour | Investigate product quality, review claim patterns |

## 1.6 Alert Routing

| Severity | Channel | Escalation |
|----------|---------|-----------|
| Critical | PagerDuty + Slack #incidents | On-call engineer → Engineering lead → CTO |
| Warning | Slack #alerts | On-call engineer acknowledges |
| Info | Slack #notifications | No action required |
