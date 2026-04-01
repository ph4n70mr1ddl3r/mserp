# Retail Point-of-Sale (POS)

In-store transaction processing with terminal management, payment handling, and offline capability, managed by the Commerce Service.

## Overview

Retail POS provides comprehensive in-store sales operations including terminal (register, cash drawer, receipt printer) management, multi-tender transaction processing (cash, card, mobile, split), real-time inventory synchronization, shift management with cash reconciliation, customer lookup with loyalty points accrual, returns and exchanges, offline transaction queuing, and end-of-day reconciliation with Z-report generation. The system supports high-transaction-volume retail environments with sub-second transaction processing.

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Terminal Management** | Commerce Service (Rust) | Register configuration, cash drawer control, receipt printer integration, barcode scanner support, and peripheral device status monitoring |
| **Transaction Processing** | Commerce Service (Rust) | Multi-tender support: cash, credit/debit card, mobile payments (NFC/QR), gift cards, split tender across payment methods |
| **Inventory Sync** | Commerce Service (Rust) | Real-time stock decrement on transaction completion with configurable batch sync interval for high-volume periods |
| **Shift Management** | Commerce Service (Rust) | Register open/close, cash float assignment, till count, variance tracking, and cash drop/payout recording |
| **Offline Mode** | Commerce Service (Rust) | Local transaction queue with encrypted storage, automatic sync on reconnection, conflict resolution for inventory and pricing |
| **Returns & Exchanges** | Commerce Service (Rust) | Receipt-based and receipt-less returns, cross-register exchanges, refund to original tender, store credit issuance |
| **EOD Reconciliation** | Commerce Service (Rust) | Z-report generation with tender breakdown, cash variance, transaction counts, and automatic GL journal preparation |
| **Storage** | PostgreSQL + Redis + SQLite | Transactions and shifts in PostgreSQL; active session cache in Redis; offline queue in local SQLite |

## Business Rules

| Rule | Description |
|------|-------------|
| **BR-POS-001** | A register must have an active shift opened before any transaction can be processed. Only one shift per register at a time. |
| **BR-POS-002** | All transactions are assigned a unique sequential transaction number per register per shift. Gaps trigger an audit alert. |
| **BR-POS-003** | Offline transactions are queued locally and synced within 5 minutes of reconnection. Offline queue capacity: 500 transactions max. |
| **BR-POS-004** | Returns exceeding the configurable return window (default 30 days) require manager override with reason code. |
| **BR-POS-005** | Cash drawer can only be opened during an active shift. Mid-shift cash drops require reason code and witness. |
| **BR-POS-006** | Split tender transactions must sum to the total amount. Overpayment is not permitted; underpayment blocks completion. |

## Data Model

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `pos_terminals` | `id, tenant_id, store_id, terminal_number, name, status, ip_address, peripheral_config{}, created_at, updated_at, version, is_deleted` | Belongs to store, has many `pos_shifts` |
| `pos_shifts` | `id, tenant_id, terminal_id, cashier_id, opened_at, closed_at, opening_float, closing_count, cash_variance, transaction_count, status, created_at, updated_at, version, is_deleted` | Belongs to terminal, has many `pos_transactions` |
| `pos_transactions` | `id, tenant_id, store_id, terminal_id, shift_id, transaction_number, type, subtotal, tax, total, currency, customer_id, status, synced, created_at, updated_at, version, is_deleted` | Has many `pos_transaction_lines`, `pos_tenders` |
| `pos_transaction_lines` | `id, tenant_id, transaction_id, product_id, quantity, unit_price, discount, line_total, created_at, updated_at, version, is_deleted` | Belongs to transaction, product |
| `pos_tenders` | `id, tenant_id, transaction_id, tender_type, amount, currency, reference_number, card_last4, created_at, updated_at, version, is_deleted` | Belongs to transaction |
| `pos_returns` | `id, tenant_id, original_transaction_id, return_reason_code, total, currency, refund_method, approved_by, created_at, updated_at, version, is_deleted` | References original transaction |

## Events

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `commerce.pos.transaction-completed` | `{transaction_id, store_id, terminal_id, total, currency, tenders[]}` | Transaction finalized | Inventory, Finance, Report |
| `commerce.pos.return-processed` | `{return_id, original_transaction_id, total, currency, reason}` | Return or exchange completed | Inventory, Finance, Report |
| `commerce.pos.shift-opened` | `{shift_id, terminal_id, cashier_id, opening_float}` | Shift started | Dashboard |
| `commerce.pos.shift-closed` | `{shift_id, transaction_count, cash_variance, closing_count}` | Shift ended | Finance, Dashboard |
| `commerce.pos.offline-sync-completed` | `{sync_id, transaction_count, conflicts_resolved}` | Offline queue synchronized | Dashboard, Notification |
| `commerce.pos.z-report-generated` | `{report_id, store_id, shift_ids[], totals{}, tender_breakdown{}}` | End-of-day reconciliation | Finance, Report |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `commerce.inventory.stock-changed` | Commerce Service | Update local inventory cache for ATP display |
| `commerce.price.updated` | Commerce Service | Refresh terminal pricing cache |
| `crm.customer.loyalty-updated` | CRM Service | Update customer loyalty points balance |
| `config.changed` | Config Service | Reload terminal config, return policy, tender options |

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/v1/pos/terminals` | Register a new POS terminal |
| GET | `/api/v1/pos/terminals` | List terminals for a store |
| POST | `/api/v1/pos/shifts/open` | Open a new shift on a terminal |
| POST | `/api/v1/pos/shifts/{id}/close` | Close shift with till count |
| POST | `/api/v1/pos/transactions` | Process a new transaction |
| GET | `/api/v1/pos/transactions/{id}` | Retrieve transaction details |
| POST | `/api/v1/pos/returns` | Process a return or exchange |
| POST | `/api/v1/pos/sync` | Sync offline transaction queue |
| GET | `/api/v1/pos/stores/{id}/z-report` | Generate end-of-day Z-report |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Inventory | Commerce Service | Internal API | Real-time stock decrement and ATP checks |
| Payment Gateways | Integration Service | Connector | Card present, mobile NFC payment processing |
| CRM / Loyalty | CRM Service | Event-driven | Customer lookup and loyalty points accrual |
| Finance (GL) | Finance Service | Event-driven | Transaction journal entries, Z-report posting |
| Notification | Platform Service | Event-driven | Shift alerts, sync failure notifications |
| Report Service | Report Service | Internal API | Sales analytics, store performance, retail KPIs |

## Cross-References

- [Order Orchestration](order-orchestration.md) — POS transactions feed order pipeline
- [CPQ](cpq.md) — Shared pricing engine
- [Commerce Service](../06-services/commerce.md) — Commerce service specification
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
