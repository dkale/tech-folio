# CQRS (Command Query Responsibility Segregation) — Interview Q&A

## Table of Contents

### Basic Level
| # | Question |
|---|---|
| 1 | [What does CQRS stand for?](#1-what-does-cqrs-stand-for) |
| 2 | [What is the main idea behind CQRS?](#2-what-is-the-main-idea-behind-cqrs) |
| 3 | [How is CQRS different from traditional CRUD?](#3-how-is-cqrs-different-from-a-traditional-crud-architecture) |
| 4 | [What are Commands and Queries?](#4-what-are-commands-and-queries-in-cqrs) |
| 5 | [Why separate read and write models?](#5-why-do-we-separate-read-and-write-models) |
| 6 | [Advantages of CQRS](#6-what-are-some-advantages-of-using-cqrs) |
| 7 | [Disadvantages / challenges](#7-what-are-some-disadvantages-or-challenges-of-implementing-cqrs) |
| 8 | [CQRS vs Event Sourcing](#8-is-cqrs-the-same-as-event-sourcing) |
| 9 | [CQRS without Event Sourcing?](#9-can-you-implement-cqrs-without-event-sourcing) |
| 10 | [When is CQRS most useful?](#10-in-what-type-of-applications-is-cqrs-most-useful) |

### Intermediate Level
| # | Question |
|---|---|
| 11 | [How does CQRS improve scalability?](#11-how-does-cqrs-improve-scalability-and-performance) |
| 12 | [Role of the read model](#12-what-role-does-the-read-model-play-in-cqrs) |
| 13 | [Role of the write model](#13-what-role-does-the-write-model-play-in-cqrs) |
| 14 | [Synchronizing read and write models](#14-how-are-read-and-write-models-synchronized) |
| 15 | [Eventual consistency and CQRS](#15-what-is-eventual-consistency-and-how-does-it-relate-to-cqrs) |
| 16 | [Designing a CQRS system in .NET](#16-how-would-you-design-a-cqrs-system-in-net) |
| 17 | [Common .NET CQRS tools/libraries](#17-what-are-common-tools-or-libraries-used-for-cqrs-in-net) |
| 18 | [Handling validation in CQRS](#18-how-would-you-handle-validation-in-a-cqrs-based-system) |
| 19 | [Read/write models out of sync](#19-what-happens-if-the-read-and-write-models-get-out-of-sync) |
| 20 | [CQRS in monoliths vs microservices](#20-can-cqrs-work-in-a-monolithic-application-or-is-it-only-for-microservices) |

### Advanced Level
| # | Question |
|---|---|
| 21 | [Event Sourcing + CQRS](#21-explain-how-event-sourcing-complements-cqrs) |
| 22 | [Cross-aggregate transactions](#22-how-would-you-handle-transactions-that-span-multiple-aggregates-in-cqrs) |
| 23 | [Ensuring consistency between stores](#23-how-do-you-ensure-consistency-between-command-and-query-data-stores) |
| 24 | [Eventual consistency in UI](#24-how-do-you-deal-with-eventual-consistency-in-a-user-facing-application) |
| 25 | [Replaying events to rebuild read model](#25-what-strategies-do-you-use-for-replaying-events-to-rebuild-the-read-model) |
| 26 | [Versioning and schema changes](#26-how-do-you-manage-versioning-and-schema-changes-in-event-sourced-systems) |
| 27 | [Implementing auditing](#27-how-would-you-implement-auditing-in-a-cqrs-system) |
| 28 | [Messaging patterns (Service Bus, Kafka)](#28-what-messaging-patterns-are-commonly-used-between-the-write-and-read-sides) |
| 29 | [CQRS vs DB replication](#29-whats-the-difference-between-cqrs-and-a-simple-readwrite-database-replication-setup) |
| 30 | [Anti-patterns](#30-what-are-the-typical-anti-patterns-when-implementing-cqrs) |

### Scenario-Based
| # | Question |
|---|---|
| 31 | [User profile update flow](#31-suppose-a-user-updates-their-profile-information) |
| 32 | [Near real-time dashboard](#32-dashboard-showing-aggregated-metrics-in-near-real-time) |
| 33 | [SQL + NoSQL (RavenDB) design](#33-designing-read-and-write-sides-with-sql--nosql) |
| 34 | [Debugging / monitoring command flow](#34-debugging-or-monitoring-the-flow-of-a-command) |
| 35 | [Testing command vs query handlers](#35-testing-a-cqrs-command-handler-vs-a-query-handler) |

---

## Basic Level

### 1. What does CQRS stand for?

CQRS stands for **Command Query Responsibility Segregation**. It is an architectural pattern that separates the responsibility of handling commands (writes/mutations) from queries (reads) into distinct models.

### 2. What is the main idea behind CQRS?

The **read side** and **write side** of an application have fundamentally different requirements — reads are typically high-volume and need to be fast/denormalized, while writes need strong consistency and business rule enforcement. CQRS separates them into two distinct models so each can be optimized independently.

### 3. How is CQRS different from a traditional CRUD architecture?

In traditional CRUD, a single model (typically one set of DTOs/entities and one database) handles both reads and writes. In CQRS:

- **Writes** go through command handlers that enforce business rules and persist to a write store.
- **Reads** go through query handlers that return data from a read-optimized store (which may be a separate database, materialized view, or cache).

The read model can be denormalized and shaped for specific UI needs, while the write model stays normalized and focused on domain integrity.

### 4. What are Commands and Queries in CQRS?

- **Commands** are intent-based messages that represent a request to change state (e.g., `PlaceOrderCommand`, `UpdateAddressCommand`). They are imperative, carry data needed for the mutation, and return no domain data (at most an acknowledgement or ID).
- **Queries** are requests for data that do **not** change state (e.g., `GetOrderByIdQuery`, `ListActiveUsersQuery`). They return data and have zero side effects.

This aligns with the **CQS (Command-Query Separation)** principle by Bertrand Meyer, extended to the architectural level.

### 5. Why do we separate read and write models?

Reads and writes have different optimization needs:

- **Write model** needs normalization, referential integrity, concurrency control, and business rule validation.
- **Read model** needs fast queries, denormalized data, and shapes tailored to specific views/screens.

Combining both in one model forces compromises — either reads are slow (joins across normalized tables) or writes lose integrity (denormalized storage). Separation lets each side be independently optimized, scaled, and evolved.

### 6. What are some advantages of using CQRS?

- **Independent scaling** — scale read and write sides separately (reads typically 10–100× more frequent).
- **Optimized read models** — denormalized views, caches, or search indexes tailored to specific queries.
- **Simpler command handlers** — focused on business logic without worrying about query needs.
- **Better security** — fine-grained authorization on commands vs. queries.
- **Flexibility** — use different storage technologies for each side (e.g., SQL for writes, Elasticsearch for reads).
- **Auditability** — when combined with Event Sourcing, you get a full history of state changes.

### 7. What are some disadvantages or challenges of implementing CQRS?

- **Increased complexity** — two models, synchronization logic, more moving parts.
- **Eventual consistency** — the read model may lag behind the write model, which complicates UX.
- **Infrastructure overhead** — may require message buses, separate databases, projection workers.
- **Learning curve** — teams unfamiliar with the pattern can misapply it.
- **Overkill for simple domains** — CRUD-heavy apps with low read/write asymmetry don't benefit much.
- **Debugging difficulty** — tracing a command through events to a read model projection is harder than debugging a single CRUD operation.

### 8. Is CQRS the same as Event Sourcing?

No. They are **complementary but independent** patterns:

- **CQRS** separates read and write models — it says nothing about *how* state is persisted.
- **Event Sourcing** persists state as a sequence of immutable domain events rather than current state — it says nothing about separating reads from writes.

They pair well because Event Sourcing naturally produces events that can be projected into read-optimized views (the query side of CQRS). But you can use CQRS with a standard relational write model, and you can use Event Sourcing without CQRS.

### 9. Can you implement CQRS without Event Sourcing?

Yes. A common approach:

- **Write side:** Standard Entity Framework / Dapper with a normalized SQL database. Command handlers validate and persist using traditional `UPDATE`/`INSERT`.
- **Read side:** A separate read database (or even the same database with denormalized views/materialized views). After a write succeeds, publish a domain event or use Change Data Capture (CDC) to update the read model.

This is actually the **more common** real-world CQRS implementation. Event Sourcing adds significant complexity and is only warranted when you need a full audit trail or temporal queries.

### 10. In what type of applications is CQRS most useful?

CQRS shines in applications with:

- **High read/write asymmetry** — dashboards, reporting systems, e-commerce catalogs (many reads, fewer writes).
- **Complex business domains** — where write-side logic is rich and distinct from read-side projections (e.g., financial systems, booking engines).
- **Collaborative domains** — multiple users modifying shared data with conflict resolution needs.
- **Event-driven architectures** — microservices communicating via events naturally benefit from separate read projections.
- **Audit/compliance requirements** — when combined with Event Sourcing, every state change is recorded.

**Not** a good fit for simple CRUD apps, internal tools with few users, or domains where reads and writes are nearly identical in shape.

---

## Intermediate Level

### 11. How does CQRS improve scalability and performance?

- **Read scaling:** The read model can be replicated across multiple read replicas, cached aggressively, or served from a search engine (Elasticsearch) or in-memory store (Redis) — without affecting write throughput.
- **Write scaling:** The write model can use partitioning/sharding strategies optimized for write patterns without worrying about query performance.
- **Independent deployment:** Read and write services can be scaled independently — if reads spike (e.g., Black Friday browsing), you add read replicas without touching the write infrastructure.
- **Denormalized reads eliminate joins:** Read models pre-compute and flatten data, so queries return in O(1) lookups instead of expensive multi-table joins.

### 12. What role does the read model play in CQRS?

The read model is a **query-optimized projection** of the domain state:

- Serves data in the exact shape the UI/API consumers need (no transformation at query time).
- Denormalized and precomputed — e.g., an `OrderSummaryView` table with customer name, total, item count all in one row.
- Disposable and rebuildable — if it gets corrupted, replay events or re-project from the write store.
- Supports multiple projections — one write event can update a dashboard view, a search index, and a notification feed simultaneously.

The read model never enforces business rules — it's a passive reflection of the write side's decisions.

### 13. What role does the write model play in CQRS?

The write model is the **source of truth** for the domain:

- Accept commands and validate them against business rules and invariants.
- Enforce domain integrity — e.g., "an order cannot be placed if inventory is zero."
- Persist state changes (either as current state in a relational DB or as events in an event store).
- Publish domain events after successful writes so that read models and other downstream systems can react.

Typically modeled using **Domain-Driven Design (DDD)** aggregates — rich objects with encapsulated behavior — and is normalized for data integrity.

### 14. How are read and write models synchronized?

Common synchronization strategies:

1. **Domain Events + Message Bus** — After a write, publish an event (e.g., `OrderPlaced`) to a message broker (RabbitMQ, Kafka, Azure Service Bus). A projection worker subscribes and updates the read model. This is **asynchronous** → eventual consistency.
2. **Change Data Capture (CDC)** — Tools like Debezium monitor the write database's transaction log and stream changes to the read side. No application code needed.
3. **Synchronous dual-write** — Update both models in the same transaction. Simple but couples them and limits scalability. Generally discouraged.
4. **In-process event dispatch** — For single-process apps, use MediatR notifications to update read models within the same request pipeline (optionally in the same transaction).

### 15. What is eventual consistency, and how does it relate to CQRS?

Eventual consistency means that after a write, the read model will **eventually** reflect the change — but there's a window (milliseconds to seconds) where the read model is stale:

- When read and write models use separate stores, updates propagate asynchronously.
- The system guarantees that given no new writes, all reads will eventually return the latest data.
- This is a **trade-off** for scalability and performance — you give up immediate consistency for independent scaling.

**Mitigation strategies:** Return the command result to the caller directly (e.g., the new order ID), use optimistic UI updates, implement read-your-own-writes semantics, or use polling/SignalR to notify when the projection catches up.

### 16. How would you design a CQRS system in .NET?

A typical .NET CQRS design:

```
API Layer (ASP.NET Core Controllers)
  ├── Commands → ICommandHandler<TCommand> (via MediatR IRequestHandler)
  │     ├── Validate (FluentValidation pipeline behavior)
  │     ├── Execute domain logic (DDD aggregates)
  │     ├── Persist (EF Core / Dapper → Write DB)
  │     └── Publish domain event (MediatR INotification → projection handler)
  └── Queries → IQueryHandler<TQuery, TResult> (via MediatR IRequestHandler)
        └── Read directly from Read DB (Dapper for speed, or EF Core with no-tracking)
```

- **Write DB:** SQL Server with normalized schema, EF Core for ORM.
- **Read DB:** Same SQL Server with denormalized views, or a separate store (Redis, Elasticsearch).
- **Mediator:** MediatR dispatches commands and queries to their handlers.
- **Projects:** `Domain` (aggregates, events), `Application` (handlers, validators), `Infrastructure` (persistence, messaging), `API` (controllers).

### 17. What are common tools or libraries used for CQRS in .NET?

| Category | Library/Tool |
|---|---|
| **Mediator/Dispatcher** | MediatR, Wolverine, Brighter |
| **Event Sourcing** | EventStoreDB, Marten (PostgreSQL-based), Equinox |
| **Validation** | FluentValidation (as MediatR pipeline behavior) |
| **Messaging** | MassTransit, NServiceBus, RabbitMQ client, Azure Service Bus SDK, Kafka (Confluent .NET) |
| **Read Store** | Dapper (lightweight SQL reads), Redis, Elasticsearch (NEST client) |
| **ORM (Write side)** | Entity Framework Core |
| **Projections** | Marten async projections, custom `INotificationHandler<T>` in MediatR |
| **API** | ASP.NET Core Minimal APIs or Controllers |

### 18. How would you handle validation in a CQRS-based system?

Validation happens at two layers:

1. **Input/Command validation (Application layer):** Validate command shape, required fields, format (e.g., valid email, positive amounts). Use **FluentValidation** as a MediatR pipeline behavior — commands that fail validation are rejected before reaching the domain. Fast and stateless.

2. **Domain/Business rule validation (Domain layer):** Validate invariants that require state — e.g., "this customer's credit limit is not exceeded," "this product is still in stock." This happens inside the aggregate/command handler, which loads current state and checks rules. Failures throw domain exceptions.

> **Key rule:** Never skip domain validation just because input validation passed. Input validation catches malformed data; domain validation catches invalid business operations.

### 19. What happens if the read and write models get out of sync?

This is a real operational concern. Causes and mitigations:

- **Projection worker crashes:** Events pile up in the queue. Fix: use durable messaging (RabbitMQ with persistent queues, Kafka with consumer offsets). When the worker recovers, it processes the backlog.
- **Bug in projection logic:** The read model has incorrect data. Fix: the read model is **disposable** — drop and rebuild it by replaying all events from the event store (or re-projecting from the write DB).
- **Message loss:** Fix: use outbox pattern — write the event to an `Outbox` table in the same transaction as the state change, then a background process publishes it. Guarantees at-least-once delivery.
- **Monitoring:** Track projection lag (difference between latest write timestamp and latest projected timestamp). Alert when lag exceeds acceptable thresholds.

### 20. Can CQRS work in a monolithic application, or is it only for microservices?

CQRS works perfectly well in a **monolith**. In fact, starting with CQRS in a monolith is often the pragmatic choice:

- Use MediatR to dispatch commands and queries within the same process.
- Use the same database with separate read views (SQL views or denormalized tables).
- In-process event dispatch (MediatR `INotification`) updates read projections synchronously — no message bus needed.

CQRS is an **application architecture pattern**, not a deployment pattern. You can apply it to a single bounded context within a monolith. Microservices benefit from CQRS because they already have separate data stores, but it's not a prerequisite.

---

## Advanced Level

### 21. Explain how Event Sourcing complements CQRS.

Event Sourcing stores every state change as an immutable event (e.g., `OrderPlaced`, `ItemAdded`, `PaymentReceived`). This complements CQRS in several ways:

- **Natural event stream for projections:** Events are already being produced — projecting them into read models is a natural fit. Each projection subscribes to the event stream and builds its own denormalized view.
- **Multiple read models from one source:** The same event stream can feed a dashboard projection, a search index, a notification system, and an analytics pipeline — each with its own optimized schema.
- **Temporal queries:** Since all events are stored, you can rebuild the read model at any point in time ("What did the order look like at 3:00 PM?").
- **No data loss:** Traditional CRUD overwrites state; Event Sourcing preserves the full history, making debugging and auditing trivial.

The write side appends events; the read side projects them. CQRS provides the architecture; Event Sourcing provides the persistence model.

### 22. How would you handle transactions that span multiple aggregates in CQRS?

In DDD/CQRS, a single command should ideally modify **one aggregate** in one transaction. For cross-aggregate consistency:

1. **Saga / Process Manager (preferred):** A long-running process that listens to domain events and issues compensating commands. Example: `PlaceOrder` → `OrderPlaced` event → Saga issues `ReserveInventory` command → if inventory fails, Saga issues `CancelOrder` compensating command. Each step is its own transaction.
2. **Domain Events with eventual consistency:** Aggregate A publishes an event; Aggregate B reacts asynchronously. Consistency is eventual but each aggregate maintains its own invariants.
3. **Choreography:** Each aggregate publishes and subscribes to events without a central coordinator. Simpler but harder to debug complex flows.
4. **Last resort — same transaction:** If both aggregates are in the same bounded context and the consistency requirement is strict, you can modify both in one DB transaction. But this violates the "one aggregate per transaction" guideline and creates coupling.

### 23. How do you ensure consistency between command and query data stores?

- **Outbox Pattern:** Write the domain event to an `Outbox` table within the same database transaction as the state change. A background publisher reads the outbox and sends events to the message bus. Guarantees at-least-once delivery without distributed transactions.
- **Idempotent projections:** Since at-least-once delivery means duplicates, read-model updaters must be idempotent — processing the same event twice produces the same result (use event ID deduplication).
- **Ordering guarantees:** Use partitioned message topics (Kafka partition keys, RabbitMQ consistent hashing) to ensure events for the same aggregate arrive in order.
- **Poison message handling:** Dead-letter queues for events that fail projection. Alert, fix, and replay.
- **Monitoring:** Track the projection checkpoint (last processed event sequence number) against the write store's latest sequence. Alert on growing lag.

### 24. How do you deal with eventual consistency in a user-facing application?

Several UX patterns mitigate the lag:

- **Optimistic UI updates:** After submitting a command, immediately update the UI with the expected result (before the read model catches up). If the command fails, roll back the UI change.
- **Read-your-own-writes:** After a command, query the write store directly for that specific entity (bypassing the read model) to show the user their own change immediately.
- **Confirmation + polling:** Show a "Processing..." indicator and poll the read model (or use SignalR/WebSocket push) until the projection catches up.
- **Command result enrichment:** Return enough data in the command response (e.g., the created entity ID and key fields) so the UI can display a result without querying the read model at all.
- **Accept and display:** Design the UX to acknowledge the command ("Your order has been submitted") rather than immediately showing the fully projected view.

### 25. What strategies do you use for replaying events to rebuild the read model?

- **Full replay:** Drop the read model tables/collections, reset the projection checkpoint to 0, and replay all events from the event store. Simple but slow for large event stores.
- **Snapshotting:** Periodically save a snapshot of the read model state with the event sequence number. On rebuild, start from the latest snapshot instead of event 0.
- **Blue-green projections:** Build the new read model in parallel (into a new table/index) while the old one continues serving. Once the new projection catches up, switch traffic. Zero downtime.
- **Partitioned replay:** Replay events per aggregate type or partition in parallel to speed up the process.
- **Event versioning:** Use upcasters to transform old event formats to the current schema during replay, so projections only need to handle the latest event shape.

### 26. How do you manage versioning and schema changes in event-sourced systems?

Events are immutable — you can never modify stored events. Strategies for evolution:

- **Upcasting:** When reading old events, transform them to the current schema on-the-fly. E.g., `OrderPlacedV1` (without `currency` field) is upcasted to `OrderPlacedV2` with `currency = "USD"` as default.
- **Weak schema deserialization:** Use tolerant readers — ignore unknown fields, provide defaults for missing fields. JSON serialization handles this naturally.
- **Event versioning:** Include a version number in the event metadata. Projection handlers can branch on version if needed.
- **New event types:** Instead of modifying an old event, introduce a new event type. `CustomerRenamed` doesn't change; you add `CustomerNameCorrected` with the new field structure.
- **Copy-and-transform (migration):** For drastic changes, create a new event stream by reading the old one, transforming events, and writing to a new stream. Rarely needed but available as a last resort.

### 27. How would you implement auditing in a CQRS system?

- **With Event Sourcing:** Auditing is essentially free. Every event already records *what* changed, *when*, and you enrich it with *who* (user ID, IP, correlation ID in metadata). The event store **is** the audit log. Build an audit projection that indexes events by user, entity, or time range.
- **Without Event Sourcing:** Capture audit data in command handlers:
  - Log every command with timestamp, user, command type, and payload (mask sensitive data).
  - Use a MediatR pipeline behavior to intercept all commands and write to an `AuditLog` table.
  - Store before/after snapshots for change tracking (EF Core's `ChangeTracker` can provide this).
- **Compliance considerations:** Store audit logs in an append-only, tamper-evident store. Separate from the operational database for retention and access control.

### 28. What messaging patterns are commonly used between the write and read sides?

| Pattern | Technology | Use Case |
|---|---|---|
| **Pub/Sub (fan-out)** | Kafka topics, RabbitMQ fanout exchange, Azure Service Bus Topics | One event → multiple projections (dashboard, search index, notifications) |
| **Point-to-point queue** | RabbitMQ direct queue, Azure Service Bus Queue, SQS | One event → one specific consumer/projection |
| **Event streaming (log-based)** | Apache Kafka, Azure Event Hubs, AWS Kinesis | Ordered, replayable event stream; consumers track their own offset |
| **In-process dispatch** | MediatR `INotification`, .NET `Channel<T>` | Monolith — synchronous or background thread projection, no external broker |
| **Outbox + polling publisher** | Transactional outbox table + Debezium or custom poller | Guarantees at-least-once delivery without 2PC |

**Kafka** is ideal when you need ordered, replayable, high-throughput event streams. **RabbitMQ** is better for complex routing and lower-latency pub/sub with acknowledgment semantics.


### 29. What's the difference between CQRS and a simple read/write database replication setup?

| Aspect | DB Replication | CQRS |
|---|---|---|
| **Schema** | Read replica has the **same** schema as the primary | Read model has a **different**, query-optimized schema |
| **Model** | One domain model for both reads and writes | **Separate** domain models — write model enforces rules, read model is a flat projection |
| **Transformation** | Data is copied as-is | Data is **projected/transformed** — denormalized, pre-aggregated, reshaped for specific queries |
| **Technology** | Same database engine on both sides | Can use **different** technologies — SQL for writes, Elasticsearch/Redis for reads |
| **Application logic** | Same ORM/query code reads from replica | Separate query handlers, potentially different data access patterns |

DB replication scales reads but doesn't address the fundamental mismatch between write-optimized and read-optimized schemas. CQRS does.

### 30. What are the typical anti-patterns when implementing CQRS?

1. **CQRS everywhere:** Applying CQRS to every bounded context, even simple CRUD domains. Use it only where read/write asymmetry justifies the complexity.
2. **Querying the write model:** Bypassing the read model for queries, defeating the purpose of separation.
3. **Fat commands with return data:** Commands returning full entity projections — they should return minimal acknowledgements (ID, status) at most.
4. **Shared models:** Using the same DTOs for commands and queries, coupling the two sides.
5. **Ignoring eventual consistency in UX:** Not designing the UI for staleness — users see stale data and submit conflicting commands.
6. **Synchronous read model updates in command handlers:** Tight-coupling the write path to the read projection, eliminating scalability benefits.
7. **No idempotency:** Processing duplicate events in projections, leading to double-counted data.
8. **Over-engineering projections:** Building complex projections for queries that a simple SQL view would handle fine.

---

## Scenario-Based Questions

### 31. Suppose a user updates their profile information.

**How would you handle this in a CQRS setup?**

1. **Client** sends an `UpdateProfileCommand { UserId, NewName, NewEmail }` to the API.
2. **Command handler** validates input (FluentValidation: email format, name length).
3. **Domain layer** loads the `UserProfile` aggregate, calls `userProfile.UpdateName(newName)` and `userProfile.UpdateEmail(newEmail)`. The aggregate enforces invariants (e.g., email uniqueness check via a domain service).
4. **Persistence** saves the updated aggregate to the write DB. An `Outbox` record with `ProfileUpdated { UserId, NewName, NewEmail, Timestamp }` is written in the same transaction.
5. **API returns** `200 OK` with `{ userId }` (minimal response).
6. **Outbox publisher** sends the `ProfileUpdated` event to the message bus.
7. **Projection handler** subscribes, updates the read model — e.g., denormalized `UserProfileView` table with name, email, last updated.
8. **UI** either optimistically updates the displayed name/email, or polls/listens via SignalR until the read model reflects the change.

### 32. Dashboard showing aggregated metrics in near real-time.

**How would you ensure the data is updated in near real-time using CQRS?**

1. **Dedicated dashboard projection:** Create a read model specifically shaped for the dashboard — e.g., `DashboardMetrics { TotalOrders, Revenue, ActiveUsers, AvgOrderValue, LastUpdated }`.
2. **Incremental updates via events:** Each domain event (`OrderPlaced`, `OrderCancelled`, `UserRegistered`) triggers an incremental update to the dashboard projection — e.g., `TotalOrders++`, `Revenue += order.Amount`. No full recalculation needed.
3. **Near real-time delivery:** Use **Kafka** or **Azure Event Hubs** for low-latency event streaming (sub-second). The projection worker processes events as they arrive.
4. **Push to clients:** Use **SignalR** (ASP.NET Core) to push updated metrics to connected dashboard clients in real time. When the projection updates, it publishes a SignalR notification.
5. **Fallback polling:** For clients that don't support WebSockets, poll the read API every 5–10 seconds.
6. **Pre-aggregation:** Store counters and running totals in the read model rather than computing aggregates on-the-fly from raw data.
7. **Monitoring:** Track projection lag. If lag exceeds a threshold (e.g., 5 seconds), show a "Data may be delayed" indicator on the dashboard.

### 33. Designing read and write sides with SQL + NoSQL.

- **Write side — SQL (e.g., SQL Server / PostgreSQL):**
  - Normalized relational schema with foreign keys and constraints.
  - EF Core for ORM, transactions, and concurrency control (optimistic concurrency via row versioning).
  - Strong ACID guarantees for business rule enforcement.
  - Outbox table for reliable event publishing.

- **Read side — RavenDB (document store):**
  - Store denormalized documents shaped exactly for each query. E.g., an `OrderDetailDocument` containing the order, all line items, customer name, and shipping address — no joins.
  - RavenDB's built-in indexing (static indexes, Map-Reduce) handles aggregation queries efficiently.
  - Projection worker subscribes to domain events and upserts documents in RavenDB.

- **Synchronization:** SQL write → Outbox → Message bus (RabbitMQ/Kafka) → Projection worker → RavenDB upsert.

- **Why this mix?** SQL gives you strong consistency and transactional integrity for writes. RavenDB gives you fast, flexible document reads with automatic indexing — no need to design complex denormalized SQL views.

### 34. Debugging or monitoring the flow of a command.

**How would you debug/monitor from client → event store → read model?**

Use **distributed tracing** with a correlation ID:

1. **Correlation ID:** Generate a unique `CorrelationId` (GUID) at the API gateway/client. Attach it to every log entry, command, event, and projection operation.
2. **Structured logging:** Use Serilog/OpenTelemetry to log at each stage:
   - `Command received: {CorrelationId}, {CommandType}` (API layer)
   - `Command handled: {CorrelationId}, aggregate {AggregateId}` (handler)
   - `Event published: {CorrelationId}, {EventType}` (outbox/bus)
   - `Event projected: {CorrelationId}, read model updated` (projection worker)
3. **Distributed tracing tools:** OpenTelemetry + Jaeger/Zipkin/Application Insights. Each stage creates a span within the same trace, so you see the full end-to-end flow in one waterfall view.
4. **Health dashboards:** Monitor:
   - Command processing latency (P50, P95, P99)
   - Event bus queue depth and consumer lag
   - Projection lag (last projected event sequence vs. latest event)
   - Dead-letter queue size
5. **Debugging specific flows:** Query logs by `CorrelationId` to trace exactly what happened at each stage for a specific user action.

### 35. Testing a CQRS command handler vs a query handler.

**Testing a Command Handler:**

- **Unit tests** focus on behavior and side effects:
  - *Given:* set up the aggregate in a known state (or mock the repository).
  - *When:* execute the command.
  - *Then:* assert the aggregate's state changed correctly, the correct domain events were published, and invariant violations throw exceptions.
- **Mock dependencies:** Mock `IRepository`, `IEventBus`, domain services.
- **Test edge cases:** Invalid input, business rule violations, concurrency conflicts, idempotency (duplicate command).
- **Integration tests:** Use a real database (TestContainers) to verify persistence and outbox behavior end-to-end.

**Testing a Query Handler:**

- **Unit tests** are simpler — no side effects to verify:
  - *Given:* seed the read model/database with known data.
  - *When:* execute the query.
  - *Then:* assert the returned DTO matches the expected shape and values.
- **Mock or in-memory DB:** Use an in-memory database or mock the read repository.
- **Test edge cases:** Empty results, pagination boundaries, filtering/sorting, authorization (user can only see their own data).

> **Key difference:** Command handler tests verify *behavior and state changes*. Query handler tests verify *data retrieval and shaping*. Command tests are typically more complex because they involve domain logic, validation, and event publication.

