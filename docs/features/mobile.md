# Mobile Platform

The mobile platform provides native mobile access to all MSERP capabilities.

| Aspect | Implementation |
|--------|---------------|
| Framework | React Native for iOS and Android |
| Offline support | Local SQLite cache with background sync on connectivity |
| Push notifications | FCM (Android), APNs (iOS) via Platform Service |
| Biometric auth | Face ID, Touch ID, fingerprint for mobile MFA |
| Mobile-specific APIs | Optimized endpoints with reduced payload sizes for mobile clients |

*See [Architecture Overview](../architecture/overview.md) for system context.*
