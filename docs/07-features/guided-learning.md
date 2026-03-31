# Guided Learning

In-app contextual guidance and interactive walkthroughs with step-by-step tours, role-based guidance paths, feature announcements, and A/B testing for effectiveness, managed by the Platform Service.

## Overview

Guided Learning provides an embedded in-app guidance system that delivers contextual help, interactive walkthroughs, and feature announcements to users based on their role, behavior, and application context. The module supports step-by-step guided walkthroughs for any application page, context-sensitive help triggered by user behavior patterns, tooltip and beacon placement for UI element highlighting, walkthrough analytics for measuring completion rates and drop-off points, role-based guidance paths, feature announcement banners, video tutorial embedding, multi-language walkthrough support, A/B testing for guidance effectiveness, user feedback collection, a drag-and-drop guided tour builder, walkthrough versioning, new feature spotlight tours, and search-triggered guidance. The module enables non-technical teams to create and manage guidance content without developer intervention.

## Data Flow Diagram

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  Tour Builder    │────▶│  Walkthrough     │────▶│  Versioned          │
│  (Drag & Drop)   │     │  Engine          │     │  Walkthrough Store  │
└──────────────────┘     └──────────────────┘     └──────────┬──────────┘
                                                              │
┌──────────────────┐     ┌──────────────────┐     ┌──────────▼──────────┐
│  User Context    │────▶│  Targeting       │────▶│  Walkthrough        │
│  (Role, Page,    │     │  & Personalize   │     │  Delivery           │
│   Behavior)      │     │                  │     │  ┌───────────────┐  │
└──────────────────┘     └──────────────────┘     │  │ Steps         │  │
                                                   │  │ Tooltips      │  │
┌──────────────────┐     ┌──────────────────┐     │  │ Beacons       │  │
│  Feature         │────▶│  Announcement    │────▶│  │ Videos        │  │
│  Announcements   │     │  Manager         │     │  └───────┬───────┘  │
└──────────────────┘     └──────────────────┘     └──────────┼──────────┘
                                                              │
┌──────────────────┐     ┌──────────────────┐     ┌──────────▼──────────┐
│  A/B Test Config │────▶│  Variant         │────▶│  Analytics Engine   │
│                  │     │  Assignment      │     │  ┌───────────────┐  │
└──────────────────┘     └──────────────────┘     │  │ Completion    │  │
                                                   │  │ Drop-off      │  │
┌──────────────────┐                              │  │ Feedback      │  │
│  User Feedback   │─────────────────────────────▶│  │ A/B Results   │  │
└──────────────────┘                              │  └───────────────┘  │
                                                   └─────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Walkthrough Engine** | Platform Service (Rust) | Step-by-step walkthrough orchestration with progress tracking, conditional branching, and resume capability |
| **Tour Builder** | Platform Service | Drag-and-drop visual editor for creating and editing walkthroughs with element selectors and action definitions |
| **Targeting Engine** | Platform Service | Role-based, page-based, and behavior-based targeting with rule composition and priority ordering |
| **Beacon Placement** | Platform Service | UI element highlighting with configurable beacon styles (pulse, dot, badge), tooltip positioning, and dismiss rules |
| **Announcement Manager** | Platform Service | Feature announcement banner lifecycle management with scheduling, audience targeting, and dismissal tracking |
| **Analytics Engine** | Platform Service | Walkthrough completion tracking, drop-off analysis, time-per-step measurement, and aggregated effectiveness metrics |
| **A/B Testing** | Platform Service | Variant assignment for walkthrough content and delivery timing with statistical significance tracking |
| **Multi-Language Support** | Platform Service | Localized walkthrough content with language fallback chains and RTL layout support |
| **Versioning** | Platform Service | Walkthrough version history with publish/draft lifecycle, rollback, and concurrent version management |
| **Storage** | PostgreSQL + Redis | Walkthrough definitions and analytics in PostgreSQL; active session state and targeting cache in Redis |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Walkthrough Manager | Walkthrough CRUD, step definition, conditional branching, lifecycle management | Platform Service |
| Tour Builder API | Drag-and-drop builder backend, element selector resolution, preview generation | Platform Service |
| Targeting Engine | Rule evaluation, role matching, page matching, behavior-based triggering | Platform Service |
| Beacon Engine | UI element identification, placement computation, style configuration, dismiss tracking | Platform Service |
| Announcement Manager | Banner CRUD, scheduling, audience targeting, dismissal tracking | Platform Service |
| Guidance Path Manager | Role-based path definition, sequence ordering, prerequisite handling | Platform Service |
| Step Executor | Runtime step delivery, progress tracking, action validation, resume handling | Platform Service |
| Analytics Collector | Event capture, completion tracking, drop-off detection, time measurement | Platform Service |
| Feedback Collector | User ratings, free-text feedback, sentiment capture, aggregation | Platform Service |
| A/B Test Manager | Variant definition, assignment, exposure tracking, significance calculation | Platform Service |
| Localization Handler | Translation management, language fallback, content rendering per locale | Platform Service |
| Version Manager | Version history, publish/draft lifecycle, rollback, diff computation | Platform Service |

## Walkthrough Step Types

| Step Type | Behavior | Configuration |
|-----------|----------|---------------|
| Tooltip | Highlight element, show instruction popover | Target selector, position, content, dismissible |
| Modal | Full-screen or centered overlay instruction | Content, media, action buttons, size |
| Beacon | Persistent indicator on UI element | Target selector, style (pulse/dot/badge), color |
| Redirect | Navigate user to a different page | Target URL, delay, trigger |
| Form Interaction | Guide user through form field completion | Field selector, validation, sample value |
| Video | Embedded video tutorial playback | Video URL, autoplay, controls, transcript |
| Decision | Branching step based on user choice | Options, branch targets per option |
| Wait | Pause until condition met | Condition (element visible, page loaded, time elapsed) |
| Completion | End walkthrough with summary | Summary content, CTA, feedback prompt |

## Targeting Rules

| Dimension | Operators | Example |
|-----------|-----------|---------|
| User Role | equals, in, not_in | Role equals `sales_manager` |
| Page URL | matches, starts_with, contains | URL starts_with `/commerce/orders` |
| User Tenure | less_than, greater_than, between | Tenure less_than 7 days |
| Feature Flag | is_enabled, is_disabled | Feature `advanced_analytics` is_enabled |
| Previous Completion | completed, not_completed | Not completed walkthrough `order-basics` |
| Visit Count | equals, greater_than, less_than | Page visit count greater_than 3 |
| Device Type | equals, in | Device in `desktop, tablet` |
| Language | equals, in, not_in | Language equals `en-US` |
| Custom Attribute | equals, contains, in | Department contains `warehouse` |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Identity & User Management | Identity Service | Event-driven | New user creation triggers onboarding walkthroughs |
| Tenant Management | Tenant Service | Event-driven | New tenant creation triggers setup guidance; feature changes enable new tours |
| Page Composer | Platform Service | Event-driven | Page publishing events trigger beacon and walkthrough re-evaluation |
| Config Service | Config Service | Event-driven | Guidance targeting rules, walkthrough defaults, feature flags |
| Report Service | Report Service | Internal API | Walkthrough effectiveness dashboards, user engagement analytics |
| Notification Service | Platform Service | Event-driven | Announcement delivery, walkthrough assignment notifications |
| Search | Platform Service | Internal API | Search queries trigger contextual guidance suggestions |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `platform.guided-learning.walkthrough.started` | `{walkthrough_id, user_id, tenant_id, version, variant_id, triggered_by, started_at}` | User begins a walkthrough | Report Service |
| `platform.guided-learning.walkthrough.completed` | `{walkthrough_id, user_id, tenant_id, version, variant_id, step_count, completed_step_count, duration_seconds, completed_at}` | User completes all walkthrough steps | Report Service |
| `platform.guided-learning.walkthrough.skipped` | `{walkthrough_id, user_id, tenant_id, version, skipped_at_step, skip_reason, duration_seconds}` | User dismisses or abandons walkthrough | Report Service |
| `platform.guided-learning.announcement.published` | `{announcement_id, tenant_id, title, audience_roles, scheduled_at, published_at}` | Feature announcement goes live | Notification Service, Report Service |
| `platform.guided-learning.feedback.submitted` | `{walkthrough_id, user_id, tenant_id, rating, comment, step_number, submitted_at}` | User submits feedback after walkthrough | Report Service |
| `platform.guided-learning.beacon.triggered` | `{beacon_id, user_id, tenant_id, page_url, element_selector, triggered_at}` | User views or interacts with a beacon placement | Report Service |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `identity.user.created` | Identity Service | Evaluate onboarding guidance paths for new user; assign role-based walkthroughs |
| `tenant.created` | Tenant Service | Seed default walkthroughs and guidance paths for new tenant |
| `tenant.feature.changed` | Tenant Service | Enable/disable walkthroughs associated with feature flags; trigger spotlight tours |
| `platform.composer.page.published` | Platform Service | Re-evaluate beacons and page-targeted walkthroughs for updated pages |
| `config.changed` | Config Service | Reload targeting rules, default guidance settings, A/B test parameters |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `guided_walkthroughs` | `id, tenant_id, name, description, status, trigger_type, target_rules, priority, locale, is_active, published_version` | Has many `walkthrough_steps`, `walkthrough_versions`, `walkthrough_analytics` |
| `walkthrough_steps` | `id, tenant_id, walkthrough_id, sequence, step_type, title, content, target_selector, position, media_refs, action_config, is_skippable` | Belongs to `guided_walkthroughs` |
| `walkthrough_analytics` | `id, tenant_id, walkthrough_id, user_id, version, variant_id, started_at, completed_at, completed_step_count, total_step_count, duration_seconds, status, skipped_at_step` | Belongs to `guided_walkthroughs` |
| `feature_announcements` | `id, tenant_id, title, content, banner_type, audience_roles, start_date, end_date, is_dismissible, cta_text, cta_url, media_ref, status, published_at` | Standalone; scoped to tenant |
| `guidance_paths` | `id, tenant_id, name, description, role_ids, locale, walkthrough_sequence, is_active, sort_order` | Has many `guided_walkthroughs` (ordered association) |
| `walkthrough_versions` | `id, tenant_id, walkthrough_id, version_number, steps_snapshot, published_at, published_by, change_summary, status` | Belongs to `guided_walkthroughs` |
| `walkthrough_feedback` | `id, tenant_id, walkthrough_id, user_id, rating, comment, step_number, feedback_type, submitted_at` | Belongs to `guided_walkthroughs` |
| `beacon_placements` | `id, tenant_id, page_url_pattern, element_selector, beacon_style, beacon_color, tooltip_content, trigger_rules, dismiss_behavior, is_active, priority` | Standalone; scoped to tenant |

All tables include standard columns: `id`, `tenant_id`, `created_at`, `updated_at`, `created_by`, `updated_by`, `version`, `is_deleted`.

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `platform.guided-learning.max_concurrent_walkthroughs` | 1 | Maximum walkthroughs shown simultaneously per user |
| `platform.guided-learning.walkthrough.resume_timeout_hours` | 72 | Hours before in-progress walkthrough resets |
| `platform.guided-learning.beacon.auto_dismiss_after_views` | 5 | Views before beacon auto-dismisses for a user |
| `platform.guided-learning.announcement.max_active_per_tenant` | 3 | Maximum simultaneous active announcements per tenant |
| `platform.guided-learning.announcement.default_dismissal_ttl_hours` | 168 | Hours before dismissed announcement can reappear |
| `platform.guided-learning.analytics.aggregation_interval_minutes` | 15 | Minutes between analytics aggregation runs |
| `platform.guided-learning.ab.min_sample_size` | 100 | Minimum users per variant before statistical evaluation |
| `platform.guided-learning.ab.confidence_level` | 0.95 | Statistical confidence level for A/B test evaluation |
| `platform.guided-learning.feedback.max_comment_length` | 1000 | Maximum characters for feedback comments |
| `platform.guided-learning.localization.fallback_locale` | `en-US` | Default locale when user locale is unavailable |

## Cross-References

- [Application Composer](application-composer.md) — Page publishing triggers beacon and walkthrough updates
- [Multi-Tenancy](multi-tenancy.md) — Tenant-scoped walkthrough isolation and defaults
- [Onboarding Journey](onboarding-journey.md) — New user onboarding walkthrough orchestration
- [Search](search.md) — Search-triggered contextual guidance
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
- [Platform Service](../06-services/platform.md) — Platform service specification
- [NFR: Performance](../10-planning/nfr.md) — Walkthrough step delivery < 100ms; beacon rendering < 50ms
