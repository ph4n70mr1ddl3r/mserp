# Available-to-Promise (ATP) / Capable-to-Promise (CTP)

Real-time availability and promise-date calculation managed by the Commerce Service with integration to Manufacturing Service (production capacity) and Report Service (analytics).

| Aspect | Implementation |
|--------|---------------|
| ATP Check | Real-time on-hand availability across all warehouses, net of open orders, safety stock reservations, and in-transit stock with sub-second response |
| CTP Check | Availability considering production capacity (work center utilization, open work orders) and procurement lead times for make-to-order and configure-to-order items |
| Allocation Rules | Configurable allocation by customer priority tier, sales channel, and geographic region with percentage-based or quantity-based reservation pools |
| Promise Date Calculation | Earliest ship-date calculation factoring inventory ATP, production CTP, inter-warehouse transfer time, and carrier lead times |
| Cross-Region ATP | Aggregated availability across regions and business units with inter-company transfer lead time included in promise date |
| ATP Dashboard | Real-time availability views by product, warehouse, region, and channel with drill-down into allocation buckets and safety stock levels |
| Safety Stock Enforcement | Configurable safety stock rules per product-warehouse combination excluded from ATP; emergency safety stock override with approval |
| Backorder Management | Automatic backorder creation for unfulfilled quantity with estimated fulfillment date from CTP or next planned receipt |
| Supply-Demand Matching | Time-phased supply-demand matching across planning horizons with projected stockout dates and replenishment suggestions |

### Promise Date Calculation Flow

```
Request (Product + Qty + Ship-To)
  → ATP Check (on-hand − allocated − safety stock)
    ├─ Sufficient → Promise Date = pick/pack time + ship time
    └─ Insufficient → CTP Check (production capacity + procurement)
        ├─ Can Produce → Promise Date = production lead time + ship time
        └─ Cannot Produce → Cross-Region ATP (transfer + ship time)
            └─ Final Promise Date (earliest feasible)
```

### Allocation Rule Priority

| Priority | Allocation Bucket | Description |
|----------|-------------------|-------------|
| 1 | Strategic Accounts | Reserved quantity for top-tier customer contracts |
| 2 | Channel Allocation | Percentage split across web, B2B portal, marketplace |
| 3 | Regional Pool | Geographic allocation to prevent cross-region depletion |
| 4 | First-Come-First-Served | Remaining unallocated stock |

### Integration Points

| Integration | Description |
|-------------|-------------|
| Commerce — Inventory | Real-time stock level queries and ATP-triggered inventory reservations |
| Manufacturing — Production Planning | CTP queries against MPS/MRP, work center capacity, and open work order schedules |
| Manufacturing — Procurement | Supplier lead time data for purchased components feeding CTP calculation |
| Report Service | ATP analytics dashboards, stockout trend analysis, and fill-rate KPIs |

*See [Product Configurator](product-configurator.md) for CTP on configured products. See [Commerce Service](../services/commerce.md) for inventory and ATP modules. See [Manufacturing Service](../services/manufacturing.md) for production planning. See [Architecture Overview](../architecture/overview.md) for system context.*
