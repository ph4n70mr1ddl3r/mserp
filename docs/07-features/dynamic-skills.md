# Dynamic Skills

AI-powered skills ontology, gap analysis, and matching engine with hierarchical taxonomy, automated skill extraction, and learning path recommendations, managed by the HCM Service.

## Overview

Dynamic Skills provides an intelligent skills management platform built on a hierarchical ontology that supports AI-powered skill extraction from job descriptions, employee profiles, and project outcomes. The module maintains a living skills taxonomy that evolves with organizational needs, performs automated skill gap analysis comparing employee profiles against role requirements, generates personalized learning path recommendations, implements dynamic skill rating decay to reflect skill currency, and powers cross-employee skill comparison for talent mobility and team composition. It integrates with Report Service for ML inference, Opportunity Marketplace for matching, and Goals & Career Development for career planning.

## Data Flow Diagram

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  Job Descriptions │────▶│  AI Skill        │────▶│  Skills Taxonomy    │
│  / Profiles /    │     │  Extraction      │     │  ┌───────────────┐  │
│  Project Outcomes │     │  Engine          │     │  │ Hierarchical  │  │
└──────────────────┘     └──────────────────┘     │  │ Ontology      │  │
                                                   │  └───────┬───────┘  │
┌──────────────────┐     ┌──────────────────┐     └──────────┼──────────┘
│  Employee Skill  │────▶│  Gap Analysis     │◀───────────────┘
│  Assessments     │     │  Engine           │
└──────────────────┘     └──────────────────┘     ┌──────────▼──────────┐
                                                   │  Learning Path      │
┌──────────────────┐     ┌──────────────────┐     │  Recommendations    │
│  Role            │────▶│  Skill Rating     │────▶│  & Matching         │
│  Requirements    │     │  Decay Calculator │     └──────────┬──────────┘
└──────────────────┘     └──────────────────┘                │
                                                    ┌────────▼──────────┐
┌──────────────────┐     ┌──────────────────┐     │  Skill Marketplace │
│  Report Service  │────▶│  ML Inference     │────▶│  & Comparison      │
│  (ML Pipeline)   │     │  Results Handler  │     │  Engine            │
└──────────────────┘     └──────────────────┘     └────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Skills Taxonomy** | HCM Service (Rust) | Hierarchical skill categories and skill definitions with synonyms, related skills, and proficiency descriptors |
| **AI Skill Extraction** | HCM Service + Report Service | Natural language processing to identify and extract skills from unstructured text (job descriptions, profiles, reviews) |
| **Skill Gap Analysis** | HCM Service | Automated comparison of employee skill profiles against role requirements with severity scoring and priority ranking |
| **Learning Recommendations** | HCM Service | Personalized learning path generation based on identified gaps, career goals, and available training resources |
| **Rating Decay Engine** | HCM Service | Time-based skill proficiency decay calculation reflecting lack of recent use, with configurable decay curves per skill category |
| **Cross-Employee Comparison** | HCM Service | Team and organizational skill landscape analysis for staffing, mobility, and workforce planning |
| **Skill Marketplace** | HCM Service | Skill supply-demand mapping across the organization for talent allocation optimization |
| **Storage** | PostgreSQL | All taxonomy, profile, assessment, gap, and recommendation data in HCM database |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Taxonomy Manager | CRUD for skill categories, skill definitions, synonyms, and relationships | HCM Service |
| Skill Extraction Engine | AI-powered skill identification from text inputs via Report Service ML pipeline | HCM Service |
| Employee Skill Profiler | Aggregation and maintenance of per-employee skill profiles from multiple sources | HCM Service |
| Gap Analysis Engine | Current vs. required skill comparison with gap severity and priority calculation | HCM Service |
| Recommendation Generator | Learning path and development activity suggestions based on gaps and goals | HCM Service |
| Rating Decay Calculator | Time-weighted proficiency decay with configurable decay curves and refresh triggers | HCM Service |
| Skill Comparator | Cross-employee and team-level skill landscape analysis | HCM Service |
| Marketplace Mapper | Organization-wide skill supply-demand analysis and visualization | HCM Service |

## Skill Proficiency Levels

| Level | Name | Description | Decay Rate |
|-------|------|-------------|------------|
| 1 | Novice | Foundational awareness; requires supervision | Slow |
| 2 | Beginner | Can perform basic tasks with guidance | Moderate |
| 3 | Intermediate | Independent execution of standard tasks | Standard |
| 4 | Advanced | Complex problem-solving; can mentor others | Slow |
| 5 | Expert | Authority in the domain; drives innovation | Very slow |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Opportunity Marketplace | HCM Service | Internal API | Skill profiles and matching scores for opportunity recommendations |
| Goals & Career Development | HCM Service | Internal API | Target role skill requirements for gap analysis |
| Learning Experience (LXP) | HCM Service | Event-driven | Training completions update skill profiles |
| Performance Management | HCM Service | Internal API | Review outcomes inform skill assessments |
| Employee Profile | HCM Service | Internal API | Employee data for profile initialization |
| Report Service (ML) | Report Service | Event-driven | ML inference results for skill extraction and recommendations |
| Notification Service | Platform Service | Event-driven | Gap alerts, recommendation notifications, assessment reminders |
| Config Service | Config Service | Event-driven | Taxonomy settings, decay curves, matching weights |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `hr.skill.created` | `{skill_id, tenant_id, name, category_id, taxonomy_version, created_by}` | New skill definition added to taxonomy | Notification Service, Report Service |
| `hr.skill.assessed` | `{assessment_id, tenant_id, employee_id, skill_id, level, assessor_type, evidence_ref, assessed_at}` | Skill assessment recorded (manual or auto) | Gap Analysis Engine, Opportunity Marketplace, Report Service |
| `hr.skill.gap-identified` | `{gap_id, tenant_id, employee_id, skill_id, current_level, required_level, severity, role_id}` | Gap analysis detects deficiency | Recommendation Generator, Notification Service |
| `hr.skill.recommendation.generated` | `{recommendation_id, tenant_id, employee_id, gap_id, learning_path_id, priority, estimated_effort_hours}` | Learning recommendation created for identified gap | Notification Service, LXP, Employee Dashboard |
| `hr.skill.taxonomy-updated` | `{taxonomy_version, tenant_id, categories_added, categories_modified, skills_added, skills_deprecated}` | Taxonomy structure modified | Report Service, Notification Service |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `hr.employee.hired` | HCM Service | Initialize empty skill profile; trigger extraction from resume/job description |
| `hr.employee.updated` | HCM Service | Re-run extraction if job title or department changes |
| `hr.competency.assessed` | HCM Service | Map competency assessment results to skill profile entries |
| `hr.training.completed` | HCM Service | Update relevant skill proficiency; refresh decay timer |
| `hr.review.completed` | HCM Service | Extract skill-related feedback and update assessments |
| `report.ml.inference.complete` | Report Service | Process ML-extracted skills, create auto-assessments |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `skills` | `id, tenant_id, name, category_id, description, synonyms (JSONB), proficiency_descriptors (JSONB), status, taxonomy_version, external_id, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `skill_categories`; referenced by `employee_skills`, `skill_assessments_auto` |
| `skill_categories` | `id, tenant_id, name, parent_category_id, level, description, sort_order, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | Self-referencing hierarchy; has many `skills` |
| `employee_skills` | `id, tenant_id, employee_id, skill_id, level, source, last_used_at, decayed_level, assessed_at, created_at, updated_at, created_by, updated_by, version, is_deleted` | References `employee`, `skills` |
| `skill_assessments_auto` | `id, tenant_id, employee_id, skill_id, extracted_level, confidence_score, source_type, source_ref, inference_id, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | References `employee`, `skills`; links to ML inference |
| `skill_gap_analyses` | `id, tenant_id, employee_id, role_id, analysis_date, gaps (JSONB), total_gap_count, critical_gap_count, overall_readiness_score, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | References `employee`; consumed by Recommendation Generator |
| `skill_recommendations` | `id, tenant_id, employee_id, gap_id, skill_id, recommendation_type, target_level, learning_path_ref (JSONB), priority, estimated_effort_hours, status, completed_at, created_at, updated_at, created_by, updated_by, version, is_deleted` | References `employee`, `skill_gap_analyses` |

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `hr.skills.decay.enabled` | true | Enable automatic skill rating decay |
| `hr.skills.decay.default_curve` | `linear` | Default decay curve type (linear, exponential, step) |
| `hr.skills.decay.inactive_months` | 12 | Months of non-use before decay begins |
| `hr.skills.extraction.confidence_threshold` | 0.75 | Minimum ML confidence to auto-create skill assessment |
| `hr.skills.gap.severity_thresholds` | `{critical: 3, moderate: 2, minor: 1}` | Level deltas for gap severity classification |
| `hr.skills.taxonomy.max_depth` | 4 | Maximum category hierarchy depth |
| `hr.skills.auto_assess_from_training` | true | Automatically assess skills upon training completion |

## Cross-References

- [Opportunity Marketplace](./opportunity-marketplace.md) — Skill profiles power matching engine
- [Goals & Career Development](./goals-career-development.md) — Career targets drive gap analysis
- [HCM Service](../06-services/hcm.md) — HCM service specification
- [Report Service](../06-services/report.md) — ML inference for skill extraction
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
- [NFR: Performance](../10-planning/nfr.md) — Gap analysis < 2s per employee; taxonomy lookup < 50ms
