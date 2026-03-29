# Domain Data Models

## 1. Revenue Recognition Data Model

| Table / Field | Description |
|--------------|-------------|
| `revenue_contracts` | Customer contracts identified for revenue recognition, sourced from sales orders, subscriptions, and project billing |
| `performance_obligations` | Distinct performance obligations per contract, with standalone selling price allocation |
| `revenue_schedules` | Recognition schedule entries: period, amount, status (pending/recognized/deferred) |
| `contract_modifications` | Contract changes with prospective or retrospective adjustment method |
| `revenue_waterfall` | Aggregate view: bookings → deferred → recognized revenue with rollforward |

### Revenue Recognition Event Flow

| Event | Source | Consumer |
|-------|--------|----------|
| `commerce.subscription.created` | Commerce | Finance (create revenue contract) |
| `commerce.subscription.amended` | Commerce | Finance (adjust revenue schedule) |
| `commerce.subscription.billing.completed` | Commerce | Finance (recognize revenue for period) |
| `finance.revenue.contract.approved` | Finance | Report (update revenue waterfall) |
| `finance.revenue.recognized` | Finance | Report (update recognized revenue) |
| `finance.revenue.deferred` | Finance | Report (update deferred revenue) |
| `finance.revenue.adjusted` | Finance | Report (update schedule adjustments) |

- **Finance Service:** Manages revenue contracts, performance obligations, recognition schedules, and disclosure data.
- **Commerce Service:** Notifies Finance of subscription lifecycle events that trigger revenue recognition.
- **Report Service:** Displays deferred revenue by period and subscription; provides revenue waterfall and disclosure reports.

> **Note:** Revenue Recognition obligations are linked to subscription billing schedules. Revenue recognition events (e.g., `finance.revenue.recognized`) update the revenue waterfall and disclosure reports.

## 2. Lease Accounting Data Model

| Table / Field | Description |
|--------------|-------------|
| `lease_contracts` | Lease agreements with classification (operating/finance), terms, payment schedules, and linked assets |
| `lease_right_of_use_assets` | Right-of-use asset records with initial measurement, amortization schedule, and impairment tracking |
| `lease_liabilities` | Lease liability calculations with present value, incremental borrowing rate, and remeasurement history |
| `lease_payments` | Scheduled and actual lease payments with variable payment tracking |
| `lease_modifications` | Lease modification history with remeasurement details and reclassification entries |

## 3. Grant Management Data Model

| Table / Field | Description |
|--------------|-------------|
| `grants` | Grant registrations with funding source, award period, conditions, and budget |
| `grant_budgets` | Grant budget entries with allowable and unallowable cost categories |
| `grant_cost_allocations` | Cost allocations against grants with indirect cost rate calculations |
| `grant_revenue_schedules` | Revenue recognition schedules for milestone-based and cost-reimbursable grants |
| `grant_compliance_checks` | Compliance monitoring records with reporting deadlines and audit trail |

## 4. Joint Venture Data Model

| Table / Field | Description |
|--------------|-------------|
| `joint_ventures` | Joint venture definitions with partner ownership percentages and venturer codes |
| `jv_partners` | Joint venture partners with ownership percentage, billing preferences, and contact details |
| `jv_cost_pools` | Shared cost pools with allocation rules by partner ownership percentage |
| `jv_allocations` | Cost and revenue allocation records with partner-specific breakdowns |
| `jv_partner_billings` | Partner billing statements with detailed cost breakdowns and settlement tracking |

## 5. Warranty Management Data Model

| Table / Field | Description |
|--------------|-------------|
| `warranty_policies` | Warranty policy definitions with coverage terms, duration, and conditions |
| `warranty_registrations` | Product warranty registrations linked to serial/lot numbers and customers |
| `warranty_claims` | Warranty claim submissions with validation status, resolution, and cost tracking |
| `warranty_claim_items` | Claim line items with defect codes, repair/replacement actions, and costs |

## 6. Advanced Collections Data Model

| Table / Field | Description |
|--------------|-------------|
| `collection_strategies` | Configurable collection strategies with aging thresholds and action rules |
| `collection_activities` | Collection activity records (calls, emails, promises to pay) with outcomes |
| `cash_applications` | Cash receipt applications with ML-based invoice matching confidence scores |
| `collection_disputes` | Dispute tracking linked to collection activities with resolution workflows |
| `collection_scores` | Predictive collection scoring per customer with historical trending |

## 7. Data Masking Strategy

### 7.1 Masking for Non-Production Environments

| Technique | Description | When |
|-----------|-------------|------|
| Static masking | Irreversible masking applied to data copies for non-prod environments | On data subset extraction |
| Dynamic masking | Runtime masking based on user permissions (production) | Per query |
| Subsetting | Extract a representative subset of production data for testing | On demand |

### 7.2 Masking Rules by Data Classification

| Classification | Masking Rule | Example |
|---------------|-------------|---------|
| Public | None | Product names |
| Internal | Partial masking | `j***@company.com` |
| Confidential | Full masking / tokenization | `***-**-1234` (SSN) |
| Restricted | Suppression (replace with placeholder) | `[REDACTED]` |

## 8. Advanced Tax Management Data Model

| Table / Field | Description |
|---|---|
| `tax_jurisdictions` | Tax jurisdictions with nexus tracking, filing obligations, and rates schedules |
| `tax_rates` | Tax rates per jurisdiction with effective dates, rate types, and thresholds |
| `tax_assessments` | Tax assessments for transactions with calculated amounts, exemptions, and tax types |
| `tax_exemption_certificates` | Customer and product exemption certificates with validity periods and renewal tracking |
| `tax_reconciliation_records` | Tax liability reconciliation entries with GL account mapping and adjustment tracking |

## 9. Commodity Management Data Model

| Table / Field | Description |
|---|---|
| `commodity_catalog` | Commodity definitions with specifications, grade standards, and market reference pricing |
| `commodity_contracts` | Commodity purchase contracts with quantity, price, delivery schedules, and escalation clauses |
| `commodity_market_prices` | Market price snapshots from exchanges with source, timestamp, and effective period |
| `commodity_hedging_positions` | Hedging position tracking with mark-to-market valuation and hedge effectiveness metrics |
| `commodity_forward_contracts` | Forward contract tracking with settlement scheduling and gain/loss recognition |

## 10. Advanced Spend Analysis Data Model

| Table / Field | Description |
|---|---|
| `spend_transactions` | Classified spend transactions with ML confidence scores and category assignments |
| `spend_categories` | Spend category taxonomy (UNSPSC, SIC, custom) with hierarchical structure |
| `spend_savings_opportunities` | AI-identified savings opportunities with estimated impact and priority |
| `maverick_spend_records` | Off-contract spend records with supplier and category deviation tracking |

## 11. Supplier Diversity Data Model

| Table / Field | Description |
|---|---|
| `diversity_classifications` | Supplier diversity classification records with certification tracking and expiry dates |
| `diversity_spend_records` | Diverse supplier spend records by category, business unit, and classification |
| `diversity_tier_reports` | Tier 1 (direct) and Tier 2 (subcontractor) diversity spend reports entries |
| `diversity_goals` | Configurable diversity spend targets with progress tracking and period definitions |

## 12. Enterprise Health & Safety Data Model

| Table / Field | Description |
|---|---|
| `ehs_incidents` | Safety incident records with severity, type, location, investigation status, and CAPA linkage |
| `ehs_risk_assessments` | Risk assessment records with risk matrix scoring (likelihood x impact) and control measures |
| `ehs_permits` | Digital work permits with approval workflows, validity periods, and safety requirements |
| `ehs_inspections` | Safety inspection records with checklists, findings, and corrective action assignments |
| `ehs_chemicals` | Chemical inventory with SDS linkage, hazard classification, and exposure limits |
| `ehs_occupational_health` | Medical surveillance programs, health assessments, exposure monitoring, and return-to-work records |

## 13. MRO Data Model

| Table / Field | Description |
|---|---|
| `mro_catalog_items` | MRO catalog entries with criticality classification, lead time, and reorder parameters |
| `mro_spare_parts` | Spare parts inventory with criticality, obsolescence tracking, and bin location |
| `mro_repair_orders` | Repair work orders with failure codes, estimates, turnaround time, and parts requirements |
| `mro_overhaul_projects` | Overhaul project records with task breakdown, parts kitting, and scheduling |
| `mro_rotable_components` | Rotable component tracking with repair cycle management, exchange pools, and condition assessment |

## 14. Product Compliance Data Model

| Table / Field | Description |
|---|---|
| `product_regulations` | Regulatory requirements by product category, market, and jurisdiction |
| `product_certifications` | Product certification lifecycle with status tracking, renewal dates, and lab linkage |
| `material_declarations` | Material composition declarations with restricted substance tracking and supplier attestations |
| `compliance_test_plans` | Test plan management with test types, lab assignments, and result tracking |
| `regulatory_changes` | Regulatory change monitoring with impact assessment status and affected product identification |

## 15. Enterprise Contact Center Data Model

| Table / Field | Description |
|---|---|
| `cc_interactions` | Omnichannel interaction records with channel, direction, queue, agent assignment, and duration |
| `cc_queues` | Queue configurations with priority rules, overflow strategies, and skill-based routing |
| `cc_agent_schedules` | Agent scheduling with shift definitions, adherence tracking, and break management |
| `cc_quality_evaluations` | Call evaluation records with scoring criteria, calibration status, and feedback |
| `cc_outbound_campaigns` | Outbound campaign configurations with dialing strategy, compliance controls, and list management |

## 16. Social Selling Data Model

| Table / Field | Description |
|---|---|
| `social_profiles` | Enriched social profiles linked to CRM contacts with platform, follower counts, and engagement metrics |
| `social_engagements` | Social interaction records linked to CRM activities with content, platform, and engagement type |
| `social_intelligence_scores` | Per-user social selling index with component scores and recommended actions |

## 17. A/B Testing Data Model

| Table / Field | Description |
|---|---|
| `ab_tests` | A/B test configurations with target, variants, traffic split, status, and statistical parameters |
| `ab_test_variants` | Test variant definitions with content differences, traffic allocation, and conversion tracking |
| `ab_test_results` | Aggregated test results with lift measurement, confidence intervals, and statistical significance |
| `ab_personalization_rules` | Personalization rule definitions with audience criteria, content variations, and priority |

## 18. Omnichannel Management Data Model

| Table / Field | Description |
|---|---|
| `omnichannel_channels` | Sales channel configurations with type, status, inventory source, and fulfillment rules |
| `omnichannel_cart` | Unified cart entries with channel source, synchronization status, and real-time inventory linkage |
| `omnichannel_order_routing` | Order routing decisions with routing algorithm, fulfillment location, and reason codes |
| `omnichannel_returns` | Cross-channel return records with originating channel, return channel, and policy enforcement |

## 19. Price Optimization Data Model

| Table / Field | Description |
|---|---|
| `price_demand_models` | Demand elasticity models by product/segment with model parameters and accuracy metrics |
| `price_simulations` | Price change scenario records with projected revenue, margin, and volume impact |
| `price_recommendations` | ML-generated price recommendations with confidence scores and business constraints |
| `price_tests` | A/B price test configurations with control/test groups and statistical significance tracking |

## 20. Customer Loyalty Data Model

| Table / Field | Description |
|---|---|
| `loyalty_programs` | Loyalty program definitions with tier structures, earning rules, and benefit catalogs |
| `loyalty_members` | Customer program membership with tier, points balance, and enrollment date |
| `loyalty_points_ledger` | Points accrual and redemption ledger with transaction linkage and expiry tracking |
| `loyalty_tier_transitions` | Tier qualification, promotion, and demotion records with threshold tracking |
| `loyalty_reward_redemptions` | Reward redemption records with fulfillment status and partner linkage |

## 21. Compliance Hub Data Model

| Table / Field | Description |
|---|---|
| `compliance_dashboard_metrics` | Composite compliance health scores per domain with trend tracking |
| `compliance_regulatory_changes` | Regulatory change records with source, relevance scoring, and impact assessment workflow |
| `compliance_control_mappings` | Cross-framework control mapping entries (SOC 2, GDPR, HIPAA, PCI DSS, SOX, ISO 27001) |
| `compliance_calendar_events` | Compliance calendar entries with filing deadlines, audit schedules, and certification renewals |

## 22. Dynamic Discounting Data Model

| Table / Field | Description |
|---|---|
| `dynamic_discount_programs` | Discount program definitions with supplier enrollment, tier structures, and APR calculations |
| `dynamic_discount_offers` | Early payment discount offers with invoice linkage, discount rate, early payment date, and APR |
| `dynamic_discount_acceptances` | Supplier acceptances of early payment offers with acceptance timestamp and terms |
| `dynamic_discount_payments` | Early payment executions with discount captured, payment reference, and cash flow impact |
| `working_capital_metrics` | Working capital KPI snapshots (DSO, DPO, DIO, CCC) with trend tracking by entity and period |

## 23. Financial Reporting Studio Data Model

| Table / Field | Description |
|---|---|
| `report_templates` | Report template definitions with type, framework (GAAP/IFRS/local), layout, and cells |
| `report_template_cells` | Individual report cells with data binding, formatting, XBRL tag, and conditional rules |
| `report_instances` | Generated report instances from templates with period, entities, parameters, and status |
| `report_xbrl_mappings` | XBRL taxonomy element mappings to financial data elements with validation rules |
| `report_distributions` | Report distribution records with recipient, format, delivery method, and timestamp |
| `report_annotations` | Collaborative annotations on report sections with author, thread, and resolution status |

## 24. Enterprise Data Quality Data Model

| Table / Field | Description |
|---|---|
| `dq_profiles` | Data profiling results by entity, source, and dimension with statistical summaries |
| `dq_rules` | Data quality rule definitions with type, configuration, severity, and active status |
| `dq_violations` | Data quality rule violation records with entity, record, rule, and remediation status |
| `dq_match_rules` | Matching and deduplication rule configurations with match criteria and survivorship rules |
| `dq_match_results` | Matching execution results with matched pairs, confidence scores, and merge actions |
| `dq_scorecards` | Data quality scorecards by entity, dimension (completeness, accuracy, consistency, timeliness, validity), and period |

## 25. Supply Chain Collaboration Data Model

| Table / Field | Description |
|---|---|
| `scc_supplier_connections` | Supplier network connections with tier, collaboration level, and data sharing preferences |
| `scc_demand_shares` | Demand signal sharing records with supplier, products, forecast data, and sharing frequency |
| `scc_capacity_commitments` | Supplier capacity commitments with product, quantity, period, and confidence level |
| `scc_cpfr_sessions` | CPFR collaboration sessions with buyer forecast, supplier forecast, and consensus forecast |
| `scc_asn_records` | Advanced Shipment Notice records with supplier, PO linkage, items, and expected delivery |
| `scc_scorecards` | Supplier collaboration scorecards with responsiveness, accuracy, on-time, and quality metrics |

## 26. Connected Planning Data Model

| Table / Field | Description |
|---|---|
| `planning_assumptions` | Shared planning assumptions library with values, effective dates, and source domain |
| `planning_models` | Planning model definitions by domain (financial, workforce, supply chain) with drivers and parameters |
| `planning_scenarios` | Planning scenario instances with versions, assumptions, and simulation results |
| `planning_drivers` | Cross-domain planning drivers with linkage to affected models and impact weights |
| `planning_model_versions` | Planning model version history with snapshots, baseline, and variance analysis |

## 27. Intelligent Process Automation Data Model

| Table / Field | Description |
|---|---|
| `ipa_cognitive_bots` | Cognitive bot definitions with type, ML model references, and capability profile |
| `ipa_bot_learning_events` | Bot self-learning events with feedback type, model adjustment, and improvement metrics |
| `ipa_process_discoveries` | Discovered automation opportunities with process signature, frequency, and automation potential |
| `ipa_orchestration_flows` | Multi-bot orchestration flow definitions with steps, dependencies, and shared state |
| `ipa_exception_patterns` | Classified exception patterns with auto-resolution rules and escalation thresholds |

## 28. MDM Golden Record Data Model

*Database: `integration_db`*

| Table / Field | Description |
|---|---|
| `mdm_golden_records` | Golden record master entries with `entity_type` (customer, supplier, product, etc.), `source_count`, `reconciliation_status` (auto_merged, pending_review, steward_approved, rejected), `steward_id`, `created_at`, `updated_at` |
| `mdm_source_records` | Source system record linkage with `golden_record_id`, `source_system`, `source_id`, `confidence_score` (0.00–1.00), `match_algorithm`, `match_details` (JSONB), `linked_at` |
| `mdm_stewardship_tasks` | Data steward review tasks with `golden_record_id`, `task_type` (merge_review, conflict_resolution, data_enrichment), `status` (open, in_progress, resolved, escalated), `assigned_to`, `resolution_notes`, `resolved_at`, `created_at` |

## 29. CDP Unified Profile Data Model

*Database: `crm_db`*

| Table / Field | Description |
|---|---|
| `cdp_unified_profiles` | Unified customer profiles with `identity_graph` (JSONB identity resolution graph linking identifiers across touchpoints), `primary_email`, `primary_phone`, `demographics` (JSONB), `first_seen_at`, `last_seen_at`, `profile_score` |
| `cdp_segments` | Customer segment definitions with `segment_type` (rule_based, ml_based), `membership_criteria` (JSONB rules or ML model reference), `member_count`, `refresh_frequency`, `last_computed_at`, `status` (active, draft, archived) |
| `cdp_journeys` | Customer journey definitions with `trigger_type`, `trigger_config` (JSONB), `steps` (JSONB ordered step definitions with channels, conditions, and wait periods), `conversion_goal`, `status` (active, paused, completed), `entry_count`, `created_at` |

## 30. IoT Telemetry Data Model

*Database: `manufacturing_db`*

| Table / Field | Description |
|---|---|
| `iot_telemetry_events` | Time-series telemetry events partitioned by month (`PARTITION BY RANGE (event_ts)`), with `device_id`, `metric_name`, `metric_value` (DOUBLE), `unit`, `quality` (good, uncertain, bad), `event_ts`, `ingested_at` |
| `iot_device_state` | Latest device state snapshots with `device_id` (PK), `state` (online, offline, maintenance, error), `last_telemetry_ts`, `battery_level`, `signal_strength`, `health_score`, `firmware_version`, `updated_at` |

## 31. Process Mining Activity Log Data Model

*Database: `report_db`*

| Table / Field | Description |
|---|---|
| `process_activity_log` | Business process activity records for process mining with `case_id`, `process_name`, `activity_name`, `activity_ts`, `resource_id`, `resource_type` (user, system, bot), `duration_ms`, `status` (started, completed, failed), `attributes` (JSONB), `tenant_id` |

## 32. RPA Bot Execution Data Model

*Database: `platform_db`*

| Table / Field | Description |
|---|---|
| `rpa_bot_executions` | RPA bot execution records with `bot_id`, `execution_id` (idempotency key), `status` (running, succeeded, failed, timeout), `started_at`, `completed_at`, `duration_ms`, `trigger_type` (scheduled, event, manual), `input_parameters` (JSONB), `output_data` (JSONB), `error_code`, `error_message`, `retry_count`, `tenant_id` |

## 33. Collaboration Channel Data Model

*Database: `platform_db`*

| Table / Field | Description |
|---|---|
| `collaboration_channels` | Team channels with `channel_type` (public, private, direct), `visibility`, `membership` (JSONB member list with roles), `linked_business_context_type` (order, invoice, project, case, etc.), `linked_business_context_id`, `description`, `created_by`, `archived_at`, `tenant_id` |
| `collaboration_messages` | Channel messages with `channel_id`, `parent_message_id` (for thread structure), `author_id`, `content_type` (text, markdown, rich_text, file_reference), `content` (TEXT), `mentions` (JSONB), `reactions` (JSONB), `edited_at`, `deleted_at`, `created_at`, `tenant_id` |

## 34. Loyalty Program Data Model

See §20 above.

## 35. Tax Assessment Data Model

See §8 above.

## 36. Compliance Control Mapping Data Model

See §21 above.

## 37. Contact Center Interaction Data Model

See §15 above.

## 38. EHS Incident Data Model

See §12 above.

---

*See [Data Architecture Overview](overview.md) for database patterns and schema elements.*
*See [Caching Strategy](caching.md) for cache architecture and policies.*
*See [Data Warehouse](warehouse.md) for analytical schema design.*
*See [Data Lake](data-lake.md) for raw and curated data zones.*
