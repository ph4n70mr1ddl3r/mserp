# Glossary

## Terminology

| Term | Definition |
|------|------------|
| ABAC | Attribute-Based Access Control — authorization model that evaluates attributes (user, resource, environment) to make access decisions, supplementing RBAC |
| AES-256 | Advanced Encryption Standard with 256-bit key — symmetric encryption algorithm for data at rest |
| AP | Accounts Payable — money owed by the company to suppliers |
| API | Application Programming Interface — set of protocols for building and integrating software |
| AR | Accounts Receivable — money owed to the company by customers |
| ArgoCD | GitOps continuous delivery tool for Kubernetes — syncs cluster state from Git repositories |
| Axum | Ergonomic and modular web framework for Rust — built on Tokio and Tower |
| BOM | Bill of Materials — list of components required for manufacturing a product |
| BPMN | Business Process Model and Notation — standard for business process diagrams |
| Cache-Aside | Caching pattern where the application checks the cache first; on miss, fetches from the database and populates the cache |
| CAPA | Corrective and Preventive Action — systematic approach to investigating and resolving non-conformances |
| cert-manager | Kubernetes add-on for automated TLS certificate management and rotation |
| Changelog | Immutable, append-only log recording all data mutations for compliance and auditing purposes |
| Choreography (Saga) | Saga coordination style where each service independently reacts to events — no central orchestrator |
| COA | Chart of Accounts — structured list of all financial account codes |
| Compensating Action | Undo operation in a saga step that reverses the effect of a previously completed step when a later step fails |
| Cors | Cross-Origin Resource Sharing — mechanism allowing web applications to access resources from different domains |
| CRM | Customer Relationship Management — strategy and tools for managing customer interactions and data |
| Cursor Pagination | Pagination using opaque tokens (cursors) instead of page numbers — better for large, frequently changing datasets |
| DLQ | Dead Letter Queue — queue for messages that failed processing after max retries |
| DuckDB | Embedded analytical database — used in Report Service for data warehouse queries |
| Dunning | Process of communicating with customers to collect payments on overdue invoices |
| E2E | End-to-End — testing that validates the entire system flow from user input to final output |
| EDI | Electronic Data Interchange — standardized format for exchanging business documents between organizations |
| ETL | Extract, Transform, Load — process of moving data from source systems to a data warehouse |
| EVM | Earned Value Management — project management technique for measuring performance |
| GDPR | General Data Protection Regulation — EU regulation governing data protection and privacy |
| GL | General Ledger — central accounting record of all financial transactions |
| Grafana | Open-source analytics and monitoring dashboard platform |
| Graceful Shutdown | Process termination procedure that completes in-flight work, closes connections, and flushes buffers before exiting |
| HPA | Horizontal Pod Autoscaler — Kubernetes component that automatically scales pod replicas |
| HIPAA | Health Insurance Portability and Accountability Act — US regulation for healthcare data protection |
| Idempotency | Property where performing the same operation multiple times produces the same result as performing it once |
| JE | Journal Entry — a single financial transaction record (debits and credits) |
| Jaeger | Open-source distributed tracing system for monitoring microservice interactions |
| JWT | JSON Web Token — compact, URL-safe means of representing claims between two parties |
| Kong | Open-source API gateway and microservices management layer — alternative to Traefik for MSERP |
| Kustomize | Kubernetes native configuration management tool for overlaying manifests across environments |
| Loki | Log aggregation system designed to work with Grafana for log querying |
| MFA | Multi-Factor Authentication — requires two or more verification methods |
| MinIO | High-performance, S3-compatible object storage server |
| MRP | Material Requirements Planning — production planning and inventory control system |
| mTLS | Mutual TLS — both client and server authenticate each other via certificates |
| NCR | Non-Conformance Report — document recording a product/process that does not meet quality standards |
| OIDC | OpenID Connect — identity layer on top of OAuth 2.0 for user authentication |
| OpenTelemetry | Open-source observability framework for generating, collecting, and exporting telemetry data (traces, metrics, logs) |
| Orchestration (Saga) | Saga coordination style where a central orchestrator service directs each step — used when a participant lacks an inbox queue |
| Outbox | Transactional outbox pattern — writes events to a database table within the same transaction as business data, then publishes asynchronously |
| Pact | Consumer-driven contract testing framework for verifying service integrations |
| PCI DSS | Payment Card Industry Data Security Standard — security standard for organizations handling credit card data |
| PDB | Pod Disruption Budget — Kubernetes policy limiting voluntary pod evictions during maintenance |
| PgBouncer | Lightweight connection pooler for PostgreSQL |
| PII | Personally Identifiable Information — data that can identify a specific individual |
| PostgreSQL | Open-source, object-relational database management system — primary data store for MSERP |
| Prometheus | Open-source monitoring and alerting toolkit with time-series data storage |
| RBAC | Role-Based Access Control — permissions assigned to roles, roles assigned to users |
| RDS | Row-Level Security — PostgreSQL feature restricting data access per row (used for tenant isolation) |
| Redis | In-memory data structure store used as cache, session store, and rate limiting backend in MSERP |
| RFQ | Request for Quotation — document sent to suppliers inviting them to bid on products or services |
| RLS | Row-Level Security — PostgreSQL feature restricting data access per row (used for tenant isolation) |
| RMA | Return Merchandise Authorization — authorization for a customer to return a product |
| RPO | Recovery Point Objective — maximum acceptable data loss measured in time |
| RTO | Recovery Time Objective — maximum acceptable downtime after a failure |
| Saga | Distributed transaction pattern using a sequence of local transactions with compensating (undo) actions on failure |
| SeaORM | Async, dynamic, and relational ORM for Rust built on SQLx — primary database toolkit for MSERP |
| SLO | Service Level Objective — target value for a service level measured by a metric |
| SLA | Service Level Agreement — commitment between service provider and client on uptime and performance |
| SKU | Stock Keeping Unit — unique identifier for a distinct product |
| SOC 2 | Service Organization Control 2 — auditing standard covering security, availability, processing integrity, confidentiality, and privacy |
| SOX | Sarbanes-Oxley Act — US law mandating financial reporting and audit practices for public companies |
| SSO | Single Sign-On — one set of credentials across multiple applications |
| Star Schema | Database schema design with a central fact table surrounded by dimension tables — used in data warehousing |
| TDE | Transparent Data Encryption — encryption of database files at the storage level |
| TLS | Transport Layer Security — cryptographic protocol for secure communications over a network |
| Traefik | Cloud-native edge router and reverse proxy for microservices — primary API Gateway for MSERP |
| Trivy | Open-source security scanner for containers, filesystems, and Git repositories |
| UoM | Unit of Measure — standard quantity used to express measurements (e.g., kg, liter, piece) |
| WAL | Write-Ahead Log — PostgreSQL mechanism ensuring data durability before writes are committed |
| WBS | Work Breakdown Structure — hierarchical decomposition of project deliverables into tasks |
| Write-Through | Caching pattern where the application writes data to both the cache and the database simultaneously |

## References

- [Rust Documentation](https://doc.rust-lang.org/)
- [Axum Framework](https://docs.rs/axum/)
- [SeaORM](https://www.sea-ql.org/SeaORM/)
- [RabbitMQ](https://www.rabbitmq.com/docs)
- [PostgreSQL](https://www.postgresql.org/docs/)
- [Kubernetes](https://kubernetes.io/docs/)
- [RFC 7807 - Problem Details](https://datatracker.ietf.org/doc/html/rfc7807)
- [OpenAPI 3.1 Specification](https://spec.openapis.org/oas/v3.1.0)
- [Pact Contract Testing](https://pact.io/)
- [Traefik Proxy](https://doc.traefik.io/traefik/)
- [OpenTelemetry](https://opentelemetry.io/docs/)
- [BPMN 2.0 Specification](https://www.omg.org/spec/BPMN/2.0/)
- [ISO 4217 (Currencies)](https://www.iso.org/iso-4217-currency-codes.html)
- [ISO 3166 (Countries)](https://www.iso.org/iso-3166-country-codes.html)
- [PCI DSS](https://www.pcisecuritystandards.org/)
- [GDPR](https://gdpr.eu/)
