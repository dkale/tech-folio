# Azure Service Bus

## Table of Contents

- [Core Concepts and Features](#core-concepts-and-features)
  - [What are the key features of Azure Service Bus?](#what-are-the-key-features-of-azure-service-bus)
  - [What are queues and topics in Azure Service Bus?](#what-are-queues-and-topics-in-azure-service-bus)
  - [What is the difference between a queue and a topic in Azure Service Bus?](#what-is-the-difference-between-a-queue-and-a-topic-in-azure-service-bus)
  - [What is a subscription in the context of Azure Service Bus topics?](#what-is-a-subscription-in-the-context-of-azure-service-bus-topics)
  - [What is a message session in Azure Service Bus?](#what-is-a-message-session-in-azure-service-bus)
  - [What is a dead-letter queue (DLQ)? When is it used?](#what-is-a-dead-letter-queue-dlq-when-is-it-used)
- [Message Processing and Reliability](#message-processing-and-reliability)
  - [What are message locks in Azure Service Bus?](#what-are-message-locks-in-azure-service-bus)
  - [What is the difference between PeekLock and ReceiveAndDelete modes?](#what-is-the-difference-between-peeklock-and-receiveanddelete-modes)
  - [What happens if a message is not completed in PeekLock mode?](#what-happens-if-a-message-is-not-completed-in-peeklock-mode)
  - [How do you handle duplicate messages in Azure Service Bus?](#how-do-you-handle-duplicate-messages-in-azure-service-bus)
  - [How would you implement retry and error handling logic in a distributed system using Azure Service Bus?](#how-would-you-implement-retry-and-error-handling-logic-in-a-distributed-system-using-azure-service-bus)
  - [If a consumer crashes after receiving a message but before completing it, what happens to the message? How would you design for resilience?](#if-a-consumer-crashes-after-receiving-a-message-but-before-completing-it-what-happens-to-the-message-how-would-you-design-for-resilience)
  - [Can a message be read by multiple consumers in a queue? If not, how do you simulate that behavior?](#can-a-message-be-read-by-multiple-consumers-in-a-queue-if-not-how-do-you-simulate-that-behavior)
- [.NET Implementation](#net-implementation)
  - [How can you send and receive messages in Azure Service Bus using .NET?](#how-can-you-send-and-receive-messages-in-azure-service-bus-using-net)
- [Messaging Patterns and Trade-offs](#messaging-patterns-and-trade-offs)
  - [What are the pros and cons of using Azure Service Bus Topics vs. multiple Queues for pub/sub scenarios?](#what-are-the-pros-and-cons-of-using-azure-service-bus-topics-vs-multiple-queues-for-pubsub-scenarios)
- [AgData Microservice Design Considerations](#agdata-microservice-design-considerations)
  - [In our AgData microservice, we have PeekLock receive mode for queues, while ReceiveAndDelete mode for topic subscriptions. Why?](#in-our-agdata-microservice-we-have-peeklock-receive-mode-for-queues-while-receiveanddelete-mode-for-topic-subscriptions-why)

---

## Core Concepts and Features

### What are the key features of Azure Service Bus?

✅ Message Queues and Topics

Queues for point-to-point communication.

Topics with Subscriptions for publish/subscribe (pub/sub) patterns.

✅ Advanced Messaging Patterns

Sessions for message grouping and ordered processing.

Message Deferral to postpone processing without losing messages.

Scheduled Messages to delay message delivery to a specified time.

Dead-lettering for handling unprocessable messages.

✅ Reliable Message Delivery

At-least-once delivery with PeekLock mode to ensure messages aren’t lost.

Duplicate detection using message ID and time window.

✅ Security & Access Control

Role-based access control (RBAC) with Azure AD or Shared Access Signatures (SAS).

Firewall rules, Virtual Network (VNet) integration, and Private Endpoints.

✅ Auto-forwarding and Chaining

Automatically forward messages from one queue/topic to another for routing scenarios.

✅ Geo-disaster Recovery

Geo-redundant namespaces for business continuity and disaster recovery.

✅ Transaction Support

Atomic operations across multiple queues and topics using Transactions.

### What are queues and topics in Azure Service Bus?

Queues are used for point-to-point communication. A sender sends a message to the queue, and one receiver reads it. Each message is processed by only one consumer.

Topics are used for publish/subscribe scenarios. A sender (publisher) sends messages to a topic, and multiple subscribers can receive a copy of the message based on filtering rules.

### What is the difference between a queue and a topic in Azure Service Bus?

| Feature | Queue | Topic |
| --- | --- | --- |
| Communication Pattern | Point-to-point | Publish/subscribe |
| Message Receiver | One receiver | Multiple subscribers |
| Filtering | Not applicable | Supports filters for subscriptions |
| Use Case | Single consumer scenarios | Broadcasting to multiple receivers |

### What is a subscription in the context of Azure Service Bus topics?

A subscription is like a virtual queue that belongs to a topic.

Each subscription gets its own copy of the message published to the topic.

You can add filter rules to control which messages each subscription receives.

### What is a message session in Azure Service Bus?

A message session enables grouping of related messages for ordered processing.

It allows a consumer to receive all messages that share the same SessionId in order.

Commonly used when processing message workflows or batches that must stay together.

### What is a dead-letter queue (DLQ)? When is it used?

A dead-letter queue is a secondary sub-queue used to store messages that cannot be delivered or processed.

A message is moved to the DLQ if:

- It is explicitly dead-lettered by the receiver.
- It exceeds the maximum delivery attempts.
- There is a filtering or processing error (e.g., TTL expired).

[↑ Back to Top](#table-of-contents)

## Message Processing and Reliability

### What are message locks in Azure Service Bus?

A message lock prevents other consumers from processing the same message while it's being handled.

It’s used in PeekLock mode to ensure reliable and safe message processing (i.e., no message loss or duplication).

### What is the difference between PeekLock and ReceiveAndDelete modes?

| Mode | Description |
| --- | --- |
| PeekLock | Messages are locked when received and must be explicitly completed. If processing fails, the message can be reprocessed. |
| ReceiveAndDelete | Message is removed from the queue immediately when received — no second chance. Use this only when you can afford message loss. |

### What happens if a message is not completed in PeekLock mode?

The message lock expires (default: 30 seconds), and the message becomes available again for other receivers.

It can also be abandoned manually, which puts it back in the queue.

After multiple failed deliveries (default: 10), the message can go to the Dead-Letter Queue (DLQ).

### How do you handle duplicate messages in Azure Service Bus?

Enable duplicate detection on the queue or topic.

Set the MessageId when sending messages.

Azure Service Bus will ignore duplicate messages that have the same MessageId sent within a defined duplicate detection time window (e.g., 10 minutes).

### How would you implement retry and error handling logic in a distributed system using Azure Service Bus?

**✅ Answer:**

PeekLock Mode: Process messages in PeekLock and only CompleteAsync() after successful handling.

Retry Strategy:

- Implement retries with exponential backoff in the consumer.
- Use libraries like Polly for retry logic.
- Abandon on transient errors: Message becomes available for reprocessing.
- Dead-letter on permanent errors: Use DeadLetterAsync() with reason details.
- Max Delivery Count: Configure (default is 10). After this, the message moves to the DLQ.
- Monitoring: Continuously monitor DLQ and integrate alerts for failed messages.
> 🧠 Tip: Always distinguish between transient and non-transient errors in your handler logic.

### If a consumer crashes after receiving a message but before completing it, what happens to the message? How would you design for resilience?

**Answer:**

If the consumer crashes before calling CompleteMessageAsync() (in PeekLock mode), the lock will expire after the configured lock duration (default: 30 seconds).

Once the lock expires, the message is made available again in the queue for another consumer to process.

**✅ To design for resilience:**

- Use PeekLock mode (default).
- Avoid ReceiveAndDelete, which would lose the message in a crash.
- Implement idempotent processing so duplicate processing doesn’t harm the system (e.g., by checking if a message has already been handled).
- Use retry logic (e.g., exponential backoff) for transient failures.
- Set the MaxDeliveryCount on the queue/subscription; after that, the message is moved to the Dead-Letter Queue (DLQ).

### Can a message be read by multiple consumers in a queue? If not, how do you simulate that behavior?

**Answer:**

By default, Azure Service Bus Queues follow a competing consumer model — only one consumer receives each message.

You cannot natively fan out a queue message to multiple consumers.

✅ To simulate this behavior:

Use a Service Bus Topic with multiple Subscriptions.

A single message published to the topic is copied to each subscription.

Each subscription acts like an independent queue for each consumer.

Add subscription filters to customize who receives what.

> 🧠 Bonus Tip: If you need multiple services to act on the same message, this pattern is more appropriate than using one queue.

[↑ Back to Top](#table-of-contents)

## .NET Implementation

### How can you send and receive messages in Azure Service Bus using .NET?

Use the Azure.Messaging.ServiceBus SDK. Example:

Sending a message:

```csharp
var client = new ServiceBusClient(connectionString);
var sender = client.CreateSender(queueName);

ServiceBusMessage message = new ServiceBusMessage("Hello, Service Bus!");
await sender.SendMessageAsync(message);
```

Receiving a message:

```csharp
var receiver = client.CreateReceiver(queueName);
ServiceBusReceivedMessage message = await receiver.ReceiveMessageAsync();

Console.WriteLine(message.Body.ToString());
await receiver.CompleteMessageAsync(message); // Acknowledge message
```
[↑ Back to Top](#table-of-contents)

## Messaging Patterns and Trade-offs

### What are the pros and cons of using Azure Service Bus Topics vs. multiple Queues for pub/sub scenarios?

**✅ Answer:**

| Criteria | Topics (with Subscriptions) | Multiple Queues |
| --- | --- | --- |
| Pros | - Native pub/sub<br>- Centralized publisher<br>- Filters for subscribers | - Simple routing logic<br>- Decoupled systems |
| Cons | - Complex filtering logic<br>- Harder to monitor per subscription<br>- Shared throughput | - Duplicated messages<br>- Higher maintenance |
| Best Use | Broadcast to multiple systems with different needs | Simple, independent queue-based processing |

> 🧠 Tip: Use topics when subscribers have diverse filtering or processing rules. Use queues when consumers are completely isolated.

[↑ Back to Top](#table-of-contents)

## AgData Microservice Design Considerations

### In our AgData microservice, we have PeekLock receive mode for queues, while ReceiveAndDelete mode for topic subscriptions. Why?

Why use PeekLock for Queues, but ReceiveAndDelete for Topic Subscriptions in a microservice architecture?

It likely comes down to the role each plays in your system and the trade-offs between reliability and performance.

**🔹 PeekLock for Queues — Reliability First**

**✅ Why?**

Queues are usually used for critical, one-to-one commands — e.g., process an order, reserve inventory, charge a customer.

In these cases, message loss is unacceptable.

PeekLock ensures that:

- A message is not removed from the queue until it’s explicitly completed.
- If processing fails (crash, timeout, etc.), the message returns to the queue for retry.
- You can use dead-lettering for messages that fail too many times.
> 🔐 Ideal for command-driven, state-altering, or financially sensitive workflows.

**🔹 ReceiveAndDelete for Topic Subscriptions — Fire & Forget**

**✅ Why?**

Topic subscriptions are often used for events, logging, or notifications.

- Example: Send an "OrderPlaced" event to multiple services.
These events are usually non-critical for the sender — they just publish and forget.

ReceiveAndDelete:

- Boosts performance — no lock management, no extra round trips to complete the message.
- Suitable when occasional message loss is acceptable (e.g., logging or telemetry).

> 🚀 Ideal for high-throughput, non-critical, or broadcast messaging scenarios.

**✅ Summary**

| Aspect | PeekLock (Queue) | ReceiveAndDelete (Topic Subscription) |
| --- | --- | --- |
| Used for | Commands / Tasks | Events / Notifications |
| Loss tolerance | Low | Medium to High |
| Failure handling | Retry, Dead-letter | Message is gone after receive |
| Performance | Slightly slower due to lock + completion | Faster — no extra operations |
| Reliability | High | Best-effort |

> 👀 Final Thought

Your system is likely optimized for reliability on queue-based commands and performance for event-driven topics — a smart and common trade-off in distributed systems.

[↑ Back to Top](#table-of-contents)
