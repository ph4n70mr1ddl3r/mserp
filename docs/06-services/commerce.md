# Commerce Service (Sales + Inventory)

| Aspect | Details |
|--------|---------|
| Port | 8010 |
| Database | `commerce_db` |
| Responsibilities | Sales operations, customer management, stock management, warehousing, pricing engine, PIM, transportation, ATP/CTP, product configurator, credit management, subscription management, drop ship, connected logistics, warranty management, B2B commerce portal, Adaptive Intelligence integration with Report Service, loyalty management, omnichannel management, price optimization |
| Rationale | See [Architecture — Service Consolidation](../01-architecture/overview.md#4-service-consolidation-rationale) |

## Sales Modules

| Module | Description |
|--------|-------------|
| Customers | Customer CRUD, credit limits, customer groups, pricing tier assignment, customer transactional data |
| Quotations | Quote creation, versioning, approval, acceptance/expiry workflow |
| Sales Orders | Order creation from quote or standalone, hold management, backorder handling |
| Deliveries & Shipments | Delivery scheduling, shipment tracking, proof of delivery |
| Sales Invoices | Invoice generation from delivery, credit memos, dunning management |
| Commissions | Commission rules, tiered rates, split commissions, payout calculation |
| Returns & Refunds | Return merchandise authorization (RMA), refund processing, restocking |
| Sales Analytics | Order volume, revenue trends, average order value, fulfillment performance |

> **CRM Handoff:** CRM Service owns leads, opportunities, and the customer 360 view (CDP). Commerce receives qualified opportunities from CRM via events and manages the order lifecycle from quotation through delivery.

## Inventory Modules

| Module | Description |
|--------|-------------|
| Products / SKUs | Product hierarchy (category → subcategory → product → variant), attributes, multi-UoM |
| Stock Levels | Real-time stock across warehouses, safety stock, reorder points |
| Warehouses | Warehouse definition, zones, bins, pick/put strategies |
| Stock Transfers | Inter-warehouse transfers, in-transit tracking, receipt confirmation |
| Stock Valuation | FIFO, LIFO, weighted average, standard cost methods (per product) |
| Serial / Batch Tracking | Full traceability, lot numbers, expiry tracking, recall management |
| Reorder Automation | Min/max, EOQ, demand-driven reorder suggestions |
| Cycle Count | Scheduled and ad-hoc cycle counts, variance analysis |

## Product Information Management (PIM / Product Hub)

| Module | Description |
|--------|-------------|
| Product Data Model | Flexible attribute schema with configurable attribute groups per product category |
| Category Management | Multi-level product categories with inheritance of attributes |
| Media Management | Product images, videos, documents with multi-format support |
| Channel Publishing | Publish product data to multiple channels (web, mobile, marketplace, EDI) |
| Data Quality | Completeness checks, mandatory attribute validation, enrichment workflows |
| Cross-Reference | Map internal SKUs to supplier, customer, and marketplace product identifiers |

## Transportation & Fleet Management

| Module | Description |
|--------|-------------|
| Carrier Management | Carrier registration, service level agreements, rate tables |
| Shipment Planning | Optimal carrier selection based on cost, transit time, and service level |
| Route Optimization | Route planning for multi-stop deliveries, fuel cost estimation |
| Freight Costing | Freight charge calculation, bill of lading generation, freight audit |
| Proof of Delivery | Digital POD capture, signature, photo, GPS coordinates |
| Carrier Performance | On-time delivery tracking, damage rate, carrier scorecards |

## Advanced Pricing Engine

| Pricing Component | Description |
|-------------------|-------------|
| Price Lists | Multiple price lists by currency, customer group, date range |
| Discount Rules | Percentage, fixed amount, buy-X-get-Y, volume/tiered discounts |
| Promotions | Time-limited promotions with coupon codes, usage limits |
| Price Formulas | Calculated pricing based on cost + margin, competitor price, or custom formula |
| Customer-Specific Pricing | Override price lists per customer or customer group |
| Multi-Currency Pricing | Automatic conversion or fixed prices per currency |
| Margin Analysis | Real-time margin calculation at line and order level |

## Available-to-Promise (ATP) / Capable-to-Promise (CTP)

| Module | Description |
|--------|-------------|
| ATP Check | Real-time availability check across warehouses considering open orders, safety stock, and lead times |
| CTP Check | Availability check considering production capacity and procurement lead times for make-to-order items |
| Allocation Rules | Configurable allocation rules by customer priority, channel, and region |
| Promise Date Calculation | Earliest ship date calculation considering inventory, production, and logistics constraints |
| Cross-Region ATP | Aggregate availability across regions with inter-company transfer time |
| ATP Dashboard | Real-time availability views by product, warehouse, and region |

## Product Configurator

| Module | Description |
|--------|-------------|
| Configuration Models | Define configurable product models with features, options, and constraints |
| Constraint Rules | Compatibility and incompatibility rules between options (e.g., engine A requires transmission B) |
| Pricing Integration | Dynamic price calculation based on selected configuration |
| BOM Generation | Automatic generation of production BOM from selected configuration |
| Validation Engine | Real-time completeness and feasibility validation |
| Guided Selling | Interactive configuration wizard with recommendations and upsell suggestions |
| Configuration Templates | Pre-defined configurations for common product variants |

## Credit Management

| Module | Description |
|--------|-------------|
| Credit Scoring | Automated credit scoring based on payment history, financial data, and external credit bureaus |
| Credit Limits | Customer credit limit management with currency support and multi-entity rules |
| Credit Hold | Automatic order hold when credit limit exceeded, configurable threshold alerts |
| Credit Release | Manual and automated credit release workflows with approval chains |
| Credit Exposure | Real-time credit exposure calculation (open orders + invoices + unbilled) |
| Collection Scoring | Predictive collection scoring for prioritizing collection activities |
| Credit Reports | Customer credit history, aging analysis, payment pattern analysis |

## Subscription Management

| Module | Description |
|--------|-------------|
| Subscription Lifecycle | Create, activate, amend, renew, suspend, cancel subscriptions |
| Billing Models | Recurring flat-rate, usage-based, tiered/graduated, per-unit, hybrid pricing |
| Billing Schedules | Automated billing cycle management with proration for mid-cycle changes |
| Usage Metering | Usage tracking and metering for consumption-based billing |
| Amendments | Mid-cycle changes with proration: upgrade, downgrade, add/remove items |
| Renewal Management | Automated renewal with configurable terms, renewal reminders, auto-renewal rules |
| Subscription Analytics | MRR/ARR tracking, churn analysis, cohort analysis, retention rates |
| Revenue Recognition Integration | Automatic performance obligation creation for Finance Service revenue recognition |

## Intercompany Drop Ship

| Module | Description |
|--------|-------------|
| Drop Ship Orders | Customer orders fulfilled directly by supplier or intercompany partner |
| Supplier Collaboration | Automated PO generation and dispatch to drop ship supplier |
| Shipment Tracking | Customer-visible tracking from supplier's shipment |
| Invoice Reconciliation | Automated reconciliation of supplier invoice, customer invoice, and PO |
| Multi-Entity Support | Intercompany drop ship across business units with automated intercompany accounting |

## Connected Logistics & Track-and-Trace

| Module | Description |
|--------|-------------|
| Real-Time Tracking | GPS-based shipment tracking with carrier integration and geofencing |
| Condition Monitoring | Temperature, humidity, and shock monitoring for sensitive shipments (IoT sensor integration) |
| Predictive ETA | ML-based estimated delivery time considering traffic, weather, and historical performance |
| Geofencing | Automatic alerts on geofence entry/exit for proactive logistics management |
| Exception Management | Automated exception detection and resolution workflows for delayed, damaged, or deviated shipments |
| Last-Mile Optimization | Route optimization for final delivery with delivery time window management |

## Warranty Management

| Module | Description |
|--------|-------------|
| Warranty Policies | Warranty policy definition with coverage terms, duration, and conditions |
| Warranty Registration | Product warranty registration linked to serial/lot numbers |
| Warranty Claims | Claim submission, validation, approval, and fulfillment workflow |
| Warranty Analytics | Claim frequency, cost analysis, and product quality correlation |
| Extended Warranties | Extended warranty sales and management with revenue recognition |

## B2B Commerce Portal

| Module | Description |
|--------|-------------|
| Customer Self-Service | Customer portal for placing and tracking orders with real-time availability checks |
| Reorder Templates | Quick-reorder from order history and configurable saved shopping lists |
| Customer Approval Workflows | Customer-side approval chains for configurable order value thresholds |
| Account Management | Customer-managed users, shipping addresses, payment methods, and preferences |
| Order Tracking | Real-time order and shipment tracking with configurable notification preferences |
| Invoice Portal | Self-service invoice viewing, download, dispute initiation, and payment status |

## Customer Loyalty Management

| Module | Description |
|--------|-------------|
| Loyalty Programs | Configurable loyalty programs with tier structures, earning rules, and benefit catalogs |
| Points Engine | Real-time points accrual and redemption with configurable earning rates by product, channel, and tier |
| Tier Management | Automated tier qualification, promotion, and demotion based on configurable thresholds |
| Reward Catalog | Digital reward catalog with merchandise, discounts, partner offers, and experiential rewards |
| Partner Programs | Coalition loyalty programs with partner point earning and shared reward pools |
| Fraud Prevention | Duplicate account detection, unusual redemption patterns, and points laundering prevention |
| Loyalty Analytics | Program ROI, member engagement, breakage rates, tier migration, and lifetime value analysis |

## Unified Omnichannel Management

| Module | Description |
|--------|-------------|
| Channel Management | Configure and manage sales channels (web, mobile, B2B portal, marketplace, POS, call center, social commerce) |
| Unified Cart | Persistent shopping cart synchronized across channels with real-time inventory visibility |
| Order Routing | Intelligent order routing across fulfillment locations based on proximity, cost, and stock availability |
| Cross-Channel Returns | Return-anywhere capability with centralized return policy enforcement and refund processing |
| Click-and-Collect | Buy online, pick up in-store (BOPIS) with real-time store inventory and notification management |
| Ship-from-Store | Store fulfillment capability with inventory reservation and carrier integration |

## AI-Driven Price Optimization

| Module | Description |
|--------|-------------|
| Demand-Based Pricing | ML models predicting demand elasticity by product, segment, channel, and season |
| Competitive Pricing | Automated competitive price monitoring and response strategies with floor/ceiling constraints |
| Dynamic Pricing | Real-time price adjustments based on inventory levels, demand signals, and competitive landscape |
| Markdown Optimization | Optimal markdown scheduling for seasonal and aging inventory with margin preservation targets |
| Margin Optimization | Multi-constraint optimization balancing margin, volume, revenue, and inventory targets |
| Price Testing | A/B price testing with statistical significance tracking and revenue impact measurement |
| What-If Simulation | Price change scenario modeling with projected revenue, margin, and volume impact |

## Advanced Inventory Optimization

| Module | Description |
|--------|-------------|
| Safety Stock Optimization | ML-driven safety stock calculation using demand variability, lead time variability, and target service levels |
| Demand-Driven Replenishment | Demand-driven material requirements planning (DDMRP) with strategically placed decoupling points |
| Service Level Planning | Service level-based inventory planning with fill rate and ready rate optimization |
| Replenishment Simulation | What-if simulation for replenishment policy changes with projected stockout and overstock impact |

## Demand Planning Integration

| Module | Description |
|--------|-------------|
| Forecast Ingestion | Integration with Report Service demand forecasting models for inventory planning |
| Forecast Consumption | Sales order consumption of forecast quantities with variance tracking |
| Collaborative Planning | Cross-functional demand review workflows with consensus forecasting |

## Supply Allocation

| Module | Description |
|--------|-------------|
| Allocation Rules | Rules-based supply allocation during shortage scenarios with configurable customer prioritization |
| Fair Share Allocation | Proportional allocation of limited supply across customers based on order date, quantity, and priority |
| Allocation Scheduling | Scheduled allocation runs with reallocation triggers on supply or demand changes |
| Allocation Analytics | Allocation fill rate analysis, shortage impact reporting, and customer satisfaction scoring |

## Supply Chain Collaboration Network

| Module | Description |
|--------|-------------|
| Supplier Collaboration Portal | Multi-tier supplier portal for demand visibility, order collaboration, and performance tracking |
| Demand Signal Sharing | Configurable demand signal sharing with suppliers based on forecast, order book, and POS data |
| Collaborative Forecasting (CPFR) | Collaborative Planning, Forecasting & Replenishment with supplier joint planning workflows |
| Capacity Commitment | Supplier capacity commitment exchange with automated alerting on capacity constraints |
| Joint Inventory Planning | Multi-party inventory planning with VMI, consignment, and safety stock sharing |
| Supply Risk Network | Multi-tier supply risk visibility with automated escalation and alternative sourcing |
| Performance Benchmarking | Supplier performance benchmarking with industry peer comparison and best practice identification |
| ASN Management | Advanced Shipment Notice collaboration with automated receiving preparation |

## Database Tables

All tables include standard columns: `id UUID PK`, `tenant_id UUID`, `created_at TIMESTAMPTZ`, `updated_at TIMESTAMPTZ`, `created_by UUID`, `updated_by UUID`, `version INT`, `is_deleted BOOLEAN`.

| Table | Additional Columns |
|-------|-------------------|
| `customers` | `code VARCHAR(50)`, `name VARCHAR(255)`, `email VARCHAR(255)`, `phone VARCHAR(50)`, `customer_group_id UUID`, `pricing_tier_id UUID`, `credit_limit DECIMAL`, `status VARCHAR(20)`, `currency VARCHAR(3)` |
| `quotations` | `quote_number VARCHAR(50)`, `customer_id UUID`, `status VARCHAR(20)`, `valid_until DATE`, `total_amount DECIMAL`, `currency VARCHAR(3)`, `parent_quote_id UUID`, `approved_by UUID` |
| `orders` | `order_number VARCHAR(50)`, `customer_id UUID`, `quotation_id UUID`, `status VARCHAR(20)`, `total_amount DECIMAL`, `currency VARCHAR(3)`, `hold_reason VARCHAR(100)`, `channel VARCHAR(20)` |
| `order_items` | `order_id UUID`, `product_id UUID`, `quantity DECIMAL`, `unit_price DECIMAL`, `discount_amount DECIMAL`, `line_total DECIMAL`, `fulfilled_quantity DECIMAL` |
| `deliveries` | `order_id UUID`, `status VARCHAR(20)`, `scheduled_date DATE`, `delivered_date TIMESTAMPTZ`, `warehouse_id UUID`, `tracking_number VARCHAR(100)`, `pod_document_id UUID` |
| `shipments` | `delivery_id UUID`, `carrier_id UUID`, `status VARCHAR(20)`, `dispatched_at TIMESTAMPTZ`, `delivered_at TIMESTAMPTZ`, `tracking_number VARCHAR(100)`, `weight DECIMAL`, `freight_cost DECIMAL` |
| `products` | `sku VARCHAR(100)`, `name VARCHAR(255)`, `description TEXT`, `category_id UUID`, `base_price DECIMAL`, `uom VARCHAR(20)`, `status VARCHAR(20)`, `is_configurable BOOLEAN`, `pim_status VARCHAR(20)` |
| `product_categories` | `name VARCHAR(255)`, `parent_id UUID`, `level INT`, `attribute_schema JSONB` |
| `stock_levels` | `product_id UUID`, `warehouse_id UUID`, `quantity_on_hand DECIMAL`, `quantity_reserved DECIMAL`, `quantity_available DECIMAL`, `safety_stock DECIMAL`, `reorder_point DECIMAL`, `valuation_method VARCHAR(20)`, `unit_cost DECIMAL` |
| `warehouses` | `code VARCHAR(50)`, `name VARCHAR(255)`, `address JSONB`, `type VARCHAR(20)`, `status VARCHAR(20)`, `capacity DECIMAL` |
| `stock_transfers` | `transfer_number VARCHAR(50)`, `from_warehouse_id UUID`, `to_warehouse_id UUID`, `status VARCHAR(20)`, `shipped_date DATE`, `received_date DATE` |
| `pricing_rules` | `name VARCHAR(255)`, `rule_type VARCHAR(30)`, `priority INT`, `conditions JSONB`, `action JSONB`, `start_date DATE`, `end_date DATE`, `is_active BOOLEAN` |
| `credit_accounts` | `customer_id UUID`, `credit_limit DECIMAL`, `credit_used DECIMAL`, `credit_available DECIMAL`, `credit_score INT`, `status VARCHAR(20)`, `last_reviewed_at TIMESTAMPTZ` |
| `subscriptions` | `customer_id UUID`, `plan_id VARCHAR(100)`, `status VARCHAR(20)`, `billing_cycle VARCHAR(20)`, `start_date DATE`, `end_date DATE`, `amount DECIMAL`, `currency VARCHAR(3)`, `auto_renew BOOLEAN` |
| `loyalty_members` | `customer_id UUID`, `program_id UUID`, `tier VARCHAR(30)`, `points_balance INT`, `lifetime_points INT`, `joined_at TIMESTAMPTZ`, `last_activity_at TIMESTAMPTZ` |

## Events Published

| Event | Description |
|-------|-------------|
| `commerce.customer.created` | Customer created |
| `commerce.customer.updated` | Customer updated |
| `commerce.customer.deleted` | Customer soft-deleted |
| `commerce.quotation.created` | Quotation created |
| `commerce.quotation.accepted` | Quotation accepted |
| `commerce.quotation.rejected` | Quotation rejected |
| `commerce.order.created` | Sales order created |
| `commerce.order.submitted` | Sales order submitted for fulfillment |
| `commerce.order.fulfilled` | Order fulfilled |
| `commerce.order.cancelled` | Order cancelled |
| `commerce.order.returned` | Return/RMA created for order |
| `commerce.product.created` | Product created |
| `commerce.product.updated` | Product updated |
| `commerce.product.approved` | Product approved and published (PIM workflow) |
| `commerce.product.published` | Product published to channels |
| `commerce.stock.updated` | Stock level changed |
| `commerce.stock.reserved` | Stock reserved for an order (saga step) |
| `commerce.stock.reservation.failed` | Stock reservation failed (saga compensation trigger) |
| `commerce.stock.reservation.released` | Reserved stock released (saga compensation) |
| `commerce.transfer.initiated` | Stock transfer started |
| `commerce.transfer.completed` | Stock transfer completed |
| `commerce.pricing.calculated` | Price calculated by pricing engine |
| `commerce.delivery.created` | Delivery scheduled |
| `commerce.delivery.completed` | Delivery confirmed |
| `commerce.shipment.dispatched` | Shipment dispatched to carrier |
| `commerce.shipment.in-transit` | Shipment in transit (carrier tracking update) |
| `commerce.shipment.delivered` | Shipment delivered with POD |
| `commerce.carrier.assigned` | Carrier assigned to shipment |
| `commerce.credit.hold.applied` | Credit hold applied to order (credit limit exceeded) |
| `commerce.credit.hold.released` | Credit hold released on order |
| `commerce.atp.checked` | ATP/CTP availability check performed |
| `commerce.configurator.completed` | Product configuration completed |
| `commerce.subscription.created` | Subscription created |
| `commerce.subscription.amended` | Subscription amended (upgrade/downgrade) |
| `commerce.subscription.renewed` | Subscription renewed |
| `commerce.subscription.cancelled` | Subscription cancelled |
| `commerce.subscription.billing.completed` | Subscription billing cycle completed |
| `commerce.dropship.order.created` | Drop ship order created and dispatched to supplier |
| `commerce.dropship.order.delivered` | Drop ship order delivered to end customer |
| `commerce.b2b.order.placed` | B2B portal order placed by customer |
| `commerce.b2b.order.approved` | B2B portal order approved by customer approver |
| `commerce.logistics.tracking.updated` | Real-time tracking update received (GPS/geofence) |
| `commerce.logistics.condition.alert` | Shipment condition alert triggered (temperature, humidity, shock) |
| `commerce.logistics.eta.updated` | Predictive ETA recalculated for shipment |
| `commerce.logistics.geofence.entered` | Shipment entered geofence zone |
| `commerce.logistics.geofence.exited` | Shipment exited geofence zone |
| `commerce.logistics.exception.detected` | Logistics exception detected (delay, deviation, damage) |
| `commerce.warranty.created` | Warranty policy created |
| `commerce.warranty.claim.submitted` | Warranty claim submitted |
| `commerce.warranty.claim.approved` | Warranty claim approved |
| `commerce.warranty.claim.rejected` | Warranty claim rejected |
| `commerce.warranty.claim.fulfilled` | Warranty claim fulfilled (repair/replacement) |
| `commerce.loyalty.points.accrued` | Points accrued for customer |
| `commerce.loyalty.points.redeemed` | Points redeemed by customer |
| `commerce.loyalty.tier.changed` | Customer loyalty tier changed |
| `commerce.omnichannel.order.routed` | Order routed to fulfillment location |
| `commerce.price.optimized` | Price optimization suggestion generated |
| `commerce.collaboration.demand-shared` | Demand signal shared with supplier |
| `commerce.collaboration.capacity.committed` | Supplier capacity commitment received |
| `commerce.collaboration.cpfr.updated` | CPFR forecast updated collaboratively |
| `commerce.collaboration.asn.submitted` | Advanced Shipment Notice submitted by supplier |

## Events Consumed

Inbox binding: `commerce.inbox` binds to the following routing keys:

| Binding Pattern | Events Consumed |
|----------------|-----------------|
| `finance.purchase-order.#` | `finance.purchase-order.created`, `finance.purchase-order.approved`, `finance.purchase-order.received` |
| `finance.invoice.#` | `finance.invoice.created`, `finance.invoice.creation.failed`, `finance.invoice.paid`, `finance.invoice.credit-memo` |
| `finance.sourcing.#` | `finance.sourcing.event.created`, `finance.sourcing.bid.submitted`, `finance.sourcing.event.awarded` |
| `finance.cash-application.#` | `finance.cash-application.matched`, `finance.cash-application.unmatched` |
| `manufacturing.work-order.#` | `manufacturing.work-order.created`, `manufacturing.work-order.started`, `manufacturing.work-order.completed` |
| `manufacturing.digital-twin.#` | `manufacturing.digital-twin.state.updated`, `manufacturing.digital-twin.simulation.completed`, `manufacturing.digital-twin.prediction.generated` |
| `crm.opportunity.won` | `crm.opportunity.won` |
| `crm.cdp.#` | `crm.cdp.profile.created`, `crm.cdp.profile.updated`, `crm.cdp.profile.merged`, `crm.cdp.segment.updated`, `crm.cdp.journey.step.completed`, `crm.cdp.engagement-score.updated` |
| `config.changed` | `config.changed` |

## See Also

- [Finance Service](finance.md)
- [CRM Service](crm.md)
- [Manufacturing Service](manufacturing.md)
- [Report Service](report.md)
- [Architecture Overview](../01-architecture/overview.md)
