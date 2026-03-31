# Marketing Automation

Multi-step nurture campaigns, behavioral lead scoring, email marketing automation, landing page builder, campaign attribution, A/B testing, marketing calendar, and segment-based targeting, managed by the CRM Service.

## Overview

Marketing Automation provides the orchestration layer for multi-channel marketing campaigns, lead nurturing workflows, and behavioral intelligence across the customer lifecycle. The module enables marketing teams to design multi-step nurture sequences triggered by customer behaviors, demographic attributes, or lifecycle events. Behavioral lead scoring combines explicit signals (job title, company size, industry) with implicit signals (email engagement, website visits, content downloads, event attendance) to produce a composite score that prioritizes leads for sales outreach.

Email marketing automation manages template design, audience segmentation, send scheduling, and post-send analytics (open, click, bounce, unsubscribe rates). The landing page builder provides a drag-and-drop editor for creating campaign-specific pages with form capture, A/B variant testing, and conversion tracking. Campaign attribution models (first-touch, last-touch, linear, time-decay, U-shaped) connect marketing touchpoints to revenue outcomes. A/B testing supports controlled experiments across email subject lines, content variants, send times, and landing page designs with statistical significance calculation.

## Data Flow Diagram

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  CDP Profiles    │────▶│  Segment Engine  │────▶│  Nurture Campaign   │
│  (Demographics)  │     │  ┌─────────────┐ │     │  Engine             │
└──────────────────┘     │  │ Dynamic     │ │     │  ┌───────────────┐  │
                         │  │ Filters     │ │     │  │ State Machine │  │
┌──────────────────┐     │  │ + Refresh   │ │     │  │ Step Router   │  │
│  Behavioral      │────▶│  └─────────────┘ │     │  └───────┬───────┘  │
│  Events          │     └──────────────────┘              │          │
│  (Email, Web)    │                                       └──────────┼──┘
└──────────────────┘     ┌──────────────────┐                         │
                         │  Lead Scoring    │◀────────────────────────┘
┌──────────────────┐     │  ┌─────────────┐ │     ┌──────────────────────┐
│  Landing Pages   │────▶│  │ Explicit +  │ │     │  Email Marketing     │
│  (Conversions)   │     │  │ Implicit    │ │     │  ┌───────────────┐   │
└──────────────────┘     │  │ + Decay     │ │     │  │ Template      │   │
                         │  └─────────────┘ │     │  │ Render + Send │   │
┌──────────────────┐     └──────────────────┘     │  └───────┬───────┘   │
│  A/B Test        │────▶│  Attribution      │     └──────────┼──────────┘
│  Results         │     │  Engine           │                │
└──────────────────┘     │  ┌─────────────┐ │     ┌──────────▼──────────┐
                         │  │ Multi-Touch │ │     │  Campaign Calendar  │
                         │  │ Models      │ │     │  + Analytics        │
                         │  └─────────────┘ │     └─────────────────────┘
                         └──────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Nurture Campaign Engine** | CRM Service (Rust) | State machine-driven workflow execution. Each campaign is a directed graph of steps (email, wait, condition branch, action). Contact state tracked per campaign with configurable entry and exit criteria |
| **Behavioral Lead Scoring** | CRM Service (Rust) | Configurable scoring model with explicit (demographic) and implicit (behavioral) score components. Score decay rules reduce inactive lead scores over time. Threshold-based tier assignments (cold, warm, hot, qualified) |
| **Email Marketing** | CRM Service + Platform Service | Template management with personalization tokens, audience segmentation, send throttling, and bounce/complaint processing. Integrates with Platform Service email delivery for send execution |
| **Landing Page Builder** | CRM Service (Rust) | Drag-and-drop page editor with responsive templates, form builder, conversion tracking pixels, and SEO metadata. Pages served via CDN for performance |
| **Campaign Attribution** | CRM Service (Rust) | Multi-touch attribution models applied to closed-won opportunities. Touchpoint tracking across email, web, event, and ad channels. Configurable model selection per reporting period |
| **A/B Testing** | CRM Service (Rust) | Controlled experiment framework with variant assignment, result tracking, and statistical significance calculation (Bayesian). Supports email subject lines, body content, CTAs, send times, and landing page elements |
| **Marketing Calendar** | CRM Service (Rust) | Unified calendar view of all scheduled campaigns, email sends, events, and content publications. Conflict detection for overlapping audience segments |
| **Segment Engine** | CRM Service (Rust) | Audience segmentation using CDP profile data combined with behavioral filters. Supports static and dynamic segments with scheduled refresh. Segment size estimation before campaign launch |
| **Storage** | PostgreSQL + Redis | Campaign definitions, workflow state, and scoring models in PostgreSQL; campaign execution state and segment membership cache in Redis |

## Business Rules

| ID | Rule | Description |
|----|------|-------------|
| BR-MA-001 | Campaign Entry Uniqueness | A contact may be active in at most one instance of a given nurture campaign. Re-entry is allowed only after the contact has exited (completed, removed, or campaign ended) and a configurable cooldown period has elapsed (default 7 days) |
| BR-MA-002 | Lead Score Thresholds | Lead scores are categorized into tiers: Cold (0-30), Warm (31-60), Hot (61-80), Sales-Ready (81-100). Tier transitions trigger configurable actions: Warm → Hot triggers sales notification; Hot → Sales-Ready triggers CRM opportunity creation |
| BR-MA-003 | Email Compliance | All marketing emails must include a one-click unsubscribe mechanism. Unsubscribe requests are processed within 60 seconds and suppress all future sends to that contact across all campaigns. Suppression lists are checked before every send |
| BR-MA-004 | Send Throttling | Email sends are throttled to respect Platform Service delivery rate limits. Default: 10,000 emails per hour per tenant. Campaigns exceeding the rate are queued and delivered progressively without manual intervention |
| BR-MA-005 | A/B Test Significance | A/B test results are not considered conclusive until the sample reaches minimum statistical significance (default: 95% confidence, minimum 100 conversions per variant). Results reported before significance are labeled as preliminary |
| BR-MA-006 | Attribution Window | Touchpoints are considered for attribution only if they occurred within the configured lookback window (default 90 days before opportunity close). Touchpoints outside the window are excluded from all attribution models |
| BR-MA-007 | Segment Refresh Frequency | Dynamic segments are refreshed at minimum every 24 hours or immediately upon CDP profile update if the profile change affects segment membership criteria. Stale segments (>48 hours since refresh) display a warning before campaign launch |
| BR-MA-008 | Campaign Approval Gate | Campaigns targeting more than 10,000 contacts require manager approval before launch. The approval workflow captures the campaign content, target segment, estimated send volume, and scheduled dates |
| BR-MA-009 | Score Decay Rules | Implicit behavioral scores decay by 10% per week of inactivity. A lead with no engagement activity for 30 consecutive days loses all implicit score contribution. Explicit scores are not subject to decay |
| BR-MA-010 | Duplicate Touchpoint Prevention | A single contact interaction (e.g., one email open) generates exactly one touchpoint event. Multiple rapid interactions within a 5-second window are deduplicated to prevent score inflation |

## Data Model

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `ma_campaign_workflows` | `id, tenant_id, name, type, status, entry_criteria, steps, exit_criteria, start_at, end_at` | Has many `ma_workflow_enrollments` |
| `ma_nurture_sequences` | `id, tenant_id, campaign_id, sequence_order, step_type, channel, content_ref, wait_duration, conditions, is_active` | Belongs to `ma_campaign_workflows` |
| `ma_lead_score_models` | `id, tenant_id, name, explicit_rules, implicit_rules, decay_config, thresholds, is_active` | Applied to all CRM contacts |
| `ma_marketing_assets` | `id, tenant_id, asset_type, name, content, template_ref, subject_line, preview_text, version, is_active` | Referenced by nurture sequences |
| `ma_attribution_rules` | `id, tenant_id, model_type, lookback_days, channel_weights, custom_weights, effective_from` | Applied to closed opportunities |

## Events

### Events Produced

| Event Type | Payload | Description |
|------------|---------|-------------|
| `crm.campaign.launched` | `{campaign_id, campaign_name, segment_id, contact_count, channel, launched_at, launched_by}` | A marketing campaign has been launched and is actively processing contacts |
| `crm.campaign.contact-engaged` | `{campaign_id, contact_id, engagement_type, asset_id, url, engaged_at}` | A contact has engaged with a campaign asset (email open, click, page visit, form submit) |
| `crm.lead.score-updated` | `{contact_id, previous_score, new_score, previous_tier, new_tier, delta, reason, updated_at}` | A contact's lead score has been recalculated and tier potentially changed |
| `crm.campaign.conversion-credited` | `{campaign_id, contact_id, touchpoint_id, opportunity_id, revenue, attribution_model, credit_amount}` | Revenue attribution has been applied to a campaign touchpoint for a closed opportunity |

### Events Consumed

| Event Type | Source | Action |
|------------|--------|--------|
| `crm.cdp.profile-updated` | CRM Service | Re-evaluate dynamic segment membership and lead score for updated profile |
| `commerce.order.placed` | Commerce Service | Create conversion touchpoint for attribution; adjust lead score for purchase behavior |
| `commerce.customer.created` | Commerce Service | Screen for nurture campaign eligibility; apply initial lead scoring |
| `platform.email.delivered` | Platform Service | Record email delivery touchpoint; update send analytics and engagement metrics |
| `config.changed` | Config Service | Reload scoring models, campaign templates, segmentation rules, and throttling parameters |

## Integration Points

| Integration | Service | Protocol | Description |
|-------------|---------|----------|-------------|
| Customer Data Platform | CRM Service | Internal API | Unified customer profiles for segmentation and lead scoring input |
| Email Delivery | Platform Service | Event-driven | Email send execution, delivery tracking, bounce and complaint processing |
| Order Management | Commerce Service | Event-driven | Purchase events feed attribution models and behavioral scoring |
| Sales CRM | CRM Service | Internal API | Lead handoff when score reaches Sales-Ready tier; opportunity data for attribution |
| Platform Config | Config Service | Event-driven | Dynamic reload of scoring models, templates, and compliance settings |

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `crm.ma.lead_score.max_score` | 100 | Maximum achievable lead score |
| `crm.ma.campaign.cooldown_days` | 7 | Days before contact re-entry into same campaign |
| `crm.ma.email.rate_limit_per_hour` | 10000 | Maximum email sends per hour per tenant |
| `crm.ma.attribution.lookback_days` | 90 | Lookback window for attribution touchpoints |
| `crm.ma.ab_test.min_confidence_pct` | 95 | Minimum statistical confidence for A/B test results |
| `crm.ma.segment.refresh_interval_hours` | 24 | Maximum hours between dynamic segment refreshes |

## Implementation Notes

- The nurture campaign engine implements a petri-net-inspired state machine where each contact's position in the workflow is tracked as `(campaign_id, current_step_id, entered_at, state_data)`. Step transitions are triggered by events (time elapsed, behavioral trigger, manual advance) and evaluated against branch conditions.
- Lead scoring uses an additive model: `total_score = Σ(explicit_weights) + Σ(implicit_event_scores) - Σ(decay_deductions)`. Scores are clamped to [0, 100]. Recalculation is event-driven — each new behavioral event triggers an incremental score update rather than a full recompute.
- Email template rendering uses a token replacement engine with support for personalization fields (`{{first_name}}`, `{{company}}`), conditional content blocks, and dynamic content based on segment membership. Templates are versioned; sends reference a specific version.
- A/B test variant assignment uses deterministic hashing (`hash(contact_id + experiment_id) % 100`) to ensure consistent variant membership across sessions. New contacts are evenly distributed across variants.
- Campaign attribution is computed asynchronously as a batch job triggered by opportunity close events. The attribution engine collects all touchpoints within the lookback window for the contact, applies the configured model, and writes credit allocations to the attribution store.
- Segment evaluation uses a rule engine that compiles segment criteria into SQL predicates executed against the CDP profile table. Dynamic segments are materialized as database views refreshed on schedule. Large segments (>1M contacts) use incremental refresh based on profile modification timestamps.

## Cross-References

- [B2C Commerce](b2c-commerce.md) — Storefront conversion tracking feeds campaign attribution
- [Sales Performance Management](sales-performance-management.md) — Marketing-sourced pipeline contribution
- [CRM Service](../06-services/crm.md) — Customer profiles, opportunities, and lead management
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
- [NFR: Performance](../10-planning/nfr.md) — Score update < 500ms, campaign launch < 30s
