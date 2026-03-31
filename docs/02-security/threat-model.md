# Threat Model & Incident Response

## 1. Threat Model

| Threat Category | Mitigation |
|----------------|------------|
| Brute-force login | Account lockout after 5 failures (15-minute cooldown), IP-based rate limiting |
| Token theft | Short-lived access tokens (15 min), refresh rotation, Redis blocklist |
| SQL injection | Parameterized queries (SeaORM), input validation |
| XSS | Output encoding, Content-Security-Policy headers |
| CSRF | Same-site cookies, CSRF tokens for state-changing operations |
| DDoS | API Gateway rate limiting, CDN-based DDoS protection |
| Insider threat | Audit logging, separation of duties, privileged access management, SoD enforcement |
| Supply chain | Container image scanning (Trivy), dependency auditing (cargo audit), image signing (Cosign), SBOM |
| Data exfiltration | Encryption at rest, network policies, egress filtering, data masking |
| Privilege escalation | Step-up authentication, SoD conflict detection, least-privilege defaults |
| API abuse | Rate limiting per tenant/user/endpoint, API key rotation, request validation |
| Model poisoning | ML model integrity verification, signed models, input sanitization for AI endpoints |
| AI/ML model bias and fairness | Bias testing in model training, fairness metrics, regular audit, diverse training data, output monitoring |
| Trade compliance violation | Automated denied party screening on all transactions, real-time list updates, screening override audit trail, export control classification enforcement |
| Trade compliance evasion | Persistent monitoring and logging of screening attempts, sanctions re-screening on updated lists |
| Data residency violation | Connection routing enforcement, RLS policies, cross-border transfer audit logging |
| Subscription fraud | Usage anomaly detection, automated billing reconciliation, payment verification |
| IoT device compromise | Device certificate authentication, firmware integrity verification, network segmentation for IoT devices, telemetry anomaly detection |
| RPA bot hijacking | Bot execution audit trail, credential vaulting for bot credentials, execution sandboxing, anomaly detection on bot behavior |
| Data exfiltration via content export | DLP policy enforcement, export auditing, content classification, watermarking |
| Privacy violation | Consent enforcement, purpose limitation, data minimization, automated DPIA, processing register audit |
| Content tampering | Document version control, immutable audit trail, digital signatures, integrity checksums |
| IoT firmware tampering | Firmware integrity verification, secure boot, certificate-based device authentication |
| Lease data tampering | Immutable lease audit trail, digital signatures on lease contracts, role-based access to lease modification |
| Grant fraud | Segregation of duties for grant approval, audit trail for all grant disbursements, compliance monitoring alerts |
| JV partner data manipulation | Partner access restricted to own venture data, allocation audit trail, approval workflow for JV modifications |
| IDP data poisoning | Input document validation, extraction confidence thresholds, human review for low-confidence extractions |
| Collection bypass | Collection strategy enforcement rules, approval workflow for write-offs, audit trail for all collection actions |
| Blockchain smart contract vulnerability | Smart contract code auditing, formal verification, multi-signature controls, testnet deployment |
| Contact center social engineering | Agent verification protocols, caller authentication, call recording, anomaly detection on access patterns |
| Loyalty points fraud | Points laundering detection, duplicate account prevention, velocity checks, geographic anomaly detection |
| Tax evasion via misclassification | Automated tax determination validation, cross-jurisdiction consistency checks, audit trail for all tax overrides |
| MRO spare parts theft | Spare parts inventory tracking, issuance authorization, cycle counting, anomaly detection on consumption patterns |
| ITSM privilege escalation | ITSM role-based access, change approval workflows, configuration item access logging, admin action audit trail |
| Price manipulation | Price change audit trail, approval workflows for manual overrides, ML-based price anomaly detection |
| Compliance hub data tampering | Immutable compliance records, digital signatures on compliance evidence, role-based access with SoD |

## 2. Security Incident Response

| Phase | Actions | SLA |
|-------|---------|-----|
| Detection | Automated alerting (SIEM), anomaly detection, user reports | < 15 min to alert |
| Triage | Severity assessment, scope determination, incident ticket creation | < 30 min |
| Containment | Isolate affected systems, revoke compromised credentials, block IPs | < 1 hour |
| Eradication | Remove threat, patch vulnerability, rotate all affected secrets | < 4 hours |
| Recovery | Restore from backup if needed, verify system integrity, resume operations | < 8 hours |
| Post-mortem | Root cause analysis, lessons learned, preventive measures, documentation | < 48 hours |

> **Trade Compliance Incident Procedure:** Trade compliance violations follow the standard incident response flow with additional steps: (1) halt affected orders/shipments immediately during Containment, (2) review screening logs and denied party match details during Eradication, (3) update denied party lists and screening rules as part of Recovery, (4) document regulatory notification requirements in Post-mortem. SLA: < 1 hour to containment (aligned with general incident response SLA).

## 3. Privacy Impact Assessment (PIA)

A PIA MUST be conducted before:
- Introducing new data processing activities involving PII
- Adding new data sharing arrangements with third parties
- Deploying new AI/ML models that process personal data
- Expanding data collection to new categories of personal data
- Entering new jurisdictions with different privacy requirements
- Implementing trade compliance screening that processes customer, supplier, or entity data against government watchlists (includes cross-border shipment screening and denied party list matching)
- Deploying IoT data collection that captures personally identifiable telemetry data
- Implementing RPA bots that access or process personal data
- Enabling Customer Data Platform (CDP) profiling or segmentation that uses personal data

PIA outcomes are recorded in the GRC module and linked to the data catalog.

---

*See [API Design](../05-api/standards.md) for rate limiting and idempotency details.*
*See [Data Architecture](../03-data/overview.md) for row-level security and encryption at rest.*

- [Security Architecture](./overview.md)
- [Authorization (RBAC + ABAC)](./authorization.md)
- [Data Protection](./data-protection.md)
- [Governance, Risk & Compliance (GRC)](./grc.md)
