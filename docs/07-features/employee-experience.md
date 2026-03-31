# Employee Experience Platform

Recognition, touchpoints, connections, and celebrations that foster engagement, belonging, and a positive workplace culture, managed by the HCM Service.

## Overview

The Employee Experience Platform provides a comprehensive engagement layer that enables peer-to-peer recognition with points and badges, automated milestone celebrations, structured feedback touchpoints, peer networking through interest groups, pulse surveys for continuous sentiment measurement, a social feed for organizational updates, AI-powered sentiment analysis from feedback, manager-employee 1:1 tracking, and a kudos wall for public acknowledgment. The module is designed to increase employee engagement, reduce turnover, and build a culture of appreciation and continuous improvement. It integrates with employee lifecycle events, performance management, and Config Service for program configuration.

## Data Flow Diagram

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  Peer-to-Peer    │────▶│  Recognition      │────▶│  Points & Badge     │
│  Recognition     │     │  Engine           │     │  Ledger             │
│  Submission      │     │  ┌─────────────┐ │     │  ┌───────────────┐  │
└──────────────────┘     │  │ Points      │ │     │  │ Balance       │  │
                          │  │ Calculation │ │     │  │ Tracking      │  │
┌──────────────────┐     │  └─────────────┘ │     │  └───────┬───────┘  │
│  Lifecycle       │────▶│  └─────────────┘ │     └──────────┼──────────┘
│  Events          │     └──────────────────┘                │
│  (Hire/Promo/   │                 │              ┌─────────▼──────────┐
│   Anniversary)   │                 │              │  Kudos Wall &      │
└──────────────────┘     ┌──────────▼──────┐       │  Social Feed       │
                          │  Celebration     │       │  ┌───────────────┐│
┌──────────────────┐     │  Scheduler       │       │  │ Posts, Likes, ││
│  Feedback        │────▶│  (Milestones)    │       │  │ Comments      ││
│  Touchpoints     │     └──────────────────┘       │  └───────────────┘│
│  & 1:1s         │                                   └─────────┬──────────┘
└──────────────────┘     ┌──────────────────┐                  │
                          │  Pulse Survey    │       ┌─────────▼──────────┐
┌──────────────────┐     │  Engine          │       │  Sentiment Analysis │
│  Interest Groups │────▶│  ┌─────────────┐ │       │  & Reporting        │
│  & Networking    │     │  │ Question    │ │       └────────────────────┘
└──────────────────┘     │  │ Templates   │ │
                          │  └─────────────┘ │
                          └──────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Recognition Engine** | HCM Service (Rust) | Peer-to-peer recognition submission with configurable points allocation, badge assignment, and tiered recognition levels |
| **Points & Badge Ledger** | HCM Service | Immutable points ledger with balance tracking, badge inventory, redemption rules, and periodic reset |
| **Celebration Scheduler** | HCM Service | Automated milestone detection (birthdays, work anniversaries, promotions) with configurable celebration templates and notifications |
| **Feedback Touchpoints** | HCM Service | Structured check-in scheduling, 1:1 meeting tracking, feedback capture forms, and action item management |
| **Interest Groups** | HCM Service | Group creation, membership management, discussion threads, and activity feeds for peer networking |
| **Pulse Surveys** | HCM Service | Configurable survey templates, scheduled distribution, anonymous response collection, and trend analysis |
| **Social Feed** | HCM Service | Activity stream for recognition posts, celebrations, group activities, and organizational updates with reactions and comments |
| **Sentiment Analysis** | HCM Service + Report Service | AI-powered sentiment scoring from feedback text and survey responses with trend visualization |
| **Storage** | PostgreSQL | All recognition, celebration, feedback, survey, and social data in HCM database |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Recognition Manager | Recognition submission, validation, points calculation, badge assignment | HCM Service |
| Points Ledger | Immutable points transaction log, balance queries, redemption processing | HCM Service |
| Badge Manager | Badge definition, earned badge tracking, tier progression | HCM Service |
| Celebration Engine | Milestone detection, celebration post generation, notification dispatch | HCM Service |
| Touchpoint Scheduler | Check-in and 1:1 scheduling, reminders, feedback capture | HCM Service |
| Group Manager | Interest group CRUD, membership, moderation, activity tracking | HCM Service |
| Pulse Survey Engine | Survey template CRUD, distribution, response collection, analytics | HCM Service |
| Social Feed Aggregator | Activity stream composition, reactions, comments, visibility rules | HCM Service |
| Sentiment Analyzer | Feedback text analysis, sentiment scoring, trend calculation (via Report Service ML) | HCM Service |
| Kudos Wall | Public recognition display, filtering, highlights | HCM Service |

## Recognition Tiers

| Tier | Points Range | Badge | Benefits |
|------|-------------|-------|----------|
| Bronze | 0–99 | Bronze Star | Basic recognition, wall display |
| Silver | 100–499 | Silver Shield | Priority in marketplace, quarterly reward |
| Gold | 500–999 | Gold Crown | Leadership visibility, annual reward |
| Platinum | 1000–2499 | Platinum Circle | Executive recognition, special perks |
| Diamond | 2500+ | Diamond Award | All-access, annual ceremony, mentorship priority |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Employee Profile | HCM Service | Internal API | Employee data for milestone detection and recognition eligibility |
| Performance Management | HCM Service | Internal API | Recognition data feeds performance reviews |
| Learning Experience (LXP) | HCM Service | Event-driven | Training completions trigger milestone celebrations |
| Goals & Career Development | HCM Service | Event-driven | Goal achievements trigger recognition suggestions |
| Report Service (ML) | Report Service | Event-driven | Sentiment analysis inference results |
| Notification Service | Platform Service | Event-driven | Recognition alerts, celebration posts, survey reminders, touchpoint notifications |
| Config Service | Config Service | Event-driven | Recognition rules, point values, celebration templates, survey schedules |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `hr.experience.recognition-given` | `{recognition_id, tenant_id, giver_id, receiver_id, recognition_type, points, badge_id, message, given_at}` | Peer submits recognition | Notification Service, Report Service, Social Feed |
| `hr.experience.celebration-posted` | `{celebration_id, tenant_id, employee_id, milestone_type, milestone_date, message, posted_at}` | Automated or manual celebration post created | Notification Service, Social Feed, Report Service |
| `hr.experience.touchpoint-scheduled` | `{touchpoint_id, tenant_id, manager_id, employee_id, touchpoint_type, scheduled_date, agenda_items (JSONB)}` | 1:1 or check-in scheduled | Notification Service, Calendar Integration |
| `hr.experience.feedback-submitted` | `{feedback_id, tenant_id, touchpoint_id, provider_id, recipient_id, feedback_type, sentiment_score, submitted_at}` | Feedback captured in touchpoint or survey | Sentiment Analyzer, Report Service |
| `hr.experience.survey-responded` | `{response_id, tenant_id, survey_id, respondent_id, responses (JSONB), submitted_at}` | Employee submits pulse survey response | Pulse Survey Analytics, Report Service |
| `hr.experience.group-joined` | `{membership_id, tenant_id, group_id, employee_id, joined_at}` | Employee joins an interest group | Notification Service, Group Activity Feed |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `hr.employee.hired` | HCM Service | Schedule welcome celebration, auto-join new-hire group, initialize points ledger |
| `hr.employee.updated` | HCM Service | Detect promotion, role change, or department transfer for milestone celebrations |
| `hr.review.completed` | HCM Service | Trigger post-review recognition suggestions and feedback touchpoint scheduling |
| `hr.goal.progress-updated` | HCM Service | Detect goal achievement milestones for celebration triggers |
| `hr.training.completed` | HCM Service | Trigger learning milestone celebrations |
| `config.changed` | Config Service | Reload recognition rules, point values, celebration templates, survey schedules |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `recognition_awards` | `id, tenant_id, giver_id, receiver_id, recognition_type, points, badge_id, message, visibility, awarded_at, created_at, updated_at, created_by, updated_by, version, is_deleted` | References `employee` (giver, receiver); belongs to `recognition_badges` |
| `recognition_badges` | `id, tenant_id, name, description, image_ref, tier, criteria (JSONB), points_value, is_active, created_at, updated_at, created_by, updated_by, version, is_deleted` | Has many `recognition_awards` |
| `celebration_events` | `id, tenant_id, employee_id, milestone_type, milestone_date, message, image_ref, visibility, posted_by, celebration_date, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | References `employee` |
| `feedback_touchpoints` | `id, tenant_id, manager_id, employee_id, touchpoint_type, scheduled_date, completed_date, agenda_items (JSONB), notes, action_items (JSONB), sentiment_score, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | References two `employee` records (manager, employee) |
| `employee_groups` | `id, tenant_id, name, description, group_type, category, privacy, member_count, owner_id, image_ref, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | Has many membership records |
| `pulse_surveys` | `id, tenant_id, title, description, questions (JSONB), cadence, target_audience (JSONB), start_date, end_date, response_count, is_anonymous, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | Has many `pulse_responses` |
| `pulse_responses` | `id, tenant_id, survey_id, respondent_id, responses (JSONB), submitted_at, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `pulse_surveys`; references `employee` (nullable if anonymous) |
| `social_posts` | `id, tenant_id, author_id, post_type, content, image_ref, reactions_summary (JSONB), comments_count, visibility, parent_post_id, posted_at, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | References `employee`; self-referencing for comments |

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `hr.experience.points.monthly_budget` | 500 | Monthly recognition points budget per employee |
| `hr.experience.points.carry_over_max` | 100 | Maximum points that carry over between months |
| `hr.experience.celebration.auto_post` | true | Automatically post milestone celebrations |
| `hr.experience.celebration.advance_days` | 1 | Days before milestone to create celebration |
| `hr.experience.touchpoint.default_cadence_days` | 14 | Default interval between scheduled 1:1s |
| `hr.experience.survey.default_cadence_days` | 30 | Default interval between pulse surveys |
| `hr.experience.sentiment.enabled` | true | Enable AI sentiment analysis on feedback |
| `hr.experience.recognition.visibility.default` | `team` | Default visibility for recognition posts |

## Cross-References

- [Goals & Career Development](./goals-career-development.md) — Goal achievements trigger celebrations
- [Performance Management](../06-services/hcm.md) — Recognition data informs reviews
- [HCM Service](../06-services/hcm.md) — HCM service specification
- [Report Service](../06-services/report.md) — Sentiment analysis ML inference
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
- [NFR: Performance](../10-planning/nfr.md) — Recognition submission < 500ms; social feed load < 1s
