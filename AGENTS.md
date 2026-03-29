# AGENTS.md — AI Agent Development Guide

This document instructs AI agents on how to use this specification to build MSERP.
Read this file FIRST before any implementation work.

## Specification Hierarchy

Agents MUST follow documents in this order of authority:

1. **SPEC.md** — Authoritative master specification. Conflicts resolved in SPEC.md's favor.
2. **AGENTS.md** — This file. Agent workflow and reading order only.
3. **Service specs** (`docs/06-services/*.md`) — Per-service modules, tables, events, integrations.
4. **Feature specs** (`docs/07-features/*.md`) — Detailed business rules for 22 features. See SPEC.md §7 for the registry.
5. **Cross-cutting specs** — Architecture, security, data, events, API standards (`docs/02-*` through `docs/05-*`).
6. **Infrastructure/dev/planning** — Operational concerns (`docs/08-*` through `docs/10-*`).

## Where to Find Authoritative Rules

All architectural rules live in SPEC.md and domain-specific docs. AGENTS.md does not duplicate them.

| Rule | Source |
|------|--------|
| Service ownership & boundary rules | SPEC.md §4 |
| Event namespace conventions | SPEC.md §3.3, `docs/04-events/overview.md` |
| IoT boundary (device registry vs. telemetry) | SPEC.md §3.4 |
| ML/AI ownership split | SPEC.md §10 |
| Critical integration points (Auth↔Identity, MDM, SoD, IoT) | SPEC.md §3.4 |
| Common violations to avoid | SPEC.md §3.3–§3.4 (6 corrected patterns) |
| Database-per-service | SPEC.md §3.3, §3 |
| Multi-tenancy & RLS | SPEC.md §8, `docs/03-data/overview.md` |
| Error handling (RFC 7807) | `docs/05-api/error-codes.md` |
| Testing requirements | `docs/10-planning/nfr.md` |
| Event-driven integration pattern & saga self-binding | `docs/04-events/overview.md` |
| Feature spec registry (22 specs) | SPEC.md §12 |

## How to Build a Service

1. `docs/06-services/{service}.md` — modules, tables, events, integration points.
2. `docs/06-services/overview.md` — service catalog, port assignments.
3. `docs/07-features/` — check for relevant feature specs; otherwise service spec is complete.
4. `docs/05-api/standards.md` — API design rules.
5. `docs/05-api/endpoints.md` — endpoint listing.
6. `docs/03-data/overview.md` — database patterns, standard columns.
7. `docs/04-events/overview.md` — event patterns, inbox bindings.
8. `docs/04-events/catalog.md` — published and consumed events.
9. `docs/02-security/overview.md` + `docs/02-security/authorization.md` — auth requirements.
10. `docs/09-development/project-structure.md` — directory layout.
11. `docs/09-development/conventions.md` — code style.
12. `docs/08-infrastructure/technology.md` — crate versions and dependencies.

## How to Add a Feature

1. Identify the owning service (SPEC.md §4).
2. Check `docs/07-features/` for an existing feature spec.
3. Add new events → `docs/04-events/catalog.md` (namespace: `{service-domain}.{entity}.{action}`).
4. Add new endpoints → `docs/05-api/endpoints.md`.
5. Add new error codes → `docs/05-api/error-codes.md`.
6. Add new data models → `docs/03-data/domain-models.md`.
7. Cross-service features: define integration in both service specs; add events to catalog.

## Agent Implementation Checklist

Before writing any code, verify every item:

1. **Ownership** — Which service owns this feature? Confirm via SPEC.md §4.
2. **Spec read** — Read the owning service spec (`docs/06-services/{service}.md`) and any feature spec (`docs/07-features/`).
3. **Events** — Do new events follow `{service-domain}.{entity}.{action}`? Added to catalog? Inbox bindings updated?
4. **Database** — New tables in the owning service's DB only. Standard columns from `docs/03-data/overview.md`.
5. **Tenant isolation** — Every table has `tenant_id`. Every query is RLS-scoped.
6. **Auth** — Every endpoint checks auth. Permissions from `docs/02-security/authorization.md`.
7. **API conformance** — Pagination, idempotency, error format per `docs/05-api/standards.md`.
8. **No cross-DB access** — Never read/write another service's database. Use events or REST APIs.
9. **Saga safety** — If distributed transaction, include self-binding (`{domain}.#`) and compensation logic.
10. **Tests** — Unit tests for logic, integration tests for endpoints, contract tests for service boundaries.
11. **Conventions** — Code style per `docs/09-development/conventions.md`. Dependencies per `docs/08-infrastructure/technology.md`.
