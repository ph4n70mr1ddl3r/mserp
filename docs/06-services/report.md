# Report Service

| Aspect | Details |
|--------|---------|
| Port | 8014 |
| Database | `report_db` (materialized views, pre-aggregated data); reads from other services' event streams |
| Responsibilities | Analytics, reporting, dashboards, BI, AI-driven insights, ESG, carbon accounting, embedded ML, narrative reporting, process mining, corporate performance management, augmented analytics |

> **Analytics Ownership:** Report Service owns the analytics platform. Other services define analytics widgets rendered by Report Service. No service should build independent analytics infrastructure.

## Modules

| Module | Description |
|--------|-------------|
| Report Builder | Drag-and-drop report designer, custom SQL queries, parameterized reports |
| Scheduled Reports | Automated report generation and delivery (email, file share), subscription management |
| Dashboards | Real-time dashboards, widget library, drill-down/drill-through, personal and shared dashboards |
| KPIs | KPI definition, target setting, trend analysis, traffic-light alerts |
| Data Export | CSV, Excel, PDF, Parquet export; scheduled exports; API-driven data extraction |
| BI Integration | ODBC/JDBC bridge, embedded analytics, data warehouse sync |
| AI Insights | Anomaly detection (spend, inventory, revenue), demand forecasting, intelligent recommendations |
| Data Warehouse | Periodic ETL from event streams (via MinIO data lake) to DuckDB-based embedded warehouse; star schema with fact/dimension tables. ETL reads from event streams, NOT from operational databases, respecting the database-per-service principle. |
| ESG / Sustainability Reporting | Emissions tracking dashboards (Scope 1, 2, 3), ESG scorecards, regulatory report generation (GRI, SASB, TCFD, EU Taxonomy) |
| Carbon Accounting | Carbon footprint per product/service, emissions intensity metrics, carbon offset tracking, science-based target progress |
| Embedded ML/AI Platform | Model training and deployment pipeline, feature store, model registry, A/B testing for ML models |
| Predictive Planning | Demand forecasting models, revenue prediction, resource utilization forecasting, what-if scenario modeling |
| Data Lake Integration | Raw data ingestion to data lake, curated data layers, schema-on-read for exploratory analytics |

## Narrative Reporting

| Module | Description |
|--------|-------------|
| Management Reports | Structured management reports with financial and operational data |
| Commentary & Annotations | Collaborative annotations, commentary workflows, and approval chains on report sections |
| Report Packages | Collection of reports organized for board meetings, investor communications |
| Versioning | Report version control with audit trail of commentary changes |
| Distribution | Automated distribution of report packages to stakeholders via email and portal |
| XBRL Tagging | Automated XBRL tagging for regulatory filing requirements |

## AI/ML Capabilities

| Capability | Description |
|-----------|-------------|
| Anomaly Detection | Statistical detection of outliers in financial transactions, inventory levels, and sales patterns |
| Demand Forecasting | Time-series forecasting using moving averages, exponential smoothing, and ML models; fed back to Commerce for reorder suggestions |
| Spend Analysis | Automatic categorization of expenses, vendor spend concentration, contract compliance |
| Churn Prediction | Customer behavior analysis identifying at-risk accounts (CRM integration) |
| Cash Flow Forecasting | Predictive cash flow based on historical patterns, outstanding receivables, and payables |
| Sentiment Analysis | NLP-based analysis of customer feedback, support tickets, and survey responses |
| Fraud Detection | Pattern-based fraud detection in financial transactions, expense reports, and procurement |
| Yield Optimization | ML-driven production yield optimization for manufacturing |

## Process Mining & Optimization

| Module | Description |
|--------|-------------|
| Process Discovery | Automated reconstruction of business processes from event logs and audit trails |
| Conformance Checking | Compare actual process execution against defined BPMN workflows |
| Bottleneck Analysis | Identify process steps with highest wait times, lowest throughput, and resource constraints |
| Simulation | What-if simulation of process changes with predicted throughput and cost impact |
| Root Cause Analysis | Drill-down from process KPI deviations to specific events, resources, and transactions |
| Continuous Monitoring | Real-time process performance dashboards with configurable deviation alerts |

## Corporate Performance Management

| Module | Description |
|--------|-------------|
| Executive Dashboards | Role-based executive dashboards with KPI summaries and drill-down to operational data |
| Strategy Maps | Visual strategy maps linking corporate objectives, KPIs, and strategic initiatives |
| OKRs | Objective and Key Results tracking with progress visualization and alignment reporting |
| Balanced Scorecard | Four-perspective scorecard: financial, customer, internal process, learning & growth |
| Initiative Tracking | Strategic initiative registration, milestone tracking, budget allocation, and status reporting |

## Embedded Analytics

| Module | Description |
|--------|-------------|
| In-Context Analytics | Analytics embedded directly in business application screens for contextual decision-making |
| Actionable Insights | One-click drill-through from analytical findings to operational transactions for immediate action |
| Embedded Dashboards | Lightweight dashboard widgets embeddable in any service UI via iframe or web component |

## Predictive Analytics

| Module | Description |
|--------|-------------|
| Demand Prediction | Predictive models for product demand by region, channel, and customer segment |
| Revenue Prediction | Revenue forecasting models incorporating pipeline, seasonality, and macroeconomic indicators |
| Resource Utilization | Predictive resource utilization modeling for workforce, equipment, and capacity planning |
| Customer Behavior | Predictive customer behavior models for churn, lifetime value, and next-best-action |

## Self-Service BI

| Module | Description |
|--------|-------------|
| Drag-and-Drop Designer | Drag-and-drop report creation for business users without SQL knowledge |
| Visual Data Exploration | Interactive visual data exploration with chart type recommendations and auto-formatting |
| Data Preparation | Self-service data preparation with join suggestions, cleansing, and transformation |
| Scheduled Delivery | Self-service report scheduling with email, Slack, and portal delivery options |

## Self-Service ML Studio

| Module | Description |
|--------|-------------|
| Project Management | ML project organization with experiment grouping, collaboration, and version control |
| AutoML | Automated machine learning pipeline with feature engineering, model selection, and hyperparameter tuning |
| Experiment Tracking | Experiment logging with parameter capture, metric tracking, and run comparison |
| Model Comparison | Side-by-side model comparison with statistical significance testing and performance visualization |
| Explainability | Model explainability with feature importance, SHAP values, and prediction-level explanations |
| Deployment | One-click model deployment with A/B testing, traffic splitting, and rollback capabilities |

## Augmented Analytics

| Module | Description |
|--------|-------------|
| Natural Language Queries | NLQ-to-SQL engine for ad-hoc data exploration without SQL knowledge |
| Auto-Insights | Automated pattern discovery, trend detection, and anomaly flagging |
| Smart Discovery | Automated correlation analysis across dimensions and measures |
| Explainable AI | Human-readable explanations for analytical findings and ML predictions |
| Contextual Narratives | Auto-generated narrative summaries for dashboards and reports |

## Database Tables

> All tables include standard columns per [SPEC.md §9.1](../SPEC.md).

| Table | Additional Columns |
|-------|-------------------|
| `report_templates` | `name VARCHAR(255)`, `type VARCHAR(30)`, `definition JSONB`, `parameters JSONB`, `data_source VARCHAR(50)`, `refresh_interval INT`, `is_system BOOLEAN`, `status VARCHAR(20)` |
| `dashboards` | `name VARCHAR(255)`, `description TEXT`, `owner_id UUID`, `is_shared BOOLEAN`, `layout JSONB`, `filters JSONB`, `auto_refresh_interval INT` |
| `dashboard_widgets` | `dashboard_id UUID`, `title VARCHAR(255)`, `type VARCHAR(30)`, `data_source VARCHAR(50)`, `query JSONB`, `position JSONB`, `filters JSONB`, `refresh_interval INT` |
| `kpi_definitions` | `name VARCHAR(255)`, `description TEXT`, `measure VARCHAR(100)`, `target DECIMAL`, `warning_threshold DECIMAL`, `critical_threshold DECIMAL`, `unit VARCHAR(20)`, `direction VARCHAR(10)`, `data_source VARCHAR(50)`, `calculation JSONB` |
| `esg_emissions` | `scope INT`, `category VARCHAR(50)`, `source_entity_id UUID`, `source_entity_type VARCHAR(30)`, `co2_equivalent DECIMAL`, `unit VARCHAR(20)`, `measurement_date DATE`, `methodology VARCHAR(30)`, `verification_status VARCHAR(20)` |
| `ml_models` | `name VARCHAR(255)`, `type VARCHAR(30)`, `version VARCHAR(20)`, `status VARCHAR(20)`, `features JSONB`, `hyperparameters JSONB`, `metrics JSONB`, `training_completed_at TIMESTAMPTZ`, `deployed_at TIMESTAMPTZ` |
| `process_activity_log` | `process_id UUID`, `activity_name VARCHAR(100)`, `case_id VARCHAR(100)`, `timestamp TIMESTAMPTZ`, `resource_id UUID`, `duration_ms INT`, `attributes JSONB` |
| `planning_scenarios` | `name VARCHAR(255)`, `description TEXT`, `status VARCHAR(20)`, `planning_domains TEXT[]`, `assumptions JSONB`, `drivers JSONB`, `results JSONB`, `baseline_scenario_id UUID` |

## Events Published

| Event | Description |
|-------|-------------|
| `report.process.discovered` | Business process discovered from event log analysis |
| `report.process.conformance.checked` | Process conformance check completed |
| `report.process.bottleneck.detected` | Process bottleneck identified |
| `report.process.simulation.completed` | Process simulation completed |
| `report.cpm.okr.updated` | OKR progress updated |
| `report.cpm.initiative.status_changed` | Strategic initiative status changed |
| `report.cpm.scorecard.updated` | Balanced scorecard recalculated |
| `report.narrative.package.published` | Report package published for distribution |
| `report.narrative.commentary.added` | Commentary added to report section |
| `report.planning.scenario.created` | Connected planning scenario created |
| `report.planning.scenario.compared` | Planning scenarios compared side-by-side |
| `report.planning.driver.updated` | Planning driver value updated affecting connected models |

## Events Consumed

Inbox binding: `report.inbox` binds to the following routing keys:

| Binding Pattern | Events Consumed |
|----------------|-----------------|
| `report.#` | Self-binding for saga compensation |
| `report.ml_studio.#` | Self-binding for ML Studio saga compensation |
| `commerce.#` | All Commerce events (orders, stock, deliveries, subscriptions, loyalty, etc.) |
| `finance.#` | All Finance events (invoices, payments, journals, budgets, revenue recognition, etc.) |
| `hr.#` | All HR events (employee lifecycle, payroll, leave, reviews, etc.) |
| `manufacturing.#` | All Manufacturing events (work orders, quality, BOMs, IoT, digital twin, etc.) |
| `crm.#` | All CRM events (opportunities, campaigns, cases, CDP, surveys, etc.) |
| `project.#` | All Project events (tasks, milestones, timesheets, invoices, etc.) |
| `workflow.#` | All Workflow events (instance lifecycle, step actions, escalations, etc.) |
| `platform.audit.#` | `platform.audit.logged` |
| `platform.rpa.#` | `platform.rpa.bot.created`, `platform.rpa.bot.executed`, `platform.rpa.bot.failed` |
| `tenant.feature.#` | `tenant.feature.changed` |
| `integration.trade_compliance.#` | `integration.trade_compliance.screening.completed`, `integration.trade_compliance.screening.flagged`, `integration.trade_compliance.license.expiring` |
| `config.changed` | `config.changed` |

## See Also

- [Finance Service](finance.md)
- [Commerce Service](commerce.md)
- [Manufacturing Service](manufacturing.md)
- [Architecture Overview](../01-architecture/overview.md)
