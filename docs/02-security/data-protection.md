# Data Protection

## 8. Data Protection

### 8.1 Encryption at Rest

| Data Type | Encryption Method |
|-----------|------------------|
| Database (PostgreSQL) | AES-256 via pgcrypto extension for PII columns; TDE for full-disk encryption |
| File Storage (MinIO) | Server-side encryption (SSE-S3) with AES-256 |
| Backups | AES-256 encrypted before writing to object storage |
| Redis | TLS in transit; persistence files encrypted at rest |
| Secrets | Kubernetes secrets (encrypted at rest via etcd encryption configuration) |
| Data Lake | SSE-S3 encryption on all zones (raw, curated, analytics) |

### 8.2 PII Handling

| Requirement | Implementation |
|-------------|---------------|
| Classification | Data classified as Public, Internal, Confidential, Restricted |
| Masking in logs | PII fields automatically redacted in structured logs (e.g., email → `j***@company.com`) |
| Masking in API responses | Restricted fields masked based on caller permissions |
| Right to access (GDPR) | API endpoint for data subject to retrieve all their data |
| Right to erasure (GDPR) | Hard delete with cascading; audit log retained with anonymized references |
| Data retention | Configurable per data category; automated purge jobs |
| Cross-border transfer | Data residency controls per tenant (region pinning) |
| Data subsetting | PII-compliant data subsets for non-production environments |

### 8.3 Key Management

| Aspect | Implementation |
|--------|---------------|
| Key Storage | HashiCorp Vault or cloud KMS (AWS KMS, Azure Key Vault — configurable) |
| Key Rotation | Automated rotation every 90 days for encryption keys |
| Key Hierarchy | Master key → Data encryption key → Per-table key |
| Access | Keys accessed via service accounts with audit logging |
| Key Backup | Split-key shamir backup for master key, stored in separate physical locations |

---

- [Security Architecture](./overview.md)
- [Authorization (RBAC + ABAC)](./authorization.md)
- [Governance, Risk & Compliance (GRC)](./grc.md)
- [Threat Model & Incident Response](./threat-model.md)
