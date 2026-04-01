# Finance Service (Finance + Procurement)

| Aspect | Details |
|--------|---------|
| Port | 8011 |
| Database | `finance_db` |
| Responsibilities | Financial accounting, purchasing, supplier management, multi-currency, budgeting, treasury, EPM, CLM, expenses, revenue recognition, strategic sourcing, supplier risk management, account reconciliation, profitability analysis, lease accounting, grant management, joint venture accounting, intelligent close, advanced collections, adaptive intelligence, advanced tax management, advanced tax filing, commodity management, spend analysis, supplier diversity |
| Rationale | See [Architecture — Service Consolidation](../01-architecture/overview.md#4-service-consolidation-rationale) |

## Finance Modules

| Module | Description |
|--------|-------------|
| Chart of Accounts | Multi-level account hierarchy, account segments, consolidation mapping |
| Journal Entries | Manual and automated journal entries, reversal, recurring entries |
| General Ledger | Periodic posting, trial balance, automatic posting from subledgers |
| Accounts Payable | Vendor invoices, payment batches, payment terms, early payment discounts |
| Accounts Receivable | Customer invoices, receipt application, dunning, collections management |
| Fixed Assets | Asset registration, depreciation (straight-line, declining balance, units-of-production), disposal, revaluation |
| Budgeting | Budget entry by account/period, budget vs. actual, budget approval workflow, multi-year budgets, rolling forecasts |
| Tax Management | Tax codes, tax rates, tax jurisdictions, automatic tax calculation, tax reporting (VAT, GST, sales tax) |
| Multi-Currency | Real-time exchange rates, triangulation, realized/unrealized gains/losses, revaluation |
| Cash Management | Cash position, bank reconciliation, cash flow forecasting |
| Intercompany | Intercompany transactions, elimination entries, consolidated reporting |
| Financial Reports | Balance sheet, P&L, cash flow statement, custom financial reports |
| Cost Centers | Cost center hierarchy, profit center allocation, overhead distribution |
| Period Management | Open/close accounting periods, soft-close (prevent modifications), hard-close (audit lock) |

## Corporate Treasury

| Module | Description |
|--------|-------------|
| Cash Position | Real-time cash position across all bank accounts and currencies |
| Bank Integration | Bank statement import (ISO 20022, OFX, BAI2), automatic reconciliation |
| Payment Formats | Multiple payment file formats (SEPA, ACH, SWIFT, local formats) |
| Debt Management | Loan tracking, interest calculation, amortization schedules |
| Investment Management | Short-term investment tracking, yield calculation |
| FX Exposure | Foreign currency exposure tracking, hedging recommendations |
| In-House Banking | Intercompany netting, cash pooling, virtual account management |

## Account Reconciliation

| Module | Description |
|--------|-------------|
| Auto-Matching | Rule-based matching of bank statements to open transactions by amount, date, and reference |
| Reconciliation Templates | Configurable reconciliation templates by account type with matching rules |
| Exception Management | Workflow-driven resolution of unmatched and partially matched transactions |
| Close Task Management | Financial close checklist with task assignment, dependencies, and deadline tracking |
| Intercompany Reconciliation | Automated matching of intercompany transactions across business units with elimination entries |

## Profitability Analysis

| Module | Description |
|--------|-------------|
| Cost-to-Serve | Full cost allocation to customer level including indirect costs, overhead, and logistics |
| Product Profitability | Gross and net margin analysis by product, category, and variant |
| Customer Profitability | Revenue vs. total cost analysis by customer and customer segment |
| Dimensional Analysis | Profitability by any dimension combination (product, customer, region, channel, business unit) |
| What-If Modeling | Scenario analysis for pricing changes, cost reductions, and product mix shifts |
| Contribution Margin | Variable cost analysis and contribution margin reporting by product line |

## Lease Accounting (ASC 842 / IFRS 16)

| Module | Description |
|--------|-------------|
| Lease Contracts | Lease contract registration with classification (operating vs. finance), terms, and payment schedules |
| Right-of-Use Assets | Right-of-use (ROU) asset calculation, amortization schedules, and impairment tracking |
| Lease Liabilities | Lease liability measurement with present value calculations, remeasurement on changes |
| Lease Payments | Payment schedule management with variable lease payment handling |
| Lease Modifications | Lease modification tracking with remeasurement and reclassification |
| Disclosure Reports | ASC 842 / IFRS 16 required disclosures: lease liabilities maturity, weighted-average discount rates, operating lease costs |

## Grant Management

| Module | Description |
|--------|-------------|
| Grant Registration | Grant registration with funding source, period, conditions, and budget |
| Grant Budgeting | Budget tracking against grant with allowable and unallowable cost categories |
| Cost Allocation | Grant cost allocation with indirect cost rate calculation |
| Revenue Recognition | Grant revenue recognition with milestone-based and cost-reimbursable methods |
| Compliance Tracking | Grant compliance monitoring with reporting requirements and audit trail |
| Grant Reporting | Grant financial reports, utilization dashboards, and funder-facing reports |

## Joint Venture Accounting

| Module | Description |
|--------|-------------|
| Joint Venture Setup | Joint venture creation with partner ownership percentages and cost sharing rules |
| Cost Allocation | Automated cost and revenue allocation across venture partners |
| Partner Billing | Partner billing statements with detailed cost breakdowns |
| Equity Accounting | Equity method accounting for joint venture investments |
| Reconciliation | Inter-partner reconciliation with automated matching |
| Reporting | Joint venture financial reports, partner statements, and contribution tracking |

## Intelligent Close

| Module | Description |
|--------|-------------|
| Close Automation | AI-driven financial close task automation with intelligent task assignment |
| Anomaly Detection | Automatic detection of anomalies in close data (unexpected balances, missing entries) |
| Close Dashboard | Real-time close progress dashboard with bottleneck identification |
| Auto-Reconciliation | ML-assisted auto-reconciliation with confidence scoring |
| Close Journal | Automated close journal entries (accruals, allocations, eliminations) |
| Continuous Close | Real-time period-end processing replacing batch close cycles |

## Advanced Collections

| Module | Description |
|--------|-------------|
| Collection Strategies | Configurable collection strategies based on aging, risk, and customer segmentation |
| Aging Analysis | Multi-dimensional aging analysis with historical trending |
| Collection Activities | Activity tracking (calls, emails, promises to pay) with outcome recording |
| Cash Application | Automated cash application with ML-based invoice matching |
| Dispute Management | Dispute tracking linked to collection activities with resolution workflows |
| Scoring | Predictive collection scoring for prioritizing collection activities |

## Dynamic Discounting & Early Payment Optimization

| Module | Description |
|--------|-------------|
| Discount Program Setup | Configure dynamic discount programs with sliding-scale discount tiers based on payment timing (e.g., 2% for payment within 10 days, 1% within 20 days) |
| Early Payment Offers | Automated early payment offer generation based on cash position, discount thresholds, and supplier discount terms |
| Discount Optimization | ML-powered optimization of which invoices to pay early based on discount APR vs. cost of capital, maximizing discount capture |
| Working Capital Analytics | Real-time working capital dashboards with DSO, DPO, DIO metrics and cash conversion cycle analysis |
| Discount Capture Rate | Discount capture rate tracking by supplier, category, and period with opportunity identification |
| APR Calculation | Automatic annualized percentage rate calculation for early payment discounts (e.g., 2/10 Net 30 = ~36.5% APR) |
| Cash Flow Impact | Simulation of early payment scenarios on cash position, liquidity, and banking covenants |
| Supplier Portal Integration | Supplier self-service portal for discount program enrollment, offer acceptance, and payment tracking |

## Enterprise Expense Management

| Module | Description |
|--------|-------------|
| Expense Reports | Expense report creation, itemization, receipt attachment |
| Per Diem | Configurable per diem rates by location and employee grade |
| Mileage Tracking | GPS-based mileage capture, configurable reimbursement rates |
| Corporate Cards | Corporate credit card transaction import, auto-matching to expenses |
| Approval Workflow | Multi-level approval with policy enforcement (amount, category, compliance) |
| Policy Compliance | Automatic policy violation detection, flagging, exception handling |
| Tax Recovery | VAT/GST recovery on eligible expenses |
| Expense Analytics | Spend by category, policy violation trends, reimbursement timing |

## Enterprise Performance Management (EPM)

| Module | Description |
|--------|-------------|
| Planning | Top-down and bottom-up planning by account, department, project |
| Forecasting | Rolling forecasts with driver-based models, statistical methods |
| What-If Scenarios | Scenario modeling with side-by-side comparison |
| Consolidation | Multi-entity financial consolidation with currency translation adjustments |
| Allocations | Rule-based allocations across dimensions (cost center, product, geography) |
| Variance Analysis | Actual vs. plan/budget/forecast variance reporting with drill-down |
| Scorecards | Financial scorecards with KPIs, targets, and traffic-light indicators |

## Financial Consolidation

| Module | Description |
|--------|-------------|
| Multi-Entity Consolidation | Consolidation of multiple legal entities with automated elimination entries and ownership percentage handling |
| Currency Translation | Foreign currency translation adjustments (temporal and current rate methods) with CTA posting |
| Elimination Entries | Automated intercompany elimination for revenue, expenses, receivables, and payables |
| Minority Interest | Non-controlling interest calculation and disclosure reporting |
| Segment Reporting | IFRS 8 / ASC 280 segment reporting with configurable segment dimensions (business unit, geography, product line) |

## Intercompany Netting

| Module | Description |
|--------|-------------|
| Netting Agreements | Configurable netting agreements between business units with settlement scheduling |
| Automated Netting | Automated offsetting of intercompany payables and receivables to reduce gross payment volumes |
| Settlement Processing | Netting settlement execution with payment file generation and reconciliation |
| Netting Dashboards | Netting exposure analysis, settlement history, and participant balance summaries |

## Cash Flow Statement (Direct Method)

| Module | Description |
|--------|-------------|
| Direct Method Reporting | Cash flow statement using the direct method showing major classes of gross cash receipts and payments |
| Indirect Method Reporting | Cash flow statement using the indirect method (net income to operating cash flow reconciliation) |
| Cash Flow Classification | Configurable mapping of GL accounts and transaction types to operating, investing, and financing activities |
| Comparative Reporting | Side-by-side cash flow comparison across periods with variance analysis |

## Revenue Recognition (ASC 606 / IFRS 15)

| Module | Description |
|--------|-------------|
| Contract Identification | Automated identification of customer contracts across sales orders, subscriptions, and project billing |
| Performance Obligations | Identification and tracking of distinct performance obligations per contract |
| Transaction Pricing | Allocation of transaction price to performance obligations based on standalone selling prices |
| Recognition Schedules | Automated recognition schedules: point-in-time, over-time (straight-line, proportional) |
| Variable Consideration | Estimation, constraint, and reassessment of variable consideration (discounts, rebates, contingencies) |
| Contract Modifications | Handling of contract changes with prospective and retrospective adjustment methods |
| Disclosure Reports | Required disclosures: remaining performance obligations, contract balances, recognized revenue |
| Revenue Waterfall | Visual waterfall from bookings to recognized revenue with deferral rollforward |
| Multi-Element Arrangements | Allocation across bundled goods, services, and licenses |

## Strategic Sourcing

| Module | Description |
|--------|-------------|
| Sourcing Events | Create and manage sourcing events (RFI, RFP, RFQ) with multi-round bidding and supplier collaboration |
| Bid Analysis | Side-by-side bid comparison with configurable scoring criteria and total cost of ownership calculation |
| Award Optimization | Award recommendation engine considering cost, quality, risk, diversity, and strategic criteria |
| Negotiation Tracking | Negotiation round management with version tracking, redline comparison, and term history |
| Sourcing Analytics | Savings tracking, supplier performance post-award, spend under management metrics |

## Supplier Risk Management

| Module | Description |
|--------|-------------|
| Risk Scoring | Composite supplier risk score from financial, operational, geographic, and regulatory indicators |
| Financial Health Monitoring | Automated monitoring of supplier financial data (credit ratings, financial statements, payment behavior) |
| Geographic Risk | Country and regional risk assessment including political stability, sanctions, and natural disaster exposure |
| Regulatory Compliance | Supplier compliance monitoring for industry certifications, regulatory requirements, and certifications |
| Risk Dashboards | Supplier risk heat maps, risk trend analysis, concentration risk visualization, alert management |
| Risk Mitigation | Mitigation plan tracking with action items, owners, and deadlines for identified risks |

## Contract Lifecycle Management (CLM)

| Module | Description |
|--------|-------------|
| Contract Authoring | Template-based contract creation with clause library |
| Clause Library | Reusable contract clauses with version control and approval status |
| Negotiation | Redline editing, version comparison, term tracking |
| Approval Workflow | Multi-level contract approval with legal review routing |
| Electronic Signatures | Integration with e-signature providers (DocuSign, Adobe Sign) |
| Obligation Tracking | Milestone and deliverable tracking against contract terms |
| Renewal Management | Automatic renewal alerts, renegotiation workflows |
| Contract Analytics | Contract value analysis, renewal rates, obligation compliance |

## Procurement Modules

| Module | Description |
|--------|-------------|
| Suppliers | Supplier registration, qualification, classification, bank details, tax information |
| Purchase Requisitions | Internal request for goods/services, approval workflow |
| RFQ (Request for Quotation) | Send RFQ to multiple suppliers, compare bids, award |
| Purchase Orders | PO creation from requisition or standalone, blanket orders, release orders |
| Goods Receipt | Receive against PO, inspection, put-away, quality hold |
| Supplier Evaluation | Scorecard (quality, delivery, price, responsiveness), periodic review |
| Contracts | Supplier contracts, terms, renewal tracking, compliance (purchasing contracts only; enterprise-wide contract lifecycle management is handled by CLM above) |
| Self-Service Procurement | Catalog-based purchasing for non-procurement users |
| Supplier Collaboration | Supplier portal for RFQ response, PO acknowledgment, invoice submission, delivery scheduling |

## Advanced Tax Management

| Module | Description |
|--------|-------------|
| Tax Determination | Automated taxability determination based on product classification, jurisdiction, and customer exemption status |
| Tax Calculation | Multi-jurisdiction tax calculation with support for VAT, GST, sales tax, use tax, withholding tax, and excise duties |
| Exemption Management | Customer and product exemption certificate management with validity tracking and renewal alerts |
| Tax Reporting | Jurisdiction-specific tax returns, VAT/GST returns, sales tax reports, and withholding tax certificates |
| Tax Reconciliation | Tax liability reconciliation with general ledger tax accounts, adjustment tracking, and audit trail |
| External Tax Engine | Pre-built connector for external tax engines (Vertex, Avalara, Sovos) for complex scenarios |
| Nexus Management | Multi-state / multi-country nexus tracking with economic nexus thresholds and filing obligation alerts |
| Tax Audit Support | Tax audit trail with transaction-level detail, exemption documentation, and assessment management |

## Commodity Management & Trading

| Module | Description |
|--------|-------------|
| Commodity Catalog | Commodity classification with specifications, grade standards, and market reference pricing |
| Market Price Integration | Real-time commodity market price feeds from exchanges and benchmark providers |
| Commodity Contracts | Commodity purchase contracts with quantity, price, delivery schedules, and escalation clauses |
| Hedging Integration | Commodity hedging position tracking with mark-to-market valuation and hedge effectiveness reporting |
| Futures & Forwards | Forward contract tracking with settlement scheduling and gain/loss recognition |

## Advanced Spend Analysis

| Module | Description |
|--------|-------------|
| Spend Classification | ML-powered automatic spend classification using UNSPSC, SIC, or custom category taxonomies |
| Spend Visibility | Real-time spend dashboards by supplier, category, business unit, geography, and contract |
| Contract Compliance | Spend under contract vs. off-contract analysis with maverick spend identification |
| Savings Identification | AI-driven savings opportunity identification across categories with estimated impact |
| Tail Spend Analysis | Automated tail spend identification with optimization recommendations |
| Predictive Analytics | Spend forecasting with trend analysis, seasonality detection, and anomaly alerting |

## Supplier Diversity Management

| Module | Description |
|--------|-------------|
| Diversity Classification | Supplier diversity classification with certification tracking |
| Spend Tracking | Real-time diverse supplier spend tracking by category, business unit, and classification |
| Tier Reporting | Tier 1 (direct) and Tier 2 (subcontractor) diversity spend reporting |
| Goal Setting | Configurable diversity spend targets with progress tracking |
| Compliance Reporting | Government-mandated diversity reporting with automated form generation |
| Sourcing Integration | Diversity criteria in sourcing events with weighted scoring |

## Advanced Financial Reporting Studio

> Finance owns the XBRL templates, regulatory report definitions, and financial report logic. Report Service owns the rendering engine and report delivery infrastructure.

| Module | Description |
|--------|-------------|
| Report Designer | Drag-and-drop report builder with grid cells, embedded charts, conditional formatting, and calculated fields |
| XBRL Taxonomy | XBRL taxonomy mapping with automated tagging of financial data elements for regulatory filing |
| Regulatory Templates | Pre-built report templates for SEC filings, IFRS/GAAP financial statements, and local regulatory requirements |
| Management Reporting | Management report builder with narrative sections, commentary, and collaborative annotations |
| Consolidated Reporting | Multi-entity consolidated report generation with elimination entries and currency translation |
| Report Scheduling | Scheduled report generation and distribution with burst-by entity/department |
| Interactive Drill-Down | Interactive reports with drill-down from summary to transaction-level detail |
| Report Versioning | Report template versioning with approval workflows and audit trail |
| Export Formats | Export to PDF, Excel, HTML, XBRL instance documents, and PowerPoint |
| Report Bursting | Automated report distribution to specific recipients based on entity, department, or role |

## Advanced Tax Filing

**Advanced Tax Filing** — Automated tax return preparation and filing across jurisdictions. Pre-populated tax returns from transactional tax data. Multi-jurisdiction filing calendar with deadline tracking and alerting. Electronic filing integration with tax authorities (IRS, HMRC, etc.). Tax return versioning, approval workflows, and audit trail. Amendment management for filed returns. See [advanced-tax-filing.md](../07-features/advanced-tax-filing.md).

| Table | Description |
|-------|-------------|
| `tax_filings` | Tax filing records with return data and filing status |
| `tax_filing_calendars` | Filing deadlines by jurisdiction and tax type |

| Event | Description |
|-------|-------------|
| `finance.tax.filing.submitted` | Tax return submitted to authority |
| `finance.tax.filing.accepted` | Tax return accepted by authority |
| `finance.tax.filing.rejected` | Tax return rejected by authority |

## Subledger Accounting Engine

**Subledger Accounting Engine** — Centralized journal generation from heterogeneous business events. Configurable accounting transformation rules map business transactions to journal entries using a rule-based engine. Multi-source aggregation from Commerce (orders, deliveries), HCM (payroll), and internal finance events. Journal entry templates with account derivation rules. Business transaction-to-journal mapping with extensible event handlers. Supports multi-book accounting with parallel journal generation for different accounting standards. See [subledger-accounting.md](../07-features/subledger-accounting.md).

## Database Tables

> All tables include standard columns per [SPEC.md §9.1](../SPEC.md).

| Table | Additional Columns |
|-------|-------------------|
| `accounts` | `code VARCHAR(30)`, `name VARCHAR(255)`, `type VARCHAR(30)`, `parent_id UUID`, `level INT`, `is_active BOOLEAN`, `normal_balance VARCHAR(10)`, `currency VARCHAR(3)` |
| `journal_entries` | `entry_number VARCHAR(50)`, `status VARCHAR(20)`, `entry_date DATE`, `posting_date DATE`, `description TEXT`, `period_id UUID`, `reversal_of_id UUID`, `source VARCHAR(30)` |
| `journal_entry_lines` | `journal_entry_id UUID`, `account_id UUID`, `debit DECIMAL`, `credit DECIMAL`, `description TEXT`, `cost_center_id UUID`, `dimension1_id UUID`, `dimension2_id UUID` |
| `invoices` | `invoice_number VARCHAR(50)`, `type VARCHAR(20)`, `status VARCHAR(20)`, `party_id UUID`, `party_type VARCHAR(20)`, `invoice_date DATE`, `due_date DATE`, `subtotal DECIMAL`, `tax_amount DECIMAL`, `total_amount DECIMAL`, `currency VARCHAR(3)`, `paid_amount DECIMAL` |
| `payments` | `payment_number VARCHAR(50)`, `type VARCHAR(20)`, `direction VARCHAR(10)`, `amount DECIMAL`, `currency VARCHAR(3)`, `payment_date DATE`, `method VARCHAR(30)`, `reference VARCHAR(100)`, `party_id UUID`, `party_type VARCHAR(20)` |
| `suppliers` | `code VARCHAR(50)`, `name VARCHAR(255)`, `email VARCHAR(255)`, `phone VARCHAR(50)`, `tax_id VARCHAR(50)`, `bank_details JSONB`, `status VARCHAR(20)`, `diversity_classification VARCHAR(50)` |
| `purchase_orders` | `po_number VARCHAR(50)`, `supplier_id UUID`, `status VARCHAR(20)`, `order_date DATE`, `expected_date DATE`, `total_amount DECIMAL`, `currency VARCHAR(3)` |
| `purchase_order_lines` | `purchase_order_id UUID`, `product_id UUID`, `description TEXT`, `quantity DECIMAL`, `unit_price DECIMAL`, `line_total DECIMAL`, `received_quantity DECIMAL` |
| `budgets` | `name VARCHAR(255)`, `fiscal_year INT`, `period VARCHAR(20)`, `account_id UUID`, `department_id UUID`, `amount DECIMAL`, `currency VARCHAR(3)`, `status VARCHAR(20)` |
| `tax_rates` | `code VARCHAR(30)`, `name VARCHAR(255)`, `rate DECIMAL`, `jurisdiction VARCHAR(100)`, `tax_type VARCHAR(20)`, `effective_from DATE`, `effective_to DATE`, `is_active BOOLEAN` |
| `exchange_rates` | `from_currency VARCHAR(3)`, `to_currency VARCHAR(3)`, `rate DECIMAL`, `rate_date DATE`, `source VARCHAR(30)` |
| `fixed_assets` | `asset_number VARCHAR(50)`, `name VARCHAR(255)`, `category VARCHAR(30)`, `acquisition_date DATE`, `acquisition_cost DECIMAL`, `currency VARCHAR(3)`, `depreciation_method VARCHAR(30)`, `useful_life_months INT`, `book_value DECIMAL`, `status VARCHAR(20)` |
| `expense_reports` | `report_number VARCHAR(50)`, `employee_id UUID`, `status VARCHAR(20)`, `submit_date DATE`, `total_amount DECIMAL`, `currency VARCHAR(3)`, `approved_by UUID` |
| `treasury_accounts` | `account_number VARCHAR(50)`, `bank_name VARCHAR(255)`, `currency VARCHAR(3)`, `balance DECIMAL`, `last_reconciled_at TIMESTAMPTZ`, `status VARCHAR(20)` |
| `contracts` | `contract_number VARCHAR(50)`, `title VARCHAR(255)`, `type VARCHAR(30)`, `party_id UUID`, `status VARCHAR(20)`, `start_date DATE`, `end_date DATE`, `value DECIMAL`, `currency VARCHAR(3)` |
| `revenue_contracts` | `contract_id UUID`, `customer_id UUID`, `total_transaction_price DECIMAL`, `currency VARCHAR(3)`, `status VARCHAR(20)`, `start_date DATE`, `end_date DATE` |
| `performance_obligations` | `revenue_contract_id UUID`, `description TEXT`, `standalone_selling_price DECIMAL`, `allocated_price DECIMAL`, `recognition_type VARCHAR(20)`, `status VARCHAR(20)` |
| `revenue_schedules` | `performance_obligation_id UUID`, `period DATE`, `amount DECIMAL`, `status VARCHAR(20)` |
| `lease_contracts` | `lease_number VARCHAR(50)`, `classification VARCHAR(20)`, `start_date DATE`, `end_date DATE`, `payment_amount DECIMAL`, `currency VARCHAR(3)`, `discount_rate DECIMAL`, `right_of_use_value DECIMAL`, `liability_value DECIMAL`, `status VARCHAR(20)` |
| `grants` | `grant_number VARCHAR(50)`, `funding_source VARCHAR(255)`, `start_date DATE`, `end_date DATE`, `total_budget DECIMAL`, `currency VARCHAR(3)`, `status VARCHAR(20)`, `compliance_requirements JSONB` |

## Events Published

| Event | Description |
|-------|-------------|
| `finance.journal.created` | Journal entry created |
| `finance.journal.posted` | Journal entry posted |
| `finance.journal.reversed` | Journal entry reversed |
| `finance.invoice.created` | Invoice created |
| `finance.invoice.creation.failed` | Invoice creation failed (saga compensation trigger) |
| `finance.invoice.paid` | Invoice paid |
| `finance.invoice.credit_memo` | Credit memo issued |
| `finance.payment.received` | Payment received |
| `finance.payment.made` | Payment made to supplier |
| `finance.supplier.created` | Supplier created |
| `finance.supplier.updated` | Supplier updated |
| `finance.purchase_order.created` | Purchase order created |
| `finance.purchase_order.approved` | Purchase order approved |
| `finance.purchase_order.received` | Goods received |
| `finance.account.created` | Account created in chart of accounts |
| `finance.account.updated` | Account modified in chart of accounts |
| `finance.period.closed` | Accounting period closed |
| `finance.budget.created` | Budget created |
| `finance.budget.exceeded` | Budget threshold exceeded (warning) |
| `finance.exchange_rate.updated` | Exchange rate updated |
| `finance.expense.submitted` | Expense report submitted |
| `finance.expense.approved` | Expense report approved |
| `finance.expense.rejected` | Expense report rejected |
| `finance.expense.paid` | Expense reimbursement paid |
| `finance.treasury.cash_position.updated` | Cash position updated across bank accounts |
| `finance.treasury.payment_batch.approved` | Payment batch approved for execution |
| `finance.treasury.payment_batch.executed` | Payment batch executed |
| `finance.contract.created` | Contract created |
| `finance.contract.approved` | Contract approved |
| `finance.contract.renewed` | Contract renewed |
| `finance.contract.expired` | Contract expired |
| `finance.plan.created` | Financial plan created (EPM) |
| `finance.plan.forecast.updated` | Forecast updated in financial plan |
| `finance.revenue.recognized` | Revenue recognized for a performance obligation |
| `finance.revenue.adjusted` | Revenue recognition schedule adjusted (contract modification) |
| `finance.revenue.deferred` | Revenue deferred to future period |
| `finance.credit_score.updated` | Customer credit score updated |
| `finance.credit_limit.changed` | Customer credit limit changed |
| `finance.sourcing.event.created` | Sourcing event (RFI/RFP/RFQ) created |
| `finance.sourcing.bid.submitted` | Supplier bid submitted for sourcing event |
| `finance.sourcing.event.awarded` | Sourcing event awarded to supplier(s) |
| `finance.supplier_risk.score.updated` | Supplier risk score updated |
| `finance.supplier_risk.alert.triggered` | Supplier risk alert triggered |
| `finance.supplier_risk.mitigation.created` | Supplier risk mitigation plan created |
| `finance.reconciliation.matched` | Reconciliation auto-match completed |
| `finance.reconciliation.exception.created` | Unmatched reconciliation exception created |
| `finance.reconciliation.completed` | Account reconciliation completed |
| `finance.close_task.completed` | Financial close task completed |
| `finance.profitability.analysis.completed` | Profitability analysis run completed |
| `finance.lease.created` | Lease contract created |
| `finance.lease.modified` | Lease contract modified |
| `finance.lease.payment.due` | Lease payment due |
| `finance.lease.right_of_use.adjusted` | Right-of-use asset value adjusted |
| `finance.grant.created` | Grant registered |
| `finance.grant.milestone.reached` | Grant milestone reached |
| `finance.grant.revenue.recognized` | Grant revenue recognized |
| `finance.grant.compliance.check_due` | Grant compliance check due |
| `finance.joint_venture.created` | Joint venture created |
| `finance.joint_venture.cost.allocated` | Joint venture cost allocated to partners |
| `finance.joint_venture.billing.generated` | Joint venture partner billing generated |
| `finance.intelligent_close.task.auto_assigned` | Close task automatically assigned by AI |
| `finance.intelligent_close.anomaly.detected` | Financial close anomaly detected |
| `finance.intelligent_close.auto_reconciled` | Account auto-reconciled during intelligent close |
| `finance.collection.strategy.triggered` | Collection strategy automatically triggered |
| `finance.collection.activity.created` | Collection activity created |
| `finance.cash_application.matched` | Cash receipt auto-matched to invoice |
| `finance.cash_application.unmatched` | Cash receipt could not be auto-matched |
| `finance.tax.assessment.created` | Tax assessment created for transaction |
| `finance.commodity.price.updated` | Commodity market price updated |
| `finance.spend.classified` | Spend transaction classified by ML |
| `finance.diversity.spend.recorded` | Diverse supplier spend recorded |
| `finance.dynamic_discount.offer_created` | Early payment discount offer generated |
| `finance.dynamic_discount.offer_accepted` | Supplier accepted early payment discount offer |
| `finance.dynamic_discount.payment_executed` | Early payment executed with discount applied |
| `finance.report.template_created` | Financial report template created |
| `finance.report.generated` | Financial report generated from template |
| `finance.report.published` | Financial report published for distribution |
| `finance.xbrl.filing_completed` | XBRL filing completed and validated |
| `finance.sla.accounting_event_created` | Subledger accounting event created |
| `finance.sla.journal_generated` | Subledger journal entry generated |
| `finance.sla.rule_executed` | Subledger accounting rule executed |
| `finance.sla.source_processed` | Subledger source transaction processed |

## Events Consumed

Inbox binding: `finance.inbox` binds to the following routing keys:

| Binding Pattern | Events Consumed |
|----------------|-----------------|
| `finance.#` | Self-binding for saga compensation |
| `commerce.order.#` | `commerce.order.created`, `commerce.order.submitted`, `commerce.order.fulfilled`, `commerce.order.cancelled`, `commerce.order.returned` |
| `commerce.stock.#` | `commerce.stock.updated`, `commerce.stock.reserved`, `commerce.stock.reservation.failed`, `commerce.stock.reservation.released` |
| `commerce.subscription.#` | `commerce.subscription.created`, `commerce.subscription.amended`, `commerce.subscription.renewed`, `commerce.subscription.cancelled`, `commerce.subscription.billing.completed` |
| `commerce.b2b.#` | `commerce.b2b.order.placed`, `commerce.b2b.order.approved` |
| `commerce.warranty.#` | `commerce.warranty.claim.submitted`, `commerce.warranty.claim.approved` |
| `hr.payroll.#` | `hr.payroll.processed` |
| `manufacturing.work_order.#` | `manufacturing.work_order.created`, `manufacturing.work_order.started`, `manufacturing.work_order.completed` |
| `project.invoice.#` | `project.invoice.generated` |
| `project.milestone.#` | `project.milestone.reached` |
| `platform.idp.#` | `platform.idp.document.classified`, `platform.idp.extraction.completed`, `platform.idp.extraction.failed` |
| `config.changed` | `config.changed` |

## See Also

- [Commerce Service](commerce.md)
- [HCM Service](hr.md)
- [Report Service](report.md)
- [Architecture Overview](../01-architecture/overview.md)
