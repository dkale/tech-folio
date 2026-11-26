# Microservice Horizontal Scaling

## Table of Contents

- [Criteria / Triggers for Horizontal Scaling](#criteria--triggers-for-horizontal-scaling)
- [How It Works for Queue-Based Services (Competing Consumers)](#how-it-works-for-queue-based-services-competing-consumers)
- [Prerequisites for Horizontal Scaling](#prerequisites-for-horizontal-scaling)
- [Scaling Mechanisms](#scaling-mechanisms)
- [Key Insight](#key-insight)
- [Service Fabric Scaling Policies](#service-fabric-scaling-policies)
- [What Actually Gets Scaled?](#what-actually-gets-scaled)
- [Why CPU Isn't Always the Best Metric](#why-cpu-isnt-always-the-best-metric)
- [Service Fabric's Role vs Your Application's Role](#service-fabrics-role-vs-your-applications-role)
- [Making Queue Metrics Available to Service Fabric](#making-queue-metrics-available-to-service-fabric)
- [Scaling Configuration & Stabilization](#scaling-configuration--stabilization)
- [Stateless vs Stateful Services](#stateless-vs-stateful-services)
- [Architecture Recommendation for Event-Driven Systems](#architecture-recommendation-for-event-driven-systems)

---

A microservice scales horizontally by running **multiple instances** of the same service behind a load balancer (or competing consumers on a message queue). Each instance is stateless and handles a portion of the total load.

## Criteria / Triggers for Horizontal Scaling

| Signal | What to watch |
|--------|--------------|
| **CPU/Memory saturation** | Instance consistently above 70-80% utilization |
| **Queue depth growing** | Messages accumulating faster than `MaxConcurrentCalls × instance_count` can drain them |
| **Response latency** | P95/P99 exceeding SLA thresholds |
| **Thread pool exhaustion** | All worker threads busy; new requests queuing internally |
| **Error rate spike** | Timeouts, 429s, or lock-expiry redeliveries increasing |

## How It Works for Queue-Based Services (Competing Consumers)

```
Service Bus Queue (100 messages)
    ├─ Instance A (MaxConcurrentCalls=10) → processes 10 concurrently
    ├─ Instance B (MaxConcurrentCalls=10) → processes 10 concurrently
    └─ Instance C (MaxConcurrentCalls=10) → processes 10 concurrently
```

Service Bus delivers messages to whichever consumer requests them first. Adding instances linearly increases throughput (30 concurrent handlers instead of 10).

## Prerequisites for Horizontal Scaling

1. **Statelessness** — no in-memory state shared between requests (use external stores for state)
2. **Idempotent handlers** — duplicate delivery must be safe (PeekLock + competing consumers can redeliver)
3. **Scoped dependencies** — each message gets its own DI scope
4. **No singleton bottlenecks** — shared resources (DB connections, external APIs) must handle the multiplied load
5. **Partitioned data** — database doesn't become the single bottleneck when instances multiply

## Scaling Mechanisms

| Platform | Mechanism |
|----------|-----------|
| Kubernetes / Service Fabric | HPA (Horizontal Pod Autoscaler) based on CPU, memory, or custom metrics (queue length) |
| Azure Container Apps | KEDA scaler watching Service Bus queue depth |
| App Service | Auto-scale rules on CPU or queue metrics |

## Key Insight

Scaling out adds more competing consumers to the same queue — the `ServiceBusProcessor` in each instance independently prefetches and processes messages, and Service Bus distributes the work automatically with no coordination needed between instances.

---

## Service Fabric Scaling Policies

Service Fabric **doesn't automatically decide to scale a microservice just because CPU or memory increases**. It depends on how you configure the service's **scaling policy**.

For a Service Fabric microservice, there are essentially two concepts:

1. **Service Fabric monitors metrics**
2. **A scaling policy tells it what to do when those metrics cross thresholds**

Suppose you have a stateless `OrderService`:

```text
                Service Fabric Cluster
                         |
              +----------+----------+
              |                     |
        OrderService            OrderService
        Instance 1               Instance 2
              |
        CPU = 75%
```

You can configure an **automatic scaling policy** such as:

```text
CPU > 70% for sustained period
             ↓
     Add another instance
             ↓
OrderService instances = 3
```

And:

```text
CPU < 30% for sustained period
             ↓
     Remove an instance
             ↓
OrderService instances = 2
```

Service Fabric uses its **metrics and scaling mechanisms** to adjust the number of service instances.

---

## What Actually Gets Scaled?

For a **stateless service**, scaling normally means increasing/decreasing the number of instances.

For example:

```text
Before:

Node 1             Node 2
-------             -------
OrderService       OrderService
Instance 1         Instance 2


After scale-out:

Node 1             Node 2             Node 3
-------             -------             -------
Instance 1         Instance 2         Instance 3
```

The important distinction is:

> **Service Fabric isn't creating another microservice process because the existing process is busy. It changes the service's instance count, and the Service Fabric placement system distributes those instances across nodes.**

---

## Why CPU Isn't Always the Best Metric

This becomes particularly important in your **Azure Service Bus + microservices** architecture.

Imagine:

```text
Azure Service Bus
       |
       v
+-------------------+
| OrderService      |
| Instance 1        |
+-------------------+
       |
       v
   Processing
```

Suppose messages arrive faster than the service can process them:

```text
Producer
   |
   v
Service Bus
   |
   +--> 10,000 messages
   |
   +--> 20,000 messages
   |
   +--> 50,000 messages
```

CPU might still be only 40%.

So a CPU-based scaling rule would say:

> "Everything looks fine."

But your **queue backlog is exploding**.

That's why event-driven systems often use a **custom/application metric**, such as:

```text
Queue depth
Messages processed/sec
Message age
Processing latency
CPU
Memory
```

For example:

```text
Queue depth > 10,000
        ↓
Scale out
        ↓
1 → 2 → 4 → 8 instances
```

---

## Service Fabric's Role vs Your Application's Role

This is the key architectural distinction.

### Service Fabric is responsible for

- Running service instances
- Monitoring cluster/service health
- Placing instances on nodes
- Restarting failed instances
- Moving replicas/instances when necessary
- Applying configured scaling policies
- Balancing services across cluster nodes

### Your application is responsible for

- Processing messages
- Controlling concurrency
- Applying backpressure
- Deciding how many messages an instance processes concurrently
- Handling retries
- Handling failures/idempotency
- Exposing meaningful metrics

So the architecture might look like:

```text
                Azure Service Bus
                       |
                       v
              +----------------+
              | Queue / Topic  |
              +----------------+
                       |
              Queue-depth metric
                       |
                       v
             +-------------------+
             | Service Fabric    |
             | Autoscaling        |
             +-------------------+
                       |
             +---------+---------+
             |         |         |
             v         v         v
          Instance  Instance  Instance
             1         2         3
             |         |         |
             +---------+---------+
                       |
                  Processing
```

---

## Making Queue Metrics Available to Service Fabric

Service Fabric **doesn't inherently know that your Azure Service Bus queue is overloaded**.

You have to make that information available as a **scaling metric** and configure the appropriate scaling mechanism.

For example, you could conceptually have:

```text
Service Bus
     |
     | QueueDepth = 25,000
     v
Metric
     |
     v
Service Fabric scaling policy
     |
     v
Increase instance count
```

This is different from Kubernetes, where you might commonly use an HPA/KEDA-style approach to scale based on queue length.

---

## Scaling Configuration & Stabilization

Service Fabric doesn't normally do:

```text
CPU = 71%
   ↓
Immediately create instance
```

You generally configure:

- minimum instance count
- maximum instance count
- metric
- threshold
- scale increment/decrement
- stabilization behavior

Conceptually:

```text
Minimum instances = 2
Maximum instances = 20

CPU > 70%
    → +1 instance

CPU < 30%
    → -1 instance
```

This prevents the service from constantly oscillating:

```text
2 → 3 → 2 → 3 → 2 → 3
```

---

## Stateless vs Stateful Services

This distinction matters a lot.

### Stateless

Autoscaling is relatively straightforward:

```text
Instances:

1 → 2 → 4 → 8
```

Each instance can process requests/messages independently.

### Stateful

You don't simply create arbitrary copies.

For example:

```text
Partition 1
Partition 2
Partition 3
```

Each partition has replicas:

```text
Partition 1
  Primary
  Secondary
  Secondary
```

Scaling involves **partitions/replicas**, and data distribution and consistency become important.

Therefore, when you're talking about **Service Bus consumers**, you're most likely dealing with **stateless Service Fabric services**, where scaling out the instance count is much simpler.

---

## Architecture Recommendation for Event-Driven Systems

Given your **Service Fabric + Azure Service Bus + event-driven microservices** architecture, I'd think about autoscaling like this:

```text
                    Producer
                       |
                       v
               Azure Service Bus
                       |
                Queue / Topic
                       |
              +--------+--------+
              |                 |
        Queue depth        Message age
              |                 |
              +--------+--------+
                       |
                 Scaling metric
                       |
                       v
              Service Fabric
                       |
          +------------+------------+
          |            |            |
       Instance 1   Instance 2   Instance 3
          |            |            |
          +------------+------------+
                       |
                  DB / APIs
```

**CPU/memory tells you how busy the instances are.**

**Queue depth/message age tells you whether the system is keeping up with incoming work.**

For an event-driven microservice, **queue backlog and message age are often much better autoscaling signals than CPU alone**.

And this connects directly to **producer backpressure**: autoscaling can absorb bursts by adding consumers, but if the downstream database/API is the bottleneck, blindly scaling from 2 → 20 consumers can actually make the system *worse*.
