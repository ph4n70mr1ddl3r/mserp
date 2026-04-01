# Property Management

Property and lease administration, tenant management, rent roll, maintenance, and property financials for landlord-side real estate operations, managed by the Commerce Service.

## Overview

Property Management provides comprehensive landlord-side real estate management covering property and lease administration, tenant lifecycle management, rent roll generation with CAM (Common Area Maintenance) charge allocation, maintenance request management, property financial reporting (NOI, cap rate, occupancy), and space management with floor plan configuration. This module complements Finance Service's lessee-side lease accounting (ASC 842) by managing the landlord perspective. Properties can be exposed as sellable products through Commerce's catalog, and all financial transactions integrate with Finance's GL for landlord revenue recognition and property accounting.

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Property Administration** | Commerce Service (Rust) | Property registry with building details, unit definitions, floor plans, amenity tracking, and property classification |
| **Lease Administration** | Commerce Service (Rust) | Lease creation with term dates, rent schedule, escalation clauses, renewal options, and tenant obligations. Lease status lifecycle management |
| **Tenant Management** | Commerce Service (Rust) | Tenant onboarding, contact management, communication preferences, lease history, and tenant portal access |
| **Rent Roll & CAM** | Commerce Service (Rust) | Monthly rent roll generation, CAM charge calculation based on pro-rata share, expense pool management, and annual reconciliation |
| **Maintenance Requests** | Commerce Service (Rust) | Tenant-submitted and internal maintenance requests with priority, category, vendor assignment, SLA tracking, and completion workflow |
| **Property Financials** | Commerce Service (Rust) | NOI calculation, cap rate analysis, occupancy reporting, budget-vs-actual tracking, and cash flow projection |
| **Space Management** | Commerce Service (Rust) | Floor plan configuration, space classification, availability tracking, and move-in/move-out management |
| **Storage** | PostgreSQL + MinIO | Properties, leases, tenants, and financials in PostgreSQL; floor plans and property documents in MinIO |

## Business Rules

| Rule | Description |
|------|-------------|
| **BR-PM-001** | A lease cannot overlap with another active lease for the same space unit. Lease start date must be on or after the previous lease end date for the unit. |
| **BR-PM-002** | CAM charges are calculated based on the tenant's pro-rata share (rentable square footage / total rentable square footage) of the expense pool. |
| **BR-PM-003** | Rent escalation clauses are applied automatically on the escalation effective date. Percentage-based escalations compound on the prior period's rent. |
| **BR-PM-004** | Maintenance requests with safety or habitability category must be acknowledged within 4 hours and resolved within 24 hours. SLA escalation is automatic. |
| **BR-PM-005** | NOI is calculated as: Gross Rental Income + Other Income - Operating Expenses - Vacancy Loss. Cap rate = NOI / Property Value. |
| **BR-PM-006** | Annual CAM reconciliation must be completed within 90 days of fiscal year-end. Over/under payments are credited or invoiced accordingly. |

## Data Model

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `properties` | `id, tenant_id, name, address{}, property_type, total_sqft, total_units, year_built, class, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | Has many `property_units`, `leases` |
| `property_units` | `id, tenant_id, property_id, unit_number, floor, sqft, unit_type, market_rent, status, created_at, updated_at, version, is_deleted` | Belongs to property, referenced by `leases` |
| `leases` | `id, tenant_id, property_id, unit_id, tenant_contact_id, start_date, end_date, base_rent, rent_schedule{}, escalation{}, cam_share_pct, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to property, unit, tenant |
| `tenants` | `id, tenant_id, name, contact{}, company_name, lease_history_refs, portal_enabled, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | Has many `leases` |
| `cam_charges` | `id, tenant_id, property_id, lease_id, period, expense_pool, pro_rata_share_pct, charge_amount, reconciled, created_at, updated_at, version, is_deleted` | Belongs to property, lease |
| `maintenance_requests` | `id, tenant_id, property_id, unit_id, lease_id, category, priority, description, assigned_vendor_id, status, submitted_at, acknowledged_at, resolved_at, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to property, unit |
| `rent_roll_entries` | `id, tenant_id, property_id, lease_id, period, base_rent, cam_charges, other_charges, total, vacancy_loss, collected, outstanding, created_at, updated_at, version, is_deleted` | Belongs to property, lease |

## Events

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `commerce.lease.created` | `{lease_id, property_id, unit_id, tenant_id, start_date, end_date, base_rent}` | New lease executed | Finance, Report |
| `commerce.lease.terminated` | `{lease_id, property_id, unit_id, termination_date, reason}` | Lease ended early or expired | Finance, Report |
| `commerce.rent-roll.generated` | `{property_id, period, total_rent, total_cam, occupancy_pct, vacancy_loss}` | Monthly rent roll generated | Finance, Report |
| `commerce.maintenance.submitted` | `{request_id, property_id, unit_id, category, priority}` | Maintenance request created | Notification, Workflow |
| `commerce.maintenance.resolved` | `{request_id, resolved_at, resolution_summary, cost}` | Maintenance completed | Finance, Report |
| `commerce.property.financials.updated` | `{property_id, period, noi, cap_rate, occupancy_pct}` | Property financial snapshot calculated | Report, Dashboard |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `finance.payment.received` | Finance Service | Update rent roll collection status |
| `finance.invoice.posted` | Finance Service | Record CAM reconciliation charges |
| `workflow.step.approved` | Workflow Service | Finalize lease approval or maintenance vendor assignment |
| `config.changed` | Config Service | Update maintenance SLAs, CAM calculation rules |
| `platform.document.uploaded` | Platform Service | Attach lease documents and floor plans |

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/v1/properties` | Create a new property |
| GET | `/api/v1/properties` | List properties with type and status filter |
| GET | `/api/v1/properties/{id}` | Retrieve property with units and occupancy |
| POST | `/api/v1/properties/{id}/units` | Add or configure a unit |
| POST | `/api/v1/leases` | Create a new lease |
| GET | `/api/v1/leases` | List leases with property and status filter |
| POST | `/api/v1/leases/{id}/terminate` | Terminate a lease |
| GET | `/api/v1/properties/{id}/rent-roll` | Generate rent roll for a period |
| POST | `/api/v1/maintenance-requests` | Submit a maintenance request |
| GET | `/api/v1/maintenance-requests` | List maintenance requests |
| GET | `/api/v1/properties/{id}/financials` | Retrieve property financial snapshot |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Revenue Recognition | Finance Service | Event-driven | Landlord revenue recognition for lease payments |
| GL Accounting | Finance Service | Internal API | Property GL accounts, rent and CAM journal entries |
| Property as Product | Commerce Service | Internal API | Properties and units exposed as catalog products for listing |
| Document Management | Platform Service | Internal API | Lease documents, floor plans, property photos |
| Approval Workflows | Workflow Service | Event-driven | Lease approval and maintenance vendor assignment |
| Notification | Platform Service | Event-driven | Lease expiry, maintenance updates, rent reminders |
| Report Service | Report Service | Internal API | Portfolio analytics, occupancy trends, NOI reporting |

## Cross-References

- [Finance Service](../06-services/finance.md) — Landlord revenue recognition, property GL accounts
- [Commerce Service](../06-services/commerce.md) — Commerce service specification, property as product
- [Platform Service](../06-services/platform.md) — Document management for leases and floor plans
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
