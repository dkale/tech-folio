# 50 Advanced System Design Interview Questions and Improved Answers

Source article: https://www.designgurus.io/blog/50-advanced-system-design-interview-questions

This guide is a study-ready rewrite of the questions from the source, with expanded, interview-focused answers and practical trade-off notes.

## Table of Contents

1. [Scalability and Performance](#1-scalability-and-performance)
2. [Data Storage and Management](#2-data-storage-and-management)
3. [Reliability and Availability](#3-reliability-and-availability)
4. [Security and Compliance](#4-security-and-compliance)
5. [Networking and Communication](#5-networking-and-communication)
6. [Caching and Content Delivery](#6-caching-and-content-delivery)
7. [Search and Recommendation Systems](#7-search-and-recommendation-systems)
8. [Real-Time Processing and Event Handling](#8-real-time-processing-and-event-handling)
9. [APIs and Integration](#9-apis-and-integration)
10. [Advanced Topics](#10-advanced-topics)

---

## 1. Scalability and Performance

### 1. Horizontal vs. vertical scaling: what are they, and when do you choose each?
**Answer:**
Vertical scaling increases resources on a single machine (more CPU/RAM/IO), while horizontal scaling adds more machines.
Vertical scaling is simpler and often low-latency at small scale, but it has hard hardware limits and larger blast radius.
Horizontal scaling is more fault-tolerant and can grow further, but it adds distributed-systems complexity (partitioning, coordination, consistency).
A common path is scale up first for speed, then scale out for long-term growth.

### 2. What is load balancing, and which balancing strategies are common?
**Answer:**
Load balancing distributes incoming requests across instances to avoid hotspots and improve availability.
Common strategies include Round Robin, Least Connections, Weighted Round Robin, and consistent-hash routing.
Choice depends on workload shape: stateless APIs often use round-robin/least-connections; sticky sessions and cache affinity often use hashing.
Production systems combine health checks, circuit breaking, and slow-start to avoid sending traffic to unhealthy nodes.

### 3. Explain CAP theorem and practical implications.
**Answer:**
Under a network partition, a distributed system can provide either stronger consistency or higher availability, but not both simultaneously.
CAP is most relevant during failure conditions, not normal operation.
In practice, teams choose where to sit on the consistency-availability spectrum per use case.
For example, payment ledgers usually favor consistency; social feeds usually favor availability with eventual convergence.

### 4. Latency vs throughput: how are they different?
**Answer:**
Latency is per-request response time; throughput is number of requests processed per unit time.
A system can have high throughput but poor tail latency if queues build up.
Good design optimizes both mean and tail latency (p95/p99) while preserving target throughput.
Backpressure, admission control, and right-sized concurrency are key levers.

### 5. What is backpressure and why is it necessary?
**Answer:**
Backpressure protects a system from overload by signaling producers to slow down when consumers cannot keep up.
Without it, queues grow unbounded, latency spikes, and failures cascade.
Mechanisms include bounded queues, rate limiting, consumer lag thresholds, and drop/shed policies.
A resilient design defines what to reject early and what to retry safely.

## 2. Data Storage and Management

### 6. SQL vs NoSQL: differences and use cases.
**Answer:**
SQL databases provide structured schema, joins, and strong transactional guarantees, making them ideal for financial records and relational workloads.
NoSQL systems trade some relational constraints for flexible models and horizontal scaling (key-value, document, wide-column, graph).
Use SQL when integrity and multi-row transactions dominate; use NoSQL when scale, flexible schema, and access-pattern speed dominate.
Many mature architectures are polyglot: SQL for source-of-truth, NoSQL for high-volume serving.

### 7. What is sharding and how does it improve scalability?
**Answer:**
Sharding splits a large dataset across multiple database nodes so each node handles only a partition of data.
It improves write throughput and storage capacity by parallelizing load.
Shard key choice is critical: a poor key causes skew and hot shards.
Rebalancing, cross-shard queries, and transactional boundaries are the main operational challenges.

### 8. Eventual consistency vs strong consistency: when to use each?
**Answer:**
Strong consistency guarantees a read reflects the latest committed write.
Eventual consistency allows temporary divergence but converges over time.
Use strong consistency for correctness-critical paths (balances, inventory reservation), and eventual consistency for high-scale user experiences where slight staleness is acceptable.
A common hybrid is strong consistency for writes in core domain, eventual views for read-heavy projections.

### 9. What does ACID mean and why does it matter?
**Answer:**
ACID means Atomicity, Consistency, Isolation, and Durability.
Atomicity prevents partial transactions, consistency preserves invariants, isolation avoids concurrency anomalies, and durability ensures committed data survives crashes.
These properties are crucial where business correctness is non-negotiable.
Designers may reduce isolation level intentionally for performance, but only with clear anomaly awareness.

### 10. Partitioning vs replication: what roles do they play?
**Answer:**
Partitioning divides data into segments to scale capacity and parallelism.
Replication copies data across nodes for availability, fault tolerance, and read scaling.
Partitioning helps you handle more data and writes; replication helps you survive failures and serve reads faster.
Most large systems use both: partition first for scale, replicate partitions for resilience.

## 3. Reliability and Availability

### 11. What is high availability and how is it achieved?
**Answer:**
High availability (HA) is the ability to keep service uptime near target SLO (for example, 99.9%+).
It is achieved through redundancy, failover automation, health checks, multi-AZ/region deployment, and safe rollouts.
HA must include operational controls: incident response, observability, and error budgets.
Without fast detection and recovery, infrastructure redundancy alone is insufficient.

### 12. Fault tolerance vs high availability: what is the difference?
**Answer:**
High availability focuses on minimizing downtime from user perspective.
Fault tolerance focuses on continuing correct operation despite component failures.
A system can be available but degraded, or fault-tolerant for some failures but not others.
Good architecture defines explicit failure modes and graceful degradation behavior.

### 13. Why is redundancy important?
**Answer:**
Redundancy removes single points of failure by having backup capacity (compute, storage, network, dependencies).
It improves resilience during hardware failures, software bugs, and maintenance events.
However, redundancy increases cost and operational complexity.
Design should prioritize redundancy at critical layers and validate it through chaos/failover tests.

### 14. What are failover strategies?
**Answer:**
Failover strategies define how traffic moves to healthy resources when primaries fail.
Common models are active-passive (warm/cold standby) and active-active (all sites serving).
Effective failover requires health-check tuning, state replication, and DNS/LB convergence planning.
Runbooks and automated drills are as important as architecture diagrams.

### 15. Active-active vs active-passive architectures.
**Answer:**
Active-active serves traffic from multiple live sites and offers better utilization and lower RTO, but needs stronger conflict handling and routing logic.
Active-passive is simpler and easier for strict consistency workloads, but failover can be slower and standby capacity is underutilized.
Choose active-active when uptime and global performance are top priorities.
Choose active-passive when simplicity and controlled failover are more valuable.

## 4. Security and Compliance

### 16. What is OAuth 2.0 and how does it secure authorization?
**Answer:**
OAuth 2.0 enables delegated authorization: users grant scoped access to clients without sharing credentials.
It separates identity from access delegation through token-based flows.
Use short-lived access tokens, refresh-token rotation, PKCE for public clients, and scope minimization.
OAuth is an authorization framework; pair with OpenID Connect when authentication identity claims are needed.

### 17. What is the principle of least privilege?
**Answer:**
Least privilege grants only the minimum permissions needed for a user or service to perform its task.
It limits blast radius if credentials are stolen or misuse occurs.
Apply it through role-based access, scoped tokens, just-in-time elevation, and periodic access reviews.
Defense-in-depth assumes compromise and narrows what compromised actors can do.

### 18. Encryption at rest vs in transit.
**Answer:**
Encryption at rest protects stored data (disks, snapshots, backups).
Encryption in transit protects data moving over networks (TLS).
You need both because attackers target both storage compromise and network interception.
Strong key management (KMS/HSM, rotation, least-privileged key access) is as important as encryption itself.

### 19. Common web vulnerabilities and mitigations.
**Answer:**
Typical risks include injection, XSS, CSRF, SSRF, broken auth, insecure deserialization, and misconfiguration.
Mitigate using parameterized queries, output encoding, CSRF tokens, strict auth/session controls, input validation, and secure defaults.
Add WAF, dependency patching, secret scanning, and continuous security testing.
Security should be built into CI/CD, not deferred to release time.

### 20. What does a firewall do?
**Answer:**
A firewall enforces network traffic policy by allowing or blocking flows based on rules.
It reduces attack surface by limiting exposure paths.
In modern cloud systems, this combines security groups, network ACLs, service mesh policy, and egress controls.
Firewalls are necessary but not sufficient; they complement identity, encryption, and application-layer security.

## 5. Networking and Communication

### 21. What is a CDN and how does it improve performance?
**Answer:**
A CDN places content near users at edge locations to reduce round-trip latency.
It offloads origin traffic and improves availability under spikes.
For static assets, cache hit ratio is key; for dynamic content, edge compute and smart routing can still reduce latency.
CDN strategy should include cache keys, TTL, purge approach, and origin shielding.

### 22. Synchronous vs asynchronous service communication.
**Answer:**
Synchronous calls wait for immediate response and are easy to reason about but tightly couple latency and availability.
Asynchronous messaging decouples producers/consumers, improves resilience, and smooths bursts.
The trade-off is higher eventual consistency and operational complexity.
Most scalable systems mix both: sync for user-critical reads/writes, async for side effects and long-running work.

### 23. What is an API gateway in microservices?
**Answer:**
An API gateway is the front door for client traffic into a microservice ecosystem.
It centralizes routing, auth, rate limiting, protocol translation, and observability.
It can also aggregate backend responses to reduce client chattiness.
Keep business logic out of the gateway to avoid turning it into a monolith bottleneck.

### 24. What is gRPC and how is it different from REST?
**Answer:**
gRPC uses HTTP/2 and Protocol Buffers for strongly typed, efficient RPC communication.
It excels in low-latency internal service-to-service calls and streaming.
REST is simpler, more human-readable, and broadly interoperable over HTTP/JSON.
A common approach is REST/GraphQL for external APIs and gRPC internally.

### 25. What is service discovery?
**Answer:**
Service discovery lets services find each other dynamically as instances scale up/down.
It avoids hardcoded endpoints and supports elasticity.
Patterns include client-side discovery with registries and server-side discovery via load balancers.
Health checks and fast deregistration are critical to avoid routing to dead instances.

## 6. Caching and Content Delivery

### 26. What is caching and why is it effective?
**Answer:**
Caching stores frequently requested data in a faster layer (typically memory) to reduce latency and backend load.
It improves read performance and can stabilize systems under peak demand.
Cache design should define ownership, invalidation strategy, and staleness tolerance.
Poor invalidation or key design can create correctness bugs and uneven performance.

### 27. Client-side vs server-side caching.
**Answer:**
Client-side caching stores responses near the user (browser/mobile/device), reducing repeated network calls.
Server-side caching stores data in infrastructure shared by many users (Redis, Memcached, reverse proxies).
Client caches reduce bandwidth and improve UX; server caches protect databases and improve backend throughput.
Use both with coherent expiration and validation semantics.

### 28. What are cache eviction policies?
**Answer:**
Eviction policy decides which items to remove when cache capacity is full.
Common policies: LRU (least recently used), LFU (least frequently used), FIFO, TTL-based expiration, and hybrids.
Best policy depends on access pattern characteristics (recency vs frequency vs volatility).
Measure hit ratio, byte hit ratio, and tail latency before and after policy changes.

### 29. What is a cache miss and how do you reduce misses?
**Answer:**
A cache miss occurs when requested data is absent from cache, forcing a slower backend read.
Reduce misses with better key design, right-sized TTLs, prewarming, and admission policies.
Prevent stampedes via request coalescing, stale-while-revalidate, jittered expirations, and single-flight locking.
Always monitor miss penalty, not just miss rate.

### 30. CDN for static and dynamic content.
**Answer:**
For static assets, CDNs cache aggressively with long TTL and versioned asset names.
For dynamic responses, CDN can use short TTL, signed caching rules, edge logic, and personalized cache keys.
Dynamic acceleration can still reduce origin hops and handshake overhead.
The challenge is balancing freshness, personalization, and cache efficiency.

## 7. Search and Recommendation Systems

### 31. What is indexing in search and why does it matter?
**Answer:**
Indexing transforms raw documents into structures optimized for fast lookup.
Without indexing, query-time scanning becomes too expensive at scale.
Good indexing choices (tokenization, stemming, fields, analyzers) directly impact latency and relevance.
Index maintenance strategy matters as much as initial build quality.

### 32. What is an inverted index?
**Answer:**
An inverted index maps terms to document IDs (and often positions/frequencies).
It enables fast term-to-document retrieval for full-text search.
Enhancements like skip lists, compression, and positional indexes improve speed and phrase matching.
This is the core primitive behind most search engines.

### 33. What is relevance ranking?
**Answer:**
Relevance ranking orders results by likely usefulness for the query intent.
Signals include textual match scores (TF-IDF/BM25), popularity, freshness, personalization, and behavioral features.
Ranking quality is evaluated with metrics like NDCG, MRR, and click-through.
In interviews, explain both ranking features and online evaluation strategy.

### 34. Collaborative vs content-based filtering.
**Answer:**
Collaborative filtering recommends based on user-item interaction patterns across similar users/items.
Content-based filtering recommends based on item attributes similar to what a user already likes.
Collaborative methods can discover serendipity but suffer in sparse data.
Content-based methods are robust early but can become narrow; hybrids usually perform best.

### 35. What is the cold start problem and how can it be solved?
**Answer:**
Cold start happens when new users or new items have insufficient interaction history.
Mitigations include onboarding preference capture, popularity priors, metadata-based models, and exploration policies.
Contextual bandits and active learning can accelerate signal collection.
The trade-off is short-term recommendation precision versus long-term personalization quality.

## 8. Real-Time Processing and Event Handling

### 36. Stream processing vs batch processing.
**Answer:**
Stream processing computes continuously on incoming events with low latency.
Batch processing computes on bounded datasets at scheduled intervals.
Streams are ideal for alerts, fraud checks, and live dashboards; batch is efficient for large historical recomputations.
Many systems use lambda/kappa-like patterns combining real-time and batch correction.

### 37. What is event sourcing?
**Answer:**
Event sourcing stores state changes as immutable events rather than only current state.
Current state is reconstructed by replaying events, enabling auditability and temporal debugging.
It pairs well with CQRS and append-only logs.
Challenges include schema evolution, replay cost, and idempotent consumers.

### 38. Role of a message broker in event-driven systems.
**Answer:**
A broker decouples producers and consumers through durable topics/queues.
It supports buffering, retries, ordering guarantees, fan-out, and backpressure handling.
Brokers improve resilience but require partition/consumer-group design discipline.
Exactly-once business effects still require end-to-end idempotency strategy.

### 39. What is windowing in stream processing?
**Answer:**
Windowing groups unbounded streams into finite segments for aggregation.
Common windows: tumbling, sliding, session, and count-based.
Correctness depends on event time, watermarking, and late-arrival policy.
Interviewers look for how you handle out-of-order and delayed events.

### 40. What does exactly-once processing mean?
**Answer:**
Exactly-once means each event affects business state as if processed one time, even with retries/failures.
In practice, this is achieved via idempotent writes, deduplication keys, transactional sinks, and careful offset/state coordination.
Infrastructure-level exactly-once claims alone are not enough.
Design around exactly-once outcomes, not just exactly-once delivery semantics.

## 9. APIs and Integration

### 41. What is RESTful API design and its core principles?
**Answer:**
RESTful design models resources with predictable URLs and standard HTTP verbs.
It emphasizes stateless requests, cacheability, layered architecture, and clear status codes.
Well-designed REST APIs include pagination, filtering, idempotency, and consistent error contracts.
Developer experience improves with good naming, docs, and version policy.

### 42. REST vs GraphQL.
**Answer:**
REST exposes multiple resource endpoints with server-defined response shapes.
GraphQL exposes a schema and lets clients request exactly needed fields.
GraphQL can reduce over-fetching but introduces complexity in caching, auth, and query cost control.
Choice depends on client diversity, aggregation needs, and operational maturity.

### 43. What is API versioning and why is it important?
**Answer:**
Versioning allows API evolution without breaking existing consumers.
Strategies include URI versioning, header/content negotiation, and schema-level evolution.
Backward compatibility discipline reduces forced client migrations.
Pair versioning with deprecation policy, changelogs, and sunset timelines.

### 44. What are webhooks?
**Answer:**
Webhooks are event-driven HTTP callbacks sent from one system to another when something changes.
They reduce polling and enable near real-time integrations.
Reliable webhook design includes signing, retries with backoff, idempotency keys, and dead-letter handling.
Consumers should verify authenticity and process events asynchronously.

### 45. API throttling and rate limiting.
**Answer:**
Rate limiting constrains request volume per client/time window to protect service health and fairness.
Common algorithms: token bucket, leaky bucket, fixed window, sliding window.
Good APIs return transparent limit headers and retry guidance.
Use per-tenant and global protections to prevent noisy-neighbor impact.

## 10. Advanced Topics

### 46. Microservices vs monolith.
**Answer:**
A monolith packages functionality in one deployable unit, often simpler for early-stage velocity.
Microservices split capabilities into independently deployable services with bounded contexts.
Microservices improve team autonomy and targeted scaling but add distributed complexity.
A modular monolith is often a strong transitional architecture before full decomposition.

### 47. What is containerization and what are its benefits?
**Answer:**
Containerization packages app code with dependencies into isolated, portable runtime units.
It improves environment consistency from local to production and speeds CI/CD.
Benefits include predictable deployment, resource isolation, and high density.
Risks include image sprawl, supply-chain vulnerabilities, and runtime misconfiguration.

### 48. What is serverless architecture: pros and cons?
**Answer:**
Serverless runs functions/services without server provisioning responsibilities.
Pros: rapid delivery, automatic scaling, pay-per-use pricing, lower ops burden.
Cons: cold starts, platform limits, observability/debugging complexity, and vendor lock-in.
Best fit is event-driven and bursty workloads, not every latency-sensitive stateful service.

### 49. Distributed ledger and blockchain relationship.
**Answer:**
A distributed ledger is a replicated, shared record maintained across participants.
Blockchain is one implementation where records are grouped into cryptographically linked blocks with consensus.
Not every distributed ledger needs blockchain-style public consensus.
Choose this model only when multi-party trust minimization and tamper evidence justify the cost.

### 50. Why is Kubernetes important for microservices?
**Answer:**
Kubernetes automates deployment, scaling, service discovery, and recovery for containerized services.
It standardizes operations through declarative desired state and controllers.
Key capabilities include rolling updates, autoscaling, health probes, secrets/config management, and workload scheduling.
It improves platform consistency, but requires strong observability and platform engineering practices.

---

## Interview Tip
For each answer, follow this structure in live interviews:
1. Define the concept in one sentence.
2. Explain trade-offs.
3. Give one real-world example.
4. Mention failure modes and mitigation.
