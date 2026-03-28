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
| Async Operations | `202 Accepted` with job reference for long-running operations |

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
| AI/ML inference endpoints | 30 requests | 1 minute |
| Digital assistant endpoints | 60 requests | 1 minute |

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

For long-running operations (data import, large report generation, ML model training), the API returns `202 Accepted` with a job reference:

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
| Platform | + RabbitMQ (connection + queue reachable), Database x2 (`platform_db` + `audit_db`), ONNX Runtime |
| Report | + RabbitMQ (connection + queue reachable), Elasticsearch, DuckDB |

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
| `AUTH_STEP_UP_REQUIRED` | 403 | Step-up authentication required for this operation |
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
| `COMMERCE_CARRIER_NOT_AVAILABLE` | 409 | No carrier available for requested route |
| `COMMERCE_SHIPMENT_INVALID` | 400 | Shipment parameters invalid |
| `COMMERCE_ATP_UNAVAILABLE` | 409 | No available-to-promise quantity for requested date |
| `COMMERCE_CREDIT_HOLD` | 409 | Order held due to credit limit exceeded |
| `COMMERCE_CONFIG_INVALID` | 400 | Product configuration violates constraint rules |
| `COMMERCE_CONFIG_INCOMPLETE` | 400 | Product configuration is incomplete |
| `COMMERCE_SUBSCRIPTION_INACTIVE` | 409 | Subscription is not in an active state |
| `COMMERCE_SUBSCRIPTION_AMENDMENT_INVALID` | 400 | Subscription amendment parameters invalid |
| `COMMERCE_DROPSHIP_UNAVAILABLE` | 409 | Drop ship not available for this product/supplier combination |

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
| `FINANCE_EXPENSE_POLICY_VIOLATION` | 400 | Expense violates corporate policy |
| `FINANCE_CONTRACT_EXPIRED` | 409 | Contract has expired |
| `FINANCE_CONTRACT_NOT_APPROVED` | 409 | Contract requires approval before execution |
| `FINANCE_PLAN_LOCKED` | 409 | Financial plan is locked for editing |
| `FINANCE_REVENUE_ALREADY_RECOGNIZED` | 409 | Revenue already recognized for this obligation |
| `FINANCE_REVENUE_ALLOCATION_INVALID` | 400 | Revenue allocation amounts do not sum to transaction price |
| `FINANCE_SSP_MISSING` | 400 | Standalone selling price not available for allocation |

### 8.4 HR Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `HR_EMPLOYEE_EXISTS` | 409 | Employee with this ID already exists |
| `HR_LEAVE_BALANCE_EXCEEDED` | 409 | Not enough leave balance |
| `HR_ATTENDANCE_CONFLICT` | 409 | Overlapping attendance record |
| `HR_PAYROLL_ALREADY_PROCESSED` | 409 | Payroll for this period already processed |
| `HR_REQUISITION_CLOSED` | 409 | Job requisition is closed |
| `HR_SUCCESSION_PLAN_EXISTS` | 409 | Succession plan already exists for this position |

### 8.5 Manufacturing Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `MFG_BOM_CIRCULAR_REF` | 400 | BOM contains circular reference |
| `MFG_WORK_ORDER_IN_PROGRESS` | 409 | Work order cannot be modified while in progress |
| `MFG_INSUFFICIENT_MATERIALS` | 409 | Not enough materials for production |
| `MFG_CAPACITY_EXCEEDED` | 409 | Work center capacity exceeded for requested time |
| `MFG_QUALITY_CHECK_FAILED` | 409 | Quality check did not pass threshold |
| `MFG_ECO_PENDING_APPROVAL` | 409 | Engineering Change Order pending approval |
| `MFG_ASSET_NOT_AVAILABLE` | 409 | Asset not available for scheduled time |
| `MFG_PLAN_CONSTRAINT_VIOLATION` | 400 | Plan violates material or capacity constraints |

### 8.6 CRM Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `CRM_DUPLICATE_CONTACT` | 409 | Contact already exists with this email |
| `CRM_LEAD_CONVERTED` | 409 | Lead already converted to opportunity/customer |
| `CRM_CAMPAIGN_ACTIVE` | 409 | Campaign is active and cannot be modified |
| `CRM_SURVEY_CLOSED` | 409 | Survey is no longer accepting responses |
| `CRM_SERVICE_ORDER_IN_PROGRESS` | 409 | Service order cannot be modified while in progress |
| `CRM_TERRITORY_CONFLICT` | 409 | Territory assignment conflicts with existing assignment |

### 8.7 Project Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `PROJECT_NOT_FOUND` | 404 | Project does not exist |
| `PROJECT_BUDGET_EXCEEDED` | 409 | Project budget exceeded |
| `PROJECT_TASK_DEPENDENCY` | 409 | Circular task dependency detected |

### 8.8 Report Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `REPORT_GENERATION_FAILED` | 500 | Report generation encountered an error |
| `REPORT_NOT_FOUND` | 404 | Report definition not found |
| `REPORT_SCHEDULE_CONFLICT` | 409 | Report schedule conflicts with existing schedule |
| `DASHBOARD_LAYOUT_INVALID` | 400 | Dashboard layout configuration invalid |
| `ML_MODEL_NOT_TRAINED` | 409 | ML model has not been trained yet |
| `ML_MODEL_DEPLOYMENT_FAILED` | 500 | ML model deployment failed |
| `PROCESS_MINING_INSUFFICIENT_DATA` | 400 | Insufficient event data for process mining analysis |

### 8.9 Workflow Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `WORKFLOW_DEFINITION_INVALID` | 400 | Workflow definition contains invalid BPMN |
| `WORKFLOW_INSTANCE_STALE` | 409 | Workflow instance state has changed |
| `WORKFLOW_STEP_NOT_ACTIONABLE` | 409 | Workflow step cannot be acted upon in current state |
| `WORKFLOW_TEMPLATE_NOT_FOUND` | 404 | Workflow template not found |
| `WORKFLOW_ESCALATION_FAILED` | 500 | Workflow escalation action failed |

### 8.10 Platform / GRC Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `PLATFORM_FILE_TOO_LARGE` | 413 | File exceeds maximum upload size |
| `PLATFORM_FILE_VIRUS_DETECTED` | 400 | Uploaded file contains malware |
| `PLATFORM_SOD_CONFLICT` | 409 | Segregation of Duties conflict detected |
| `PLATFORM_COMPLIANCE_VIOLATION` | 403 | Operation violates compliance policy |
| `PLATFORM_MASKING_RULE_INVALID` | 400 | Data masking rule configuration invalid |
| `PLATFORM_JOB_ALREADY_RUNNING` | 409 | Scheduled job is already running |
| `PLATFORM_JOB_NOT_FOUND` | 404 | Scheduled job definition not found |
| `PLATFORM_SCREENING_MATCH_FOUND` | 409 | Trade compliance screening flagged a restricted party match |

### 8.11 Integration Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `INTEGRATION_SYNC_IN_PROGRESS` | 409 | Sync already in progress for this integration |
| `INTEGRATION_CONNECTION_FAILED` | 502 | Failed to connect to external system |
| `INTEGRATION_AUTH_EXPIRED` | 401 | OAuth token or credential expired for external system |
| `INTEGRATION_MAPPING_INVALID` | 400 | Field mapping configuration invalid |
| `INTEGRATION_SCREENING_MATCH` | 409 | Restricted party match found during screening |
| `INTEGRATION_LICENSE_EXPIRED` | 409 | Export license has expired |
| `INTEGRATION_CLASSIFICATION_REQUIRED` | 400 | Export control classification required |

### 8.12 Error Code Rules

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
| POST | `/api/v1/auth/step-up` | Step-up authentication (elevated privilege) |
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
| POST | `/api/v1/identity/api-keys` | Create API key |
| GET | `/api/v1/identity/api-keys` | List API keys |
| DELETE | `/api/v1/identity/api-keys/{id}` | Revoke API key |

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
| PUT | `/api/v1/tenant/data-residency` | Update data residency settings (Super Admin only) |

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

### Commerce Service (Sales + Inventory + PIM + Transportation)
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
| POST | `/api/v1/commerce/products/{id}/submit-approval` | Submit product for PIM approval |
| POST | `/api/v1/commerce/products/{id}/publish` | Publish product to channels |
| GET | `/api/v1/commerce/price-lists` | List price lists |
| POST | `/api/v1/commerce/price-lists` | Create price list |
| POST | `/api/v1/commerce/pricing/calculate` | Calculate price for a product/customer/qty combination |
| GET | `/api/v1/commerce/warehouses` | List warehouses |
| GET | `/api/v1/commerce/stock` | Get stock levels |
| POST | `/api/v1/commerce/transfers` | Create stock transfer |
| POST | `/api/v1/commerce/adjustments` | Stock adjustment |
| GET | `/api/v1/commerce/carriers` | List carriers |
| POST | `/api/v1/commerce/carriers` | Register carrier |
| GET | `/api/v1/commerce/shipments` | List shipments |
| POST | `/api/v1/commerce/shipments` | Create shipment |
| GET | `/api/v1/commerce/shipments/{id}/tracking` | Get shipment tracking |
| POST | `/api/v1/commerce/shipments/{id}/dispatch` | Dispatch shipment to carrier |
| POST | `/api/v1/commerce/shipments/{id}/deliver` | Confirm delivery with POD |
| GET | `/api/v1/commerce/atp/check` | Check ATP/CTP availability |
| POST | `/api/v1/commerce/configurator/configure` | Configure a product |
| GET | `/api/v1/commerce/configurator/models` | List configurator models |
| GET | `/api/v1/commerce/credit/{customerId}` | Get customer credit status |
| PUT | `/api/v1/commerce/credit/{customerId}/limit` | Update credit limit |
| POST | `/api/v1/commerce/credit/{customerId}/hold` | Apply credit hold |
| POST | `/api/v1/commerce/credit/{customerId}/release` | Release credit hold |
| GET | `/api/v1/commerce/subscriptions` | List subscriptions |
| POST | `/api/v1/commerce/subscriptions` | Create subscription |
| GET | `/api/v1/commerce/subscriptions/{id}` | Get subscription |
| POST | `/api/v1/commerce/subscriptions/{id}/amend` | Amend subscription |
| POST | `/api/v1/commerce/subscriptions/{id}/cancel` | Cancel subscription |
| POST | `/api/v1/commerce/subscriptions/{id}/renew` | Renew subscription |
| POST | `/api/v1/commerce/dropship/orders` | Create drop ship order |
| POST | `/api/v1/commerce/b2b/portal/orders` | Create B2B portal order (customer self-service) |
| GET | `/api/v1/commerce/b2b/portal/orders` | List B2B portal orders |
| GET | `/api/v1/commerce/b2b/portal/reorder-templates` | List reorder templates |
| POST | `/api/v1/commerce/b2b/portal/reorder-templates` | Create reorder template |
| POST | `/api/v1/commerce/b2b/portal/reorder/{id}` | Execute reorder from template |
| GET | `/api/v1/commerce/logistics/tracking/{id}` | Get real-time shipment tracking with geofence data |
| GET | `/api/v1/commerce/logistics/condition/{shipmentId}` | Get shipment condition monitoring data |
| GET | `/api/v1/commerce/logistics/eta/{shipmentId}` | Get predictive ETA for shipment |
| POST | `/api/v1/commerce/logistics/geofences` | Create geofence for logistics monitoring |
| GET | `/api/v1/commerce/logistics/exceptions` | List logistics exceptions (delays, deviations) |

### Finance Service (Finance + Procurement + Treasury + Expenses + CLM + EPM)
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
| GET | `/api/v1/finance/treasury/cash-position` | Get current cash position |
| GET | `/api/v1/finance/treasury/bank-accounts` | List bank accounts |
| POST | `/api/v1/finance/treasury/payment-batches` | Create payment batch |
| POST | `/api/v1/finance/treasury/payment-batches/{id}/approve` | Approve payment batch |
| POST | `/api/v1/finance/treasury/payment-batches/{id}/execute` | Execute payment batch |
| GET | `/api/v1/finance/expenses` | List expense reports |
| POST | `/api/v1/finance/expenses` | Create expense report |
| GET | `/api/v1/finance/expenses/{id}` | Get expense report |
| POST | `/api/v1/finance/expenses/{id}/submit` | Submit for approval |
| POST | `/api/v1/finance/expenses/{id}/approve` | Approve expense report |
| POST | `/api/v1/finance/expenses/{id}/reject` | Reject expense report |
| GET | `/api/v1/finance/contracts` | List contracts |
| POST | `/api/v1/finance/contracts` | Create contract |
| GET | `/api/v1/finance/contracts/{id}` | Get contract |
| PUT | `/api/v1/finance/contracts/{id}` | Update contract |
| POST | `/api/v1/finance/contracts/{id}/submit` | Submit for approval |
| POST | `/api/v1/finance/contracts/{id}/approve` | Approve contract |
| POST | `/api/v1/finance/contracts/{id}/sign` | Record e-signature |
| GET | `/api/v1/finance/contracts/{id}/obligations` | List contract obligations |
| GET | `/api/v1/finance/plans` | List financial plans (EPM) |
| POST | `/api/v1/finance/plans` | Create financial plan |
| PUT | `/api/v1/finance/plans/{id}` | Update plan values |
| POST | `/api/v1/finance/plans/{id}/forecast` | Generate forecast |
| POST | `/api/v1/finance/plans/{id}/what-if` | Run what-if scenario |
| GET | `/api/v1/finance/plans/{id}/variance` | Get variance analysis |
| GET | `/api/v1/finance/revenue/contracts` | List revenue recognition contracts |
| POST | `/api/v1/finance/revenue/contracts` | Create revenue contract |
| GET | `/api/v1/finance/revenue/contracts/{id}/obligations` | List performance obligations |
| GET | `/api/v1/finance/revenue/contracts/{id}/schedule` | Get revenue recognition schedule |
| POST | `/api/v1/finance/revenue/contracts/{id}/recognize` | Recognize revenue for period |
| GET | `/api/v1/finance/revenue/waterfall` | Get revenue waterfall report |
| GET | `/api/v1/finance/revenue/disclosures` | Get revenue disclosure data |
| POST | `/api/v1/finance/sourcing/events` | Create sourcing event (RFI/RFP/RFQ) |
| GET | `/api/v1/finance/sourcing/events` | List sourcing events |
| GET | `/api/v1/finance/sourcing/events/{id}/bids` | List bids for sourcing event |
| POST | `/api/v1/finance/sourcing/events/{id}/award` | Award sourcing event |
| GET | `/api/v1/finance/supplier-risk/scores` | List supplier risk scores |
| GET | `/api/v1/finance/supplier-risk/scores/{supplierId}` | Get supplier risk detail |
| GET | `/api/v1/finance/supplier-risk/alerts` | List supplier risk alerts |
| POST | `/api/v1/finance/supplier-risk/mitigation-plans` | Create risk mitigation plan |
| GET | `/api/v1/finance/reconciliation/matches` | List reconciliation matches |
| POST | `/api/v1/finance/reconciliation/auto-match` | Run auto-matching for account |
| POST | `/api/v1/finance/reconciliation/exceptions` | Create reconciliation exception |
| GET | `/api/v1/finance/close/tasks` | List financial close tasks |
| POST | `/api/v1/finance/close/tasks` | Create close task |
| POST | `/api/v1/finance/close/tasks/{id}/complete` | Mark close task complete |
| GET | `/api/v1/finance/profitability/customer` | Get customer profitability analysis |
| GET | `/api/v1/finance/profitability/product` | Get product profitability analysis |
| GET | `/api/v1/finance/profitability/cost-to-serve/{customerId}` | Get cost-to-serve analysis |
| POST | `/api/v1/finance/profitability/what-if` | Run profitability what-if scenario |

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
| GET | `/api/v1/hr/payroll/countries` | List supported payroll countries |
| POST | `/api/v1/hr/payroll/multi-country/run` | Start multi-country payroll cycle |
| GET | `/api/v1/hr/recruitment/requisitions` | List job requisitions |
| POST | `/api/v1/hr/recruitment/requisitions` | Create job requisition |
| GET | `/api/v1/hr/recruitment/applicants` | List applicants |
| POST | `/api/v1/hr/performance/reviews` | Create performance review |
| GET | `/api/v1/hr/org-chart` | Get organization chart |
| GET | `/api/v1/hr/training/courses` | List training courses |
| GET | `/api/v1/hr/talent/reviews` | List talent reviews |
| POST | `/api/v1/hr/talent/reviews` | Initiate talent review |
| GET | `/api/v1/hr/talent/succession-plans` | List succession plans |
| POST | `/api/v1/hr/talent/succession-plans` | Create succession plan |
| POST | `/api/v1/hr/workforce/simulate` | Run workforce modeling simulation |
| GET | `/api/v1/hr/benefits/plans` | List benefit plans |
| POST | `/api/v1/hr/benefits/enroll` | Enroll employee in benefits |

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
| GET | `/api/v1/manufacturing/quality/spc` | Get SPC charts and data |
| GET | `/api/v1/manufacturing/work-centers` | List work centers |
| GET | `/api/v1/manufacturing/routing` | List routing definitions |
| GET | `/api/v1/manufacturing/plm/revisions` | List product revisions |
| POST | `/api/v1/manufacturing/plm/revisions` | Create product revision |
| GET | `/api/v1/manufacturing/plm/eco` | List engineering change orders |
| POST | `/api/v1/manufacturing/plm/eco` | Create ECO |
| POST | `/api/v1/manufacturing/plm/eco/{id}/approve` | Approve ECO |
| POST | `/api/v1/manufacturing/plm/eco/{id}/implement` | Implement ECO |
| GET | `/api/v1/manufacturing/assets` | List enterprise assets |
| POST | `/api/v1/manufacturing/assets` | Register asset |
| GET | `/api/v1/manufacturing/assets/{id}` | Get asset details |
| POST | `/api/v1/manufacturing/assets/{id}/maintenance` | Schedule asset maintenance |
| GET | `/api/v1/manufacturing/assets/{id}/history` | Get asset maintenance history |
| GET | `/api/v1/manufacturing/planning/mrp` | Get MRP results |
| POST | `/api/v1/manufacturing/planning/mrp/run` | Run MRP |
| POST | `/api/v1/manufacturing/planning/simulate` | Run ASCP simulation |
| GET | `/api/v1/manufacturing/sustainability/emissions` | Get production emissions data |
| GET | `/api/v1/manufacturing/iot/devices` | List IoT devices |
| POST | `/api/v1/manufacturing/iot/devices` | Register IoT device |
| GET | `/api/v1/manufacturing/iot/devices/{id}/telemetry` | Get device telemetry |
| POST | `/api/v1/manufacturing/iot/alerts` | Create IoT alert rule |
| GET | `/api/v1/manufacturing/digital-twins` | List digital twins |
| GET | `/api/v1/manufacturing/digital-twins/{assetId}` | Get asset digital twin state |
| POST | `/api/v1/manufacturing/digital-twins/{assetId}/simulate` | Run digital twin simulation |
| GET | `/api/v1/manufacturing/digital-twins/{assetId}/predictions` | Get predictive maintenance predictions |

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
| GET | `/api/v1/crm/cases` | List service cases |
| POST | `/api/v1/crm/cases` | Create service case |
| POST | `/api/v1/crm/cases/{id}/resolve` | Resolve service case |
| GET | `/api/v1/crm/service-orders` | List service orders |
| POST | `/api/v1/crm/service-orders` | Create service order |
| GET | `/api/v1/crm/service-orders/{id}` | Get service order |
| POST | `/api/v1/crm/service-orders/{id}/dispatch` | Dispatch technician |
| POST | `/api/v1/crm/service-orders/{id}/complete` | Complete service order |
| GET | `/api/v1/crm/surveys` | List surveys |
| POST | `/api/v1/crm/surveys` | Create survey |
| POST | `/api/v1/crm/surveys/{id}/responses` | Submit survey response |
| GET | `/api/v1/crm/surveys/{id}/analytics` | Get survey analytics |
| GET | `/api/v1/crm/territories` | List sales territories |
| POST | `/api/v1/crm/territories` | Create territory |
| PUT | `/api/v1/crm/territories/{id}/assign` | Assign accounts to territory |
| GET | `/api/v1/crm/quotas` | List sales quotas |
| POST | `/api/v1/crm/quotas` | Allocate quotas |
| GET | `/api/v1/crm/cdp/profiles` | List unified customer profiles (CDP) |
| GET | `/api/v1/crm/cdp/profiles/{id}` | Get unified customer profile |
| POST | `/api/v1/crm/cdp/segments` | Create customer segment |
| GET | `/api/v1/crm/cdp/segments` | List customer segments |
| POST | `/api/v1/crm/cdp/journeys` | Create customer journey |
| GET | `/api/v1/crm/cdp/journeys/{id}/analytics` | Get journey analytics |
| GET | `/api/v1/crm/cdp/identity-resolution/{id}` | Get identity resolution graph |

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
| GET | `/api/v1/projects/{id}/evm` | Get Earned Value Management metrics |
| GET | `/api/v1/programs` | List programs |
| POST | `/api/v1/programs` | Create program |

### Platform Service (Notification + File + Audit + Digital Assistant + App Builder + GRC)
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
| POST | `/api/v1/platform/files/{id}/ocr` | Extract text via OCR |
| GET | `/api/v1/platform/audit` | Query audit log |
| GET | `/api/v1/platform/audit/entity/{type}/{id}` | Get audit trail for entity |
| POST | `/api/v1/platform/assistant/query` | Query digital assistant (natural language) |
| GET | `/api/v1/platform/assistant/history` | Get conversation history |
| POST | `/api/v1/platform/assistant/feedback` | Submit feedback on assistant response |
| GET | `/api/v1/platform/app-builder/apps` | List custom applications |
| POST | `/api/v1/platform/app-builder/apps` | Create custom application |
| GET | `/api/v1/platform/app-builder/apps/{id}` | Get application definition |
| PUT | `/api/v1/platform/app-builder/apps/{id}` | Update application |
| POST | `/api/v1/platform/app-builder/apps/{id}/publish` | Publish application |
| POST | `/api/v1/platform/app-builder/objects` | Create custom business object |
| GET | `/api/v1/platform/app-builder/objects` | List custom business objects |
| GET | `/api/v1/platform/grc/sod-rules` | List SoD rules |
| POST | `/api/v1/platform/grc/sod-rules` | Create SoD rule |
| GET | `/api/v1/platform/grc/sod-conflicts` | List SoD conflicts |
| GET | `/api/v1/platform/grc/risks` | List GRC risks |
| POST | `/api/v1/platform/grc/risks` | Register GRC risk |
| POST | `/api/v1/platform/grc/assessments` | Run compliance assessment |
| GET | `/api/v1/platform/grc/incidents` | List GRC incidents |
| POST | `/api/v1/platform/grc/incidents` | Create GRC incident |
| GET | `/api/v1/platform/data-masking/rules` | List data masking rules |
| POST | `/api/v1/platform/data-masking/rules` | Create data masking rule |
| POST | `/api/v1/platform/data-masking/subset` | Generate masked data subset for non-prod |
| GET | `/api/v1/platform/scheduler/jobs` | List scheduled jobs |
| POST | `/api/v1/platform/scheduler/jobs` | Create scheduled job |
| GET | `/api/v1/platform/scheduler/jobs/{id}` | Get job definition |
| PUT | `/api/v1/platform/scheduler/jobs/{id}` | Update job definition |
| POST | `/api/v1/platform/scheduler/jobs/{id}/trigger` | Trigger manual job execution |
| GET | `/api/v1/platform/scheduler/jobs/{id}/executions` | Get job execution history |
| GET | `/api/v1/platform/knowledge/articles` | List knowledge articles |
| POST | `/api/v1/platform/knowledge/articles` | Create knowledge article |
| GET | `/api/v1/platform/knowledge/articles/{id}` | Get article |
| PUT | `/api/v1/platform/knowledge/articles/{id}` | Update article |
| POST | `/api/v1/platform/knowledge/articles/{id}/publish` | Publish article |
| GET | `/api/v1/platform/knowledge/search` | Search knowledge base |
| POST | `/api/v1/platform/signatures/request` | Request digital signature |
| GET | `/api/v1/platform/signatures/{id}` | Get signature status |
| POST | `/api/v1/platform/signatures/{id}/sign` | Sign document |
| GET | `/api/v1/platform/rpa/bots` | List RPA bots |
| POST | `/api/v1/platform/rpa/bots` | Create RPA bot |
| GET | `/api/v1/platform/rpa/bots/{id}` | Get bot definition |
| POST | `/api/v1/platform/rpa/bots/{id}/execute` | Execute RPA bot |
| GET | `/api/v1/platform/rpa/bots/{id}/executions` | Get bot execution history |
| GET | `/api/v1/platform/collaboration/channels` | List collaboration channels |
| POST | `/api/v1/platform/collaboration/channels` | Create channel |
| POST | `/api/v1/platform/collaboration/channels/{id}/messages` | Send message |
| GET | `/api/v1/platform/collaboration/channels/{id}/messages` | List channel messages |
| GET | `/api/v1/platform/collaboration/tasks` | List collaboration tasks |
| POST | `/api/v1/platform/collaboration/tasks` | Create collaboration task |
| GET | `/api/v1/platform/iot/devices` | List IoT devices (device registry) |
| POST | `/api/v1/platform/iot/devices` | Register IoT device |
| GET | `/api/v1/platform/iot/devices/{id}` | Get device details |
| POST | `/api/v1/platform/iot/devices/{id}/certificates` | Issue device certificate |
| GET | `/api/v1/platform/self-service/portal` | Get employee self-service portal data |
| GET | `/api/v1/platform/self-service/payslips` | List payslips (self-service) |
| POST | `/api/v1/platform/self-service/time-off` | Submit time-off request |
| GET | `/api/v1/platform/self-service/benefits` | List available benefits |
| POST | `/api/v1/platform/self-service/expenses` | Submit expense (self-service) |

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
| POST | `/api/v1/report/reports/{id}/export` | Export report (CSV, PDF, Excel, Parquet) |
| POST | `/api/v1/report/reports/{id}/schedule` | Schedule report delivery |
| GET | `/api/v1/report/dashboards` | List dashboards |
| GET | `/api/v1/report/dashboards/{id}` | Get dashboard data |
| POST | `/api/v1/report/dashboards` | Create dashboard |
| GET | `/api/v1/report/kpis` | List KPIs |
| POST | `/api/v1/report/kpis` | Define KPI |
| GET | `/api/v1/report/insights` | Get AI-driven insights and anomalies |
| GET | `/api/v1/report/esg/emissions` | Get emissions data (Scope 1, 2, 3) |
| GET | `/api/v1/report/esg/scorecards` | Get ESG scorecards |
| POST | `/api/v1/report/esg/reports` | Generate ESG regulatory report |
| GET | `/api/v1/report/carbon/footprint` | Get carbon footprint data |
| POST | `/api/v1/report/carbon/offsets` | Record carbon offset |
| GET | `/api/v1/report/carbon/targets` | Get science-based target progress |
| GET | `/api/v1/report/ml/models` | List ML models |
| POST | `/api/v1/report/ml/models` | Register ML model |
| POST | `/api/v1/report/ml/models/{id}/deploy` | Deploy ML model |
| POST | `/api/v1/report/ml/models/{id}/predict` | Run ML prediction |
| GET | `/api/v1/report/data-lake/datasets` | List data lake datasets |
| POST | `/api/v1/report/data-lake/query` | Query data lake (schema-on-read) |
| GET | `/api/v1/report/process-mining/processes` | List discovered processes |
| GET | `/api/v1/report/process-mining/processes/{id}/conformance` | Get conformance checking results |
| GET | `/api/v1/report/process-mining/bottlenecks` | Get bottleneck analysis |
| POST | `/api/v1/report/process-mining/simulate` | Run process simulation |
| GET | `/api/v1/report/cpm/strategy-map` | Get strategy map |
| GET | `/api/v1/report/cpm/okrs` | List objectives and key results |
| POST | `/api/v1/report/cpm/okrs` | Create OKR |
| GET | `/api/v1/report/cpm/scorecard` | Get balanced scorecard |
| GET | `/api/v1/report/cpm/initiatives` | List strategic initiatives |
| POST | `/api/v1/report/cpm/initiatives` | Create strategic initiative |
| GET | `/api/v1/report/narrative/packages` | List report packages |
| POST | `/api/v1/report/narrative/packages` | Create report package |
| POST | `/api/v1/report/narrative/commentary` | Add commentary to report section |

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
| GET | `/api/v1/integrations/mdm/golden-records` | List golden records |
| GET | `/api/v1/integrations/mdm/golden-records/{entity}/{id}` | Get golden record for entity |
| POST | `/api/v1/integrations/mdm/match` | Run matching on entity records |
| POST | `/api/v1/integrations/mdm/merge` | Merge duplicate records |
| GET | `/api/v1/integrations/mdm/quality` | Get data quality scorecards |
| GET | `/api/v1/integrations/governance/catalog` | Browse data catalog |
| GET | `/api/v1/integrations/governance/lineage/{entity}/{id}` | Get data lineage |
| GET | `/api/v1/integrations/governance/classifications` | List data classifications |
| POST | `/api/v1/integrations/trade-compliance/screen` | Screen entity against denied party lists |
| GET | `/api/v1/integrations/trade-compliance/screening-results` | Get screening results |
| GET | `/api/v1/integrations/trade-compliance/licenses` | List export licenses |
| POST | `/api/v1/integrations/trade-compliance/licenses` | Create export license |
| GET | `/api/v1/integrations/trade-compliance/classifications` | List export control classifications |

---

*See [Security](security.md) for authentication and authorization details.*
*See [Events](events.md) for event schemas and async communication patterns.*
