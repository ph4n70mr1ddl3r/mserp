# CRM Service

| Aspect | Details |
|--------|---------|
| Port | 8016 |
| Database | `crm_db` |
| Responsibilities | Customer relationship management, marketing automation, customer analytics, adaptive intelligence, contact center, social selling, A/B testing |

> **Service Boundary:** CRM owns the PRE-SALE funnel (leads, opportunities, campaigns). Commerce Service owns the ORDER lifecycle (quotations, orders, deliveries). The boundary is at opportunity-to-quote conversion.

## Modules

| Module | Description |
|--------|-------------|
| Contact Management | Contact CRUD, segmentation, tagging, activity history |
| Lead Management | Lead capture (web forms, imports, API), scoring, qualification, assignment rules |
| Deal Pipeline | Opportunity stages, probability, expected close date, competitor tracking |
| Campaigns | Email campaigns, audience segmentation, A/B testing, open/click tracking |
| Marketing Automation | Drip sequences, trigger-based actions, lead nurturing workflows |
| Customer Analytics | Customer lifetime value, acquisition cost, retention rate, NPS tracking |
| Account Management | Key account identification, relationship mapping, strategic account plans |
| Activity Tracking | Calls, emails, meetings, tasks, notes linked to contacts/deals |
| Customer Service | Case management, SLA tracking, knowledge base, escalation workflows |
| Social Listening | Social media mention tracking, sentiment analysis integration |

## Field Service Management

| Module | Description |
|--------|-------------|
| Service Orders | Service order creation from CRM cases, warranty claims, and maintenance contracts |
| Technician Scheduling | Skills-based assignment, geographic optimization, real-time calendar management |
| Work Orders | Detailed service work orders with tasks, parts, and labor requirements |
| Parts Logistics | Spare parts reservation and dispatch, van stock management, returns |
| SLA Management | Response time and resolution time SLA tracking with escalation |
| Field Technician Portal | Mobile-optimized portal with schedule, work order details, parts inventory, and completion forms |
| Service Contracts | Service level agreements, warranty tracking, preventive maintenance agreements |
| Service Analytics | Technician utilization, first-time fix rate, customer satisfaction, cost-per-service-call |

## Survey & Feedback Management

| Module | Description |
|--------|-------------|
| Survey Builder | Drag-and-drop survey designer with multiple question types (rating, multiple choice, open text, NPS) |
| Distribution Channels | Email, in-app, SMS, website embed, post-interaction trigger |
| NPS / CSAT Collection | Automated Net Promoter Score and Customer Satisfaction surveys at configurable touchpoints |
| Response Analytics | Real-time response dashboards, sentiment analysis, trend analysis, cross-tabulation |
| Trigger Surveys | Event-driven survey dispatch (post-purchase, post-support, post-delivery) |
| Integration | Auto-create CRM cases from negative responses, feed sentiment data to customer analytics |

## Sales Territory & Quota Planning

| Module | Description |
|--------|-------------|
| Territory Design | Geographic, industry, and account-based territory modeling and assignment |
| Territory Optimization | Balanced territory allocation based on revenue potential, workload, and coverage metrics |
| Quota Allocation | Top-down and bottom-up quota allocation by territory, product, and sales rep |
| What-If Modeling | Territory and quota scenario planning with revenue impact analysis |
| Historical Analysis | Territory performance trending, quota attainment analysis, overlay comparison |
| Integration | Territory assignments synced to CRM pipeline and opportunity routing |

## Customer Data Platform (CDP)

> CDP maintains unified customer profiles. Commerce publishes transactional events (orders, deliveries, returns) that CDP consumes.

| Module | Description |
|--------|-------------|
| Unified Profiles | Single customer view aggregated from Commerce, CRM, Support, and Marketing touchpoints |
| Identity Resolution | Deterministic and probabilistic matching across channels, devices, and identifiers |
| Segmentation | Dynamic segment builder with behavioral, demographic, and transactional criteria |
| Journey Orchestration | Multi-step customer journey designer with trigger-based actions and cross-channel delivery |
| Engagement Scoring | Real-time engagement score computation based on recency, frequency, and monetary value |
| Cross-Channel Analytics | Attribution modeling, channel preference analysis, and journey completion rate reporting |
| Profile Enrichment | Third-party data enrichment integration for enhanced customer attributes |

## Adaptive Intelligence (CRM)

| Module | Description |
|--------|-------------|
| Lead Scoring | ML-driven lead scoring based on engagement patterns and conversion history |
| Next-Best-Action | Recommended actions for sales reps based on deal context and history |
| Churn Prediction | Real-time churn risk scoring with retention action recommendations |
| Cross-Sell / Up-Sell | Product recommendations based on purchase history and customer profile |
| Deal Risk Assessment | Automated deal risk scoring based on engagement, timeline, and competitive signals |

## Enterprise Contact Center

| Module | Description |
|--------|-------------|
| Omnichannel Routing | Intelligent routing of voice, email, chat, SMS, and social interactions with skills-based assignment |
| IVR / Voice Bot | Interactive voice response with NLP-powered voice bot for self-service and intelligent call routing |
| Workforce Management | Contact center agent scheduling, forecast-based staffing, real-time adherence monitoring |
| Quality Management | Call recording, screen capture, evaluation forms, and calibration workflows for agent quality |
| Script Management | Configurable agent scripts with branching logic, compliance prompts, and knowledge base integration |
| Queue Management | Configurable queues with priority rules, overflow strategies, and callback scheduling |
| Agent Dashboard | Unified agent desktop with customer 360 view, interaction history, and knowledge base |
| Interaction Analytics | Speech analytics, sentiment analysis, and interaction trend reporting across all channels |
| Outbound Campaigns | Predictive and preview dialer campaigns with compliance controls (DNC, TCPA) |

## Social Selling

| Module | Description |
|--------|-------------|
| Social Profile Enrichment | Automatic enrichment of CRM contacts with social media profile data and activity signals |
| Social Engagement Tracking | Track social interactions linked to CRM contacts and opportunities |
| Social Intelligence | AI-driven social selling index scoring with recommended actions for sales representatives |
| Relationship Mapping | Social network relationship visualization for identifying warm introduction paths |
| Compliance Guardrails | Configurable social media policy enforcement with content approval workflows |

## CX Digital & A/B Testing

| Module | Description |
|--------|-------------|
| A/B Testing | Controlled A/B tests for web pages, email campaigns, and in-app experiences |
| Multivariate Testing | Multi-factor testing with fractional factorial designs for complex experience optimization |
| Statistical Engine | Automated statistical significance calculation with confidence intervals and sample size recommendations |
| Personalization | AI-driven personalization rules based on customer profile, behavior, and context |
| Content Variants | Visual content editor for creating test variants without developer involvement |
| Reporting | Test performance dashboards with lift measurement, confidence levels, and revenue impact |

## Marketing Automation

**Marketing Automation** — Multi-step nurture campaign orchestration with visual workflow builder. Behavioral lead scoring using engagement signals (email opens, page visits, content downloads). Email marketing with template builder, A/B testing, and delivery optimization. Landing page builder with form capture and CRM integration. Campaign attribution models (first-touch, last-touch, multi-touch, custom). Marketing calendar with drag-and-drop scheduling. Segment-based targeting from CDP unified profiles. Campaign performance analytics with ROI tracking. See [marketing-automation.md](../07-features/marketing-automation.md).

## Database Tables

> All tables include standard columns per [SPEC.md §9.1](../SPEC.md).

| Table | Additional Columns |
|-------|-------------------|
| `contacts` | `first_name VARCHAR(100)`, `last_name VARCHAR(100)`, `email VARCHAR(255)`, `phone VARCHAR(50)`, `company VARCHAR(255)`, `title VARCHAR(100)`, `source VARCHAR(30)`, `status VARCHAR(20)`, `tags TEXT[]` |
| `accounts` | `name VARCHAR(255)`, `website VARCHAR(255)`, `industry VARCHAR(100)`, `size VARCHAR(30)`, `revenue DECIMAL`, `billing_address JSONB`, `shipping_address JSONB`, `owner_id UUID`, `status VARCHAR(20)` |
| `leads` | `contact_id UUID`, `account_id UUID`, `source VARCHAR(30)`, `status VARCHAR(20)`, `score INT`, `assigned_to UUID`, `converted_opportunity_id UUID`, `converted_at TIMESTAMPTZ` |
| `opportunities` | `account_id UUID`, `contact_id UUID`, `name VARCHAR(255)`, `stage VARCHAR(30)`, `amount DECIMAL`, `currency VARCHAR(3)`, `probability DECIMAL`, `expected_close_date DATE`, `assigned_to UUID`, `competitor_ids UUID[]` |
| `campaigns` | `name VARCHAR(255)`, `type VARCHAR(30)`, `status VARCHAR(20)`, `start_date DATE`, `end_date DATE`, `budget DECIMAL`, `currency VARCHAR(3)`, `actual_cost DECIMAL`, `responses INT`, `converted_leads INT` |
| `activities` | `subject VARCHAR(255)`, `type VARCHAR(20)`, `status VARCHAR(20)`, `due_at TIMESTAMPTZ`, `completed_at TIMESTAMPTZ`, `related_contact_id UUID`, `related_opportunity_id UUID`, `assigned_to UUID`, `notes TEXT` |
| `service_cases` | `case_number VARCHAR(50)`, `contact_id UUID`, `account_id UUID`, `type VARCHAR(30)`, `priority VARCHAR(10)`, `status VARCHAR(20)`, `assigned_to UUID`, `sla_due_at TIMESTAMPTZ`, `resolved_at TIMESTAMPTZ`, `satisfaction_score INT` |
| `surveys` | `title VARCHAR(255)`, `type VARCHAR(20)`, `status VARCHAR(20)`, `questions JSONB`, `trigger_event VARCHAR(50)`, `start_date DATE`, `end_date DATE`, `response_count INT` |
| `territories` | `name VARCHAR(255)`, `type VARCHAR(20)`, `region VARCHAR(100)`, `owner_id UUID`, `parent_id UUID`, `rules JSONB`, `is_active BOOLEAN` |
| `cdp_unified_profiles` | `primary_email VARCHAR(255)`, `primary_phone VARCHAR(50)`, `identity_ids JSONB`, `segments TEXT[]`, `engagement_score DECIMAL`, `lifetime_value DECIMAL`, `first_seen_at TIMESTAMPTZ`, `last_activity_at TIMESTAMPTZ`, `attributes JSONB` |
| `field_service_orders` | Field service work orders with technician assignment and scheduling |
| `survey_responses` | Individual survey response records with answers and metadata |
| `ab_tests` | A/B test configurations with variants and statistical parameters |
| `social_mentions` | Social media mentions linked to CRM contacts with sentiment |
| `cdp_segments` | Customer segment definitions with membership criteria |
| `cdp_journeys` | Customer journey definitions with trigger and step configuration |
| `contact_center_interactions` | Omnichannel interaction records with channel and agent assignment |
| `contact_center_queues` | Queue configurations with routing rules and overflow strategies |

## Events Published

| Event | Description |
|-------|-------------|
| `crm.contact.created` | Contact created |
| `crm.contact.updated` | Contact updated |
| `crm.lead.created` | Lead created |
| `crm.lead.converted` | Lead converted to opportunity |
| `crm.opportunity.created` | Opportunity created |
| `crm.opportunity.stage-changed` | Opportunity moved to new stage |
| `crm.opportunity.won` | Deal won |
| `crm.opportunity.lost` | Deal lost |
| `crm.campaign.launched` | Campaign launched |
| `crm.campaign.completed` | Campaign completed |
| `crm.activity.logged` | Activity (call, email, meeting) logged |
| `crm.case.created` | Service case created |
| `crm.case.resolved` | Service case resolved |
| `crm.service-order.created` | Field service order created |
| `crm.service-order.completed` | Field service order completed |
| `crm.service-order.dispatched` | Technician dispatched for service |
| `crm.survey.created` | Survey created |
| `crm.survey.response.received` | Survey response submitted |
| `crm.territory.updated` | Sales territory assignment updated |
| `crm.quota.allocated` | Sales quota allocated |
| `crm.cdp.profile.created` | Unified customer profile created in CDP |
| `crm.cdp.profile.updated` | Unified customer profile updated |
| `crm.cdp.profile.merged` | Customer profiles merged (identity resolution) |
| `crm.cdp.segment.updated` | Customer segment membership updated |
| `crm.cdp.journey.step.completed` | Customer journey step completed |
| `crm.cdp.engagement-score.updated` | Customer engagement score recalculated |
| `crm.contact.center.interaction.created` | Contact center interaction received |
| `crm.social.mention.detected` | Social media mention detected |
| `crm.ab.test.completed` | A/B test completed with results |
| `crm.campaign.contact-engaged` | Campaign contact engagement recorded |
| `crm.lead.score-updated` | Lead score updated |
| `crm.campaign.conversion-credited` | Campaign conversion credited |

## Events Consumed

Inbox binding: `crm.inbox` binds to the following routing keys:

| Binding Pattern | Events Consumed |
|----------------|-----------------|
| `crm.#` | Self-binding for saga compensation |
| `commerce.customer.#` | `commerce.customer.created`, `commerce.customer.updated`, `commerce.customer.deleted` |
| `commerce.order.#` | `commerce.order.created`, `commerce.order.submitted`, `commerce.order.fulfilled`, `commerce.order.cancelled`, `commerce.order.returned` |
| `config.changed` | `config.changed` |

## See Also

- [Commerce Service](commerce.md)
- [Report Service](report.md)
- [Integration Service](integration.md)
- [Architecture Overview](../01-architecture/overview.md)
