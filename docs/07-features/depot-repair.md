# Depot Repair

Service logistics and depot repair management for returned products, managed by the CRM Service. Commerce Service provides return order and warranty data; Manufacturing Service provides quality inspection results; Workflow Service manages repair approval workflows.

> **Ownership Note**: Depot Repair is owned by CRM Service. Commerce Service provides return order and warranty claim data via events (`commerce.order.returned`, `commerce.warranty.*`). Manufacturing Service provides quality inspection results (`manufacturing.quality.inspection-completed`). Workflow Service manages repair estimate approval and RMA approval workflows. CRM owns RMA processing, repair execution, diagnostics, quality testing, loaner management, and all repair events.

## Overview

Depot Repair provides end-to-end management of product service and repair operations at centralized depot facilities. The module covers the full repair lifecycle from RMA (Return Material Authorization) issuance through repair estimation, customer approval, diagnostic workflows, parts requisition, repair execution, quality testing, and return-to-customer logistics. It integrates warranty claim processing to determine coverage and cost allocation, supports loaner and exchange programs to maintain customer satisfaction during repair, and includes repair costing and margin analysis for service profitability. A repair knowledge base captures diagnostic patterns and repair solutions for continuous improvement, while refurbishment programs enable economic recovery of returned products through certified refurbishment processes.

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **RMA Processing** | CRM Service (Rust) | Return Material Authorization creation with eligibility validation, return shipping instructions, tracking integration, and receipt confirmation |
| **Repair Estimation** | CRM Service (Rust) | Generate repair estimates based on failure description, product history, and standard repair costs; customer approval workflow for estimates exceeding threshold |
| **Diagnostic Workflow** | CRM Service (Rust) | Structured diagnostic process with configurable test sequences, fault code catalog, root cause analysis, and findings documentation |
| **Parts Requisition** | CRM Service (Rust) | Parts needed for repair requisitioned from inventory with availability check, reservation, and backorder handling |
| **Repair Execution Tracking** | CRM Service (Rust) | Track repair progress through stages (received → diagnosed → parts ready → in repair → testing → shipped) with time and cost capture |
| **Quality Testing** | CRM Service (Rust) | Post-repair quality verification with configurable test plans, pass/fail criteria, and certification for refurbished units |
| **Return Logistics** | CRM Service + Commerce Service | Shipping label generation, carrier selection, tracking, and delivery confirmation for returning repaired products to customers |
| **Warranty Claim Integration** | CRM Service + Commerce Service | Automatic warranty eligibility check on RMA creation, coverage determination, cost allocation between warranty and customer |
| **Repair Costing & Margin Analysis** | CRM Service (Rust) | Capture labor, parts, and overhead costs per repair; analyze margin by product, repair type, and warranty status |
| **Repair Knowledge Base** | CRM Service (Rust) | Searchable repository of diagnostic patterns, repair procedures, failure codes, and resolution history with ML-driven suggestion |
| **Loaner / Exchange Management** | CRM Service (Rust) | Loaner inventory pool management, assignment to customers during repair, return tracking, and exchange program for unrepairable units |
| **Refurbishment Programs** | CRM Service (Rust) | Certified refurbishment workflows for returned products with grading, repair, testing, and re-certification for resale |
| **Storage** | PostgreSQL | All RMA, repair, diagnostic, quality, loaner, refurbishment, and knowledge base data in CRM database |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| RMA Manager | Create, validate, track, and close return authorizations | CRM Service |
| Estimate Generator | Generate repair estimates from failure data and cost models | CRM Service |
| Diagnostic Engine | Execute diagnostic workflows and capture findings | CRM Service |
| Parts Requisition Engine | Requisition and reserve repair parts from inventory | CRM Service |
| Repair Execution Tracker | Track repair progress, labor, and costs | CRM Service |
| Quality Test Manager | Execute post-repair test plans and capture results | CRM Service |
| Return Logistics Coordinator | Manage return shipping and delivery confirmation | CRM Service |
| Warranty Integration | Determine warranty coverage and cost allocation | CRM Service |
| Costing & Margin Analyzer | Calculate repair costs and analyze margins | CRM Service |
| Knowledge Base Manager | Maintain and search repair knowledge repository | CRM Service |
| Loaner Manager | Manage loaner pool and assignments | CRM Service |
| Refurbishment Manager | Manage refurbishment grading, repair, and certification | CRM Service |

## Repair Order States

| State | Description | Next States |
|-------|-------------|-------------|
| RMA Issued | Return authorization created, awaiting product receipt | Received |
| Received | Product received at depot, awaiting diagnosis | Diagnosing |
| Diagnosing | Diagnostic workflow in progress | Estimated, Beyond Repair |
| Estimated | Repair estimate generated, awaiting customer approval | Approved, Declined, Cancelled |
| Approved | Customer approved estimate, awaiting parts | Parts Ready |
| Parts Ready | All repair parts available, awaiting repair | In Repair |
| In Repair | Repair work in progress | Testing |
| Testing | Post-repair quality testing | Passed, Failed |
| Passed | Quality test passed, awaiting shipment | Shipped |
| Failed | Quality test failed, rework required | In Repair, Beyond Repair |
| Shipped | Repaired product shipped to customer | Completed |
| Completed | Delivery confirmed, repair closed | Terminal |
| Beyond Repair | Product cannot be economically repaired | Refurbishment, Scrapped |
| Declined | Customer declined repair estimate | Returned Unrepaired |
| Returned Unrepaired | Product returned without repair | Terminal |
| Scrapped | Product disposed | Terminal |
| Refurbishment | Product routed to refurbishment program | Refurbished |
| Refurbished | Product refurbished and certified | Terminal |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Return Orders | Commerce Service | Event-driven | Return order creation triggers RMA eligibility check |
| Warranty Management | Commerce Service | Event-driven | Warranty claim submissions for coverage determination |
| Inventory Management | Commerce Service | Event-driven | Parts availability check and reservation for repair |
| Quality Inspection | Manufacturing Service | Event-driven | Quality inspection results for repair validation |
| Approval Workflows | Workflow Service | Event-driven | Repair estimate approval, RMA approval |
| Configuration | Config Service | Event-driven | Repair parameters, test plan templates, cost models |
| Notification Service | Platform Service | Event-driven | RMA status updates, estimate ready, shipment tracking |
| Report Service | Report Service | Internal API | Repair analytics, cost analysis, knowledge base insights |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `crm.repair.rma-created` | `{rma_id, customer_id, product_id, serial_number, failure_description, warranty_status, return_tracking}` | RMA issued for product return | Notification, Warranty, Dashboard |
| `crm.repair.estimate-submitted` | `{estimate_id, rma_id, repair_order_id, labor_cost, parts_cost, total_cost, currency, estimated_days, coverage_pct}` | Repair estimate generated and sent for approval | Workflow, Notification, Customer Portal |
| `crm.repair.diagnosis-completed` | `{diagnostic_id, repair_order_id, fault_codes[], root_cause, findings{}, recommended_actions[], parts_needed[]}` | Diagnostic workflow completed | Knowledge Base, Parts Requisition, Dashboard |
| `crm.repair.completed` | `{repair_order_id, rma_id, total_cost, currency, labor_hours, parts_used[], warranty_covered_pct}` | Repair finished and quality tested | Costing, Warranty, Notification, Dashboard |
| `crm.repair.quality-tested` | `{test_id, repair_order_id, test_plan, results{}, passed, certified, tested_by}` | Post-repair quality test completed | Refurbishment, Notification, Dashboard |
| `crm.repair.returned` | `{repair_order_id, rma_id, carrier, tracking_number, shipped_at}` | Repaired product shipped to customer | Notification, Dashboard |
| `crm.repair.loaner-assigned` | `{assignment_id, rma_id, customer_id, loaner_product_id, loaner_serial, assigned_at, expected_return}` | Loaner unit assigned to customer | Inventory, Notification, Dashboard |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `commerce.order.returned` | Commerce Service | Trigger RMA creation and eligibility validation |
| `commerce.warranty.claim.submitted` | Commerce Service | Determine warranty coverage for repair cost allocation |
| `commerce.stock.updated` | Commerce Service | Check parts availability for repair requisition |
| `manufacturing.quality.inspection-completed` | Manufacturing Service | Incorporate quality inspection data for repair validation |
| `workflow.step.approved` | Workflow Service | Finalize repair estimate approval or RMA approval |
| `config.changed` | Config Service | Update repair parameters, test templates, cost models |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `rma_requests` | `id, tenant_id, rma_number, customer_id, product_id, serial_number, failure_description, failure_codes[], warranty_status, return_tracking, status, received_at, created_at, updated_at, created_by, updated_by, version, is_deleted` | Has one `repair_orders`, optionally has `loaner_assignments` |
| `repair_orders` | `id, tenant_id, rma_id, product_id, serial_number, status, assigned_technician, labor_hours, labor_cost, parts_cost, overhead_cost, total_cost, currency, warranty_covered_pct, started_at, completed_at, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `rma_requests`, has many `diagnostic_reports`, `repair_parts`, `quality_tests` |
| `repair_estimates` | `id, tenant_id, rma_id, repair_order_id, labor_cost, parts_cost[], overhead_cost, total_cost, currency, estimated_days, warranty_coverage_pct, customer_approved, approved_at, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `rma_requests`, `repair_orders` |
| `diagnostic_reports` | `id, tenant_id, repair_order_id, fault_codes[], root_cause, findings{}, test_results{}, recommended_actions[], parts_needed[], diagnosed_by, diagnosed_at, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `repair_orders` |
| `repair_parts` | `id, tenant_id, repair_order_id, part_id, part_number, part_name, quantity_needed, quantity_used, unit_cost, currency, source, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `repair_orders` |
| `quality_tests` | `id, tenant_id, repair_order_id, test_plan, test_results{}, passed, certified, failure_reason, tested_by, tested_at, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `repair_orders` |
| `loaner_assignments` | `id, tenant_id, rma_id, customer_id, loaner_product_id, loaner_serial, assigned_at, expected_return, actual_return, status, created_at, updated_at, created_by, updated_by, version, is_deleted` | Belongs to `rma_requests` |
| `refurbishment_batches` | `id, tenant_id, batch_number, source_rma_ids[], product_id, grade, refurbishment_plan, status, certified, certified_by, created_at, updated_at, created_by, updated_by, version, is_deleted` | References multiple `rma_requests` |
| `repair_knowledge` | `id, tenant_id, product_id, fault_code, symptom, root_cause, resolution, parts_used[], labor_hours, difficulty, success_rate, usage_count, created_at, updated_at, created_by, updated_by, version, is_deleted` | Standalone knowledge entry |

## Configuration

| Setting | Default | Description |
|---------|---------|-------------|
| `crm.repair.auto_approve_estimate_threshold` | 0 | Auto-approve estimates below this currency amount |
| `crm.repair.loaner_pool_enabled` | true | Enable loaner unit assignment during repair |
| `crm.repair.max_diagnostic_days` | 5 | Maximum days allowed for diagnostic phase |
| `crm.repair.max_repair_days` | 15 | Maximum days allowed for complete repair |
| `crm.repair.quality_retest_limit` | 3 | Maximum quality test retest attempts before escalation |
| `crm.repair.knowledge_suggestion_enabled` | true | Enable ML-driven knowledge base suggestions during diagnosis |
| `crm.repair.refurbishment_grading_scale` | [A, B, C] | Refurbishment grade definitions |
| `crm.repair.parts_backorder_max_days` | 10 | Max days to wait for backordered parts before escalation |

## Cross-References

- [Order Orchestration](order-orchestration.md) — Return orders trigger RMA creation
- [MES](mes.md) — Quality inspection integration
- [CRM Service](../06-services/crm.md) — CRM service specification
- [Commerce Service](../06-services/commerce.md) — Return order and warranty data
- [Manufacturing Service](../06-services/manufacturing.md) — Quality inspection results
- [Event Architecture](../04-events/overview.md) — Event naming and inbox bindings
- [Data Architecture](../03-data/overview.md) — Standard columns and RLS policies
- [NFR: Performance](../10-planning/nfr.md) — RMA creation < 2s, diagnostic workflow < 500ms per step
