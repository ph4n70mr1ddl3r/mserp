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
| Network | mTLS, Network policies, DDoS protection, IP allowlisting per tenant |
| Transport | TLS 1.3, Certificate pinning, HSTS headers |
| Application | Input validation, SQL injection prevention (parameterized queries), XSS protection (output encoding), CSRF tokens for browser clients |
| Authentication | JWT with short expiry, Refresh tokens with rotation, MFA (TOTP, SMS), SSO (SAML 2.0 bridge), brute-force lockout |
| Authorization | RBAC + ABAC with granular permissions, Row-level security (tenant + org scope) |
| Data | Encryption at rest (AES-256), PII masking in logs/responses, Audit logging, Data classification labels |

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
| Sales Manager | Sales operations | `commerce.*` |
| Sales Rep | Limited sales | `commerce.order.create`, `commerce.quotation.*`, `commerce.customer.read` |
| Inventory Manager | Stock operations | `commerce.stock.*`, `commerce.warehouse.*` |
| HR Manager | HR operations | `hr.*` |
| HR Specialist | Recruitment & onboarding | `hr.recruitment.*`, `hr.employee.read`, `hr.onboarding.*` |
| Employee | Self-service | `hr.attendance.own`, `hr.leave.own`, `hr.payroll.own` |
| Project Manager | Project management | `project.*`, `hr.employee.read` |
| CRM User | Sales & marketing | `crm.*`, `commerce.customer.read` |
| Manufacturing Manager | Production operations | `manufacturing.*` |
| Report Viewer | Read-only analytics | `report.*.read` |

## 8. Data Protection

### 8.1 Encryption at Rest

| Data Type | Encryption Method |
|-----------|------------------|
| Database (PostgreSQL) | AES-256 via pgcrypto extension for PII columns; TDE for full-disk encryption |
| File Storage (MinIO) | Server-side encryption (SSE-S3) with AES-256 |
| Backups | AES-256 encrypted before writing to object storage |
| Redis | TLS in transit; persistence files encrypted at rest |
| Secrets | Kubernetes secrets (encrypted at rest via etcd encryption configuration) |

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

### 8.3 Key Management

| Aspect | Implementation |
|--------|---------------|
| Key Storage | HashiCorp Vault or AWS KMS (configurable) |
| Key Rotation | Automated rotation every 90 days for encryption keys |
| Key Hierarchy | Master key → Data encryption key → Per-table key |
| Access | Keys accessed via service accounts with audit logging |

## 9. Threat Model

| Threat Category | Mitigation |
|----------------|------------|
| Brute-force login | Account lockout after 5 failures (15-minute cooldown), IP-based rate limiting |
| Token theft | Short-lived access tokens (15 min), refresh rotation, Redis blocklist |
| SQL injection | Parameterized queries (SeaORM), input validation |
| XSS | Output encoding, Content-Security-Policy headers |
| CSRF | Same-site cookies, CSRF tokens for state-changing operations |
| DDoS | API Gateway rate limiting, CDN-based DDoS protection |
| Insider threat | Audit logging, separation of duties, privileged access management |
| Supply chain | Container image scanning (Trivy), dependency auditing (cargo audit), image signing (Cosign) |
| Data exfiltration | Encryption at rest, network policies, egress filtering |

---

*See [API Design](api-design.md) for rate limiting and idempotency details.*
*See [Data Architecture](data-architecture.md) for row-level security and encryption at rest.*
