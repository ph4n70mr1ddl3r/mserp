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

---

- [Authorization (RBAC + ABAC)](./authorization.md)
- [Data Protection](./data-protection.md)
- [Governance, Risk & Compliance (GRC)](./grc.md)
- [Threat Model & Incident Response](./threat-model.md)
