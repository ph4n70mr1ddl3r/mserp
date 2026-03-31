# API Endpoints

### Auth Service
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/v1/auth/login` | User login |
| POST | `/api/v1/auth/logout` | User logout (invalidates tokens) |
| POST | `/api/v1/auth/refresh` | Refresh tokens (rotation) |
| POST | `/api/v1/auth/password/reset` | Request password reset email |
| POST | `/api/v1/auth/password/confirm` | Confirm password reset |
| POST | `/api/v1/auth/mfa/enable` | Enable MFA for user |
| POST | `/api/v1/auth/mfa/verify` | Verify MFA code |
| POST | `/api/v1/auth/mfa/disable` | Disable MFA (requires password) |
| POST | `/api/v1/auth/step-up` | Step-up authentication (elevated privilege) |
| GET | `/api/v1/auth/sessions` | List active sessions for current user |
| DELETE | `/api/v1/auth/sessions/{id}` | Revoke specific session |
| DELETE | `/api/v1/auth/sessions` | Revoke all sessions except current |

### Identity Service
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/identity/users` | List users |
| POST | `/api/v1/identity/users` | Create user |
| GET | `/api/v1/identity/users/{id}` | Get user |
| PUT | `/api/v1/identity/users/{id}` | Update user |
| DELETE | `/api/v1/identity/users/{id}` | Soft-delete user |
| GET | `/api/v1/identity/roles` | List roles |
| POST | `/api/v1/identity/roles` | Create role |
| GET | `/api/v1/identity/roles/{id}/permissions` | List permissions for a role |
| PUT | `/api/v1/identity/roles/{id}/permissions` | Set permissions for a role (Tenant Admin only) |
| GET | `/api/v1/identity/groups` | List user groups |
| POST | `/api/v1/identity/groups` | Create user group |
| GET | `/api/v1/identity/groups/{id}/members` | List group members |
| POST | `/api/v1/identity/groups/{id}/members` | Add members to group |
| DELETE | `/api/v1/identity/groups/{id}/members/{userId}` | Remove member from group |
| GET | `/api/v1/identity/organizations` | List organizational units |
| POST | `/api/v1/identity/organizations` | Create organizational unit |
| POST | `/api/v1/identity/api-keys` | Create API key |
| GET | `/api/v1/identity/api-keys` | List API keys |
| DELETE | `/api/v1/identity/api-keys/{id}` | Revoke API key |

### Tenant Service
> Feature flag types, evaluation flow, and rules are specified in [Services — Feature Flags](../06-services/tenant.md).

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/tenant` | Get current tenant details |
| POST | `/api/v1/tenant` | Create tenant (Super Admin only) |
| PUT | `/api/v1/tenant` | Update tenant settings |
| GET | `/api/v1/tenant/features` | List feature flags |
| GET | `/api/v1/tenant/features/{flag}` | Get specific flag value |
| PUT | `/api/v1/tenant/features/{flag}` | Set flag value (Super Admin only) |
| DELETE | `/api/v1/tenant/features/{flag}` | Remove flag override |
| GET | `/api/v1/tenant/quotas` | Get tenant quota usage |
| GET | `/api/v1/tenant/subscriptions` | Get subscription details |
| PUT | `/api/v1/tenant/data-residency` | Update data residency settings (Super Admin only) |

### Config Service
> Config hierarchy, propagation, and schema are specified in [Services — Config Service](../06-services/config.md).

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/config` | List all config for current tenant |
| GET | `/api/v1/config/{key}` | Get specific config value |
| PUT | `/api/v1/config/{key}` | Set config value (Tenant Admin only) |
| DELETE | `/api/v1/config/{key}` | Reset to default |
| GET | `/api/v1/config/audit` | Config change history |
| POST | `/api/v1/config/cache/flush` | Flush tenant cache (Tenant Admin only) |
| GET | `/api/v1/config/cors/origins` | List allowed CORS origins |
| POST | `/api/v1/config/cors/origins` | Add allowed CORS origin (Tenant Admin only) |
| DELETE | `/api/v1/config/cors/origins` | Remove allowed CORS origin (Tenant Admin only) |
| GET | `/api/v1/config/locales` | List supported locales |
| GET | `/api/v1/config/locales/{locale}/translations` | Get translations for locale |

### Commerce Service (Sales + Inventory + PIM + Transportation)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/commerce/customers` | List customers |
| POST | `/api/v1/commerce/customers` | Create customer |
| GET | `/api/v1/commerce/customers/{id}` | Get customer |
| PUT | `/api/v1/commerce/customers/{id}` | Update customer |
| GET | `/api/v1/commerce/quotations` | List quotations |
| POST | `/api/v1/commerce/quotations` | Create quotation |
| GET | `/api/v1/commerce/quotations/{id}` | Get quotation details |
| POST | `/api/v1/commerce/quotations/{id}/submit` | Submit quotation |
| POST | `/api/v1/commerce/quotations/{id}/accept` | Accept quotation |
| POST | `/api/v1/commerce/quotations/{id}/reject` | Reject quotation |
| GET | `/api/v1/commerce/orders` | List sales orders |
| POST | `/api/v1/commerce/orders` | Create sales order |
| GET | `/api/v1/commerce/orders/{id}` | Get order details |
| PUT | `/api/v1/commerce/orders/{id}` | Update order (draft only) |
| POST | `/api/v1/commerce/orders/{id}/submit` | Submit order for fulfillment |
| POST | `/api/v1/commerce/orders/{id}/cancel` | Cancel order |
| GET | `/api/v1/commerce/orders/{id}/deliveries` | List deliveries for order |
| POST | `/api/v1/commerce/returns` | Create return (RMA) |
| GET | `/api/v1/commerce/products` | List products |
| POST | `/api/v1/commerce/products` | Create product |
| GET | `/api/v1/commerce/products/{id}` | Get product |
| PUT | `/api/v1/commerce/products/{id}` | Update product |
| POST | `/api/v1/commerce/products/bulk` | Bulk create products |
| POST | `/api/v1/commerce/products/{id}/submit-approval` | Submit product for PIM approval |
| POST | `/api/v1/commerce/products/{id}/publish` | Publish product to channels |
| GET | `/api/v1/commerce/price-lists` | List price lists |
| POST | `/api/v1/commerce/price-lists` | Create price list |
| POST | `/api/v1/commerce/pricing/calculate` | Calculate price for a product/customer/qty combination |
| GET | `/api/v1/commerce/warehouses` | List warehouses |
| GET | `/api/v1/commerce/stock` | Get stock levels |
| POST | `/api/v1/commerce/transfers` | Create stock transfer |
| POST | `/api/v1/commerce/adjustments` | Stock adjustment |
| GET | `/api/v1/commerce/carriers` | List carriers |
| POST | `/api/v1/commerce/carriers` | Register carrier |
| GET | `/api/v1/commerce/shipments` | List shipments |
| POST | `/api/v1/commerce/shipments` | Create shipment |
| GET | `/api/v1/commerce/shipments/{id}/tracking` | Get shipment tracking |
| POST | `/api/v1/commerce/shipments/{id}/dispatch` | Dispatch shipment to carrier |
| POST | `/api/v1/commerce/shipments/{id}/deliver` | Confirm delivery with POD |
| GET | `/api/v1/commerce/atp/check` | Check ATP/CTP availability |
| POST | `/api/v1/commerce/configurator/configure` | Configure a product |
| GET | `/api/v1/commerce/configurator/models` | List configurator models |
| GET | `/api/v1/commerce/credit/{customerId}` | Get customer credit status |
| PUT | `/api/v1/commerce/credit/{customerId}/limit` | Update credit limit |
| POST | `/api/v1/commerce/credit/{customerId}/hold` | Apply credit hold |
| POST | `/api/v1/commerce/credit/{customerId}/release` | Release credit hold |
| GET | `/api/v1/commerce/subscriptions` | List subscriptions |
| POST | `/api/v1/commerce/subscriptions` | Create subscription |
| GET | `/api/v1/commerce/subscriptions/{id}` | Get subscription |
| POST | `/api/v1/commerce/subscriptions/{id}/amend` | Amend subscription |
| POST | `/api/v1/commerce/subscriptions/{id}/cancel` | Cancel subscription |
| POST | `/api/v1/commerce/subscriptions/{id}/renew` | Renew subscription |
| POST | `/api/v1/commerce/dropship/orders` | Create drop ship order |
| POST | `/api/v1/commerce/b2b/portal/orders` | Create B2B portal order (customer self-service) |
| GET | `/api/v1/commerce/b2b/portal/orders` | List B2B portal orders |
| GET | `/api/v1/commerce/b2b/portal/reorder-templates` | List reorder templates |
| POST | `/api/v1/commerce/b2b/portal/reorder-templates` | Create reorder template |
| POST | `/api/v1/commerce/b2b/portal/reorder/{id}` | Execute reorder from template |
| GET | `/api/v1/commerce/logistics/tracking/{id}` | Get real-time shipment tracking with geofence data |
| GET | `/api/v1/commerce/logistics/condition/{shipmentId}` | Get shipment condition monitoring data |
| GET | `/api/v1/commerce/logistics/eta/{shipmentId}` | Get predictive ETA for shipment |
| POST | `/api/v1/commerce/logistics/geofences` | Create geofence for logistics monitoring |
| GET | `/api/v1/commerce/logistics/exceptions` | List logistics exceptions (delays, deviations) |
| GET | `/api/v1/commerce/warranties` | List warranty policies |
| POST | `/api/v1/commerce/warranties` | Create warranty policy |
| GET | `/api/v1/commerce/warranties/{id}` | Get warranty details |
| POST | `/api/v1/commerce/warranties/{id}/claim` | Submit warranty claim |
| GET | `/api/v1/commerce/warranties/{id}/claims` | List warranty claims |
| GET | `/api/v1/commerce/loyalty/programs` | List loyalty programs |
| POST | `/api/v1/commerce/loyalty/programs` | Create loyalty program |
| GET | `/api/v1/commerce/loyalty/programs/{id}/members` | List program members |
| POST | `/api/v1/commerce/loyalty/points/accrue` | Accrue loyalty points |
| POST | `/api/v1/commerce/loyalty/points/redeem` | Redeem loyalty points |
| GET | `/api/v1/commerce/loyalty/members/{customerId}` | Get customer loyalty status |
| GET | `/api/v1/commerce/omnichannel/channels` | List sales channels |
| POST | `/api/v1/commerce/omnichannel/channels` | Configure sales channel |
| GET | `/api/v1/commerce/omnichannel/cart` | Get unified cart |
| POST | `/api/v1/commerce/omnichannel/order/route` | Route order to fulfillment location |
| POST | `/api/v1/commerce/omnichannel/returns` | Create cross-channel return |
| GET | `/api/v1/commerce/omnichannel/bopis/availability` | Check BOPIS availability |
| POST | `/api/v1/commerce/omnichannel/ship-from-store` | Create ship-from-store order |
| GET | `/api/v1/commerce/price-optimization/demand-models` | List demand elasticity models |
| POST | `/api/v1/commerce/price-optimization/simulate` | Run price change simulation |
| GET | `/api/v1/commerce/price-optimization/recommendations` | Get price optimization recommendations |
| POST | `/api/v1/commerce/price-optimization/test` | Create A/B price test |
| GET | `/api/v1/commerce/price-optimization/tests/{id}/results` | Get price test results |
| GET | `/api/v1/commerce/collaboration/partners` | List supply chain collaboration partners |
| POST | `/api/v1/commerce/collaboration/demand-signals` | Submit demand signal |
| GET | `/api/v1/commerce/collaboration/demand-signals` | List demand signals |
| POST | `/api/v1/commerce/collaboration/capacity-commitments` | Create capacity commitment |
| GET | `/api/v1/commerce/orchestration/rules` | List orchestration rules |
| POST | `/api/v1/commerce/orchestration/rules` | Create orchestration rule |
| GET | `/api/v1/commerce/orchestration/processes/{id}` | Get orchestration process status |
| POST | `/api/v1/commerce/orchestration/processes/{id}/retry` | Retry failed orchestration step |
| GET | `/api/v1/commerce/backorders` | List backordered items |
| POST | `/api/v1/commerce/backorders/{id}/fulfill` | Fulfill backorder |
| GET | `/api/v1/commerce/gop/promise` | Get global order promising result |
| POST | `/api/v1/commerce/gop/sourcing-rules` | Create GOP sourcing rule |
| GET | `/api/v1/commerce/storefront/pages` | List storefront pages |
| POST | `/api/v1/commerce/storefront/pages` | Create storefront page |
| PUT | `/api/v1/commerce/storefront/pages/{id}` | Update storefront page |
| POST | `/api/v1/commerce/storefront/pages/{id}/publish` | Publish storefront page |
| GET | `/api/v1/commerce/storefront/seo` | Get SEO metadata |
| PUT | `/api/v1/commerce/storefront/seo/{pageId}` | Update SEO metadata |
| GET | `/api/v1/commerce/promotions` | List promotions |
| POST | `/api/v1/commerce/promotions` | Create promotion |
| POST | `/api/v1/commerce/reviews` | Submit product review |
| GET | `/api/v1/commerce/reviews` | List product reviews |
| POST | `/api/v1/commerce/warehouse/zones` | Create warehouse zone |
| GET | `/api/v1/commerce/warehouse/zones` | List warehouse zones |
| GET | `/api/v1/commerce/warehouse/zones/{id}` | Get warehouse zone |
| PUT | `/api/v1/commerce/warehouse/zones/{id}` | Update warehouse zone |
| POST | `/api/v1/commerce/warehouse/waves` | Create pick wave |
| GET | `/api/v1/commerce/warehouse/waves` | List pick waves |
| GET | `/api/v1/commerce/warehouse/waves/{id}` | Get pick wave |
| POST | `/api/v1/commerce/warehouse/waves/{id}/release` | Release wave for picking |
| POST | `/api/v1/commerce/warehouse/picks/{id}/complete` | Complete pick task |
| POST | `/api/v1/commerce/warehouse/pack` | Complete pack operation |
| POST | `/api/v1/commerce/warehouse/putaway` | Complete putaway |
| GET | `/api/v1/commerce/warehouse/cycle-counts` | List cycle counts |
| POST | `/api/v1/commerce/warehouse/cycle-counts` | Create cycle count |
| POST | `/api/v1/commerce/transportation/loads` | Plan load |
| GET | `/api/v1/commerce/transportation/loads` | List loads |
| GET | `/api/v1/commerce/transportation/loads/{id}` | Get load |
| POST | `/api/v1/commerce/transportation/carrier-selection` | Select carrier |
| GET | `/api/v1/commerce/transportation/shipments/{id}/track` | Track shipment |
| POST | `/api/v1/commerce/transportation/freight-audit` | Audit freight charges |

### Finance Service (Finance + Procurement + Treasury + Expenses + CLM + EPM)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/finance/accounts` | List chart of accounts |
| POST | `/api/v1/finance/accounts` | Create account |
| GET | `/api/v1/finance/journals` | List journal entries |
| POST | `/api/v1/finance/journals` | Create journal entry |
| POST | `/api/v1/finance/journals/{id}/post` | Post journal entry |
| POST | `/api/v1/finance/journals/{id}/reverse` | Reverse posted journal entry |
| GET | `/api/v1/finance/reports/balance-sheet` | Balance sheet report |
| GET | `/api/v1/finance/reports/profit-loss` | P&L report |
| GET | `/api/v1/finance/reports/cash-flow` | Cash flow statement |
| GET | `/api/v1/finance/reports/trial-balance` | Trial balance |
| GET | `/api/v1/finance/budgets` | List budgets |
| POST | `/api/v1/finance/budgets` | Create budget |
| GET | `/api/v1/finance/budgets/{id}/vs-actual` | Budget vs. actual report |
| GET | `/api/v1/finance/exchange-rates` | Get exchange rates |
| PUT | `/api/v1/finance/exchange-rates` | Update exchange rates |
| GET | `/api/v1/finance/periods` | List accounting periods |
| POST | `/api/v1/finance/periods/{id}/close` | Close accounting period |
| GET | `/api/v1/finance/suppliers` | List suppliers |
| POST | `/api/v1/finance/suppliers` | Create supplier |
| GET | `/api/v1/finance/suppliers/{id}` | Get supplier |
| PUT | `/api/v1/finance/suppliers/{id}` | Update supplier |
| GET | `/api/v1/finance/purchase-orders` | List purchase orders |
| POST | `/api/v1/finance/purchase-orders` | Create purchase order |
| GET | `/api/v1/finance/purchase-orders/{id}` | Get purchase order |
| POST | `/api/v1/finance/purchase-orders/{id}/receive` | Receive goods |
| GET | `/api/v1/finance/requisitions` | List purchase requisitions |
| POST | `/api/v1/finance/requisitions` | Create purchase requisition |
| GET | `/api/v1/finance/treasury/cash-position` | Get current cash position |
| GET | `/api/v1/finance/treasury/bank-accounts` | List bank accounts |
| POST | `/api/v1/finance/treasury/payment-batches` | Create payment batch |
| POST | `/api/v1/finance/treasury/payment-batches/{id}/approve` | Approve payment batch |
| POST | `/api/v1/finance/treasury/payment-batches/{id}/execute` | Execute payment batch |
| GET | `/api/v1/finance/expenses` | List expense reports |
| POST | `/api/v1/finance/expenses` | Create expense report |
| GET | `/api/v1/finance/expenses/{id}` | Get expense report |
| POST | `/api/v1/finance/expenses/{id}/submit` | Submit for approval |
| POST | `/api/v1/finance/expenses/{id}/approve` | Approve expense report |
| POST | `/api/v1/finance/expenses/{id}/reject` | Reject expense report |
| GET | `/api/v1/finance/contracts` | List contracts |
| POST | `/api/v1/finance/contracts` | Create contract |
| GET | `/api/v1/finance/contracts/{id}` | Get contract |
| PUT | `/api/v1/finance/contracts/{id}` | Update contract |
| POST | `/api/v1/finance/contracts/{id}/submit` | Submit for approval |
| POST | `/api/v1/finance/contracts/{id}/approve` | Approve contract |
| POST | `/api/v1/finance/contracts/{id}/sign` | Record e-signature |
| GET | `/api/v1/finance/contracts/{id}/obligations` | List contract obligations |
| GET | `/api/v1/finance/plans` | List financial plans (EPM) |
| POST | `/api/v1/finance/plans` | Create financial plan |
| PUT | `/api/v1/finance/plans/{id}` | Update plan values |
| POST | `/api/v1/finance/plans/{id}/forecast` | Generate forecast |
| POST | `/api/v1/finance/plans/{id}/what-if` | Run what-if scenario |
| GET | `/api/v1/finance/plans/{id}/variance` | Get variance analysis |
| GET | `/api/v1/finance/revenue/contracts` | List revenue recognition contracts |
| POST | `/api/v1/finance/revenue/contracts` | Create revenue contract |
| GET | `/api/v1/finance/revenue/contracts/{id}/obligations` | List performance obligations |
| GET | `/api/v1/finance/revenue/contracts/{id}/schedule` | Get revenue recognition schedule |
| POST | `/api/v1/finance/revenue/contracts/{id}/recognize` | Recognize revenue for period |
| GET | `/api/v1/finance/revenue/waterfall` | Get revenue waterfall report |
| GET | `/api/v1/finance/revenue/disclosures` | Get revenue disclosure data |
| POST | `/api/v1/finance/sourcing/events` | Create sourcing event (RFI/RFP/RFQ) |
| GET | `/api/v1/finance/sourcing/events` | List sourcing events |
| GET | `/api/v1/finance/sourcing/events/{id}/bids` | List bids for sourcing event |
| POST | `/api/v1/finance/sourcing/events/{id}/award` | Award sourcing event |
| GET | `/api/v1/finance/supplier-risk/scores` | List supplier risk scores |
| GET | `/api/v1/finance/supplier-risk/scores/{supplierId}` | Get supplier risk detail |
| GET | `/api/v1/finance/supplier-risk/alerts` | List supplier risk alerts |
| POST | `/api/v1/finance/supplier-risk/mitigation-plans` | Create risk mitigation plan |
| GET | `/api/v1/finance/reconciliation/matches` | List reconciliation matches |
| POST | `/api/v1/finance/reconciliation/auto-match` | Run auto-matching for account |
| POST | `/api/v1/finance/reconciliation/exceptions` | Create reconciliation exception |
| GET | `/api/v1/finance/close/tasks` | List financial close tasks |
| POST | `/api/v1/finance/close/tasks` | Create close task |
| POST | `/api/v1/finance/close/tasks/{id}/complete` | Mark close task complete |
| GET | `/api/v1/finance/profitability/customer` | Get customer profitability analysis |
| GET | `/api/v1/finance/profitability/product` | Get product profitability analysis |
| GET | `/api/v1/finance/profitability/cost-to-serve/{customerId}` | Get cost-to-serve analysis |
| POST | `/api/v1/finance/profitability/what-if` | Run profitability what-if scenario |
| GET | `/api/v1/finance/leases` | List lease contracts |
| POST | `/api/v1/finance/leases` | Create lease contract |
| GET | `/api/v1/finance/leases/{id}` | Get lease details |
| POST | `/api/v1/finance/leases/{id}/amend` | Amend lease |
| GET | `/api/v1/finance/leases/{id}/schedule` | Get lease payment schedule |
| POST | `/api/v1/finance/leases/{id}/recognize` | Recognize lease expense for period |
| GET | `/api/v1/finance/leases/{id}/right-of-use` | Get right-of-use asset data |
| GET | `/api/v1/finance/grants` | List grants |
| POST | `/api/v1/finance/grants` | Create grant |
| GET | `/api/v1/finance/grants/{id}` | Get grant details |
| POST | `/api/v1/finance/grants/{id}/recognize` | Recognize grant revenue |
| GET | `/api/v1/finance/grants/{id}/compliance` | Get grant compliance status |
| GET | `/api/v1/finance/joint-ventures` | List joint ventures |
| POST | `/api/v1/finance/joint-ventures` | Create joint venture |
| GET | `/api/v1/finance/joint-ventures/{id}` | Get joint venture details |
| POST | `/api/v1/finance/joint-ventures/{id}/allocate` | Allocate joint venture costs |
| GET | `/api/v1/finance/joint-ventures/{id}/billing` | Get partner billing statements |
| GET | `/api/v1/finance/intelligent-close/status` | Get financial close automation status |
| POST | `/api/v1/finance/intelligent-close/tasks/auto-assign` | Auto-assign close tasks |
| GET | `/api/v1/finance/intelligent-close/anomalies` | Get close anomaly detection results |
| GET | `/api/v1/finance/collections/aging` | Get accounts receivable aging |
| POST | `/api/v1/finance/collections/strategies` | Create collection strategy |
| GET | `/api/v1/finance/collections/activities` | List collection activities |
| POST | `/api/v1/finance/collections/cash-application` | Apply cash receipt to invoices |
| GET | `/api/v1/finance/tax/assessments` | List tax assessments |
| POST | `/api/v1/finance/tax/calculate` | Calculate tax for transaction |
| GET | `/api/v1/finance/tax/jurisdictions` | List tax jurisdictions |
| PUT | `/api/v1/finance/tax/jurisdictions/{id}/rates` | Update jurisdiction tax rates |
| GET | `/api/v1/finance/tax/exemptions` | List tax exemptions |
| POST | `/api/v1/finance/tax/exemptions` | Create tax exemption certificate |
| GET | `/api/v1/finance/tax/nexus` | List nexus tracking data |
| POST | `/api/v1/finance/tax/reconciliation` | Run tax reconciliation |
| POST | `/api/v1/finance/commodity/catalog` | Create commodity catalog entry |
| GET | `/api/v1/finance/commodity/prices` | Get commodity market prices |
| GET | `/api/v1/finance/commodity/contracts` | List commodity contracts |
| POST | `/api/v1/finance/commodity/contracts` | Create commodity contract |
| GET | `/api/v1/finance/commodity/hedging/positions` | List hedging positions |
| GET | `/api/v1/finance/spend/classification` | Get spend classification results |
| GET | `/api/v1/finance/spend/analysis` | Get spend analysis dashboard |
| POST | `/api/v1/finance/spend/classify` | Trigger ML spend classification |
| GET | `/api/v1/finance/spend/maverick` | Identify maverick spend |
| GET | `/api/v1/finance/diversity/classifications` | List supplier diversity classifications |
| GET | `/api/v1/finance/diversity/spend` | Get diverse supplier spend data |
| GET | `/api/v1/finance/diversity/tier-report` | Generate tier diversity report |
| POST | `/api/v1/finance/diversity/goals` | Set diversity spend goals |
| GET | `/api/v1/finance/discounting/programs` | List dynamic discounting programs |
| POST | `/api/v1/finance/discounting/programs` | Create dynamic discounting program |
| GET | `/api/v1/finance/discounting/programs/{id}` | Get discounting program details |
| POST | `/api/v1/finance/discounting/programs/{id}/simulate` | Simulate discounting program scenario |
| GET | `/api/v1/finance/discounting/working-capital` | Get working capital position |

### HCM Service
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/hr/employees` | List employees |
| POST | `/api/v1/hr/employees` | Create employee |
| GET | `/api/v1/hr/employees/{id}` | Get employee |
| PUT | `/api/v1/hr/employees/{id}` | Update employee |
| GET | `/api/v1/hr/attendance` | Get attendance records |
| POST | `/api/v1/hr/attendance/check-in` | Check in |
| POST | `/api/v1/hr/attendance/check-out` | Check out |
| GET | `/api/v1/hr/leaves` | List leave requests |
| POST | `/api/v1/hr/leaves` | Create leave request |
| POST | `/api/v1/hr/leaves/{id}/approve` | Approve leave |
| POST | `/api/v1/hr/leaves/{id}/reject` | Reject leave |
| GET | `/api/v1/hr/leaves/balances` | Get leave balances |
| GET | `/api/v1/hr/payroll/runs` | List payroll runs |
| POST | `/api/v1/hr/payroll/runs` | Start payroll run |
| POST | `/api/v1/hr/payroll/runs/{id}/confirm` | Confirm and post payroll |
| GET | `/api/v1/hr/payroll/countries` | List supported payroll countries |
| POST | `/api/v1/hr/payroll/multi-country/run` | Start multi-country payroll cycle |
| GET | `/api/v1/hr/recruitment/requisitions` | List job requisitions |
| POST | `/api/v1/hr/recruitment/requisitions` | Create job requisition |
| GET | `/api/v1/hr/recruitment/applicants` | List applicants |
| POST | `/api/v1/hr/performance/reviews` | Create performance review |
| GET | `/api/v1/hr/org-chart` | Get organization chart |
| GET | `/api/v1/hr/training/courses` | List training courses |
| GET | `/api/v1/hr/talent/reviews` | List talent reviews |
| POST | `/api/v1/hr/talent/reviews` | Initiate talent review |
| GET | `/api/v1/hr/talent/succession-plans` | List succession plans |
| POST | `/api/v1/hr/talent/succession-plans` | Create succession plan |
| POST | `/api/v1/hr/workforce/simulate` | Run workforce modeling simulation |
| GET | `/api/v1/hr/benefits/plans` | List benefit plans |
| POST | `/api/v1/hr/benefits/enroll` | Enroll employee in benefits |
| GET | `/api/v1/hr/self-service/portal` | Get employee self-service portal data |
| GET | `/api/v1/hr/self-service/payslips` | List payslips (self-service) |
| POST | `/api/v1/hr/self-service/time-off` | Submit time-off request |
| GET | `/api/v1/hr/self-service/benefits` | List available benefits |
| POST | `/api/v1/hr/self-service/expenses` | Submit expense (self-service) |
| GET | `/api/v1/hr/goals` | List goals |
| POST | `/api/v1/hr/goals` | Create goal |
| GET | `/api/v1/hr/goals/{id}` | Get goal details |
| PUT | `/api/v1/hr/goals/{id}/progress` | Update goal progress |
| GET | `/api/v1/hr/goals/alignment` | Get goal cascade/alignment tree |
| GET | `/api/v1/hr/career/paths` | List career paths |
| POST | `/api/v1/hr/career/plans` | Create development plan |
| GET | `/api/v1/hr/career/plans/{id}` | Get development plan |
| GET | `/api/v1/hr/competencies` | List competency frameworks |
| POST | `/api/v1/hr/competencies/assess` | Submit competency assessment |
| GET | `/api/v1/hr/onboarding/journeys` | List onboarding journeys |
| POST | `/api/v1/hr/onboarding/journeys` | Create onboarding journey from template |
| GET | `/api/v1/hr/onboarding/journeys/{id}` | Get onboarding journey status |
| POST | `/api/v1/hr/onboarding/journeys/{id}/tasks/{taskId}/complete` | Complete onboarding task |

### Manufacturing Service
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/manufacturing/boms` | List BOMs |
| POST | `/api/v1/manufacturing/boms` | Create BOM |
| GET | `/api/v1/manufacturing/boms/{id}` | Get BOM with components |
| GET | `/api/v1/manufacturing/work-orders` | List work orders |
| POST | `/api/v1/manufacturing/work-orders` | Create work order |
| GET | `/api/v1/manufacturing/work-orders/{id}` | Get work order |
| POST | `/api/v1/manufacturing/work-orders/{id}/start` | Start work order |
| POST | `/api/v1/manufacturing/work-orders/{id}/complete` | Complete work order |
| GET | `/api/v1/manufacturing/quality/inspections` | List quality inspections |
| POST | `/api/v1/manufacturing/quality/inspections` | Create quality inspection |
| GET | `/api/v1/manufacturing/quality/spc` | Get SPC charts and data |
| GET | `/api/v1/manufacturing/work-centers` | List work centers |
| GET | `/api/v1/manufacturing/routing` | List routing definitions |
| GET | `/api/v1/manufacturing/plm/revisions` | List product revisions |
| POST | `/api/v1/manufacturing/plm/revisions` | Create product revision |
| GET | `/api/v1/manufacturing/plm/eco` | List engineering change orders |
| POST | `/api/v1/manufacturing/plm/eco` | Create ECO |
| POST | `/api/v1/manufacturing/plm/eco/{id}/approve` | Approve ECO |
| POST | `/api/v1/manufacturing/plm/eco/{id}/implement` | Implement ECO |
| GET | `/api/v1/manufacturing/assets` | List enterprise assets |
| POST | `/api/v1/manufacturing/assets` | Register asset |
| GET | `/api/v1/manufacturing/assets/{id}` | Get asset details |
| POST | `/api/v1/manufacturing/assets/{id}/maintenance` | Schedule asset maintenance |
| GET | `/api/v1/manufacturing/assets/{id}/history` | Get asset maintenance history |
| GET | `/api/v1/manufacturing/planning/mrp` | Get MRP results |
| POST | `/api/v1/manufacturing/planning/mrp/run` | Run MRP |
| POST | `/api/v1/manufacturing/planning/simulate` | Run ASCP simulation |
| GET | `/api/v1/manufacturing/sustainability/emissions` | Get production emissions data |
| GET | `/api/v1/manufacturing/iot/devices` | List IoT devices |
| POST | `/api/v1/manufacturing/iot/devices/_sync` | Sync device cache from Platform registry |
| GET | `/api/v1/manufacturing/iot/devices/{id}/telemetry` | Get device telemetry |
| POST | `/api/v1/manufacturing/iot/alerts` | Create IoT alert rule |
| GET | `/api/v1/manufacturing/digital-twins` | List digital twins |
| GET | `/api/v1/manufacturing/digital-twins/{assetId}` | Get asset digital twin state |
| POST | `/api/v1/manufacturing/digital-twins/{assetId}/simulate` | Run digital twin simulation |
| GET | `/api/v1/manufacturing/digital-twins/{assetId}/predictions` | Get predictive maintenance predictions |
| GET | `/api/v1/manufacturing/intelligence/oee` | Get OEE metrics |
| GET | `/api/v1/manufacturing/intelligence/downtime` | Get downtime analysis |
| GET | `/api/v1/manufacturing/digital-thread/traceability/{serialId}` | Get full traceability for serial/lot |
| GET | `/api/v1/manufacturing/digital-thread/genealogy/{productId}` | Get product genealogy |
| GET | `/api/v1/manufacturing/intelligence/capacity` | Get capacity utilization |
| GET | `/api/v1/manufacturing/intelligence/production-rate` | Get production rate tracking |
| GET | `/api/v1/manufacturing/intelligence/energy` | Get energy analytics |
| GET | `/api/v1/manufacturing/intelligence/predictive-maintenance` | Get predictive maintenance analytics |
| POST | `/api/v1/manufacturing/ehs/incidents` | Report safety incident |
| GET | `/api/v1/manufacturing/ehs/incidents` | List safety incidents |
| GET | `/api/v1/manufacturing/ehs/incidents/{id}` | Get incident details |
| POST | `/api/v1/manufacturing/ehs/risk-assessments` | Create risk assessment |
| POST | `/api/v1/manufacturing/ehs/inspections` | Schedule safety inspection |
| GET | `/api/v1/manufacturing/ehs/chemicals/sds` | List Safety Data Sheets |
| POST | `/api/v1/manufacturing/ehs/permits` | Create permit-to-work |
| GET | `/api/v1/manufacturing/ehs/training/compliance` | Get safety training compliance |
| GET | `/api/v1/manufacturing/mro/catalog` | List MRO catalog items |
| POST | `/api/v1/manufacturing/mro/catalog` | Create MRO catalog item |
| GET | `/api/v1/manufacturing/mro/spare-parts` | List spare parts inventory |
| POST | `/api/v1/manufacturing/mro/repair-orders` | Create repair order |
| GET | `/api/v1/manufacturing/mro/repair-orders/{id}` | Get repair order |
| POST | `/api/v1/manufacturing/mro/overhaul` | Create overhaul project |
| GET | `/api/v1/manufacturing/mro/rotable` | List rotable components |
| GET | `/api/v1/manufacturing/compliance/regulations` | List regulatory requirements |
| GET | `/api/v1/manufacturing/compliance/certifications` | List product certifications |
| POST | `/api/v1/manufacturing/compliance/certifications` | Register certification |
| GET | `/api/v1/manufacturing/compliance/material-declarations` | List material declarations |
| POST | `/api/v1/manufacturing/compliance/tests` | Create compliance test plan |
| GET | `/api/v1/manufacturing/mes/dispatch` | Get work center dispatch list |
| POST | `/api/v1/manufacturing/mes/start` | Start production at station |
| POST | `/api/v1/manufacturing/mes/complete` | Complete production at station |
| POST | `/api/v1/manufacturing/mes/labor` | Log labor transaction |
| GET | `/api/v1/manufacturing/mes/instructions/{workOrderId}` | Get work instructions |
| GET | `/api/v1/manufacturing/mes/shifts` | Get shift schedules |
| POST | `/api/v1/manufacturing/mes/data-collection` | Submit production data |
| GET | `/api/v1/manufacturing/mes/oee/station` | Get station-level OEE |
| GET | `/api/v1/manufacturing/schedules` | List production schedules |
| POST | `/api/v1/manufacturing/schedules` | Create production schedule |
| POST | `/api/v1/manufacturing/schedules/{id}/simulate` | Simulate schedule (what-if) |
| POST | `/api/v1/manufacturing/schedules/{id}/firm` | Firm production schedule |
| GET | `/api/v1/manufacturing/schedules/gantt` | Get scheduling Gantt data |

### CRM Service
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/crm/contacts` | List contacts |
| POST | `/api/v1/crm/contacts` | Create contact |
| GET | `/api/v1/crm/leads` | List leads |
| POST | `/api/v1/crm/leads` | Create lead |
| POST | `/api/v1/crm/leads/{id}/convert` | Convert lead to opportunity/customer |
| GET | `/api/v1/crm/opportunities` | List opportunities |
| POST | `/api/v1/crm/opportunities` | Create opportunity |
| PUT | `/api/v1/crm/opportunities/{id}/stage` | Update opportunity stage |
| GET | `/api/v1/crm/campaigns` | List campaigns |
| POST | `/api/v1/crm/campaigns` | Create campaign |
| GET | `/api/v1/crm/activities` | List activities |
| POST | `/api/v1/crm/activities` | Log activity |
| GET | `/api/v1/crm/analytics/pipeline` | Pipeline analytics |
| GET | `/api/v1/crm/cases` | List service cases |
| POST | `/api/v1/crm/cases` | Create service case |
| POST | `/api/v1/crm/cases/{id}/resolve` | Resolve service case |
| GET | `/api/v1/crm/service-orders` | List service orders |
| POST | `/api/v1/crm/service-orders` | Create service order |
| GET | `/api/v1/crm/service-orders/{id}` | Get service order |
| POST | `/api/v1/crm/service-orders/{id}/dispatch` | Dispatch technician |
| POST | `/api/v1/crm/service-orders/{id}/complete` | Complete service order |
| GET | `/api/v1/crm/surveys` | List surveys |
| POST | `/api/v1/crm/surveys` | Create survey |
| POST | `/api/v1/crm/surveys/{id}/responses` | Submit survey response |
| GET | `/api/v1/crm/surveys/{id}/analytics` | Get survey analytics |
| GET | `/api/v1/crm/territories` | List sales territories |
| POST | `/api/v1/crm/territories` | Create territory |
| PUT | `/api/v1/crm/territories/{id}/assign` | Assign accounts to territory |
| GET | `/api/v1/crm/quotas` | List sales quotas |
| POST | `/api/v1/crm/quotas` | Allocate quotas |
| GET | `/api/v1/crm/cdp/profiles` | List unified customer profiles (CDP) |
| GET | `/api/v1/crm/cdp/profiles/{id}` | Get unified customer profile |
| POST | `/api/v1/crm/cdp/segments` | Create customer segment |
| GET | `/api/v1/crm/cdp/segments` | List customer segments |
| POST | `/api/v1/crm/cdp/journeys` | Create customer journey |
| GET | `/api/v1/crm/cdp/journeys/{id}/analytics` | Get journey analytics |
| GET | `/api/v1/crm/cdp/identity-resolution/{id}` | Get identity resolution graph |
| GET | `/api/v1/crm/contact-center/interactions` | List contact center interactions |
| POST | `/api/v1/crm/contact-center/interactions` | Create interaction |
| GET | `/api/v1/crm/contact-center/queues` | List contact center queues |
| PUT | `/api/v1/crm/contact-center/queues/{id}` | Update queue configuration |
| GET | `/api/v1/crm/contact-center/agents` | List agent statuses |
| GET | `/api/v1/crm/contact-center/scripts` | List agent scripts |
| POST | `/api/v1/crm/contact-center/outbound-campaigns` | Create outbound campaign |
| GET | `/api/v1/crm/contact-center/analytics` | Get contact center analytics |
| GET | `/api/v1/crm/social/profiles/{contactId}` | Get social profile enrichment |
| POST | `/api/v1/crm/social/engage` | Track social engagement |
| GET | `/api/v1/crm/social/intelligence` | Get social selling index |
| GET | `/api/v1/crm/social/relationships` | Get relationship mapping |
| GET | `/api/v1/crm/ab/tests` | List A/B tests |
| POST | `/api/v1/crm/ab/tests` | Create A/B test |
| GET | `/api/v1/crm/ab/tests/{id}` | Get A/B test details |
| POST | `/api/v1/crm/ab/tests/{id}/start` | Start A/B test |
| POST | `/api/v1/crm/ab/tests/{id}/stop` | Stop A/B test |
| GET | `/api/v1/crm/ab/tests/{id}/results` | Get A/B test results |

### Project Management Service
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/projects` | List projects |
| POST | `/api/v1/projects` | Create project |
| GET | `/api/v1/projects/{id}` | Get project details |
| PUT | `/api/v1/projects/{id}` | Update project |
| GET | `/api/v1/projects/{id}/tasks` | List project tasks |
| POST | `/api/v1/projects/{id}/tasks` | Create task |
| PUT | `/api/v1/projects/{id}/tasks/{taskId}` | Update task |
| GET | `/api/v1/projects/{id}/timeline` | Get Gantt/timeline data |
| GET | `/api/v1/projects/{id}/resources` | List allocated resources |
| POST | `/api/v1/projects/{id}/resources` | Allocate resource |
| POST | `/api/v1/projects/{id}/timesheets` | Submit timesheet entry |
| GET | `/api/v1/projects/{id}/expenses` | List project expenses |
| POST | `/api/v1/projects/{id}/expenses` | Submit expense |
| GET | `/api/v1/projects/{id}/billing` | Get billing/invoicing status |
| POST | `/api/v1/projects/{id}/billing/invoice` | Generate project invoice |
| GET | `/api/v1/projects/{id}/risks` | List project risks |
| POST | `/api/v1/projects/{id}/risks` | Register risk |
| GET | `/api/v1/projects/{id}/evm` | Get Earned Value Management metrics |
| GET | `/api/v1/programs` | List programs |
| POST | `/api/v1/programs` | Create program |

### Platform Service (Notification + File + Audit + Digital Assistant + App Builder + GRC)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/platform/notifications` | List notifications |
| POST | `/api/v1/platform/notifications` | Send notification |
| PUT | `/api/v1/platform/notifications/{id}/read` | Mark notification read |
| POST | `/api/v1/platform/files/upload` | Upload file (multipart) |
| GET | `/api/v1/platform/files/{id}` | Get file metadata |
| GET | `/api/v1/platform/files/{id}/download` | Download file |
| DELETE | `/api/v1/platform/files/{id}` | Soft-delete file |
| GET | `/api/v1/platform/files/{id}/preview` | Get document preview |
| POST | `/api/v1/platform/files/{id}/ocr` | Extract text via OCR |
| GET | `/api/v1/platform/audit` | Query audit log |
| GET | `/api/v1/platform/audit/entity/{type}/{id}` | Get audit trail for entity |
| POST | `/api/v1/platform/assistant/query` | Query digital assistant (natural language) |
| GET | `/api/v1/platform/assistant/history` | Get conversation history |
| POST | `/api/v1/platform/assistant/feedback` | Submit feedback on assistant response |
| GET | `/api/v1/platform/app-builder/apps` | List custom applications |
| POST | `/api/v1/platform/app-builder/apps` | Create custom application |
| GET | `/api/v1/platform/app-builder/apps/{id}` | Get application definition |
| PUT | `/api/v1/platform/app-builder/apps/{id}` | Update application |
| POST | `/api/v1/platform/app-builder/apps/{id}/publish` | Publish application |
| POST | `/api/v1/platform/app-builder/objects` | Create custom business object |
| GET | `/api/v1/platform/app-builder/objects` | List custom business objects |
| GET | `/api/v1/platform/grc/sod-rules` | List SoD rules |
| POST | `/api/v1/platform/grc/sod-rules` | Create SoD rule |
| GET | `/api/v1/platform/grc/sod-conflicts` | List SoD conflicts |
| GET | `/api/v1/platform/grc/risks` | List GRC risks |
| POST | `/api/v1/platform/grc/risks` | Register GRC risk |
| POST | `/api/v1/platform/grc/assessments` | Run compliance assessment |
| GET | `/api/v1/platform/grc/incidents` | List GRC incidents |
| POST | `/api/v1/platform/grc/incidents` | Create GRC incident |
| GET | `/api/v1/platform/data-masking/rules` | List data masking rules |
| POST | `/api/v1/platform/data-masking/rules` | Create data masking rule |
| POST | `/api/v1/platform/data-masking/subset` | Generate masked data subset for non-prod |
| GET | `/api/v1/platform/scheduler/jobs` | List scheduled jobs |
| POST | `/api/v1/platform/scheduler/jobs` | Create scheduled job |
| GET | `/api/v1/platform/scheduler/jobs/{id}` | Get job definition |
| PUT | `/api/v1/platform/scheduler/jobs/{id}` | Update job definition |
| POST | `/api/v1/platform/scheduler/jobs/{id}/trigger` | Trigger manual job execution |
| GET | `/api/v1/platform/scheduler/jobs/{id}/executions` | Get job execution history |
| GET | `/api/v1/platform/knowledge/articles` | List knowledge articles |
| POST | `/api/v1/platform/knowledge/articles` | Create knowledge article |
| GET | `/api/v1/platform/knowledge/articles/{id}` | Get article |
| PUT | `/api/v1/platform/knowledge/articles/{id}` | Update article |
| POST | `/api/v1/platform/knowledge/articles/{id}/publish` | Publish article |
| GET | `/api/v1/platform/knowledge/search` | Search knowledge base |
| POST | `/api/v1/platform/signatures/request` | Request digital signature |
| GET | `/api/v1/platform/signatures/{id}` | Get signature status |
| POST | `/api/v1/platform/signatures/{id}/sign` | Sign document |
| GET | `/api/v1/platform/rpa/bots` | List RPA bots |
| POST | `/api/v1/platform/rpa/bots` | Create RPA bot |
| GET | `/api/v1/platform/rpa/bots/{id}` | Get bot definition |
| POST | `/api/v1/platform/rpa/bots/{id}/execute` | Execute RPA bot |
| GET | `/api/v1/platform/rpa/bots/{id}/executions` | Get bot execution history |
| GET | `/api/v1/platform/collaboration/channels` | List collaboration channels |
| POST | `/api/v1/platform/collaboration/channels` | Create channel |
| POST | `/api/v1/platform/collaboration/channels/{id}/messages` | Send message |
| GET | `/api/v1/platform/collaboration/channels/{id}/messages` | List channel messages |
| GET | `/api/v1/platform/collaboration/tasks` | List collaboration tasks |
| POST | `/api/v1/platform/collaboration/tasks` | Create collaboration task |
| GET | `/api/v1/platform/iot/devices` | List IoT devices (device registry) |
| POST | `/api/v1/platform/iot/devices` | Register IoT device |
| GET | `/api/v1/platform/iot/devices/{id}` | Get device details |
| POST | `/api/v1/platform/iot/devices/{id}/certificates` | Issue device certificate |

> **Note:** Employee Self-Service Portal endpoints are documented under HCM Service. Platform Service provides notification and file storage infrastructure.

| GET | `/api/v1/platform/content/repositories` | List content repositories |
| POST | `/api/v1/platform/content/repositories` | Create content repository |
| GET | `/api/v1/platform/content/documents` | List documents |
| POST | `/api/v1/platform/content/documents` | Create document |
| GET | `/api/v1/platform/content/documents/{id}` | Get document |
| PUT | `/api/v1/platform/content/documents/{id}` | Update document metadata |
| POST | `/api/v1/platform/content/documents/{id}/lifecycle` | Transition document lifecycle state |
| GET | `/api/v1/platform/content/records/retention-policies` | List retention policies |
| POST | `/api/v1/platform/content/records/retention-policies` | Create retention policy |
| GET | `/api/v1/platform/privacy/consents` | List consent records |
| POST | `/api/v1/platform/privacy/consents` | Record consent |
| GET | `/api/v1/platform/privacy/dsar` | List data subject access requests |
| POST | `/api/v1/platform/privacy/dsar` | Create DSAR |
| POST | `/api/v1/platform/privacy/dsar/{id}/fulfill` | Fulfill DSAR |
| GET | `/api/v1/platform/privacy/processing-register` | Get processing register |
| POST | `/api/v1/platform/privacy/dpia` | Create DPIA |
| GET | `/api/v1/platform/privacy/dpia` | List DPIAs |
| GET | `/api/v1/platform/dlp/policies` | List DLP policies |
| POST | `/api/v1/platform/dlp/policies` | Create DLP policy |
| GET | `/api/v1/platform/dlp/incidents` | List DLP incidents |
| POST | `/api/v1/platform/dlp/incidents/{id}/resolve` | Resolve DLP incident |
| GET | `/api/v1/platform/dlp/classifications` | List data classifications |
| POST | `/api/v1/platform/idp/extract` | Extract data from document (Intelligent Document Processing) |
| GET | `/api/v1/platform/idp/models` | List IDP extraction models |
| POST | `/api/v1/platform/idp/models` | Train IDP extraction model |
| GET | `/api/v1/platform/idp/jobs/{id}` | Get IDP extraction job status |
| GET | `/api/v1/platform/compliance-hub/dashboard` | Get unified compliance dashboard |
| GET | `/api/v1/platform/compliance-hub/calendar` | Get compliance calendar |
| GET | `/api/v1/platform/compliance-hub/regulatory-changes` | List regulatory changes |
| POST | `/api/v1/platform/compliance-hub/regulatory-changes/{id}/assess` | Assess regulatory change impact |
| GET | `/api/v1/platform/compliance-hub/control-mapping` | Get cross-framework control mapping |
| GET | `/api/v1/platform/compliance-hub/scores` | Get compliance health scores |
| GET | `/api/v1/platform/compliance-hub/intelligence` | Get AI-powered regulatory intelligence |
| POST | `/api/v1/platform/privacy/erasure-requests` | Submit GDPR erasure request |
| GET | `/api/v1/platform/composer/custom-fields` | List custom field definitions |
| POST | `/api/v1/platform/composer/custom-fields` | Create custom field on entity |
| GET | `/api/v1/platform/composer/custom-objects` | List custom objects |
| POST | `/api/v1/platform/composer/custom-objects` | Create custom object |
| GET | `/api/v1/platform/composer/custom-objects/{id}/records` | List custom object records |
| POST | `/api/v1/platform/composer/custom-objects/{id}/records` | Create custom object record |
| POST | `/api/v1/platform/composer/scripts` | Create extension script |
| POST | `/api/v1/platform/composer/scripts/{id}/execute` | Execute extension script |
| GET | `/api/v1/platform/composer/pages` | List composed pages |
| POST | `/api/v1/platform/composer/pages` | Create composed page |
| GET | `/api/v1/platform/grc/certification/campaigns` | List certification campaigns |
| POST | `/api/v1/platform/grc/certification/campaigns` | Create certification campaign |
| GET | `/api/v1/platform/grc/certification/campaigns/{id}/reviews` | List reviews for campaign |
| POST | `/api/v1/platform/grc/certification/reviews/{id}/decide` | Submit certification decision |
| GET | `/api/v1/platform/grc/certification/analytics` | Get certification analytics |
| GET | `/api/v1/platform/grc/role-mining` | Get role mining results |

### Workflow Service
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/workflow/workflows` | List workflow definitions |
| POST | `/api/v1/workflow/workflows` | Create workflow definition |
| GET | `/api/v1/workflow/workflows/{id}` | Get workflow definition |
| PUT | `/api/v1/workflow/workflows/{id}` | Update workflow definition |
| POST | `/api/v1/workflow/workflows/{id}/start` | Start workflow instance |
| GET | `/api/v1/workflow/instances` | List workflow instances |
| GET | `/api/v1/workflow/instances/{id}` | Get instance status and history |
| POST | `/api/v1/workflow/instances/{id}/approve` | Approve current step |
| POST | `/api/v1/workflow/instances/{id}/reject` | Reject current step |
| POST | `/api/v1/workflow/instances/{id}/delegate` | Delegate to another user |
| GET | `/api/v1/workflow/templates` | List workflow templates |
| GET | `/api/v1/workflow/my-tasks` | List pending tasks for current user |

### Report Service
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/report/reports` | List reports |
| POST | `/api/v1/report/reports` | Create report |
| GET | `/api/v1/report/reports/{id}` | Get report result |
| POST | `/api/v1/report/reports/{id}/export` | Export report (CSV, PDF, Excel, Parquet) |
| POST | `/api/v1/report/reports/{id}/schedule` | Schedule report delivery |
| GET | `/api/v1/report/dashboards` | List dashboards |
| GET | `/api/v1/report/dashboards/{id}` | Get dashboard data |
| POST | `/api/v1/report/dashboards` | Create dashboard |
| GET | `/api/v1/report/kpis` | List KPIs |
| POST | `/api/v1/report/kpis` | Define KPI |
| GET | `/api/v1/report/insights` | Get AI-driven insights and anomalies |
| GET | `/api/v1/report/esg/emissions` | Get emissions data (Scope 1, 2, 3) |
| GET | `/api/v1/report/esg/scorecards` | Get ESG scorecards |
| POST | `/api/v1/report/esg/reports` | Generate ESG regulatory report |
| GET | `/api/v1/report/carbon/footprint` | Get carbon footprint data |
| POST | `/api/v1/report/carbon/offsets` | Record carbon offset |
| GET | `/api/v1/report/carbon/targets` | Get science-based target progress |
| GET | `/api/v1/report/ml/models` | List ML models |
| POST | `/api/v1/report/ml/models` | Register ML model |
| POST | `/api/v1/report/ml/models/{id}/deploy` | Deploy ML model |
| POST | `/api/v1/report/ml/models/{id}/predict` | Run ML prediction |
| GET | `/api/v1/report/data-lake/datasets` | List data lake datasets |
| POST | `/api/v1/report/data-lake/query` | Query data lake (schema-on-read) |
| GET | `/api/v1/report/process-mining/processes` | List discovered processes |
| GET | `/api/v1/report/process-mining/processes/{id}/conformance` | Get conformance checking results |
| GET | `/api/v1/report/process-mining/bottlenecks` | Get bottleneck analysis |
| POST | `/api/v1/report/process-mining/simulate` | Run process simulation |
| GET | `/api/v1/report/cpm/strategy-map` | Get strategy map |
| GET | `/api/v1/report/cpm/okrs` | List objectives and key results |
| POST | `/api/v1/report/cpm/okrs` | Create OKR |
| GET | `/api/v1/report/cpm/scorecard` | Get balanced scorecard |
| GET | `/api/v1/report/cpm/initiatives` | List strategic initiatives |
| POST | `/api/v1/report/cpm/initiatives` | Create strategic initiative |
| GET | `/api/v1/report/narrative/packages` | List report packages |
| POST | `/api/v1/report/narrative/packages` | Create report package |
| POST | `/api/v1/report/narrative/commentary` | Add commentary to report section |
| POST | `/api/v1/report/augmented/query` | Natural language query |
| GET | `/api/v1/report/augmented/insights` | Get auto-generated insights |
| POST | `/api/v1/report/augmented/discovery` | Run smart data discovery |
| GET | `/api/v1/report/planning/assumptions` | List planning assumptions |
| POST | `/api/v1/report/planning/assumptions` | Create planning assumption |
| POST | `/api/v1/report/planning/scenarios` | Create connected planning scenario |
| POST | `/api/v1/report/planning/scenarios/{id}/simulate` | Simulate connected planning scenario |
| GET | `/api/v1/report/ml-studio/projects` | List ML studio projects |
| POST | `/api/v1/report/ml-studio/projects` | Create ML studio project |
| GET | `/api/v1/report/ml-studio/projects/{id}/experiments` | List experiments |
| POST | `/api/v1/report/ml-studio/projects/{id}/experiments` | Create experiment |
| POST | `/api/v1/report/ml-studio/experiments/{id}/train` | Start model training |
| GET | `/api/v1/report/ml-studio/experiments/{id}/compare` | Compare experiment results |
| POST | `/api/v1/report/ml-studio/models/{id}/deploy-studio` | Deploy model from studio |
| GET | `/api/v1/report/ml-studio/explainability/{modelId}` | Get model explainability report |

### Integration Service
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/integrations` | List integrations |
| POST | `/api/v1/integrations` | Create integration |
| GET | `/api/v1/integrations/{id}` | Get integration details |
| PUT | `/api/v1/integrations/{id}` | Update integration |
| DELETE | `/api/v1/integrations/{id}` | Disable integration |
| POST | `/api/v1/integrations/{id}/test` | Test integration connection |
| POST | `/api/v1/integrations/{id}/sync` | Trigger manual sync |
| GET | `/api/v1/integrations/{id}/logs` | Get integration sync logs |
| POST | `/api/v1/integrations/import` | Upload file for import |
| GET | `/api/v1/integrations/jobs/{id}` | Get import/export job status |
| GET | `/api/v1/integrations/connectors` | List available connector types |
| POST | `/api/v1/integrations/webhooks` | Register webhook endpoint |
| GET | `/api/v1/integrations/mdm/golden-records` | List golden records |
| GET | `/api/v1/integrations/mdm/golden-records/{entity}/{id}` | Get golden record for entity |
| POST | `/api/v1/integrations/mdm/match` | Run matching on entity records |
| POST | `/api/v1/integrations/mdm/merge` | Merge duplicate records |
| GET | `/api/v1/integrations/mdm/quality` | Get data quality scorecards |
| GET | `/api/v1/integrations/governance/catalog` | Browse data catalog |
| GET | `/api/v1/integrations/governance/lineage/{entity}/{id}` | Get data lineage |
| GET | `/api/v1/integrations/governance/classifications` | List data classifications |
| POST | `/api/v1/integrations/trade-compliance/screen` | Screen entity against denied party lists |
| GET | `/api/v1/integrations/trade-compliance/screening-results` | Get screening results |
| GET | `/api/v1/integrations/trade-compliance/licenses` | List export licenses |
| POST | `/api/v1/integrations/trade-compliance/licenses` | Create export license |
| GET | `/api/v1/integrations/trade-compliance/classifications` | List export control classifications |
| GET | `/api/v1/integrations/api-marketplace/catalog` | Browse API catalog |
| POST | `/api/v1/integrations/api-marketplace/subscriptions` | Subscribe to API |
| GET | `/api/v1/integrations/api-marketplace/usage` | Get API usage analytics |
| GET | `/api/v1/integrations/api-marketplace/consumers` | List API consumers |
| GET | `/api/v1/integrations/event-mesh/topics` | List event topics |
| POST | `/api/v1/integrations/event-mesh/gateway/publish` | Publish event via HTTP gateway |
| POST | `/api/v1/integrations/event-mesh/gateway/subscribe` | Subscribe to events via HTTP gateway |
| GET | `/api/v1/integrations/event-mesh/schema-registry` | List event schemas |
| POST | `/api/v1/integrations/blockchain/anchor` | Anchor provenance record to blockchain |
| GET | `/api/v1/integrations/blockchain/verify/{hash}` | Verify blockchain record |
| POST | `/api/v1/integrations/blockchain/smart-contracts/deploy` | Deploy smart contract |
| GET | `/api/v1/integrations/blockchain/transactions/{id}` | Get blockchain transaction status |
| GET | `/api/v1/integrations/developer-portal/docs` | Get developer portal documentation |
| GET | `/api/v1/integrations/developer-portal/sandbox/status` | Get sandbox environment status |
| POST | `/api/v1/integrations/developer-portal/api-keys` | Provision developer portal API key |

### Admin
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/v1/admin/dlq/{service}/replay` | Replay messages from dead-letter queue for a service |

---

*See [Security](../02-security/overview.md) for authentication and authorization details.*
*See [Events](../04-events/overview.md) for event schemas and async communication patterns.*
*See [Standards](standards.md) for API design standards.*
*See [Error Codes](error-codes.md) for the full error code taxonomy.*
