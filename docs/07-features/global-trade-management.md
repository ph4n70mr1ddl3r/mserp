# Global Trade Management (GTM)

Denied party screening, export license management, customs documentation, duty drawback, preferential trade agreement utilization, harmonized tariff schedule management, and country of origin determination, managed by the Integration Service.

## Overview

Global Trade Management (GTM) ensures regulatory compliance and cost optimization for cross-border trade operations. The module screens all trading parties (customers, suppliers, carriers, consignees) against government-maintained denied party lists and sanctions databases before transactions proceed. Export license management tracks controlled goods classifications against applicable export control regimes (EAR, ITAR, EU Dual-Use) and validates license coverage before shipment release.

Customs documentation automation generates required declarations (commercial invoices, packing lists, certificates of origin, shipper's letter of instruction) with pre-populated data from order, product, and party records. Duty drawback capabilities track imported materials used in exported finished goods for duty recovery. Preferential trade agreement analysis evaluates FTA eligibility (USMCA, EU-Japan EPA, RCEP) to apply reduced tariff rates. Harmonized tariff schedule management maintains country-specific HS code mappings with change tracking. Country of origin determination applies substantial transformation rules, de minimis thresholds, and regional value content calculations.

## Data Flow Diagram

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  Customer/       │────▶│  Denied Party    │────▶│  Compliance Gate    │
│  Supplier Master │     │  Screening       │     │  ┌───────────────┐  │
└──────────────────┘     │  ┌─────────────┐ │     │  │ Pass / Hold / │  │
                         │  │ OFAC, EU,   │ │     │  │ Block         │  │
┌──────────────────┐     │  │ UN, BIS     │ │     │  └───────┬───────┘  │
│  Sales Order     │────▶│  │ Fuzzy Match │ │     └──────────┼──────────┘
│  (International) │     │  └─────────────┘ │                │
└──────────────────┘     └──────────────────┘     ┌──────────▼──────────┐
                                                   │  Export License      │
┌──────────────────┐     ┌──────────────────┐     │  Validation          │
│  Product Master  │────▶│  HTS / Origin    │────▶│  ┌───────────────┐  │
│  + BOM           │     │  Classification  │     │  │ EAR / ITAR    │  │
└──────────────────┘     │  ┌─────────────┐ │     │  │ License Check │  │
                         │  │ HS Code     │ │     │  └───────────────┘  │
┌──────────────────┐     │  │ Lookup      │ │     └──────────┬──────────┘
│  FTA Rules       │────▶│  │ Origin Calc │ │                │
│  (USMCA, EPA)    │     │  └─────────────┘ │     ┌──────────▼──────────┐
└──────────────────┘     └──────────────────┘     │  Customs Filing     │
                                                   │  ┌───────────────┐  │
┌──────────────────┐     ┌──────────────────┐     │  │ AES / ABI /   │  │
│  Import Records  │────▶│  Duty Drawback   │────▶│  │ ATLAS /       │  │
│  (Consumption)   │     │  Engine          │     │  │ Single Window │  │
└──────────────────┘     └──────────────────┘     │  └───────────────┘  │
                                                   └─────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Denied Party Screening** | Integration Service (Rust) | Real-time and batch screening against consolidated sanctions lists (OFAC SDN, EU Consolidated List, UN Security Council, BIS Entity List). Fuzzy matching with configurable threshold for name variations, aliases, and transliterations |
| **Export License Management** | Integration Service (Rust) | License inventory tracking with quantity consumed, remaining balance, expiration dates, and end-use restrictions. Automated hold/release decisions at order and shipment level |
| **Customs Document Generator** | Integration Service (Rust) | Template-driven generation of customs declarations in standard formats (CBP ABI, AESDirect, EU ATLAS, national single-window systems). Supports paper and electronic filing |
| **Duty Drawback Engine** | Integration Service (Rust) | Tracks imported material consumption against exported finished goods. Calculates recoverable duties using substitution drawback and unused merchandise drawback methodologies |
| **FTA Eligibility Analyzer** | Integration Service (Rust) | Evaluates preferential tariff eligibility using rules of origin (tariff shift, regional value content, de minimis). Maintains product-level FTA qualification status with supporting documentation |
| **HTS Code Management** | Integration Service (Rust) | Hierarchical HS code management with country-specific tariff schedules. Historical rate tracking, annual Harmonized System update migration, and classification change audit |
| **Country of Origin Engine** | Integration Service (Rust) | Applies substantial transformation tests, cumulative origin rules, and regional value content calculations. Supports multi-tier bill-of-material origin tracing |
| **Storage** | PostgreSQL + Redis | Trade compliance data, party records, and customs documents in PostgreSQL; screening result cache and HTS lookup index in Redis |

## Business Rules

| ID | Rule | Description |
|----|------|-------------|
| BR-GTM-001 | Mandatory Pre-Shipment Screening | All parties associated with an international shipment (ship-to, sold-to, bill-to, carrier, end-user) must be screened against all active denied party lists before customs filing. A match at any confidence level blocks the shipment |
| BR-GTM-002 | Export License Exhaustion | When an export license's remaining quantity falls below 10% of the original authorized quantity, the system generates a proactive alert to the trade compliance team. At 100% consumption, shipments requiring that license are auto-held |
| BR-GTM-003 | Classification Completeness | No product may be included in an international shipment without a valid HS code classified for the destination country. Unclassified products trigger a hold pending trade compliance review |
| BR-GTM-004 | Dual-Use Determination | Products with ECCN classifications requiring BIS export licenses must undergo end-use and end-user vetting before license application. Red flags (unusual routing, cash payment, unfamiliar consignee) escalate to compliance officer review |
| BR-GTM-005 | FTA Certificate Validity | Preferential tariff rates may only be applied if a valid certificate of origin exists for the product-FTA-destination combination. Certificates must be renewed before expiration with updated supplier declarations |
| BR-GTM-006 | Duty Drawback Time Limit | Drawback claims must be filed within the statutory limitation period (3 years for US, varies by country). The system tracks the filing deadline and generates alerts 90 days before expiration |
| BR-GTM-007 | Country of Origin Aggregation | For products with multi-tier BOMs, origin is determined at the component level and aggregated. If any non-originating component exceeds the de minimis threshold (default 10% of transaction value), the finished good does not qualify for preferential origin |
| BR-GTM-008 | Sanctions List Update Frequency | Denied party screening databases must be refreshed within 24 hours of any government list update. During the refresh window, a conservative matching strategy is applied (lower confidence threshold) |
| BR-GTM-009 | Customs Filing Deadline | Export declarations must be filed at least 24 hours before carrier departure for ocean freight and 2 hours for air freight. Late filings trigger an escalation to the trade compliance manager |
| BR-GTM-010 | Audit Trail Retention | All trade compliance decisions, screening results, license checks, and customs filings must be retained for a minimum of 7 years. Records are immutable after creation — corrections create new records with references to originals |
| BR-GTM-011 | Restricted Destination Holds | Shipments to countries under comprehensive sanctions (embargoed destinations) are automatically held regardless of screening results. Release requires documented approval from the Chief Compliance Officer |
| BR-GTM-012 | Valuation Method Consistency | Customs valuation must consistently apply the same WTO-accepted method (transaction value, identical goods, similar goods, deductive, computed, or fallback) per product across all declarations within a fiscal year |

## Data Model

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `gtm_trade_parties` | `id, tenant_id, party_type, name, aliases, country, tax_id, screening_status, last_screened_at` | Has many `gtm_screening_results` |
| `gtm_export_licenses` | `id, tenant_id, license_number, control_regime, classification, qty_authorized, qty_consumed, valid_from, valid_to, status` | Has many `gtm_license_usage_records` |
| `gtm_customs_declarations` | `id, tenant_id, shipment_id, declaration_type, destination_country, filing_ref, status, filed_at, response` | Belongs to `shipment`; has many `gtm_declaration_line_items` |
| `gtm_tariff_classifications` | `id, tenant_id, product_id, country_code, hs_code, duty_rate, fta_rate, fta_code, valid_from, valid_to` | Belongs to `product` |
| `gtm_trade_agreements` | `id, tenant_id, agreement_code, name, origin_rules, de_minimis_pct, rvc_threshold, valid_from, valid_to` | Has many `gtm_fta_certificates` |
| `gtm_country_of_origin_rules` | `id, tenant_id, product_id, origin_country, determination_method, component_origins, rvc_pct, is_qualified` | Belongs to `product` |

## Events

### Events Produced

| Event Type | Payload | Description |
|------------|---------|-------------|
| `integration.trade.party-screened` | `{screening_id, party_id, party_name, lists_checked, match_status, confidence, screened_at}` | A trade party has been screened against denied party lists |
| `integration.trade.license-validated` | `{license_id, shipment_id, classification, remaining_qty, valid_to, validated_at}` | An export license has been validated for a shipment |
| `integration.trade.customs-filed` | `{declaration_id, shipment_id, destination_country, filing_ref, filing_type, filed_at}` | A customs declaration has been filed with the destination authority |
| `integration.trade.duty-calculated` | `{shipment_id, product_id, hs_code, duty_rate, duty_amount, fta_applied, fta_savings, calculated_at}` | Import duties have been calculated for a product in a shipment |

### Events Consumed

| Event Type | Source | Action |
|------------|--------|--------|
| `commerce.order.submitted` | Commerce Service | Screen order parties, validate export licenses, initiate customs documentation |
| `commerce.customer.created` | Commerce Service | Screen new customer against denied party lists before activation |
| `finance.supplier.created` | Finance Service | Screen new supplier against denied party lists before activation |
| `finance.purchase-order.approved` | Finance Service | Validate import classification, calculate duties, check FTA eligibility |
| `config.changed` | Config Service | Reload sanctions list sources, FTA rules, HTS schedules, and compliance thresholds |

## Integration Points

| Integration | Service | Protocol | Description |
|-------------|---------|----------|-------------|
| Order Management | Commerce Service | Event-driven | Order submission triggers party screening and license validation |
| Customer Master | CRM Service | Internal API | Customer creation triggers denied party screening workflow |
| Supplier Master | Finance Service | Event-driven | Supplier creation triggers denied party screening workflow |
| Customs Authorities | Integration Service | EDI / API | Electronic customs filing via national single-window systems (ABI, ATLAS, etc.) |
| Platform Config | Config Service | Event-driven | Dynamic reload of sanctions lists, HTS schedules, and FTA parameters |

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `integration.gtm.screening.match_threshold` | 85 | Minimum fuzzy match confidence to flag a potential denial list match (0-100) |
| `integration.gtm.license.exhaustion_alert_pct` | 10 | Percentage remaining to trigger license exhaustion alert |
| `integration.gtm.drawback.filing_deadline_days` | 1095 | Statutory deadline for drawback claim filing (3 years) |
| `integration.gtm.origin.de_minimis_pct` | 10 | Maximum non-originating component value for FTA qualification |
| `integration.gtm.audit.retention_years` | 7 | Minimum years to retain trade compliance records |
| `integration.gtm.screening.refresh_interval_hours` | 24 | Maximum hours between sanctions list database refreshes |

## Implementation Notes

- Denied party screening uses a multi-pass fuzzy matching algorithm: exact match → normalized match (diacritics removed, whitespace collapsed) → phonetic match (Soundex/Metaphone) → tokenized Jaro-Winkler similarity. Each pass applies progressively lower confidence thresholds.
- Export license consumption tracking is atomic — the `qty_consumed` field is incremented within the same database transaction as the shipment allocation to prevent oversubscription. Optimistic concurrency via `version` field handles concurrent allocation attempts.
- Customs document generation uses a template engine with country-specific templates stored as versioned JSON. Templates are loaded from the database and rendered with shipment, product, and party data. The output is validated against the target authority's XML schema before filing.
- FTA eligibility analysis traces the full bill of materials for each product, applying tariff shift rules at each component level. Results are cached per product-FTA-destination with invalidation on BOM or component origin changes.
- HTS code management supports the World Customs Organization's 5-year Harmonized System update cycle. Migration tools compare old and new codes, flagging products affected by code changes for re-classification.
- The duty drawback engine maintains a first-in-first-out linkage between imported material lots and exported finished goods. Substitution drawback allows matching imported and exported materials of the same HTS classification regardless of physical lot.

## Cross-References

- [Order Orchestration](order-orchestration.md) — International orders trigger trade compliance checks
- [EDI](edi.md) — EDI 856/DESADV for customs filing data exchange
- [Integration Service](../06-services/integration.md) — Trade compliance and customs operations
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
- [NFR: Performance](../10-planning/nfr.md) — Party screening < 3s, customs filing < 10s
