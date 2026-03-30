# Error Code Taxonomy

## 8. Error Code Taxonomy

All error codes follow the pattern `{DOMAIN}_{CATEGORY}_{SPECIFIC}` and are defined in a centralized registry per service.

### 8.1 Common Error Codes (All Services)

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `AUTH_UNAUTHORIZED` | 401 | Missing or invalid authentication |
| `AUTH_TOKEN_EXPIRED` | 401 | JWT access token has expired |
| `AUTH_FORBIDDEN` | 403 | Insufficient permissions for this operation |
| `AUTH_STEP_UP_REQUIRED` | 403 | Step-up authentication required for this operation |
| `VALIDATION_INVALID_FORMAT` | 400 | Field format validation failed |
| `VALIDATION_REQUIRED_FIELD` | 400 | Required field is missing |
| `VALIDATION_INVALID_VALUE` | 400 | Field value is out of allowed range |
| `RESOURCE_NOT_FOUND` | 404 | Requested resource does not exist |
| `RESOURCE_CONFLICT` | 409 | Resource already exists (unique constraint) |
| `RATE_LIMITED` | 429 | Too many requests |
| `IDEMPOTENCY_CONFLICT` | 422 | Idempotency key already used with different payload |
| `INTERNAL_ERROR` | 500 | Unexpected server error |
| `SERVICE_UNAVAILABLE` | 503 | Dependency temporarily unavailable |
| `QUOTA_EXCEEDED` | 429 | Tenant quota exceeded (users, storage, API calls) |

### 8.2 Auth Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `AUTH_MFA_ALREADY_ENABLED` | 409 | MFA is already enabled for this user |
| `AUTH_MFA_INVALID_CODE` | 400 | MFA verification code is invalid or expired |
| `AUTH_SESSION_NOT_FOUND` | 404 | Session does not exist or has expired |
| `AUTH_PASSWORD_TOO_WEAK` | 400 | Password does not meet complexity requirements |
| `AUTH_LOGIN_LOCKED_OUT` | 423 | Account is temporarily locked due to too many failed login attempts |
| `AUTH_TOKEN_EXPIRED` | 401 | JWT access token has expired |
| `AUTH_REFRESH_TOKEN_INVALID` | 401 | Refresh token is invalid or has been revoked |
| `AUTH_STEP_UP_REQUIRED` | 403 | Step-up authentication required for this operation |

### 8.3 Identity Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `IDENTITY_ROLE_EXISTS` | 409 | Role with this name already exists |
| `IDENTITY_USER_EXISTS` | 409 | User with this email already exists |
| `IDENTITY_GROUP_NOT_FOUND` | 404 | User group not found |
| `IDENTITY_API_KEY_EXPIRED` | 401 | API key has expired |
| `IDENTITY_CANNOT_DELETE_LAST_ADMIN` | 409 | Cannot remove the last tenant admin |
| `IDENTITY_PERMISSION_DENIED` | 403 | User lacks the required permission |
| `IDENTITY_ORG_UNIT_NOT_FOUND` | 404 | Organizational unit not found |
| `IDENTITY_ABAC_POLICY_INVALID` | 400 | ABAC policy expression is invalid |

### 8.4 Tenant Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `TENANT_NOT_FOUND` | 404 | Tenant not found |
| `TENANT_SUSPENDED` | 403 | Tenant account is suspended |
| `TENANT_QUOTA_EXCEEDED` | 429 | Tenant quota exceeded |
| `TENANT_FEATURE_NOT_AVAILABLE` | 403 | Feature not available on current plan |
| `TENANT_DATA_RESIDENCY_VIOLATION` | 409 | Operation violates data residency constraints |
| `TENANT_BRANDING_INVALID` | 400 | Tenant branding configuration is invalid |

### 8.5 Config Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `CONFIG_KEY_NOT_FOUND` | 404 | Configuration key not found |
| `CONFIG_VALIDATION_FAILED` | 400 | Configuration value failed validation |
| `CONFIG_CORS_ORIGIN_INVALID` | 400 | CORS origin URL is invalid |
| `CONFIG_SCHEMA_VIOLATION` | 400 | Configuration value violates schema constraints |
| `CONFIG_LOCALIZATION_KEY_MISSING` | 400 | Required localization key is missing |
| `CONFIG_MAINTENANCE_WINDOW_CONFLICT` | 409 | Maintenance window overlaps with existing window |

### 8.6 Commerce Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `COMMERCE_DUPLICATE_SKU` | 409 | Product SKU already exists |
| `COMMERCE_INSUFFICIENT_STOCK` | 409 | Not enough stock to fulfill |
| `COMMERCE_ORDER_ALREADY_SUBMITTED` | 409 | Order cannot be modified after submission |
| `COMMERCE_QUOTATION_EXPIRED` | 410 | Quotation has expired |
| `COMMERCE_CUSTOMER_INACTIVE` | 409 | Customer account is inactive |
| `COMMERCE_WAREHOUSE_NOT_FOUND` | 404 | Warehouse does not exist |
| `COMMERCE_TRANSFER_INVALID` | 400 | Stock transfer parameters invalid |
| `COMMERCE_RETURN_NOT_ALLOWED` | 400 | Return not allowed for this order/line |
| `COMMERCE_PRICE_INVALID` | 400 | Pricing rule produced invalid price |
| `COMMERCE_CARRIER_NOT_AVAILABLE` | 409 | No carrier available for requested route |
| `COMMERCE_SHIPMENT_INVALID` | 400 | Shipment parameters invalid |
| `COMMERCE_ATP_UNAVAILABLE` | 409 | No available-to-promise quantity for requested date |
| `COMMERCE_CREDIT_HOLD` | 409 | Order held due to credit limit exceeded |
| `COMMERCE_CONFIG_INVALID` | 400 | Product configuration violates constraint rules |
| `COMMERCE_CONFIG_INCOMPLETE` | 400 | Product configuration is incomplete |
| `COMMERCE_SUBSCRIPTION_INACTIVE` | 409 | Subscription is not in an active state |
| `COMMERCE_SUBSCRIPTION_AMENDMENT_INVALID` | 400 | Subscription amendment parameters invalid |
| `COMMERCE_DROPSHIP_UNAVAILABLE` | 409 | Drop ship not available for this product/supplier combination |
| `COMMERCE_WARRANTY_EXPIRED` | 410 | Warranty has expired |
| `COMMERCE_WARRANTY_CLAIM_DUPLICATE` | 409 | Duplicate warranty claim submitted |
| `COMMERCE_LOYALTY_INSUFFICIENT_POINTS` | 400 | Customer does not have enough loyalty points for redemption |
| `COMMERCE_LOYALTY_TIER_INELIGIBLE` | 403 | Customer is not eligible for the requested tier benefit |
| `COMMERCE_LOYALTY_FRAUD_DETECTED` | 409 | Potential loyalty fraud detected (duplicate account, unusual pattern) |
| `COMMERCE_OMNICHANNEL_CHANNEL_UNAVAILABLE` | 409 | Requested sales channel is not available |
| `COMMERCE_OMNICHANNEL_FULFILLMENT_NO_LOCATION` | 409 | No fulfillment location available for order routing |
| `COMMERCE_OMNICHANNEL_BOPIS_UNAVAILABLE` | 409 | Buy online pick up in store not available for this item/location |
| `COMMERCE_PRICE_OPTIMIZATION_MODEL_STALE` | 409 | Price optimization model needs retraining |
| `COMMERCE_PRICE_OPTIMIZATION_SIMULATION_FAILED` | 500 | Price optimization simulation failed to converge |
| `COMMERCE_COLLABORATION_PARTNER_UNREACHABLE` | 502 | Supply chain collaboration partner endpoint unreachable |
| `COMMERCE_COLLABORATION_DEMAND_SIGNAL_INVALID` | 400 | Demand signal data is invalid or incomplete |
| `COMMERCE_COLLABORATION_CAPACITY_CONFLICT` | 409 | Capacity commitment conflicts with existing commitments |
| `COMMERCE_ORCHESTRATION_RULE_INVALID` | 400 | Orchestration rule configuration invalid |
| `COMMERCE_ORCHESTRATION_STEP_FAILED` | 500 | Orchestration step execution failed |
| `COMMERCE_BACKORDER_ALREADY_FULFILLED` | 409 | Backorder already fulfilled |
| `COMMERCE_GOP_NO_SOURCE_AVAILABLE` | 409 | No fulfillment source available for promised date |
| `COMMERCE_STOREFRONT_PAGE_DUPLICATE` | 409 | Storefront page with this URL already exists |
| `COMMERCE_PROMOTION_OVERLAP` | 409 | Promotion dates overlap with existing promotion |
| `COMMERCE_REVIEW_ALREADY_SUBMITTED` | 409 | Customer already reviewed this product |

### 8.7 Finance Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `FINANCE_ACCOUNT_CODE_EXISTS` | 409 | Account code already in use |
| `FINANCE_JOURNAL_UNBALANCED` | 400 | Journal entry must be balanced (debits = credits) |
| `FINANCE_JOURNAL_ALREADY_POSTED` | 409 | Journal entry cannot be modified after posting |
| `FINANCE_PERIOD_CLOSED` | 409 | Accounting period is closed |
| `FINANCE_DUPLICATE_INVOICE` | 409 | Invoice number already exists |
| `FINANCE_SUPPLIER_INACTIVE` | 409 | Supplier is inactive |
| `FINANCE_CURRENCY_UNSUPPORTED` | 400 | Currency code not supported |
| `FINANCE_EXCHANGE_RATE_MISSING` | 400 | No exchange rate for currency pair and date |
| `FINANCE_BUDGET_EXCEEDED` | 409 | Budget limit exceeded for this account/period |
| `FINANCE_INTERCOMPANY_MISMATCH` | 400 | Intercompany transaction tenant mismatch |
| `FINANCE_EXPENSE_POLICY_VIOLATION` | 400 | Expense violates corporate policy |
| `FINANCE_CONTRACT_EXPIRED` | 409 | Contract has expired |
| `FINANCE_CONTRACT_NOT_APPROVED` | 409 | Contract requires approval before execution |
| `FINANCE_PLAN_LOCKED` | 409 | Financial plan is locked for editing |
| `FINANCE_REVENUE_ALREADY_RECOGNIZED` | 409 | Revenue already recognized for this obligation |
| `FINANCE_REVENUE_ALLOCATION_INVALID` | 400 | Revenue allocation amounts do not sum to transaction price |
| `FINANCE_SSP_MISSING` | 400 | Standalone selling price not available for allocation |
| `FINANCE_LEASE_NOT_FOUND` | 404 | Lease contract not found |
| `FINANCE_LEASE_AMENDMENT_INVALID` | 400 | Lease amendment parameters invalid |
| `FINANCE_GRANT_NOT_FOUND` | 404 | Grant not found |
| `FINANCE_GRANT_COMPLIANCE_VIOLATION` | 409 | Grant compliance violation detected |
| `FINANCE_JOINT_VENTURE_ALLOCATION_INVALID` | 400 | Joint venture cost allocation invalid |
| `FINANCE_CLOSE_ANOMALY_DETECTED` | 409 | Financial close anomaly detected |
| `FINANCE_COLLECTION_STRATEGY_INVALID` | 400 | Collection strategy configuration invalid |
| `FINANCE_TAX_JURISDICTION_UNSUPPORTED` | 400 | Tax jurisdiction not supported |
| `FINANCE_TAX_EXEMPTION_INVALID` | 400 | Tax exemption certificate is invalid or expired |
| `FINANCE_TAX_NEXUS_NOT_ESTABLISHED` | 400 | No tax nexus established for the jurisdiction |
| `FINANCE_COMMODITY_PRICE_UNAVAILABLE` | 400 | Commodity market price unavailable for the requested commodity/date |
| `FINANCE_COMMODITY_HEDGE_EXPIRED` | 409 | Commodity hedging position has expired |
| `FINANCE_SPEND_CLASSIFICATION_LOW_CONFIDENCE` | 400 | ML spend classification confidence below threshold |
| `FINANCE_DIVERSITY_CLASSIFICATION_PENDING` | 409 | Supplier diversity classification pending verification |
| `FINANCE_DIVERSITY_GOAL_ALREADY_SET` | 409 | Diversity spend goal already exists for this period |
| `FINANCE_DISCOUNT_CALCULATION_ERROR` | 500 | Dynamic discount calculation failed |
| `FINANCE_DISCOUNT_PROGRAM_INACTIVE` | 409 | Discount program is not active |
| `FINANCE_WORKING_CAPITAL_SIMULATION_FAILED` | 500 | Working capital simulation failed to converge |

### 8.8 HR Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `HR_EMPLOYEE_EXISTS` | 409 | Employee with this ID already exists |
| `HR_LEAVE_BALANCE_EXCEEDED` | 409 | Not enough leave balance |
| `HR_ATTENDANCE_CONFLICT` | 409 | Overlapping attendance record |
| `HR_PAYROLL_ALREADY_PROCESSED` | 409 | Payroll for this period already processed |
| `HR_REQUISITION_CLOSED` | 409 | Job requisition is closed |
| `HR_SUCCESSION_PLAN_EXISTS` | 409 | Succession plan already exists for this position |
| `HR_GOAL_ALIGNMENT_CYCLE` | 400 | Goal alignment creates circular reference |
| `HR_CAREER_PATH_NOT_FOUND` | 404 | Career path not found |
| `HR_COMPETENCY_LEVEL_INVALID` | 400 | Competency assessment level out of range |
| `HR_ONBOARDING_JOURNEY_ALREADY_ACTIVE` | 409 | Employee already has active onboarding journey |
| `HR_ONBOARDING_TEMPLATE_NOT_FOUND` | 404 | Onboarding journey template not found |

### 8.9 Manufacturing Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `MFG_BOM_CIRCULAR_REF` | 400 | BOM contains circular reference |
| `MFG_WORK_ORDER_IN_PROGRESS` | 409 | Work order cannot be modified while in progress |
| `MFG_INSUFFICIENT_MATERIALS` | 409 | Not enough materials for production |
| `MFG_CAPACITY_EXCEEDED` | 409 | Work center capacity exceeded for requested time |
| `MFG_QUALITY_CHECK_FAILED` | 409 | Quality check did not pass threshold |
| `MFG_ECO_PENDING_APPROVAL` | 409 | Engineering Change Order pending approval |
| `MFG_ASSET_NOT_AVAILABLE` | 409 | Asset not available for scheduled time |
| `MFG_PLAN_CONSTRAINT_VIOLATION` | 400 | Plan violates material or capacity constraints |
| `MFG_EHS_INCIDENT_DUPLICATE` | 409 | Duplicate safety incident report detected |
| `MFG_EHS_PERMIT_EXPIRED` | 409 | Work permit has expired |
| `MFG_EHS_CHEMICAL_NOT_REGISTERED` | 404 | Chemical not found in SDS registry |
| `MFG_MRO_SPARE_PART_UNAVAILABLE` | 409 | Spare part not available in inventory |
| `MFG_MRO_ROTABLE_EXCHANGE_POOL_EMPTY` | 409 | No rotable components available in exchange pool |
| `MFG_COMPLIANCE_CERTIFICATION_EXPIRED` | 409 | Product certification has expired |
| `MFG_COMPLIANCE_MATERIAL_VIOLATION` | 409 | Material declaration contains restricted substance |
| `MFG_COMPLIANCE_TEST_IN_PROGRESS` | 409 | Compliance test already in progress for this product |
| `MFG_MES_STATION_UNAVAILABLE` | 409 | Production station unavailable |
| `MFG_MES_WORK_ORDER_NOT_STARTED` | 409 | Cannot complete production - work order not started at station |
| `MFG_SCHEDULE_CONFLICT` | 409 | Schedule conflicts with existing production |
| `MFG_SCHEDULE_RESOURCE_UNAVAILABLE` | 409 | Required resource unavailable in time slot |
| `MFG_SCHEDULE_CONSTRAINT_VIOLATION` | 400 | Schedule violates material or capacity constraints |

### 8.10 CRM Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `CRM_DUPLICATE_CONTACT` | 409 | Contact already exists with this email |
| `CRM_LEAD_CONVERTED` | 409 | Lead already converted to opportunity/customer |
| `CRM_CAMPAIGN_ACTIVE` | 409 | Campaign is active and cannot be modified |
| `CRM_SURVEY_CLOSED` | 409 | Survey is no longer accepting responses |
| `CRM_SERVICE_ORDER_IN_PROGRESS` | 409 | Service order cannot be modified while in progress |
| `CRM_TERRITORY_CONFLICT` | 409 | Territory assignment conflicts with existing assignment |
| `CRM_CONTACT_CENTER_QUEUE_FULL` | 409 | Contact center queue at maximum capacity |
| `CRM_CONTACT_CENTER_AGENT_UNAVAILABLE` | 409 | No agent available for the requested channel |
| `CRM_CONTACT_CENTER_SCRIPT_INVALID` | 400 | Agent script contains invalid branching logic |
| `CRM_SOCIAL_PROFILE_NOT_FOUND` | 404 | Social profile not found for contact |
| `CRM_SOCIAL_ENGAGEMENT_RATE_LIMITED` | 429 | Social platform engagement rate limit reached |
| `CRM_AB_TEST_INSUFFICIENT_SAMPLE` | 400 | Insufficient sample size for statistical significance |
| `CRM_AB_TEST_ALREADY_RUNNING` | 409 | A/B test is already running for this target |
| `CRM_AB_TEST_VARIANT_INVALID` | 400 | A/B test variant configuration invalid |

### 8.11 Project Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `PROJECT_NOT_FOUND` | 404 | Project does not exist |
| `PROJECT_BUDGET_EXCEEDED` | 409 | Project budget exceeded |
| `PROJECT_TASK_DEPENDENCY` | 409 | Circular task dependency detected |

### 8.12 Report Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `REPORT_GENERATION_FAILED` | 500 | Report generation encountered an error |
| `REPORT_NOT_FOUND` | 404 | Report definition not found |
| `REPORT_SCHEDULE_CONFLICT` | 409 | Report schedule conflicts with existing schedule |
| `DASHBOARD_LAYOUT_INVALID` | 400 | Dashboard layout configuration invalid |
| `ML_MODEL_NOT_TRAINED` | 409 | ML model has not been trained yet |
| `ML_MODEL_DEPLOYMENT_FAILED` | 500 | ML model deployment failed |
| `PROCESS_MINING_INSUFFICIENT_DATA` | 400 | Insufficient event data for process mining analysis |
| `REPORT_PLANNING_ASSUMPTION_CONFLICT` | 409 | Planning assumption conflicts with existing assumptions |
| `REPORT_PLANNING_SIMULATION_FAILED` | 500 | Connected planning simulation failed |
| `REPORT_PLANNING_SCENARIO_LIMIT_EXCEEDED` | 429 | Planning scenario limit exceeded |
| `REPORT_PLANNING_SCENARIO_ERROR` | 400 | Planning scenario configuration is invalid |
| `REPORT_PLANNING_DRIVER_CONFLICT` | 409 | Planning driver conflicts with existing drivers in the scenario |
| `REPORT_PLANNING_MODEL_INVALID` | 400 | Connected planning model definition is invalid |
| `REPORT_MLSTUDIO_DATASET_INVALID` | 400 | ML studio dataset format invalid |
| `REPORT_MLSTUDIO_EXPERIMENT_LIMIT` | 429 | ML studio experiment limit exceeded |
| `REPORT_MLSTUDIO_TRAINING_FAILED` | 500 | ML studio model training failed |
| `REPORT_MLSTUDIO_DEPLOYMENT_CONFLICT` | 409 | ML studio model deployment conflicts with active deployment |

### 8.13 Workflow Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `WORKFLOW_DEFINITION_INVALID` | 400 | Workflow definition contains invalid BPMN |
| `WORKFLOW_INSTANCE_STALE` | 409 | Workflow instance state has changed |
| `WORKFLOW_STEP_NOT_ACTIONABLE` | 409 | Workflow step cannot be acted upon in current state |
| `WORKFLOW_TEMPLATE_NOT_FOUND` | 404 | Workflow template not found |
| `WORKFLOW_ESCALATION_FAILED` | 500 | Workflow escalation action failed |

### 8.14 Platform / GRC Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `PLATFORM_FILE_TOO_LARGE` | 413 | File exceeds maximum upload size |
| `PLATFORM_FILE_VIRUS_DETECTED` | 400 | Uploaded file contains malware |
| `PLATFORM_SOD_CONFLICT` | 409 | Segregation of Duties conflict detected |
| `PLATFORM_COMPLIANCE_VIOLATION` | 403 | Operation violates compliance policy |
| `PLATFORM_MASKING_RULE_INVALID` | 400 | Data masking rule configuration invalid |
| `PLATFORM_JOB_ALREADY_RUNNING` | 409 | Scheduled job is already running |
| `PLATFORM_JOB_NOT_FOUND` | 404 | Scheduled job definition not found |
| `PLATFORM_SCREENING_MATCH_FOUND` | 409 | **Deprecated** — mapped to `INTEGRATION_SCREENING_MATCH_FOUND`. Trade compliance is owned by Integration Service |
| `PLATFORM_CONTENT_REPOSITORY_FULL` | 413 | Content repository storage limit exceeded |
| `PLATFORM_CONTENT_INVALID_LIFECYCLE` | 400 | Invalid document lifecycle transition |
| `PLATFORM_PRIVACY_CONSENT_REQUIRED` | 403 | Consent required for this data processing operation |
| `PLATFORM_PRIVACY_DSAR_IN_PROGRESS` | 409 | Data subject access request already in progress |
| `PLATFORM_DLP_VIOLATION` | 403 | Data loss prevention policy violation |
| `PLATFORM_DLP_BLOCKED` | 403 | Operation blocked by DLP policy |
| `PLATFORM_IDP_EXTRACTION_FAILED` | 500 | Intelligent document processing extraction failed |
| `PLATFORM_IDP_MODEL_NOT_TRAINED` | 409 | IDP extraction model has not been trained |
| `PLATFORM_ASSISTANT_INTENT_UNRECOGNIZED` | 400 | Digital assistant could not recognize the intent |
| `PLATFORM_ASSISTANT_ACTION_FAILED` | 500 | Digital assistant failed to execute the requested action |
| `PLATFORM_ASSISTANT_CONTEXT_EXPIRED` | 410 | Digital assistant conversation context has expired |
| `PLATFORM_EMAIL_DELIVERY_FAILED` | 502 | Email delivery to recipient failed |
| `PLATFORM_EMAIL_TEMPLATE_NOT_FOUND` | 404 | Email template not found |
| `PLATFORM_EMAIL_SUPPRESSED_RECIPIENT` | 403 | Recipient is on the email suppression list |
| `PLATFORM_SEARCH_INDEX_UNAVAILABLE` | 503 | Full-text search index is temporarily unavailable |
| `PLATFORM_SEARCH_QUERY_TIMEOUT` | 504 | Full-text search query timed out |
| `PLATFORM_COMPLIANCE_ALERT_TRIGGERED` | 409 | Compliance alert triggered by monitored activity |
| `PLATFORM_COMPLIANCE_CONTROL_INVALID` | 400 | Compliance control configuration is invalid |
| `PLATFORM_COMPLIANCE_REGULATION_NOT_FOUND` | 404 | Referenced regulation not found in compliance registry |
| `PLATFORM_EVENTMESH_ROUTING_ERROR` | 500 | Event mesh failed to route message to destination |
| `PLATFORM_EVENTMESH_SCHEMA_INVALID` | 400 | Event mesh message schema validation failed |
| `PLATFORM_EVENTMESH_GATEWAY_UNAVAILABLE` | 503 | Event mesh gateway is temporarily unavailable |
| `PLATFORM_COMPOSER_FIELD_LIMIT_EXCEEDED` | 409 | Custom field limit exceeded for entity |
| `PLATFORM_COMPOSER_SCRIPT_TIMEOUT` | 504 | Extension script execution timed out |
| `PLATFORM_COMPOSER_SCRIPT_ERROR` | 500 | Extension script execution error |
| `PLATFORM_CERTIFICATION_CAMPAIGN_ACTIVE` | 409 | Certification campaign is active and cannot be modified |
| `PLATFORM_CERTIFICATION_REVIEW_OVERDUE` | 409 | Certification review is past deadline |
| `PLATFORM_ROLE_MINING_INSUFFICIENT_DATA` | 400 | Insufficient access data for role mining |

### 8.15 Integration Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `INTEGRATION_SYNC_IN_PROGRESS` | 409 | Sync already in progress for this integration |
| `INTEGRATION_CONNECTION_FAILED` | 502 | Failed to connect to external system |
| `INTEGRATION_AUTH_EXPIRED` | 401 | OAuth token or credential expired for external system |
| `INTEGRATION_MAPPING_INVALID` | 400 | Field mapping configuration invalid |
| `INTEGRATION_SCREENING_MATCH_FOUND` | 409 | Trade compliance screening flagged a restricted party match |
| `INTEGRATION_LICENSE_EXPIRED` | 409 | Export license has expired |
| `INTEGRATION_CLASSIFICATION_REQUIRED` | 400 | Export control classification required |
| `INTEGRATION_BLOCKCHAIN_ANCHOR_FAILED` | 500 | Failed to anchor record to blockchain |
| `INTEGRATION_BLOCKCHAIN_VERIFICATION_FAILED` | 500 | Blockchain record verification failed |
| `INTEGRATION_BLOCKCHAIN_SMART_CONTRACT_ERROR` | 500 | Smart contract execution error |
| `INTEGRATION_DEVELOPER_PORTAL_SANDBOX_UNAVAILABLE` | 503 | Developer portal sandbox environment unavailable |
| `INTEGRATION_EDI_VALIDATION_FAILED` | 400 | EDI document failed validation |
| `INTEGRATION_EDI_PARTNER_UNREACHABLE` | 502 | EDI trading partner endpoint unreachable |
| `INTEGRATION_EDI_TRANSACTION_DUPLICATE` | 409 | Duplicate EDI transaction detected |
| `INTEGRATION_DQ_PROFILE_FAILED` | 500 | Data quality profile execution failed |
| `INTEGRATION_DQ_RULE_VIOLATION` | 409 | Data quality rule violation detected |
| `INTEGRATION_DQ_MATCH_ERROR` | 500 | Data quality matching engine encountered an error |
| `INTEGRATION_DQ_CLEANSING_FAILED` | 500 | Data quality cleansing transformation failed |
| `INTEGRATION_MARKETPLACE_KEY_INVALID` | 401 | API marketplace subscription key is invalid or revoked |
| `INTEGRATION_MARKETPLACE_QUOTA_EXCEEDED` | 429 | API marketplace subscription quota exceeded |
| `INTEGRATION_MARKETPLACE_API_DEPRECATED` | 410 | API marketplace endpoint has been deprecated and is no longer available |

### 8.16 Error Code Rules

- Each service MUST register its error codes in its OpenAPI spec via the `errors` extension.
- New error codes MUST be additive only (never remove or rename).
- Error codes are case-sensitive and use `UPPER_SNAKE_CASE`.
- Consumer services MUST handle unknown error codes gracefully (treat as generic error).
- Error responses MUST include a `type` URI linking to human-readable documentation for the error code.

---

*See [Standards](standards.md) for API design standards.*
*See [Endpoints](endpoints.md) for the API endpoints overview.*
