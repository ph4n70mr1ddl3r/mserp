# Structured Onboarding Journey Builder

Configurable onboarding journey templates with task management, milestone tracking, buddy assignment, and equipment provisioning workflows, managed by the HCM Service.

## Overview

The Structured Onboarding Journey Builder enables HR teams to design, schedule, and monitor personalized onboarding experiences for new hires. The module provides a template-driven approach where journeys are composed of sequenced tasks grouped by milestones, with support for conditional branching based on role, department, or location. It integrates with Identity Service for automated account provisioning, Platform Service for notifications, and equipment management workflows to ensure new employees are productive from day one.

## Data Flow Diagram

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  Offer Accepted  │────▶│  Journey Builder  │────▶│  Journey Instance   │
│  (HR Event)      │     │  ┌─────────────┐ │     │  ┌───────────────┐  │
└──────────────────┘     │  │ Template     │ │     │  │ Task Queue    │  │
                         │  │ Selection    │ │     │  │ (personalized)│  │
┌──────────────────┐     │  └──────┬──────┘ │     │  └───────┬───────┘  │
│  Employee        │────▶│         │        │     └──────────┼──────────┘
│  Profile Data    │     │  ┌──────▼──────┐ │                │
└──────────────────┘     │  │ Conditional │ │     ┌──────────▼──────────┐
                         │  │ Branching   │ │     │  Task Execution     │
                         │  └─────────────┘ │     │  ┌───────────────┐  │
                         └──────────────────┘     │  │ Assignees     │  │
                                                   │  │ (self/manager │  │
┌──────────────────┐     ┌──────────────────┐     │  │  /buddy/IT)   │  │
│  Identity        │◀────│  Provisioning    │◀────│  └───────┬───────┘  │
│  Service         │     │  Workflow        │     └──────────┼──────────┘
└──────────────────┘     └──────────────────┘                │
                                                   ┌─────────▼──────────┐
                                                   │  Milestone Tracker  │
                                                   │  (progress & alerts)│
                                                   └────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Journey Template Engine** | HCM Service (Rust) | Visual template builder with drag-and-drop task sequencing, conditional branches, and milestone grouping |
| **Task Management** | HCM Service | Assignable tasks with due dates, dependencies, completion evidence, and automated reminders |
| **Milestone Tracking** | HCM Service | Progress gates that aggregate task completion and trigger next journey phase |
| **Buddy Assignment** | HCM Service + Workflow Service | Automated buddy matching based on department, team, skills, and availability with opt-out support |
| **Equipment Provisioning** | HCM Service + Workflow Service | IT equipment and access request workflows triggered by journey tasks with approval routing |
| **Document Collection** | HCM Service + MinIO | Secure document upload, verification workflows, and compliance checklist tracking |
| **Journey Analytics** | HCM Service + Report Service | Time-to-completion metrics, bottleneck detection, and new hire satisfaction scoring |
| **Storage** | PostgreSQL + MinIO | Journey and task data in PostgreSQL; uploaded documents in MinIO |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Journey Template Manager | CRUD for journey templates with task and milestone definitions | HCM Service |
| Journey Instance Engine | Template instantiation, personalization, and lifecycle management | HCM Service |
| Task Executor | Task assignment, tracking, reminder scheduling, and completion validation | HCM Service |
| Milestone Tracker | Progress aggregation, gate evaluation, and phase transition logic | HCM Service |
| Buddy Matcher | Automated buddy assignment with configurable matching criteria | HCM Service |
| Provisioning Coordinator | Identity and equipment provisioning workflow orchestration | HCM Service |
| Document Collector | Secure document upload, storage, and verification status tracking | HCM Service |
| Journey Analytics | Completion metrics, satisfaction surveys, and trend analysis | HCM Service |

## Onboarding Task Types

| Task Type | Assignee | Auto-Completable | Example |
|-----------|----------|-----------------|---------|
| Self-Service | New hire | Yes (on action) | Complete personal profile, upload ID |
| Manager | Hiring manager | No | Welcome meeting, set expectations |
| Buddy | Assigned buddy | No | Office tour, introductions |
| IT Provisioning | IT team / automated | Yes (on system event) | Laptop setup, account creation |
| Document Collection | New hire + HR | Yes (on upload + verify) | Tax forms, NDA, benefits enrollment |
| Training | New hire | Yes (on LMS completion) | Compliance training, safety orientation |
| Social | New hire + team | No | Team lunch, cross-functional introductions |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Employee Lifecycle | HCM Service | Internal API | New hire data, employment status changes |
| Identity Service | Identity Service | Event-driven | Account provisioning, role assignment |
| Notification Service | Platform Service | Event-driven | Task reminders, milestone alerts, welcome emails |
| Workflow Engine | Workflow Service | Event-driven | Approval workflows for equipment and access |
| Learning (LXP) | HCM Service | Internal API | Training task completion tracking |
| Document Management | HCM Service + MinIO | Internal API | Secure document storage and retrieval |
| Report Service | Report Service | Internal API | Onboarding effectiveness dashboards |
| Config Service | Config Service | Event-driven | Journey template configurations |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `hr.onboarding.journey-started` | `{journey_id, employee_id, template_id, start_date}` | Journey instance created for new hire | Notification Service, Report Service |
| `hr.onboarding.task-completed` | `{journey_id, task_id, task_type, completed_by, completed_at}` | Individual task marked complete | Milestone Tracker, Notification Service |
| `hr.onboarding.milestone-reached` | `{journey_id, milestone_id, milestone_name, reached_at}` | All milestone tasks completed | Notification Service, Journey Analytics |
| `hr.onboarding.journey-completed` | `{journey_id, employee_id, completed_at, duration_days}` | Final milestone reached | Report Service, Employee Lifecycle |
| `hr.onboarding.task-overdue` | `{journey_id, task_id, assignee_id, due_date}` | Task passes due date without completion | Notification Service, Manager Dashboard |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `hr.employee.hired` | HCM Service | Create journey instance from template matching role/department |
| `hr.employee.onboarding-started` | HCM Service | Activate journey and assign first milestone tasks |
| `identity.user.provisioned` | Identity Service | Auto-complete account provisioning task |
| `workflow.step.approved` | Workflow Service | Complete equipment or access approval task |
| `hr.learning.course-completed` | HCM Service | Mark training task as complete |
| `config.changed` | Config Service | Reload journey template settings |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `onboarding_journeys` | `id, tenant_id, employee_id, template_id, status, start_date, target_end_date, completed_at` | Has many `onboarding_tasks`, `onboarding_milestones` |
| `onboarding_tasks` | `id, tenant_id, journey_id, milestone_id, title, task_type, assignee_id, due_date, status, completed_at` | Belongs to `onboarding_journeys`, `onboarding_milestones` |
| `onboarding_milestones` | `id, tenant_id, journey_id, name, sequence, target_days, status, reached_at` | Has many `onboarding_tasks` |
| `equipment_requests` | `id, tenant_id, journey_id, task_id, item_type, specifications, status, approved_by, fulfilled_at` | Belongs to `onboarding_journeys`, `onboarding_tasks` |

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `hr.onboarding.default_template` | null | Default journey template ID for new hires |
| `hr.onboarding.reminder_frequency_days` | 2 | Days between task reminder notifications |
| `hr.onboarding.overdue_alert_hours` | 24 | Hours after due date before escalation alert |
| `hr.onboarding.buddy_required` | true | Whether buddy assignment is mandatory |
| `hr.onboarding.auto_start_days_before` | 7 | Days before start date to pre-create journey |
| `hr.onboarding.document_retention_days` | 365 | Days to retain uploaded onboarding documents |

## Cross-References

- [Employee Lifecycle](../06-services/hcm.md) — Hire event triggers journey creation
- [Identity Service](../06-services/identity.md) — Account provisioning integration
- [Workflow Engine](../06-services/workflow.md) — Equipment and access approval workflows
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
- [HCM Service](../06-services/hcm.md) — HCM service specification
- [NFR: Performance](../10-planning/nfr.md) — Journey instantiation < 2s per employee
