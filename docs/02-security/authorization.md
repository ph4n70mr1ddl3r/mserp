# Authorization (RBAC + ABAC)

> Authorization rules (RBAC permission format, ABAC attributes, authorization flow) are defined in SPEC.md §11.3–11.5. This document provides the full role-to-permission mapping.

## 6. RBAC + ABAC Model

```
┌─────────────────────────────────────────────────────────────┐
│                      RBAC + ABAC Model                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   User ──────▶ UserGroup ──────▶ Role ──────▶ Permission    │
│       │                                      + Attributes   │
│       └──▶ Org Unit (for scope filtering)                    │
│                                                              │
│   ┌─────────────────────────────────────────────────────┐   │
│   │ Example:                                            │   │
│   │                                                      │   │
│   │ User: john@company.com                              │   │
│   │   └─▶ Groups: [Sales Team]                          │   │
│   │         └─▶ Roles: [Sales Manager]                  │   │
│   │               └─▶ Permissions:                      │   │
│   │                    - commerce.order.create           │   │
│   │                    - commerce.order.read             │   │
│   │                    - commerce.order.update           │   │
│   │                    - commerce.quotation.*            │   │
│   │                    - commerce.stock.read             │   │
│   │   └─▶ Org Unit: North America Sales                 │   │
│   │         (Row-level filter: region = 'NA')            │   │
│   └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### 6.1 Permission Format

Permission format is `{domain}.{entity}.{action}` per SPEC.md §11.3.

### 6.2 Attribute-Based Access Control (ABAC)

ABAC attributes are defined in SPEC.md §11.4.

### 6.3 Authorization Flow

Authorization flow is defined in SPEC.md §11.5.

## 7. Predefined Roles

| Role | Description | Key Permissions |
|------|-------------|-----------------|
| Super Admin | Full system access across all tenants | `*` |
| Tenant Admin | Tenant-level admin | `tenant.*`, `identity.*`, `config.*` |
| Finance Manager | Financial operations | `finance.*`, `report.finance.*` |
| Accountant | Accounting operations | `finance.journal.*`, `finance.accounts.*` |
| Treasury Manager | Treasury operations | `finance.treasury.*`, `finance.payment.*` |
| Sales Manager | Sales operations | `commerce.*` |
| Sales Rep | Limited sales | `commerce.order.create`, `commerce.quotation.*`, `commerce.customer.read` |
| Inventory Manager | Stock operations | `commerce.stock.*`, `commerce.warehouse.*` |
| Procurement Manager | Procurement operations | `finance.purchase-order.*`, `finance.supplier.*`, `finance.contract.*` |
| HR Manager | HR operations | `hr.*` |
| HR Specialist | Recruitment & onboarding | `hr.recruitment.*`, `hr.employee.read`, `hr.onboarding.*` |
| Employee | Self-service | `hr.attendance.own`, `hr.leave.own`, `hr.payroll.own`, `finance.expense.own` |
| Project Manager | Project management | `project.*`, `hr.employee.read` |
| CRM User | Sales & marketing | `crm.*`, `commerce.customer.read` |
| Manufacturing Manager | Production operations | `manufacturing.*` |
| Quality Manager | Quality control | `manufacturing.quality.*`, `manufacturing.bom.read` |
| Report Viewer | Read-only analytics | `report.*.read` |
| ESG Analyst | Sustainability reporting | `report.esg.*`, `report.carbon.*` |
| GRC Officer | Governance, risk, compliance | `platform.grc.*`, `platform.audit.read` |
| App Builder | Low-code application development | `platform.app-builder.*` |
| Subscription Manager | Subscription lifecycle management | `commerce.subscription.*`, `commerce.order.read`, `finance.invoice.read` |
| Field Service Manager | Field service operations | `crm.service-order.*`, `crm.case.read`, `commerce.stock.read` |
| Trade Compliance Officer | Trade compliance screening | `integration.trade-compliance.*`, `integration.screening.*` |
| Knowledge Manager | Knowledge base management | `platform.knowledge.*` |
| Data Privacy Officer | Privacy management and GDPR compliance | `platform.privacy.*`, `platform.grc.audit.read`, `platform.data-masking.*` |
| Content Manager | Enterprise content and records management | `platform.content.*`, `platform.file.*` |
| DLP Analyst | Data loss prevention monitoring | `platform.dlp.*`, `platform.audit.read` |
| IoT Manager | IoT device and digital twin operations | `manufacturing.iot.*`, `manufacturing.digital-twin.*` |
| Collaboration Admin | Team collaboration administration | `platform.collaboration.*` |
| Logistics Manager | Connected logistics operations | `commerce.logistics.*`, `commerce.shipment.*` |
| Lease Accountant | Lease accounting operations | `finance.lease.*`, `finance.asset.read` |
| Grant Manager | Grant lifecycle management | `finance.grant.*`, `report.grant.*` |
| Joint Venture Accountant | Joint venture accounting | `finance.joint-venture.*`, `finance.gl.read` |
| Collections Specialist | Accounts receivable collections | `finance.collections.*`, `finance.invoice.read`, `commerce.customer.read` |
| Warranty Manager | Warranty policy and claims management | `commerce.warranty.*`, `manufacturing.quality.read` |
| Tax Manager | Tax determination and reporting | `finance.tax.*`, `report.tax.*` |
| Commodity Manager | Commodity procurement and trading | `finance.commodity.*`, `finance.purchase-order.*` |
| Spend Analyst | Spend analysis and procurement analytics | `report.spend.*`, `finance.supplier.read` |
| Diversity Program Manager | Supplier diversity program management | `finance.diversity.*`, `finance.supplier.read`, `report.diversity.*` |
| Loyalty Manager | Customer loyalty program management | `commerce.loyalty.*`, `commerce.customer.read` |
| Contact Center Agent | Contact center operations | `crm.contact-center.*`, `crm.case.*`, `commerce.customer.read` |
| Contact Center Manager | Contact center management | `crm.contact-center.*`, `crm.case.*`, `report.contact-center.*` |
| Social Seller | Social selling activities | `crm.social-selling.*`, `crm.contact.read`, `crm.opportunity.read` |
| CX Testing Manager | CX digital testing and personalization | `crm.ab-testing.*`, `crm.campaign.*` |
| EHS Manager | Enterprise health and safety | `manufacturing.ehs.*`, `manufacturing.quality.*`, `report.ehs.*` |
| EHS Specialist | Safety incident and inspection management | `manufacturing.ehs.incident.*`, `manufacturing.ehs.inspection.*` |
| MRO Manager | MRO operations management | `manufacturing.mro.*`, `commerce.stock.read`, `finance.purchase-order.read` |
| Product Compliance Manager | Product regulatory compliance | `manufacturing.compliance.*`, `report.compliance.*` |
| IT Service Agent | IT service desk operations | `platform.itsm.incident.*`, `platform.itsm.request.*`, `platform.knowledge.read` |
| IT Service Manager | IT service management | `platform.itsm.*`, `report.itsm.*` |
| Omnichannel Manager | Omnichannel operations | `commerce.omnichannel.*`, `commerce.order.*`, `commerce.stock.read` |
| Price Analyst | Price optimization and analysis | `commerce.pricing.*`, `report.pricing.*` |
| Portfolio Manager | Project portfolio management | `project.portfolio.*`, `project.project.read`, `report.project.*` |
| Compliance Hub Manager | Unified compliance oversight | `platform.compliance-hub.*`, `platform.grc.read`, `report.compliance.*` |
| Career Development Manager | Career development operations | `hr.career.*`, `hr.competency.*`, `hr.goal.read` |
| Production Scheduler | Production scheduling | `manufacturing.schedule.*`, `manufacturing.mes.read`, `manufacturing.work-order.read` |
| Storefront Manager | B2C commerce storefront | `commerce.storefront.*`, `commerce.promotion.*`, `commerce.review.*` |
| MES Operator | MES operations | `manufacturing.mes.*`, `manufacturing.work-order.read` |
| Extension Developer | Application composer | `platform.composer.*` |
| ML Studio User | Self-service ML | `report.ml-studio.*`, `report.data-lake.read` |
| Access Certifier | Access certification reviews | `platform.grc.certification.*`, `identity.role.read` |
| S&OP Planner | Sales and operations planning | `commerce.sop.*`, `commerce.demand.read`, `report.planning.read` |
| Channel Revenue Manager | Channel revenue management | `commerce.channel-revenue.*`, `commerce.partner.read` |
| Tax Provision Analyst | Tax provision and reporting | `finance.tax-provision.*`, `report.tax.read` |
| Shared Services Specialist | Shared services center operations | `finance.shared-service.*`, `finance.invoice.read`, `finance.payment.read` |
| IT Financial Manager | IT financial management | `finance.it-financial.*`, `report.it-financial.read` |
| Opportunity Marketplace Manager | Internal opportunity marketplace | `hr.opportunity.*`, `hr.employee.read` |
| Skills Administrator | Dynamic skills management | `hr.skills.*`, `hr.competency.*` |
| Workforce Safety Officer | Workforce health and safety | `hr.safety.*`, `hr.incident.*` |
| Shift Scheduler | Workforce shift scheduling | `hr.scheduling.*`, `hr.shift.*`, `hr.attendance.read` |
| Employee Experience Manager | Employee experience management | `hr.experience.*`, `hr.feedback.*` |
| Contract Manufacturing Coordinator | Contract manufacturing coordination | `manufacturing.contract-mfg.*`, `manufacturing.work-order.read` |
| Innovation Manager | Innovation management | `manufacturing.innovation.*`, `manufacturing.idea.*` |
| Sales Compensation Analyst | Sales compensation analysis | `crm.compensation.*`, `crm.quota.read` |
| Depot Repair Technician | Depot repair operations | `crm.depot-repair.*`, `commerce.stock.read` |
| Guided Learning Author | Guided learning content authoring | `platform.guided-learning.*`, `platform.content.write` |
| Warehouse Manager | Warehouse management operations | `commerce.warehouse.*`, `commerce.stock.*` |
| Transportation Manager | Transportation management operations | `commerce.transportation.*`, `commerce.shipment.*` |
| Global Trade Compliance Manager | Global trade compliance management | `integration.trade.*`, `integration.compliance.*` |
| Marketing Automation Manager | Marketing automation management | `crm.campaign.*`, `crm.lead.*`, `crm.segment.*` |
| Subledger Accountant | Subledger accounting operations | `finance.sla.*`, `finance.journal.*` |

> **Note:** Entity-level wildcards are supported (e.g., `report.*.read` grants read access to all entities within the report domain).

---

- [Security Architecture](./overview.md)
- [Data Protection](./data-protection.md)
- [Governance, Risk & Compliance (GRC)](./grc.md)
- [Threat Model & Incident Response](./threat-model.md)
