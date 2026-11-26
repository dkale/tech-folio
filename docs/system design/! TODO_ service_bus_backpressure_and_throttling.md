# Backpressure and Throttling When a Microservice Sends Messages to Service Bus

## Question

How can a microservice sending messages to a Service Bus throttle the dispatch?

If the messages in the queue are piling up, shouldn't we lower the rate of these messages from the producer? But how will the producer handle the backpressure?

---

## 1. The basic situation

Imagine:

```text
Producer
   │
   │  1,000 msg/sec
   ▼
Service Bus Queue
   │
   │  200 msg/sec
   ▼
Consumer
```

The queue will grow by roughly:

```text
1,000 - 200 = 800 messages/sec
```

Eventually you need one or more of:

- producer throttling
- more consumers
- faster consumers
- batching
- load shedding
- a larger queue
- some combination of these

The key question is: **how does the producer know that it should slow down?**

This is essentially **backpressure**.

---

## 2. Service Bus doesn't normally automatically tell the producer that the consumer is overloaded

This is where messaging systems differ from synchronous HTTP.

The producer typically does:

```text
SendMessageAsync(message)
```

and Service Bus accepts the message.

The producer doesn't automatically receive a message such as:

> "The consumer is overloaded. Please slow down."

Instead, the producer needs a **feedback mechanism**.

There are several ways to implement this.

---

## 3. Approach 1 — Producer checks queue depth

The producer can periodically check queue metrics:

```text
Queue depth < 10,000
       ↓
Produce normally

Queue depth 10,000–50,000
       ↓
Reduce production rate

Queue depth > 50,000
       ↓
Throttle heavily / stop temporarily
```

For example:

```text
Normal    → 1,000 msg/sec
Warning   →   500 msg/sec
Critical  →   100 msg/sec
Emergency →   pause
```

Azure Service Bus exposes metrics such as active message count, which can be used for this type of control.

However, it is generally undesirable to make every producer message depend on a queue-depth query. That adds latency and unnecessary load.

A better approach is usually to use metrics periodically and maintain a local throttling state.

---

## 4. Approach 2 — Limit producer concurrency

This is often more useful.

Suppose your application receives 10,000 events.

A naive implementation might effectively do:

```text
for each event
    SendMessageAsync()
```

If all sends are fired concurrently, you can overwhelm the Service Bus client or downstream systems.

Instead, allow only a fixed number of concurrent sends:

```text
             ┌── Send ──┐
             ├── Send ──┤
Producer ────┼── Send ──┼── Service Bus
             ├── Send ──┤
             └── Send ──┘
                max 20
```

When all 20 slots are occupied, new work has to wait.

For example:

```csharp
await semaphore.WaitAsync();

try
{
    await serviceBusSender.SendMessageAsync(message);
}
finally
{
    semaphore.Release();
}
```

This doesn't directly look at queue depth. It controls how aggressively your service sends.

---

## 5. Approach 3 — Use a bounded internal queue

One of the strongest patterns for a high-throughput producer is:

```text
Application
    ↓
Bounded in-memory queue
    ↓
Producer workers
    ↓
Service Bus
```

For example:

```text
             ┌─────────────┐
             │ Application │
             └──────┬──────┘
                    │
                    ▼
          ┌───────────────────┐
          │ Bounded Queue     │
          │ capacity = 10,000 │
          └─────────┬─────────┘
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
       Worker 1  Worker 2  Worker 3
          │         │         │
          └─────────┼─────────┘
                    ▼
              Service Bus
```

When the internal queue reaches 10,000:

```text
Application
     ↓
Bounded queue FULL
     ↓
producer must WAIT
```

That's **real backpressure**.

You have effectively transferred the pressure all the way back to the source.

### Important caveat

An in-memory queue is not durable. If losing buffered messages during a process crash is unacceptable, use a durable mechanism instead of relying only on in-memory buffering.

---

## 6. But should the producer always slow down when the queue grows?

Not necessarily.

A queue is specifically designed to absorb temporary differences between production and consumption.

For example:

```text
Production: 1,000/sec
Consumption: 800/sec
```

This may be perfectly fine for a few minutes.

The queue is acting as a **buffer**.

The real problem occurs when:

```text
Production > Consumption
```

persistently.

Then the queue becomes unbounded and message latency keeps increasing.

So queue depth alone is not enough. **Queue growth rate and message age are also important signals.**

---

## 7. Approach 4 — Autoscale consumers

Instead of immediately throttling the producer, increase consumer capacity.

For example:

```text
                  Service Bus
                      │
              1,000 msg/sec
                      │
       ┌──────────────┼──────────────┐
       ▼              ▼              ▼
   Consumer 1     Consumer 2     Consumer 3
    200/sec        200/sec        200/sec
```

Total consumption:

```text
600 msg/sec
```

The queue still grows.

Scale to six consumers:

```text
6 × 200 = 1,200 msg/sec
```

Now:

```text
Production = 1,000/sec
Consumption = 1,200/sec
```

The queue drains.

This is why **queue depth and message age are often used as autoscaling signals**.

---

## 8. What if the producer itself cannot slow down?

Suppose:

```text
Kafka → Your Microservice → Service Bus → Consumer
```

If Service Bus becomes overloaded, your microservice needs somewhere to put incoming work.

There are several choices.

### Option A — Block

```text
Incoming event
      ↓
Producer busy
      ↓
WAIT
```

This works when the upstream system supports backpressure.

### Option B — Buffer

```text
Upstream
   ↓
Your service
   ↓
Bounded queue
   ↓
Service Bus
```

This works well when short bursts are expected.

### Option C — Reject or defer

For an HTTP upstream, for example:

```text
HTTP 429
Retry-After
```

This requires the upstream system to support retries.

### Option D — Persist locally

For stronger durability:

```text
Incoming request
       ↓
Durable DB / local durable queue
       ↓
Background publisher
       ↓
Service Bus
```

Now temporary Service Bus problems do not necessarily result in lost work.

---

## 9. Service Bus throttling is another form of backpressure

Azure Service Bus can itself throttle clients when service limits are exceeded.

The producer may encounter transient failures such as `ServerBusyException`.

The producer should use:

```text
Retry
+
Exponential backoff
+
Jitter
```

For example:

```text
failure
  ↓
wait 100ms
  ↓
failure
  ↓
wait 250ms
  ↓
failure
  ↓
wait 700ms
  ↓
success
```

This is **reactive backpressure**.

It is different from queue-depth-based throttling.

---

## 10. A production-grade architecture

For a serious microservice architecture, combine several mechanisms:

```text
                    ┌──────────────────┐
                    │   Upstream       │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │ Producer Service │
                    │                  │
                    │ Rate limiter     │
                    │ Concurrency      │
                    │ Bounded buffer   │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │   Azure Service  │
                    │      Bus         │
                    └────────┬─────────┘
                             │
                     queue depth / age
                             │
                             ▼
                    ┌──────────────────┐
                    │ Autoscaler       │
                    └────────┬─────────┘
                             │
                  ┌──────────┼──────────┐
                  ▼          ▼          ▼
              Consumer    Consumer    Consumer
```

### Producer side

Use:

- bounded concurrency
- rate limiting where necessary
- bounded buffering
- retry with exponential backoff + jitter
- circuit breaker for prolonged failures

### Service Bus

Monitor:

- active message count
- oldest message age
- dead-letter count
- incoming/outgoing message rates
- throttling/errors

### Consumer side

Use:

- controlled concurrency
- autoscaling based on queue depth and/or message age
- idempotent processing
- dead-letter handling

---

## 11. The complete backpressure feedback loop

The most important principle is:

> Don't think of backpressure as simply "the queue is full, so Service Bus tells the producer to slow down."

Instead, think of it as a **chain of pressure**:

```text
Consumer slows
      ↓
Queue grows
      ↓
Queue-depth / message-age metric increases
      ↓
Autoscaler adds consumers
      ↓
If consumer capacity cannot increase enough
      ↓
Producer throttling kicks in
      ↓
Producer's bounded buffer fills
      ↓
Upstream producer is eventually slowed/rejected
```

That's the complete feedback loop.

---

## 12. The architectural principle to remember

The critical question is:

> **Where should the backpressure ultimately stop?**

A well-designed distributed system should avoid uncontrolled growth at every layer.

You generally want:

```text
        ┌───────────────┐
        │    Upstream   │
        └───────┬───────┘
                │
          backpressure
                ↓
        ┌───────────────┐
        │    Producer   │
        │ bounded work  │
        └───────┬───────┘
                │
                ▼
        ┌───────────────┐
        │ Service Bus   │
        │ durable buffer│
        └───────┬───────┘
                │
                ▼
        ┌───────────────┐
        │   Consumers   │
        │ autoscaling   │
        └───────────────┘
```

The queue should absorb **temporary bursts**, while autoscaling and throttling handle **sustained imbalance**.

The producer should have a deliberate policy for what happens when the entire pipeline is saturated: wait, buffer, retry, reject, or persist durably.

