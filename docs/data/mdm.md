# Master Data Management Schema

## 11. Master Data Management Schema

### 11.1 Golden Record Tables

The Integration Service maintains golden record tables for core master data entities:

```sql
CREATE TABLE mdm_golden_records (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    entity_type     VARCHAR(50) NOT NULL,
    source_id       UUID NOT NULL,
    source_service  VARCHAR(50) NOT NULL,
    golden_data     JSONB NOT NULL,
    match_score     DECIMAL(5,4),
    status          VARCHAR(20) NOT NULL DEFAULT 'active',
    tenant_id       UUID NOT NULL,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_mdm_golden_entity ON mdm_golden_records (tenant_id, entity_type);
```

### 11.2 Data Quality Rules

```sql
CREATE TABLE mdm_quality_rules (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    entity_type     VARCHAR(50) NOT NULL,
    rule_name       VARCHAR(100) NOT NULL,
    rule_type       VARCHAR(30) NOT NULL,
    rule_config     JSONB NOT NULL,
    severity        VARCHAR(10) NOT NULL DEFAULT 'warning',
    is_active       BOOLEAN NOT NULL DEFAULT TRUE,
    tenant_id       UUID NOT NULL,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

Rule types: `completeness`, `uniqueness`, `format`, `range`, `referential`, `custom`.

### 11.3 Customer Data Platform (CDP) Schema

```sql
CREATE TABLE cdp_unified_profiles (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID NOT NULL,
    customer_id     UUID,
    email           VARCHAR(255),
    phone           VARCHAR(50),
    full_name       VARCHAR(255),
    segments        JSONB DEFAULT '[]',
    attributes      JSONB DEFAULT '{}',
    identity_graph  JSONB DEFAULT '{}',
    first_seen_at   TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_seen_at    TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    engagement_score DECIMAL(5,2),
    lifetime_value  DECIMAL(19,4),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_cdp_profiles_tenant ON cdp_unified_profiles (tenant_id);
CREATE INDEX idx_cdp_profiles_email ON cdp_unified_profiles (tenant_id, email);
```

### 11.4 IoT Telemetry Schema

```sql
CREATE TABLE iot_device_telemetry (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID NOT NULL,
    device_id       UUID NOT NULL,
    telemetry_type  VARCHAR(50) NOT NULL,
    payload         JSONB NOT NULL,
    ingested_at     TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_by      UUID,
    updated_by      UUID,
    version         INTEGER NOT NULL DEFAULT 1,
    is_deleted      BOOLEAN NOT NULL DEFAULT FALSE
) PARTITION BY RANGE (ingested_at);
```

- Telemetry data is partitioned monthly and archived to the data lake after 90 days.
- IoT device registry is stored in `platform_db`.

### 11.5 Process Mining Schema

```sql
CREATE TABLE process_mining_activity_log (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID NOT NULL,
    process_type    VARCHAR(100) NOT NULL,
    activity_name   VARCHAR(200) NOT NULL,
    resource_id     UUID,
    case_id         UUID NOT NULL,
    started_at      TIMESTAMPTZ NOT NULL,
    completed_at    TIMESTAMPTZ,
    attributes      JSONB DEFAULT '{}',
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_pm_activity_case ON process_mining_activity_log (tenant_id, case_id);
CREATE INDEX idx_pm_activity_process ON process_mining_activity_log (tenant_id, process_type);
```

- Process mining data is derived from existing audit events and domain events.
- Activities are correlated by `case_id` (mapped from `aggregate_id` in the event store).

### 11.6 RPA Execution Schema

```sql
CREATE TABLE rpa_bot_executions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID NOT NULL,
    bot_id          UUID NOT NULL,
    execution_status VARCHAR(20) NOT NULL DEFAULT 'pending',
    trigger_type    VARCHAR(20) NOT NULL,
    started_at      TIMESTAMPTZ,
    completed_at    TIMESTAMPTZ,
    duration_ms     INTEGER,
    error_message   TEXT,
    result_data     JSONB,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_by      UUID,
    updated_by      UUID,
    version         INTEGER NOT NULL DEFAULT 1,
    is_deleted      BOOLEAN NOT NULL DEFAULT FALSE
);

CREATE INDEX idx_rpa_executions_tenant ON rpa_bot_executions (tenant_id);
CREATE INDEX idx_rpa_executions_bot ON rpa_bot_executions (tenant_id, bot_id);
```

### 11.7 Collaboration Schema

```sql
CREATE TABLE collaboration_channels (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID NOT NULL,
    channel_type    VARCHAR(20) NOT NULL DEFAULT 'team',
    name            VARCHAR(200) NOT NULL,
    context_type    VARCHAR(50),
    context_id      UUID,
    members         JSONB DEFAULT '[]',
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_by      UUID,
    updated_by      UUID,
    version         INTEGER NOT NULL DEFAULT 1,
    is_deleted      BOOLEAN NOT NULL DEFAULT FALSE
);

CREATE INDEX idx_collab_channels_tenant ON collaboration_channels (tenant_id);
```

---

*See [Data Architecture Overview](overview.md) for database patterns and schema elements.*
*See [Caching Strategy](caching.md) for cache architecture and policies.*
*See [Data Warehouse](warehouse.md) for analytical schema design.*
*See [Data Lake](data-lake.md) for raw and curated data zones.*
*See [Domain Data Models](domain-models.md) for domain-specific data models.*
