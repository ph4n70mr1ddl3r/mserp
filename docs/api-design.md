# API Design

## 1. API Standards

| Standard | Specification |
|----------|---------------|
| Style | RESTful with OpenAPI 3.1 |
| Versioning | URL path (`/api/v1/...`) |
| Authentication | Bearer JWT (access + refresh tokens) |
| Content-Type | `application/json` |
| Error Format | RFC 7807 Problem Details |
| Pagination | Cursor-based with `limit`/`after` (opt-in total count) |
| Filtering | Query parameters with operators (`eq`, `ne`, `gt`, `gte`, `lt`, `lte`, `like`, `in`) |
| Sorting | `sort` query parameter (e.g., `sort=created_at:desc,name:asc`) |
| Idempotency | `Idempotency-Key` header on all state-changing requests |
| Compression | `gzip` encoding supported (via `Accept-Encoding`); responses > 1KB compressed |
| Locale | `Accept-Language` header for response localization |
| Timezone | `Time-Zone` header for date/time interpretation (defaults to tenant timezone) |

## 2. Idempotency

All `POST`, `PUT`, `PATCH`, and `DELETE` requests MUST include an `Idempotency-Key` header. The server stores the response for each key and returns the cached response on duplicate requests within a 24-hour window.

| Header | Example | Behavior |
|--------|---------|----------|
| `Idempotency-Key` | `idemp_550e8400-e29b-41d4-a716-446655440000` | UUID v4 per unique operation |

**Rules:**
- Keys are scoped per tenant + user + endpoint combination.
- If a request with a known key is retried, the server returns the original response with the original HTTP status code (e.g., `201 Created` for a successful creation, not `200 OK`).
- Keys expire after 24 hours and are cleaned up by a background job.
- If a key is reused with a different request payload, the server returns `422 Unprocessable Entity` with error code `IDEMPOTENCY_CONFLICT`.

## 3. Rate Limiting

Rate limiting is enforced at the API Gateway layer (Traefik by default; Kong as an alternative) with per-tenant and per-endpoint granularity.

| Scope | Limit | Window |
|-------|-------|--------|
| Per tenant (global) | 10,000 requests | 1 minute |
| Per user | 1,000 requests | 1 minute |
| Auth endpoints (login, password reset) | 10 requests | 1 minute |
| File upload endpoints | 50 requests | 1 minute |
| Report generation endpoints | 10 requests | 1 minute |
| Bulk operation endpoints | 20 requests | 1 minute |

- Limits are stored in Redis with sliding window counters.
- Responses include `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` headers.
- Exceeding returns `429 Too Many Requests` with a `Retry-After` header.
- **Exempt endpoints:** Health check endpoints (`/healthz`, `/readyz`) are exempt from rate limiting.
- Rate limit overrides per tenant are configurable via Tenant Service quota settings.

## 4. API Versioning

| Policy | Detail |
|--------|--------|
| Strategy | URL path (`/api/v1/...`) |
| Supported Versions | Maximum 2 concurrent versions |
| Sunset Notice | Deprecated versions return `Warning: 299 - "Deprecated API"` header 90 days before removal |
| Breaking Changes | Require a new version bump (e.g., `v1` -> `v2`); documented in changelog |
| Non-Breaking Changes | Additive changes (new fields, new endpoints) within the same version |
| Version Lifecycle | New version announced → 90-day dual-support → Old version sunset → Removal |

## 5. Standard Response Format

### Success Response
```json
{
  "data": {
    "id": "uuid",
    "version": 1,
    "...": "resource fields"
  },
  "meta": {
    "request_id": "uuid",
    "timestamp": "2026-03-27T10:30:00Z"
  }
}
```

> **Optimistic Concurrency:** Every resource includes a `version` field that increments on each update. `PUT` and `PATCH` requests MUST include the current `version` in the request body. If the version does not match the stored value, the server returns `409 Conflict` with error code `RESOURCE_CONFLICT`. The client should re-fetch the resource, merge changes, and retry.

### List Response
```json
{
  "data": [...],
  "meta": {
    "request_id": "uuid",
    "timestamp": "2026-03-27T10:30:00Z",
    "pagination": {
      "limit": 50,
      "has_more": true,
      "cursor": "eyJpZCI6MTAwfQ==",
      "total_count": 1523
    }
  }
}
```

**Pagination Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | integer | 50 | Max items per page (max: 100) |
| `after` | string | null | Cursor for next page (base64-encoded) |
| `include_total` | boolean | false | Include `total_count` in pagination metadata |

- `include_total=false` (default): O(1) query, no count. Recommended for general list views. The `total_count` field is omitted from the response.
- `include_total=true`: Performs a `COUNT(*)` query. Use sparingly — suitable for exports, reports, and accounting reconciliation where exact totals are required.
- When `after` is provided, `include_total` is ignored and `total_count` is omitted from the response (cursor-based traversal does not support mid-stream counts).

### Error Response

Follows [RFC 7807 Problem Details](https://datatracker.ietf.org/doc/html/rfc7807) with an additional `meta` envelope for request tracing:

```json
{
  "type": "https://api.mserp.io/errors/validation",
  "title": "Validation Error",
  "status": 400,
  "detail": "The 'email' field must be a valid email address",
  "instance": "/api/v1/users",
  "errors": [
    {
      "field": "email",
      "message": "invalid email format",
      "code": "INVALID_FORMAT"
    }
  ],
  "meta": {
    "request_id": "uuid",
    "timestamp": "2026-03-27T10:30:00Z"
  }
}
```

> **Note:** The `errors` array is optional. It is present for field-level validation failures (HTTP 400) and omitted for non-validation errors.

### Async Job Response

For long-running operations (data import, large report generation), the API returns `202 Accepted` with a job reference:

```json
{
  "data": {
    "job_id": "uuid",
    "status": "pending",
    "status_url": "/api/v1/integrations/jobs/uuid"
  },
  "meta": {
    "request_id": "uuid",
    "timestamp": "2026-03-27T10:30:00Z"
  }
}
```

Clients poll `status_url` until the job reaches a terminal state (`completed` or `failed`).

## 6. Health Check Endpoints

All services MUST expose standardized health check endpoints. These are used by Kubernetes for liveness and readiness probes.

### 6.1 Endpoints

| Endpoint | Purpose | K8s Probe |
|----------|---------|-----------|
| `GET /healthz` | Liveness — process is running | `livenessProbe` |
| `GET /readyz` | Readiness — can accept traffic | `readinessProbe` |

### 6.2 Liveness (`/healthz`)

Returns `200 OK` if the process is running. No dependency checks. Fails only on unrecoverable errors (e.g., deadlock, fatal panic).

```json
{
  "status": "ok",
  "uptime_seconds": 86400,
  "version": "1.2.3"
}
```

### 6.3 Readiness (`/readyz`)

Returns `200 OK` only when all critical dependencies are reachable. Returns `503 Service Unavailable` with details if any check fails.

```json
{
  "status": "ok",
  "checks": {
    "database": "ok",
    "redis": "ok",
    "rabbitmq": "ok"
  }
}
```

### 6.4 Check Configuration

| Service | Required Checks |
|---------|-----------------|
| All services | Database, Redis |
| Auth, Tenant | + RabbitMQ (connection — publish only, no inbox queue) |
| Config | + RabbitMQ (connection — publish only) |
| Identity | Database, Redis only (does not use RabbitMQ) |
| Commerce, Finance, HR, Manufacturing, Workflow, CRM/Marketing, Project, Integration | + RabbitMQ (connection + queue reachable) |
| Platform | + RabbitMQ (connection + queue reachable), Database x2 (`platform_db` + `audit_db`) |
| Report | + RabbitMQ (connection + queue reachable), Elasticsearch |

- Each individual dependency check within `/readyz` has a 1-second timeout.
- Checks run concurrently (not sequentially). The overall readiness probe HTTP timeout is 5 seconds.
- Failure of any check returns `503` with `"status": "degraded"` or `"status": "unavailable"`.

## 7. Bulk / Batch API Patterns

### 7.1 Design Principles

| Principle | Rule |
|-----------|------|
| Method | Bulk operations MUST use `POST` (not `GET`) |
| Batch Size | Maximum 100 items per request |
| Idempotency | Each bulk request requires a single `Idempotency-Key` |
| Response | Returns summary with per-item results |

### 7.2 Request Format

```json
POST /api/v1/commerce/products/bulk
Idempotency-Key: idemp_<uuid>
Content-Type: application/json

{
  "items": [
    { "sku": "SKU-001", "name": "Product A", "price": 10.00 },
    { "sku": "SKU-002", "name": "Product B", "price": 20.00 }
  ]
}
```

### 7.3 Response Format

```json
{
  "data": {
    "total_requested": 2,
    "succeeded": 1,
    "failed": 1,
    "results": [
      {
        "index": 0,
        "status": "created",
        "id": "uuid-1",
        "sku": "SKU-001"
      },
      {
        "index": 1,
        "status": "failed",
        "error": {
          "code": "DUPLICATE_SKU",
          "message": "SKU 'SKU-002' already exists"
        }
      }
    ]
  },
  "meta": {
    "request_id": "uuid",
    "timestamp": "2026-03-27T10:30:00Z"
  }
}
```

### 7.4 Bulk Processing Rules

- **Default (atomic):** All items processed within a single database transaction. If any item fails validation, all items are rolled back.
- **Non-atomic (`Prefer: partial-success` header):** Items processed individually with independent transactions. Partial success is possible. On retry with the same `Idempotency-Key`, previously succeeded items are skipped.
- Bulk operations exceeding 100 items MUST return `413 Payload Too Large`.
- For datasets larger than 100 items, use the Integration Service file-based import (CSV/Excel).

## 8. Error Code Taxonomy

All error codes follow the pattern `{DOMAIN}_{CATEGORY}_{SPECIFIC}` and are defined in a centralized registry per service.

### 8.1 Common Error Codes (All Services)

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `AUTH_UNAUTHORIZED` | 401 | Missing or invalid authentication |
| `AUTH_TOKEN_EXPIRED` | 401 | JWT access token has expired |
| `AUTH_FORBIDDEN` | 403 | Insufficient permissions for this operation |
| `VALIDATION_INVALID_FORMAT` | 400 | Field format validation failed |
| `VALIDATION_REQUIRED_FIELD` | 400 | Required field is missing |
| `VALIDATION_INVALID_VALUE` | 400 | Field value is out of allowed range |
| `RESOURCE_NOT_FOUND` | 404 | Requested resource does not exist |
| `RESOURCE_CONFLICT` | 409 | Resource already exists (unique constraint) |
| `RATE_LIMITED` | 429 | Too many requests |
| `IDEMPOTENCY_CONFLICT` | 422 | Idempotency key already used with different payload |
| `INTERNAL_ERROR` | 500 | Unexpected server error |
| `SERVICE_UNAVAILABLE` | 503 | Dependency temporarily unavailable |
| `QUOTA_EXCEEDED` | 429 | Tenant quota exceeded (users, storage, API calls) |

### 8.2 Commerce Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `COMMERCE_DUPLICATE_SKU` | 409 | Product SKU already exists |
| `COMMERCE_INSUFFICIENT_STOCK` | 409 | Not enough stock to fulfill |
| `COMMERCE_ORDER_ALREADY_SUBMITTED` | 409 | Order cannot be modified after submission |
| `COMMERCE_QUOTATION_EXPIRED` | 410 | Quotation has expired |
| `COMMERCE_CUSTOMER_INACTIVE` | 409 | Customer account is inactive |
| `COMMERCE_WAREHOUSE_NOT_FOUND` | 404 | Warehouse does not exist |
| `COMMERCE_TRANSFER_INVALID` | 400 | Stock transfer parameters invalid |
| `COMMERCE_RETURN_NOT_ALLOWED` | 400 | Return not allowed for this order/line |
| `COMMERCE_PRICE_INVALID` | 400 | Pricing rule produced invalid price |

### 8.3 Finance Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `FINANCE_ACCOUNT_CODE_EXISTS` | 409 | Account code already in use |
| `FINANCE_JOURNAL_UNBALANCED` | 400 | Journal entry must be balanced (debits = credits) |
| `FINANCE_JOURNAL_ALREADY_POSTED` | 409 | Journal entry cannot be modified after posting |
| `FINANCE_PERIOD_CLOSED` | 409 | Accounting period is closed |
| `FINANCE_DUPLICATE_INVOICE` | 409 | Invoice number already exists |
| `FINANCE_SUPPLIER_INACTIVE` | 409 | Supplier is inactive |
| `FINANCE_CURRENCY_UNSUPPORTED` | 400 | Currency code not supported |
| `FINANCE_EXCHANGE_RATE_MISSING` | 400 | No exchange rate for currency pair and date |
| `FINANCE_BUDGET_EXCEEDED` | 409 | Budget limit exceeded for this account/period |
| `FINANCE_INTERCOMPANY_MISMATCH` | 400 | Intercompany transaction tenant mismatch |

### 8.4 HR Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `HR_EMPLOYEE_EXISTS` | 409 | Employee with this ID already exists |
| `HR_LEAVE_BALANCE_EXCEEDED` | 409 | Not enough leave balance |
| `HR_ATTENDANCE_CONFLICT` | 409 | Overlapping attendance record |
| `HR_PAYROLL_ALREADY_PROCESSED` | 409 | Payroll for this period already processed |
| `HR_REQUISITION_CLOSED` | 409 | Job requisition is closed |

### 8.5 Manufacturing Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `MFG_BOM_CIRCULAR_REF` | 400 | BOM contains circular reference |
| `MFG_WORK_ORDER_IN_PROGRESS` | 409 | Work order cannot be modified while in progress |
| `MFG_INSUFFICIENT_MATERIALS` | 409 | Not enough materials for production |
| `MFG_CAPACITY_EXCEEDED` | 409 | Work center capacity exceeded for requested time |
| `MFG_QUALITY_CHECK_FAILED` | 409 | Quality check did not pass threshold |

### 8.6 CRM Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `CRM_DUPLICATE_CONTACT` | 409 | Contact already exists with this email |
| `CRM_LEAD_CONVERTED` | 409 | Lead already converted to opportunity/customer |
| `CRM_CAMPAIGN_ACTIVE` | 409 | Campaign is active and cannot be modified |

### 8.7 Project Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `PROJECT_NOT_FOUND` | 404 | Project does not exist |
| `PROJECT_BUDGET_EXCEEDED` | 409 | Project budget exceeded |
| `PROJECT_TASK_DEPENDENCY` | 409 | Circular task dependency detected |

### 8.8 Error Code Rules

- Each service MUST register its error codes in its OpenAPI spec via the `errors` extension.
- New error codes MUST be additive only (never remove or rename).
- Error codes are case-sensitive and use `UPPER_SNAKE_CASE`.
- Consumer services MUST handle unknown error codes gracefully (treat as generic error).
- Error responses MUST include a `type` URI linking to human-readable documentation for the error code.

## 9. API Endpoints Overview

### Auth Service
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/v1/auth/login` | User login |
| POST | `/api/v1/auth/logout` | User logout (invalidates tokens) |
| POST | `/api/v1/auth/refresh` | Refresh tokens (rotation) |
| POST | `/api/v1/auth/password/reset` | Request password reset email |
| POST | `/api/v1/auth/password/confirm` | Confirm password reset |
| POST | `/api/v1/auth/mfa/enable` | Enable MFA for user |
| POST | `/api/v1/auth/mfa/verify` | Verify MFA code |
| POST | `/api/v1/auth/mfa/disable` | Disable MFA (requires password) |
| GET | `/api/v1/auth/sessions` | List active sessions for current user |
| DELETE | `/api/v1/auth/sessions/{id}` | Revoke specific session |
| DELETE | `/api/v1/auth/sessions` | Revoke all sessions except current |

### Identity Service
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/identity/users` | List users |
| POST | `/api/v1/identity/users` | Create user |
| GET | `/api/v1/identity/users/{id}` | Get user |
| PUT | `/api/v1/identity/users/{id}` | Update user |
| DELETE | `/api/v1/identity/users/{id}` | Soft-delete user |
| GET | `/api/v1/identity/roles` | List roles |
| POST | `/api/v1/identity/roles` | Create role |
| GET | `/api/v1/identity/roles/{id}/permissions` | List permissions for a role |
| PUT | `/api/v1/identity/roles/{id}/permissions` | Set permissions for a role (Tenant Admin only) |
| GET | `/api/v1/identity/groups` | List user groups |
| POST | `/api/v1/identity/groups` | Create user group |
| GET | `/api/v1/identity/groups/{id}/members` | List group members |
| POST | `/api/v1/identity/groups/{id}/members` | Add members to group |
| DELETE | `/api/v1/identity/groups/{id}/members/{userId}` | Remove member from group |
| GET | `/api/v1/identity/organizations` | List organizational units |
| POST | `/api/v1/identity/organizations` | Create organizational unit |

### Tenant Service
> Feature flag types, evaluation flow, and rules are specified in [Services — Feature Flags](services.md#4-feature-flag-implementation).

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/tenant` | Get current tenant details |
| POST | `/api/v1/tenant` | Create tenant (Super Admin only) |
| PUT | `/api/v1/tenant` | Update tenant settings |
| GET | `/api/v1/tenant/features` | List feature flags |
| GET | `/api/v1/tenant/features/{flag}` | Get specific flag value |
| PUT | `/api/v1/tenant/features/{flag}` | Set flag value (Super Admin only) |
| DELETE | `/api/v1/tenant/features/{flag}` | Remove flag override |
| GET | `/api/v1/tenant/quotas` | Get tenant quota usage |
| GET | `/api/v1/tenant/subscriptions` | Get subscription details |

### Config Service
> Config hierarchy, propagation, and schema are specified in [Services — Config Service](services.md#14-config-service).

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/config` | List all config for current tenant |
| GET | `/api/v1/config/{key}` | Get specific config value |
| PUT | `/api/v1/config/{key}` | Set config value (Tenant Admin only) |
| DELETE | `/api/v1/config/{key}` | Reset to default |
| GET | `/api/v1/config/audit` | Config change history |
| POST | `/api/v1/config/cache/flush` | Flush tenant cache (Tenant Admin only) |
| GET | `/api/v1/config/cors/origins` | List allowed CORS origins |
| POST | `/api/v1/config/cors/origins` | Add allowed CORS origin (Tenant Admin only) |
| DELETE | `/api/v1/config/cors/origins` | Remove allowed CORS origin (Tenant Admin only) |
| GET | `/api/v1/config/locales` | List supported locales |
| GET | `/api/v1/config/locales/{locale}/translations` | Get translations for locale |

### Commerce Service (Sales + Inventory)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/commerce/customers` | List customers |
| POST | `/api/v1/commerce/customers` | Create customer |
| GET | `/api/v1/commerce/customers/{id}` | Get customer |
| PUT | `/api/v1/commerce/customers/{id}` | Update customer |
| GET | `/api/v1/commerce/quotations` | List quotations |
| POST | `/api/v1/commerce/quotations` | Create quotation |
| GET | `/api/v1/commerce/quotations/{id}` | Get quotation details |
| POST | `/api/v1/commerce/quotations/{id}/submit` | Submit quotation |
| POST | `/api/v1/commerce/quotations/{id}/accept` | Accept quotation |
| POST | `/api/v1/commerce/quotations/{id}/reject` | Reject quotation |
| GET | `/api/v1/commerce/orders` | List sales orders |
| POST | `/api/v1/commerce/orders` | Create sales order |
| GET | `/api/v1/commerce/orders/{id}` | Get order details |
| PUT | `/api/v1/commerce/orders/{id}` | Update order (draft only) |
| POST | `/api/v1/commerce/orders/{id}/submit` | Submit order for fulfillment |
| POST | `/api/v1/commerce/orders/{id}/cancel` | Cancel order |
| GET | `/api/v1/commerce/orders/{id}/deliveries` | List deliveries for order |
| POST | `/api/v1/commerce/returns` | Create return (RMA) |
| GET | `/api/v1/commerce/products` | List products |
| POST | `/api/v1/commerce/products` | Create product |
| GET | `/api/v1/commerce/products/{id}` | Get product |
| PUT | `/api/v1/commerce/products/{id}` | Update product |
| POST | `/api/v1/commerce/products/bulk` | Bulk create products |
| GET | `/api/v1/commerce/price-lists` | List price lists |
| POST | `/api/v1/commerce/price-lists` | Create price list |
| POST | `/api/v1/commerce/pricing/calculate` | Calculate price for a product/customer/qty combination |
| GET | `/api/v1/commerce/warehouses` | List warehouses |
| GET | `/api/v1/commerce/stock` | Get stock levels |
| POST | `/api/v1/commerce/transfers` | Create stock transfer |
| POST | `/api/v1/commerce/adjustments` | Stock adjustment |

### Finance Service (Finance + Procurement)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/finance/accounts` | List chart of accounts |
| POST | `/api/v1/finance/accounts` | Create account |
| GET | `/api/v1/finance/journals` | List journal entries |
| POST | `/api/v1/finance/journals` | Create journal entry |
| POST | `/api/v1/finance/journals/{id}/post` | Post journal entry |
| POST | `/api/v1/finance/journals/{id}/reverse` | Reverse posted journal entry |
| GET | `/api/v1/finance/reports/balance-sheet` | Balance sheet report |
| GET | `/api/v1/finance/reports/profit-loss` | P&L report |
| GET | `/api/v1/finance/reports/cash-flow` | Cash flow statement |
| GET | `/api/v1/finance/reports/trial-balance` | Trial balance |
| GET | `/api/v1/finance/budgets` | List budgets |
| POST | `/api/v1/finance/budgets` | Create budget |
| GET | `/api/v1/finance/budgets/{id}/vs-actual` | Budget vs. actual report |
| GET | `/api/v1/finance/exchange-rates` | Get exchange rates |
| PUT | `/api/v1/finance/exchange-rates` | Update exchange rates |
| GET | `/api/v1/finance/periods` | List accounting periods |
| POST | `/api/v1/finance/periods/{id}/close` | Close accounting period |
| GET | `/api/v1/finance/suppliers` | List suppliers |
| POST | `/api/v1/finance/suppliers` | Create supplier |
| GET | `/api/v1/finance/suppliers/{id}` | Get supplier |
| PUT | `/api/v1/finance/suppliers/{id}` | Update supplier |
| GET | `/api/v1/finance/purchase-orders` | List purchase orders |
| POST | `/api/v1/finance/purchase-orders` | Create purchase order |
| GET | `/api/v1/finance/purchase-orders/{id}` | Get purchase order |
| POST | `/api/v1/finance/purchase-orders/{id}/receive` | Receive goods |
| GET | `/api/v1/finance/requisitions` | List purchase requisitions |
| POST | `/api/v1/finance/requisitions` | Create purchase requisition |

### HR Service
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/hr/employees` | List employees |
| POST | `/api/v1/hr/employees` | Create employee |
| GET | `/api/v1/hr/employees/{id}` | Get employee |
| PUT | `/api/v1/hr/employees/{id}` | Update employee |
| GET | `/api/v1/hr/attendance` | Get attendance records |
| POST | `/api/v1/hr/attendance/check-in` | Check in |
| POST | `/api/v1/hr/attendance/check-out` | Check out |
| GET | `/api/v1/hr/leaves` | List leave requests |
| POST | `/api/v1/hr/leaves` | Create leave request |
| POST | `/api/v1/hr/leaves/{id}/approve` | Approve leave |
| POST | `/api/v1/hr/leaves/{id}/reject` | Reject leave |
| GET | `/api/v1/hr/leaves/balances` | Get leave balances |
| GET | `/api/v1/hr/payroll/runs` | List payroll runs |
| POST | `/api/v1/hr/payroll/runs` | Start payroll run |
| POST | `/api/v1/hr/payroll/runs/{id}/confirm` | Confirm and post payroll |
| GET | `/api/v1/hr/recruitment/requisitions` | List job requisitions |
| POST | `/api/v1/hr/recruitment/requisitions` | Create job requisition |
| GET | `/api/v1/hr/recruitment/applicants` | List applicants |
| POST | `/api/v1/hr/performance/reviews` | Create performance review |
| GET | `/api/v1/hr/org-chart` | Get organization chart |
| GET | `/api/v1/hr/training/courses` | List training courses |

### Manufacturing Service
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/manufacturing/boms` | List BOMs |
| POST | `/api/v1/manufacturing/boms` | Create BOM |
| GET | `/api/v1/manufacturing/boms/{id}` | Get BOM with components |
| GET | `/api/v1/manufacturing/work-orders` | List work orders |
| POST | `/api/v1/manufacturing/work-orders` | Create work order |
| GET | `/api/v1/manufacturing/work-orders/{id}` | Get work order |
| POST | `/api/v1/manufacturing/work-orders/{id}/start` | Start work order |
| POST | `/api/v1/manufacturing/work-orders/{id}/complete` | Complete work order |
| GET | `/api/v1/manufacturing/quality/inspections` | List quality inspections |
| POST | `/api/v1/manufacturing/quality/inspections` | Create quality inspection |
| GET | `/api/v1/manufacturing/work-centers` | List work centers |
| GET | `/api/v1/manufacturing/routing` | List routing definitions |

### CRM / Marketing Service
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/crm/contacts` | List contacts |
| POST | `/api/v1/crm/contacts` | Create contact |
| GET | `/api/v1/crm/leads` | List leads |
| POST | `/api/v1/crm/leads` | Create lead |
| POST | `/api/v1/crm/leads/{id}/convert` | Convert lead to opportunity/customer |
| GET | `/api/v1/crm/opportunities` | List opportunities |
| POST | `/api/v1/crm/opportunities` | Create opportunity |
| PUT | `/api/v1/crm/opportunities/{id}/stage` | Update opportunity stage |
| GET | `/api/v1/crm/campaigns` | List campaigns |
| POST | `/api/v1/crm/campaigns` | Create campaign |
| GET | `/api/v1/crm/activities` | List activities |
| POST | `/api/v1/crm/activities` | Log activity |
| GET | `/api/v1/crm/analytics/pipeline` | Pipeline analytics |

### Project Management Service
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/projects` | List projects |
| POST | `/api/v1/projects` | Create project |
| GET | `/api/v1/projects/{id}` | Get project details |
| PUT | `/api/v1/projects/{id}` | Update project |
| GET | `/api/v1/projects/{id}/tasks` | List project tasks |
| POST | `/api/v1/projects/{id}/tasks` | Create task |
| PUT | `/api/v1/projects/{id}/tasks/{taskId}` | Update task |
| GET | `/api/v1/projects/{id}/timeline` | Get Gantt/timeline data |
| GET | `/api/v1/projects/{id}/resources` | List allocated resources |
| POST | `/api/v1/projects/{id}/resources` | Allocate resource |
| POST | `/api/v1/projects/{id}/timesheets` | Submit timesheet entry |
| GET | `/api/v1/projects/{id}/expenses` | List project expenses |
| POST | `/api/v1/projects/{id}/expenses` | Submit expense |
| GET | `/api/v1/projects/{id}/billing` | Get billing/invoicing status |
| POST | `/api/v1/projects/{id}/billing/invoice` | Generate project invoice |
| GET | `/api/v1/projects/{id}/risks` | List project risks |
| POST | `/api/v1/projects/{id}/risks` | Register risk |

### Platform Service (Notification + File + Audit)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/platform/notifications` | List notifications |
| POST | `/api/v1/platform/notifications` | Send notification |
| PUT | `/api/v1/platform/notifications/{id}/read` | Mark notification read |
| POST | `/api/v1/platform/files/upload` | Upload file (multipart) |
| GET | `/api/v1/platform/files/{id}` | Get file metadata |
| GET | `/api/v1/platform/files/{id}/download` | Download file |
| DELETE | `/api/v1/platform/files/{id}` | Soft-delete file |
| GET | `/api/v1/platform/files/{id}/preview` | Get document preview |
| GET | `/api/v1/platform/audit` | Query audit log |
| GET | `/api/v1/platform/audit/entity/{type}/{id}` | Get audit trail for entity |

### Workflow Service
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/workflow/workflows` | List workflow definitions |
| POST | `/api/v1/workflow/workflows` | Create workflow definition |
| GET | `/api/v1/workflow/workflows/{id}` | Get workflow definition |
| PUT | `/api/v1/workflow/workflows/{id}` | Update workflow definition |
| POST | `/api/v1/workflow/workflows/{id}/start` | Start workflow instance |
| GET | `/api/v1/workflow/instances` | List workflow instances |
| GET | `/api/v1/workflow/instances/{id}` | Get instance status and history |
| POST | `/api/v1/workflow/instances/{id}/approve` | Approve current step |
| POST | `/api/v1/workflow/instances/{id}/reject` | Reject current step |
| POST | `/api/v1/workflow/instances/{id}/delegate` | Delegate to another user |
| GET | `/api/v1/workflow/templates` | List workflow templates |
| GET | `/api/v1/workflow/my-tasks` | List pending tasks for current user |

### Report Service
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/report/reports` | List reports |
| POST | `/api/v1/report/reports` | Create report |
| GET | `/api/v1/report/reports/{id}` | Get report result |
| POST | `/api/v1/report/reports/{id}/export` | Export report (CSV, PDF, Excel) |
| POST | `/api/v1/report/reports/{id}/schedule` | Schedule report delivery |
| GET | `/api/v1/report/dashboards` | List dashboards |
| GET | `/api/v1/report/dashboards/{id}` | Get dashboard data |
| POST | `/api/v1/report/dashboards` | Create dashboard |
| GET | `/api/v1/report/kpis` | List KPIs |
| POST | `/api/v1/report/kpis` | Define KPI |
| GET | `/api/v1/report/insights` | Get AI-driven insights and anomalies |

### Integration Service
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/integrations` | List integrations |
| POST | `/api/v1/integrations` | Create integration |
| GET | `/api/v1/integrations/{id}` | Get integration details |
| PUT | `/api/v1/integrations/{id}` | Update integration |
| DELETE | `/api/v1/integrations/{id}` | Disable integration |
| POST | `/api/v1/integrations/{id}/test` | Test integration connection |
| POST | `/api/v1/integrations/{id}/sync` | Trigger manual sync |
| GET | `/api/v1/integrations/{id}/logs` | Get integration sync logs |
| POST | `/api/v1/integrations/import` | Upload file for import |
| GET | `/api/v1/integrations/jobs/{id}` | Get import/export job status |
| GET | `/api/v1/integrations/connectors` | List available connector types |
| POST | `/api/v1/integrations/webhooks` | Register webhook endpoint |

---

*See [Security](security.md) for authentication and authorization details.*
*See [Events](events.md) for event schemas and async communication patterns.*
