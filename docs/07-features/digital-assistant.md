# Digital Assistant

NLP-powered conversational interface with intent recognition, action execution, and context management across multiple channels, managed by the Platform Service.

## Overview

The Digital Assistant provides an intelligent conversational interface that enables users to interact with MSERP using natural language. It supports intent recognition to map user utterances to system actions, multi-turn context management for complex workflows, and multi-channel delivery via web, mobile, Slack, and Microsoft Teams. The assistant integrates with all MSERP services to execute business actions (create orders, approve invoices, look up customer data) and with the knowledge base for FAQ-style answers. Conversation history and user preferences are persisted for personalization and audit.

## Architecture Diagram

```
┌──────────────────────────────────────────────────────────────────────┐
│                        Digital Assistant                              │
│                                                                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │
│  │ Web Chat     │  │ Mobile SDK   │  │ Slack / Teams│               │
│  │ Widget       │  │              │  │ Adapter      │               │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘               │
│         │                 │                  │                        │
│         └─────────────────┼──────────────────┘                        │
│                           ▼                                          │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │                   Channel Adapter Layer                         │  │
│  │  Normalizes input from each channel into a canonical message   │  │
│  └────────────────────────────┬───────────────────────────────────┘  │
│                               ▼                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐   │
│  │ NLP Engine   │  │ Context      │  │ Action Router            │   │
│  │ (Intent      │  │ Manager      │  │ (dispatches to service   │   │
│  │  Recognition,│  │ (conversation│  │  APIs based on intent)   │   │
│  │  Entity      │  │  state,      │  │                          │   │
│  │  Extraction) │  │  slot fill)  │  │                          │   │
│  └──────┬───────┘  └──────┬───────┘  └────────────┬─────────────┘   │
│         │                 │                        │                  │
│         └─────────────────┼────────────────────────┘                  │
│                           ▼                                          │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │              Response Generator                                 │  │
│  │  Templates, cards, quick replies, knowledge base answers       │  │
│  └────────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────┘
```

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **NLP Engine** | Platform Service (ONNX + Candle) | Intent classification and named-entity extraction models running on-device; intent confidence scoring with configurable thresholds; fallback to knowledge-base FAQ search for unrecognised intents |
| **Context Manager** | Platform Service (Rust) | Multi-turn conversation state machine with slot-filling for required parameters; context window of configurable length; per-session and per-user persistent context |
| **Action Router** | Platform Service (Rust) | Maps recognised intents to service API calls; handles authentication token propagation, tenant context (X-Tenant-Id), and request correlation; supports parallel action execution for multi-intent queries |
| **Channel Adapters** | Platform Service (Rust) | Adapters for web WebSocket, mobile push, Slack Bolt API, and Microsoft Teams Bot Framework; normalises channel-specific payloads into canonical message format; handles channel-specific rich responses (cards, buttons, carousels) |
| **Knowledge Base Integration** | Platform Service + Knowledge Management | FAQ lookup against knowledge articles; semantic search for answer ranking; article reference in responses with direct links |
| **Conversation History** | Platform Service (PostgreSQL) | Full conversation persistence per user with search; used for personalisation, audit, and analytics; GDPR-compliant retention and deletion |
| **User Preferences** | Platform Service (Rust) | Per-user preferences for language, response verbosity, default channels, notification frequency; learned preferences from interaction patterns |
| **Response Generator** | Platform Service (Rust) | Template-based and dynamic response generation; supports text, cards, quick-reply buttons, and adaptive cards; i18n-aware response templates |

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Channel Adapter | Multi-channel input/output normalisation | Platform Service |
| NLP Engine | Intent classification, entity extraction, confidence scoring | Platform Service |
| Context Manager | Conversation state, slot filling, session tracking | Platform Service |
| Action Router | Intent-to-API dispatch, token propagation, tenant scoping | Platform Service |
| Response Generator | Template rendering, rich message formatting, i18n | Platform Service |
| Knowledge Lookup | FAQ search and article retrieval | Platform Service |
| Conversation Store | Conversation and message persistence, history search | Platform Service |
| Preference Manager | User preference CRUD, learning from behaviour | Platform Service |
| Analytics Collector | Usage metrics, intent accuracy, response quality | Platform Service |

## Multi-Channel Support

| Channel | Transport | Rich Responses | Auth Method |
|---------|-----------|---------------|-------------|
| Web Chat | WebSocket | Cards, buttons, carousels | SSO token |
| Mobile SDK | HTTP + Push | Cards, buttons, deep links | OAuth2 token |
| Slack | Bolt API (WebSocket) | Blocks, attachments, modals | OAuth2 (workspace install) |
| Microsoft Teams | Bot Framework | Adaptive cards, task modules | OAuth2 (app registration) |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| CRM Service | CRM Service | Internal API | Look up leads, opportunities, contacts; create activities |
| Commerce Service | Commerce Service | Internal API | Create/query orders, quotes, deliveries; check inventory |
| Finance Service | Finance Service | Internal API | Query invoices, approve payments, check budgets |
| HCM Service | HCM Service | Internal API | Look up employees, submit leave requests, view payslips |
| Project Service | Project Service | Internal API | Query project status, log time, manage tasks |
| Report Service | Report Service | Internal API | Generate on-demand reports, query dashboards |
| Knowledge Management | Platform Service | Internal API | FAQ lookup, article search and retrieval |
| Workflow Engine | Platform Service | Event-driven | Initiate and progress workflow instances |
| Notification Service | Platform Service | Event-driven | Proactive notifications to users via assistant |
| Search | Platform Service | Internal API | Cross-domain unified search queries |
| Auth Service | Auth Service | Internal API | Token validation and user context resolution |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `platform.assistant.conversation.started` | `{conversation_id, user_id, channel, started_at}` | New conversation initiated | Analytics, Audit Trail |
| `platform.assistant.conversation.ended` | `{conversation_id, user_id, message_count, ended_at}` | Conversation closed or timed out | Analytics, Audit Trail |
| `platform.assistant.intent.recognized` | `{conversation_id, message_id, intent, entities[], confidence}` | Intent classification completes | Analytics, Action Router |
| `platform.assistant.action.executed` | `{conversation_id, message_id, intent, action, target_service, success, duration_ms}` | Action API call completes | Analytics, Audit Trail |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `platform.notification.created` | Platform Service | Push proactive notification to user via active channel |
| `platform.workflow.step-completed` | Platform Service | Notify user of workflow progress |
| `commerce.order.created` | Commerce Service | Proactive order confirmation to user |
| `finance.invoice.approved` | Finance Service | Proactive approval notification |
| `knowledge.article.published` | Platform Service | Update FAQ index |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `assistant_conversation` | `id, tenant_id, user_id, channel, status, started_at, ended_at` | Has many `assistant_message` |
| `assistant_message` | `id, conversation_id, role, content, intent, entities[], channel_payload` | Belongs to `assistant_conversation` |
| `assistant_intent` | `id, tenant_id, name, description, training_phrases[], action_config, is_active` | Referenced by `assistant_message` |
| `assistant_action` | `id, intent_id, target_service, api_endpoint, method, parameter_mapping[]` | Belongs to `assistant_intent` |
| `assistant_user_preference` | `user_id, tenant_id, language, verbosity, default_channel, notification_prefs` | Per-user singleton |

## Cross-References

- [Knowledge Management](knowledge-management.md) — Knowledge base for FAQ answers
- [Enterprise Collaboration](collaboration.md) — Bot presence in collaboration channels
- [Intelligent Process Automation](intelligent-process-automation.md) — RPA bot integration
- [Search](search.md) — Cross-domain search queries
- [Multi-Tenancy](multi-tenancy.md) — Tenant context propagation
- [Architecture Overview](../01-architecture/overview.md) — System context
- [Platform Service](../06-services/platform.md) — Platform service specification
