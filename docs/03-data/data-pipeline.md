# Data Pipeline

Report Service manages the complete analytical pipeline from event ingestion to warehouse queries. Report Service **never reads from operational PostgreSQL databases directly**.

## 1. End-to-End Pipeline

```
Service Events → Outbox Tables → RabbitMQ → Event Archive Worker → MinIO Raw (Bronze)
  → ETL (validate, deduplicate) → MinIO Curated (Silver)
  → Aggregation Jobs → MinIO Gold (Analytics)
  → ETL Load → DuckDB Warehouse (Star Schema)
```

At runtime, Report Service queries **only DuckDB**. It does not access operational databases or MinIO to serve user requests.

## 2. Data Lake (MinIO)

| Zone | Purpose | Format | Retention |
|------|---------|--------|-----------|
| Raw (Bronze) | Unprocessed event and operational data | JSON, Parquet | 2 years |
| Curated (Silver) | Cleaned, deduplicated, typed data | Parquet | Indefinite |
| Analytics (Gold) | Pre-aggregated, business-ready datasets | Parquet | Indefinite |

- Ingestion: Outbox poller publishes events to RabbitMQ. Separate Event Archive Worker consumes and writes to MinIO raw zone.
- ETL jobs promote raw to curated with schema validation and deduplication.
- Analytics zone populated by scheduled aggregation jobs.
- Schema-on-read: Raw zone has no enforced schema; curated zone uses Apache Arrow schema definitions.
- Query access via DuckDB's S3/Parquet integration or direct API.
- The Data Lake is the **sole data source** for the warehouse.

## 3. Data Warehouse (DuckDB)

| Table Type | Examples | Refresh Frequency |
|-----------|----------|-------------------|
| Dimension tables | `dim_customer`, `dim_product`, `dim_date`, `dim_employee`, `dim_facility` | Hourly |
| Fact tables | `fact_sales`, `fact_inventory_movement`, `fact_payroll`, `fact_emissions` | Hourly |
| Aggregate tables | `agg_daily_sales`, `agg_monthly_revenue`, `agg_monthly_emissions` | Daily |

- ETL batch process extracts from Gold zone Parquet files, transforms (denormalize, aggregate, compute measures), and loads into DuckDB star schema.
- Real-time dashboards use event stream aggregation, NOT warehouse.
- Historical reports and trend analysis use the warehouse.
- Max 1 hour stale for dimension and fact data. Aggregate tables refreshed daily during off-peak.

---

*See [Data Architecture Overview](overview.md) for database patterns and schema elements.*
*See [Caching Strategy](caching.md) for cache architecture and policies.*
*See [Domain Data Models](domain-models.md) for domain-specific data models.*
