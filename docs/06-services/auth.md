# Auth Service

| Aspect | Details |
|--------|---------|
| Port | 8001 |
| Database | `auth_db` |
| Responsibilities | Authentication, token management, session handling |

## Features

| Feature | Description |
|---------|-------------|
| OAuth2/OIDC Provider | Full OAuth2 authorization server with OpenID Connect extension |
| MFA | Multi-factor authentication with TOTP, SMS, email, and biometric factors |
| SSO | SAML 2.0 bridge for enterprise identity provider federation |
| Password Policies | Configurable complexity, rotation, history, and breach-check rules |
| JWT Issuance & Rotation | RS256-signed JWTs with weekly key rotation and Redis-based revocation |
| Brute-Force Detection | Adaptive lockout with IP-based rate limiting and account protection |
| Step-Up Authentication | Elevated session for sensitive operations with configurable triggers |
| Session Management | Concurrent session control, idle timeout, device binding, revocation |

## OAuth2/OIDC Flows

| Flow | Use Case | Endpoints |
|------|----------|-----------|
| Authorization Code | Web applications (server-rendered, SSR) | `GET /auth/authorize` → `POST /auth/token` |
| Client Credentials | Service-to-service (see Service JWT below) | `POST /auth/token` (grant_type=client_credentials) |
| Authorization Code + PKCE | Mobile apps, single-page applications (SPA) | `GET /auth/authorize` + `code_challenge` → `POST /auth/token` + `code_verifier` |
| Refresh Token | Token renewal without re-authentication | `POST /auth/token` (grant_type=refresh_token) |

### Token Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/auth/authorize` | GET | Authorization endpoint — renders consent screen, issues authorization code |
| `/auth/token` | POST | Token endpoint — exchanges code/credentials/refresh token for tokens |
| `/auth/revoke` | POST | Revokes an access or refresh token |
| `/auth/.well-known/openid-configuration` | GET | OIDC discovery document |
| `/auth/.well-known/jwks.json` | GET | JSON Web Key Set for JWT verification |
| `/auth/userinfo` | GET | OIDC UserInfo endpoint — returns authenticated user claims |

> **Note:** Auth Service exposes two categories of endpoints: (1) OAuth2/OIDC protocol endpoints (e.g., `/auth/authorize`, `/auth/token`) for standards-based authentication flows, and (2) Application API endpoints (e.g., `/api/v1/auth/login`, `/api/v1/auth/mfa/enable`) for direct client interactions. The OAuth2 endpoints are listed in this spec; the application API endpoints are documented in `docs/05-api/endpoints.md`.

### Refresh Token Rotation

- A new refresh token is issued on every `grant_type=refresh_token` exchange; the previous refresh token is immediately invalidated.
- If a previously used (rotated-out) refresh token is presented, the entire token family is revoked — all sessions for that user are terminated, and a security alert is published.

## MFA

### Supported Factors

| Factor | Description |
|--------|-------------|
| TOTP | Time-based one-time password (RFC 6238) via authenticator apps (Google Authenticator, Authy, etc.) |
| SMS | One-time code delivered via SMS to verified phone number |
| Email | One-time code delivered to verified email address |
| Biometric | Device-native biometric (fingerprint, face) via WebAuthn/FIDO2 |

### Enrollment Flow

1. User initiates MFA enrollment via `POST /auth/mfa/enroll` with chosen factor type.
2. Auth Service generates a challenge (TOTP secret QR, SMS/email verification code, WebAuthn registration options).
3. User completes the challenge via `POST /auth/mfa/verify-enrollment`.
4. Upon successful verification, the factor is marked `active` in `mfa_factors`.
5. Ten single-use recovery codes are generated and displayed once (hashed before storage).

### Verification Flow

1. After primary authentication (password), Auth Service checks if any MFA factor is `active`.
2. If MFA is required, the login response includes `mfa_required: true` and a `mfa_challenge_token`.
3. Client submits `POST /auth/mfa/challenge` with the challenge token and the MFA code.
4. On success, the standard access/refresh token pair is issued.
5. On failure (invalid code), the attempt is logged and brute-force counters are incremented.

### Recovery Codes

- Ten single-use recovery codes generated per enrollment.
- Each code is a 12-character alphanumeric string, stored as a bcrypt hash.
- Using a recovery code satisfies MFA verification; the used code is invalidated.
- Users can regenerate recovery codes via `POST /auth/mfa/recovery-codes/regenerate`.

## SSO (SAML 2.0 Bridge)

| Capability | Description |
|------------|-------------|
| SAML Assertion Consumption | Auth Service acts as a Service Provider (SP), consuming SAML assertions from enterprise IdPs |
| Attribute Mapping | Configurable mapping of SAML attributes (email, displayName, groups) to MSERP user fields |
| IdP-Initiated SSO | Unsolicited SAML responses accepted at `/auth/saml/acs/{tenant_id}` |
| SP-Initiated SSO | Login page redirects to IdP via `GET /auth/saml/login?idp={idp_id}` |
| Just-in-Time Provisioning | Auto-create MSERP user on first SAML assertion if user does not exist (configurable per tenant) |
| SAML Logout | Single Logout (SLO) support — IdP-initiated and SP-initiated |

### SAML Configuration per Tenant

| Setting | Description |
|---------|-------------|
| IdP Metadata URL | URL to SAML IdP metadata XML |
| Entity ID | SP entity ID (unique per tenant) |
| ACS URL | Assertion Consumer Service URL |
| Certificate | IdP signing certificate (PEM) |
| Attribute Mappings | JSON map of SAML attribute → MSERP field |

## JWT Specification

JWT token specification including claims, signing algorithm, and token lifetimes is defined in SPEC.md §11.1.

## Brute-Force Detection

| Rule | Threshold | Action |
|------|-----------|--------|
| Account lockout | 5 consecutive failed login attempts for the same `username` | Account locked for 15 minutes; subsequent attempts rejected with `429` |
| IP rate limit | 20 failed login attempts from the same IP within 10 minutes | IP blocked for 30 minutes at the gateway level |
| MFA brute-force | 5 failed MFA verification attempts for the same `mfa_challenge_token` | Challenge token invalidated; user must restart login flow |

### Lockout vs Block

| Type | Scope | Duration | Resolution |
|------|-------|----------|------------|
| Account lockout | Per user (global across IPs) | 15 min auto-expiry | Admin can unlock via `POST /auth/admin/unlock-user` |
| IP block | Per source IP | 30 min auto-expiry | Auto-expires; no manual override |

## Step-Up Authentication

### Triggers

| Trigger | Condition | Description |
|---------|-----------|-------------|
| Sensitive operation | User invokes a protected action (e.g., user deletion, permission change) | Endpoint declares `requires_step_up: true` |
| Amount threshold | Transaction amount exceeds configurable limit (default: $10,000) | Evaluated per domain by the owning service |
| Outside business hours | Request time is outside tenant-configured business hours | Additional verification required |
| Unmanaged device | Device trust level is below threshold | Device not enrolled or fingerprint changed |
| IP outside corporate range | Source IP is not in tenant's trusted IP ranges | Evaluated at gateway |

### Elevated Session

| Property | Value |
|----------|-------|
| Duration | 5 minutes (configurable per tenant) |
| Token type | Standard access token with additional claim `elevated: true` |
| Re-authentication | Requires MFA verification (any active factor) |
| Scope | Elevated claim is session-scoped — not persisted across login sessions |

## Session Management

- Concurrent session control: configurable max sessions per user (default: 5)
- Session revocation: individual or all sessions (e.g., on password change)
- Idle timeout: configurable per tenant (default: 30 minutes)
- Device binding: optional device fingerprint for session validation

## Token Lifecycle

| Token Type | Lifetime | Rotation | Revocation |
|------------|----------|----------|------------|
| Access token | 15 minutes | N/A (short-lived) | Redis blocklist keyed by `jti`; checked by API Gateway on every request |
| Refresh token | 7 days | Rotated on every use; old token immediately invalidated | Deleted from `refresh_tokens` table; token family revocation on reuse detection |
| Service JWT | 1 hour | Re-issued by the calling service using client credentials | Not revocable (short-lived); service account credentials can be disabled |
| JWT signing key | Rotated weekly | New key pair generated; old key remains valid for 2 days (overlap) for graceful rotation | Compromise: immediate rotation via `POST /auth/admin/rotate-keys` |

### Redis Blocklist

- Access token revocation stores the `jti` in Redis with TTL equal to the token's remaining lifetime.
- API Gateway checks Redis blocklist before forwarding requests (O(1) lookup).
- Refresh token revocation removes the token row from `refresh_tokens`.

## Database Tables

| Table | Purpose | Key Columns |
|-------|---------|-------------|
| `users_auth` | Credentials and MFA status per user | `user_id` (PK), `tenant_id`, `password_hash`, `mfa_enabled`, `locked_until`, `password_changed_at`, `failed_attempts` |
| `sessions` | Active user sessions | `session_id` (PK), `user_id`, `tenant_id`, `device_fingerprint`, `ip_address`, `user_agent`, `expires_at`, `created_at` |
| `refresh_tokens` | Hashed refresh tokens for rotation tracking | `id` (PK), `user_id`, `token_hash`, `token_family`, `expires_at`, `revoked_at`, `created_at` |
| `mfa_factors` | Enrolled MFA devices per user | `factor_id` (PK), `user_id`, `factor_type`, `secret` (encrypted), `phone` (encrypted), `verified`, `last_used_at` |
| `login_attempts` | Audit log of all authentication attempts | `attempt_id` (PK), `user_id`, `ip_address`, `success`, `failure_reason`, `mfa_used`, `user_agent`, `created_at` |
| `jwt_keys` | Signing key rotation history | `key_id` (PK), `public_key`, `private_key` (encrypted), `active_from`, `active_until`, `revoked` |
| `saml_configurations` | Per-tenant SAML IdP configurations | `id` (PK), `tenant_id`, `idp_metadata_url`, `entity_id`, `idp_certificate`, `attribute_mappings` |

> **Standard Columns & RLS:** All tables include standard columns per SPEC.md §9.1 (`id`, `tenant_id`, `created_at`, `updated_at`, `created_by`, `updated_by`, `version`, `is_deleted`). RLS enforced per SPEC.md §9.3. Exception: `jwt_keys` is a global table without `tenant_id`.

## Events Published

| Event | Payload | Description |
|-------|---------|-------------|
| `auth.login.succeeded` | `{ user_id, tenant_id, ip, user_agent, mfa_used }` | User authenticated successfully |
| `auth.login.failed` | `{ username, tenant_id, ip, user_agent, failure_reason }` | Authentication attempt failed |
| `auth.token.refreshed` | `{ user_id, tenant_id, old_jti, new_jti }` | Access token refreshed via refresh token |
| `auth.mfa.enabled` | `{ user_id, tenant_id, factor_type }` | MFA factor enrolled and activated |
| `auth.password.changed` | `{ user_id, tenant_id }` | User password changed |
| `auth.session.revoked` | `{ user_id, tenant_id, session_id, reason }` | Session explicitly revoked |
| `auth.step-up.requested` | `{ user_id, tenant_id, trigger_reason, ip }` | Step-up authentication triggered |

> **Note:** Auth events are consumed by the Platform Service for security audit logging and alerting (e.g., brute-force detection on repeated `auth.login.failed`).

> **Important:** Auth Service does **NOT** consume events from other services. Auth uses **synchronous HTTP calls** to query Identity Service for user data during login. This is by design — events are asynchronous and too slow for real-time authentication decisions. Do not refactor the Auth→Identity login flow to be event-driven.

## See Also

- [Identity Service](identity.md)
- [Tenant Service](tenant.md)
- [Security Architecture](../02-security/overview.md)
- [Authorization (RBAC + ABAC)](../02-security/authorization.md)
- [Architecture Overview](../01-architecture/overview.md)
