# Identity Service

| Aspect | Details |
|--------|---------|
| Port | 8002 |
| Database | `identity_db` |
| Responsibilities | User management, roles, permissions, organizations |

## Features

User CRUD with lifecycle (active, suspended, deactivated), RBAC + ABAC, user groups / teams, organization hierarchy (departments, divisions, business units), user preferences (locale, timezone, notification preferences), delegated administration, API key management for service integrations

## Organizational Hierarchy

```
Organization
  └── Business Unit
       └── Department
            └── Team
                 └── User
```

- Users belong to one or more teams.
- Roles are assigned at the organization, business unit, or department level.
- Row-level security can filter data by the user's organizational scope.

## See Also

- [Auth Service](auth.md)
- [Tenant Service](tenant.md)
- [Architecture Overview](../architecture/overview.md)
