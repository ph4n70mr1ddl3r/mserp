# Master Data Management Schema

## 1. Master Data Management Schema

### 1.1 Golden Record Tables

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

### 1.2 Data Quality Rules

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

### 1.3 Customer Data Platform (CDP) Schema

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

### 1.4 IoT Telemetry Schema

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

### 1.5 Process Mining Schema

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

### 1.6 RPA Execution Schema

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

### 1.7 Collaboration Schema

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

### 1.8 Loyalty Program Schema

```sql
CREATE TABLE loyalty_programs (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID NOT NULL,
    program_name    VARCHAR(200) NOT NULL,
    program_type    VARCHAR(30) NOT NULL DEFAULT 'points',
    tier_structure  JSONB DEFAULT '[]',
    earning_rules   JSONB DEFAULT '{}',
    benefit_catalog JSONB DEFAULT '[]',
    status          VARCHAR(20) NOT NULL DEFAULT 'active',
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_by      UUID,
    updated_by      UUID,
    version         INTEGER NOT NULL DEFAULT 1,
    is_deleted      BOOLEAN NOT NULL DEFAULT FALSE
);

CREATE INDEX idx_loyalty_programs_tenant ON loyalty_programs (tenant_id);
```

### 1.9 Tax Assessment Schema

```sql
CREATE TABLE tax_assessments (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID NOT NULL,
    transaction_id  UUID NOT NULL,
    transaction_type VARCHAR(50) NOT NULL,
    jurisdiction    VARCHAR(100) NOT NULL,
    tax_type        VARCHAR(30) NOT NULL,
    tax_amount      DECIMAL(19,4) NOT NULL,
    tax_rate        DECIMAL(10,6) NOT NULL,
    taxable_amount  DECIMAL(19,4) NOT NULL,
    exemption_uuid  UUID,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_by      UUID,
    updated_by      UUID,
    version         INTEGER NOT NULL DEFAULT 1,
    is_deleted      BOOLEAN NOT NULL DEFAULT FALSE
);

CREATE INDEX idx_tax_assessments_tenant ON tax_assessments (tenant_id);
CREATE INDEX idx_tax_assessments_transaction ON tax_assessments (tenant_id, transaction_id);
```

### 1.10 Compliance Hub Schema

```sql
CREATE TABLE compliance_control_mappings (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID NOT NULL,
    control_id      VARCHAR(100) NOT NULL,
    framework       VARCHAR(30) NOT NULL,
    control_domain  VARCHAR(100) NOT NULL,
    description     TEXT,
    status          VARCHAR(20) NOT NULL DEFAULT 'active',
    evidence_links  JSONB DEFAULT '[]',
    last_assessed   TIMESTAMPTZ,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_by      UUID,
    updated_by      UUID,
    version         INTEGER NOT NULL DEFAULT 1,
    is_deleted      BOOLEAN NOT NULL DEFAULT FALSE
);

CREATE INDEX idx_compliance_controls_tenant ON compliance_control_mappings (tenant_id);
CREATE INDEX idx_compliance_controls_framework ON compliance_control_mappings (tenant_id, framework);
```

### 1.11 Contact Center Schema

```sql
CREATE TABLE cc_interactions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID NOT NULL,
    interaction_type VARCHAR(20) NOT NULL,
    channel         VARCHAR(20) NOT NULL,
    direction       VARCHAR(10) NOT NULL,
    customer_id     UUID,
    agent_id        UUID,
    queue_id        UUID,
    started_at      TIMESTAMPTZ NOT NULL,
    ended_at        TIMESTAMPTZ,
    duration_seconds INTEGER,
    outcome         VARCHAR(30),
    recording_url   VARCHAR(500),
    transcript      TEXT,
    sentiment_score DECIMAL(5,4),
    attributes      JSONB DEFAULT '{}',
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_by      UUID,
    updated_by      UUID,
    version         INTEGER NOT NULL DEFAULT 1,
    is_deleted      BOOLEAN NOT NULL DEFAULT FALSE
);

CREATE INDEX idx_cc_interactions_tenant ON cc_interactions (tenant_id);
CREATE INDEX idx_cc_interactions_customer ON cc_interactions (tenant_id, customer_id);
```

### 1.12 EHS Incident Schema

```sql
CREATE TABLE ehs_incidents (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID NOT NULL,
    incident_type   VARCHAR(50) NOT NULL,
    severity        VARCHAR(10) NOT NULL,
    location        VARCHAR(200),
    description     TEXT NOT NULL,
    reporter_id     UUID NOT NULL,
    investigation_status VARCHAR(20) NOT NULL DEFAULT 'reported',
    root_cause      TEXT,
    capa_id         UUID,
    occurred_at     TIMESTAMPTZ NOT NULL,
    attributes      JSONB DEFAULT '{}',
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_by      UUID,
    updated_by      UUID,
    version         INTEGER NOT NULL DEFAULT 1,
    is_deleted      BOOLEAN NOT NULL DEFAULT FALSE
);

CREATE INDEX idx_ehs_incidents_tenant ON ehs_incidents (tenant_id);
CREATE INDEX idx_ehs_incidents_severity ON ehs_incidents (tenant_id, severity);
```

---

*See [Data Architecture Overview](overview.md) for database patterns and schema elements.*
*See [Caching Strategy](caching.md) for cache architecture and policies.*
*See [Data Warehouse](warehouse.md) for analytical schema design.*
*See [Data Lake](data-lake.md) for raw and curated data zones.*
*See [Domain Data Models](domain-models.md) for domain-specific data models.*
