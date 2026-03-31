# Identity Service

| Aspect | Details |
|--------|---------|
| Port | 8002 |
| Database | `identity_db` |
| Responsibilities | User management, roles, permissions, organizations |

## Features

| Feature | Description |
|---------|-------------|
| User Lifecycle | CRUD with state machine: pending → active → suspended → deactivated |
| RBAC | Role-based access control with 77+ predefined roles and granular permissions |
| ABAC | Attribute-based policies for fine-grained access (org unit, amount, time, IP, data classification, device trust) |
| User Groups / Teams | Group-based membership with group-level role assignment |
| Organization Hierarchy | Multi-level organizational structure (departments, divisions, business units) |
| User Preferences | Per-user settings for locale, timezone, notifications, accessibility, and dashboard layout |
| Delegated Administration | Scoped admin roles for org-unit and helpdesk operations |
| API Key Management | Scoped API keys with automatic rotation for service integrations |

## User Lifecycle

### State Machine

```
                    activate()
  ┌──────────┐  ──────────────►  ┌──────────┐
  │ pending  │                   │  active   │ ◄──┐
  └──────────┘                   └──────────┘    │
       │                              │    ▲      │
       │ invite expires /             │    │      │ reactivate()
       │ admin cancels                │    │      │
       ▼                              │    │      │
  ┌──────────────┐          suspend() │    │      │
  │  cancelled   │◄───────────────────┘    │      │
  └──────────────┘                         │      │
                                           │      │
                              deactivate() │      │
                                           ▼      │
                                     ┌──────────────┐
                                     │ deactivated  │
                                     └──────────────┘
```

### State Transitions

| Transition | Trigger | Side Effects |
|------------|---------|--------------|
| pending → active | User sets password via invite link, or admin activates | User can authenticate; `identity.user.created` event published |
| active → suspended | Admin action, security incident, or failed compliance check | All sessions revoked; user cannot authenticate |
| suspended → active | Admin reactivation | User can authenticate again |
| active → deactivated | Admin action or user self-service account closure | Sessions revoked; data retained per retention policy; permissions preserved for audit |
| deactivated → active | Admin reactivation (within retention period) | Permissions restored; new password required |
| pending → cancelled | Invite expires or admin cancels | Invite invalidated; no user data retained |

### User Profile Fields

| Field | Required | Description |
|-------|----------|-------------|
| `email` | Yes | Unique within tenant; used as login identifier |
| `first_name` | Yes | Given name |
| `last_name` | Yes | Family name |
| `display_name` | No | Display name (defaults to first + last) |
| `employee_id` | No | External employee identifier for HR integration |
| `phone` | No | Phone number (E.164 format) |
| `avatar_url` | No | Profile picture URL (stored via Platform File Service) |
| `org_unit_id` | No | Primary organizational unit assignment |
| `job_title` | No | Job title |
| `department` | No | Department name (denormalized from org hierarchy) |

## RBAC Model

### Permission Format

Permission format follows `{domain}.{entity}.{action}` as defined in SPEC.md §11.3.

### Role → Permission Mapping

- Roles are collections of permissions. A role maps to one or more permission strings.
- Users acquire permissions through role assignment (direct or via group membership).
- Permissions are embedded in the JWT at login time; changes take effect on next token refresh.

### Role Hierarchy

| Level | Description | Example |
|-------|-------------|---------|
| System | Cross-tenant roles with global scope | Super Admin |
| Tenant | Tenant-wide administrative roles | Tenant Admin, GRC Officer |
| Domain | Roles scoped to a functional domain | Finance Manager, Sales Manager |
| Functional | Day-to-day operational roles | Accountant, Sales Rep, Employee |
| Scoped | Roles with limited scope (delegated admin) | Org-Unit Admin, Helpdesk Admin |

### Predefined Roles

> See [Authorization — Predefined Roles](../02-security/authorization.md#7-predefined-roles) for the complete list of 40+ predefined roles with their permission assignments. Key roles include: Super Admin, Tenant Admin, Finance Manager, Accountant, Treasury Manager, Sales Manager, Sales Rep, Inventory Manager, Procurement Manager, HR Manager, HR Specialist, Employee, Project Manager, CRM User, Manufacturing Manager, Quality Manager, Report Viewer, ESG Analyst, GRC Officer, and 25+ additional domain-specific roles.

## ABAC Attributes

| Attribute | Source | Example Policy |
|-----------|--------|----------------|
| User's organization unit | Identity Service (user profile) | `org_unit = 'North America Sales'` → filter orders by region `NA` |
| Resource's department | Data record (domain-specific) | `order.department = 'NA-Sales'` → match against user's org unit |
| Amount threshold | Request payload | `amount > 10000` → require manager approval |
| Time of day | System clock | `outside business hours` → trigger step-up authentication |
| IP range | Request source IP | `source_ip NOT IN corporate_range` → restrict sensitive operations |
| Data classification | Resource metadata | `classification = 'Restricted'` → limit to named users |
| Device trust level | Auth Service (session) | `device_trust = 'unmanaged'` → block export operations |

- ABAC policies are evaluated by the service layer (not the API Gateway) since they require access to resource attributes.
- ABAC rules are defined per tenant and stored in the `abac_policies` table.

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
- The hierarchy is stored as an adjacency list in the `organizations` table with `parent_id` self-reference.
- Org unit paths are materialized as `ltree` for efficient ancestor/descendant queries (e.g., `NA.Sales.Enterprise`).

## User Groups & Teams

| Group Type | Description | Membership |
|------------|-------------|------------|
| Team | Operational team within a department | Users assigned by admin or team lead |
| Distribution List | Notification and communication group | Users opt-in or are added by admin |
| Access Group | Role assignment group — members inherit group's roles | Admin-managed |
| Project Team | Cross-functional team for a specific project | Project manager assigns members |

### Membership Management

- Group membership is managed via `POST /api/v1/identity/groups/{group_id}/members` (add) and `DELETE` (remove).
- Bulk membership operations supported via `POST /api/v1/identity/groups/{group_id}/members/bulk`.
- Group-based role assignment: when a role is assigned to a group, all members inherit the role's permissions.
- Membership changes propagate to the user's JWT on next token refresh (up to 15 minutes).

## API Key Management

### Key Lifecycle

```
active ──► rotated ──► expired
   │
   └──► revoked
```

| State | Description |
|-------|-------------|
| active | Key can authenticate requests |
| rotated | Key is replaced by a new key; old key remains valid for a configurable grace period (default: 24 hours) |
| expired | Key has passed its expiration date; cannot authenticate |
| revoked | Key manually revoked by admin; immediately invalid |

### Key Properties

| Property | Description |
|----------|-------------|
| Key format | `msak_{environment}_{random_32_chars}` (prefixed for identification) |
| Storage | Only the SHA-256 hash is stored in `api_keys`; the full key is shown once at creation |
| Scopes | Array of permission strings, identical in format to user permissions |
| Rate limits | Configurable per key: requests per minute (default: 100), requests per day (default: 10,000) |
| Expiration | Mandatory; default 90 days, configurable per tenant (max: 365 days) |
| Automatic rotation | Configurable: disabled, 30-day, 60-day, 90-day; new key generated automatically and owner notified |

### Key Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/identity/api-keys` | POST | Create a new API key |
| `/api/v1/identity/api-keys` | GET | List API keys (masked) for the current user |
| `/api/v1/identity/api-keys/{id}` | DELETE | Revoke an API key |
| `/api/v1/identity/api-keys/{id}/rotate` | POST | Rotate an API key (generates replacement, old key enters grace period) |

## Delegated Administration

| Admin Role | Scope | Capabilities |
|------------|-------|--------------|
| Org-Unit Admin | Single organizational unit and its descendants | Create/edit/deactivate users within scope; assign roles within scope; manage group membership within scope |
| Helpdesk Admin | Tenant-wide (limited) | Reset passwords; unlock accounts; view user profiles; cannot assign roles or modify permissions |
| User Admin | Tenant-wide (limited) | Create/edit/deactivate users; assign a restricted set of roles (functional roles only, not system/tenant-level roles) |

- Delegated admins cannot escalate their own permissions.
- All delegated admin actions are audit-logged with the admin's identity and the action performed.

## User Preferences

| Preference | Type | Default | Description |
|------------|------|---------|-------------|
| `locale` | string | `en-US` | UI language and locale (ISO 639-1 + ISO 3166-1) |
| `timezone` | string | `UTC` | IANA timezone identifier (e.g., `America/New_York`) |
| `date_format` | string | `YYYY-MM-DD` | Display date format |
| `time_format` | string | `24h` | `12h` or `24h` |
| `currency` | string | `USD` | Default display currency (ISO 4217) |
| `notification.email` | boolean | `true` | Enable email notifications |
| `notification.push` | boolean | `true` | Enable push notifications |
| `notification.in_app` | boolean | `true` | Enable in-app notifications |
| `accessibility.high_contrast` | boolean | `false` | High contrast UI mode |
| `accessibility.screen_reader` | boolean | `false` | Screen reader optimized layout |
| `accessibility.font_size` | string | `medium` | `small`, `medium`, `large` |
| `dashboard.layout` | JSON | `{}` | Custom dashboard widget arrangement |
| `dashboard.default_landing` | string | `/dashboard` | Default landing page after login |

- Preferences stored as key-value pairs in `user_preferences` table.
- Read via `GET /api/v1/identity/users/me/preferences`, updated via `PATCH /api/v1/identity/users/me/preferences`.

## Database Tables

| Table | Purpose | Key Columns |
|-------|---------|-------------|
| `users` | User profiles and lifecycle state | `user_id` (PK), `tenant_id`, `email`, `first_name`, `last_name`, `status`, `org_unit_id`, `employee_id`, `created_at`, `updated_at` |
| `roles` | Role definitions | `role_id` (PK), `tenant_id`, `name`, `description`, `level`, `is_system`, `created_at` |
| `permissions` | Defined permissions (materialized from code) | `permission_id` (PK), `domain`, `entity`, `action`, `description` |
| `role_permissions` | Role → permission mapping | `role_id` (FK), `permission_id` (FK), composite PK |
| `user_roles` | User → role assignments with scope | `id` (PK), `user_id` (FK), `role_id` (FK), `org_unit_id` (FK, nullable), `assigned_by`, `assigned_at` |
| `groups` | User groups and teams | `group_id` (PK), `tenant_id`, `name`, `type`, `org_unit_id` (FK), `description`, `created_at` |
| `group_members` | Group membership | `group_id` (FK), `user_id` (FK), `role` (admin/member), `added_at`, composite PK |
| `group_roles` | Group → role assignments | `group_id` (FK), `role_id` (FK), composite PK |
| `organizations` | Organizational hierarchy (adjacency list + ltree path) | `org_id` (PK), `tenant_id`, `name`, `type` (organization/business_unit/department/team), `parent_id` (FK, self-ref), `path` (ltree), `created_at` |
| `api_keys` | API key credentials | `key_id` (PK), `tenant_id`, `user_id` (FK), `key_hash`, `key_prefix`, `scopes` (JSONB), `rate_limit_rpm`, `rate_limit_rpd`, `status`, `expires_at`, `rotated_from`, `created_at` |
| `user_preferences` | Per-user preference key-value store | `user_id` (FK), `key`, `value`, composite PK |
| `abac_policies` | Attribute-based access control rules | `policy_id` (PK), `tenant_id`, `name`, `description`, `condition` (JSONB), `effect` (allow/deny), `priority`, `enabled` |

> **Standard Columns & RLS:** All tables include standard columns per SPEC.md §9.1 (`id`, `tenant_id`, `created_at`, `updated_at`, `created_by`, `updated_by`, `version`, `is_deleted`). RLS enforced per SPEC.md §9.3. Exception: `permissions` is a global table seeded from code; `role_permissions` uses composite PK.

## Events Published

| Event | Payload | Description |
|-------|---------|-------------|
| `identity.user.created` | `{ user_id, tenant_id, email, org_unit_id, status }` | New user created and activated |
| `identity.user.updated` | `{ user_id, tenant_id, changed_fields, old_values }` | User profile fields updated |
| `identity.user.suspended` | `{ user_id, tenant_id, reason, suspended_by }` | User account suspended |
| `identity.user.deactivated` | `{ user_id, tenant_id, reason, deactivated_by }` | User account deactivated |
| `identity.user.reactivated` | `{ user_id, tenant_id, reactivated_by }` | Deactivated user reactivated |
| `identity.role.created` | `{ role_id, tenant_id, name, permissions }` | Custom role created |
| `identity.role.updated` | `{ role_id, tenant_id, name, added_permissions, removed_permissions }` | Role permissions modified |
| `identity.group.updated` | `{ group_id, tenant_id, added_members, removed_members }` | Group membership changed |
| `identity.api-key.created` | `{ key_id, tenant_id, user_id, key_prefix, expires_at }` | API key created |
| `identity.api-key.rotated` | `{ key_id, tenant_id, old_key_prefix, new_key_prefix }` | API key rotated |
| `identity.api-key.revoked` | `{ key_id, tenant_id, revoked_by }` | API key revoked |

> **Note:** Auth Service queries Identity Service via synchronous HTTP for user/role data during login. It does NOT consume Identity events. Identity events are consumed by: Platform Service (security audit), Integration Service (MDM), and HR Service (provisioning check via HTTP). Tenant Service consumes user events for tenant user counts.

## See Also

- [Auth Service](auth.md)
- [Tenant Service](tenant.md)
- [Authorization (RBAC + ABAC)](../02-security/authorization.md)
- [Security Architecture](../02-security/overview.md)
- [Architecture Overview](../01-architecture/overview.md)
