# B2C Commerce and Storefront

Storefront CMS, product catalog pages, SEO management, shopping cart, checkout flow, and customer engagement features for direct-to-consumer commerce, managed by the Commerce Service.

## Overview

B2C Commerce and Storefront provides the consumer-facing layer for direct-to-consumer sales channels. The module includes a content management system for building and managing storefront pages, SEO metadata management for search engine visibility, a shopping cart and checkout flow with payment integration, a promotions engine for discount campaigns, and customer review functionality. It integrates with PIM for product data, the pricing engine for real-time pricing, inventory for availability display, and payment gateways for transaction processing.

## Data Flow Diagram

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  PIM / Catalog   │────▶│  Storefront CMS  │────▶│  Page Renderer      │
│  (Products)      │     │  ┌─────────────┐ │     │  ┌───────────────┐  │
└──────────────────┘     │  │ Template     │ │     │  │ SEO Meta      │  │
                         │  │ Engine       │ │     │  │ Generation    │  │
┌──────────────────┐     │  └─────────────┘ │     │  └───────────────┘  │
│  Pricing Engine  │────▶│  ┌─────────────┐ │     └──────────┬──────────┘
│  (Prices)        │     │  │ Content     │ │                │
└──────────────────┘     │  │ Blocks      │ │     ┌──────────▼──────────┐
                         │  └─────────────┘ │     │  Shopping Cart       │
┌──────────────────┐     └──────────────────┘     │  ┌───────────────┐  │
│  Search Index    │◀─────────────────────────────│  │ Add/Update    │  │
│  (Platform)      │──────────────────────────────▶│  │ Remove        │  │
└──────────────────┘     ┌──────────────────┐     │  └───────┬───────┘  │
                         │  Promotions      │────▶│          │          │
                         │  Engine          │     │  ┌───────▼───────┐  │
                         └──────────────────┘     │  │ Checkout      │  │
                                                   │  │ Flow          │  │
                         ┌──────────────────┐     │  └───────┬───────┘  │
                         │  Payment         │◀────│          │          │
                         │  Gateway         │     └──────────┼──────────┘
                         └──────────────────┘                │
                                                   ┌─────────▼──────────┐
                                                   │  Customer Reviews   │
                                                   │  + Ratings          │
                                                   └────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Storefront CMS** | Commerce Service (Rust) | Template-based page builder with content blocks, layout management, and responsive rendering |
| **SEO Management** | Commerce Service | Automated metadata generation, URL slug management, sitemap generation, and structured data markup |
| **Shopping Cart** | Commerce Service | Persistent cart with guest and authenticated modes, quantity management, and price recalculation |
| **Checkout Flow** | Commerce Service + Workflow Service | Multi-step checkout with address validation, shipping method selection, payment processing, and order placement |
| **Promotions Engine** | Commerce Service | Rule-based promotion evaluation supporting percentage off, fixed amount, buy-X-get-Y, free shipping, and coupon codes |
| **Customer Reviews** | Commerce Service | Star ratings, text reviews, moderation workflow, and review aggregation for product display |
| **Site Search** | Commerce Service + Platform Service | Faceted product search with autocomplete, spell correction, and relevance tuning |
| **Storage** | PostgreSQL + Redis | Page content and configurations in PostgreSQL; cart sessions and search cache in Redis |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Page Manager | Storefront page CRUD, template rendering, and publishing | Commerce Service |
| SEO Generator | Metadata, sitemap, robots.txt, and structured data management | Commerce Service |
| Cart Manager | Shopping cart lifecycle, pricing updates, and cart persistence | Commerce Service |
| Checkout Processor | Multi-step checkout orchestration and validation | Commerce Service |
| Promotions Engine | Rule evaluation, coupon validation, and discount calculation | Commerce Service |
| Review Manager | Review submission, moderation, and aggregation | Commerce Service |
| Template Renderer | Page template compilation and content block assembly | Commerce Service |
| Search Integrator | Search index management and query proxy | Commerce Service |

## Promotions Rule Types

| Rule Type | Configuration | Example |
|-----------|--------------|---------|
| Percentage Off | `percentage, min_purchase, applicable_categories` | 20% off electronics over $100 |
| Fixed Amount Off | `amount, min_purchase, applicable_products` | $15 off order over $75 |
| Buy X Get Y | `buy_qty, buy_product, get_qty, get_product, discount` | Buy 2 shirts, get 1 at 50% off |
| Free Shipping | `min_purchase, excluded_categories` | Free shipping on orders over $50 |
| Coupon Code | `code, rule_ref, usage_limit, expires_at` | SUMMER25 for 25% off |
| Bundle Discount | `product_ids, bundle_price` | 3-item skincare bundle at $49.99 |
| Tiered Discount | `tiers: [{min, discount}]` | $10 off $50, $25 off $100 |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| PIM | Commerce Service | Internal API | Product data for catalog pages and search |
| Pricing Engine | Commerce Service | Internal API | Real-time price calculation for cart and display |
| Inventory | Commerce Service | Internal API | Stock availability display and reservation |
| Payment Gateways | Integration Service | Connector | Payment processing via Stripe, PayPal, etc. |
| Platform Search | Platform Service | Internal API | Search indexing and query serving |
| Order Orchestration | Commerce Service | Event-driven | Order placement triggers fulfillment |
| Notification Service | Platform Service | Event-driven | Order confirmation, shipping updates |
| Content Management | Platform Service | Internal API | Media asset management for storefront |
| Report Service | Report Service | Internal API | Conversion analytics and funnel reporting |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `commerce.storefront.page-viewed` | `{page_id, page_type, visitor_id, session_id, referrer}` | Page render completes | Report Service, Analytics |
| `commerce.cart.created` | `{cart_id, visitor_id, channel}` | New cart initialized | Report Service, Analytics |
| `commerce.cart.updated` | `{cart_id, item_count, total, coupon_code}` | Cart contents modified | Promotions Engine, Analytics |
| `commerce.checkout.started` | `{cart_id, customer_id, item_count, total}` | Customer enters checkout flow | Report Service, Analytics |
| `commerce.checkout.completed` | `{order_id, cart_id, customer_id, total, payment_method}` | Order placed successfully | Order Orchestration, Notification Service, Report Service |
| `commerce.review.submitted` | `{review_id, product_id, customer_id, rating}` | Customer submits product review | Review Moderation, PIM, Report Service |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `commerce.product.published` | Commerce Service | Update search index and catalog pages |
| `commerce.price.updated` | Commerce Service | Recalculate cart totals and display prices |
| `commerce.inventory.stock-changed` | Commerce Service | Update availability display on product pages |
| `commerce.promotion.activated` | Commerce Service | Apply promotion rules to active carts |
| `commerce.order.placed` | Commerce Service | Clear cart, send confirmation |
| `platform.search.index-updated` | Platform Service | Refresh search results cache |
| `config.changed` | Config Service | Reload storefront template or promotion settings |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `storefront_pages` | `id, tenant_id, slug, title, template_id, page_type, content_blocks, seo_title, seo_description, is_published` | Belongs to `storefront_templates` |
| `storefront_templates` | `id, tenant_id, name, layout, regions, is_active` | Has many `storefront_pages` |
| `seo_metadata` | `id, tenant_id, entity_type, entity_id, meta_title, meta_description, canonical_url, og_tags, structured_data` | Polymorphic: belongs to page, product, or category |
| `promotions` | `id, tenant_id, name, rule_type, conditions, actions, priority, starts_at, ends_at, usage_limit, used_count, is_active` | Evaluated by promotions engine |
| `customer_reviews` | `id, tenant_id, product_id, customer_id, rating, title, content, status, moderated_by, moderated_at` | Belongs to `product`, `customer` |
| `checkout_sessions` | `id, tenant_id, cart_id, customer_id, status, shipping_address, shipping_method, payment_method, totals, created_at` | Belongs to `cart`, `customer` |

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `commerce.storefront.cart.expiry_hours` | 168 | Hours before abandoned cart expires (7 days) |
| `commerce.storefront.cart.guest_to_user_merge` | true | Merge guest cart on login |
| `commerce.storefront.reviews.auto_approve` | false | Auto-publish reviews without moderation |
| `commerce.storefront.reviews.min_length` | 20 | Minimum character count for review text |
| `commerce.storefront.promo.max_per_cart` | 3 | Maximum promotions applicable per cart |
| `commerce.storefront.checkout.session_timeout_minutes` | 30 | Minutes before checkout session expires |

## Cross-References

- [Order Orchestration](order-orchestration.md) — Checkout triggers order fulfillment
- [PIM & Catalog](../06-services/commerce.md) — Product data for storefront display
- [Platform Search](search.md) — Search indexing and query infrastructure
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
- [Commerce Service](../06-services/commerce.md) — Commerce service specification
- [NFR: Performance](../10-planning/nfr.md) — Page render < 200ms, cart update < 100ms
