# Mobile Platform

Comprehensive native mobile access to all MSERP capabilities, built on React Native with offline-first architecture.

| Aspect | Implementation |
|--------|---------------|
| Framework | React Native for iOS and Android with shared codebase |
| Offline Support | Local SQLite cache with background sync on connectivity; conflict resolution queue |
| Push Notifications | FCM (Android), APNs (iOS) via Platform Service with deep linking |
| Biometric Auth | Face ID, Touch ID, fingerprint for mobile MFA step-up authentication |
| Mobile-Specific APIs | Optimized endpoints with reduced payload sizes, pagination, and field selection for mobile clients |
| Barcode/RFID | Camera-based barcode and QR code scanning for inventory, receiving, and asset management |
| GPS & Geolocation | GPS-based mileage tracking, attendance check-in, delivery proof, and field service routing |
| Camera Integration | Document scanning, receipt capture, expense receipt OCR, and photo attachments |
| Offline Forms | Configurable offline forms for data collection with validation and sync-on-connect |
| Role-Based Home | Adaptive home screen based on user role (sales rep, field technician, warehouse worker, executive) |
| Push-Based Dashboards | Real-time dashboard updates via WebSocket with offline caching |
| App Security | Certificate pinning, jailbreak/root detection, app-level encryption, remote wipe capability |
| Deep Linking | Universal links for email/notification deep navigation into specific records |
| Multi-Language | Full RTL support and locale-aware formatting matching i18n configuration |
| App Config | Remote feature flags and configuration via Config Service for phased feature rollout |

*See [Field Service Management](field-service.md) for field technician mobile features. See [Internationalization](i18n.md) for localization. See [Architecture Overview](../architecture/overview.md) for system context.*
