# Data Lake

## 1. Data Lake

The Report Service manages a data lake built on MinIO (S3-compatible) for raw and curated analytical data.

### 1.1 Data Lake Zones

| Zone | Purpose | Format | Retention |
|------|---------|--------|-----------|
| Raw (Bronze) | Unprocessed event and operational data | JSON, Parquet | 2 years |
| Curated (Silver) | Cleaned, deduplicated, typed data | Parquet | Indefinite |
| Analytics (Gold) | Pre-aggregated, business-ready datasets | Parquet | Indefinite |

### 1.2 Ingestion

- The outbox poller publishes events to RabbitMQ. A separate Event Archive Worker consumes events from RabbitMQ and writes them to the MinIO raw zone.
- ETL jobs promote raw data to curated, applying schema validation and deduplication.
- Analytics zone is populated by scheduled aggregation jobs.

### 1.3 Schema-on-Read

The data lake uses a schema-on-read approach for exploratory analytics:
- Raw zone: No enforced schema; events stored in their original format.
- Curated zone: Schema enforced by ETL; Apache Arrow schema definitions.
- Query access via DuckDB's S3/Parquet integration or direct API.

### 1.4 Relationship to DuckDB Data Warehouse

The Data Lake and DuckDB Data Warehouse form a single analytical pipeline. The Data Lake is the **sole data source** for the warehouse — the Report Service never reads from operational PostgreSQL databases directly.

**Full pipeline:**

```
Service Events → Outbox Tables → RabbitMQ → Event Archive Worker → MinIO Raw (Bronze)
  → ETL (validate, deduplicate) → MinIO Curated (Silver)
  → Aggregation Jobs → MinIO Gold (Analytics)
  → ETL Load → DuckDB Warehouse (Star Schema)
```

- The Gold zone contains pre-aggregated Parquet files stored in MinIO.
- The DuckDB warehouse loads from these Gold zone files during its ETL batch process.
- At runtime, the Report Service queries only DuckDB — it never accesses operational databases or MinIO directly to serve user requests.

---

*See [Data Architecture Overview](overview.md) for database patterns and schema elements.*
*See [Caching Strategy](caching.md) for cache architecture and policies.*
*See [Data Warehouse](warehouse.md) for analytical schema design.*
*See [Domain Data Models](domain-models.md) for domain-specific data models including MDM, CDP, IoT, and other schemas.*
