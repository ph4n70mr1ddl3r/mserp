# Security Architecture

## 1. Authentication Flow

```
┌─────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Client │     │ API Gateway │     │ Auth Service│     │Identity Svc │
└────┬────┘     └──────┬──────┘     └──────┬──────┘     └──────┬──────┘
     │                 │                   │                   │
     │ POST /auth/login│                   │                   │
     │────────────────▶│                   │                   │
     │                 │ Validate creds    │                   │
     │                 │──────────────────▶│                   │
     │                 │                   │ Get user + roles  │
     │                 │                   │──────────────────▶│
     │                 │                   │◀──────────────────│
     │                 │◀──────────────────│                   │
     │  Access Token   │                   │                   │
     │  Refresh Token  │                   │                   │
     │◀────────────────│                   │                   │
     │                 │                   │                   │
     │ API Request     │                   │                   │
     │ + Bearer Token  │                   │                   │
     │────────────────▶│                   │                   │
     │                 │ Verify JWT        │                   │
     │                 │ (local validation)│                   │
     │                 │ Extract permissions│                   │
     │                 │ from JWT claims   │                   │
     │                 │                   │                   │
     │                 │ RBAC check        │                   │
     │                 │ (local — compare  │                   │
     │                 │  required perm    │                   │
     │                 │  against JWT perms)│                  │
     │                 │                   │                   │
     │  Response       │                   │                   │
     │◀────────────────│                   │                   │
```

## 2. Security Layers

| Layer | Controls |
|-------|----------|
| Network | mTLS, Network policies, DDoS protection, IP allowlisting per tenant, egress filtering |
| Transport | TLS 1.3, Certificate pinning, HSTS headers |
| Application | Input validation, SQL injection prevention (parameterized queries), XSS protection (output encoding), CSRF tokens for browser clients, Trade Compliance — screening at network entry, DDoS protection, IP allowlisting for trade compliance systems |
| Authentication | JWT with short expiry, Refresh tokens with rotation, MFA (TOTP, SMS, biometric), SSO (SAML 2.0 bridge), brute-force lockout, step-up authentication |
| Authorization | RBAC + ABAC with granular permissions, Row-level security (tenant + org scope), Segregation of Duties |
| Data | Encryption at rest (AES-256), PII masking in logs/responses, Audit logging, Data classification labels, Data masking for non-production, Trade compliance data residency controls, denied party list data protection |
| GRC | Compliance policy enforcement, risk register, control assessments, incident management, SoD conflict detection |

## 3. JWT Token Specification

| Claim | Access Token | Refresh Token |
|-------|-------------|---------------|
| `sub` | User UUID | User UUID |
| `tenant_id` | Tenant UUID | Tenant UUID |
| `roles` | Array of role strings | Not included |
| `permissions` | Array of permission strings | Not included |
| `org_scope` | Organization unit UUID (optional) | Not included |
| `exp` | 15 minutes | 7 days |
| `iat` | Issued at | Issued at |
| `jti` | Token UUID (unique) | Token UUID (unique) |
| `type` | `access` | `refresh` |

- Access tokens are verified locally by services (no network call to Auth Service) using the JWT public key (cached, rotated weekly).
- Token revocation is handled via Redis blocklist for access tokens and DB deletion for refresh tokens.
- Refresh token rotation: a new refresh token is issued on every refresh, and the old one is invalidated.
- JWT signing algorithm: `RS256` (asymmetric — services verify with public key, Auth signs with private key).

## 4. Service-to-Service Authentication

### 4.1 Overview

Internal service-to-service calls use a combination of **mTLS** for transport security and **Service JWTs** for identity.

### 4.2 Service Accounts

Each service has a dedicated service account:
- Credentials stored in Kubernetes secrets (auto-rotated every 90 days).
- Service account format: `svc-{service-name}` (e.g., `svc-commerce`, `svc-finance`).

### 4.3 Service JWT

| Claim | Value |
|-------|-------|
| `sub` | `svc-{service-name}` |
| `type` | `service` |
| `exp` | 1 hour |
| `jti` | Token UUID |
| `scope` | Array of allowed target services |

- Service JWTs are signed with a dedicated service signing key (separate from user JWTs).
- Services exchange Service JWTs when making internal HTTP calls.
- Services validate incoming Service JWTs by checking:
  1. Valid signature (using the shared service public key).
  2. `type` is `service`.
  3. `sub` is a known service account.
  4. Token is not expired.
  5. The receiving service's name is within the `scope` array.

### 4.4 Internal Call Patterns

| Scenario | Authentication Method |
|----------|----------------------|
| Service A calls Service B REST API | Service JWT in `Authorization: Bearer` header |
| Service A publishes event to RabbitMQ | mTLS + RabbitMQ SASL |
| Service A queries Service B database | NOT ALLOWED (database per service) |
| API Gateway routes to Service A | mTLS termination + user JWT forwarding |

### 4.5 mTLS Configuration

| Setting | Value |
|---------|-------|
| Certificate Authority | Internal CA managed by cert-manager |
| Certificate Validity | 90 days |
| Auto-Rotation | cert-manager with `Issuing` condition |
| CRL Check | Enabled |
| Cipher Suites | TLS 1.3 only |

## 5. CORS Policy

### 5.1 Configuration

CORS is enforced at the API Gateway layer. The configuration is tenant-specific and stored in the Config Service.

| Setting | Default | Description |
|---------|---------|-------------|
| Allowed Origins | Configured per tenant | Regex or explicit list |
| Allowed Methods | `GET, POST, PUT, PATCH, DELETE, OPTIONS` | HTTP methods |
| Allowed Headers | `Authorization, Content-Type, Idempotency-Key, X-Request-ID, Accept-Language, Time-Zone` | Request headers |
| Exposed Headers | `X-Request-ID, X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset` | Response headers |
| Allow Credentials | `true` | Required for cookies/auth headers |
| Max Age | 3600 seconds | Preflight cache duration |

### 5.2 Origin Management

| Environment | Allowed Origins |
|-------------|-----------------|
| Development | `http://localhost:*`, `http://127.0.0.1:*` |
| Staging | `https://staging-*.mserp.io` |
| Production | Configured per tenant via Config API (`GET/POST/DELETE /api/v1/config/cors/origins`) |

- Wildcard origins (`*`) are NEVER allowed in staging or production.
- Tenant Admins can manage their allowed origins via the Config API.
- Origin validation is case-insensitive and matches scheme + host + port.

### 5.3 Preflight Handling

- `OPTIONS` requests are handled entirely at the Gateway (no backend call).
- Preflight responses are cached per origin for `Max-Age` seconds.
- Non-allowed origins receive `403 Forbidden` with no CORS headers.

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

## 8. Data Protection

### 8.1 Encryption at Rest

| Data Type | Encryption Method |
|-----------|------------------|
| Database (PostgreSQL) | AES-256 via pgcrypto extension for PII columns; TDE for full-disk encryption |
| File Storage (MinIO) | Server-side encryption (SSE-S3) with AES-256 |
| Backups | AES-256 encrypted before writing to object storage |
| Redis | TLS in transit; persistence files encrypted at rest |
| Secrets | Kubernetes secrets (encrypted at rest via etcd encryption configuration) |
| Data Lake | SSE-S3 encryption on all zones (raw, curated, analytics) |

### 8.2 PII Handling

| Requirement | Implementation |
|-------------|---------------|
| Classification | Data classified as Public, Internal, Confidential, Restricted |
| Masking in logs | PII fields automatically redacted in structured logs (e.g., email → `j***@company.com`) |
| Masking in API responses | Restricted fields masked based on caller permissions |
| Right to access (GDPR) | API endpoint for data subject to retrieve all their data |
| Right to erasure (GDPR) | Hard delete with cascading; audit log retained with anonymized references |
| Data retention | Configurable per data category; automated purge jobs |
| Cross-border transfer | Data residency controls per tenant (region pinning) |
| Data subsetting | PII-compliant data subsets for non-production environments |

### 8.3 Key Management

| Aspect | Implementation |
|--------|---------------|
| Key Storage | HashiCorp Vault or cloud KMS (AWS KMS, Azure Key Vault — configurable) |
| Key Rotation | Automated rotation every 90 days for encryption keys |
| Key Hierarchy | Master key → Data encryption key → Per-table key |
| Access | Keys accessed via service accounts with audit logging |
| Key Backup | Split-key shamir backup for master key, stored in separate physical locations |

## 9. Governance, Risk & Compliance (GRC)

### 9.1 Segregation of Duties (SoD)

| Aspect | Implementation |
|--------|---------------|
| SoD Rules | Configurable rules defining incompatible role/permission combinations |
| Conflict Detection | Real-time detection on role assignment and workflow approval |
| Mitigation | Accept risk (with approval), assign compensating controls, or enforce separation |
| Reporting | SoD conflict report, mitigation status, outstanding violations |
| Integration | SoD checks integrated into Workflow Service approval flows |

### 9.2 Risk Management

| Aspect | Implementation |
|--------|---------------|
| Risk Register | Centralized risk register with risk scoring (likelihood × impact) |
| Risk Categories | Financial, Operational, Compliance, Strategic, Technology |
| Assessments | Scheduled and ad-hoc risk assessments with questionnaire workflows |
| Treatment Plans | Mitigate, Transfer, Accept, Avoid with action items and deadlines |
| Monitoring | Risk KRI (Key Risk Indicator) dashboards with threshold alerts |

### 9.3 Compliance Management

| Framework | Controls |
|-----------|----------|
| SOC 2 Type II | Audit logging, access controls, encryption, incident response |
| GDPR | Data erasure, data portability, consent management, DPA, privacy impact assessment |
| HIPAA | PHI encryption, access logging, BAAs, minimum necessary access |
| PCI DSS | Tokenization, network segmentation, vulnerability scanning |
| SOX | Financial audit trail, change management, access controls, SoD |
| ISO 27001 | Information security management system, risk assessment, controls |
| ESG Regulations | Emissions reporting accuracy, audit trail for sustainability data |

### 9.4 Trade Compliance

| Aspect | Implementation |
|--------|---------------|
| Denied Party Screening | Automated screening against OFAC SDN, EU Consolidated List, UN Security Council, and other government denied party lists |
| Export Control Classification | Product classification per ECCN (US), EU Dual-Use, and other jurisdictional regimes |
| License Management | Export license tracking with expiration alerts, usage monitoring, and exception management |
| Customs Compliance | Automated generation of customs declarations, commercial invoices, packing lists, certificates of origin |
| Screening Triggers | Real-time screening on: customer creation, order submission, supplier onboarding, shipment dispatch |
| Sanctions Screening | Continuous re-screening of existing customer/supplier base against updated lists |
| Audit Trail | Complete log of all screening decisions, matches, overrides, and false positive dispositions |

## 10. Threat Model

| Threat Category | Mitigation |
|----------------|------------|
| Brute-force login | Account lockout after 5 failures (15-minute cooldown), IP-based rate limiting |
| Token theft | Short-lived access tokens (15 min), refresh rotation, Redis blocklist |
| SQL injection | Parameterized queries (SeaORM), input validation |
| XSS | Output encoding, Content-Security-Policy headers |
| CSRF | Same-site cookies, CSRF tokens for state-changing operations |
| DDoS | API Gateway rate limiting, CDN-based DDoS protection |
| Insider threat | Audit logging, separation of duties, privileged access management, SoD enforcement |
| Supply chain | Container image scanning (Trivy), dependency auditing (cargo audit), image signing (Cosign), SBOM |
| Data exfiltration | Encryption at rest, network policies, egress filtering, data masking |
| Privilege escalation | Step-up authentication, SoD conflict detection, least-privilege defaults |
| API abuse | Rate limiting per tenant/user/endpoint, API key rotation, request validation |
| Model poisoning | ML model integrity verification, signed models, input sanitization for AI endpoints |
| Trade compliance violation | Automated denied party screening on all transactions, real-time list updates, override audit trail |
| Data residency violation | Connection routing enforcement, RLS policies, cross-border transfer audit logging |
| AI/ML model bias | Bias testing in model training, fairness metrics, regular audit, diverse training data |
| Subscription fraud | Usage anomaly detection, automated billing reconciliation, credit card verification |
| Trade Compliance | Export control evasion — without proper license, trade compliance data residency violation |
| Trade compliance fraud | Fraud detection in trade compliance screening results |
| AI/ML model fairness | Fairness violation monitoring in AI/ML outputs, privacy regulations compliance |
| Supply chain (ML models) | 3rd-party dependency audit for trade compliance lists (Trivy, cargo audit) |
| Trade compliance | Cross-border export control enforcement, customs documentation inaccuracy |
| Screening evasion | Persistent monitoring and logging of screening attempts and alerting |

## 11. Security Incident Response

| Phase | Actions | SLA |
|-------|---------|-----|
| Detection | Automated alerting (SIEM), anomaly detection, user reports | < 15 min to alert |
| Triage | Severity assessment, scope determination, incident ticket creation | < 30 min |
| Containment | Isolate affected systems, revoke compromised credentials, block IPs | < 1 hour |
| Eradication | Remove threat, patch vulnerability, rotate all affected secrets | < 4 hours |
| Recovery | Restore from backup if needed, verify system integrity, resume operations | < 8 hours |
| Post-mortem | Root cause analysis, lessons learned, preventive measures, documentation | < 48 hours |
| Trade compliance violation | Triage → Containment → Review logs for denied party list match; Eradication → Patch vulnerability → Update denied party lists | < 4 hours |
| Trade compliance | Triage → Review logs, check screening history, notify affected parties | < 24 hours |

## 12. Privacy Impact Assessment (PIA)

A PIA MUST be conducted before:
- Introducing new data processing activities involving PII
- Adding new data sharing arrangements with third parties
- Deploying new AI/ML models that process personal data
- Expanding data collection to new categories of personal data
- Entering new jurisdictions with different privacy requirements
- Implementing trade compliance screening that processes customer/supplier entity data against government watchlists
- Deploying new trade compliance screening capabilities that involve denied party lists
- Processing cross-border shipments with trade compliance screening results (which may affect data residency)

PIA outcomes are recorded in the GRC module and linked to the data catalog.

---

*See [API Design](api-design.md) for rate limiting and idempotency details.*
*See [Data Architecture](data-architecture.md) for row-level security and encryption at rest.*
