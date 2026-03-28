# Data Import/Export Framework

A standardized framework for importing and exporting data across all services, managed by the Integration Service.

| Format | Import | Export | Max Size |
|--------|--------|--------|----------|
| CSV | Yes | Yes | 100MB |
| JSON | Yes | Yes | 50MB |
| Excel (.xlsx) | Yes | Yes | 50MB |
| PDF | No | Yes (reports only) | N/A |
| XML | Yes | Yes | 50MB |
| Parquet | No | Yes (data lake export) | Unlimited |

Import operations are asynchronous — the client submits a file, receives a job ID, and polls for status. Validation errors are returned per-row with line numbers.

*See [Master Data Management](mdm.md) for data quality rules. See [Architecture Overview](../architecture/overview.md) for system context.*
