# IoT Integration

IoT device management and telemetry ingestion managed jointly by Platform Service (device registry) and Manufacturing Service (industrial IoT).

| Aspect | Implementation |
|--------|---------------|
| Device Registry | IoT device registration, certification, certificate-based authentication |
| Telemetry Ingestion | High-throughput telemetry ingestion via MQTT broker or HTTP streaming |
| Alert Rules | Configurable threshold and anomaly-based alerting on telemetry data |
| Data Routing | Route telemetry to appropriate services (manufacturing for production data, logistics for tracking) |
| Edge Processing | Optional edge computing for latency-sensitive processing before cloud ingestion |
| Digital Twin | Virtual representation of physical assets with real-time state sync and predictive models |

*See [Digital Thread](digital-thread.md) for product traceability. See [Architecture Overview](../architecture/overview.md) for system context.*
