# Kafka with .NET Guide

> For a senior .NET developer. Assumes distributed systems experience. Draws comparisons to RabbitMQ, Azure Service Bus, and SQL where relevant.

<a id="top"></a>
## Table of Contents

- [Kafka in Simple Language](#kafka-in-simple-language)
- [1. Kafka Architecture — The Mental Model](#1-kafka-architecture--the-mental-model)
    - [Core Concepts](#core-concepts)
    - [Key Insight — Why Kafka Isn't RabbitMQ](#key-insight--why-kafka-isnt-rabbitmq)
- [2. Kafka Guarantees — Interview Favorite](#2-kafka-guarantees--interview-favorite)
    - [Producer Delivery Semantics](#producer-delivery-semantics)
    - [Consumer Delivery Semantics](#consumer-delivery-semantics)
- [3. .NET Integration — Confluent.Kafka](#3-net-integration--confluentkafka)
    - [3.1 Producer — Basic](#31-producer--basic)
    - [3.2 Consumer — With Manual Commit](#32-consumer--with-manual-commit)
    - [3.3 DI Registration — Production Pattern](#33-di-registration--production-pattern)
- [4. Partitioning — The Scaling Mechanism](#4-partitioning--the-scaling-mechanism)
    - [The Simple Explanation](#the-simple-explanation)
    - [How Partitioning Works](#how-partitioning-works)
    - [Consumer Group Scaling](#consumer-group-scaling)
    - [Custom Partitioner](#custom-partitioner)
- [5. Error Handling & Dead Letter Topics](#5-error-handling--dead-letter-topics)
    - [Retry + DLT Pattern](#retry--dlt-pattern)
- [6. Exactly-Once Semantics (Transactional Producer)](#6-exactly-once-semantics-transactional-producer)
- [7. Schema Registry & Serialization](#7-schema-registry--serialization)
- [8. Kafka vs Azure Service Bus — You'll Be Asked This](#8-kafka-vs-azure-service-bus--youll-be-asked-this)
    - [8.1 AMQP Transactions](#81-amqp-transactions)
- [9. Production Patterns](#9-production-patterns)
    - [9.1 Outbox Pattern (Kafka + SQL)](#91-outbox-pattern-kafka--sql)
    - [9.2 Idempotent Consumer (at-least-once + dedup)](#92-idempotent-consumer-at-least-once--dedup)
    - [9.3 Consumer Lag Monitoring](#93-consumer-lag-monitoring)
- [10. Rebalancing — The Gotcha](#10-rebalancing--the-gotcha)
    - [Rebalance Strategies](#rebalance-strategies)
- [11. Kafka Streams Concepts (Not .NET Native)](#11-kafka-streams-concepts-not-net-native)
- [12. Docker Compose — Local Dev](#12-docker-compose--local-dev)
- [13. Interview Questions & Answers](#13-interview-questions--answers)
    - [Architecture](#architecture)
    - [.NET Specific](#net-specific)
    - [Design / Scenario](#design--scenario)
- [14. Common Pitfalls](#14-common-pitfalls)
- [15. Quick Reference — Config Cheat Sheet](#15-quick-reference--config-cheat-sheet)
- [16. Testing Kafka in .NET](#16-testing-kafka-in-net)
- [Cheat Sheet — One-Page Summary](#cheat-sheet--one-page-summary)

---

## Kafka in Simple Language

## What Kafka Is Really For

Kafka is used when many applications need to **share a large, continuous flow of events reliably**.

An event is a fact that happened, such as:

- An order was created
- A payment was completed
- A sensor sent a reading
- A user logged in
- A file was uploaded

Instead of one application calling every other application directly, it publishes the event to Kafka. Other applications can read that event independently.

```text
Order Service → Kafka → Payment Service
                    ├→ Inventory Service
                    ├→ Email Service
                    └→ Analytics Service
```

The Order Service does not need to know whether the other services are running, how fast they are, or how many of them exist.

## Why Do We Need Kafka?

### 1. To decouple applications

**Q: What does decoupling mean in this example?**
> The producer does not need a direct connection to every consumer. Each service can change, restart, or scale independently as long as it understands the event contract.

Without Kafka:

```text
Order Service → Payment Service
              → Inventory Service
              → Email Service
              → Analytics Service
```

The Order Service must call all four services. If one is down, slow, or unavailable, the order process may be affected.

With Kafka:

```text
Order Service → Kafka
```

The Order Service publishes `OrderCreated` and finishes. Each interested service reads the event when it is ready.

### 2. To handle high traffic

**Q: Does Kafka make a slow downstream service faster?**
> No. Kafka absorbs and distributes traffic, but the downstream service still needs enough consumers and capacity. Lag will grow if it processes slower than events arrive.

Kafka splits data into partitions. Multiple brokers and consumers can process different partitions at the same time.

This allows Kafka to handle a large number of events without forcing one application or one server to process everything sequentially.

### 3. To keep events for later

**Q: How does replay help in practice?**
> A new or repaired consumer can read historical events and rebuild its state instead of requiring the source system to resend everything.

A normal queue usually removes a message after successful processing. Kafka keeps messages for a configured period, such as seven days, or sometimes much longer.

That means a consumer can:

- Restart from where it stopped
- Replay old events
- Rebuild a database or cache
- Add a new consumer that reads historical data
- Investigate what happened

This is one of Kafka's biggest differences from a traditional queue.

### 4. To allow multiple independent consumers

**Q: Why does each service usually use a different consumer group?**
> Separate groups give each service its own progress. If all services shared one group, they would compete for messages and some services would miss events intended for them.

Suppose Kafka contains an `order-events` topic. These separate applications can all read the same events:

- Inventory service
- Billing service
- Notification service
- Reporting service

Each application uses its own consumer group and maintains its own progress. One consumer does not remove the event for the others.

## How Kafka Actually Works

The easiest way to understand Kafka is to imagine a warehouse that stores orders in labeled aisles. Kafka does not directly deliver each message to a named consumer. The producer puts the message into the correct aisle, Kafka assigns aisles to workers, and each worker comes to collect messages from its assigned aisle.

### Real-World Terminology

| Kafka term | Online-shop equivalent | Meaning |
|---|---|---|
| **Producer** | Checkout system | Creates and publishes an order event |
| **Kafka cluster** | All warehouse buildings | The complete Kafka installation |
| **Broker** | One warehouse building | One Kafka server that stores data |
| **Topic** | An order category | A named stream such as `orders` or `payments` |
| **Partition** | An aisle or shelf in that category | One ordered log inside a topic |
| **Partition key** | A sorting label, such as customer ID | Helps Kafka choose the partition |
| **Consumer group** | A team of order-processing workers | A logical application sharing work |
| **Consumer** | One worker in that team | Reads and processes assigned partitions |
| **Offset** | A worker's bookmark | Position of a record within a partition |

### 1. The Producer Creates an Event

The checkout system creates an order:

```json
{
  "orderId": "ORD-1001",
  "customerId": "C-55",
  "product": "Laptop"
}
```

It sends the record to the `orders` topic with a key:

```text
Topic: orders
Key: customer-55
Value: order ORD-1001
```

The producer chooses the topic and key. It does not choose the actual consumer.

### 2. The Topic Is Split into Partitions

A topic is a named stream of related messages. Kafka divides that stream into partitions so it can scale:

```text
orders
├── Partition 0
├── Partition 1
└── Partition 2
```

Each partition is an ordered, append-only log:

```text
Partition 0: Order A, Order D, Order G
Partition 1: Order B, Order E, Order H
Partition 2: Order C, Order F, Order I
```

Kafka guarantees order inside one partition, not across the entire topic.

### 3. The Partition Key Chooses the Partition

Kafka hashes the key and uses the result to select a partition:

```text
hash("customer-55") % 3 = 1
```

So the record is written to Partition 1:

```text
customer-55 order 1 -> Partition 1
customer-55 order 2 -> Partition 1
customer-55 order 3 -> Partition 1
```

Using `customerId` as the key keeps that customer's events in the same partition and preserves their order. Using `orderId` would preserve order for each individual order instead. The key chooses a partition; it does not identify a consumer.

### 4. Brokers Store the Partitions

A broker is one Kafka server. A cluster may contain several brokers:

```text
Broker 1 -> orders Partition 0
Broker 2 -> orders Partition 1
Broker 3 -> orders Partition 2
```

In production, each partition normally has replicas on other brokers:

```text
Partition 1:
  Leader:  Broker 2
  Replica: Broker 3
  Replica: Broker 1
```

The producer does not send the record to every broker. It sends it to the leader of the selected partition. The leader replicates the data to the other brokers for fault tolerance. If the leader fails, one replica can become the new leader.

### 5. A Consumer Group Is a Team of Workers

Suppose the order-processing application has one consumer group with three consumers:

```text
Consumer group: order-processors

Worker A -> Partition 0
Worker B -> Partition 1
Worker C -> Partition 2
```

Kafka assigns each partition to one consumer in that group at a time. Therefore, the message in Partition 1 is eventually read by Worker B, but the producer never selected Worker B. Kafka assigned Partition 1 to Worker B, and Worker B reads from it.

The complete routing path is:

```text
Checkout system
    |
    | produces order with key "customer-55"
    v
Topic: orders
    |
    | hash(key) selects Partition 1
    v
Partition 1, led by Broker 2
    |
    | assigned by Kafka to Worker B
    v
Worker B polls and processes the order
```

### 6. Consumers Poll Kafka

Kafka consumers normally pull messages rather than receiving a direct push. Worker B repeatedly asks:

```text
"Give me the next messages from Partition 1."
```

In .NET, this is represented by:

```csharp
var result = consumer.Consume(cancellationToken);
```

Polling lets the consumer control its pace and request batches that match its capacity.

### 7. Offsets Are Consumer Bookmarks

Every record has a position within its partition:

```text
Partition 1:
Offset 0 -> Order A
Offset 1 -> Order B
Offset 2 -> Order C
Offset 3 -> Order D
```

If Worker B has successfully processed through offset 2, its next record is offset 3. Kafka retains messages according to the topic's retention settings, so a consumer can restart from an earlier offset and replay them.

### 8. Consumers Commit Progress

After successful processing, Worker B commits its position:

```text
"I successfully processed through offset 2."
```

If it crashes before committing, Kafka can deliver the record again after restart. This avoids message loss but means consumers should usually be idempotent.

### 9. Multiple Groups Can Read the Same Event

Different applications use different consumer groups:

```text
Topic: orders
├── order-processors group -> fulfills orders
├── billing-service group   -> charges customers
└── analytics-service group -> builds reports
```

Each group gets its own logical copy of the stream and maintains independent offsets. Inside one group, only one consumer handles a particular partition at a time. This is similar to sending one newspaper to three departments, while only one employee in each department handles that department's copy.

### 10. What Happens When a Consumer Fails?

If Worker B fails, Kafka rebalances the group and assigns Partition 1 to another available worker:

```text
Before: Partition 1 -> Worker B
After:  Partition 1 -> Worker C
```

Worker C continues from the group's committed offset. This is called a **rebalance**.

With three partitions, at most three consumers in one group can actively process partitions:

```text
3 partitions + 3 consumers = all consumers useful
3 partitions + 5 consumers = 2 consumers idle
```

The maximum parallelism of one consumer group is limited by the number of partitions.

## What Is a Consumer Group?

A consumer group is a team of consumers sharing work. It is the mechanism that turns partitions into parallel workers: Kafka assigns each partition to one member of the group, tracks the group's offsets, and reassigns partitions when members join or leave.

## What Happens When Processing Fails?

Kafka does not automatically assume that a message was successfully handled just because it delivered it.

The usual pattern is:

```text
Read message
    ↓
Process message
    ↓
Success? ── Yes → Commit offset
    ↓ No
Retry or send to a Dead Letter Topic
```

If the application crashes before committing, Kafka delivers the message again after restart. This provides **at-least-once delivery**, which means duplicates are possible.

Therefore, consumers should usually be **idempotent**: processing the same event twice should not create an incorrect result.

## A Real Example

An online shop receives an order:

1. The Order Service creates the order.
2. It publishes `OrderCreated` to Kafka.
3. The Inventory Service reads it and reserves stock.
4. The Payment Service reads it and charges the customer.
5. The Email Service reads it and sends confirmation.
6. The Analytics Service reads it and updates reports.

If the Email Service is temporarily down:

- The order can still be created.
- Kafka keeps the event.
- The Email Service reads it when it comes back.
- It can continue from its saved offset.

## What Kafka Is Not

Kafka is not automatically:

- A database for business queries
- A replacement for every HTTP API
- A guarantee that external database work is exactly once
- A system that preserves ordering across all partitions
- A built-in retry and dead-letter solution in the same way Azure Service Bus provides one

Kafka is best understood as a **durable, scalable event log**.

## One-Sentence Interview Answer

> Kafka is a distributed event log that lets applications publish, store, process, replay, and independently consume large volumes of events, while remaining decoupled from one another.

---

## 1. Kafka Architecture — The Mental Model

Forget "message queue." Kafka is a **distributed, immutable, append-only commit log** with consumer-driven reads.

```
Producers ──▶ [Broker Cluster] ──▶ Consumers
                   │
          ┌────────┼────────┐
        Topic A  Topic B  Topic C
        P0 P1 P2  P0 P1   P0 P1 P2 P3
```

### Core Concepts

| Concept | What It Is | SQL Analogy |
|---|---|---|
| **Broker** | A Kafka server node. Cluster = multiple brokers | A SQL Server node in AG |
| **Topic** | Named log stream (e.g., `order-events`) | A table |
| **Partition** | Ordered, immutable sequence within a topic | A table partition by hash |
| **Offset** | Sequential ID per message within a partition | An identity column |
| **Consumer Group** | Set of consumers sharing work on a topic | Competing consumers pattern |
| **Replication Factor** | Copies of each partition across brokers | AG replicas |

**Q: What is the difference between a topic and a partition?**
> A topic is the named stream that applications publish to. A partition is one ordered slice of that topic. Kafka distributes partitions across brokers so the topic can scale.

**Q: What does an offset identify?**
> An offset identifies a message's position inside one partition. It is not globally unique across the topic because every partition has its own offset sequence.

### Brokers, Partitions, and Replication — The Complete Picture

**Does a producer send to all brokers?**

No. A producer sends to a **topic**, and Kafka automatically determines which broker (leader) holds each partition. Here's how it differs from traditional message brokers:

**Traditional Message Broker (RabbitMQ-style):**
```
One Smart Broker (centralized control)
    ↓
    ├→ Consumer A (broker pushes to them)
    ├→ Consumer B (broker pushes to them)
    └→ Consumer C (broker pushes to them)

The broker:
  - Decides which consumer gets which message
  - Tracks what each consumer has read
  - Manages delivery control
  - Is a single point of failure
```

**Kafka Cluster (distributed dumb brokers):**
```
Broker 1 ─┐
Broker 2 ─┼→ All hold copies of partitions
Broker 3 ─┘

Consumers pull independently:
  - Each consumer controls its own pace
  - Brokers don't track consumption
  - Each broker is stateless about consumer progress
  - No single point of failure — replicas provide resilience
```

**Complete Kafka System Diagram:**

```
                           KAFKA CLUSTER (3 Brokers)
                         
PRODUCER                    ┌─────────────────────────────────────┐
  │                         │  Broker 1      Broker 2     Broker 3│
  │ Publish to topic        │                                     │
  │ with Key="order-123"    │  order-events (3 partitions, RF=3)  │
  │                         │                                     │
  └────────────────────────►│  Partition 0:                       │
                            │    ├─ Leader (Broker 1)             │
       hash("order-123")%3  │    ├─ Replica (Broker 2)            │
       = Partition 0        │    └─ Replica (Broker 3)            │
                            │                                     │
                            │  Partition 1:                       │
                            │    ├─ Leader (Broker 2)             │
                            │    ├─ Replica (Broker 3)            │
                            │    └─ Replica (Broker 1)            │
                            │                                     │
                            │  Partition 2:                       │
                            │    ├─ Leader (Broker 3)             │
                            │    ├─ Replica (Broker 1)            │
                            │    └─ Replica (Broker 2)            │
                            │                                     │
                            └─────────────────────────────────────┘
                                         ▲
                                         │ All consumers read
                                         │ from their assigned
                                         │ partitions
                                         │
                    ┌────────────────────┼────────────────────┐
                    │                    │                    │
                 CONSUMER GROUP: "order-processor"
                    │                    │                    │
                Consumer A          Consumer B          Consumer C
              (reads P0)          (reads P1)          (reads P2)
              Offset: 42          Offset: 38          Offset: 51
```

**Key insights:**

1. **Producer → Topic (not broker)**
   - Producer sends: Topic + Key + Value
   - Kafka calculates: `hash(key) % partition_count` to determine partition
   - Kafka finds the partition's leader and writes there
   - Replication is automatic: leader replicates to other brokers

2. **Partitioning + Replication working together**
   - Each message goes to ONE partition
   - That partition has multiple replicas across brokers
   - When a broker fails, a replica becomes the new leader
   - Data is not lost because replicas exist on other brokers

3. **Multiple consumer groups reading same topic independently**
   - Consumer Group "order-processor": reads Partitions 0, 1, 2 at offsets 42, 38, 51
   - Consumer Group "order-analytics": reads same partitions at offsets 5, 8, 10
   - Consumer Group "order-audit": reads all partitions independently
   - Each group maintains its own offset — no interference

4. **Complete message flow:**
   - Producer calls: `ProduceAsync("order-events", key="cust-123", value={...})`
   - Kafka calculates: `hash("cust-123") % 3 = partition 1`
   - Message is written to Partition 1's leader (Broker 2)
   - Broker 2 replicates to Brokers 3 & 1, waits for acks
   - Consumer B (reading Partition 1) polls and gets offset 100
   - Consumers A and C (reading other partitions) never see this message
   - Consumer B processes and commits offset 100

### Key Insight — Why Kafka Isn't RabbitMQ

| Behavior | RabbitMQ | Kafka |
|---|---|---|
| Message delivery | Push to consumer, deleted after ack | Consumer pulls, message stays |
| Replay | Not possible after ack | Seek to any offset anytime |
| Ordering | Per-queue only | Per-partition guaranteed |
| Scaling consumers | Add consumers to queue | Add partitions (consumer ≤ partitions) |
| Storage | Memory + optional disk | Disk-first, retention-based |
| Throughput | ~50K msg/s | ~1M+ msg/s |

**RabbitMQ is a smart broker / dumb consumer. Kafka is a dumb broker / smart consumer.**

**Q: When would you choose Kafka instead of RabbitMQ?**
> Choose Kafka when replay, high throughput, multiple independent consumers, or event history matters. Choose RabbitMQ when you mainly need task distribution, routing, acknowledgements, and broker-managed delivery.

**Q: Does Kafka replace an API?**
> No. APIs are useful when a caller needs an immediate response. Kafka is useful when applications can communicate asynchronously through events.

---

<p align="right"><a href="#top">Back to top</a></p>

---

## 2. Kafka Guarantees — Interview Favorite

### Producer Delivery Semantics

**Q: Is `acks=all` the same as exactly-once delivery?**
> No. `acks=all` improves durability by waiting for in-sync replicas. It does not prevent a consumer from processing a record twice or make external database work atomic.

| `acks` Setting | Guarantee | Latency | Use When |
|---|---|---|---|
| `0` | Fire-and-forget | Lowest | Metrics, logs |
| `1` | Leader acknowledged | Medium | Most use cases |
| `all` (-1) | All ISR replicas ack'd | Highest | Financial, critical data |

### Consumer Delivery Semantics

| Semantic | How | Trade-off |
|---|---|---|
| **At-most-once** | Commit offset before processing | Can lose messages |
| **At-least-once** | Commit offset after processing | Can get duplicates (most common) |
| **Exactly-once** | Transactional API + idempotent producer | Higher latency, complexity |

> **Interview tip:** "We use at-least-once with idempotent consumers" is the pragmatic senior answer. Exactly-once has overhead and only works within Kafka-to-Kafka pipelines.

**Q: What does `acks=all` actually guarantee?**
> It means the leader waits for all currently in-sync replicas to acknowledge the record. It is strongest when combined with replication factor 3 and `min.insync.replicas=2`; it does not protect data if the producer never receives a successful response.

**Q: Why is at-least-once usually preferred?**
> It avoids losing a message when a consumer fails before committing. The trade-off is possible duplicates, so the business operation must be idempotent.

---

<p align="right"><a href="#top">Back to top</a></p>

---

## 3. .NET Integration — Confluent.Kafka

The official .NET client. NuGet: `Confluent.Kafka`

```bash
dotnet add package Confluent.Kafka
dotnet add package Confluent.SchemaRegistry.Serdes.Json  # optional: for schema registry
```

### 3.1 Producer — Basic

**Q: Why should a Kafka producer usually be a singleton?**
> A producer maintains connections, buffers, and background work. Reusing one producer improves batching and avoids the cost of creating a new client for every request.

**Q: What is the difference between `Produce` and `ProduceAsync`?**
> `Produce` queues the record and returns quickly, while `ProduceAsync` lets you await delivery confirmation. Use the asynchronous form when the application needs to know whether Kafka accepted the record.

```csharp
using Confluent.Kafka;

public class OrderEventProducer : IDisposable
{
    private readonly IProducer<string, string> _producer;

    public OrderEventProducer(string bootstrapServers)
    {
        var config = new ProducerConfig
        {
            BootstrapServers = bootstrapServers,
            Acks = Acks.All,                    // durability
            EnableIdempotence = true,           // prevents duplicates on retry
            MaxInFlight = 5,                    // max unack'd requests (idempotent allows up to 5)
            MessageSendMaxRetries = 3,
            RetryBackoffMs = 100,
            LingerMs = 5,                       // batch for 5ms before sending (throughput vs latency)
            BatchSize = 16384,                  // 16KB batch
            CompressionType = CompressionType.Snappy
        };

        _producer = new ProducerBuilder<string, string>(config)
            .SetErrorHandler((_, e) => Console.Error.WriteLine($"Kafka producer error: {e.Reason}"))
            .Build();
    }

    // Fire-and-forget (fastest, no guarantee)
    public void ProduceFireAndForget(string topic, string key, string value)
    {
        _producer.Produce(topic, new Message<string, string> { Key = key, Value = value });
    }

    // Async with delivery confirmation (recommended)
    public async Task<DeliveryResult<string, string>> ProduceAsync(
        string topic, string key, string value)
    {
        try
        {
            var result = await _producer.ProduceAsync(topic,
                new Message<string, string>
                {
                    Key = key,
                    Value = value,
                    Headers = new Headers
                    {
                        { "correlation-id", Encoding.UTF8.GetBytes(Guid.NewGuid().ToString()) },
                        { "source", Encoding.UTF8.GetBytes("order-service") }
                    }
                });

            // result.Partition, result.Offset — where it landed
            return result;
        }
        catch (ProduceException<string, string> ex)
        {
            // ex.DeliveryResult.Status — PossiblyPersisted | NotPersisted
            throw;
        }
    }

    public void Dispose() => _producer?.Dispose();
}
```

### 3.2 Consumer — With Manual Commit

**Q: Why should the consumer commit after processing instead of before it?**
> Committing after processing prevents the consumer from losing a message if it crashes. The message may be processed again, but it will not be silently skipped.

**Q: Why should a Kafka consumer not be shared across multiple threads?**
> The standard Confluent consumer is not designed for concurrent use. Keep polling and consumer operations on one thread, and use separate consumers when you need parallel consumption.

#### Simple Explanation

A Kafka consumer is an application that **reads messages from topics**. Think of it like a subscriber to a newspaper.

**Key concepts:**

1. **Pull model** — Consumers actively pull messages from brokers (unlike RabbitMQ which pushes). You control the pace.

2. **Consumer group** — Multiple consumers can work together as a "group" to split the work. Each partition is read by only one consumer in the group, so they share the load.

3. **Offset** — A bookmark that tracks which message you've read. It says "I've read up to message #42 on partition 0." This lets you resume from where you left off if the consumer restarts.

4. **Commit** — Telling Kafka "I've successfully processed messages up to this offset, save my progress." Two strategies:
   - **Auto-commit:** Kafka saves your progress automatically (faster but less control)
   - **Manual commit:** You decide when to save progress (safer, gives you control)

5. **Rebalancing** — When consumers join/leave a group, Kafka redistributes partitions among them. During rebalance, reading pauses.

**How it works step-by-step:**
1. Consumer joins a group and subscribes to a topic
2. Kafka assigns partitions to the consumer
3. Consumer polls (asks) Kafka: "Give me the next batch of messages"
4. Consumer processes the messages
5. Consumer commits the offset: "I'm done with these messages, remember I got to offset 42"
6. Consumer repeats from step 3

**Why manual commit is often safer:**
Auto-commit might save your progress before you finish processing, so you could lose messages if you crash. Manual commit waits until you're done processing, then saves. You're saying "I've definitely processed this, it's safe to save."

---

```csharp
using Confluent.Kafka;

public class OrderEventConsumer : BackgroundService
{
    private readonly IConsumer<string, string> _consumer;
    private readonly ILogger<OrderEventConsumer> _logger;

    public OrderEventConsumer(string bootstrapServers, string groupId, ILogger<OrderEventConsumer> logger)
    {
        _logger = logger;

        var config = new ConsumerConfig
        {
            BootstrapServers = bootstrapServers,
            GroupId = groupId,
            AutoOffsetReset = AutoOffsetReset.Earliest,   // start from beginning if no committed offset
            EnableAutoCommit = false,                      // WE control when offsets commit
            EnableAutoOffsetStore = false,                 // WE control what offset is stored
            MaxPollIntervalMs = 300000,                    // 5 min max between polls
            SessionTimeoutMs = 45000,                      // heartbeat timeout
            IsolationLevel = IsolationLevel.ReadCommitted  // only read committed transactional messages
        };

        _consumer = new ConsumerBuilder<string, string>(config)
            .SetErrorHandler((_, e) => _logger.LogError("Kafka error: {Reason}", e.Reason))
            .SetPartitionsAssignedHandler((c, partitions) =>
            {
                _logger.LogInformation("Assigned: {Partitions}",
                    string.Join(", ", partitions.Select(p => $"{p.Topic}[{p.Partition.Value}]")));
            })
            .SetPartitionsRevokedHandler((c, partitions) =>
            {
                // Commit what we've processed before losing partitions
                _logger.LogInformation("Revoking: {Partitions}",
                    string.Join(", ", partitions.Select(p => $"{p.Topic}[{p.Partition.Value}]")));
            })
            .Build();
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _consumer.Subscribe("order-events");

        await Task.Yield(); // release the startup thread

        while (!stoppingToken.IsCancellationRequested)
        {
            try
            {
                var result = _consumer.Consume(stoppingToken); // blocking call

                await ProcessMessage(result);

                // Store offset for this partition, then commit
                _consumer.StoreOffset(result);
                _consumer.Commit(result);
            }
            catch (ConsumeException ex)
            {
                _logger.LogError(ex, "Consume error");
            }
            catch (KafkaException ex)
            {
                _logger.LogError(ex, "Kafka error");
            }
        }

        _consumer.Close(); // triggers final commit + graceful leave
    }

    private Task ProcessMessage(ConsumeResult<string, string> result)
    {
        _logger.LogInformation(
            "Topic={Topic} Partition={Partition} Offset={Offset} Key={Key}",
            result.Topic, result.Partition.Value, result.Offset.Value, result.Message.Key);

        // Your business logic here
        return Task.CompletedTask;
    }
}
```

### 3.3 DI Registration — Production Pattern

**Q: Why is the producer registered as a singleton but the consumer as a hosted service?**
> The producer is a reusable, thread-safe client shared by requests. The consumer owns a long-running polling loop, so a hosted service gives it a controlled application lifetime.

**Q: How should Kafka configuration be supplied in a real application?**
> Keep broker addresses, credentials, and topic names in configuration or a secret store. Validate required settings at startup instead of discovering a missing broker address during message processing.

```csharp
// Program.cs or Startup.cs
builder.Services.AddSingleton<IProducer<string, string>>(sp =>
{
    var config = new ProducerConfig
    {
        BootstrapServers = builder.Configuration["Kafka:BootstrapServers"],
        Acks = Acks.All,
        EnableIdempotence = true
    };
    return new ProducerBuilder<string, string>(config).Build();
});

// Register as hosted service
builder.Services.AddHostedService<OrderEventConsumer>();
```

---

<p align="right"><a href="#top">Back to top</a></p>

---

## 4. Partitioning — The Scaling Mechanism

### The Simple Explanation

**Q: Can Kafka guarantee the order of every message in a topic?**
> No. Kafka guarantees order only inside an individual partition. If the whole topic needs one global order, it effectively needs one partition and loses most parallelism.

**Q: How do you choose a partition key?**
> Choose an entity whose events must stay ordered, such as `CustomerId` or `OrderId`. The key should also distribute traffic evenly; a hot key can overload one partition.

Partitioning is splitting one big Kafka topic into smaller independent pieces.

**Analogy:** Think of a topic like a highway:
- Without partitioning: 1 lane, all cars in one line → bottleneck
- With partitioning: many lanes, traffic flows in parallel → higher throughput

**What it means:**
- A topic is broken into multiple partitions
- Each partition is an ordered, immutable log
- Messages are distributed across partitions using a key
- Messages with the same key go to the same partition, preserving order for that key

**Why we need it:**

1. **Higher throughput** — Kafka writes/reads from multiple partitions simultaneously, so overall data flow increases dramatically

2. **Better scaling** — Multiple consumers can process data in parallel (each partition is assigned to only one consumer in a consumer group at a time)

3. **Avoid bottlenecks** — Without partitions, one broker and one consumer path become the limit; partitioning distributes load

4. **Preserve order where it matters** — Use a consistent key (e.g., customer ID) to ensure related events stay ordered in the same partition, while still scaling overall

5. **Reliability** — Partitions are replicated across brokers, so if one broker fails, replicas keep data safe and available

**Bottom line:** Partitioning is how Kafka achieves high throughput, horizontal scalability, and practical reliability for real-time systems.

---

### How Partitioning Works

**Q: What happens if the number of partitions is too small?**
> Consumer parallelism and write throughput are capped. Adding consumers cannot help once every partition already has a consumer.

**Q: Can partitions be reduced later?**
> Kafka supports increasing partitions, but reducing them is not a normal in-place operation. Choose the initial count carefully or migrate to a new topic if the design must change.

```
Topic: order-events (3 partitions)

Producer sends Key="customer-123" ──▶ hash("customer-123") % 3 = Partition 1
Producer sends Key="customer-456" ──▶ hash("customer-456") % 3 = Partition 0
Producer sends Key=null            ──▶ Round-robin across partitions
```

### Consumer Group Scaling

**Q: Can two consumers in the same group process the same partition at the same time?**
> No. Kafka assigns a partition to only one consumer in a group. Different consumer groups can independently read the same partition.

**Q: Why might adding consumers fail to improve throughput?**
> There may be fewer partitions than consumers, or the bottleneck may be the database, API, or downstream system rather than Kafka itself.

```
Consumer Group: "order-processor"

3 partitions, 2 consumers:
  Consumer A ──▶ P0, P1
  Consumer B ──▶ P2

3 partitions, 3 consumers:
  Consumer A ──▶ P0
  Consumer B ──▶ P1
  Consumer C ──▶ P2

3 partitions, 4 consumers:
  Consumer A ──▶ P0
  Consumer B ──▶ P1
  Consumer C ──▶ P2
  Consumer D ──▶ IDLE (no partition to read)  ← wasted
```

**Rule: consumers in a group ≤ partitions.** More consumers than partitions = idle consumers.

### Custom Partitioner

**Q: When is a custom partitioner justified?**
> Use one only when the default key hashing cannot meet a clear routing requirement. It adds operational and testing complexity, so a well-chosen message key is usually preferable.

**Q: What risk exists in the sample custom partitioner?**
> The partition count must be positive, and every returned partition must be valid. Routing rules also need to remain stable, or related events may be split across partitions.

```csharp
public class RegionPartitioner : IPartitioner
{
    public Partition Partition(string topic, ReadOnlySpan<byte> keyData,
        bool keyIsNull, int partitionCount)
    {
        if (keyIsNull) return Partition.Any;

        var key = Encoding.UTF8.GetString(keyData);

        // Route by region prefix: "US-order-123" → partition 0, "EU-order-456" → partition 1
        if (key.StartsWith("US")) return new Partition(0);
        if (key.StartsWith("EU")) return new Partition(1 % partitionCount);
        return new Partition(Math.Abs(key.GetHashCode()) % partitionCount);
    }
}
```

---

<p align="right"><a href="#top">Back to top</a></p>

---

## 5. Error Handling & Dead Letter Topics

### Simple Explanation

**Q: Should every processing failure be retried?**
> No. Retry temporary failures such as timeouts, but send permanent failures such as invalid data to a DLT sooner. Retrying a bad message wastes resources and delays healthy messages.

**Q: What information should a DLT message contain?**
> Keep the original key and value, plus the original topic, partition, offset, failure reason, exception type, retry count, and timestamp. This makes investigation and replay possible.

**The Problem:**
When processing messages from Kafka, sometimes things go wrong:
- The message format is corrupted
- An external API call fails
- A database is down
- Business logic throws an exception

If a message fails permanently, you can't just ignore it — it needs to go somewhere for investigation.

**Dead Letter Topic (DLT):**
A DLT is simply another Kafka topic where failed messages are sent after they've been retried too many times. Think of it like a "failed mail" bin at a post office.

**The Pattern:**
1. Try to process the message
2. If it fails, increment a retry counter (in message headers)
3. If retries < max (e.g., 3), re-publish the message back to the original topic to try again
4. If retries are exhausted, send it to the DLT with error details
5. Later, a human or automated system can inspect the DLT and decide what to do

**Why this is better than crashing:**
- Without DLT: One bad message blocks the entire consumer, everything stops
- With DLT: Bad messages are isolated, good messages keep flowing, and you have a queue of failed items to fix later

**Key benefits:**
- **Resilience** — One poison message doesn't crash your system
- **Auditability** — All failed messages are tracked with error reasons and timestamps
- **Manual recovery** — You can fix the underlying issue and reprocess from the DLT
- **Monitoring** — Alert when DLT grows (indicates systemic issues)

**Simple flow:**
```
Message → Try to Process
            ↓
          Success? → Commit & continue
          ↓ (No)
        Retry < Max? → Re-publish to main topic
          ↓ (No)
        Send to DLT with error details → Human investigates later
```

---

### Retry + DLT Pattern

**Q: Why is the original message committed after publishing a retry copy?**
> The consumer commits the original only after the retry copy has been successfully published. Otherwise, a crash could lose both the original progress and the retry attempt.

**Q: What is a danger of re-publishing retries to the same topic?**
> A failing message can repeatedly return to the normal stream and delay other work. Delayed retry topics or a DLT are often better for failures that need time or human intervention.

```csharp
public class ResilientConsumer : BackgroundService
{
    private readonly IConsumer<string, string> _consumer;
    private readonly IProducer<string, string> _dlqProducer;
    private const int MaxRetries = 3;
    private const string DltTopic = "order-events.DLT";

    protected override async Task ExecuteAsync(CancellationToken ct)
    {
        _consumer.Subscribe("order-events");

        while (!ct.IsCancellationRequested)
        {
            var result = _consumer.Consume(ct);
            var retryCount = GetRetryCount(result.Message.Headers);

            try
            {
                await ProcessWithRetry(result, retryCount);
                _consumer.Commit(result);
            }
            catch (Exception ex) when (retryCount < MaxRetries)
            {
                // Re-publish with incremented retry count
                await RepublishForRetry(result, retryCount + 1);
                _consumer.Commit(result); // commit original — retry copy is in the topic
            }
            catch (Exception ex)
            {
                // Max retries exceeded — send to Dead Letter Topic
                await SendToDeadLetter(result, ex);
                _consumer.Commit(result);
            }
        }
    }

    private async Task SendToDeadLetter(ConsumeResult<string, string> result, Exception ex)
    {
        var headers = new Headers(result.Message.Headers)
        {
            { "dlq-reason", Encoding.UTF8.GetBytes(ex.Message) },
            { "dlq-original-topic", Encoding.UTF8.GetBytes(result.Topic) },
            { "dlq-timestamp", Encoding.UTF8.GetBytes(DateTimeOffset.UtcNow.ToString("O")) }
        };

        await _dlqProducer.ProduceAsync(DltTopic,
            new Message<string, string>
            {
                Key = result.Message.Key,
                Value = result.Message.Value,
                Headers = headers
            });
    }

    private int GetRetryCount(Headers headers)
    {
        var header = headers.FirstOrDefault(h => h.Key == "retry-count");
        return header != null ? int.Parse(Encoding.UTF8.GetString(header.GetValueBytes())) : 0;
    }
}
```

---

<p align="right"><a href="#top">Back to top</a></p>

---

## 6. Exactly-Once Semantics (Transactional Producer)

**Q: What does Kafka exactly-once semantics cover?**
> It can atomically combine Kafka reads, Kafka writes, and offset commits in a Kafka-to-Kafka flow. It does not automatically make a database update or external API call exactly once.

**Q: What happens when a transaction is aborted?**
> Kafka discards the transaction's output records for transactional readers, and the input offset is not committed as part of that transaction. The input can then be processed again.

```csharp
var config = new ProducerConfig
{
    BootstrapServers = "localhost:9092",
    TransactionalId = "order-processor-txn-1",  // must be unique per producer instance
    Acks = Acks.All,
    EnableIdempotence = true
};

using var producer = new ProducerBuilder<string, string>(config).Build();

producer.InitTransactions(TimeSpan.FromSeconds(30));

try
{
    producer.BeginTransaction();

    await producer.ProduceAsync("output-topic",
        new Message<string, string> { Key = "k1", Value = "processed-v1" });

    await producer.ProduceAsync("audit-topic",
        new Message<string, string> { Key = "k1", Value = "audit-entry" });

    // Commit consumer offsets as part of the transaction
    producer.SendOffsetsToTransaction(
        new List<TopicPartitionOffset>
        {
            new("input-topic", 0, new Offset(42))
        },
        _consumer.ConsumerGroupMetadata,
        TimeSpan.FromSeconds(10));

    producer.CommitTransaction();
}
catch (Exception)
{
    producer.AbortTransaction();
    throw;
}
```

### Real-World Scenario: E-Commerce Order Processing Pipeline

**Situation:** An online retailer processes orders through multiple transformation stages, all within Kafka:

```
RAW ORDERS                VALIDATED ORDERS              ROUTED ORDERS
(from web, mobile,   ←→  (enriched with price,   ←→  (by fulfillment center)
 3rd-party markets)       inventory, customer tier)    
    │                           │                           │
    ├→ Topic: raw-orders       ├→ Topic: validated-orders  ├→ Topic: orders-west-fc
    │   (thousands/sec)        │   (with headers)          ├→ Topic: orders-east-fc
    │                          │                          ├→ Topic: orders-intl-fc
    └─ Multiple producers      └─ Stream processor        │
                                  (Kafka Streams)         └─ Stream processor
                                                             (routing logic)
                                                          
                                                        Fulfillment centers
                                                        (independent consumers)
```

**Why exactly-once is critical:**

1. **Validation-to-enrichment:**
   - Consumer reads from `raw-orders`
   - Fetches customer tier from database
   - Calculates discounted price
   - Publishes to `validated-orders` with enrichment data
   - **Problem without exactly-once:** Crashes mid-enrichment → order republished with stale/missing pricing

2. **Routing logic:**
   - Consumer reads `validated-orders`
   - Queries inventory system: "Can west DC fulfill this?"
   - Publishes to `orders-west-fc` or `orders-east-fc`
   - **Problem without exactly-once:** Duplicate shipments from different DCs; inventory decremented twice

3. **Preventing order duplication:**
   - Order 12345 enriched and published ✓
   - Processor crashes before committing offset
   - Restarts, sees order 12345 again
   - **Without idempotency + exactly-once:** Two fulfillment requests, customer charged twice, receives duplicate shipment

**The atomic guarantee:**
- Either: order enriched + offset committed (together) ✓
- Or: both rolled back, order re-processed ✓
- Never: order published but offset not committed ✗

This is why exactly-once only works Kafka→Kafka. The database lookup is separate—if the database has side effects (like decrementing inventory), you're back to **at-least-once + idempotent operations** (idempotency key tracking).

> **Interview insight:** Exactly-once in Kafka only works for Kafka-to-Kafka pipelines (consume → transform → produce). The moment you touch an external system (database, API), you're back to at-least-once + idempotency.

---

<p align="right"><a href="#top">Back to top</a></p>

---

## 7. Schema Registry & Serialization

**Q: What problem does a schema registry solve?**
> It gives producers and consumers a shared contract for message data and checks whether schema changes remain compatible. This catches breaking changes before they become runtime failures.

**Q: Is JSON alone a schema registry?**
> No. JSON describes the data format, but a schema registry stores versions, assigns schema IDs, and enforces compatibility rules between versions.

### Why Schema Registry?

**Q: What can go wrong without a schema registry?**
> A producer can change a field or its type while an older consumer is still running. The message may be accepted by Kafka but fail when the consumer tries to deserialize or interpret it.

Without it, producers and consumers silently disagree on message format = runtime failures. Schema Registry enforces compatibility rules at produce time.

```bash
dotnet add package Confluent.SchemaRegistry
dotnet add package Confluent.SchemaRegistry.Serdes.Json
```

### JSON Schema Example

**Q: Why should event contracts evolve backward-compatibly?**
> Existing consumers may still be running when a producer is upgraded. Adding optional fields lets old consumers continue reading new messages while newer consumers use the extra data.

**Q: What should you do when an event change is breaking?**
> Introduce a new event version or topic and migrate consumers deliberately. Avoid changing the meaning of an existing field while keeping its name.

```csharp
// Define your event as a POCO
public class OrderCreatedEvent
{
    public string OrderId { get; set; }
    public string CustomerId { get; set; }
    public decimal Amount { get; set; }
    public DateTime CreatedAt { get; set; }
}

// Producer with schema registry
var schemaRegistryConfig = new SchemaRegistryConfig
{
    Url = "http://localhost:8081"
};

using var schemaRegistry = new CachedSchemaRegistryClient(schemaRegistryConfig);

using var producer = new ProducerBuilder<string, OrderCreatedEvent>(producerConfig)
    .SetValueSerializer(new JsonSerializer<OrderCreatedEvent>(schemaRegistry))
    .Build();

await producer.ProduceAsync("order-events",
    new Message<string, OrderCreatedEvent>
    {
        Key = order.OrderId,
        Value = new OrderCreatedEvent
        {
            OrderId = order.OrderId,
            CustomerId = order.CustomerId,
            Amount = order.TotalAmount,
            CreatedAt = DateTime.UtcNow
        }
    });

// Consumer with schema registry
using var consumer = new ConsumerBuilder<string, OrderCreatedEvent>(consumerConfig)
    .SetValueDeserializer(new JsonDeserializer<OrderCreatedEvent>().AsSyncOverAsync())
    .Build();
```

### Schema Compatibility Modes

**Q: Which compatibility mode is a common production default?**
> `BACKWARD` is common because it allows newer consumers to read older messages, provided new fields have suitable defaults or are optional.

**Q: Why is `NONE` risky outside development?**
> It permits incompatible changes, so a producer can deploy successfully while consumers fail when they receive the new data shape.

| Mode | Allowed Changes | Use Case |
|---|---|---|
| `BACKWARD` (default) | Delete fields, add optional fields | Consumers upgrade first |
| `FORWARD` | Add fields, delete optional fields | Producers upgrade first |
| `FULL` | Add/delete optional fields only | Independent upgrades |
| `NONE` | Anything goes | Development only |

---

<p align="right"><a href="#top">Back to top</a></p>

---

## 8. Kafka vs Azure Service Bus — You'll Be Asked This

**Q: How would you choose between Kafka and Azure Service Bus?**
> Choose Kafka for durable event streams, replay, high throughput, and many independent consumers. Choose Service Bus for commands, workflows, sessions, scheduled delivery, and built-in dead-letter handling.

**Q: Can Kafka and Service Bus be used together?**
> Yes. Kafka can carry the high-volume event stream while Service Bus handles targeted commands or workflow messages. The boundary should be explicit so delivery and retry expectations remain clear.

| Dimension | Kafka | Azure Service Bus |
|---|---|---|
| **Model** | Distributed log (pull) | Message broker (push/pull) |
| **Ordering** | Per-partition | Per-session |
| **Replay** | Yes — seek to any offset | No — once consumed, gone (unless peek-lock expires) |
| **Throughput** | Millions msg/s | Thousands msg/s |
| **Retention** | Configurable (hours to forever) | 14 days max |
| **Dead lettering** | Manual (DLT topic) | Built-in DLQ |
| **Transactions** | Kafka-to-Kafka only | Full AMQP transactions |
| **Managed option** | Confluent Cloud, Azure Event Hubs (Kafka-compatible) | Fully managed PaaS |
| **Best for** | Event streaming, event sourcing, high-volume analytics | Command/request messaging, workflows, enterprise integration |

> **Interview answer:** "Kafka for event streaming and event sourcing where replay and high throughput matter. Service Bus for command messaging, workflows, and when you need built-in DLQ, sessions, and scheduled delivery."

### Topic/Subscription vs Queue — Kafka Equivalents

**Azure Service Bus Queue** ≈ **Kafka Topic with one Consumer Group**
- Messages are consumed once and typically deleted.
- Multiple consumers in the group compete for partitions.

**Azure Service Bus Topic with Subscriptions** ≈ **Kafka Topic with multiple Consumer Groups**
- Multiple independent subscriber groups each read the entire topic.
- Each group maintains its own progress (committed offset).
- One group's consumption does not affect another's.

**Key differences:**

| Aspect | Azure Service Bus | Kafka |
|---|---|---|
| **Replay** | Not possible after ack | Seek to any offset anytime |
| **Multiple subscribers** | Built-in (Subscriptions) | Use separate consumer groups |
| **Filtering** | Per-subscription | Application-level in consumer |
| **Message lifecycle** | Deleted after ack | Retained by time/size policy |
| **Scaling model** | Broker-managed message distribution | Consumer pulls; partition assignment |

**In practice:**

If you have:
- One service consuming orders → Use one Kafka consumer group
- Three services (inventory, billing, email) each consuming the same orders → Use three separate Kafka consumer groups on the same topic

This is why Kafka is often called a "publish-subscribe" system: each consumer group acts like a subscription.

### 8.1 AMQP Transactions

**Q: What are AMQP transactions?**
> AMQP transactions provide all-or-nothing semantics for message operations on an AMQP channel. A producer can publish several messages, and a consumer can acknowledge deliveries within the same transaction. The broker applies all operations when the transaction commits or discards them when it rolls back.

The lifecycle is:

```text
Tx.Select()       -> Enable transaction mode on the channel
    |
Publish / Ack     -> Perform message operations
    |
Tx.Commit()       -> Apply all operations atomically
or Tx.Rollback()  -> Discard all operations
```

In RabbitMQ, transactions are scoped to a single channel, and a channel can have only one active transaction at a time:

```csharp
using var channel = connection.CreateModel();

channel.TxSelect();

try
{
    channel.BasicPublish(
        exchange: "",
        routingKey: "billing-commands",
        basicProperties: null,
        body: Encoding.UTF8.GetBytes("Charge order 12345"));

    channel.BasicPublish(
        exchange: "",
        routingKey: "fulfillment-commands",
        basicProperties: null,
        body: Encoding.UTF8.GetBytes("Ship order 12345"));

    channel.TxCommit();
}
catch
{
    channel.TxRollback();
    throw;
}
```

If the second publish or the commit fails, the transaction can be rolled back so the first publish is not treated as a completed operation. Transactions can also include consumer acknowledgements, which is useful when a workflow must atomically acknowledge an input message while publishing related output messages.

**Q: How do AMQP transactions compare with publisher confirms?**
> Publisher confirms tell the producer that the broker accepted a published message; they do not group several publishes into one atomic unit. AMQP transactions provide grouping and rollback, but require synchronous commit or rollback operations and generally reduce throughput.

| Aspect | AMQP transaction | Kafka transaction |
|---|---|---|
| **Scope** | One AMQP channel | Transactional Kafka producer and participating Kafka partitions/topics |
| **Atomic operations** | Publishes and consumer acknowledgements on that channel | Kafka publishes plus consumed-offset commits |
| **Rollback** | `TxRollback()` discards uncommitted operations | `AbortTransaction()` makes transactional records invisible to `read_committed` consumers |
| **Performance** | Higher coordination cost; commit/rollback is synchronous | Batching and pipelining are designed for event-stream workloads |
| **Best fit** | Small atomic workflows in a traditional message broker | Kafka-to-Kafka consume-transform-produce pipelines |

**Interview insight:** AMQP transactions are broker-managed, channel-scoped atomic operations. They are useful when a small set of queue operations must succeed together, but publisher confirms plus consumer acknowledgements are often preferred for higher-throughput workflows when full transaction atomicity is not required.

---

<p align="right"><a href="#top">Back to top</a></p>

---

## 9. Production Patterns

**Q: Why are production patterns needed when Kafka already stores messages?**
> Kafka solves durable event transport, but it does not make a database write, event publication, retries, or deduplication one atomic operation. These patterns close those application-level gaps.

**Q: Which pattern would you use to avoid publishing an event for a rolled-back database transaction?**
> Use the outbox pattern: save the business change and an outbox record in the same database transaction, then publish the outbox record asynchronously.

### 9.1 Outbox Pattern (Kafka + SQL)

**Q: Does the outbox publisher guarantee exactly-once publication?**
> Not by itself. If it publishes successfully but crashes before marking the row processed, it may publish again. Use a stable event ID and make consumers idempotent.

**Q: How do you safely run multiple outbox publisher instances?**
> Claim rows with a lease or database locking strategy, and make publishing safe to repeat. Do not rely only on reading `ProcessedAt == null` because multiple workers can select the same rows.

Solves: dual-write problem (write to DB + publish to Kafka atomically).

```csharp
// Step 1: Write to DB in a transaction (your SQL knowledge applies directly)
await using var transaction = await _dbContext.Database.BeginTransactionAsync();

var order = new Order { /* ... */ };
_dbContext.Orders.Add(order);

// Write to outbox table in SAME transaction
_dbContext.OutboxMessages.Add(new OutboxMessage
{
    Id = Guid.NewGuid(),
    Topic = "order-events",
    Key = order.Id.ToString(),
    Payload = JsonSerializer.Serialize(new OrderCreatedEvent(order)),
    CreatedAt = DateTime.UtcNow,
    ProcessedAt = null
});

await _dbContext.SaveChangesAsync();
await transaction.CommitAsync();

// Step 2: Background worker polls outbox and publishes to Kafka
public class OutboxPublisher : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken ct)
    {
        while (!ct.IsCancellationRequested)
        {
            var messages = await _dbContext.OutboxMessages
                .Where(m => m.ProcessedAt == null)
                .OrderBy(m => m.CreatedAt)
                .Take(100)
                .ToListAsync(ct);

            foreach (var msg in messages)
            {
                await _producer.ProduceAsync(msg.Topic,
                    new Message<string, string> { Key = msg.Key, Value = msg.Payload });

                msg.ProcessedAt = DateTime.UtcNow;
            }

            await _dbContext.SaveChangesAsync(ct);
            await Task.Delay(1000, ct);
        }
    }
}
```

### 9.2 Idempotent Consumer (at-least-once + dedup)

**Q: What makes a consumer idempotent?**
> Repeating the same event produces the same final business state as processing it once. A durable processed-event record or a database uniqueness constraint is usually safer than an in-memory flag.

**Q: Why is `key + offset` not always a good event ID?**
> An offset is unique only within a partition and can change as data is copied to another topic. Prefer a producer-generated stable event ID when deduplicating across systems.

```csharp
public class IdempotentOrderConsumer
{
    private readonly IDistributedCache _cache; // Redis

    public async Task ProcessAsync(ConsumeResult<string, string> result)
    {
        var messageId = GetMessageId(result); // from header or key+offset

        // Check if already processed
        var existing = await _cache.GetStringAsync($"processed:{messageId}");
        if (existing != null) return; // already handled

        await HandleOrder(result.Message.Value);

        // Mark as processed with TTL matching retention
        await _cache.SetStringAsync($"processed:{messageId}", "1",
            new DistributedCacheEntryOptions { AbsoluteExpirationRelativeToNow = TimeSpan.FromDays(7) });
    }
}
```

### 9.3 Consumer Lag Monitoring

**Q: What is consumer lag?**
> Lag is the distance between the newest available record and the consumer group's committed position. Growing lag means the consumer is falling behind production.

**Q: What should an alert on lag consider?**
> Consider both the lag size and its age or growth rate. A fixed message count may be harmless during a burst but serious if it keeps increasing or violates processing latency targets.

```csharp
// Admin client to check consumer lag
using var adminClient = new AdminClientBuilder(
    new AdminClientConfig { BootstrapServers = "localhost:9092" }).Build();

var groups = adminClient.ListConsumerGroups();
// Use with Prometheus/Grafana for alerting

// Or use the consumer's statistics callback
var config = new ConsumerConfig
{
    StatisticsIntervalMs = 5000 // emit stats every 5s
};

var consumer = new ConsumerBuilder<string, string>(config)
    .SetStatisticsHandler((_, json) =>
    {
        var stats = JsonDocument.Parse(json);
        // Extract consumer_lag from stats JSON
    })
    .Build();
```

---

<p align="right"><a href="#top">Back to top</a></p>

---

## 10. Rebalancing — The Gotcha

**Q: What causes a consumer group rebalance?**
> Consumers joining, leaving, crashing, or failing to poll within `MaxPollIntervalMs` can trigger a rebalance so Kafka can redistribute partitions.

**Q: How can long processing cause a rebalance?**
> If processing blocks the consumer from polling for longer than the maximum poll interval, Kafka assumes it is unhealthy and assigns its partitions elsewhere. Separate polling from long-running work or tune the interval carefully.

When a consumer joins/leaves a group, Kafka **rebalances** — reassigns partitions. During rebalance, **no messages are consumed**.

### Rebalance Strategies

**Q: What is the benefit of CooperativeSticky assignment?**
> It changes assignments incrementally, so healthy consumers keep partitions where possible. This usually reduces the pause and duplicate work caused by a full eager rebalance.

**Q: What should happen when partitions are revoked?**
> Stop work for those partitions and commit only successfully processed offsets before giving them up. Otherwise, the next owner may repeat more work than necessary.

| Strategy | Behavior | Config |
|---|---|---|
| **Eager (Range/RoundRobin)** | Revoke ALL partitions, reassign all | Default |
| **Cooperative (Incremental)** | Only reassign affected partitions | `PartitionAssignmentStrategy = "CooperativeSticky"` |

```csharp
var config = new ConsumerConfig
{
    PartitionAssignmentStrategy = PartitionAssignmentStrategy.CooperativeSticky,
    // Reduces rebalance downtime significantly
};
```

> **Interview tip:** Always mention CooperativeSticky. It shows you understand rebalancing impact on availability.

---

<p align="right"><a href="#top">Back to top</a></p>

---

## 11. Kafka Streams Concepts (Not .NET Native)

**Q: What is the difference between a KStream and a KTable?**
> A `KStream` represents a sequence of events. A `KTable` represents the latest value for each key, usually maintained from a changelog stream.

**Q: When might raw consumers be preferable to a streams library in .NET?**
> Raw consumers can be simpler for straightforward event handling and give direct control over external databases and APIs. A streams library is useful when you need reusable stateful processing, joins, or windows.

Kafka Streams is JVM-only, but you'll be asked about it. Know the concepts:

| Concept | What It Does | .NET Alternative |
|---|---|---|
| **KStream** | Unbounded event stream | Manual consumer loop |
| **KTable** | Changelog → materialized view | Consumer + local RocksDB/SQLite |
| **GlobalKTable** | Full copy on every instance | Shared cache (Redis) |
| **Windowing** | Time-based aggregation | Custom logic with `Timestamp` |
| **Join** | Stream-stream, stream-table | Manual correlation by key |

> For .NET, use **Streamiz** (community Kafka Streams port) or build with raw consumers + state stores.

---

<p align="right"><a href="#top">Back to top</a></p>

---

## 12. Docker Compose — Local Dev

**Q: Why is this Docker Compose setup suitable for development but not production?**
> It runs a single Kafka broker with plaintext networking and replication factor 1. Production needs multiple brokers, secure listeners, persistent storage, monitoring, backups, and carefully managed credentials.

**Q: What is KRaft in this setup?**
> KRaft is Kafka's built-in metadata quorum. It replaces the separate ZooKeeper dependency for cluster coordination and simplifies the deployment.

```yaml
version: '3.8'
services:
  kafka:
    image: confluentinc/cp-kafka:7.6.0
    ports:
      - "9092:9092"
    environment:
      KAFKA_NODE_ID: 1
      KAFKA_PROCESS_ROLES: broker,controller
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092,CONTROLLER://0.0.0.0:9093
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@localhost:9093
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_LOG_RETENTION_HOURS: 168
      CLUSTER_ID: 'MkU3OEVBNTcwNTJENDM2Qk'

  schema-registry:
    image: confluentinc/cp-schema-registry:7.6.0
    ports:
      - "8081:8081"
    environment:
      SCHEMA_REGISTRY_HOST_NAME: schema-registry
      SCHEMA_REGISTRY_KAFKASTORE_BOOTSTRAP_SERVERS: kafka:9092

  kafka-ui:
    image: provectuslabs/kafka-ui:latest
    ports:
      - "8080:8080"
    environment:
      KAFKA_CLUSTERS_0_NAME: local
      KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka:9092
      KAFKA_CLUSTERS_0_SCHEMAREGISTRY: http://schema-registry:8081
```

> This uses KRaft mode (no ZooKeeper). ZooKeeper is deprecated since Kafka 3.3+. If an interviewer mentions ZooKeeper, mention KRaft migration.

---

<p align="right"><a href="#top">Back to top</a></p>

---

## 13. Interview Questions & Answers

### Architecture

**Q: Why does Kafka keep messages after a consumer reads them?**
> Kafka separates reading from deletion. Retention is controlled by time or size, so consumers can replay data or recover after downtime without the producer sending it again.

**Q: How does Kafka distribute messages to brokers?**
> Topic partitions are distributed across brokers. One replica is the leader for a partition and handles normal reads and writes, while follower replicas copy the data for failover.

**Q: How does Kafka guarantee message ordering?**
> Ordering is guaranteed within a single partition only. Use a consistent partition key (e.g., customer ID) to ensure all events for an entity go to the same partition. Cross-partition ordering requires application-level logic (timestamps, sequence numbers).

**Q: What happens when a consumer in a group crashes?**
> Kafka detects missing heartbeats after `session.timeout.ms`. It triggers a rebalance — the crashed consumer's partitions are reassigned to surviving consumers. With CooperativeSticky, only the orphaned partitions move. With Eager, all partitions revoke and reassign.

**Q: How does Kafka achieve high throughput?**
> 1. Sequential disk I/O (append-only log — faster than random writes)
> 2. Zero-copy transfer (sendfile syscall — data goes from disk to NIC without user-space copy)
> 3. Batching (producer batches messages, broker writes batches)
> 4. Page cache (OS file cache, no JVM GC overhead)
> 5. Partition parallelism (horizontal scaling)

**Q: Explain ISR (In-Sync Replicas).**
> ISR = set of replicas that are caught up with the leader within `replica.lag.time.max.ms`. When `acks=all`, a produce is only acknowledged when all ISR replicas have written it. If a follower falls behind, it's removed from ISR. `min.insync.replicas` sets the minimum ISR size for `acks=all` to succeed — typically set to 2 with replication factor 3.

### .NET Specific

**Q: How do you shut down a Kafka consumer gracefully in .NET?**
> Cancel the hosted service, stop polling, call `Close()` to leave the group cleanly, and dispose the consumer. This reduces unnecessary rebalances and gives the application a chance to commit completed work.

**Q: How do you prevent a slow external API from blocking Kafka consumption?**
> Bound concurrency, keep polling within the allowed interval, and use timeouts and circuit breakers for the API. Commit only after the event's required work succeeds, and route repeated failures to retry or DLT handling.

**Q: Should the Kafka producer be a singleton or transient?**
> **Singleton.** The producer maintains TCP connections, batching buffers, and internal threads. Creating a producer per request is expensive and defeats batching. Register as singleton in DI.

**Q: How do you handle poison messages (deserialization failures)?**
> Use a `SetErrorHandler` on the consumer that logs the raw message. For deserialization, wrap with a try-catch in a custom deserializer, or use `SetValueDeserializer` with error handling that routes to a DLT. Never let a poison message block the partition.

**Q: Explain `EnableAutoCommit` vs manual commit.**
> Auto-commit commits offsets on a timer (`auto.commit.interval.ms`). If processing takes longer than the interval, you get at-most-once delivery — offsets commit before processing finishes. Manual commit (`EnableAutoCommit = false`) gives at-least-once: you commit after successful processing. Trade-off: manual is safer but you must handle `CommitAsync` failures.

**Q: What's the relationship between partitions and consumer parallelism?**
> In a consumer group, each partition is assigned to exactly one consumer. So max parallelism = partition count. If you have 6 partitions and 8 consumers, 2 sit idle. You can't increase parallelism without adding partitions — and repartitioning redistributes keys.

### Design / Scenario

**Q: How would you design an order-events topic?**
> Use the order ID as the key to preserve event order for each order, choose enough partitions for expected parallelism, define a backward-compatible event schema, and monitor consumer lag and DLT volume.

**Q: How would you handle a database outage while consuming events?**
> Do not commit the offset until the database operation succeeds. Apply bounded retries for temporary failures, then pause or route messages to a retry topic or DLT according to the recovery policy. Ensure the database operation is idempotent.

**Q: How would you implement event sourcing with Kafka?**
> Kafka is the event store. Each aggregate type gets a topic (e.g., `order-events`). Aggregate ID is the partition key. Consumers build read models (projections) by replaying events. Compacted topics can serve as snapshot stores. Caveat: Kafka lacks conditional writes (optimistic concurrency) — you need an external sequence check or use the Outbox pattern with a database as the primary store.

**Q: Design a real-time analytics pipeline.**
> ```
> Microservices → Kafka Topics → Stream Processor (Flink/ksqlDB) → Kafka Topics → Sink Connectors → Data Store (ClickHouse/Elastic)
> ```
> In .NET: microservices produce events, background consumers aggregate windows and write to time-series DB. Use consumer groups for horizontal scaling.

**Q: How do you handle schema evolution in production?**
> Schema Registry with BACKWARD compatibility. Consumers can read old and new formats. Add fields with defaults (backward-compatible). Never rename or remove required fields without a migration topic. Version your events: `OrderCreatedV1`, `OrderCreatedV2` — or use a `schemaVersion` header.

**Q: Kafka vs Event Hubs for an Azure shop?**
> Event Hubs has a Kafka-compatible endpoint — you can use `Confluent.Kafka` client against Event Hubs with minimal config changes. Use Event Hubs when: you want managed PaaS, Azure-native integration (Functions, Stream Analytics), and don't need infinite retention. Use Kafka (self-hosted or Confluent Cloud) when: you need compacted topics, Kafka Streams, Connect ecosystem, or tiered storage.

---

<p align="right"><a href="#top">Back to top</a></p>

---

## 14. Common Pitfalls

**Q: Why is using a null key everywhere a problem?**
> Records can be spread across partitions without entity-level ordering. Use a meaningful stable key when events for the same customer, order, or account must be processed in sequence.

**Q: Why can adding partitions be disruptive?**
> More partitions increase possible parallelism, but they also change distribution and can move keyed traffic. Consumers rebalance, and existing assumptions about capacity and ordering must be reviewed.

| Pitfall | Why It Hurts | Fix |
|---|---|---|
| `Key = null` everywhere | No ordering guarantee, round-robin distribution | Always set a meaningful key |
| Too few partitions | Can't scale consumers | Start with 3× expected consumer count |
| Too many partitions | More memory, longer rebalances, more leader elections | 6-12 per topic for most workloads |
| `MaxPollIntervalMs` too low | Long-processing messages cause rebalance | Set to 2-5× your max processing time |
| Not handling rebalance | Duplicate processing on reassignment | Commit in `PartitionsRevokedHandler` |
| Single consumer group for everything | Tight coupling | One consumer group per service/concern |
| Ignoring consumer lag | Silent data loss or processing delays | Monitor lag, alert when > threshold |
| `acks=1` for critical data | Leader crash before replication = data loss | Use `acks=all` + `min.insync.replicas=2` |

---

<p align="right"><a href="#top">Back to top</a></p>

---

## 15. Quick Reference — Config Cheat Sheet

**Q: Which producer settings are a sensible durable baseline?**
> Use `Acks.All`, idempotence, retries, and compression where appropriate. Tune batching settings against latency requirements instead of copying defaults blindly.

**Q: Which consumer settings support at-least-once processing?**
> Disable automatic commits, process the message successfully, and commit afterward. Set `MaxPollIntervalMs` high enough for the worst expected processing time and monitor rebalances.

### Producer

**Q: Why should producer configuration be tuned instead of copied unchanged?**
> Larger batches and compression can improve throughput, but they may add latency or CPU usage. Settings should match the message size, traffic pattern, and durability requirement.

```csharp
new ProducerConfig
{
    BootstrapServers = "broker1:9092,broker2:9092",
    Acks = Acks.All,
    EnableIdempotence = true,
    MaxInFlight = 5,
    LingerMs = 5,              // batch window
    BatchSize = 16384,         // 16KB
    CompressionType = CompressionType.Snappy,
    MessageSendMaxRetries = 3,
    RetryBackoffMs = 100
};
```

### Consumer

**Q: What is the relationship between `SessionTimeoutMs` and `MaxPollIntervalMs`?**
> `SessionTimeoutMs` detects a consumer that stops heartbeating, while `MaxPollIntervalMs` detects a consumer that does not poll often enough. Long processing can violate the poll interval even while the consumer is still connected.

```csharp
new ConsumerConfig
{
    BootstrapServers = "broker1:9092,broker2:9092",
    GroupId = "my-service",
    AutoOffsetReset = AutoOffsetReset.Earliest,
    EnableAutoCommit = false,
    EnableAutoOffsetStore = false,
    SessionTimeoutMs = 45000,
    MaxPollIntervalMs = 300000,
    PartitionAssignmentStrategy = PartitionAssignmentStrategy.CooperativeSticky,
    IsolationLevel = IsolationLevel.ReadCommitted
};
```

---

<p align="right"><a href="#top">Back to top</a></p>

---

## 16. Testing Kafka in .NET

**Q: What should a Kafka integration test verify?**
> Start a real Kafka instance, publish a record, consume it with a test group, and verify key, value, headers, partition behavior, and offset handling. Also test restart and failure paths when those guarantees matter.

**Q: What should unit tests mock and what should integration tests cover?**
> Unit tests can mock producer and consumer interfaces to test application decisions quickly. Integration tests should cover serialization, broker configuration, commits, partition assignment, and real retry or DLT behavior.

```csharp
// Use Testcontainers for integration tests
dotnet add package Testcontainers.Kafka

public class KafkaIntegrationTests : IAsyncLifetime
{
    private readonly KafkaContainer _kafka = new KafkaBuilder()
        .WithImage("confluentinc/cp-kafka:7.6.0")
        .Build();

    public async Task InitializeAsync() => await _kafka.StartAsync();
    public async Task DisposeAsync() => await _kafka.DisposeAsync();

    [Fact]
    public async Task ProduceAndConsume_RoundTrip()
    {
        var bootstrapServers = _kafka.GetBootstrapAddress();

        // Produce
        using var producer = new ProducerBuilder<string, string>(
            new ProducerConfig { BootstrapServers = bootstrapServers }).Build();

        await producer.ProduceAsync("test-topic",
            new Message<string, string> { Key = "k1", Value = "hello" });

        // Consume
        using var consumer = new ConsumerBuilder<string, string>(
            new ConsumerConfig
            {
                BootstrapServers = bootstrapServers,
                GroupId = "test-group",
                AutoOffsetReset = AutoOffsetReset.Earliest
            }).Build();

        consumer.Subscribe("test-topic");
        var result = consumer.Consume(TimeSpan.FromSeconds(10));

        Assert.Equal("k1", result.Message.Key);
        Assert.Equal("hello", result.Message.Value);
    }
}

// Unit testing: mock IProducer<TKey, TValue>
// Confluent.Kafka interfaces are mockable with Moq/NSubstitute
```

---

<p align="right"><a href="#top">Back to top</a></p>

---

## Cheat Sheet — One-Page Summary

```
ARCHITECTURE:  Broker cluster → Topics → Partitions → Offsets
               Consumer Groups for parallel processing
               Replication Factor for fault tolerance

GUARANTEES:    Ordering = per-partition only (use consistent key)
               Durability = acks=all + min.insync.replicas=2
               At-least-once = manual commit after processing
               Exactly-once = transactional API (Kafka-to-Kafka only)

.NET CLIENT:   Confluent.Kafka — producer=singleton, consumer=BackgroundService
               Manual commit for at-least-once
               Schema Registry for type-safe serialization

KEY PATTERNS:  Outbox Pattern — atomic DB write + eventual Kafka publish
               Idempotent Consumer — dedup with Redis/DB check
               Dead Letter Topic — failed messages after N retries
               CooperativeSticky — minimize rebalance disruption

SCALING:       Partitions = max consumer parallelism
               Consumer count ≤ partition count
               Multiple consumer groups = independent subscribers

VS AZURE SB:   Kafka = log, replay, high throughput, event streaming
               Service Bus = queue, workflows, DLQ built-in, lower throughput
```

<p align="right"><a href="#top">Back to top</a></p>
