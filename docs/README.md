# MSERP Documentation

## Architecture & Design

| Document | Description |
|----------|-------------|
| [Architecture Overview](architecture/overview.md) | High-level architecture, design principles, service topology, communication patterns |
| [Security Architecture](security/overview.md) | Authentication, authorization, data protection, threat model |
| [Event-Driven Architecture](events/overview.md) | Event bus, outbox pattern, sagas, event versioning |

## Services

| Document | Port | Description |
|----------|------|-------------|
| [Services Overview](services/overview.md) | — | Service categories, consolidation rationale, full service index |
| [Auth Service](services/auth.md) | 8001 | Authentication, JWT, MFA, SSO, session management |
| [Identity Service](services/identity.md) | 8002 | Users, roles, permissions, organizations |
| [Tenant Service](services/tenant.md) | 8003 | Multi-tenancy, feature flags, subscriptions |
| [Config Service](services/config.md) | 8004 | Dynamic configuration, CORS, localization |
| [Commerce Service](services/commerce.md) | 8010 | Sales, inventory, pricing, PIM, ATP/CTP, subscriptions |
| [Finance Service](services/finance.md) | 8011 | GL, AP/AR, procurement, treasury, revenue recognition |
| [HR Service](services/hr.md) | 8012 | Employee lifecycle, payroll, recruitment, talent |
| [Manufacturing Service](services/manufacturing.md) | 8013 | BOM, work orders, PLM, EAM, quality, IoT |
| [Report Service](services/report.md) | 8014 | Analytics, dashboards, ESG, ML/AI, process mining |
| [Workflow Service](services/workflow.md) | 8015 | BPMN engine, approvals, SLA management |
| [CRM / Marketing Service](services/crm.md) | 8016 | Contacts, leads, campaigns, field service, CDP |
| [Project Management Service](services/project.md) | 8017 | Projects, tasks, resources, EVM |
| [Platform Service](services/platform.md) | 8020 | Notifications, files, audit, assistant, GRC, scheduler, ITSM, compliance hub |
| [Integration Service](services/integration.md) | 8021 | Connectors, EDI, MDM, trade compliance |

## API & Data

| Document | Description |
|----------|-------------|
| [API Design Standards](api/standards.md) | REST standards, pagination, idempotency, versioning |
| [API Endpoints](api/endpoints.md) | Endpoint listings for all services |
| [Error Codes](api/error-codes.md) | Complete error code taxonomy |
| [Data Architecture](data/overview.md) | Database patterns, multi-tenancy, migrations, soft delete |
| [Caching Strategy](data/caching.md) | Redis cache-aside, policies, invalidation |
| [Data Warehouse](data/warehouse.md) | DuckDB star schema, ETL pipeline |
| [Data Lake](data/data-lake.md) | MinIO zones, schema-on-read |
| [MDM Schema](data/mdm.md) | Golden records, data quality, CDP schema |
| [Domain Data Models](data/domain-models.md) | Revenue recognition, lease, grants, JV, warranty schemas |
| [Event Catalog](events/catalog.md) | All domain events by service |
| [Saga Patterns](events/sagas.md) | Distributed transaction patterns |

## Feature Specifications

| Document | Description |
|----------|-------------|
| [Internationalization](features/i18n.md) | Locale resolution, RTL support, 40+ locales |
| [Multi-Currency](features/multi-currency.md) | Exchange rates, triangulation, gains/losses |
| [Pricing Engine](features/pricing-engine.md) | Price lists, discounts, promotions, formulas |
| [ATP / CTP](features/atp-ctp.md) | Available-to-Promise, Capable-to-Promise |
| [Product Configurator](features/product-configurator.md) | Constraint-based configuration |
| [Credit Management](features/credit-management.md) | Credit scoring, holds, release |
| [Data Import/Export](features/data-import-export.md) | File import/export, bulk data operations |
| [Subscriptions](features/subscriptions.md) | Recurring billing, lifecycle management |
| [Revenue Recognition](features/revenue-recognition.md) | ASC 606 / IFRS 15 compliance |
| [Lease Accounting](features/lease-accounting.md) | ASC 842 / IFRS 16 compliance |
| [ESG / Sustainability](features/esg.md) | Carbon accounting, emissions tracking |
| [Trade Compliance](features/trade-compliance.md) | Export controls, denied party screening |
| [IoT Integration](features/iot.md) | Device management, telemetry ingestion |
| [Digital Twin](features/digital-twin.md) | Asset twins, predictive maintenance |
| [Process Mining](features/process-mining.md) | Process discovery, conformance checking |
| [RPA](features/rpa.md) | Bot designer, task automation |
| [CDP](features/cdp.md) | Customer Data Platform, identity resolution |
| [Blockchain & Distributed Ledger](features/blockchain.md) | Supply chain provenance, smart contracts, cross-party settlement |
| [Enterprise Health & Safety (EHS)](features/health-safety.md) | Incident management, risk assessment, chemical management, OSHA/ISO 45001 |
| [Maintenance, Repair & Operations](features/mro.md) | MRO catalog, spare parts, repair orders, rotable management |
| [Customer Loyalty Management](features/loyalty.md) | Tiered programs, points engine, reward catalogs, coalition programs |
| [Advanced Tax Management](features/tax-management.md) | Multi-jurisdiction tax, external engine integration, nexus tracking |
| [CX Digital & A/B Testing](features/ab-testing.md) | A/B testing, multivariate testing, personalization |
| [Supplier Diversity Management](features/supplier-diversity.md) | Diversity classification, spend tracking, tier reporting |
| [Commodity Management & Trading](features/commodity-management.md) | Market pricing, hedging, futures, supply risk |
| [Advanced Spend Analysis](features/spend-analysis.md) | ML classification, maverick detection, savings identification |
| [Unified Omnichannel Management](features/omnichannel.md) | Cross-channel cart, order routing, click-and-collect |
| [AI-Driven Price Optimization](features/price-optimization.md) | Demand elasticity, dynamic pricing, markdown optimization |
| [IT Service Management](features/itsm.md) | Incident, problem, change, CMDB, service catalog |
| [Social Selling](features/social-selling.md) | Social enrichment, engagement tracking, relationship mapping |
| [Project Portfolio Analysis](features/project-portfolio.md) | Portfolio balancing, investment analysis, benefit realization |
| [Enterprise Contact Center](features/contact-center.md) | Omnichannel routing, IVR, workforce management |
| [Product Compliance & Regulatory](features/product-compliance.md) | Regulatory database, certifications, material compliance |
| [Compliance Hub](features/compliance-hub.md) | Unified compliance dashboard, regulatory change tracker |
| [Full feature index →](features/) | All feature specifications |

## Security

| Document | Description |
|----------|-------------|
| [Security Overview](security/overview.md) | Auth flow, JWT, service auth, CORS |
| [Authorization](security/authorization.md) | RBAC, ABAC, predefined roles |
| [Data Protection](security/data-protection.md) | Encryption, PII handling, key management |
| [GRC](security/grc.md) | SoD, risk management, compliance, privacy |
| [Threat Model](security/threat-model.md) | Threat categories, incident response |

## Infrastructure & Operations

| Document | Description |
|----------|-------------|
| [Deployment](infrastructure/deployment.md) | Kubernetes, CI/CD, health probes, multi-region |
| [Technology Stack](infrastructure/technology.md) | Rust crates, infrastructure, versioning |
| [Monitoring & Observability](infrastructure/observability.md) | Metrics, dashboards, logging, tracing, alerts |

## Development

| Document | Description |
|----------|-------------|
| [Project Structure](development/project-structure.md) | Repository layout |
| [Local Setup](development/local-setup.md) | Docker Compose, dev workflows |
| [Code Conventions](development/conventions.md) | Naming, error handling, module structure |

## Planning

| Document | Description |
|----------|-------------|
| [Development Phases](planning/phases.md) | Implementation roadmap and milestones |
| [Non-Functional Requirements](planning/nfr.md) | Performance, availability, scalability, compliance |

## Reference

| Document | Description |
|----------|-------------|
| [Glossary](glossary.md) | Terminology, abbreviations, references |
