# AGENTS.md — AI Agent Development Guide

This document instructs AI agents on how to use this specification to build MSERP.
Read this file FIRST before any implementation work.

---

## 1. Specification Hierarchy

Agents MUST follow documents in this order of authority:

1. **SPEC.md** — Authoritative single source of truth. All rules live here or are referenced from here. Conflicts are always resolved in SPEC.md's favor.
2. **AGENTS.md** — This file. Agent workflow and reading order only. Does not duplicate rules.
3. **Service specs** (`docs/06-services/*.md`) — Per-service modules, tables, events, integration points.
4. **Feature specs** (`docs/07-features/*.md`) — Detailed business rules for 36 features. See SPEC.md §12 for the registry.
5. **Cross-cutting specs** (`docs/02-*` through `docs/05-*`) — Security, data, events, API standards.
6. **Infrastructure/dev/planning** (`docs/08-*` through `docs/10-*`) — Operational concerns, conventions, NFRs.

> **Note**: Sub-documents have been reorganized to reference SPEC.md for authoritative rules. They provide ONLY implementation details (full table schemas, event payloads, endpoint definitions). For any rule, always defer to SPEC.md.

### Where to Find Authoritative Rules

| Rule | Source |
|------|--------|
| Service catalog & boundaries | SPEC.md §4 |
| Domain ownership (every feature → one service) | SPEC.md §6 |
| Event naming & namespace conventions | SPEC.md §5.1, `docs/04-events/overview.md` |
| Service inbox bindings | SPEC.md §5.3, `docs/04-events/overview.md` |
| Transactional outbox pattern | SPEC.md §5.6, `docs/04-events/overview.md` |
| Saga pattern & self-binding | SPEC.md §5.9, `docs/04-events/sagas.md` |
| Database-per-service | SPEC.md §3.3, `docs/03-data/overview.md` |
| Standard columns, indexes, RLS | SPEC.md §9, `docs/03-data/overview.md` |
| Migration rules | SPEC.md §9.4, `docs/03-data/overview.md` |
| Multi-tenancy & RLS | SPEC.md §8, `docs/03-data/overview.md` |
| API design standards | SPEC.md §10, `docs/05-api/standards.md` |
| Error handling (RFC 7807) | `docs/05-api/error-codes.md` |
| JWT specification & auth flow | SPEC.md §11 |
| RBAC permission format | SPEC.md §11.3, `docs/02-security/authorization.md` |
| IoT boundary (device registry vs. telemetry) | SPEC.md §7.6 |
| SoD rule boundary | SPEC.md §7.7 |
| MDM golden records | SPEC.md §7.8 |
| CRM → Commerce handoff | SPEC.md §7.1 |
| Commerce → Finance handoff | SPEC.md §7.2 |
| Analytics ownership | SPEC.md §7.3 |
| ML/AI ownership split | SPEC.md §14 |
| Feature-to-service registry (36 specs) | SPEC.md §12 |
| Testing requirements & quality gates | SPEC.md §18, `docs/10-planning/nfr.md` |
| Code conventions | `docs/09-development/conventions.md` |
| Project structure | `docs/09-development/project-structure.md` |
| Technology stack & crate versions | `docs/08-infrastructure/technology.md` |

---

## 2. How to Read the Spec

1. **Start with SPEC.md for the rule.** SPEC.md contains or references every architectural rule. Always read it first.
2. **Go to the sub-document for detail.** SPEC.md gives the rule; sub-docs give the implementation detail (full table schemas, event payloads, endpoint definitions, etc.).
3. **Follow the section cross-references.** SPEC.md sections reference each other (e.g., §5 references §6 for domain names). Follow these links to build a complete picture.

### Reading Order for Any Task

```
SPEC.md (relevant sections)
  → docs/06-services/{service}.md  (owning service spec)
    → docs/07-features/{feature}.md (feature spec, if exists)
      → docs/03-data/overview.md + domain-models.md (data patterns)
        → docs/04-events/overview.md + catalog.md (event patterns)
          → docs/05-api/standards.md + endpoints.md (API rules)
            → docs/02-security/overview.md + authorization.md (auth)
              → docs/09-development/conventions.md (code style)
```

---

## 3. How to Build a Service

Follow these 10 steps in order:

1. **Read SPEC.md §4** (Service Catalog) — understand the service definition, port, database, tier, and which modules it owns.
2. **Read SPEC.md §6** (Domain Ownership) — confirm exact boundary of what this service owns vs. what other services own.
3. **Read `docs/06-services/{service}.md`** — modules, tables, events, integration points for this specific service.
4. **Read `docs/06-services/overview.md`** — service catalog, port assignments, inter-service dependencies.
5. **Read `docs/07-features/`** — check for relevant feature specs that apply to this service's modules.
6. **Read `docs/05-api/standards.md`** and `docs/05-api/endpoints.md` — API design rules and endpoint definitions.
7. **Read `docs/03-data/overview.md`** and `docs/03-data/domain-models.md`** — database patterns, standard columns, domain models.
8. **Read `docs/04-events/overview.md`** and `docs/04-events/catalog.md`** — event patterns, inbox bindings, event catalog.
9. **Read `docs/02-security/overview.md`** and `docs/02-security/authorization.md`** — auth requirements, RBAC permissions.
10. **Read `docs/09-development/project-structure.md`** and `docs/09-development/conventions.md`** — directory layout and code style.

Also reference as needed:
- `docs/08-infrastructure/technology.md` — crate versions and dependencies.
- `docs/04-events/sagas.md` — saga definitions for this service's distributed transactions.
- `docs/05-api/error-codes.md` — error codes for this service's domain.

---

## 4. How to Add a Feature

Follow these 7 steps in order:

1. **Identify the owning service.** Use SPEC.md §6 (Domain Ownership) to confirm which service owns this feature. Every feature belongs to EXACTLY ONE service.
2. **Check for an existing feature spec.** Look in `docs/07-features/` for an existing spec. If one exists, read it for detailed business rules. See SPEC.md §12 for the full registry of 36 feature specs.
3. **Add new events** to `docs/04-events/catalog.md`. Use namespace `{domain}.{entity}.{action}` where domain matches the service (e.g., `commerce`, `finance`, `hr`, `manufacturing`, `platform`). Also update inbox bindings in `docs/04-events/overview.md` if new cross-service subscriptions are needed.
4. **Add new endpoints** to `docs/05-api/endpoints.md`. Follow API design rules from `docs/05-api/standards.md` (RESTful, versioned `/api/v1/...`, Bearer JWT, cursor-based pagination, RFC 7807 errors, Idempotency-Key).
5. **Add new error codes** to `docs/05-api/error-codes.md`. Follow the existing error code taxonomy. Format: RFC 7807 Problem Details.
6. **Add new data models** to `docs/03-data/domain-models.md`. Include standard columns (`id`, `tenant_id`, `created_at`, `updated_at`, `created_by`, `updated_by`, `version`, `is_deleted`), standard indexes, and RLS policy.
7. **Update the service spec** at `docs/06-services/{service}.md` to reflect the new module, tables, events, and integration points.

For **cross-service features**: define integration in both service specs; add events to catalog; update inbox bindings for both services.

---

## 5. How to Add an Endpoint

Follow API standards from SPEC.md §10 and `docs/05-api/standards.md`. See SPEC.md §10.5 for response format specifications.

### Steps

1. Define the endpoint in `docs/05-api/endpoints.md` with method, path, request/response schemas, auth requirements.
2. Define error codes in `docs/05-api/error-codes.md` for all failure cases.
3. Implement the route, handler, and permission check.
4. Write integration tests covering all success and error cases.

---

## 6. How to Add an Event

Follow event architecture from SPEC.md §5 and `docs/04-events/overview.md`.

### Naming Convention

```
{domain}.{entity}.{action}
```

- **Domain**: Service name (`auth`, `identity`, `tenant`, `config`, `commerce`, `finance`, `hr`, `manufacturing`, `report`, `workflow`, `platform`, `integration`, `crm`, `project`).
- **Entity**: Business entity (e.g., `order`, `invoice`, `employee`, `work-order`).
- **Action**: `created`, `updated`, `deleted`, `submitted`, `approved`, `rejected`, `completed`, `failed`, etc.

### Steps

1. Define the event in `docs/04-events/catalog.md` with type, version, payload schema, publisher, consumers.
2. Update inbox bindings in `docs/04-events/overview.md` if a new service needs to consume this event.
3. Add the event to the outbox table in the publishing service's database.
4. Implement the consumer with idempotent processing (dedup on `event_id`).
5. If the event is part of a saga, update `docs/04-events/sagas.md` with compensating actions.

---

## 7. How to Add a Database Table

Follow data architecture from SPEC.md §9 and `docs/03-data/overview.md`. Standard columns, indexes, RLS policies, migration rules, schema change safety patterns, and soft delete conventions are all defined in SPEC.md §9.

### Steps

1. Add the table definition to `docs/03-data/domain-models.md` with all standard columns.
2. Write a forward-only, idempotent migration with timestamp prefix.
3. Add RLS policy for the table.
4. Add standard indexes plus any domain-specific indexes.
5. Ensure `tenant_id` is on every table. Event/outbox tables use `tenant_id UUID` (nullable) for system events.

---

## 8. Implementation Checklist

Verify every item before writing any code:

1. **Ownership** — Which service owns this feature? Confirm via SPEC.md §6 (Domain Ownership). If unclear, check SPEC.md §4 (Service Catalog) and the service spec.
2. **Spec read** — Read the owning service spec (`docs/06-services/{service}.md`) and any feature spec (`docs/07-features/`).
3. **Events** — Do new events follow `{domain}.{entity}.{action}` namespace? Added to `docs/04-events/catalog.md`? Inbox bindings updated in `docs/04-events/overview.md`?
4. **Database** — New tables in the owning service's DB only. Standard columns from SPEC.md §9.1. RLS policy on every table. No cross-DB access.
5. **Tenant isolation** — `tenant_id` on every table. Every query is RLS-scoped via `app.current_tenant` session variable. Redis keys prefixed with `mserp:{tenant_id}:`.
6. **Auth** — Every endpoint checks Bearer JWT. Permissions from SPEC.md §11.3 format (`{domain}.{entity}.{action}`). Legacy mappings respected (`sales.*` → `commerce.*`).
7. **API conformance** — Cursor-based pagination, `Idempotency-Key` header, RFC 7807 error format, optimistic concurrency via `version` field. Per `docs/05-api/standards.md`.
8. **No cross-DB access** — Never read/write another service's database. Use events (RabbitMQ) or REST APIs for cross-service data.
9. **Saga safety** — If distributed transaction, include self-binding (`{domain}.#`) in inbox for compensation. Define compensating action for every step. State tracked in originating service's DB. 30-minute timeout.
10. **Tests** — Unit tests (`#[cfg(test)]`, >= 80% coverage per crate). Integration tests (testcontainers, all endpoints). Contract tests (Pact, all service boundaries). Saga compensation tests (all sagas).
11. **Conventions** — Code style per `docs/09-development/conventions.md`. Dependencies per `docs/08-infrastructure/technology.md`. Workspace-managed dependency versions only.
12. **Quality gates** — `#![deny(unsafe_code)]`. `cargo tarpaulin` >= 80%. CI blocks on: clippy warnings, format errors, test failures, coverage regression. Conventional Commits required.

---

## 9. Common Violations to Avoid

1. **Querying another service's database directly.** Use events or REST APIs. Database-per-service is absolute. (SPEC.md §3.3)
2. **Publishing events without the outbox pattern.** All events MUST go through the transactional outbox. Writing business data + event must be atomic in one DB transaction. (SPEC.md §5.6)
3. **Missing `tenant_id` on a table.** Every table MUST have `tenant_id`. Event/outbox tables use nullable `tenant_id`. (SPEC.md §9.1)
4. **Missing RLS policy.** Every table MUST have a `tenant_isolation` RLS policy. Never rely on application-level filtering alone. (SPEC.md §9.3)
5. **Using the wrong event namespace.** Domains are `auth`, `identity`, `tenant`, `config`, `commerce`, `finance`, `hr`, `manufacturing`, `report`, `workflow`, `platform`, `integration`, `crm`, `project`. Legacy mappings: `sales.*` → `commerce.*`, `inventory.*` → `commerce.*`, `procurement.*` → `finance.*`. (SPEC.md §5.1, §11.3)
6. **Adding a column without a default value.** Breaks zero-downtime deployments. All new columns MUST have a default. (SPEC.md §9.5)
7. **Missing `Idempotency-Key` on state-changing requests.** Required on all `POST`, `PUT`, `PATCH`, `DELETE`. 24-hour window, scoped to tenant + user + endpoint. (SPEC.md §10.2)
8. **Creating circular service dependencies.** Workflow Service is used by all services for approvals — it MUST NOT depend on them. Core services (Auth, Identity, Tenant, Config) do NOT have inbox queues. (SPEC.md §4, §5.2)
9. **Missing saga compensation for distributed transactions.** Every saga step MUST have a compensating action. Self-binding (`{domain}.#`) is required for saga participants. 30-minute timeout. (SPEC.md §5.9)
10. **Hard-coding configuration that belongs in Config Service.** Use hierarchical configuration from Config Service. Subscribe to `config.changed` events. (SPEC.md §4.1, §5.3)
11. **Building independent analytics infrastructure.** Report Service owns all analytics. Other services publish domain events; Report Service consumes them for its data warehouse. (SPEC.md §7.3)
12. **Owning IoT device registration in Manufacturing Service.** Platform Service owns the authoritative device registry. Manufacturing's `iot_devices` table is a local cache. Manufacturing owns telemetry only. (SPEC.md §7.6)
13. **Duplicating SoD rules in Workflow Service.** Platform Service (GRC) owns SoD rule definitions. Workflow queries Platform API for rules. Workflow does NOT have its own `sod_rules` table. (SPEC.md §7.7)

---

## 10. AI Agent Workflow (Step-by-Step)

Follow this sequence when implementing a service module:

### Phase 1: Understand the Task

1. **Parse the task requirement.** Identify what needs to be built: new module, new endpoint, new event, new table, bug fix, or refactor.
2. **Identify the owning service** from SPEC.md §6 (Domain Ownership). If the task spans multiple services, break it into per-service subtasks.
3. **Read the service spec** (`docs/06-services/{service}.md`) for existing module context, tables, events, and integration points.
4. **Check for a feature spec** in `docs/07-features/`. If one exists, it contains the detailed business rules. See SPEC.md §12 for the registry.

### Phase 2: Design

5. **Define database tables** per data architecture rules (SPEC.md §9). Include all standard columns, standard indexes, RLS policy. Write forward-only, idempotent migration with timestamp prefix.
6. **Define events** per event architecture rules (SPEC.md §5). Use `{domain}.{entity}.{action}` namespace. Add to `docs/04-events/catalog.md`. Update inbox bindings if needed.
7. **Define API endpoints** per API standards (SPEC.md §10, `docs/05-api/standards.md`). Include request/response schemas, auth requirements, pagination, idempotency.
8. **Define error codes** per error taxonomy (`docs/05-api/error-codes.md`). Use RFC 7807 Problem Details format.

### Phase 3: Implement

9. **Write the migration** (forward-only, idempotent, timestamp-ordered). Include table creation, indexes, RLS policy, seed data if needed.
10. **Implement handler/service/repository layers** following `docs/09-development/project-structure.md` directory layout and `docs/09-development/conventions.md` code style.
11. **Implement event publisher** (via outbox). Write business data + event to outbox in the same DB transaction.
12. **Implement event consumer** (idempotent). Dedup on `event_id` with 24-hour TTL. Handle retries with exponential backoff. Route failures to DLQ after 5 attempts.

### Phase 4: Verify

13. **Write unit tests** (`#[cfg(test)]` in same file). Target >= 80% coverage per crate. Test all business logic, edge cases, error paths.
14. **Write integration tests** (testcontainers). Test all endpoints: success cases, validation errors, auth failures, pagination, idempotency, optimistic concurrency.
15. **Verify all checklist items** from §8 above. Go through each of the 12 items and confirm compliance before considering the task complete.
