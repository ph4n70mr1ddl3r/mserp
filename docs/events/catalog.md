# Event Catalog

## 2. Domain Events

### Commerce Events (Sales + Inventory + PIM + Transportation)
| Event | Description |
|-------|-------------|
| `commerce.customer.created` | Customer created |
| `commerce.customer.updated` | Customer updated |
| `commerce.customer.deleted` | Customer soft-deleted |
| `commerce.quotation.created` | Quotation created |
| `commerce.quotation.accepted` | Quotation accepted |
| `commerce.quotation.rejected` | Quotation rejected |
| `commerce.order.created` | Sales order created |
| `commerce.order.submitted` | Sales order submitted for fulfillment |
| `commerce.order.fulfilled` | Order fulfilled |
| `commerce.order.cancelled` | Order cancelled |
| `commerce.order.returned` | Return/RMA created for order |
| `commerce.product.created` | Product created |
| `commerce.product.updated` | Product updated |
| `commerce.product.approved` | Product approved and published (PIM workflow) |
| `commerce.product.published` | Product published to channels |
| `commerce.stock.updated` | Stock level changed |
| `commerce.stock.reserved` | Stock reserved for an order (saga step) |
| `commerce.stock.reservation.failed` | Stock reservation failed (saga compensation trigger) |
| `commerce.stock.reservation.released` | Reserved stock released (saga compensation) |
| `commerce.transfer.initiated` | Stock transfer started |
| `commerce.transfer.completed` | Stock transfer completed |
| `commerce.pricing.calculated` | Price calculated by pricing engine |
| `commerce.delivery.created` | Delivery scheduled |
| `commerce.delivery.completed` | Delivery confirmed |
| `commerce.shipment.dispatched` | Shipment dispatched to carrier |
| `commerce.shipment.in-transit` | Shipment in transit (carrier tracking update) |
| `commerce.shipment.delivered` | Shipment delivered with POD |
| `commerce.carrier.assigned` | Carrier assigned to shipment |
| `commerce.credit.hold.applied` | Credit hold applied to order (credit limit exceeded) |
| `commerce.credit.hold.released` | Credit hold released on order |
| `commerce.atp.checked` | ATP/CTP availability check performed |
| `commerce.configurator.completed` | Product configuration completed |
| `commerce.subscription.created` | Subscription created |
| `commerce.subscription.amended` | Subscription amended (upgrade/downgrade) |
| `commerce.subscription.renewed` | Subscription renewed |
| `commerce.subscription.cancelled` | Subscription cancelled |
| `commerce.subscription.billing.completed` | Subscription billing cycle completed |
| `commerce.dropship.order.created` | Drop ship order created and dispatched to supplier |
| `commerce.dropship.order.delivered` | Drop ship order delivered to end customer |
| `commerce.b2b.order.placed` | B2B portal order placed by customer |
| `commerce.b2b.order.approved` | B2B portal order approved by customer approver |
| `commerce.logistics.tracking.updated` | Real-time tracking update received (GPS/geofence) |
| `commerce.logistics.condition.alert` | Shipment condition alert triggered (temperature, humidity, shock) |
| `commerce.logistics.eta.updated` | Predictive ETA recalculated for shipment |
| `commerce.logistics.geofence.entered` | Shipment entered geofence zone |
| `commerce.logistics.geofence.exited` | Shipment exited geofence zone |
| `commerce.logistics.exception.detected` | Logistics exception detected (delay, deviation, damage) |
| `commerce.warranty.created` | Warranty policy created |
| `commerce.warranty.claim.submitted` | Warranty claim submitted |
| `commerce.warranty.claim.approved` | Warranty claim approved |
| `commerce.warranty.claim.rejected` | Warranty claim rejected |
| `commerce.warranty.claim.fulfilled` | Warranty claim fulfilled (repair/replacement) |

### Finance Events (Finance + Procurement + Treasury + Expenses + CLM + EPM)
| Event | Description |
|-------|-------------|
| `finance.journal.created` | Journal entry created |
| `finance.journal.posted` | Journal entry posted |
| `finance.journal.reversed` | Journal entry reversed |
| `finance.invoice.created` | Invoice created |
| `finance.invoice.creation.failed` | Invoice creation failed (saga compensation trigger) |
| `finance.invoice.paid` | Invoice paid |
| `finance.invoice.credit-memo` | Credit memo issued |
| `finance.payment.received` | Payment received |
| `finance.payment.made` | Payment made to supplier |
| `finance.supplier.created` | Supplier created |
| `finance.supplier.updated` | Supplier updated |
| `finance.purchase-order.created` | Purchase order created |
| `finance.purchase-order.approved` | Purchase order approved |
| `finance.purchase-order.received` | Goods received |
| `finance.account.created` | Account created in chart of accounts |
| `finance.account.updated` | Account modified in chart of accounts |
| `finance.period.closed` | Accounting period closed |
| `finance.budget.created` | Budget created |
| `finance.budget.exceeded` | Budget threshold exceeded (warning) |
| `finance.exchange-rate.updated` | Exchange rate updated |
| `finance.expense.submitted` | Expense report submitted |
| `finance.expense.approved` | Expense report approved |
| `finance.expense.rejected` | Expense report rejected |
| `finance.expense.paid` | Expense reimbursement paid |
| `finance.treasury.cash-position.updated` | Cash position updated across bank accounts |
| `finance.treasury.payment-batch.approved` | Payment batch approved for execution |
| `finance.treasury.payment-batch.executed` | Payment batch executed |
| `finance.contract.created` | Contract created |
| `finance.contract.approved` | Contract approved |
| `finance.contract.renewed` | Contract renewed |
| `finance.contract.expired` | Contract expired |
| `finance.plan.created` | Financial plan created (EPM) |
| `finance.plan.forecast.updated` | Forecast updated in financial plan |
| `finance.revenue.recognized` | Revenue recognized for a performance obligation |
| `finance.revenue.adjusted` | Revenue recognition schedule adjusted (contract modification) |
| `finance.revenue.deferred` | Revenue deferred to future period |
| `finance.credit-score.updated` | Customer credit score updated |
| `finance.credit-limit.changed` | Customer credit limit changed |
| `finance.sourcing.event.created` | Sourcing event (RFI/RFP/RFQ) created |
| `finance.sourcing.bid.submitted` | Supplier bid submitted for sourcing event |
| `finance.sourcing.event.awarded` | Sourcing event awarded to supplier(s) |
| `finance.supplier-risk.score.updated` | Supplier risk score updated |
| `finance.supplier-risk.alert.triggered` | Supplier risk alert triggered |
| `finance.supplier-risk.mitigation.created` | Supplier risk mitigation plan created |
| `finance.reconciliation.matched` | Reconciliation auto-match completed |
| `finance.reconciliation.exception.created` | Unmatched reconciliation exception created |
| `finance.reconciliation.completed` | Account reconciliation completed |
| `finance.close-task.completed` | Financial close task completed |
| `finance.profitability.analysis.completed` | Profitability analysis run completed |
| `finance.lease.created` | Lease contract created |
| `finance.lease.modified` | Lease contract modified |
| `finance.lease.payment.due` | Lease payment due |
| `finance.lease.right-of-use.adjusted` | Right-of-use asset value adjusted |
| `finance.grant.created` | Grant registered |
| `finance.grant.milestone.reached` | Grant milestone reached |
| `finance.grant.revenue.recognized` | Grant revenue recognized |
| `finance.grant.compliance.check-due` | Grant compliance check due |
| `finance.joint-venture.created` | Joint venture created |
| `finance.joint-venture.cost.allocated` | Joint venture cost allocated to partners |
| `finance.joint-venture.billing.generated` | Joint venture partner billing generated |
| `finance.intelligent-close.task.auto-assigned` | Close task automatically assigned by AI |
| `finance.intelligent-close.anomaly.detected` | Financial close anomaly detected |
| `finance.intelligent-close.auto-reconciled` | Account auto-reconciled during intelligent close |
| `finance.collection.strategy.triggered` | Collection strategy automatically triggered |
| `finance.collection.activity.created` | Collection activity created |
| `finance.cash-application.matched` | Cash receipt auto-matched to invoice |
| `finance.cash-application.unmatched` | Cash receipt could not be auto-matched |

### HR Events
| Event | Description |
|-------|-------------|
| `hr.employee.created` | Employee record created |
| `hr.employee.hired` | Employee onboarded |
| `hr.employee.updated` | Employee profile updated |
| `hr.employee.separated` | Employee offboarded |
| `hr.attendance.recorded` | Attendance check-in/out |
| `hr.leave.requested` | Leave request submitted |
| `hr.leave.approved` | Leave request approved |
| `hr.leave.rejected` | Leave request rejected |
| `hr.payroll.processed` | Payroll run completed |
| `hr.payroll.multi-country.completed` | Multi-country payroll cycle completed |
| `hr.requisition.created` | Job requisition created |
| `hr.applicant.submitted` | Job application submitted |
| `hr.review.started` | Performance review cycle started |
| `hr.review.completed` | Performance review completed |
| `hr.training.completed` | Training course completed |
| `hr.talent-review.initiated` | Talent review process initiated |
| `hr.succession.plan-created` | Succession plan created for a key position |
| `hr.workforce.simulation-completed` | Workforce modeling simulation completed |

### Manufacturing Events (Production + PLM + EAM)
| Event | Description |
|-------|-------------|
| `manufacturing.bom.created` | BOM created |
| `manufacturing.bom.updated` | BOM updated |
| `manufacturing.work-order.created` | Work order created |
| `manufacturing.work-order.started` | Work order started (production began) |
| `manufacturing.work-order.completed` | Work order completed |
| `manufacturing.quality.checked` | Quality check result recorded |
| `manufacturing.quality.failed` | Quality check failed (NCR created) |
| `manufacturing.maintenance.due` | Scheduled maintenance due |
| `manufacturing.eco.submitted` | Engineering Change Order submitted |
| `manufacturing.eco.approved` | Engineering Change Order approved |
| `manufacturing.eco.implemented` | Engineering Change Order implemented |
| `manufacturing.product-lifecycle.revision.created` | Product revision created in PLM |
| `manufacturing.product-lifecycle.phase-in` | Product phase-in initiated |
| `manufacturing.product-lifecycle.phase-out` | Product phase-out initiated |
| `manufacturing.asset.registered` | Enterprise asset registered |
| `manufacturing.asset.maintenance-completed` | Asset maintenance completed |
| `manufacturing.asset.decommissioned` | Asset decommissioned |
| `manufacturing.plan.firmed` | Production plan firmed |
| `manufacturing.plan.simulation.completed` | ASCP simulation completed |
| `manufacturing.iot.device.registered` | IoT device registered |
| `manufacturing.iot.telemetry.received` | Telemetry data received from IoT device |
| `manufacturing.iot.alert.triggered` | IoT alert rule triggered |
| `manufacturing.iot.device.offline` | IoT device went offline |
| `manufacturing.digital-twin.state.updated` | Digital twin state synchronized with physical asset |
| `manufacturing.digital-twin.simulation.completed` | Digital twin simulation completed |
| `manufacturing.digital-twin.prediction.generated` | Predictive maintenance prediction generated |
| `manufacturing.intelligence.oee.threshold-breached` | OEE dropped below configured threshold |
| `manufacturing.intelligence.downtime.categorized` | Downtime event categorized by root cause |
| `manufacturing.intelligence.energy.anomaly` | Energy consumption anomaly detected |
| `manufacturing.intelligence.predictive-maintenance.alert` | Predictive maintenance alert triggered |

### Platform Events (Notification + File + Audit + Digital Assistant + GRC)
| Event | Description |
|-------|-------------|
| `platform.notification.sent` | Notification dispatched |
| `platform.notification.failed` | Notification delivery failed |
| `platform.file.uploaded` | File uploaded and stored |
| `platform.file.deleted` | File soft-deleted |
| `platform.audit.logged` | Audit log entry created |
| `platform.digital-assistant.intent.resolved` | Digital assistant resolved user intent |
| `platform.digital-assistant.action.executed` | Digital assistant executed an action |
| `platform.digital-assistant.feedback.recorded` | User feedback on assistant response recorded |
| `platform.app-builder.app.published` | Low-code application published |
| `platform.app-builder.app.updated` | Low-code application updated |
| `platform.grc.compliance.violation.detected` | Compliance violation detected |
| `platform.grc.risk.assessment.completed` | Risk assessment completed |
| `platform.grc.sod.conflict.detected` | Segregation of Duties conflict detected |
| `platform.grc.incident.created` | GRC incident created |
| `platform.data-mask.applied` | Data masking rule applied to non-production environment |
| `platform.scheduler.job.started` | Scheduled job started execution |
| `platform.scheduler.job.completed` | Scheduled job completed |
| `platform.scheduler.job.failed` | Scheduled job failed |
| `platform.knowledge.article.created` | Knowledge base article created |
| `platform.knowledge.article.published` | Knowledge base article published |
| `platform.knowledge.article.updated` | Knowledge base article updated |
| `platform.signature.requested` | Digital signature requested |
| `platform.signature.completed` | Digital signature completed |
| `platform.rpa.bot.created` | RPA bot created |
| `platform.rpa.bot.executed` | RPA bot execution completed |
| `platform.rpa.bot.failed` | RPA bot execution failed |
| `platform.collaboration.message.posted` | Collaboration message posted |
| `platform.collaboration.task.created` | Collaboration task created |
| `platform.collaboration.task.completed` | Collaboration task completed |
| `platform.iot.device.registered` | IoT device registered in device registry |
| `platform.iot.device.certificate.issued` | IoT device certificate issued |
| `platform.iot.device.decommissioned` | IoT device decommissioned |
| `platform.idp.document.classified` | Document classified by intelligent document processing |
| `platform.idp.extraction.completed` | Data extraction from document completed |
| `platform.idp.extraction.failed` | Data extraction from document failed |
| `platform.idp.model.trained` | IDP extraction model training completed |

> **Note:** `platform.audit.logged` is an internal event published for observability. Report Service subscribes for compliance dashboards. The authoritative audit log is stored directly in `audit_db` at write time (not event-sourced).

### Tenant Events
| Event | Description |
|-------|-------------|
| `tenant.created` | New tenant provisioned |
| `tenant.updated` | Tenant settings updated |
| `tenant.suspended` | Tenant suspended (billing or admin action) |
| `tenant.feature.changed` | Feature flag value changed for a tenant |

### Auth Events
| Event | Description |
|-------|-------------|
| `auth.login.succeeded` | User login succeeded |
| `auth.login.failed` | User login failed |
| `auth.token.refreshed` | Access token refreshed |
| `auth.mfa.enabled` | MFA enabled for a user |
| `auth.password.changed` | Password changed |
| `auth.session.revoked` | Session revoked |
| `auth.step-up.requested` | Step-up authentication requested (elevated privilege) |

> **Note:** Auth events are consumed by the Platform Service for security audit logging and alerting (e.g., brute-force detection on repeated `auth.login.failed`). Auth Service does not consume events from other services.

### Config Events
| Event | Description |
|-------|-------------|
| `config.changed` | System or tenant configuration value changed |

### Workflow Events
| Event | Description |
|-------|-------------|
| `workflow.instance.started` | Workflow instance started |
| `workflow.instance.completed` | Workflow instance completed |
| `workflow.instance.failed` | Workflow instance failed |
| `workflow.step.approved` | Workflow step approved |
| `workflow.step.rejected` | Workflow step rejected |
| `workflow.step.escalated` | Workflow step escalated (timeout or threshold) |
| `workflow.step.delegated` | Workflow step delegated to another user |
| `workflow.sod.conflict.flagged` | SoD conflict flagged during approval workflow |

### CRM / Marketing Events
| Event | Description |
|-------|-------------|
| `crm.contact.created` | Contact created |
| `crm.contact.updated` | Contact updated |
| `crm.lead.created` | Lead created |
| `crm.lead.converted` | Lead converted to opportunity |
| `crm.opportunity.created` | Opportunity created |
| `crm.opportunity.stage-changed` | Opportunity moved to new stage |
| `crm.opportunity.won` | Deal won |
| `crm.opportunity.lost` | Deal lost |
| `crm.campaign.launched` | Campaign launched |
| `crm.campaign.completed` | Campaign completed |
| `crm.activity.logged` | Activity (call, email, meeting) logged |
| `crm.case.created` | Service case created |
| `crm.case.resolved` | Service case resolved |
| `crm.service-order.created` | Field service order created |
| `crm.service-order.completed` | Field service order completed |
| `crm.service-order.dispatched` | Technician dispatched for service |
| `crm.survey.created` | Survey created |
| `crm.survey.response.received` | Survey response submitted |
| `crm.territory.updated` | Sales territory assignment updated |
| `crm.quota.allocated` | Sales quota allocated |
| `crm.cdp.profile.created` | Unified customer profile created in CDP |
| `crm.cdp.profile.updated` | Unified customer profile updated |
| `crm.cdp.profile.merged` | Customer profiles merged (identity resolution) |
| `crm.cdp.segment.updated` | Customer segment membership updated |
| `crm.cdp.journey.step.completed` | Customer journey step completed |
| `crm.cdp.engagement-score.updated` | Customer engagement score recalculated |

### Project Events
| Event | Description |
|-------|-------------|
| `project.created` | Project created |
| `project.updated` | Project updated |
| `project.task.created` | Task created |
| `project.task.completed` | Task completed |
| `project.milestone.reached` | Milestone reached |
| `project.timesheet.submitted` | Timesheet entry submitted |
| `project.timesheet.approved` | Timesheet approved |
| `project.expense.submitted` | Expense report submitted |
| `project.invoice.generated` | Project billing invoice generated |
| `project.risk.identified` | Risk identified |

### Integration Events
| Event | Description |
|-------|-------------|
| `integration.sync.started` | External system sync initiated |
| `integration.sync.completed` | External system sync completed |
| `integration.sync.failed` | External system sync failed |
| `integration.import.completed` | Data import completed |
| `integration.import.failed` | Data import failed |
| `integration.master-data.matched` | Master data matching result produced |
| `integration.master-data.merged` | Master data records merged into golden record |
| `integration.master-data.quality.violation` | Data quality rule violated |
| `integration.trade-compliance.screening.completed` | Trade compliance screening completed |
| `integration.trade-compliance.screening.flagged` | Trade compliance screening flagged a match |
| `integration.trade-compliance.license.expiring` | Export license approaching expiration |

> **Note:** Integration Service primarily produces outbound events (sync/import status). External system notifications are handled via direct outbound HTTP calls or webhooks. MDM events are consumed by Report Service for data quality dashboards.

### Report Events (Analytics + Process Mining + CPM)
| Event | Description |
|-------|-------------|
| `report.process.discovered` | Business process discovered from event log analysis |
| `report.process.conformance.checked` | Process conformance check completed |
| `report.process.bottleneck.detected` | Process bottleneck identified |
| `report.process.simulation.completed` | Process simulation completed |
| `report.cpm.okr.updated` | OKR progress updated |
| `report.cpm.initiative.status-changed` | Strategic initiative status changed |
| `report.cpm.scorecard.updated` | Balanced scorecard recalculated |
| `report.narrative.package.published` | Report package published for distribution |
| `report.narrative.commentary.added` | Commentary added to report section |

### Cross-Domain Events
| Event | Publisher | Subscribers |
|-------|-----------|-------------|
| `commerce.order.created` | Commerce | Commerce (saga), Finance, Platform, Report, Workflow, CRM |
| `commerce.order.fulfilled` | Commerce | Finance, Platform, Report, CRM |
| `commerce.order.cancelled` | Commerce | Finance, Platform, Report, CRM |
| `commerce.quotation.accepted` | Commerce | Finance, Report, Workflow, CRM |
| `commerce.stock.updated` | Commerce | Manufacturing, Finance, Report |
| `commerce.customer.created` | Commerce | CRM, Report, Platform |
| `commerce.pricing.calculated` | Commerce | Report |
| `commerce.product.approved` | Commerce | Report, Integration (MDM) |
| `commerce.shipment.dispatched` | Commerce | Report, Platform |
| `commerce.credit.hold.applied` | Commerce | Finance, Platform, Report, Workflow |
| `commerce.subscription.created` | Commerce | Finance, Report |
| `commerce.subscription.cancelled` | Commerce | Finance, Report |
| `finance.purchase-order.received` | Finance | Commerce, Platform, Report |
| `finance.invoice.created` | Finance | Commerce (saga), Platform, Report |
| `finance.invoice.paid` | Finance | Report, CRM |
| `finance.exchange-rate.updated` | Finance | Commerce, Report |
| `finance.period.closed` | Finance | Report |
| `finance.expense.submitted` | Finance | Workflow, Report |
| `finance.treasury.cash-position.updated` | Finance | Report |
| `finance.contract.approved` | Finance | Report, Platform |
| `finance.plan.forecast.updated` | Finance | Report |
| `finance.revenue.recognized` | Finance | Report, Commerce (for subscription billing) |
| `manufacturing.work-order.completed` | Manufacturing | Commerce, Finance, Report |
| `manufacturing.eco.approved` | Manufacturing | Commerce, Report |
| `manufacturing.product-lifecycle.phase-out` | Manufacturing | Commerce, Report |
| `hr.employee.hired` | HR | Platform, Report, Identity (HTTP) |
| `hr.employee.separated` | HR | Platform, Report, Identity (HTTP) |
| `hr.leave.requested` | HR | Workflow, Report |
| `hr.payroll.processed` | HR | Finance, Report |
| `hr.review.completed` | HR | Report |
| `hr.talent-review.initiated` | HR | Report, Platform |
| `crm.opportunity.won` | CRM | Commerce, Report |
| `crm.opportunity.lost` | CRM | Report |
| `crm.survey.response.received` | CRM | Report |
| `crm.service-order.completed` | CRM | Commerce (parts), Report |
| `project.milestone.reached` | Project | Platform, Report, Finance |
| `project.invoice.generated` | Project | Finance, Report |
| `platform.audit.logged` | Platform | Report |
| `platform.grc.compliance.violation.detected` | Platform | Report, Workflow |
| `platform.grc.sod.conflict.detected` | Platform | Report, Workflow |
| `integration.master-data.merged` | Integration | Report |
| `integration.trade-compliance.screening.flagged` | Integration | Platform, Report, Workflow |
| `tenant.feature.changed` | Tenant | Platform (flushes Redis), Report |
| `auth.login.failed` | Auth | Platform (security audit) |
| `config.changed` | Config | All services with inboxes |
| `commerce.b2b.order.placed` | Commerce | Finance, Report |
| `commerce.logistics.exception.detected` | Commerce | Platform, Report |
| `manufacturing.iot.telemetry.received` | Manufacturing | Report, Platform |
| `manufacturing.digital-twin.prediction.generated` | Manufacturing | Commerce (spare parts), Report |
| `finance.sourcing.event.awarded` | Finance | Commerce, Report |
| `finance.supplier-risk.alert.triggered` | Finance | Platform, Report, Workflow |
| `finance.reconciliation.completed` | Finance | Report |
| `crm.cdp.profile.merged` | CRM | Commerce, Report |
| `crm.cdp.segment.updated` | CRM | Commerce, Platform (notifications) |
| `report.process.bottleneck.detected` | Report | Workflow, Platform |
| `platform.rpa.bot.failed` | Platform | Report, Workflow |
| `finance.lease.created` | Finance | Report, Platform |
| `finance.intelligent-close.anomaly.detected` | Finance | Report, Platform, Workflow |
| `finance.cash-application.matched` | Finance | Commerce, Report |
| `commerce.warranty.claim.submitted` | Commerce | Finance, Report |
| `commerce.warranty.claim.fulfilled` | Commerce | Manufacturing, Finance, Report |
| `platform.idp.extraction.completed` | Platform | Finance (auto-invoice), Report |
| `manufacturing.intelligence.oee.threshold-breached` | Manufacturing | Report, Platform |

> **Note:** Report Service subscribes to key business events for analytics aggregation. Workflow Service subscribes to events that trigger approval workflows. Core services (Auth, Identity, Tenant, Config) do not have inbox queues. Identity receives updates via direct service-to-service HTTP calls.

---

*See [Event-Driven Architecture Overview](./overview.md) for exchange structure, schema, and versioning.*
*See [Saga Patterns](./sagas.md) for distributed transaction patterns.*
