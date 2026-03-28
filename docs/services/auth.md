# Auth Service

| Aspect | Details |
|--------|---------|
| Port | 8001 |
| Database | `auth_db` |
| Responsibilities | Authentication, token management, session handling |

## Features

OAuth2/OIDC provider, MFA (TOTP, SMS, email, biometric), SSO (SAML 2.0 bridge), password policies (complexity, rotation, history), JWT issuance and rotation, brute-force detection, account lockout, step-up authentication

## Session Management

- Concurrent session control: configurable max sessions per user (default: 5)
- Session revocation: individual or all sessions (e.g., on password change)
- Idle timeout: configurable per tenant (default: 30 minutes)
- Device binding: optional device fingerprint for session validation

## See Also

- [Identity Service](identity.md)
- [Tenant Service](tenant.md)
- [Architecture Overview](../architecture/overview.md)
