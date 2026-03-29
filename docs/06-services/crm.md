# CRM / Marketing Service

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

## See Also

- [Commerce Service](commerce.md)
- [Report Service](report.md)
- [Integration Service](integration.md)
- [Architecture Overview](../01-architecture/overview.md)
