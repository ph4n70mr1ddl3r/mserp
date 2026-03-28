# Master Data Management (MDM)

Master data is governed centrally by the Integration Service to ensure data quality and consistency across all services.

| Aspect | Implementation |
|--------|---------------|
| Master data domains | Customers, Suppliers, Products, Employees, Chart of Accounts |
| Golden record | Integration Service maintains the authoritative "golden record" per entity |
| Deduplication | Automated matching and merging based on configurable rules (fuzzy + exact) |
| Quality rules | Data quality scorecards with completeness, accuracy, consistency checks |
| Distribution | Master data changes propagated via events to consuming services |
| Data stewardship | Dashboard for manual review of matching conflicts and quality violations |

*See [Data Import/Export Framework](data-import-export.md) for bulk data operations. See [Architecture Overview](../architecture/overview.md) for system context.*
