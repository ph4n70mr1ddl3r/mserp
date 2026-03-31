# API Design Standards

API design rules (REST standards, idempotency, rate limiting, versioning) are defined in SPEC.md §10. This document provides implementation details for response formats, health checks, and bulk operations.

## 1. Standard Response Format

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

## 2. Health Check Endpoints

All services MUST expose standardized health check endpoints. These are used by Kubernetes for liveness and readiness probes.

### 2.1 Endpoints

| Endpoint | Purpose | K8s Probe |
|----------|---------|-----------|
| `GET /healthz` | Liveness — process is running | `livenessProbe` |
| `GET /readyz` | Readiness — can accept traffic | `readinessProbe` |

### 2.2 Liveness (`/healthz`)

Returns `200 OK` if the process is running. No dependency checks. Fails only on unrecoverable errors (e.g., deadlock, fatal panic).

```json
{
  "status": "ok",
  "uptime_seconds": 86400,
  "version": "1.2.3"
}
```

### 2.3 Readiness (`/readyz`)

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

### 2.4 Check Configuration

| Service | Required Checks |
|---------|-----------------|
| All services | Database, Redis |
| Auth, Tenant | + RabbitMQ (connection — publish only, no inbox queue) |
| Config | + RabbitMQ (connection — publish only) |
| Identity | Database, Redis only (does not use RabbitMQ) |
| Commerce, Finance, HCM, Manufacturing, Workflow, CRM, Project, Integration | + RabbitMQ (connection + queue reachable) |
| Platform | + RabbitMQ (connection + queue reachable), Database x2 (`platform_db` + `audit_db`), ONNX Runtime, IDP Service |
| Report | + RabbitMQ (connection + queue reachable), Elasticsearch, DuckDB |

- Each individual dependency check within `/readyz` has a 1-second timeout.
- Checks run concurrently (not sequentially). The overall readiness probe HTTP timeout is 5 seconds.
- Failure of any check returns `503` with `"status": "degraded"` or `"status": "unavailable"`.

## 3. Bulk / Batch API Patterns

### 3.1 Design Principles

| Principle | Rule |
|-----------|------|
| Method | Bulk operations MUST use `POST` (not `GET`) |
| Batch Size | Maximum 100 items per request |
| Idempotency | Each bulk request requires a single `Idempotency-Key` |
| Response | Returns summary with per-item results |

### 3.2 Request Format

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

### 3.3 Response Format

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

### 3.4 Bulk Processing Rules

- **Default (atomic):** All items processed within a single database transaction. If any item fails validation, all items are rolled back.
- **Non-atomic (`Prefer: partial-success` header):** Items processed individually with independent transactions. Partial success is possible. On retry with the same `Idempotency-Key`, previously succeeded items are skipped.
- Bulk operations exceeding 100 items MUST return `413 Payload Too Large`.
- For datasets larger than 100 items, use the Integration Service file-based import (CSV/Excel).

---

*See [Endpoints](endpoints.md) for the API endpoints overview.*
*See [Error Codes](error-codes.md) for the full error code taxonomy.*
