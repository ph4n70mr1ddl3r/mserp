# Data Warehouse (Report Service)

## 9. Data Warehouse (Report Service)

The Report Service maintains a DuckDB-based embedded data warehouse for analytical queries, separate from the operational PostgreSQL databases.

### 9.1 ETL Pipeline

```
Operational DBs (PostgreSQL) ──▶ ETL Jobs (scheduled) ──▶ DuckDB (Star Schema)
                                        │
                                        ├── Extract: Read-only queries on operational data
                                        ├── Transform: Denormalize, aggregate, compute measures
                                        └── Load: Upsert into fact/dimension tables
```

### 9.2 Schema Design

| Table Type | Examples | Refresh Frequency |
|-----------|----------|-------------------|
| Dimension tables | `dim_customer`, `dim_product`, `dim_date`, `dim_employee`, `dim_facility` | Hourly |
| Fact tables | `fact_sales`, `fact_inventory_movement`, `fact_payroll`, `fact_emissions` | Hourly |
| Aggregate tables | `agg_daily_sales`, `agg_monthly_revenue`, `agg_monthly_emissions` | Daily |

### 9.3 Data Freshness

- Real-time dashboards use event stream aggregation (not warehouse).
- Historical reports and trend analysis use the data warehouse.
- Warehouse data is at most 1 hour stale for dimension data and fact data.
- Aggregate tables are refreshed daily during off-peak hours.

---

*See [Data Architecture Overview](overview.md) for database patterns and schema elements.*
*See [Caching Strategy](caching.md) for cache architecture and policies.*
*See [Data Lake](data-lake.md) for raw and curated data zones.*
*See [Master Data Management Schema](mdm.md) for MDM and related schemas.*
*See [Domain Data Models](domain-models.md) for domain-specific data models.*
