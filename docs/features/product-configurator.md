# Product Configurator

Constraint-based product configuration managed by the Commerce Service with manufacturing BOM integration.

| Aspect | Implementation |
|--------|---------------|
| Configuration Models | Define configurable product models with features, options, constraints, and default values |
| Constraint Rules | Compatibility and incompatibility rules between options (e.g., engine A requires transmission B) with real-time validation |
| Pricing Integration | Dynamic price calculation based on selected configuration with surcharges, discounts, and margin enforcement |
| BOM Generation | Automatic generation of production BOM from selected configuration with routing and cost roll-up |
| Validation Engine | Real-time completeness and feasibility validation with conflict resolution suggestions |
| Guided Selling | Interactive configuration wizard with recommendations, upsell suggestions, and smart defaults |
| Configuration Templates | Pre-defined configurations for common product variants with one-click selection |
| Multi-Level Configuration | Nested configuration for complex assemblies with sub-assembly configuration propagation |
| Attribute-Based Pricing | Price rules based on configuration attributes (size, material, finish) with formula-based calculation |
| Visual Configuration | 2D/3D visual product preview with configuration-driven rendering for customer-facing sales |
| Quote Integration | One-click conversion from configuration to sales quotation with pricing and lead time |
| Manufacturing Handoff | Configuration-to-order handoff with BOM, routing, and work instructions for Manufacturing Service |
| Version Control | Configuration model versioning with effectivity dates and backward compatibility for existing orders |
| Performance | Sub-second constraint evaluation with cached rule graphs for complex product models |

*See [Available-to-Promise](../services/commerce.md) for availability checks. See [Manufacturing Service](../services/manufacturing.md) for BOM integration. See [Architecture Overview](../architecture/overview.md) for system context.*
