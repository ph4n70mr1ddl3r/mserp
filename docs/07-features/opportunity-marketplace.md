# Opportunity Marketplace

Internal gig work and mobility marketplace enabling employees to discover and apply for short-term projects, stretch assignments, and mentorship opportunities with skill-based matching and manager approval workflows, managed by the HCM Service.

## Overview

The Opportunity Marketplace provides an internal talent mobility platform where employees can find and apply for project-based gigs, cross-functional assignments, and mentorship pairings. The module supports internal job postings with configurable visibility, skill-based matching algorithms that surface relevant opportunities to qualified employees, structured application and approval workflows, and career path insights derived from completed gig history. It integrates with the Dynamic Skills module for skill matching, Goals & Career Development for career path suggestions, and Workflow Service for manager approvals.

## Data Flow Diagram

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  Manager Creates │────▶│  Opportunity      │────▶│  Skill-Based        │
│  Posting / Gig   │     │  Posting Engine   │     │  Matching Engine    │
└──────────────────┘     └──────────────────┘     └──────────┬──────────┘
                                                              │
┌──────────────────┐     ┌──────────────────┐                 │
│  Employee Skill  │────▶│  Match Scorer    │◀────────────────┘
│  Profile         │     │  & Ranker        │
└──────────────────┘     └──────────────────┘     ┌──────────▼──────────┐
                                                   │  Employee           │
┌──────────────────┐     ┌──────────────────┐     │  Notifications      │
│  Employee        │────▶│  Application      │────▶│  & Suggestions      │
│  Browses / Apply │     │  Workflow         │     └──────────┬──────────┘
└──────────────────┘     └──────────────────┘                │
                                                    ┌────────▼──────────┐
┌──────────────────┐     ┌──────────────────┐     │  Manager Approval  │
│  Workflow        │◀────│  Approval Router │◀────│  & Assignment       │
│  Service         │     │                  │     └──────────┬──────────┘
└──────────────────┘     └──────────────────┘                │
                                                    ┌────────▼──────────┐
┌──────────────────┐     ┌──────────────────┐     │  Assignment        │
│  Career Path     │◀────│  Gig History      │◀────│  Tracking &        │
│  Suggestions     │     │  Analyzer         │     │  Completion        │
└──────────────────┘     └──────────────────┘     └────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Opportunity Posting Engine** | HCM Service (Rust) | CRUD for internal job postings and gig opportunities with configurable visibility, skill requirements, and duration |
| **Skill-Based Matching** | HCM Service | Matching algorithm comparing employee skill profiles against opportunity requirements with relevance scoring and ranking |
| **Application Workflow** | HCM Service + Workflow Service | Structured application submission, status tracking, manager review, and approval/rejection with configurable approval chains |
| **Assignment Tracking** | HCM Service | Active gig assignment lifecycle management with start/end dates, objectives, deliverables, and completion evaluation |
| **Mentorship Matching** | HCM Service | Mentor-mentee pairing based on skills, career goals, department diversity, and availability with preference weighting |
| **Career Path Suggestions** | HCM Service + Report Service | Analysis of completed gig history to suggest career trajectories, skill development paths, and future opportunities |
| **Notification & Discovery** | HCM Service + Platform Service | Personalized opportunity recommendations, application status updates, and assignment reminders |
| **Storage** | PostgreSQL | All opportunity, application, assignment, and mentorship data in HCM database |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Opportunity Posting Manager | CRUD for internal postings and gig definitions | HCM Service |
| Skill Matching Engine | Profile-to-requirement matching with scoring and ranking | HCM Service |
| Application Processor | Application submission, status management, and workflow triggering | HCM Service |
| Approval Router | Manager notification, approval collection, and decision recording | HCM Service |
| Assignment Tracker | Active assignment lifecycle, progress check-ins, and completion processing | HCM Service |
| Mentorship Matcher | Mentor availability, preference collection, pair scoring, and matching | HCM Service |
| Skill Profile Aggregator | Employee skill profile assembly from assessments, gigs, and training | HCM Service |
| Career Path Analyzer | Gig history analysis for career trajectory and development suggestions | HCM Service |
| Marketplace Search | Full-text and faceted search for opportunity discovery | HCM Service |

## Opportunity Types

| Type | Duration | Approval | Example |
|------|----------|----------|---------|
| Project Gig | 1–12 weeks | Manager | Cross-team feature development |
| Stretch Assignment | 2–8 weeks | Manager + skip-level | Leading a strategic initiative |
| Job Shadowing | 1–5 days | Self-enroll | Observing a role of interest |
| Mentorship | 3–12 months | Mentor + manager | Senior-junior skill transfer |
| Cross-Functional Rotation | 1–6 months | Both managers | Department immersion |
| Innovation Challenge | 1–4 weeks | Self-enroll | Hackathon or idea sprint |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Dynamic Skills | HCM Service | Internal API | Skill profiles and gap data for matching |
| Goals & Career Development | HCM Service | Internal API | Career aspirations and development plans |
| Performance Management | HCM Service | Internal API | Gig performance feeds review cycle |
| Employee Profile | HCM Service | Internal API | Employee data, reporting structure, availability |
| Workflow Engine | Workflow Service | Event-driven | Application approval workflows |
| Notification Service | Platform Service | Event-driven | Opportunity alerts, status updates, reminders |
| Report Service | Report Service | Internal API | Marketplace utilization and mobility analytics |
| Config Service | Config Service | Event-driven | Posting templates, matching algorithm weights |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `hr.opportunity.posting-created` | `{posting_id, tenant_id, title, opportunity_type, required_skills, posting_owner_id, visibility, open_date, close_date}` | New opportunity posting published | Notification Service, Report Service |
| `hr.opportunity.application-submitted` | `{application_id, posting_id, applicant_id, submitted_at, match_score}` | Employee submits application | Workflow Service, Notification Service |
| `hr.opportunity.assignment-started` | `{assignment_id, posting_id, employee_id, manager_id, start_date, end_date, objectives}` | Application approved and assignment begins | Report Service, Skill Profile Aggregator |
| `hr.opportunity.assignment-completed` | `{assignment_id, posting_id, employee_id, completed_at, skills_gained, outcome_rating}` | Assignment end date reached or manually completed | Career Path Analyzer, Report Service, Dynamic Skills |
| `hr.opportunity.match.suggested` | `{suggestion_id, posting_id, employee_id, match_score, match_factors}` | Matching engine identifies high-fit candidate | Notification Service, Employee Dashboard |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `hr.employee.hired` | HCM Service | Initialize skill profile and surface onboarding-period opportunities |
| `hr.employee.updated` | HCM Service | Update employee availability, department, and reporting structure for matching |
| `hr.goal.progress-updated` | HCM Service | Refine opportunity suggestions based on evolving career goals |
| `hr.competency.assessed` | HCM Service | Update skill profile and re-score pending opportunity matches |
| `workflow.step.approved` | Workflow Service | Finalize application approval and create assignment |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `opportunity_postings` | `id, tenant_id, title, description, opportunity_type, owning_department_id, required_skills (JSONB), visibility, status, posting_owner_id, open_date, close_date, created_at, updated_at, created_by, updated_by, version, is_deleted` | Has many `gig_applications`, `gig_assignments` |
| `gig_applications` | `id, tenant_id, posting_id, applicant_id, status, cover_note, match_score, submitted_at, reviewed_by, reviewed_at, decision_reason, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `opportunity_postings`; references `employee` |
| `gig_assignments` | `id, tenant_id, posting_id, application_id, employee_id, manager_id, start_date, end_date, objectives (JSONB), status, completed_at, outcome_rating, skills_gained (JSONB), created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `opportunity_postings`, `gig_applications`; references `employee` |
| `skill_profiles` | `id, tenant_id, employee_id, skills (JSONB), proficiency_summary (JSONB), last_assessed_at, source_count, created_at, updated_at, created_by, updated_by, version, is_deleted` | References `employee`; consumed by matching engine |
| `mentorship_pairs` | `id, tenant_id, mentor_id, mentee_id, status, goals (JSONB), start_date, end_date, meeting_cadence_days, completed_at, outcome_notes, created_at, updated_at, created_by, updated_by, version, is_deleted` | References two `employee` records (mentor, mentee) |

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `hr.opportunity.max_concurrent_gigs` | 2 | Maximum active gig assignments per employee |
| `hr.opportunity.match_score_threshold` | 0.6 | Minimum match score to surface opportunity suggestion |
| `hr.opportunity.auto_close_days` | 30 | Days after close date to automatically close posting |
| `hr.opportunity.mentorship_max_mentees` | 3 | Maximum active mentees per mentor |
| `hr.opportunity.application_window_days` | 14 | Default open-to-close window for postings |
| `hr.opportunity.visibility.default` | `tenant_wide` | Default visibility scope for new postings |

## Cross-References

- [Dynamic Skills](./dynamic-skills.md) — Skill profiles feed matching engine
- [Goals & Career Development](./goals-career-development.md) — Career aspirations inform suggestions
- [HCM Service](../06-services/hcm.md) — HCM service specification
- [Workflow Engine](../06-services/workflow.md) — Approval workflows for applications
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
- [NFR: Performance](../10-planning/nfr.md) — Skill matching query < 500ms across 10k profiles
