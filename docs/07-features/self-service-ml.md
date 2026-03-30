# Self-Service ML Studio

Visual model training UI with AutoML, feature selection, model comparison, A/B deployment, and explainability dashboards for business users, managed by the Report Service.

## Overview

Self-Service ML Studio democratizes machine learning by enabling business analysts and domain experts to build, evaluate, and deploy ML models without writing code. The module provides a visual experiment builder with AutoML capabilities that automate feature engineering, algorithm selection, and hyperparameter tuning. It supports model comparison across multiple experiments, A/B deployment for production testing, explainability dashboards for model transparency, and governance controls that ensure responsible AI practices across the organization.

## Data Flow Diagram

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  Data Lake       │────▶│  Dataset Builder  │────▶│  Experiment Studio  │
│  (Training Data) │     │  ┌─────────────┐ │     │  ┌───────────────┐  │
└──────────────────┘     │  │ Feature     │ │     │  │ AutoML Engine │  │
                         │  │ Selection   │ │     │  │ (algorithm    │  │
┌──────────────────┐     │  └─────────────┘ │     │  │  search)      │  │
│  Domain Events   │────▶│  ┌─────────────┐ │     │  └───────┬───────┘  │
│  (Commerce/HR/   │     │  │ Transform   │ │     │  ┌───────▼───────┐  │
│   Finance/CRM)   │     │  │ Pipeline    │ │     │  │ Hyperparameter│  │
└──────────────────┘     │  └─────────────┘ │     │  │ Optimization  │  │
                         └──────────────────┘     │  └───────┬───────┘  │
                                                   └──────────┼──────────┘
                                                              │
                                                   ┌──────────▼──────────┐
                                                   │  Model Evaluation    │
                                                   │  ┌────────────────┐  │
                                                   │  │ Comparison     │  │
                                                   │  │ Explainability │  │
                                                   │  │ Governance     │  │
                                                   │  └───────┬────────┘  │
                                                   └──────────┼──────────┘
                                                              │
                                                   ┌──────────▼──────────┐
                                                   │  Deployment Manager  │
                                                   │  ┌────────────────┐  │
                                                   │  │ A/B Testing    │  │
                                                   │  │ Champion/      │  │
                                                   │  │ Challenger     │  │
                                                   │  └────────────────┘  │
                                                   └─────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Visual Experiment Builder** | Report Service (Rust + React) | Drag-and-drop experiment composition with data source selection, feature configuration, and training parameters |
| **AutoML Engine** | Report Service (Python + ONNX) | Automated algorithm selection across classification, regression, and forecasting with ensemble methods |
| **Feature Selection** | Report Service | Automated feature importance ranking, correlation analysis, missing value handling, and encoding strategies |
| **Model Comparison** | Report Service | Side-by-side model evaluation with configurable metrics, statistical significance testing, and recommendation |
| **A/B Deployment** | Report Service | Traffic splitting between champion and challenger models with automated promotion based on performance thresholds |
| **Explainability Dashboard** | Report Service (SHAP + LIME) | Feature importance visualization, prediction explanation, and model behavior analysis for transparency |
| **Model Governance** | Report Service + Workflow Service | Model approval workflows, bias detection, fairness metrics, and audit trail for regulatory compliance |
| **Storage** | PostgreSQL + MinIO + MLflow | Experiment metadata in PostgreSQL; datasets and model artifacts in MinIO; experiment tracking via MLflow |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Project Manager | ML project CRUD, collaboration, and access control | Report Service |
| Dataset Builder | Data source connection, feature selection, and transformation | Report Service |
| Experiment Runner | AutoML execution, training job management, and result collection | Report Service |
| Model Comparator | Multi-model evaluation, statistical testing, and ranking | Report Service |
| Deployment Manager | Model deployment, A/B traffic splitting, and promotion | Report Service |
| Explainability Engine | SHAP/LIME-based prediction explanation and feature importance | Report Service |
| Governance Manager | Approval workflows, bias detection, and compliance tracking | Report Service |
| Model Registry | Model versioning, lineage tracking, and artifact storage | Report Service |

## Supported Model Types

| Category | Algorithms | Use Cases |
|----------|-----------|-----------|
| Classification | Logistic Regression, Random Forest, XGBoost, Neural Networks | Churn prediction, fraud detection, lead scoring |
| Regression | Linear, Ridge, Lasso, XGBoost, LightGBM | Demand forecasting, price optimization, cost prediction |
| Time Series | ARIMA, Prophet, LSTM, Temporal Fusion Transformer | Revenue forecasting, inventory planning, workforce planning |
| Clustering | K-Means, DBSCAN, Hierarchical | Customer segmentation, anomaly grouping, pattern discovery |
| Recommendation | Collaborative Filtering, Content-Based, Hybrid | Product recommendations, content personalization |
| NLP (Basic) | TF-IDF, Sentiment, Text Classification | Review analysis, ticket categorization, feedback mining |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Data Lake | Report Service | Internal API | Training data access from curated data warehouse |
| Commerce Events | Commerce Service | Event-driven | Sales, order, and customer behavior data for training |
| Finance Events | Finance Service | Event-driven | Transaction and accounting data for anomaly detection |
| HR Events | HCM Service | Event-driven | Workforce data for attrition and performance models |
| CRM Events | CRM Service | Event-driven | Lead and opportunity data for scoring models |
| Manufacturing Events | Manufacturing Service | Event-driven | Production data for quality and maintenance models |
| Workflow Engine | Workflow Service | Event-driven | Model approval and deployment workflows |
| Notification Service | Platform Service | Event-driven | Training completion, deployment alerts |
| Adaptive Intelligence | Report Service | Internal API | Core ML platform infrastructure and inference serving |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `report.ml-studio.experiment.created` | `{experiment_id, project_id, dataset_id, model_type, config}` | New experiment configuration saved | Project Dashboard, Notification Service |
| `report.ml-studio.model.trained` | `{experiment_id, model_id, algorithm, metrics, training_duration, created_at}` | Model training job completes | Model Comparator, Governance Manager, Notification Service |
| `report.ml-studio.model.deployed` | `{model_id, deployment_id, deployment_type, traffic_percent, deployed_by}` | Model deployed to production endpoint | Governance Manager, A/B Monitor, Notification Service |
| `report.ml-studio.model.promoted` | `{model_id, deployment_id, promoted_from, promoted_by, metrics}` | Challenger model promoted to champion | Audit Trail, Report Service, Notification Service |
| `report.ml-studio.bias.detected` | `{model_id, metric_type, affected_group, severity, detected_at}` | Bias check threshold exceeded | Governance Manager, Notification Service, Compliance Dashboard |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `commerce.order.placed` | Commerce Service | Append to training dataset (with consent) |
| `finance.transaction.created` | Finance Service | Append to fraud detection training set |
| `hr.employee.terminated` | HCM Service | Append to attrition model dataset |
| `crm.opportunity.updated` | CRM Service | Append to lead scoring dataset |
| `manufacturing.quality.inspection-completed` | Manufacturing Service | Append to quality prediction dataset |
| `workflow.step.approved` | Workflow Service | Finalize model deployment approval |
| `config.changed` | Config Service | Reload AutoML or governance parameters |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `ml_studio_projects` | `id, tenant_id, name, description, domain, owner_id, status, created_at` | Has many `ml_studio_experiments`, `ml_studio_datasets` |
| `ml_studio_experiments` | `id, tenant_id, project_id, dataset_id, model_type, algorithm, config, status, metrics, trained_at` | Belongs to `ml_studio_projects`, `ml_studio_datasets` |
| `ml_studio_datasets` | `id, tenant_id, project_id, name, source_type, source_ref, features, row_count, version` | Belongs to `ml_studio_projects` |
| `ml_studio_comparisons` | `id, tenant_id, project_id, experiment_ids, metrics_comparison, winner_id, compared_at` | Links multiple `ml_studio_experiments` |
| `ml_studio_deployments` | `id, tenant_id, model_id, model_artifact_ref, deployment_type, traffic_percent, status, deployed_at, promoted_at` | Belongs to `ml_studio_experiments` |

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `report.ml-studio.training.max_duration_minutes` | 120 | Maximum training job duration |
| `report.ml-studio.automl.max_algorithms` | 10 | Maximum algorithms evaluated per AutoML run |
| `report.ml-studio.deployment.ab.default_traffic` | 10 | Default traffic percentage for challenger model |
| `report.ml-studio.deployment.ab.promotion_threshold` | 0.05 | Metric improvement threshold for auto-promotion |
| `report.ml-studio.governance.approval_required` | true | Require approval before production deployment |
| `report.ml-studio.governance.bias_check_enabled` | true | Enable automated bias detection on trained models |

## Cross-References

- [Adaptive Intelligence](adaptive-intelligence.md) — Core ML platform and inference serving
- [Report Service](../06-services/report.md) — Report service specification and ML infrastructure
- [Data Architecture](../03-data/overview.md) — Data lake and warehouse patterns
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Security Architecture](../02-security/overview.md) — Model access control and data privacy
- [Workflow Engine](../06-services/workflow.md) — Model approval workflows
- [NFR: Performance](../10-planning/nfr.md) — AutoML training completion within configured timeout
