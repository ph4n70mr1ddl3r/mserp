# Adaptive Intelligence

ML-powered recommendations and intelligent suggestions embedded into every business workflow across all MSERP domains, managed by the Report Service (inference) and Platform Service (model lifecycle).

## Overview

Adaptive Intelligence brings ML-powered decision support directly into business workflows across all MSERP domains. Rather than a standalone analytics tool, it surfaces context-aware recommendations, predictions, and smart suggestions within the applications users already work in — lead scores in CRM, reorder suggestions in procurement, attrition risk alerts in HCM, payment delay predictions in Finance, and next-best-action recommendations in customer service. The system supports multiple recommendation categories, provides explainability for every prediction, includes A/B testing for recommendation effectiveness, and captures user feedback to continuously improve models.

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **ML-Powered Recommendation Categories** | Report Service (ONNX) | Six primary categories: lead scoring, next-best-action, churn prediction, cross-sell/up-sell, deal risk prediction, and demand forecasting — each with specialized models and features |
| **Integration Architecture** | Report Service + All Business Services | Recommendations surfaced via API responses (`X-Recommendations` header), WebSocket push, and embedded UI widgets; business services call Recommendation API with context and receive scored suggestions |
| **Model Training Pipeline** | Platform Service + Report Service | End-to-end pipeline: data extraction → feature engineering → training → validation → registry → deployment → monitoring → retraining; supports both Candle (Rust-native) and Python bridge for complex models |
| **A/B Testing for Recommendations** | Report Service | Controlled experiments comparing recommendation strategies; statistical significance testing; automated winner selection based on business metrics (conversion, revenue, engagement) |
| **Explainability Features** | Report Service | SHAP/LIME-based explanations for every prediction; human-readable reason codes; feature importance rankings; "Why this recommendation?" UI component |
| **Confidence Scoring** | Report Service (ONNX) | Every recommendation includes confidence score (0-100) and uncertainty estimate; low-confidence recommendations flagged for human review; confidence thresholds configurable per use case |
| **Recommendation Feedback Loop** | Platform Service | User feedback (accepted, rejected, ignored, modified) captured and fed back to model retraining; implicit feedback (click-through, conversion) tracked; feedback analytics for model owners |

## Recommendation Categories

| Category | Domain | Model Type | Input Features | Output | Example |
|----------|--------|-----------|----------------|--------|---------|
| **Lead Scoring** | CRM | Classification | Demographics, behavior, engagement, firmographics | Score 0-100, tier (hot/warm/cold) | Score new leads for sales prioritization |
| **Next-Best-Action** | CRM / CX | Multi-class classification | Customer history, segment, lifecycle stage, channel | Ranked action list with probabilities | "Call customer about renewal" (87% confidence) |
| **Churn Prediction** | CRM / Commerce | Binary classification | Usage patterns, support tickets, payment history, engagement | Churn probability 0-100, risk factors | "Customer X has 78% churn risk — last login 30 days ago" |
| **Cross-Sell / Up-Sell** | Commerce | Collaborative filtering + classification | Purchase history, segment, browsing, similar customers | Product recommendations with expected value | "Customers like this also bought Y ($2,300 expected revenue)" |
| **Deal Risk** | CRM | Classification | Deal characteristics, stage duration, competitor presence, engagement | Risk score 0-100, risk factors | "Deal at 65% risk — no stakeholder engagement in 14 days" |
| **Demand Forecasting** | Supply Chain | Time-series (LSTM/Prophet) | Historical demand, seasonality, promotions, external factors | Forecast with confidence intervals | "Expected demand for SKU-X: 1,200 units (±150, 95% CI)" |
| **Payment Delay** | Finance | Classification | Customer payment history, invoice amount, industry, seasonality | Delay probability, expected days | "Invoice has 60% chance of >30 day delay — escalate" |
| **Attrition Risk** | HCM | Classification | Performance, tenure, compensation, engagement, market conditions | Attrition probability, retention actions | "Employee at 72% flight risk — compensation below market" |
| **Reorder Suggestion** | Supply Chain | Classification + time-series | Inventory levels, lead time, demand forecast, safety stock | Reorder recommendation with quantity | "Reorder SKU-Y: 500 units, stockout risk in 12 days" |
| **Anomaly Detection** | All domains | Isolation forest / autoencoder | Transaction patterns, user behavior, system metrics | Anomaly score, deviation details | "Unusual GL balance detected — 3.2σ from historical mean" |

## Integration Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                   Business Services Layer                        │
│                                                                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │ Finance  │  │ Commerce │  │   HCM    │  │   CRM    │  ...   │
│  │ Service  │  │ Service  │  │ Service  │  │ Service  │       │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘       │
│       │              │             │             │               │
│       │  Context:    │  Context:   │  Context:   │  Context:    │
│       │  customer,   │  product,   │  employee,  │  lead,       │
│       │  invoice,    │  order,     │  team,      │  deal,       │
│       │  payment     │  cart       │  history    │  activity    │
│       │              │             │             │               │
│       └──────────────┴─────────────┴─────────────┘               │
│                            │                                      │
│                            ▼                                      │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                Recommendation API                         │   │
│  │  POST /api/v1/recommendations                            │   │
│  │  {context, category, max_results, min_confidence}        │   │
│  │                                                           │   │
│  │  Response:                                                │   │
│  │  {recommendations: [{id, type, score, confidence,        │   │
│  │    explanation, action, metadata}]}                       │   │
│  └──────────────────────────┬───────────────────────────────┘   │
│                              │                                   │
│  ┌──────────────────────────▼───────────────────────────────┐   │
│  │              Recommendation Engine                        │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐    │   │
│  │  │ Feature  │ │ Model    │ │ Ensemble │ │ Explain  │    │   │
│  │  │ Extract  │ │ Router   │ │ Scorer   │ │ Module   │    │   │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘    │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐    │   │
│  │  │ A/B Test │ │ Feedback │ │ Confid.  │ │ Cache    │    │   │
│  │  │ Manager  │ │ Capture  │ │ Scorer   │ │ Layer    │    │   │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘    │   │
│  └──────────────────────────────────────────────────────────┘   │
│                              │                                   │
│  ┌──────────────────────────▼───────────────────────────────┐   │
│  │              Model Training Pipeline                      │   │
│  │  Data Extract → Feature Eng → Train → Validate → Deploy  │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

## A/B Testing Framework

| Stage | Description | Duration |
|-------|-------------|----------|
| Hypothesis | Define metric improvement goal (e.g., "New lead model increases conversion by 5%") | 1 day |
| Experiment Setup | Define control (current model) and treatment (new model); set traffic split | 1 day |
| Execution | Route traffic to models based on split; collect outcomes | 2-4 weeks |
| Analysis | Statistical significance test (p < 0.05); measure impact on business metric | 1 day |
| Decision | Promote winner, iterate, or rollback based on results | 1 day |

| Metric | Description | Target |
|--------|-------------|--------|
| Recommendation Acceptance Rate | % of recommendations accepted by users | > 30% |
| Click-Through Rate | % of recommendations that users engage with | > 15% |
| Conversion Lift | % improvement in conversion with vs. without recommendations | > 5% |
| Revenue Impact | Incremental revenue attributed to recommendations | Track and report |
| Model Accuracy | Precision/recall of prediction models | > 85% precision |
| Recommendation Generation Latency | Time to generate recommendation (model inference only) | < 200ms |
| End-to-End ML Inference Latency | Full inference including feature retrieval, model scoring, and response assembly | < 500ms (see [NFR](../10-planning/nfr.md)) |

## Explainability Framework

| Component | Description | User-Facing Display |
|-----------|-------------|-------------------|
| Feature Importance | Top N features driving the prediction | "Top factors: payment history (32%), industry (24%), amount (18%)" |
| SHAP Values | Per-feature contribution to score | Bar chart with positive/negative contributions |
| Reason Codes | Human-readable reason categories | "At risk due to: late payments, reduced order volume, support tickets" |
| Counterfactuals | What would change the prediction | "Score would improve to 80% if payment terms were Net-30" |
| Similar Cases | Historical cases with similar profiles | "3 similar deals: 2 won (67%), avg deal size $45K" |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Finance Service | Finance Service | API + Event | Payment delay prediction, revenue forecasting, spend classification |
| Commerce Service | Commerce Service | API + Event | Demand forecasting, price optimization, cross-sell/up-sell, churn prediction |
| HCM Service | HCM Service | API + Event | Attrition risk, performance prediction, workforce planning |
| CRM Service | CRM Service | API + Event | Lead scoring, next-best-action, deal risk, customer lifetime value |
| Manufacturing Service | Manufacturing Service | API + Event | Predictive maintenance, quality prediction, yield optimization |
| Project Service | Project Service | API + Event | Project risk, timeline estimation, resource optimization |
| Report Service | Report Service | Internal | Model hosting, inference, A/B testing, explainability |
| Platform Service | Platform Service | Internal | Model lifecycle, feedback capture, feature store |
| Digital Assistant | Platform Service | API | AI suggestions surfaced through conversational interface |
| Audit Trail | Platform Service | Event-driven | Recommendation acceptance/rejection logged |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `intelligence.recommendation.generated` | `{rec_id, category, context, score, confidence, model_version}` | Recommendation generated | Feature Store, A/B Testing |
| `intelligence.recommendation.accepted` | `{rec_id, user_id, action_taken, outcome_metric}` | User accepts recommendation | Feedback Pipeline, Analytics |
| `intelligence.recommendation.rejected` | `{rec_id, user_id, rejection_reason, alternative_action}` | User rejects recommendation | Feedback Pipeline, Analytics |
| `intelligence.prediction.generated` | `{pred_id, category, entity_id, prediction, confidence, features}` | ML prediction generated | Feature Store, Audit Trail |
| `intelligence.anomaly.detected` | `{anomaly_id, domain, entity_id, anomaly_score, details}` | Anomaly detected in data pattern | Notification Service, Dashboard |
| `intelligence.model.deployed` | `{model_id, version, category, accuracy_metrics, deployed_by}` | New model version deployed | Audit Trail, Monitoring |
| `intelligence.experiment.completed` | `{experiment_id, winner, lift, statistical_significance}` | A/B test reaches significance | Model Owners, Analytics |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| All business events | All Services | Extract features for recommendation context |
| `intelligence.recommendation.accepted` | Report Service | Feed positive signal to retraining pipeline |
| `intelligence.recommendation.rejected` | Report Service | Feed negative signal to retraining pipeline |
| `finance.transaction.posted` | Finance Service | Update financial features for prediction models |
| `commerce.order.completed` | Commerce Service | Update purchase history for recommendation models |
| `crm.lead.created` | CRM Service | Trigger lead scoring model |
| `crm.deal.stage-changed` | CRM Service | Update deal risk model features |
| `hcm.employee.terminate` | HCM Service | Update attrition model training data |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `intelligence_recommendation` | `id, category, context_type, context_id, recommendation, score, confidence, model_version` | Has many `intelligence_feedback` |
| `intelligence_feedback` | `id, recommendation_id, user_id, feedback_type, reason, outcome_metric, created_at` | Belongs to `intelligence_recommendation` |
| `intelligence_prediction` | `id, category, entity_type, entity_id, prediction, confidence, features, model_version` | Audit record |
| `intelligence_model` | `id, name, category, version, accuracy_metrics, training_date, deployed_at, status` | Has many `intelligence_model_version` |
| `intelligence_model_version` | `id, model_id, version, artifact_ref, metrics, validation_results` | Belongs to `intelligence_model` |
| `intelligence_experiment` | `id, name, category, control_model, treatment_model, traffic_split, status, start_date, end_date` | Has many `intelligence_experiment_result` |
| `intelligence_experiment_result` | `id, experiment_id, model_version, impressions, conversions, metric_value` | Belongs to `intelligence_experiment` |
| `intelligence_feature` | `id, entity_type, entity_id, feature_name, feature_value, computed_at` | Feature store record |

## Cross-References

- [Intelligent Close](intelligent-close.md) — AI-driven financial close automation
- [Intelligent Document Processing](idp.md) — AI-powered document classification and extraction
- [Augmented Analytics](augmented-analytics.md) — Natural language query and auto-insights
- [Process Mining](process-mining.md) — Process discovery and optimization
- [Architecture Overview](../01-architecture/overview.md) — System context
- [Report Service](../06-services/report.md) — ML model serving and analytics
- [NFR: Performance](../10-planning/nfr.md) — ML inference latency < 500ms
