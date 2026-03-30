# Integration Service

| Aspect | Details |
|--------|---------|
| Port | 8021 |
| Database | `integration_db` |
| Responsibilities | External connectivity, connector framework, webhooks, EDI, data import/export, MDM, data governance, trade compliance, API marketplace, event mesh, blockchain, enterprise data quality |

## Features

| Feature | Description |
|---------|-------------|
| Connectors | 50+ pre-built connectors across 6 categories (see below) |
| Connector SDK | Rust-based plugin framework for building custom connectors |
| Webhooks | Inbound webhook registration, payload mapping, signature verification, retry |
| EDI | ANSI X12 / EDIFACT processing, AS2/SFTP transport |
| Data Import/Export | CSV/JSON/Excel/XML/PDF/Parquet, field mapping, validation, bulk processing |
| Integration Monitoring | Connection health, throughput metrics, error tracking |
| OAuth2 Client | External OAuth2 flows for connector authentication |
| MDM | Golden records, deduplication, Fellegi-Sunter matching, data stewardship |
| Data Governance | Data ownership, lineage tracking, policy enforcement |
| Trade Compliance | OFAC/EU/UN screening, export controls (ECCN), license management, customs documentation |
| API Marketplace | Gateway management, rate limiting, developer portal, API key management |
| Event Mesh | RabbitMQ topology management, event gateway, cross-region replication |
| Blockchain | Hyperledger/Ethereum adapters, smart contract management, supply chain provenance |
| Enterprise Data Quality | Profiling, cleansing, probabilistic matching, enrichment, quality scorecards |

## Connectors

**Financial (14):** SAP S/4HANA, NetSuite, Dynamics 365 Finance (ERP); QuickBooks, Xero, Sage (Accounting); Stripe, PayPal, Adyen, Square (Payments); Plaid, SWIFT (Banking); Vertex, Avalara (Tax)

**CRM & CX (8):** Salesforce, HubSpot, Dynamics 365 Sales (CRM); Zendesk, Freshdesk (Support); Intercom, Twilio (Messaging); SendGrid (Email delivery)

**HR & People (6):** Workday, BambooHR (HCM); ADP, Paychex (Payroll); LinkedIn Recruiter, Greenhouse (ATS)

**Supply Chain & Logistics (8):** FedEx, UPS, DHL, USPS (Shipping); Amazon Seller Central (Marketplace); Shopify, Magento, WooCommerce (Commerce)

**Collaboration & Productivity (7):** Microsoft 365, Google Workspace (Productivity); Slack, Microsoft Teams (Messaging); Jira (Project management); Confluence, Notion (Knowledge)

**Infrastructure & Developer (14):** AWS S3, Azure Blob, GCS (Object storage); SMTP/IMAP (Email); SFTP (File transfer); Generic REST, Generic SOAP, Generic Webhook (API); EDI X12/EDIFACT (B2B); OPC-UA (Industrial); MQTT (IoT); Snowflake (Data warehouse); Power BI, Tableau (Analytics)

## Connector SDK

Rust-based plugin framework for custom connectors as dynamically loaded modules.

### Connector Trait

```rust
trait Connector: Send + Sync {
    fn metadata(&self) -> ConnectorMetadata;
    fn validate_config(&self, config: &Value) -> Result<(), ConnectorError>;
    fn test_connection(&self, config: &Value) -> Result<(), ConnectorError>;
    fn discover_schema(&self, config: &Value) -> Result<Schema, ConnectorError>;
    fn read(&self, config: &Value, opts: ReadOptions) -> Result<DataStream, ConnectorError>;
    fn write(&self, config: &Value, data: DataStream) -> Result<WriteResult, ConnectorError>;
}
```

### Lifecycle & Error Handling

| Phase | Description |
|-------|-------------|
| Registration | Metadata (name, version, category, config schema) registered at startup |
| Validation | Config validated against connector's JSON Schema before persisting |
| Health Check | Periodic connectivity test (every 60s) updates connection status |
| Execution | Read/write with configurable retry and timeout |
| Teardown | Graceful shutdown drains in-flight operations |

| Error Type | Retry | Description |
|------------|-------|-------------|
| Transient (502, 503, 504) | Yes — exponential backoff (1s→8s, max 5) | External system unavailable |
| Auth (401, 403) | Yes — one auto token refresh, then fail | Credential expiration |
| Validation (400) | No | Invalid request or mapping |
| Rate Limit (429) | Yes — respect `Retry-After` header | API rate limit exceeded |

## Webhooks

1. External system registers subscription via `POST /api/v1/integrations/webhooks` with content type and verification method.
2. Service generates unique endpoint: `/webhooks/{tenant_id}/{subscription_id}`.
3. Payloads transformed via JSONata/JMESPath mapping rules (per subscription, versioned).

### Verification & Retry

| Method | Description |
|--------|-------------|
| HMAC-SHA256 | Signature from raw body + shared secret vs `X-Signature-256` header |
| Timestamp + HMAC | Rejects signatures older than 5 minutes to prevent replay |
| IP Whitelist | Restricts calls to configured source IPs |

Failed deliveries retried up to 5 times with exponential backoff (1s–16s). Dead-lettered to `webhook_dlq` after max retries. Deduplicated by `X-Webhook-Id` within 24h window.

## EDI

### Standards & Transport

| Standard | Message Types |
|----------|---------------|
| ANSI X12 | 850 (PO), 810 (Invoice), 856 (ASN), 855 (PO Ack), 997 (Functional Ack) |
| EDIFACT | ORDERS, INVOIC, DESADV, ORDRSP, CONTRL |

| Transport | Description |
|-----------|-------------|
| AS2 | Encrypted, signed, MDN acknowledgment over HTTPS |
| SFTP | File-based exchange with PGP encryption |
| VAN | Value-Added Network relay |

### Processing Pipeline

1. Inbound document received via AS2/SFTP.
2. Parsed and validated against partner-specific implementation guide.
3. Functional acknowledgment (997/CONTRL) sent automatically.
4. Transformed to canonical format via partner mapping.
5. Published to outbox as `integration.sync.completed`.
6. Validation failures emit `INTEGRATION_EDI_VALIDATION_FAILED` and reject the document.

## Data Import/Export

| Format | Import | Export | Notes |
|--------|--------|--------|-------|
| CSV | Yes | Yes | Delimiter auto-detection |
| JSON | Yes | Yes | Nested objects flattened via dot-notation |
| Excel (.xlsx) | Yes | Yes | Multi-sheet; first row as headers |
| XML | Yes | Yes | XPath-based field extraction |
| PDF | No | Yes | Table-based export via template |
| Parquet | No | Yes | Columnar for data lake / DuckDB |

**Import:** Upload via `POST /api/v1/integrations/import` → async job → field mapping → row-level validation → write to target service REST API → summary (total, imported, skipped, failed).

**Export:** Request via target service API or scheduled job → fetch with pagination → map and serialize → store in MinIO `{tenant_id}/exports/` → optional SFTP delivery.

## Integration Monitoring

| Metric | Source |
|--------|--------|
| Connection health (up/down) | Periodic health check probe |
| Sync throughput (records/min) | Sync execution logs |
| Error rate (failures/hour) | Error event aggregation |
| Latency p50/p99 | OpenTelemetry spans |
| Data volume (bytes/day) | Import/export job logs |

All metrics exposed via Prometheus at `:8021/metrics`.

## MDM (Master Data Management)

### Golden Records

| Domain | Source Systems |
|--------|----------------|
| Customer | CRM, Commerce, Finance |
| Vendor | Finance (AP), Commerce (procurement) |
| Product | Commerce (PIM), Manufacturing (BOM) |
| Employee | HCM, Identity |

### Matching

| Method | Description |
|--------|-------------|
| Deterministic | Exact/normalized match on key attributes (tax ID, email) |
| Probabilistic (Fellegi-Sunter) | Log-likelihood ratio scoring with configurable per-attribute weights |
| Fuzzy | Levenshtein, Jaro-Winkler for name/address |

Auto-merge above upper threshold, auto-reject below lower threshold, manual review between thresholds.

### Data Stewardship

Tasks assigned to stewards by domain and org unit. SLA tracking with escalation (default: 48h). Actions: merge, reject match, edit golden record, request re-profiling.

## Data Governance

| Capability | Description |
|------------|-------------|
| Data Catalog | Entity inventory with ownership, classification, sensitivity tags |
| Lineage Tracking | Graph-based end-to-end data flow from source to consumption |
| Policy Enforcement | Retention, access, quality, and privacy rules at integration points |
| Classification | Automatic and manual (PII, PHI, financial, confidential, public) |
| Retention Policies | Per-entity rules with automated archive/purge |

## Trade Compliance

### Restricted Party Screening

| List | Authority | Frequency |
|------|-----------|-----------|
| SDN | OFAC (US Treasury) | Daily |
| EU Consolidated | EU | Daily |
| UN Consolidated | UN Security Council | Daily |
| Entity List | US BIS | Weekly |
| Denied Persons | US BIS | Weekly |

### Export Controls

ECCN classification per product. License management tracks validity, scope (country, commodity, quantity), and conditions. Re-screening on configurable schedules (daily, weekly, on-change). Automated customs documentation (commercial invoices, packing lists, certificates of origin).

### Screening Flow

1. Record submitted via `POST /api/v1/integrations/trade-compliance/screen`.
2. Fields matched against all active sanctions lists using fuzzy matching.
3. Matches above threshold flagged with `integration.trade-compliance.screening.flagged`.
4. Confirmed matches blocked from transactions until cleared by compliance officer.
5. Full audit trail (match score, list source, reviewer, disposition).

## API Marketplace

| Capability | Description |
|------------|-------------|
| API Catalog | Centralized registry with OpenAPI 3.1 specs, versioning, lifecycle states |
| Gateway Management | Traefik route configuration for published endpoints |
| Rate Limiting | Per-subscription limits (req/min, req/day, bytes/month) at gateway |
| API Key Management | Provisioned via developer portal; keys hashed (bcrypt) at rest |
| Developer Portal | Self-service docs, sandbox, migration guides, try-it-out console |
| Usage Analytics | Per-subscription call counts, latency, error rates |
| Monetization | Tiered plans (free/standard/premium) with quota and overage billing to Finance |

## Event Mesh

| Capability | Description |
|------------|-------------|
| Topology Management | RabbitMQ exchange/queue/binding management via API |
| Event Gateway | HTTP-to-AMQP bridge for external producers |
| Schema Registry | JSON Schema storage and validation |
| Cross-Region Replication | RabbitMQ federation with retry and DLQ |
| Event Filtering | Header-based and content-based subscription filters |

## Blockchain

| Capability | Description |
|------------|-------------|
| Adapters | Hyperledger Fabric and Ethereum-compatible |
| Smart Contracts | Deploy, invoke, query via API |
| Provenance | Anchor immutable records (hash + metadata) to ledger |
| Verification | Verify anchored records by hash |

## Enterprise Data Quality

| Capability | Description |
|------------|-------------|
| Data Profiling | Statistical analysis: completeness, uniqueness, distribution, patterns |
| Cleansing | Standardization, normalization, enrichment transformations |
| Matching | Fellegi-Sunter probabilistic model with configurable thresholds |
| Enrichment | Auto-enrichment from master data and external sources |
| Scorecards | Per-entity: completeness, accuracy, consistency, timeliness, validity |
| Stewardship | Review/remediation tasks with SLA tracking, integrated with MDM |

## Database Tables

> All tables include standard columns per [SPEC.md §9.1](../SPEC.md).

| Table | Purpose | Key Columns |
|-------|---------|-------------|
| `connectors` | Registered connector instances | `connector_type`, `name`, `category`, `status` |
| `connector_configs` | Per-connector config (encrypted) | `connector_id` (FK), `config_data` (JSONB, encrypted), `config_schema` (JSONB) |
| `webhook_subscriptions` | Inbound webhook registrations | `path`, `source_system`, `content_type`, `verification_method`, `secret` (encrypted), `mapping_rules` (JSONB) |
| `edi_transactions` | Inbound/outbound EDI documents | `direction`, `standard`, `message_type`, `partner_id`, `status`, `raw_payload`, `canonical_payload` (JSONB) |
| `import_export_jobs` | Async import/export jobs | `job_type`, `format`, `entity_type`, `status`, `total_rows`, `imported_rows`, `failed_rows`, `file_path` |
| `mdm_golden_records` | Authoritative master data | `domain`, `attributes` (JSONB), `confidence_score`, `source_count`, `last_merged_at` |
| `mdm_source_records` | Source records → golden records | `golden_record_id` (FK), `source_system`, `source_id`, `raw_attributes` (JSONB), `match_score` |
| `mdm_stewardship_tasks` | Data steward review tasks | `domain`, `golden_record_id` (FK), `task_type`, `status`, `assigned_to`, `due_at`, `resolved_at` |
| `trade_compliance_screenings` | Sanctions screening results | `entity_type`, `entity_id`, `screening_type`, `status`, `matches` (JSONB), `reviewed_by`, `disposition` |
| `api_marketplace_listings` | Published API catalog | `api_name`, `version`, `openapi_spec` (JSONB), `lifecycle_state`, `tier` |
| `api_marketplace_subscriptions` | Consumer subscriptions | `listing_id` (FK), `subscriber_id`, `api_key_hash`, `rate_limit`, `quota`, `usage_counters` (JSONB) |
| `data_quality_rules` | Quality rule definitions | `domain`, `entity_type`, `rule_type`, `expression` (JSONB), `severity`, `is_active` |
| `data_quality_violations` | Detected violations | `rule_id` (FK), `entity_type`, `record_id`, `violation_details` (JSONB), `status`, `resolved_at` |

## Events Published

| Event | Payload | Description |
|-------|---------|-------------|
| `integration.sync.started` | `{ integration_id, connector_type, direction }` | External system sync initiated |
| `integration.sync.completed` | `{ integration_id, records_synced, duration_ms }` | Sync completed |
| `integration.sync.failed` | `{ integration_id, error_code, error_message }` | Sync failed |
| `integration.import.completed` | `{ job_id, entity_type, total_rows, imported_rows, failed_rows }` | Import completed |
| `integration.import.failed` | `{ job_id, entity_type, error_message }` | Import failed |
| `integration.master-data.matched` | `{ domain, golden_record_id, source_record_id, match_score }` | Matching result |
| `integration.master-data.merged` | `{ domain, golden_record_id, merged_source_ids }` | Records merged |
| `integration.master-data.quality.violation` | `{ domain, rule_id, record_id, violation_details }` | Quality violation |
| `integration.trade-compliance.screening.completed` | `{ screening_id, entity_type, entity_id, result }` | Screening completed |
| `integration.trade-compliance.screening.flagged` | `{ screening_id, entity_type, entity_id, match_score, sanctions_list }` | Screening flagged |
| `integration.trade-compliance.license.expiring` | `{ license_id, eccn, country, expires_at }` | License expiring |
| `integration.blockchain.record.anchored` | `{ anchor_id, asset_id, hash, network, timestamp }` | Record anchored |
| `integration.blockchain.record.verified` | `{ anchor_id, verifier_id, timestamp }` | Record verified |
| `integration.blockchain.smart-contract.deployed` | `{ contract_id, address, network, deployer }` | Contract deployed |
| `integration.blockchain.smart-contract.executed` | `{ contract_id, transaction_hash, result }` | Contract executed |
| `integration.developer-portal.api-key.provisioned` | `{ key_id, developer_id, scopes }` | API key provisioned |
| `integration.developer-portal.sandbox.reset` | `{ sandbox_id, requester_id }` | Sandbox reset |
| `integration.data-quality.profile.completed` | `{ profile_id, entity_type, records_profiled, quality_score }` | Profiling completed |
| `integration.data-quality.rule.violated` | `{ violation_id, rule_id, entity_type, record_id, violation_details }` | Quality rule violated |
| `integration.data-quality.cleansing.completed` | `{ batch_id, records_processed, records_cleansed, records_flagged }` | Cleansing completed |
| `integration.data-quality.match.completed` | `{ match_id, entity_type, records_matched, records_merged, survivors }` | Matching completed |

## Events Consumed

Inbox binding: `integration.inbox` binds to the following routing keys:

| Binding Pattern | Events Consumed | Action |
|----------------|-----------------|--------|
| `integration.#` | Self-binding for saga compensation | — |
| `config.changed` | `config.changed` | Reload connector configurations and rate limit thresholds |
| `commerce.customer.#` | `commerce.customer.created`, `commerce.customer.updated` | MDM: ingest customer master data for golden record matching/deduplication |
| `commerce.product.#` | `commerce.product.created`, `commerce.product.updated` | MDM: ingest product master data for golden record matching/deduplication |
| `finance.supplier.#` | `finance.supplier.created`, `finance.supplier.updated` | MDM: ingest supplier/vendor master data for golden record matching/deduplication |
| `hr.employee.#` | `hr.employee.created`, `hr.employee.hired`, `hr.employee.updated` | MDM: ingest employee master data for golden record matching/deduplication |
| `identity.user.#` | `identity.user.created`, `identity.user.updated` | MDM: ingest user/identity master data for golden record matching/deduplication |

> **MDM Note:** Integration Service consumes master data events from all services to build and maintain MDM golden records. Events are processed through the matching/deduplication pipeline (deterministic, probabilistic Fellegi-Sunter, and fuzzy matching) to produce unified golden records. Services MUST publish master data events; they MUST NOT write to Integration's database directly.

> **Note:** Integration Service is primarily an event producer. Report Service consumes data quality events for dashboards. Commerce and Finance consume sync events for cross-system status. Platform Service compliance hub consumes trade compliance screening events.

## See Also

- [Platform Service](platform.md)
- [Config Service](config.md)
- [Commerce Service](commerce.md)
- [EDI Feature Spec](../07-features/edi.md)
- [API Marketplace Feature Spec](../07-features/api-marketplace.md)
- [Enterprise Data Quality Feature Spec](../07-features/enterprise-data-quality.md)
- [Architecture Overview](../01-architecture/overview.md)
