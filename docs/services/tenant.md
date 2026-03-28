# Tenant Service

| Aspect | Details |
|--------|---------|
| Port | 8003 |
| Database | `tenant_db` |
| Responsibilities | Multi-tenancy, tenant provisioning, feature flags |

## Features

Tenant lifecycle (provisioning, active, suspended, decommissioned), subscription tier management, feature toggles with rollout strategies, tenant-specific branding, tenant quota enforcement (users, storage, API calls), data residency controls (region pinning per tenant)

## Tenant Quotas

| Resource | Standard | Professional | Enterprise |
|----------|----------|-------------|------------|
| Users | 50 | 500 | Unlimited |
| Storage | 10 GB | 100 GB | 1 TB |
| API calls/min | 1,000 | 10,000 | 100,000 |
| Custom integrations | 3 | 25 | Unlimited |
| Custom applications (low-code) | 1 | 10 | Unlimited |
| Data warehouse storage | 5 GB | 50 GB | 500 GB |

## See Also

- [Auth Service](auth.md)
- [Identity Service](identity.md)
- [Config Service](config.md)
- [Architecture Overview](../architecture/overview.md)
