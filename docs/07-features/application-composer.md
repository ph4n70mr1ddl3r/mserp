# Application Composer

Runtime extension framework with custom fields, custom objects, serverless scripts, and UI page composition for tenant-isolated application customization, managed by the Platform Service.

## Overview

Application Composer provides a no-code/low-code extension framework that allows tenants to customize and extend MSERP without modifying core application code. The module supports custom fields on any standard entity, fully custom objects with their own CRUD operations, serverless scripts (hooks) that execute on business events, a drag-and-drop UI page composer, and event subscription management for custom workflows. All extensions are tenant-isolated, version-controlled, and governed by lifecycle management to ensure safe upgrades.

## Data Flow Diagram

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  Extension        │────▶│  Composer Engine  │────▶│  Runtime Execution  │
│  Designer (UI)    │     │  ┌─────────────┐ │     │  ┌───────────────┐  │
└──────────────────┘     │  │ Validator    │ │     │  │ Script Runner │  │
                         │  └──────┬──────┘ │     │  │ (sandboxed)   │  │
┌──────────────────┐     │         │        │     │  └───────┬───────┘  │
│  Custom Field    │────▶│  ┌──────▼──────┐ │     │  ┌───────▼───────┐  │
│  Definitions     │     │  │ Schema      │ │     │  │ Hook Executor │  │
└──────────────────┘     │  │ Generator   │ │     │  │ (before/after)│  │
                         │  └─────────────┘ │     │  └───────┬───────┘  │
┌──────────────────┐     │  ┌─────────────┐ │     │          │          │
│  Custom Object   │────▶│  │ CRUD        │ │     │  ┌───────▼───────┐  │
│  Definitions     │     │  │ Generator   │ │     │  │ Page Renderer │  │
└──────────────────┘     │  └─────────────┘ │     │  │ (dynamic UI)  │  │
                         └──────────────────┘     └──────────┼──────────┘
┌──────────────────┐     ┌──────────────────┐                │
│  Event Bus       │────▶│  Subscription    │────▶  ┌─────────▼──────────┐
│  (RabbitMQ)      │     │  Manager         │      │  Standard Services  │
└──────────────────┘     └──────────────────┘      │  (with extensions)  │
                                                   └────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Custom Fields** | Platform Service (Rust) | Type-safe custom field definitions on standard entities with validation rules, stored in EAV pattern or JSON columns |
| **Custom Objects** | Platform Service | Tenant-defined entities with full CRUD, relationships, views, and access control independent of standard entities |
| **Serverless Scripts** | Platform Service (Rust + WASM) | Sandbox-executed scripts triggered by hooks (before/after) on entity operations, written in supported scripting languages |
| **UI Page Composer** | Platform Service | Drag-and-drop page builder with component library, data binding, and responsive layout for custom pages |
| **Workflow Triggers** | Platform Service + Workflow Service | Custom event-driven triggers that launch workflows or scripts on business events |
| **Event Subscriptions** | Platform Service | Tenant-managed subscriptions to domain events with configurable filters and delivery to scripts or webhooks |
| **Lifecycle Management** | Platform Service | Extension versioning, dependency tracking, upgrade compatibility checks, and rollback support |
| **Storage** | PostgreSQL | Extension metadata in Platform database; custom object data in tenant-scoped schemas |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Extension Manager | Extension lifecycle, versioning, and dependency management | Platform Service |
| Custom Field Engine | Field definition, validation, storage, and retrieval | Platform Service |
| Custom Object Engine | Dynamic entity CRUD, schema migration, and indexing | Platform Service |
| Script Runtime | Sandboxed WASM execution with resource limits | Platform Service |
| Hook Executor | Before/after hook invocation on entity operations | Platform Service |
| Page Composer | Component assembly, data binding, and page rendering | Platform Service |
| Subscription Manager | Event subscription CRUD and delivery routing | Platform Service |
| Compatibility Checker | Extension upgrade validation against core version | Platform Service |

## Script Hook Points

| Hook Type | Timing | Use Cases |
|-----------|--------|-----------|
| Before Create | Pre-validation | Data enrichment, default calculation, cross-field validation |
| After Create | Post-commit | Notifications, derivative record creation, integration sync |
| Before Update | Pre-validation | Change validation, approval routing, state machine enforcement |
| After Update | Post-commit | Change logging, cascade updates, event publishing |
| Before Delete | Pre-validation | Dependency check, soft-delete enforcement, archive trigger |
| After Delete | Post-commit | Cleanup, reference resolution, audit logging |
| Custom Event | On subscription | Event-driven processing, webhook delivery, data transformation |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| All Services | All | Extension points | Hook injection points on standard entity operations |
| Workflow Engine | Workflow Service | Event-driven | Custom workflow triggers from extension events |
| Event Bus | Platform Service (RabbitMQ) | Event-driven | Event subscription and delivery to scripts |
| Identity Service | Identity Service | Internal API | Permission checks for extension access control |
| Tenant Service | Tenant Service | Internal API | Tenant isolation and extension scoping |
| Config Service | Config Service | Internal API | Extension configuration parameters |
| Notification Service | Platform Service | Event-driven | Script-triggered notifications |
| Audit Trail | Platform Service | Event-driven | Full audit log of extension changes and executions |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `platform.composer.extension.created` | `{extension_id, tenant_id, type, name, version}` | New extension package published | Audit Trail, Compatibility Checker |
| `platform.composer.extension.updated` | `{extension_id, tenant_id, old_version, new_version}` | Extension version updated | Audit Trail, Dependency Manager |
| `platform.composer.script.executed` | `{script_id, tenant_id, hook_type, entity_type, duration_ms, status}` | Script hook execution completes | Script Analytics, Rate Limiter |
| `platform.composer.page.published` | `{page_id, tenant_id, slug, version}` | Custom page published | CDN Cache, Audit Trail |
| `platform.composer.custom-object.created` | `{object_id, tenant_id, entity_name, schema_version}` | Custom object definition created | Custom Object Engine |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `{domain}.{entity}.created` | Any service | Execute before/after create hooks |
| `{domain}.{entity}.updated` | Any service | Execute before/after update hooks |
| `{domain}.{entity}.deleted` | Any service | Execute before/after delete hooks |
| `tenant.tenant.created` | Tenant Service | Initialize tenant extension namespace |
| `identity.role.updated` | Identity Service | Revalidate extension access controls |
| `config.changed` | Config Service | Reload extension configuration |
| `workflow.step.completed` | Workflow Service | Resume script waiting on workflow outcome |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `composer_custom_fields` | `id, tenant_id, entity_type, field_name, field_type, label, validation_rules, is_required, default_value` | Scoped to entity type |
| `composer_custom_objects` | `id, tenant_id, entity_name, display_name, schema, relationships, is_active` | Has associated data table |
| `composer_scripts` | `id, tenant_id, name, language, source_code, hook_type, entity_type, is_active, version` | Belongs to extension |
| `composer_pages` | `id, tenant_id, slug, title, layout, components, data_bindings, is_published, version` | Belongs to extension |
| `composer_extensions` | `id, tenant_id, name, description, version, status, dependencies, published_at` | Contains scripts, pages, custom fields |
| `composer_event_subscriptions` | `id, tenant_id, event_type, filter_rules, target_type, target_id, is_active` | Links events to scripts or webhooks |

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `platform.composer.script.timeout_ms` | 5000 | Maximum script execution time |
| `platform.composer.script.memory_mb` | 64 | Maximum memory per script execution |
| `platform.composer.custom_fields.max_per_entity` | 50 | Maximum custom fields per entity |
| `platform.composer.custom_objects.max_per_tenant` | 100 | Maximum custom objects per tenant |
| `platform.composer.page.max_components` | 50 | Maximum components per custom page |
| `platform.composer.extension.max_per_tenant` | 50 | Maximum extensions per tenant |

## Cross-References

- [Multi-Tenancy](multi-tenancy.md) — Tenant isolation for extensions
- [Workflow Engine](../06-services/workflow.md) — Custom workflow triggers
- [Event Architecture](../04-events/overview.md) — Event subscription patterns
- [Security Architecture](../02-security/overview.md) — Extension sandbox security
- [Data Architecture](../03-data/overview.md) — Custom field storage patterns
- [Platform Service](../06-services/platform.md) — Platform service specification
- [NFR: Performance](../10-planning/nfr.md) — Script execution < 100ms p99
