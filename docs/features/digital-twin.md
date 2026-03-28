# Digital Twin

Virtual representation of physical assets and production processes with real-time state synchronization, managed jointly by Manufacturing Service (asset twins, production simulation) and Platform Service (IoT telemetry ingestion).

| Aspect | Implementation |
|--------|---------------|
| Asset Digital Twins | Virtual representation of machines, production lines, and facilities with real-time state sync from IoT telemetry (temperature, vibration, throughput, power consumption) |
| Simulation Engine | What-if simulation for production scheduling, capacity planning, and maintenance impact analysis with configurable simulation parameters and result comparison |
| Predictive Maintenance | ML-based failure prediction from telemetry patterns (vibration analysis, thermal signatures, degradation curves) with remaining useful life (RUL) estimation |
| Real-Time Monitoring | Live dashboard with asset health indicators, anomaly detection alerts, and configurable threshold breach notifications |
| Twin Analytics | Performance comparison across twin instances, degradation trend analysis, mean time between failures (MTBF) tracking, and asset utilization benchmarking |
| Model Management | Digital twin model versioning with calibration parameters, physics-based and data-driven model support, and model accuracy scoring |
| State Synchronization | Bi-directional state sync between physical asset and twin; telemetry ingestion via MQTT/HTTP streaming with configurable sampling rates |
| Historical Replay | Replay of historical twin states for root cause analysis, incident investigation, and process improvement identification |

### Twin Architecture Layers

| Layer | Function |
|-------|----------|
| Physical Layer | IoT sensors and PLCs on equipment generating telemetry streams |
| Data Ingestion | MQTT broker / HTTP streaming via Platform Service with edge pre-processing |
| State Model | Real-time twin state maintained in Manufacturing Service with event-sourced update log |
| Analytics Layer | Predictive models (Report Service ML) and simulation engine running against twin state |
| Visualization | Live dashboards, 3D asset views, and alert consoles via UI layer |

### Predictive Maintenance Pipeline

```
Telemetry Stream → Feature Extraction → ML Model Inference
  → Anomaly Score → Alert Generation (if above threshold)
  → RUL Estimation → Maintenance Work Order Suggestion
  → Feedback Loop (actual failure data → model retraining)
```

### Integration Points

| Integration | Description |
|-------------|-------------|
| Manufacturing — EAM | Predictive maintenance alerts create maintenance work orders in Enterprise Asset Management |
| Manufacturing — Production Planning | Simulation results feed capacity planning and what-if scheduling scenarios |
| Platform — IoT | Telemetry ingestion pipeline from device registry through MQTT broker to twin state model |
| Report Service | ML model training and hosting for predictive maintenance, anomaly detection, and RUL estimation |
| Commerce — Order Processing | Asset twin availability feeds ATP/CTP calculations for capacity-aware promise dates |

*See [IoT Integration](iot.md) for device management and telemetry ingestion. See [Digital Thread](digital-thread.md) for product-level traceability. See [Manufacturing Service](../services/manufacturing.md) for EAM and production modules. See [Architecture Overview](../architecture/overview.md) for system context.*
