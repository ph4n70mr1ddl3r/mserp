# Integration Service

| Aspect | Details |
|--------|---------|
| Port | 8021 |
| Database | `integration_db` |
| Responsibilities | External integrations, API management, API marketplace, connector framework, data import/export, MDM, data governance, event mesh |

## Modules

| Module | Description |
|--------|-------------|
| Connectors | Pre-built connectors: Stripe, PayPal, QuickBooks, Xero, Salesforce, HubSpot, Slack, Teams, government tax portals, shipping carriers (FedEx, UPS, DHL) |
| Connector SDK | Rust-based plugin framework for building custom connectors |
| Webhooks | Inbound webhook handling with signature verification, retry policy |
| EDI | EDIFACT, X12 support for B2B document exchange (POs, invoices, ASN) |
| API Versioning | External API gateway with version management, deprecation notices |
| Data Import | File-based import (CSV, JSON, Excel, XML), field mapping, validation, error reporting |
| Data Export | Scheduled exports, on-demand exports, FTP/SFTP delivery, Parquet for data lake |
| Integration Monitoring | Connection health, sync status, error rates, data volume tracking |
| OAuth2 Client | OAuth2 client credentials for connecting to external APIs |
| Master Data Management (MDM) | Golden record management, automated deduplication, data quality rules, matching engine, data stewardship workflows |
| Data Governance | Data catalog, lineage tracking, quality scorecards, classification and tagging, retention policies |
| Trade Compliance | Restricted party screening (OFAC, EU, UN denied party lists), export control classification, license management, customs documentation generation, compliance decision audit trail, dual-use goods screening |
| Compliance Screening Engine | Real-time screening on order creation, supplier onboarding, and shipment dispatch with configurable rules and alerting |
| API Marketplace | Centralized API catalog, developer portal, API key provisioning, usage analytics, API monetization |
| Event Mesh | Event gateway (HTTP-to-event bridge), topic hierarchy management, schema registry, event filtering |
| Developer Portal | Self-service portal for API consumers with sandbox access, documentation, and migration guides |

## See Also

- [Platform Service](platform.md)
- [Config Service](config.md)
- [Commerce Service](commerce.md)
- [Architecture Overview](../architecture/overview.md)
