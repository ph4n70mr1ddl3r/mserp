# Advanced Pricing Engine

Flexible, rule-based pricing engine managed by the Commerce Service with integration to Report Service for optimization models and Manufacturing Service for cost data.

| Aspect | Implementation |
|--------|---------------|
| Price Lists | Multiple price lists per currency, customer group, and date range with priority-based resolution and overlap handling via effective-date precedence |
| Discount Rules | Percentage off, fixed amount off, buy-X-get-Y (free or discounted), volume-based tiered discounts, and combination discount stacking with configurable exclusions |
| Promotions | Time-limited promotions with start/end dates, coupon codes (single-use and multi-use), usage limits per customer and total, and channel-specific targeting |
| Price Formulas | Calculated pricing via cost-plus-margin, competitor-price-based, or custom formula expressions with formula variables sourced from product attributes and cost roll-up |
| Customer-Specific Pricing | Override price lists at the individual customer or customer-group level with optional date effectivity and approval workflows |
| Multi-Currency Pricing | Automatic conversion from base currency using Finance Service exchange rates, or fixed per-currency price overrides per price list |
| Margin Analysis | Real-time margin calculation at line and order level using standard or actual cost, with margin threshold alerts and margin-kill prevention rules |
| Price Waterfall | Full price waterfall visualization from list price through discounts, promotions, rebates, and net price with per-layer margin impact |
| Price Book Versioning | Price list version control with draft/published states, effectivity dates, and audit trail for price change history |
| Cost Integration | Real-time cost data from Manufacturing Service (standard cost roll-up) for margin-aware pricing decisions |
| Configurator Integration | Attribute-based pricing rules triggered by Product Configurator selections with formula-driven option pricing |
| Rounding Rules | Configurable rounding rules per currency and price component (line total, discount amount, tax) with midpoint and banker's rounding support |

### Discount Rule Types

| Rule Type | Description |
|-----------|-------------|
| Percentage | Flat percentage off list price, applicable at line or order level with optional category exclusions |
| Fixed Amount | Fixed currency amount off per unit or per order with currency conversion for cross-border orders |
| Buy X Get Y | Trigger quantity of qualifying items earns discount on target items; supports same-item and cross-item variants |
| Volume / Tiered | Progressive discount tiers based on cumulative quantity with configurable threshold boundaries and reset rules |
| Mix & Match | Discount applied when a combination of items from different categories meets a qualifying condition |
| Multi-Buy | Fixed price for a bundle of items (e.g., 3 for $10) with optional partial-application when threshold not met |

### Price Resolution Order

1. Customer-specific price (highest priority)
2. Customer group price list
3. Promotion / coupon price
4. Default price list for currency and date range
5. Product base price (fallback)

*See [Price Optimization](price-optimization.md) for AI-driven pricing strategies. See [Product Configurator](product-configurator.md) for configuration-based pricing. See [Multi-Currency](multi-currency.md) for exchange rate handling. See [Commerce Service](../services/commerce.md) for pricing module details. See [Architecture Overview](../architecture/overview.md) for system context.*
