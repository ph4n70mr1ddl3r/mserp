# Data Warehouse (Report Service)

## 1. Data Warehouse (Report Service)

The Report Service maintains a DuckDB-based embedded data warehouse for analytical queries, separate from the operational PostgreSQL databases.

### 1.1 ETL Pipeline

The Report Service **never reads from operational PostgreSQL databases directly**. All analytical data flows through the Data Lake (see [Data Lake](data-lake.md)). The ETL batch process extracts from the Data Lake Gold zone — pre-aggregated Parquet files stored in MinIO — and loads them into DuckDB.

```
Data Lake Gold Zone (MinIO Parquet) ──▶ ETL Jobs (scheduled) ──▶ DuckDB (Star Schema)
                                                  │
                                                  ├── Extract: Read Parquet files from Gold zone
                                                  ├── Transform: Denormalize, aggregate, compute measures
                                                  └── Load: Upsert into fact/dimension tables
```

**Important:** At runtime, the Report Service queries **only DuckDB**. It does not access operational databases or MinIO to serve user requests. The full end-to-end pipeline is:

```
Service Events → Outbox → RabbitMQ → Event Archive Worker → MinIO Raw (Bronze)
  → ETL → MinIO Curated (Silver) → Aggregation → MinIO Gold (Analytics)
  → ETL Load → DuckDB Warehouse (Star Schema)
```

### 1.2 Schema Design

| Table Type | Examples | Refresh Frequency |
|-----------|----------|-------------------|
| Dimension tables | `dim_customer`, `dim_product`, `dim_date`, `dim_employee`, `dim_facility` | Hourly |
| Fact tables | `fact_sales`, `fact_inventory_movement`, `fact_payroll`, `fact_emissions` | Hourly |
| Aggregate tables | `agg_daily_sales`, `agg_monthly_revenue`, `agg_monthly_emissions` | Daily |

### 1.3 Data Freshness

- Real-time dashboards use event stream aggregation (not warehouse).
- Historical reports and trend analysis use the data warehouse.
- Warehouse data is at most 1 hour stale for dimension data and fact data.
- Aggregate tables are refreshed daily during off-peak hours.

---

*See [Data Architecture Overview](overview.md) for database patterns and schema elements.*
*See [Caching Strategy](caching.md) for cache architecture and policies.*
*See [Data Lake](data-lake.md) for raw and curated data zones.*
*See [Domain Data Models](domain-models.md) for domain-specific data models including MDM, CDP, IoT, and other schemas.*
