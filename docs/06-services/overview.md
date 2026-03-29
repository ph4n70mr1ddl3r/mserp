# Microservices Overview

The platform is composed of 14 microservices organized into 3 categories: Core, Business, and Supporting. Each service owns its database and communicates via asynchronous events (RabbitMQ) and synchronous REST/gRPC calls through an API Gateway.

## Service Categories

| Category | Port Range | Services |
|----------|-----------|----------|
| Core | 8001–8009 | Auth, Identity, Tenant, Config |
| Business | 8010–8019 | Commerce, Finance, HCM, Manufacturing, Report, Workflow, CRM, Project |
| Supporting | 8020–8029 | Platform, Integration |

## Service Consolidation Rationale

Multiple functional areas were consolidated into single services to reduce operational complexity, simplify inter-service communication, and enable cohesive transactional boundaries. See [Architecture — Service Consolidation](../01-architecture/overview.md#4-service-consolidation-rationale) for the detailed rationale.

## All Services

| Service | Port | Database | Key Responsibilities | File |
|---------|------|----------|---------------------|------|
| Auth | 8001 | `auth_db` | Authentication, token management, session handling | [auth.md](auth.md) |
| Identity | 8002 | `identity_db` | User management, roles, permissions, organizations | [identity.md](identity.md) |
| Tenant | 8003 | `tenant_db` | Multi-tenancy, tenant provisioning, feature flags | [tenant.md](tenant.md) |
| Config | 8004 | `config_db` | System configuration, settings management | [config.md](config.md) |
| Commerce | 8010 | `commerce_db` | Sales, inventory, pricing, PIM, transportation, ATP/CTP, product configurator, credit management, subscriptions, B2B portal, loyalty, omnichannel, price optimization | [commerce.md](commerce.md) |
| Finance | 8011 | `finance_db` | Financial accounting, procurement, treasury, EPM, CLM, revenue recognition, strategic sourcing, tax management, commodity management, spend analysis, supplier diversity | [finance.md](finance.md) |
| HCM | 8012 | `hr_db` | Human resources, payroll, workforce management, talent management | [hr.md](hr.md) |
| Manufacturing | 8013 | `manufacturing_db` | Production, quality, PLM, EAM, manufacturing intelligence, digital thread, IoT, digital twin, EHS, MRO, product compliance | [manufacturing.md](manufacturing.md) |
| Report | 8014 | `report_db` | Analytics, reporting, dashboards, BI, AI/ML, ESG, narrative reporting, process mining, CPM | [report.md](report.md) |
| Workflow | 8015 | `workflow_db` | Business process automation, approvals, BPM engine, SLA management | [workflow.md](workflow.md) |
| CRM | 8016 | `crm_db` | Customer relationship management, marketing automation, field service, CDP, adaptive intelligence, contact center, social selling, A/B testing | [crm.md](crm.md) |
| Project | 8017 | `project_db` | Project planning, execution, resource management, project accounting | [project.md](project.md) |
| Platform | 8020 | `platform_db`, `audit_db` | Notifications, file storage, audit logging, digital assistant, app builder, GRC, RPA, ECM, ITSM, compliance hub | [platform.md](platform.md) |
| Integration | 8021 | `integration_db` | External integrations, API management, connectors, MDM, data governance, event mesh, blockchain integration | [integration.md](integration.md) |
