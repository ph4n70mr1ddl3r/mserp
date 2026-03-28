# Multi-Currency

All financial amounts are stored in a base currency per tenant and converted using exchange rates maintained by the Finance Service.

| Aspect | Implementation |
|--------|---------------|
| Base currency | Configured per tenant (e.g., `USD`, `EUR`) |
| Exchange rates | Daily automated feed from ECB or configurable provider; manual override supported |
| Storage | Amounts stored as `(amount DECIMAL(19,4), currency_code CHAR(3), exchange_rate DECIMAL(19,10))` |
| Triangulation | EUR-based triangulation for non-EUR conversions per ECB convention |
| Realized gains/losses | Automatically calculated on settlement (payment vs. invoice rate difference) |
| Unrealized gains/losses | Calculated during period-end revaluation of open foreign-currency items |
| Corporate rates | Tenant-defined corporate exchange rates overriding market rates for internal transfers |

*See [Internationalization](i18n.md) for locale-aware currency formatting. See [Architecture Overview](../architecture/overview.md) for system context.*
