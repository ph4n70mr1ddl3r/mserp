# Project Structure

## 1. Repository Layout

```
mserp/
├── Cargo.toml                    # Workspace root
├── Cargo.lock
├── docker-compose.yml            # Full local development (all services)
├── docker-compose.dev.yml        # Minimal dev (infra + core services only)
├── Makefile                      # Development commands
├── README.md
├── SPEC.md                       # Master spec index
│
├── docs/                         # Documentation
│   ├── README.md
│   ├── glossary.md
│   ├── 05-api/                    # API documentation
│   │   ├── endpoints.md
│   │   ├── error-codes.md
│   │   └── standards.md
│   ├── 01-architecture/           # Architecture documentation
│   │   └── overview.md
│   ├── 03-data/                   # Data documentation
│   │   ├── caching.md
│   │   ├── data-pipeline.md
│   │   ├── domain-models.md
│   │   └── overview.md
│   ├── 09-development/            # Development guides
│   │   ├── conventions.md
│   │   ├── local-setup.md
│   │   └── project-structure.md
│   ├── 04-events/                 # Event documentation
│   │   ├── catalog.md
│   │   ├── overview.md
│   │   └── sagas.md
│   ├── 07-features/               # Feature documentation
│   ├── 08-infrastructure/         # Infrastructure documentation
│   │   ├── deployment.md
│   │   ├── observability.md
│   │   └── technology.md
│   ├── 10-planning/               # Planning documentation
│   │   ├── nfr.md
│   │   └── phases.md
│   ├── 02-security/               # Security documentation
│   │   ├── authorization.md
│   │   ├── data-protection.md
│   │   ├── grc.md
│   │   ├── overview.md
│   │   └── threat-model.md
│   └── 06-services/               # Service documentation
│       ├── auth.md
│       ├── commerce.md
│       ├── config.md
│       ├── crm.md
│       ├── finance.md
│       ├── hr.md
│       ├── identity.md
│       ├── integration.md
│       ├── manufacturing.md
│       ├── overview.md
│       ├── platform.md
│       ├── project.md
│       ├── report.md
│       ├── tenant.md
│       └── workflow.md
│
├── crates/
│   ├── mserp-core/               # Shared core library
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── config.rs         # Configuration management
│   │       ├── error.rs          # Error types (RFC 7807 compatible)
│   │       ├── result.rs         # Result types
│   │       ├── pagination.rs     # Cursor pagination utilities
│   │       ├── i18n.rs           # Internationalization helpers
│   │       ├── money.rs          # Currency/money types (rust_decimal based)
│   │       ├── classification.rs # Data classification types
│   │       └── utils/            # Utilities
│   │
│   ├── mserp-auth/               # Auth domain types
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── domain/
│   │       ├── dto/
│   │       └── events/
│   │
│   ├── mserp-infrastructure/     # Infrastructure concerns
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── database/
│   │       │   ├── mod.rs
│   │       │   ├── pool.rs       # Connection pool with RLS support
│   │       │   ├── outbox.rs     # Transactional outbox
│   │       │   └── migration.rs  # Migration runner
│   │       ├── cache/
│   │       │   ├── mod.rs
│   │       │   └── redis.rs      # Cache-aside implementation
│   │       ├── messaging/
│   │       │   ├── mod.rs
│   │       │   ├── rabbitmq.rs   # RabbitMQ publisher/consumer
│   │       │   └── saga.rs       # Saga state tracking
│   │       └── tracing/
│   │           ├── mod.rs
│   │           ├── jaeger.rs     # OpenTelemetry setup
│   │           └── metrics.rs    # Prometheus metrics setup
│   │
│   └── mserp-proto/              # Protocol definitions
│       ├── Cargo.toml
│       └── src/
│           ├── lib.rs
│           └── events/           # Event type definitions for all services
│
│   └── mserp-migrations/        # Database migration runner
│       ├── Cargo.toml
│       └── src/
│           └── main.rs           # Migration CLI binary
│
├── services/
│   ├── auth-service/
│   │   ├── Cargo.toml
│   │   ├── Dockerfile
│   │   └── src/
│   │       ├── main.rs
│   │       ├── config.rs
│   │       ├── routes/
│   │       ├── handlers/
│   │       ├── models/
│   │       ├── repository/
│   │       └── services/
│   │
│   ├── identity-service/
│   ├── tenant-service/
│   ├── config-service/
│   ├── commerce-service/         # Sales + Inventory + PIM + Transportation
│   │   ├── src/
│   │   │   ├── main.rs
│   │   │   ├── config.rs
│   │   │   ├── sales/            # Sales domain module
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   ├── repository/
│   │   │   │   └── services/
│   │   │   ├── inventory/        # Inventory domain module
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   ├── repository/
│   │   │   │   └── services/
│   │   │   ├── pricing/          # Pricing engine module
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   └── services/
│   │   │   ├── pim/              # Product Information Management
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── transportation/   # Fleet & Transportation
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── atp/                # Available-to-Promise
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── configurator/       # Product Configurator
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── credit/             # Credit Management
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── subscriptions/      # Subscription Management
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── dropship/           # Intercompany Drop Ship
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── logistics/          # Connected Logistics & Track-and-Trace
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── b2b-portal/         # B2B Commerce Portal
│   │   │       ├── routes/
│   │   │       ├── handlers/
│   │   │       ├── models/
│   │   │       └── services/
│   │   │   ├── warranty/               # Warranty Management
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── cpq/                    # Configure Price Quote
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── retail-pos/             # Retail Point of Sale
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   └── property-management/    # Property Management
│   │   │       ├── routes/
│   │   │       ├── handlers/
│   │   │       ├── models/
│   │   │       └── services/
│   ├── finance-service/          # Finance Service (all finance, procurement, treasury, revenue, lease, grants, JV, collections)
│   │   ├── src/
│   │   │   ├── finance/          # Finance domain module
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   ├── repository/
│   │   │   │   └── services/
│   │   │   ├── procurement/      # Procurement domain module
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   ├── repository/
│   │   │   │   └── services/
│   │   │   ├── treasury/         # Corporate Treasury
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── expenses/         # Enterprise Expense Management
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── contracts/        # Contract Lifecycle Management
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── epm/              # Enterprise Performance Management
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── currency/         # Multi-currency module
│   │   │   │   └── services/
│   │   │   ├── revenue-recognition/ # Revenue Recognition (ASC 606/IFRS 15)
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── sourcing/            # Strategic Sourcing
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── supplier-risk/       # Supplier Risk Management
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── reconciliation/      # Account Reconciliation
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   └── profitability/       # Profitability Analysis
│   │   │       ├── routes/
│   │   │       ├── handlers/
│   │   │       ├── models/
│   │   │       └── services/
│   │   │   ├── lease-accounting/      # Lease Accounting (ASC 842 / IFRS 16)
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── grants/               # Grant Management
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── joint-venture/         # Joint Venture Accounting
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── intelligent-close/     # Intelligent Close
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── collections/           # Advanced Collections
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   └── advanced-tax-filing/    # Advanced Tax Filing
│   │   │       ├── routes/
│   │   │       ├── handlers/
│   │   │       ├── models/
│   │   │       └── services/
│   ├── hr-service/
│   │   ├── src/
│   │   │   ├── employee/
│   │   │   ├── attendance/
│   │   │   ├── leave/
│   │   │   ├── payroll/
│   │   │   │   ├── domestic/     # Single-country payroll
│   │   │   │   └── multi-country/ # Multi-country payroll localizations
│   │   │   ├── recruitment/
│   │   │   ├── onboarding/
│   │   │   ├── performance/
│   │   │   ├── talent/           # Talent review & succession planning
│   │   │   ├── workforce/        # Workforce modeling
│   │   │   ├── training/
│   │   │   ├── compensation/
│   │   │   └── benefits/
│   ├── manufacturing-service/
│   │   ├── src/
│   │   │   ├── bom/
│   │   │   ├── work-orders/
│   │   │   ├── planning/         # MPS, MRP
│   │   │   ├── ascp/             # Advanced Supply Chain Planning
│   │   │   ├── quality/          # Quality control + AQM + SPC
│   │   │   ├── maintenance/
│   │   │   ├── eam/              # Enterprise Asset Management
│   │   │   ├── work-centers/
│   │   │   ├── routing/
│   │   │   ├── cost-accounting/
│   │   │   ├── shop-floor/
│   │   │   ├── plm/              # Product Lifecycle Management
│   │   │   │   ├── revisions/
│   │   │   │   ├── eco/          # Engineering Change Orders
│   │   │   │   └── phase-management/
│   │   │   ├── sustainability/   # Energy, waste, emissions tracking
│   │   │   ├── iot/                 # IoT Integration
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── digital-twin/        # Digital Twin
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── manufacturing-intelligence/ # Manufacturing Intelligence
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   └── digital-thread/     # Digital Thread
│   │   │       ├── routes/
│   │   │       ├── handlers/
│   │   │       ├── models/
│   │   │       └── services/
│   ├── report-service/
│   │   ├── src/
│   │   │   ├── reports/
│   │   │   ├── dashboards/
│   │   │   ├── kpis/
│   │   │   ├── insights/         # AI-driven insights
│   │   │   ├── warehouse/        # DuckDB star schema
│   │   │   ├── data-lake/        # Data lake zone management
│   │   │   ├── esg/              # ESG / sustainability reporting
│   │   │   ├── carbon/           # Carbon accounting
│   │   │   ├── ml/               # Embedded ML/AI platform
│   │   │   │   ├── models/       # Model registry
│   │   │   │   ├── features/     # Feature store
│   │   │   │   └── training/     # Model training pipeline
│   │   │   ├── process-mining/       # Process Mining & Optimization
│   │   │   │   ├── discovery/
│   │   │   │   ├── conformance/
│   │   │   │   ├── simulation/
│   │   │   │   └── dashboards/
│   │   │   ├── cpm/                  # Corporate Performance Management
│   │   │   │   ├── okrs/
│   │   │   │   ├── scorecard/
│   │   │   │   ├── strategy-maps/
│   │   │   │   └── initiatives/
│   │   │   ├── narrative/            # Narrative Reporting
│   │   │   │   ├── packages/
│   │   │   │   ├── commentary/
│   │   │   │   └── xbrl/
│   │   │   └── augmented-analytics/  # Augmented Analytics
│   │   │       ├── nlq/
│   │   │       ├── insights/
│   │   │       └── discovery/
│   ├── workflow-service/
│   │   ├── src/
│   │   │   ├── engine/           # BPMN engine
│   │   │   ├── approvals/
│   │   │   ├── escalations/
│   │   │   ├── templates/
│   │   │   ├── sla/
│   │   │   ├── rules/            # Business rules engine
│   │   │   └── sod/              # Segregation of Duties checks
│   ├── crm-service/
│   │   ├── src/
│   │   │   ├── contacts/
│   │   │   ├── leads/
│   │   │   ├── opportunities/
│   │   │   ├── campaigns/
│   │   │   ├── activities/
│   │   │   ├── cases/            # Customer service cases
│   │   │   ├── analytics/
│   │   │   ├── field-service/      # Field Service Management
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── surveys/            # Survey & Feedback
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   ├── territories/        # Sales Territory & Quota
│   │   │   │   ├── routes/
│   │   │   │   ├── handlers/
│   │   │   │   ├── models/
│   │   │   │   └── services/
│   │   │   └── cdp/                  # Customer Data Platform
│   │   │       ├── routes/
│   │   │       ├── handlers/
│   │   │       ├── models/
│   │   │       └── services/
│   ├── project-service/
│   │   ├── src/
│   │   │   ├── projects/
│   │   │   ├── tasks/
│   │   │   ├── resources/
│   │   │   ├── timesheets/
│   │   │   ├── expenses/
│   │   │   ├── billing/
│   │   │   ├── risks/
│   │   │   ├── evm/              # Earned Value Management
│   │   │   ├── programs/         # Program management
│   │   │   └── construction-engineering/ # Construction & Engineering
│   │   │       ├── routes/
│   │   │       ├── handlers/
│   │   │       ├── models/
│   │   │       └── services/
│   ├── platform-service/         # Notification + File + Audit + Digital Assistant + App Builder + GRC
│   │   ├── src/
│   │   │   ├── notification/
│   │   │   │   ├── email/
│   │   │   │   ├── sms/
│   │   │   │   ├── push/
│   │   │   │   ├── websocket/    # Real-time notifications
│   │   │   │   └── webhooks/
│   │   │   ├── file/
│   │   │   │   ├── storage/
│   │   │   │   ├── preview/
│   │   │   │   └── ocr/          # Optical character recognition
│   │   │   ├── audit/
│   │   │   ├── assistant/        # Digital Assistant
│   │   │   │   ├── nlp/          # NLP intent recognition
│   │   │   │   ├── dialog/       # Multi-turn dialog management
│   │   │   │   └── actions/      # Service API dispatch
│   │   │   ├── app-builder/      # Low-code Application Builder
│   │   │   │   ├── designer/     # Page designer
│   │   │   │   ├── objects/      # Custom business objects
│   │   │   │   └── publishing/   # App versioning and publishing
│   │   │   ├── grc/              # Governance, Risk & Compliance
│   │   │   │   ├── sod/          # Segregation of Duties
│   │   │   │   ├── risk/         # Risk register
│   │   │   │   ├── compliance/   # Compliance assessments
│   │   │   │   └── incidents/    # GRC incidents
│   │   │   ├── data-masking/     # Data masking and subsetting
│   │   │   ├── scheduler/          # Enterprise Job Scheduler
│   │   │   │   ├── jobs/
│   │   │   │   ├── triggers/
│   │   │   │   └── monitoring/
│   │   │   ├── knowledge/          # Knowledge Management
│   │   │   │   ├── articles/
│   │   │   │   ├── categories/
│   │   │   │   ├── search/
│   │   │   │   └── analytics/
│   │   │   ├── signatures/         # Digital Signatures
│   │   │   │   ├── workflows/
│   │   │   │   ├── audit/
│   │   │   │   └── providers/
│   │   │   ├── self-service/       # Employee Self-Service Portal
│   │   │   │   ├── portal/
│   │   │   │   ├── dashboard/
│   │   │   │   └── preferences/
│   │   │   ├── rpa/                  # Robotic Process Automation
│   │   │   │   ├── designer/
│   │   │   │   ├── execution/
│   │   │   │   ├── actions/
│   │   │   │   └── monitoring/
│   │   │   ├── collaboration/        # Enterprise Collaboration
│   │   │   │   ├── messaging/
│   │   │   │   ├── documents/
│   │   │   │   ├── tasks/
│   │   │   │   └── search/
│   │   │   ├── iot-registry/         # IoT Device Registry
│   │   │   │   ├── devices/
│   │   │   │   ├── certificates/
│   │   │   │   └── telemetry/
│   │   │   ├── content/              # Enterprise Content Management
│   │   │   │   ├── repositories/
│   │   │   │   ├── lifecycle/
│   │   │   │   ├── records/
│   │   │   │   └── search/
│   │   │   ├── idp/                    # Intelligent Document Processing
│   │   │       ├── classification/
│   │   │       ├── extraction/
│   │   │       ├── training/
│   │   │       └── integration/
│   │   │   └── policy-automation/      # Policy Automation
│   │   │       ├── rules/
│   │   │       ├── assessments/
│   │   │       └── simulations/
│   └── integration-service/
│       ├── src/
│       │   ├── connectors/
│       │   ├── webhooks/
│       │   ├── edi/
│       │   ├── import-export/
│       │   ├── mdm/              # Master Data Management
│       │   │   ├── golden-records/
│       │   │   ├── matching/     # Deduplication engine
│       │   │   ├── quality/      # Data quality rules
│       │   │   └── stewardship/  # Data steward workflows
│       │   ├── governance/       # Data governance
│       │   │   ├── catalog/      # Data catalog
│       │   │   ├── lineage/      # Data lineage tracking
│       │   │   └── classification/
│       │   └── trade-compliance/   # Trade Compliance
│       │       ├── screening/      # Denied party screening
│       │       ├── classifications/ # Export control classification
│       │       ├── licenses/       # License management
│       │       └── customs/        # Customs documentation
│
├── migrations/
│   ├── auth/
│   ├── identity/
│   ├── tenant/
│   ├── config/
│   ├── commerce/
│   ├── finance/
│   ├── hr/
│   ├── manufacturing/
│   ├── report/
│   ├── workflow/
│   ├── crm/
│   ├── project/
│   ├── platform/
│   ├── audit/
│   └── integration/
│
├── infra/                          # Infrastructure as Code
│   ├── terraform/                  # Cloud infrastructure
│   │   ├── modules/
│   │   └── environments/
│   │       ├── production/
│   │       └── staging/
│   ├── helm/                       # Helm charts
│   └── k8s/                        # Kubernetes manifests
│       ├── base/
│       │   ├── deployments/
│       │   ├── services/
│       │   ├── configmaps/
│       │   ├── secrets/
│       │   ├── hpa/               # Horizontal Pod Autoscalers
│       │   └── pdb/               # Pod Disruption Budgets
│       └── overlays/
│           ├── development/
│           ├── staging/
│           └── production/
│
├── web/                      # React + TypeScript web application
│   ├── package.json
│   ├── tsconfig.json
│   ├── vite.config.ts
│   ├── src/
│   │   ├── components/       # Shared UI components
│   │   ├── pages/            # Route pages
│   │   ├── hooks/            # Custom React hooks
│   │   ├── services/         # API client services
│   │   ├── stores/           # Client-side state
│   │   └── utils/            # Utilities
│   └── tests/
├── mobile/                   # React Native mobile application
│   ├── package.json
│   ├── tsconfig.json
│   ├── app/                  # Expo Router screens
│   ├── components/           # Shared mobile components
│   ├── services/             # API client services
│   └── tests/
│
├── contracts/                    # Pact contract definitions
│   ├── consumers/
│   └── providers/
│
├── connectors/                   # Integration connector plugins
│   ├── connector-sdk/           # SDK for building connectors
│   ├── stripe/
│   ├── quickbooks/
│   └── ...
│
├── ml/                            # ML model artifacts
│   ├── models/                    # Trained model storage
│   ├── features/                  # Feature definitions
│   └── training/                  # Training scripts and configs
│
├── scripts/
│   ├── setup.sh
│   ├── migrate.sh
│   ├── seed.sh
│   ├── generate-openapi.sh
│   ├── mask-data.sh              # Generate masked data subset
│   └── validate-sod.sh           # Validate SoD rules
│
└── reference-data/               # Static seed data
    ├── currencies.csv
    ├── countries.csv
    ├── locales.csv
    ├── tax-jurisdictions.csv
    ├── uoms.csv
    ├── emission-factors.csv      # EPA/DEFRA emission factors
    ├── esg-frameworks.csv        # GRI, SASB, TCFD mappings
    ├── carrier-service-levels.csv
    ├── payroll-jurisdictions/    # Per-country payroll config
    ├── denied-party-lists/        # Trade compliance screening lists
    ├── export-classifications.csv # ECCN classifications
    ├── revenue-recognition-rules.csv # ASC 606/IFRS 15 standard methods
    ├── subscription-cycles.csv    # Standard billing cycle definitions
    ├── knowledge-categories.csv   # Knowledge base category seed data
    ├── iot-device-types.csv          # IoT device type definitions
    ├── process-activity-classes.csv  # Process mining activity classifications
    ├── rpa-bot-templates.csv         # RPA bot action templates
    ├── customer-segments.csv         # CDP default customer segment definitions
    ├── supplier-risk-indicators.csv  # Supplier risk indicator definitions
    ├── lease-classifications.csv      # ASC 842/IFRS 16 classification types
    ├── grant-funding-sources.csv      # Grant funding source types
    ├── jv-partner-types.csv           # Joint venture partner types
    ├── collection-strategy-templates.csv # Collection strategy templates
    ├── warranty-claim-reasons.csv     # Warranty claim reason codes
    └── idp-document-types.csv         # IDP document type definitions
```
