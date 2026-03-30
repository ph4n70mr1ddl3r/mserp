# Authorization (RBAC + ABAC)

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

```
{domain}.{entity}.{action}
```

- `{domain}`: Service domain — `auth`, `identity`, `tenant`, `config`, `commerce`, `finance`, `hr`, `manufacturing`, `report`, `workflow`, `platform`, `integration`, `crm`, `project`
- `{entity}`: `order`, `customer`, `product`, `journal`, `employee`, etc.
- `{action}`: `create`, `read`, `update`, `delete`, `own` (self-scoped read/write on own records only), `approve`, `export`, `*` (wildcard for all actions)

> **Note:** Permissions use the **service name** as the domain (e.g., `commerce.order.create`), not the internal module name. For legacy compatibility, `sales.*` → `commerce.*`, `inventory.*` → `commerce.*`, `procurement.*` → `finance.*`.

### 6.2 Attribute-Based Access Control (ABAC)

In addition to RBAC, the system supports attribute-based policies for fine-grained access:

| Attribute | Source | Example |
|-----------|--------|---------|
| User's organization unit | Identity Service | `org_unit = 'North America Sales'` |
| Resource's department | Data record | `order.department = 'NA-Sales'` |
| Amount threshold | Request data | `amount > 10000` requires manager approval |
| Time of day | System clock | `outside business hours` triggers additional MFA |
| IP range | Request source | `source_ip NOT IN corporate_range` triggers step-up auth |
| Data classification | Resource metadata | `classification = 'Restricted'` limits access to named users |
| Device trust level | Auth Service | `unmanaged device` triggers step-up auth |

ABAC policies are evaluated by the service layer (not the gateway) since they require access to resource attributes.

### 6.3 Authorization Flow

1. User makes request with JWT.
2. API Gateway extracts `permissions` from JWT (embedded by Auth Service at login).
3. Gateway checks if the required permission for the endpoint is in the user's permission set (local comparison, no network call).
4. If authorized, request is forwarded with `X-User-Permissions` and `X-User-Org-Scope` headers.
5. Service applies additional row-level checks (tenant isolation via RLS, org scope filtering, amount-based rules).

> **Note:** Permission changes take effect after the user's next token refresh (access token expires in 15 minutes). This is an acceptable trade-off for eliminating the latency of a per-request authorization call.

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

> **Note:** Entity-level wildcards are supported (e.g., `report.*.read` grants read access to all entities within the report domain).

---

- [Security Architecture](./overview.md)
- [Data Protection](./data-protection.md)
- [Governance, Risk & Compliance (GRC)](./grc.md)
- [Threat Model & Incident Response](./threat-model.md)
