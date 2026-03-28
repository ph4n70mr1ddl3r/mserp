# Glossary

## Terminology

| Term | Definition |
|------|------------|
| AAC | Advanced Access Controls — comprehensive access governance including SoD, access certification, and privileged access management |
| ABAC | Attribute-Based Access Control — authorization model that evaluates attributes (user, resource, environment) to make access decisions, supplementing RBAC |
| AES-256 | Advanced Encryption Standard with 256-bit key — symmetric encryption algorithm for data at rest |
| AMQP | Advanced Message Queuing Protocol — messaging protocol used by RabbitMQ for reliable, standards-based messaging |
| AP | Accounts Payable — money owed by the company to suppliers |
| API | Application Programming Interface — set of protocols for building and integrating software |
| AQM | Advanced Quality Management — comprehensive quality control including statistical process control, Six Sigma, and continuous improvement |
| AR | Accounts Receivable — money owed to the company by customers |
| ArgoCD | GitOps continuous delivery tool for Kubernetes — syncs cluster state from Git repositories |
| ASC 606 | Accounting Standards Codification Topic 606 — US GAAP standard for revenue recognition from contracts with customers |
| ASCP | Advanced Supply Chain Planning — constraint-based planning and optimization for supply chain operations |
| ATP | Available-to-Promise — inventory availability check that considers current stock, open orders, and planned receipts to determine if a customer order can be fulfilled by a specific date |
| Axum | Ergonomic and modular web framework for Rust — built on Tokio and Tower |
| BOM | Bill of Materials — list of components required for manufacturing a product |
| BPMN | Business Process Model and Notation — standard for business process diagrams |
| Cache-Aside | Caching pattern where the application checks the cache first; on miss, fetches from the database and populates the cache |
| Cap Table | Table tracking loyalty program member tier thresholds, qualification criteria, and benefit entitlements |
| CAPA | Corrective and Preventive Action — systematic approach to investigating and resolving non-conformances |
| cert-manager | Kubernetes add-on for automated TLS certificate management and rotation |
| Changelog | Immutable, append-only log recording all data mutations for compliance and auditing purposes |
| Choreography (Saga) | Saga coordination style where each service independently reacts to events — no central orchestrator |
| COA | Chart of Accounts — structured list of all financial account codes |
| Compensating Action | Undo operation in a saga step that reverses the effect of a previously completed step when a later step fails |
| Compliance Hub | Unified compliance management dashboard aggregating regulatory status across all compliance domains |
| Contact Center | Centralized operation handling inbound and outbound customer interactions across multiple channels |
| CORS | Cross-Origin Resource Sharing — mechanism allowing web applications to access resources from different domains |
| CRM | Customer Relationship Management — strategy and tools for managing customer interactions and data |
| CTP | Capable-to-Promise — availability check that considers production capacity and procurement lead times for items not currently in stock |
| Cursor Pagination | Pagination using opaque tokens (cursors) instead of page numbers — better for large, frequently changing datasets |
| DLQ | Dead Letter Queue — queue for messages that failed processing after max retries |
| DuckDB | Embedded analytical database — used in Report Service for data warehouse queries |
| Dunning | Process of communicating with customers to collect payments on overdue invoices |
| E2E | End-to-End — testing that validates the entire system flow from user input to final output |
| EAM | Enterprise Asset Management — lifecycle management of physical assets including maintenance, depreciation, and retirement |
| ECCN | Export Control Classification Number — code identifying the level of export control for an item under US export regulations |
| ECO | Engineering Change Order — formal document authorizing changes to a product's design, BOM, or manufacturing process |
| EDI | Electronic Data Interchange — standardized format for exchanging business documents between organizations |
| EPM | Enterprise Performance Management — integrated planning, budgeting, forecasting, and financial consolidation |
| ESG | Environmental, Social, and Governance — framework for evaluating corporate sustainability and societal impact |
| ETL | Extract, Transform, Load — process of moving data from source systems to a data warehouse |
| EVM | Earned Value Management — project management technique for measuring performance |
| GDPR | General Data Protection Regulation — EU regulation governing data protection and privacy |
| GL | General Ledger — central accounting record of all financial transactions |
| Golden Record | Single, authoritative version of a data entity maintained by MDM — the trusted source across all systems |
| Grafana | Open-source analytics and monitoring dashboard platform |
| Graceful Shutdown | Process termination procedure that completes in-flight work, closes connections, and flushes buffers before exiting |
| GRC | Governance, Risk, and Compliance — integrated approach to organizational governance, risk management, and regulatory compliance |
| Hedge | Financial derivative position taken to reduce the risk of adverse price movements in an asset |
| HPA | Horizontal Pod Autoscaler — Kubernetes component that automatically scales pod replicas |
| HIPAA | Health Insurance Portability and Accountability Act — US regulation for healthcare data protection |
| IaC | Infrastructure as Code — managing infrastructure through version-controlled configuration files |
| IFRS 15 | International Financial Reporting Standard 15 — global standard for revenue recognition from contracts with customers |
| Idempotency | Property where performing the same operation multiple times produces the same result as performing it once |
| JE | Journal Entry — a single financial transaction record (debits and credits) |
| Jaeger | Open-source distributed tracing system for monitoring microservice interactions |
| JWT | JSON Web Token — compact, URL-safe means of representing claims between two parties |
| Kong | Open-source API gateway and microservices management layer — alternative to Traefik for MSERP |
| Kustomize | Kubernetes native configuration management tool for overlaying manifests across environments |
| Loki | Log aggregation system designed to work with Grafana for log querying |
| Maverick Spend | Organizational purchases made outside of approved contract or preferred suppliers |
| MDM | Master Data Management — discipline of managing critical enterprise data to provide a single trusted source |
| MFA | Multi-Factor Authentication — requires two or more verification methods |
| MinIO | High-performance, S3-compatible object storage server |
| MPS | Master Production Schedule — plan for production of individual items by time period |
| MRP | Material Requirements Planning — production planning and inventory control system |
| mTLS | Mutual TLS — both client and server authenticate each other via certificates |
| NCR | Non-Conformance Report — document recording a product/process that does not meet quality standards |
| Nexus | Sufficient connection between a taxing jurisdiction and a business that creates a tax filing obligation |
| NLP | Natural Language Processing — AI capability enabling computers to understand and respond to human language |
| NPS | Net Promoter Score — metric measuring customer loyalty based on the question "How likely are you to recommend this company?" |
| OCR | Optical Character Recognition — technology for extracting text from images and scanned documents |
| OFAC | Office of Foreign Assets Control — US government agency administering economic and trade sanctions |
| OIDC | OpenID Connect — identity layer on top of OAuth 2.0 for user authentication |
| ONNX | Open Neural Network Exchange — open format for representing machine learning models |
| OpenTelemetry | Open-source observability framework for generating, collecting, and exporting telemetry data (traces, metrics, logs) |
| Orchestration (Saga) | Saga coordination style where a central orchestrator service directs each step — used when a participant lacks an inbox queue |
| Outbox | Transactional outbox pattern — writes events to a database table within the same transaction as business data, then publishes asynchronously |
| Pact | Consumer-driven contract testing framework for verifying service integrations |
| PCI DSS | Payment Card Industry Data Security Standard — security standard for organizations handling credit card data |
| PDB | Pod Disruption Budget — Kubernetes policy limiting voluntary pod evictions during maintenance |
| PgBouncer | Lightweight connection pooler for PostgreSQL |
| PII | Personally Identifiable Information — data that can identify a specific individual |
| PIM | Product Information Management — centralized management of product data across all channels |
| PLM | Product Lifecycle Management — managing a product from inception through design, manufacturing, service, and disposal |
| Points Ledger | Accounting ledger tracking loyalty point accruals, redemptions, and expiry for audit and reconciliation |
| PostgreSQL | Open-source, object-relational database management system — primary data store for MSERP |
| Price Elasticity | Measure of how demand for a product changes in response to price changes |
| Prometheus | Open-source monitoring and alerting toolkit with time-series data storage |
| RBAC | Role-Based Access Control — permissions assigned to roles, roles assigned to users |
| Redis | In-memory data structure store used as cache, session store, and rate limiting backend in MSERP |
| RFQ | Request for Quotation — document sent to suppliers inviting them to bid on products or services |
| RLS | Row-Level Security — PostgreSQL feature restricting data access per row (used for tenant isolation) |
| RMA | Return Merchandise Authorization — authorization for a customer to return a product |
| Rotable | Repairable component that can be removed, overhauled, and reinstalled in a maintenance cycle, tracked through an exchange pool |
| RPO | Recovery Point Objective — maximum acceptable data loss measured in time |
| RTO | Recovery Time Objective — maximum acceptable downtime after a failure |
| Saga | Distributed transaction pattern using a sequence of local transactions with compensating (undo) actions on failure |
| SeaORM | Async, dynamic, and relational ORM for Rust built on SQLx — primary database toolkit for MSERP |
| SLO | Service Level Objective — target value for a service level measured by a metric |
| SLA | Service Level Agreement — commitment between service provider and client on uptime and performance |
| SKU | Stock Keeping Unit — unique identifier for a distinct product |
| Social Selling | Use of social media platforms to identify, connect with, and build relationships with prospects and customers |
| SoD | Segregation of Duties — internal control mechanism ensuring no single person controls all aspects of a transaction |
| SOC 2 | Service Organization Control 2 — auditing standard covering security, availability, processing integrity, confidentiality, and privacy |
| SOX | Sarbanes-Oxley Act — US law mandating financial reporting and audit practices for public companies |
| SPC | Statistical Process Control — quality control method using statistical methods to monitor and control processes |
| SSP | Standalone Selling Price — the price at which an entity would sell a promised good or service separately to a customer |
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
| XBRL | eXtensible Business Reporting Language — standard for electronic communication of business and financial data |
| CDP | Customer Data Platform — system that creates a unified, persistent customer database accessible to other systems for segmentation, analytics, and campaign execution |
| CPM | Corporate Performance Management — framework and tools for managing organizational strategy, objectives, KPIs, and initiatives across the enterprise |
| Digital Twin | Virtual representation of a physical asset that mirrors its real-time state and enables simulation, predictive maintenance, and optimization |
| IoT | Internet of Things — network of physical devices embedded with sensors, software, and connectivity to collect and exchange data |
| OKR | Objectives and Key Results — goal-setting framework defining objectives (what to achieve) and key results (how to measure progress) |
| Omnichannel | Retail strategy providing a seamless customer experience across all sales channels (web, mobile, store, marketplace) |
| Process Mining | Analytical technique that discovers, monitors, and improves business processes by extracting knowledge from event logs |
| RPA | Robotic Process Automation — technology for automating repetitive, rule-based tasks using software bots that mimic human interactions with applications |
| Strategic Sourcing | Systematic approach to optimizing an organization's supply base and improving the overall value proposition through structured sourcing events and supplier evaluation |
| Identity Resolution | Process of determining that different data points (cookies, device IDs, emails) belong to the same individual across channels and touchpoints |
| DLP | Data Loss Prevention — strategies and tools for preventing unauthorized data exfiltration |
| ECM | Enterprise Content Management — strategy and tools for managing an organization's content lifecycle |
| GraphQL | Query language and runtime for APIs enabling clients to request exactly the data they need |
| Digital Thread | End-to-end traceability framework connecting data across the product lifecycle from design through manufacturing to service |
| Augmented Analytics | AI-powered analytics that automate data preparation, insight discovery, and narrative generation |
| Adaptive Intelligence | ML-powered recommendations and predictions embedded directly into business workflows |
| API Marketplace | Centralized portal for discovering, subscribing to, and managing API consumption |
| Event Mesh | Event routing infrastructure enabling event-driven communication across distributed systems and services |
| Privacy by Design | Framework requiring privacy considerations to be embedded into the design of systems from inception |
| NLQ | Natural Language Query — ability to query data using natural language instead of SQL or code |
| OEE | Overall Equipment Effectiveness — manufacturing metric combining availability, performance, and quality |
| MTBF | Mean Time Between Failures — average time between equipment breakdowns |
| MTTR | Mean Time To Repair — average time to restore equipment to operational status |
| ASC 842 | Accounting Standards Codification Topic 842 — US GAAP standard for lease accounting requiring lessees to recognize right-of-use assets and lease liabilities |
| IFRS 16 | International Financial Reporting Standard 16 — global standard for lease accounting requiring lessees to recognize most leases on the balance sheet |
| ROU Asset | Right-of-Use Asset — an asset that represents a lessee's right to use an underlying asset for the lease term |
| Grant Management | Process of administering financial grants including registration, budgeting, compliance tracking, and reporting |
| Joint Venture | Business arrangement where two or more parties agree to pool resources for a specific task or project with shared costs, risks, and rewards |
| Intelligent Close | AI-driven approach to financial period-end close that automates tasks, detects anomalies, and enables continuous accounting |
| Cash Application | Process of matching incoming payments (cash, checks, wire transfers) to the corresponding open invoices |
| IDP | Intelligent Document Processing — AI-powered technology that automatically extracts, classifies, and validates data from documents |
| Warranty Management | End-to-end management of product warranties including policy definition, registration, claims processing, and analytics |
| A/B Testing | Controlled experiment comparing two variants to determine which performs better against defined goals |
| BOPIS | Buy Online, Pick Up In Store — omnichannel fulfillment model where customers purchase online and collect from a physical store |
| CMDB | Configuration Management Database — IT asset registry with relationship mapping |
| Commodity | Raw material or primary agricultural product that can be bought and sold, typically through exchanges with standardized contracts |
| DPIA | Data Protection Impact Assessment — privacy risk assessment required by GDPR |
| EHS | Enterprise Health & Safety — workplace safety management and regulatory compliance |
| ISCC | International Sustainability & Carbon Certification |
| ITSM | IT Service Management — IT service delivery and support framework |
| JHA | Job Hazard Analysis — systematic method for identifying workplace hazards |
| MRO | Maintenance, Repair & Operations — supplies and activities for keeping assets operational |
| SDS | Safety Data Sheet — standardized chemical hazard information document |
| UNSPSC | United Nations Standard Products and Services Code — spend classification taxonomy |

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
- [ONNX Runtime](https://onnxruntime.ai/)
- [React Native](https://reactnative.dev/)
- [qdrant Vector Search](https://qdrant.tech/)
- [DuckDB](https://duckdb.org/)
- [Apache Arrow](https://arrow.apache.org/)
- [BPMN 2.0](https://www.omg.org/spec/BPMN/2.0/)
- [ASC 606 / IFRS 15](https://www.ifrs.org/issued-standards/list-of-standards/ifrs-15-revenue-from-contracts-with-customers/)
- [GRI Standards](https://www.globalreporting.org/standards/)
- [GraphQL](https://graphql.org/)
- [Oracle Fusion Cloud](https://www.oracle.com/cloud/applications/)
- [OWASP ZAP](https://www.zaproxy.org/)
