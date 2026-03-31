# Channel Revenue Management

Partner pricing, rebate programs, and channel incentive management for multi-tier distribution networks, managed by the Commerce Service. Finance Service provides invoice and payment data for rebate accruals; Workflow Service manages rebate approval workflows.

> **Ownership Note**: Channel Revenue Management is owned by Commerce Service. Finance Service provides invoice approval and payment data via events (`finance.invoice.*`, `finance.payment.*`). Workflow Service manages rebate claim approval and deal registration approval. Commerce owns channel partner management, rebate programs, special pricing agreements, deal registration, and all channel events.

## Overview

Channel Revenue Management provides comprehensive tools for managing indirect sales channels through partner tiering, incentive programs, and pricing agreements. The module supports channel partner lifecycle management from registration through performance-based tier progression, tiered pricing rules that adjust based on partner tier and volume, rebate program creation with configurable accrual and claim rules, sell-in and sell-through tracking for channel inventory visibility, and special pricing agreements (SPAs) for negotiated partner-specific pricing. It includes channel fund management for Market Development Funds (MDF) and co-op programs, deal registration to prevent channel conflict, and partner performance scorecards that drive tier decisions. Accrual calculations run automatically against eligible transactions, and claim validation ensures rebate payouts align with program rules.

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Channel Partner Tier Management** | Commerce Service (Rust) | Partner registration with qualification criteria, tier assignment based on revenue thresholds and certifications, automated tier re-evaluation on performance cycle boundaries |
| **Tiered Pricing Rules** | Commerce Service (Rust) | Price list engine with tier-based discount schedules, volume-break pricing, and product-category-level rules evaluated at order time |
| **Rebate Program Management** | Commerce Service (Rust) | Configurable rebate programs with eligibility rules, earning periods, payout schedules, tiered rates, and exclusion lists |
| **Sell-In / Sell-Through Tracking** | Commerce Service (Rust) | Track products sold into the channel (sell-in) and through to end customers (sell-through) via POS data feeds or partner reporting, with channel inventory calculation |
| **Special Pricing Agreements (SPAs)** | Commerce Service (Rust) | Negotiated pricing contracts with start/end dates, product scope, volume commitments, and approval workflows |
| **Channel Fund Management** | Commerce Service (Rust) | MDF and co-op fund allocation, claim submission against fund programs, proof-of-performance validation, and fund utilization tracking |
| **Deal Registration** | Commerce Service (Rust) | Partner deal registration with duplicate detection, conflict resolution rules, registration expiry, and approval workflows |
| **Channel Conflict Detection** | Commerce Service (Rust) | Automated detection of overlapping deal registrations, pricing conflicts, and territory violations with configurable resolution rules |
| **Partner Performance Scorecards** | Commerce Service + Report Service | Revenue attainment, deal registration conversion, sell-through velocity, rebate utilization, and compliance metrics driving tier decisions |
| **Accrual Calculation Engine** | Commerce Service (Rust) | Automated rebate accrual calculation on eligible transactions with configurable frequency (real-time, daily, monthly), support for tiered and growth-based rates |
| **Claim Submission & Validation** | Commerce Service + Workflow Service | Partner claim submission with supporting documentation, automated validation against program rules, approval workflow for exceptions |
| **Channel Analytics** | Commerce Service + Report Service | Channel revenue trends, partner contribution analysis, rebate ROI, fund utilization effectiveness, and program performance |
| **Storage** | PostgreSQL | All channel partner, rebate, pricing, fund, deal, and performance data in Commerce database |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Channel Partner Manager | Partner registration, qualification, tier assignment, lifecycle management | Commerce Service |
| Tiered Pricing Engine | Evaluate partner-specific pricing at order time based on tier and volume | Commerce Service |
| Rebate Program Manager | Create, configure, and manage rebate programs and rules | Commerce Service |
| Accrual Calculation Engine | Calculate rebate accruals on eligible transactions | Commerce Service |
| Claim Processing Engine | Validate, approve, and process rebate claims | Commerce Service + Workflow Service |
| SPA Manager | Create, negotiate, approve, and track special pricing agreements | Commerce Service |
| Channel Fund Manager | MDF/co-op fund allocation, claims, and utilization tracking | Commerce Service |
| Deal Registration Engine | Register deals, detect conflicts, manage approvals | Commerce Service |
| Conflict Detection Engine | Identify and resolve channel conflicts automatically | Commerce Service |
| Sell-Through Tracker | Track channel inventory from sell-in to sell-through | Commerce Service |
| Performance Scorecard Engine | Calculate partner performance metrics and tier eligibility | Commerce Service + Report Service |
| Channel Analytics | Revenue, rebate, fund, and program analytics | Commerce Service + Report Service |

## Channel Partner Lifecycle

| Stage | Description | Entry Criteria | Exit Criteria |
|-------|-------------|---------------|---------------|
| Prospective | Partner expressed interest, not yet qualified | Registration submitted | Application approved or rejected |
| Registered | Partner qualified and onboarded | Qualification criteria met, agreements signed | Tier assignment or churn |
| Silver | Base tier partner with standard benefits | Minimum revenue threshold met | Upgrade or downgrade |
| Gold | Mid-tier partner with enhanced benefits | Revenue and certification thresholds met | Upgrade or downgrade |
| Platinum | Top-tier partner with premium benefits | High revenue, certifications, and strategic value | Downgrade on re-evaluation |
| Suspended | Partner temporarily inactive | Compliance violation or inactivity | Reactivation or termination |
| Terminated | Partner relationship ended | Contract breach or mutual agreement | Terminal |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Sales Orders | Commerce Service | Internal API | Order data for sell-in tracking and rebate accrual triggers |
| Customer Management | Commerce Service | Internal API | Channel partner profiles linked to customer master |
| Pricing Engine | Commerce Service | Internal API | SPA and tiered pricing applied at order time |
| Invoice Processing | Finance Service | Event-driven | Invoice approvals trigger rebate accrual eligibility check |
| Payment Processing | Finance Service | Event-driven | Payment receipts trigger rebate payout eligibility |
| Approval Workflows | Workflow Service | Event-driven | Rebate claim approval, deal registration approval, SPA approval |
| Configuration | Config Service | Event-driven | Channel program parameters, tier thresholds, conflict rules |
| Notification Service | Platform Service | Event-driven | Partner notifications, claim status updates, tier changes |
| Report Service | Report Service | Internal API | Performance scorecards, channel analytics, rebate ROI |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `commerce.channel.partner-registered` | `{partner_id, tier, company_name, territory, certifications[], registered_at}` | New channel partner registered and qualified | Notification, Dashboard, Performance Tracker |
| `commerce.channel.rebate-accrued` | `{accrual_id, program_id, partner_id, transaction_id, amount, currency, period}` | Rebate accrual calculated on eligible transaction | Finance, Dashboard, Claim Engine |
| `commerce.channel.rebate-claimed` | `{claim_id, program_id, partner_id, accrual_ids[], amount, currency, status}` | Rebate claim submitted by partner or auto-generated | Finance, Workflow, Dashboard |
| `commerce.channel.spa-created` | `{spa_id, partner_id, product_scope[], pricing_rules[], valid_from, valid_to}` | Special pricing agreement created and approved | Pricing Engine, Notification |
| `commerce.channel.deal-registered` | `{deal_id, partner_id, customer_id, products[], estimated_value, territory, expires_at}` | Deal registration submitted by partner | Conflict Engine, Workflow, Dashboard |
| `commerce.channel.fund-allocated` | `{fund_id, program_id, partner_id, amount, currency, period, purpose}` | Channel fund (MDF/co-op) allocated to partner | Fund Manager, Notification, Dashboard |
| `commerce.channel.conflict.detected` | `{conflict_id, deal_ids[], partner_ids[], conflict_type, resolution, detected_at}` | Channel conflict identified between deal registrations | Notification, Workflow, Dashboard |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `commerce.order.completed` | Commerce Service | Evaluate rebate eligibility, update sell-in tracking, accrue rebates |
| `commerce.customer.created` | Commerce Service | Link new customer to channel partner if applicable |
| `finance.invoice.approved` | Finance Service | Validate rebate accrual against approved invoice |
| `finance.payment.received` | Finance Service | Trigger rebate payout eligibility check |
| `workflow.step.approved` | Workflow Service | Finalize rebate claim, deal registration, or SPA approval |
| `config.changed` | Config Service | Update channel program parameters, tier thresholds, conflict rules |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `channel_partners` | `id, tenant_id, company_name, tier, territory, certifications[], contact_email, status, qualified_at, created_at, updated_at, created_by, updated_by, version, is_deleted` | Has many `rebate_claims`, `deal_registrations`, `channel_funds`, `channel_performance_scores` |
| `channel_tiers` | `id, tenant_id, tier_name, tier_level, min_revenue, required_certifications[], benefits{}, pricing_discount_pct, fund_allocation_pct, created_at, updated_at, created_by, updated_by, version, is_deleted` | Referenced by `channel_partners` |
| `rebate_programs` | `id, tenant_id, name, program_type, eligibility_rules{}, earning_period_start, earning_period_end, payout_schedule, rates[], exclusion_rules[], status, created_at, updated_at, created_by, updated_by, version, is_deleted` | Has many `rebate_claims` |
| `rebate_claims` | `id, tenant_id, program_id, partner_id, accrual_ids[], claim_amount, currency, supporting_docs[], status, submitted_at, approved_at, paid_at, rejection_reason, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `rebate_programs`, `channel_partners` |
| `special_pricing_agreements` | `id, tenant_id, partner_id, name, product_scope[], pricing_rules{}, volume_commitments, valid_from, valid_to, status, approved_by, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `channel_partners` |
| `channel_funds` | `id, tenant_id, partner_id, program_type, allocated_amount, utilized_amount, remaining_amount, currency, period, purpose, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `channel_partners` |
| `deal_registrations` | `id, tenant_id, partner_id, customer_id, opportunity_name, products[], estimated_value, currency, territory, status, conflict_id, expires_at, approved_at, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `channel_partners`, optionally links to conflict |
| `channel_performance_scores` | `id, tenant_id, partner_id, period, revenue_attainment_pct, deal_conversion_rate, sell_through_velocity, rebate_utilization_pct, compliance_score, overall_score, tier_recommendation, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `channel_partners` |

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `commerce.channel.deal_registration_expiry_days` | 90 | Days before unapproved deal registration expires |
| `commerce.channel.conflict_detection_enabled` | true | Enable automatic channel conflict detection |
| `commerce.channel.accrual_calculation_frequency` | daily | Rebate accrual calculation frequency: real-time, daily, monthly |
| `commerce.channel.default_rebate_payout_net_days` | 30 | Default net days for rebate payout after approval |
| `commerce.channel.tier_reevaluation_cycle` | quarterly | Partner tier re-evaluation frequency |
| `commerce.channel.max_deals_per_partner` | 50 | Maximum active deal registrations per partner |
| `commerce.channel.fund_utilization_threshold_pct` | 80 | Alert when fund utilization drops below threshold |

## Cross-References

- [Order Orchestration](order-orchestration.md) — Channel orders flow through orchestration
- [B2C Commerce](b2c-commerce.md) — End-customer commerce driving sell-through
- [Commerce Service](../06-services/commerce.md) — Commerce service specification
- [Finance Service](../06-services/finance.md) — Invoice and payment processing
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
- [NFR: Performance](../10-planning/nfr.md) — Accrual calculation < 60s for daily batch
