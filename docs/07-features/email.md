# Email

Email templating, sending, receiving, delivery tracking, bounce management, and email-to-case integration, managed by the Platform Service.

## Overview

The Email feature provides a comprehensive email infrastructure for MSERP, covering template management with variables and i18n, outbound sending via SMTP, inbound processing via IMAP, delivery tracking across the full lifecycle (sent, delivered, bounced, opened, clicked), bounce management with suppression lists, and email-to-case routing for CRM integration. Bulk email is supported with rate limiting and throttling to respect provider limits and prevent abuse. All email activity is tracked per tenant with full audit capability.

## Architecture Diagram

```
┌──────────────────────────────────────────────────────────────────────┐
│                          Email System                                 │
│                                                                       │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────────────┐ │
│  │ Template     │  │ Outbound     │  │ Inbound Processing         │ │
│  │ Engine       │  │ Pipeline     │  │                            │ │
│  │ ┌──────────┐ │  │ ┌──────────┐ │  │ ┌──────────┐ ┌──────────┐ │ │
│  │ │ Variable │ │  │ │ Rate     │ │  │ │ IMAP     │ │ Email-to │ │ │
│  │ │ Subst.   │ │  │ │ Limiter  │ │  │ │ Poller   │ │ Case     │ │ │
│  │ └──────────┘ │  │ └──────────┘ │  │ └──────────┘ └──────────┘ │ │
│  │ ┌──────────┐ │  │ ┌──────────┐ │  │ ┌──────────┐ ┌──────────┐ │ │
│  │ │ i18n     │ │  │ │ SMTP     │ │  │ │ Attachment│ │ Auto-    │ │ │
│  │ │ Rendering│ │  │ │ Sender   │ │  │ │ Extract  │ │ Reply    │ │ │
│  │ └──────────┘ │  │ └──────────┘ │  │ └──────────┘ └──────────┘ │ │
│  │ ┌──────────┐ │  │ ┌──────────┐ │  └────────────────────────────┘ │
│  │ │ Layout   │ │  │ │ Queue    │ │                                 │
│  │ │ Builder  │ │  │ └──────────┘ │  ┌────────────────────────────┐ │
│  │ └──────────┘ │  └──────────────┘  │ Tracking & Analytics        │ │
│  └──────────────┘                     │ ┌──────────┐ ┌──────────┐ │ │
│                                        │ │ Webhook  │ │ Bounce   │ │ │
│                                        │ │ Receiver │ │ Handler  │ │ │
│                                        │ └──────────┘ └──────────┘ │ │
│                                        │ ┌──────────┐ ┌──────────┐ │ │
│                                        │ │ Link     │ │ Suppres- │ │ │
│                                        │ │ Wrapper  │ │ sion List│ │ │
│                                        │ └──────────┘ └──────────┘ │ │
│                                        └────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Template Engine** | Platform Service (Rust + Tera) | HTML and text template rendering with variable substitution, conditional blocks, and loops; per-locale template variants; versioned templates with draft/published states |
| **SMTP Sending** | Platform Service (Rust + lettre) | Outbound email delivery via configurable SMTP relay; connection pooling, TLS enforcement; Message-ID and custom header management for tracking correlation |
| **IMAP Receiving** | Platform Service (Rust + imap) | Inbound email polling via IMAP IDLE or periodic fetch; multi-folder support; attachment extraction and storage via File Service; auto-reply and forwarding rules |
| **Rate Limiter** | Platform Service (Rust) | Token-bucket rate limiting per tenant and per sender; configurable hourly/daily limits; bulk email queue with paced delivery; backpressure on provider rate-limit responses |
| **Delivery Tracking** | Platform Service (Rust) | Webhook receivers for provider events (sent, delivered, bounced, opened, clicked); link wrapping for click tracking; open tracking via 1px beacon; event correlation to original message |
| **Bounce Management** | Platform Service (Rust) | Hard bounce detection (permanent failures) adds recipients to suppression list; soft bounce tracking with configurable retry; bounce classification (spam, mailbox-full, invalid-domain, etc.) |
| **Suppression List** | Platform Service (PostgreSQL) | Per-tenant suppression list; automatic addition on hard bounce; manual add/remove by admin; checked before every send to prevent reputation damage |
| **Email-to-Case** | Platform Service + CRM Service | Inbound emails matching configured rules (to-address, subject pattern) are automatically converted to CRM cases; attachment preservation; auto-acknowledgement to sender |
| **Bulk Email** | Platform Service (Rust + PostgreSQL) | Bulk send with recipient list, template, and personalisation variables; batch processing with rate limiting; per-recipient tracking; unsubscribe link management |
| **i18n** | Platform Service | Locale-aware template selection; date, number, and currency formatting in templates; RTL layout support for Arabic/Hebrew |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Template Manager | Template CRUD, versioning, preview, i18n variants | Platform Service |
| Rendering Engine | Variable substitution, layout rendering, inline CSS | Platform Service |
| SMTP Sender | Outbound delivery, connection management, retry | Platform Service |
| IMAP Receiver | Inbound polling, parsing, attachment handling | Platform Service |
| Rate Limiter | Per-tenant and per-sender throttling | Platform Service |
| Tracking Engine | Open/click/bounce event collection and correlation | Platform Service |
| Link Wrapper | URL rewriting for click tracking | Platform Service |
| Bounce Handler | Bounce classification, suppression list management | Platform Service |
| Email-to-Case Router | Rule-based inbound routing to CRM | Platform Service → CRM Service |
| Bulk Send Manager | Bulk campaign orchestration, batch processing | Platform Service |
| Unsubscribe Manager | Opt-out processing, preference centre | Platform Service |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| CRM Service (Cases) | CRM Service | Internal API + Events | Email-to-case auto-creation, case update on reply |
| CRM Service (Contacts) | CRM Service | Internal API | Recipient lookup, contact activity logging |
| Commerce Service | Commerce Service | Events | Order confirmation, shipment notification emails |
| Finance Service | Finance Service | Events | Invoice delivery, payment reminder emails |
| HCM Service | HCM Service | Events | Payslip delivery, onboarding emails |
| Notification Service | Platform Service | Internal API | Email as a notification delivery channel |
| File Service | Platform Service | Internal API | Attachment storage and retrieval |
| Auth Service | Auth Service | Events | Password reset, email verification emails |
| Workflow Engine | Workflow Service | Event-driven | Email sending steps in workflows |
| Search | Platform Service | Internal API | Email content indexing for search |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `platform.email.sent` | `{message_id, tenant_id, template_id, recipients[], sent_at}` | Email handed to SMTP relay | Analytics, Audit Trail |
| `platform.email.delivered` | `{message_id, recipient, delivered_at}` | Delivery confirmation from provider | Analytics |
| `platform.email.bounced` | `{message_id, recipient, bounce_type, bounce_detail, bounced_at}` | Bounce received from provider | Bounce Handler, Suppression List, Analytics |
| `platform.email.opened` | `{message_id, recipient, opened_at, user_agent}` | Open tracking beacon triggered | Analytics, CRM Service |
| `platform.email.clicked` | `{message_id, recipient, url, clicked_at}` | Click tracking link visited | Analytics, CRM Service |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `commerce.order.created` | Commerce Service | Send order confirmation email |
| `commerce.shipment.dispatched` | Commerce Service | Send shipment notification email |
| `finance.invoice.created` | Finance Service | Send invoice email to customer |
| `finance.payment.received` | Finance Service | Send payment receipt email |
| `crm.case.created` | CRM Service | Send case acknowledgement email |
| `identity.user.created` | Auth Service | Send welcome / verification email |
| `auth.password.changed` | Auth Service | Send password reset email |
| `platform.notification.dispatched` | Platform Service | Send email for notification channel |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `email_template` | `id, tenant_id, name, subject, html_body, text_body, locale, variables[], version, status` | Referenced by `email_message` |
| `email_message` | `id, tenant_id, template_id, from, to[], cc[], bcc[], subject, html_body, text_body, status, message_id_header, created_at, sent_at` | Has many `email_tracking`, `email_attachment` |
| `email_tracking` | `id, message_id, recipient, event_type, event_at, detail, user_agent, url` | Belongs to `email_message` |
| `email_attachment` | `id, message_id, file_ref, filename, content_type, size` | Belongs to `email_message` |
| `email_suppression` | `id, tenant_id, email, reason, source, created_at` | Per-tenant, per-address singleton |
| `email_bulk_campaign` | `id, tenant_id, template_id, recipient_list_ref, status, total, sent, failed, created_at` | Has many `email_message` |

## Cross-References

- [Digital Assistant](digital-assistant.md) — Email channel for assistant notifications
- [Enterprise Collaboration](collaboration.md) — Email-to-channel forwarding
- [Knowledge Management](../06-services/platform.md) — Email-triggered knowledge article suggestions (Knowledge Management module)
- [Multi-Tenancy](multi-tenancy.md) — Per-tenant email configuration and isolation
- [Architecture Overview](../01-architecture/overview.md) — System context
- [Platform Service](../06-services/platform.md) — Platform service specification
