# Goals & Career Development

Cascading organizational goals with OKR alignment, career path planning, competency frameworks, and skill gap analysis, managed by the HCM Service.

## Overview

Goals and Career Development provides a unified framework for setting, tracking, and aligning individual and organizational objectives alongside structured career progression planning. The module supports cascading goals from company-level strategy through department and team objectives down to individual contributors, with OKR-style key results and progress tracking. Career development features include competency frameworks, skill gap analysis, suggested learning paths, and development plans that connect employee aspirations with organizational capability needs.

## Data Flow Diagram

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  Strategic       │────▶│  Goal Cascade    │────▶│  Individual Goals    │
│  Planning        │     │  Engine          │     │  ┌───────────────┐  │
└──────────────────┘     └──────────────────┘     │  │ Key Results   │  │
                                                   │  │ Progress      │  │
┌──────────────────┐     ┌──────────────────┐     │  └───────┬───────┘  │
│  Competency      │────▶│  Skill Gap       │────▶│          │          │
│  Framework       │     │  Analysis        │     └──────────┼──────────┘
└──────────────────┘     └──────────────────┘                │
                                   │                         │
                    ┌──────────────▼─────────────────────────▼──┐
                    │          Career Development Engine          │
                    │  ┌────────────┐  ┌──────────────────────┐  │
                    │  │ Career     │  │ Development Plan     │  │
                    │  │ Paths      │  │ Builder              │  │
                    │  └─────┬──────┘  └──────────┬───────────┘  │
                    └────────┼────────────────────┼───────────────┘
                             │                    │
                    ┌────────▼──────┐   ┌─────────▼──────────┐
                    │  Learning     │   │  Career             │
                    │  Suggestions  │   │  Conversations      │
                    └───────────────┘   └────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Goal Cascade Engine** | HCM Service (Rust) | Top-down goal decomposition with alignment validation and progress roll-up across organizational hierarchy |
| **OKR Tracking** | HCM Service | Key result tracking with quantitative and qualitative measure types, check-in cadences, and confidence scoring |
| **Competency Framework** | HCM Service | Multi-level competency models with proficiency scales, behavioral indicators, and evidence requirements |
| **Skill Gap Analysis** | HCM Service + Report Service | Current vs. required skill comparison with automated gap identification and prioritized development recommendations |
| **Career Path Builder** | HCM Service | Configurable career ladders and lateral move paths with role competency requirements and progression milestones |
| **Development Plans** | HCM Service + Workflow Service | Action-oriented development plans with SMART goals, learning activities, milestone reviews, and manager approval workflows |
| **Goal Templates** | HCM Service + Config Service | Reusable goal templates by role, department, or organization with customizable key result patterns |
| **Storage** | PostgreSQL | All goal, competency, career path, and development plan data in HCM database |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Goal Manager | CRUD for goals, key results, and alignment links | HCM Service |
| Cascade Engine | Goal decomposition, alignment validation, progress roll-up | HCM Service |
| Competency Manager | Competency framework CRUD, proficiency level definitions | HCM Service |
| Skill Gap Analyzer | Gap identification, prioritization, recommendation generation | HCM Service |
| Career Path Engine | Career ladder configuration, path recommendation, milestone tracking | HCM Service |
| Development Plan Builder | Plan creation, activity assignment, progress tracking | HCM Service |
| Goal Template Library | Template CRUD, instantiation, and customization | HCM Service |
| Career Conversation Manager | Scheduled check-ins, feedback capture, action item tracking | HCM Service |

## Skill Gap Analysis Process

| Step | Action | Output |
|------|--------|--------|
| 1. Profile Assessment | Collect current skills from assessments, certifications, projects | Skill inventory per employee |
| 2. Requirement Mapping | Map target role competency requirements | Required skill profile |
| 3. Gap Identification | Compare current vs. required with proficiency delta calculation | Gap report with severity levels |
| 4. Priority Scoring | Rank gaps by business impact and career relevance | Prioritized development actions |
| 5. Path Suggestion | Match gaps to learning resources and development activities | Recommended learning path |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Performance Management | HCM Service | Internal API | Goal achievement feeds performance reviews |
| Learning Experience (LXP) | HCM Service | Event-driven | Skill gap suggestions trigger learning path enrollment |
| Succession Planning | HCM Service | Internal API | Career readiness scores feed succession candidates |
| Employee Profile | HCM Service | Internal API | Current skills, certifications, and experience data |
| Workflow Engine | Workflow Service | Event-driven | Development plan approval and review workflows |
| Notification Service | Platform Service | Event-driven | Goal assignments, check-in reminders, milestone alerts |
| Report Service | Report Service | Internal API | Goal progress dashboards, skill gap analytics |
| Config Service | Config Service | Event-driven | Goal template configurations, competency framework settings |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `hr.goal.created` | `{goal_id, owner_id, parent_goal_id, period_id, type}` | New goal created | Cascade Engine, Notification Service, Report Service |
| `hr.goal.progress-updated` | `{goal_id, progress_percent, key_result_id, updated_by}` | Key result check-in recorded | Cascade Engine, Performance Management, Dashboard |
| `hr.goal.aligned` | `{goal_id, parent_goal_id, alignment_type, weight}` | Goal linked to parent objective | Cascade Engine, Report Service |
| `hr.career.plan-created` | `{plan_id, employee_id, target_role_id, target_date}` | Development plan created | Workflow Service, Notification Service, LXP |
| `hr.competency.assessed` | `{assessment_id, employee_id, competency_id, level, assessor_id}` | Skill assessment completed | Skill Gap Analyzer, Succession Planning, Report Service |
| `hr.career.milestone-reached` | `{plan_id, milestone_id, employee_id, milestone_type}` | Development milestone completed | Notification Service, Career Conversation Manager |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `hr.employee.hired` | HCM Service | Initialize default goals and competency baseline |
| `hr.employee.transferred` | HCM Service | Re-evaluate goals against new role requirements |
| `hr.review.period-opened` | HCM Service | Trigger goal setting window for period |
| `hr.review.completed` | HCM Service | Update competency assessments from review results |
| `hr.learning.completed` | HCM Service | Update skill profile and recalculate gaps |
| `workflow.step.approved` | Workflow Service | Finalize development plan approval |
| `config.changed` | Config Service | Reload competency framework or template settings |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `goals` | `id, tenant_id, owner_id, parent_goal_id, title, type, status, period_id, progress_percent, due_date` | Has many `goal_alignments`, `key_results`; belongs to `employee` |
| `goal_alignments` | `id, goal_id, parent_goal_id, alignment_type, weight` | Links goals in cascade hierarchy |
| `competency_frameworks` | `id, tenant_id, name, description, proficiency_scale, is_active` | Has many `competency_items` |
| `career_paths` | `id, tenant_id, from_role_id, to_role_id, path_type, competency_requirements` | Links roles in progression ladder |
| `development_plans` | `id, tenant_id, employee_id, target_role_id, status, target_date, approved_by` | Has many `development_activities`, `milestones` |
| `skill_assessments` | `id, tenant_id, employee_id, competency_id, level, assessor_id, evidence, assessed_at` | Belongs to `employee`, `competency` |

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `hr.goals.max_cascade_depth` | 5 | Maximum levels of goal cascade |
| `hr.goals.check_in_frequency_days` | 14 | Default interval for goal progress check-ins |
| `hr.goals.confidence_threshold` | 0.7 | Minimum confidence score for auto-alignment |
| `hr.career.plan_review_cadence_days` | 90 | Interval between development plan reviews |
| `hr.competency.max_proficiency_level` | 5 | Maximum levels in proficiency scale |
| `hr.skills.gap_analysis.min_responses` | 3 | Minimum assessments before gap analysis runs |

## Cross-References

- [Performance Management](../06-services/hcm.md) — Goal achievement feeds review cycle
- [Succession Planning](../06-services/hcm.md) — Career readiness informs talent pipeline
- [Workflow Engine](../06-services/workflow.md) — Development plan approval workflows
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
- [HCM Service](../06-services/hcm.md) — HCM service specification
- [NFR: Performance](../10-planning/nfr.md) — Goal cascade roll-up < 5s across 10k goals
