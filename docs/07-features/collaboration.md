# Enterprise Collaboration

Channel-based messaging, document co-editing, task management, and unified search with deep business context integration, managed by the Platform Service.

## Overview

Enterprise Collaboration provides a unified workspace for teams to communicate, share documents, manage tasks, and coordinate work — all deeply integrated with MSERP's business context. Rather than a standalone chat tool, collaboration features are embedded in business workflows: a sales order discussion links to the order record, a project channel surfaces project milestones, and a support case thread connects to the CRM case. The system supports real-time messaging with channels, presence-based availability, document co-editing with version control, task boards integrated with projects and workflows, and unified search across all content types.

## Architecture & Implementation

| Aspect | Implementation | Detail |
|--------|---------------|--------|
| **Channel-Based Messaging** | Platform Service (Rust + WebSocket) | Persistent channel-based messaging with threads, reactions, mentions, file attachments, and rich message formatting; channels can be linked to business objects (orders, projects, cases) for contextual collaboration |
| **Document Co-Editing** | Platform Service (CRDT-based) | Real-time collaborative editing using Conflict-free Replicated Data Types (CRDTs) for documents, spreadsheets, and diagrams; version control with branching and merge; commenting and review workflows |
| **Task Management** | Platform Service (Rust) | Personal and team task boards with Kanban and list views; task creation from messages, documents, and business objects; integration with Project Service for project tasks and Platform Workflow for workflow tasks |
| **Presence & Availability** | Platform Service (WebSocket) | Real-time user presence (online, away, busy, offline) with calendar integration for automatic status updates; do-not-disturb mode with exception rules |
| **Unified Search** | Platform Service + Elasticsearch | Single search interface across messages, documents, tasks, knowledge articles, and business records; faceted navigation with filters by type, date, author, and business context |
| **Notification Integration** | Platform Service + Notification Service | Consolidated notification feed from messages, mentions, task assignments, document reviews, and business events; configurable notification preferences per channel and type |
| **Mobile Support** | Platform Service (React Native SDK) | Native mobile experience with push notifications, offline message caching, document viewing, and task management; responsive design for tablet |
| **Business Context Linking** | Platform Service | Deep linking of collaboration artifacts to business objects: order context, project context, case context, customer context; context panel shows related business data |

## Channel Architecture

```
┌───────────────────────────────────────────────────────────────────┐
│                     Collaboration Channels                         │
│                                                                    │
│  ┌── General Channels ────────┐  ┌── Business Context Channels ──┐│
│  │  #general                  │  │  #order-ORD-2024-1234         ││
│  │  #engineering              │  │  #project-PRJ-5001            ││
│  │  #announcements            │  │  #case-CS-7890               ││
│  │  #random                   │  │  #customer-CUST-456          ││
│  │  #help-desk                │  │  #vendor-VEN-123             ││
│  └────────────────────────────┘  └───────────────────────────────┘│
│                                                                    │
│  ┌── Direct Messages ────────┐  ┌── Cross-Organization ─────────┐│
│  │  @user1 ↔ @user2          │  │  Shared channels with          ││
│  │  Group DMs (up to 8)      │  │  partners/suppliers            ││
│  └────────────────────────────┘  └───────────────────────────────┘│
│                                                                    │
│  Features per channel:                                             │
│  • Pinned messages and documents                                   │
│  • Channel description and topic                                   │
│  • Member management (roles: admin, member, guest)                 │
│  • Notification preferences per channel                            │
│  • Linked business objects (context panel)                         │
│  • Integrated task board                                           │
└───────────────────────────────────────────────────────────────────┘
```

## Modules & Components

| Component | Responsibility | Service |
|-----------|---------------|---------|
| Channel Manager | Channel CRUD, membership, permissions | Platform Service |
| Message Service | Message persistence, threading, search | Platform Service |
| Presence Service | Real-time presence tracking, status | Platform Service |
| Co-Edit Engine | CRDT-based collaborative editing | Platform Service |
| Task Board Manager | Task CRUD, assignments, state transitions | Platform Service |
| Search Engine | Unified cross-content search | Platform Service + Elasticsearch |
| Notification Router | Collaboration notification delivery | Platform Service + Notification Service |
| Context Linker | Business object association and display | Platform Service |
| Mobile Push Service | Push notification delivery to mobile | Platform Service |

## Business Context Integration

| Business Object | Service | Context Display | Actions |
|----------------|---------|----------------|---------|
| Sales Order | Commerce Service | Order summary, status, line items, total | View order, create invoice, ship |
| Project | Project Service | Project timeline, milestones, team, budget | View project, create task, log time |
| Support Case | CRM Service | Case details, customer, priority, SLA | Update case, assign, escalate |
| Customer | CRM Service | Customer profile, recent orders, interactions | View profile, create order, log call |
| Vendor | Commerce/Manufacturing | Vendor profile, active POs, performance | View profile, create PO, rate |
| Invoice | Finance Service | Invoice details, amount, status, due date | Approve, pay, dispute |
| Task (workflow) | Platform Service | Workflow task details, form data | Approve, reject, reassign |
| Knowledge Article | Platform Service | Article content, rating, related articles | View, edit, share |

## Integration Points

| Integrates With | Service | Integration Type | Description |
|----------------|---------|-----------------|-------------|
| Knowledge Management | Platform Service | Internal API | Knowledge articles in search, article sharing in channels |
| Content Management | Platform Service | Internal API | Document storage, version control, compliance archive |
| Project Service | Project Service | Internal API | Project tasks, milestones, team members for context |
| Commerce Service | Commerce Service | Internal API | Order, quotation, customer context linking |
| CRM Service | CRM Service | Internal API | Case, opportunity, contact context linking |
| Finance Service | Finance Service | Internal API | Invoice, expense, payment context linking |
| Notification Service | Platform Service | Event-driven | Message notifications, task assignments, mentions |
| Workflow Engine | Platform Service | Event-driven | Workflow task creation from messages, approvals |
| Search (Elasticsearch) | Platform Service | Internal API | Unified search indexing and querying |
| Digital Assistant | Platform Service | Internal API | Bot responses in channels, AI-assisted summaries |

## Event Flow

### Domain Events Produced

| Event | Payload | Trigger | Consumers |
|-------|---------|---------|-----------|
| `platform.collaboration.channel.created` | `{channel_id, name, type, creator, business_context}` | New channel created | Notification Service, Search Indexer |
| `platform.collaboration.message.posted` | `{message_id, channel_id, author, mentions[], attachments[]}` | New message posted | Notification Service, Search Indexer |
| `platform.collaboration.message.reaction` | `{message_id, user_id, reaction_type}` | Reaction added to message | Notification Service |
| `platform.collaboration.document.shared` | `{document_id, channel_id, shared_by, version}` | Document shared in channel | Search Indexer, Content Management |
| `platform.collaboration.task.created` | `{task_id, channel_id, assignee, due_date, business_ref}` | Task created from collaboration | Notification Service, Project Service |
| `platform.collaboration.task.completed` | `{task_id, completed_by, completed_at}` | Task marked complete | Notification Service, Project Service |
| `platform.collaboration.presence.updated` | `{user_id, status, updated_at}` | User presence changes | All connected clients (WebSocket) |
| `platform.collaboration.document.edited` | `{document_id, editor, edit_summary, version}` | Co-edit save point | Content Management, Search Indexer |

### Domain Events Consumed

| Event | Source | Action |
|-------|--------|--------|
| `commerce.order.created` | Commerce Service | Auto-create order context channel |
| `project.project.created` | Project Service | Auto-create project context channel |
| `crm.case.created` | CRM Service | Auto-create case context channel |
| `finance.invoice.created` | Finance Service | Post notification to relevant channel |
| `workflow.step.approved` | Workflow Service | Create collaboration task |
| `platform.notification.sent` | Platform Service | Deliver notification to collaboration feed |

## Data Model Reference

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| `collab_channel` | `id, name, type, business_object_type, business_object_id, creator_id` | Has many `collab_message`, `collab_member` |
| `collab_member` | `id, channel_id, user_id, role, joined_at, notification_prefs` | Belongs to `collab_channel` |
| `collab_message` | `id, channel_id, parent_message_id, author_id, content, mentions[], attachments[]` | Belongs to `collab_channel`, has many `collab_reaction` |
| `collab_reaction` | `id, message_id, user_id, reaction_type` | Belongs to `collab_message` |
| `collab_task` | `id, channel_id, title, assignee_id, status, due_date, business_ref_type, business_ref_id` | Belongs to `collab_channel` |
| `collab_document` | `id, channel_id, content_id, version, lock_holder` | Belongs to `collab_channel`, references content management |
| `collab_presence` | `user_id, status, last_seen_at, device_type` | Per-user singleton |

## Cross-References

- [Knowledge Management](../06-services/platform.md) — Knowledge base features (Knowledge Management module)
- [Content Management](content-management.md) — Document management and lifecycle
- [Digital Assistant](digital-assistant.md) — AI-powered bot in channels
- [Architecture Overview](../01-architecture/overview.md) — System context
- [Platform Service](../06-services/platform.md) — Platform service specification
