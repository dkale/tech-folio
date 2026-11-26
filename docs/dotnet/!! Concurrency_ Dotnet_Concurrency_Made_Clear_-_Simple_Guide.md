# .NET Concurrency Made Clear: A Simple Guide

> A plain-language companion to [.NET concurrency, made clear](https://concurrency.liorsherbaty.com/).
>
> The examples use modern C# and .NET concepts. They are intentionally small so the important behavior is visible.

## Table of Contents

- [1. The Big Picture](#1-the-big-picture)
  - [Three tools, three jobs](#three-tools-three-jobs)
  - [A running example](#a-running-example)
- [2. Tasks: Wait for Work to Finish](#2-tasks-wait-for-work-to-finish)
  - [A task is a promise](#a-task-is-a-promise)
  - [What `await` really does](#what-await-really-does)
  - [`Task.Run` is not a synonym for async](#taskrun-is-not-a-synonym-for-async)
  - [Task completion sources](#task-completion-sources)
  - [When to use `ValueTask`](#when-to-use-valuetask)
  - [Failures and `Task.WhenAll`](#failures-and-taskwhenall)
  - [Cancellation is a request](#cancellation-is-a-request)
  - [Threads, contexts, and blocking](#threads-contexts-and-blocking)
  - [Background work needs an owner](#background-work-needs-an-owner)
- [3. Channels: Move Work Through a Queue](#3-channels-move-work-through-a-queue)
  - [A channel is an in-process async queue](#a-channel-is-an-in-process-async-queue)
  - [Give producers and consumers only what they need](#give-producers-and-consumers-only-what-they-need)
  - [Bound the queue and choose an overload policy](#bound-the-queue-and-choose-an-overload-policy)
  - [Backpressure must reach the producer](#backpressure-must-reach-the-producer)
  - [Competing consumers are not broadcast](#competing-consumers-are-not-broadcast)
  - [Completion is not the same as worker shutdown](#completion-is-not-the-same-as-worker-shutdown)
- [4. Observables: React to Event Streams](#4-observables-react-to-event-streams)
  - [The observer contract](#the-observer-contract)
  - [Prefer established factories](#prefer-established-factories)
  - [Choose operators by meaning](#choose-operators-by-meaning)
  - [Serialization and scheduling are different](#serialization-and-scheduling-are-different)
  - [Hot and cold observables](#hot-and-cold-observables)
  - [Errors end a subscription](#errors-end-a-subscription)
- [5. Reliability: Say What the System Guarantees](#5-reliability-say-what-the-system-guarantees)
  - [Accepted, processed, and durable are different](#accepted-processed-and-durable-are-different)
  - [Recover selectively](#recover-selectively)
  - [Acknowledgments need precise meaning](#acknowledgments-need-precise-meaning)
  - [Dead-letter queues are workflows](#dead-letter-queues-are-workflows)
  - [Retry only what is safe to retry](#retry-only-what-is-safe-to-retry)
  - [Shut down in dependency order](#shut-down-in-dependency-order)
- [6. Putting the Pieces Together](#6-putting-the-pieces-together)
  - [Observable to channel](#observable-to-channel)
  - [Fan-out, fan-in, and ordering](#fan-out-fan-in-and-ordering)
  - [Batching and capacity](#batching-and-capacity)
  - [A supervised worker pool](#a-supervised-worker-pool)
- [7. Before Shipping](#7-before-shipping)
- [8. Quick Decision Guide](#8-quick-decision-guide)

## 1. The Big Picture

Concurrency is not just about making several things run at once. It is about deciding:

- how work starts;
- how callers wait for it;
- how much work may be pending;
- what happens when the system is full;
- how failures travel;
- and what the system can honestly promise after a crash or timeout.

### Three tools, three jobs

| Tool | Simple meaning | Good fit |
|---|---|---|
| `Task` / `Task<T>` | One operation that will finish later | Calling an API, saving one record, calculating one result |
| `Channel<T>` | A queue between producers and consumers | Worker pools, pipelines, burst smoothing, bounded work |
| `IObservable<T>` | A stream that pushes notifications to subscribers | UI events, timers, market ticks, reactive composition |

A useful mental picture is a restaurant:

- A **task** is your order receipt. It represents one promised outcome.
- A **channel** is the kitchen ticket rail. Orders wait there, and its size can be limited.
- An **observable** is the stream of events from the dining room: new orders, cancellations, and status updates.

These tools can be combined, but they do not provide the same guarantee. A successful enqueue is not proof that work finished, and a notification is not proof that a receiver stored it.

### A running example

Imagine an application receiving sensor readings:

1. A device produces readings.
2. The application admits readings into a bounded queue.
3. Several workers store them.
4. A dashboard receives live updates.
5. Failed readings are retried or sent to a recovery workflow.

The device input is a stream, the queue is a channel, and each store operation is a task. Most production concurrency problems happen at the boundaries between those pieces.

## 2. Tasks: Wait for Work to Finish

### A task is a promise

`Task` represents eventual completion without a result. `Task<T>` represents eventual completion with a result.

```csharp
public async Task SaveReadingAsync(Reading reading, CancellationToken ct)
{
    await repository.SaveAsync(reading, ct);
}

public async Task<Reading> LoadReadingAsync(int id, CancellationToken ct)
{
    return await repository.LoadAsync(id, ct);
}
```

A task does not necessarily own a thread. For I/O, the program can start a network request, return the thread to the pool, and resume later when the operating system reports progress.

**Analogy:** A dry-cleaning ticket is not a person standing beside the washing machine. It is a claim that the clothes will eventually be ready, together with a way to check or wait for that result.

Do not build control flow around repeatedly checking `Task.Status`. Await the task and let completion, failure, or cancellation drive the next step.

### What `await` really does

An `async` method starts running synchronously. When it reaches an awaitable that is already complete, it continues immediately. When the operation is incomplete, the method stores enough state to resume later and returns its incomplete task to the caller.

```csharp
public static async Task<string> GetTextAsync(
    HttpClient client,
    Uri uri,
    CancellationToken ct)
{
    using HttpResponseMessage response =
        await client.GetAsync(uri, ct);

    response.EnsureSuccessStatusCode();
    return await response.Content.ReadAsStringAsync(ct);
}
```

The compiler turns the method into a resumable state machine. This sounds elaborate, but the practical rule is simple: keep the path asynchronous and await the native async API.

Reuse a long-lived `HttpClient` or one supplied by `IHttpClientFactory`. The method above owns the response it creates, but it does not own the client passed into it.

### `Task.Run` is not a synonym for async

`Task.Run` explicitly queues work to the ThreadPool. It is useful when you deliberately want to move CPU-heavy work away from the current caller:

```csharp
int result = await Task.Run(
    () => ExpensiveCpuCalculation(input),
    ct);
```

It is usually not useful for wrapping an operation that already has a proper async API:

```csharp
// Usually unnecessary for I/O:
await Task.Run(() => httpClient.GetStringAsync(uri), ct);
```

**Analogy:** Async I/O is like leaving your phone number with a repair shop and doing something else. `Task.Run` is like assigning another employee to do a calculation. Hiring an employee to wait beside the phone does not make the repair faster.

### Task completion sources

`TaskCompletionSource<T>` is useful when the thing that signals completion is not already task-based, such as a callback, event, or one-time notification.

```csharp
public sealed class AsyncSignal
{
    private readonly TaskCompletionSource<bool> completion = new(
        TaskCreationOptions.RunContinuationsAsynchronously);

    public Task WaitAsync(CancellationToken ct = default) =>
        completion.Task.WaitAsync(ct);

    public bool Signal() => completion.TrySetResult(true);
}
```

Important details:

- `TrySetResult` handles competing signal attempts without throwing.
- `RunContinuationsAsynchronously` avoids running a waiting continuation inline inside the code that signals completion.
- Canceling one wait does not cancel the shared signal.
- A canceled wait does not automatically stop the underlying callback or operation.

Treat ownership as part of the adapter design. Decide who unregisters callbacks, disposes cancellation registrations, and stops the underlying operation.

### When to use `ValueTask`

`ValueTask<T>` can represent a result that is already available, a normal `Task<T>`, or another reusable async source. It can reduce allocations on a measured hot path, but it has stricter consumption rules.

Practical rules:

- Usually consume each returned value once with a direct `await`.
- If the result must be shared or passed to `Task.WhenAll`, convert it to a task once with `AsTask()`.
- Do not block on an incomplete `ValueTask`.
- Do not mix multiple consumption styles for the same instance.

**Analogy:** A normal task is a reusable receipt. A `ValueTask` is more like a single-use claim ticket that might contain the answer directly or might point somewhere else. It can be cheaper, but you must follow its handling rules.

Use it because measurement shows it helps, not because it is a struct.

### Failures and `Task.WhenAll`

Exceptions from an `async Task` method are stored in its task and normally rethrown when you await it. Avoid `async void` except for event handlers, because an `async void` failure has no task for normal supervision.

`Task.WhenAll` waits for every task. Awaiting it reports failure, but it does not cancel sibling tasks automatically.

```csharp
Task[] saves = readings.Select(
    reading => SaveAsync(reading, ct)).ToArray();

Task all = Task.WhenAll(saves);
try
{
    await all;
}
catch
{
    if (all.Exception is { } aggregate)
    {
        foreach (Exception error in aggregate.Flatten().InnerExceptions)
            logger.LogError(error, "A save failed");
    }

    throw;
}
```

**Analogy:** `WhenAll` is a manager who waits for every contractor to report. If one contractor fails, the manager still waits for the others. The manager does not fire the remaining contractors unless you add that policy yourself.

Report failures at the boundary that owns the operation. Otherwise, the same exception may be logged repeatedly by several layers.

### Cancellation is a request

A cancellation token is a request to stop, not a time machine and not a rollback mechanism.

```csharp
public async Task ProcessAsync(
    IReadOnlyList<Reading> readings,
    CancellationToken ct)
{
    foreach (Reading reading in readings)
    {
        ct.ThrowIfCancellationRequested();
        await ProcessOneAsync(reading, ct);
    }
}
```

Combine caller cancellation with an operation deadline:

```csharp
using CancellationTokenSource deadline =
    CancellationTokenSource.CreateLinkedTokenSource(ct);
deadline.CancelAfter(TimeSpan.FromSeconds(10));

string text = await client.GetStringAsync(uri, deadline.Token);
```

A timeout on the wait is different from cancellation of the operation:

```csharp
await operation.WaitAsync(TimeSpan.FromSeconds(5));
```

`WaitAsync` can stop the caller from waiting while `operation` continues in the background. Pass the token into the actual operation when it must be asked to stop, and retain an owner for work that may outlive the timeout.

An operation may also have committed a side effect before cancellation was noticed. That is why retries often need idempotency keys.

### Threads, contexts, and blocking

`await` does not automatically mean “switch to another thread.” In an application with a synchronization context, a continuation may return to that context. UI code often needs this behavior so it can update controls safely. ASP.NET Core normally does not install a custom synchronization context.

In reusable library code that does not depend on the caller’s context, `ConfigureAwait(false)` can avoid capturing it:

```csharp
await dependency.CallAsync(ct).ConfigureAwait(false);
```

It does not force a ThreadPool hop and does not suppress all execution context.

Avoid `.Result` and `.Wait()` in async code. They can deadlock in some environments and can consume ThreadPool threads while waiting. Wrapping blocking work in `Task.Run` does not make the blocking disappear.

### Background work needs an owner

A detached task is not supervised merely because it writes a log message when it finishes.

An owner must:

- retain the task;
- observe its exceptions;
- keep required dependencies alive;
- define cancellation behavior;
- and await it during shutdown.

Choose an ownership model:

- Request-critical work: await it before returning success.
- Application background work: run it under a hosted or supervised service.
- Work that must survive restart: persist it in a durable queue or outbox before acknowledging acceptance.

Do not capture request-scoped services in untracked background work.

## 3. Channels: Move Work Through a Queue

### A channel is an in-process async queue

`Channel<T>` separates admission from processing. Writers add items; readers remove them. It is useful for smoothing bursts and limiting the number of workers.

```csharp
Channel<Reading> queue = Channel.CreateBounded<Reading>(
    new BoundedChannelOptions(256)
    {
        FullMode = BoundedChannelFullMode.Wait,
        SingleWriter = true,
        SingleReader = false,
        AllowSynchronousContinuations = false
    });
```

A channel lives in process memory. It is not crash recovery. If the process disappears, buffered items disappear too.

Compared with `BlockingCollection<T>`, channels are designed around asynchronous waiting and integrate naturally with `await foreach`.

### Give producers and consumers only what they need

Pass `ChannelWriter<T>` to producers and `ChannelReader<T>` to consumers. This makes accidental misuse harder and clarifies ownership.

```csharp
async Task ProduceAsync(
    IAsyncEnumerable<Reading> source,
    ChannelWriter<Reading> writer,
    CancellationToken ct)
{
    await foreach (Reading reading in source.WithCancellation(ct))
        await writer.WriteAsync(reading, ct);
}
```

Choose one owner for `TryComplete()`. Usually the component that knows all producers are finished closes admission.

Useful APIs:

- `WriteAsync`: wait asynchronously until the item is accepted or the operation fails.
- `TryWrite`: attempt an immediate write.
- `WaitToWriteAsync`: wait for an opportunity to write; it does not reserve a slot.
- `ReadAllAsync`: read until the channel completes and its buffered items are drained.

### Bound the queue and choose an overload policy

A bounded queue forces you to make an explicit decision when capacity is exhausted. With three slots containing `A`, `B`, and `C`, what should happen to incoming `D`?

| Full mode | Result | Meaning |
|---|---|---|
| `Wait` | `D` waits | Slow the producer until space exists |
| `DropOldest` | `A` is discarded; queue becomes `B`, `C`, `D` | Keep newer data |
| `DropNewest` | `C` is discarded; queue becomes `A`, `B`, `D` | Preserve older buffered data |
| `DropWrite` | `D` is discarded | Reject the incoming item |

**Analogy:** A small clinic waiting room needs a policy when every chair is occupied. Patients can wait outside, the oldest person can be removed, the newest person can be turned away, or the new arrival can be refused. Each choice is a product decision.

Capacity limits buffered items, not total memory. Active workers, payload size, pending writes, and downstream buffers count too.

A drop-mode `TryWrite` may return `true` even though the item was discarded according to the mode. Count drops using the available `itemDropped` callback and keep that callback fast and non-throwing.

### Backpressure must reach the producer

Backpressure means the system’s slowness reaches the source of new work. Awaiting each write does that:

```csharp
await foreach (Reading reading in source.WithCancellation(ct))
    await writer.WriteAsync(reading, ct);
```

This pattern does not:

```csharp
// The queue may look bounded, but pending tasks become the new backlog.
foreach (Reading reading in readings)
    _ = writer.WriteAsync(reading, ct);
```

Starting one task per input item merely moves the queue from the channel into the collection of incomplete tasks.

**Analogy:** A traffic light only controls traffic if cars can see it. If you let every car enter a hidden side road, the main road looks empty while the system is still overloaded.

### Competing consumers are not broadcast

Several readers on one channel compete for work. Each item goes to one reader, not all readers.

Use:

- one channel and multiple readers for a worker pool;
- one channel per destination for broadcast;
- a channel between stages for a pipeline.

With multiple workers, dequeue order may be FIFO while completion order is not. If all events for one customer or sensor must remain ordered, partition by key and process each partition sequentially.

Multiple writers merge input, but concurrent producers do not create a useful global source order unless you add one.

### Completion is not the same as worker shutdown

`TryComplete()` closes admission. Readers can still drain already-buffered items.

`reader.Completion` means that no more items will arrive after the queue drains. It does not necessarily mean that a worker has finished processing the last item it removed.

The owner should await the worker tasks:

```csharp
Task[] workers = Enumerable.Range(0, workerCount)
    .Select(_ => ConsumeAsync(queue.Reader, ct))
    .ToArray();

await Task.WhenAll(workers);
```

If a consumer fails, blocked producers need to be canceled or released. `Task.WhenAll` alone does not solve that dependency.

Measure the real workload: end-to-end latency, queue age, producer wait time, allocations, throughput, and behavior under overload. `SingleReader` and `SingleWriter` describe your usage contract; they are not runtime locks.

## 4. Observables: React to Event Streams

### The observer contract

`IObservable<T>` pushes values to an `IObserver<T>`. For each subscription, the normal lifecycle is:

```text
zero or more OnNext(value)
then exactly one OnCompleted() or OnError(error)
```

No values should arrive after termination. Notifications for one subscription are serialized even if the source uses multiple threads.

Disposing a subscription requests unsubscription. It does not undo a side effect that already happened and does not necessarily wait for an in-flight callback to finish.

**Analogy:** A newspaper subscription delivers issues until it ends. Canceling the subscription does not erase newspapers already delivered or necessarily interrupt the delivery person currently at your door.

Rx.NET adds operators through the `System.Reactive` package. The two interfaces themselves are part of the base framework.

### Prefer established factories

Use existing factories for common sources:

- `Observable.Defer` for work created per subscription;
- `Observable.FromAsync` for task-based operations;
- `Observable.FromEventPattern` for .NET events.

```csharp
public static IObservable<string> FetchOnSubscribe(
    HttpClient client,
    Uri uri) =>
    Observable.Defer(() => Observable.FromAsync(
        ct => client.GetStringAsync(uri, ct)));
```

With this definition, each subscriber starts its own operation. Disposal can request cancellation through the token, but cancellation remains cooperative.

A handwritten observable must coordinate subscription, callbacks, serialization, disposal, termination, and reentrancy. Use one only when the existing factories do not express the source correctly.

For asynchronous pull with natural producer pacing, `IAsyncEnumerable<T>` may be a better fit than creating a small custom Rx system.

### Choose operators by meaning

| Operator | What it does | Important consequence |
|---|---|---|
| `Where` | Keeps values matching a predicate | Predicate errors normally terminate the stream |
| `Select` | Transforms each value | Selector errors normally terminate the stream |
| `Buffer` | Groups values by count or time | Adds memory and latency |
| `Throttle` | Emits after a quiet period | A constantly busy source may suppress values |
| `Sample` | Emits the latest value at sampling times | Intermediate values are intentionally omitted |

Do not use `Throttle` or `Sample` when every event represents a required business action. A UI search box may debounce keystrokes; an invoice processor must not debounce invoices away.

A leading-edge rate limiter and Rx.NET `Throttle` are different policies even if both reduce event frequency.

### Serialization and scheduling are different

These concepts solve different problems:

- Serialization prevents concurrent notifications from corrupting an observer’s assumptions.
- `ObserveOn` chooses where downstream callbacks run.
- `SubscribeOn` affects where subscription work starts.

Moving notifications to another scheduler may create a queue, but it does not provide bounded backpressure. A lock may exclude simultaneous threads while still permitting reentrant calls on the same thread.

Avoid action-based subscriptions with async lambdas:

```csharp
// The async lambda becomes async void.
observable.Subscribe(async value => await SaveAsync(value));
```

Failures and lifetime are then outside normal task supervision. Compose task-backed observables or put the work into an explicitly managed channel and worker pool.

### Hot and cold observables

A **cold** observable usually starts its work separately for each subscriber. A **hot** observable exists independently; late subscribers generally miss earlier values unless replay is configured.

**Analogy:** A cold observable is a personal tour that starts when each guest arrives. A hot observable is a live concert; a guest who enters late misses the opening song.

Cold does not mean durable or infallible. It can fail, be canceled, or read changing external data. An in-memory replay buffer disappears when the process stops.

### Errors end a subscription

`OnError` ends that subscription. Operators such as `Catch` and `Retry` can switch to another sequence or resubscribe, but they do not magically resume the exact failed operation.

Resubscription can repeat side effects, so retry only when repeating is safe or when the operation is idempotent.

If an individual item may fail but the stream should continue, represent the per-item outcome as data:

```csharp
public sealed record ItemResult<T>(
    T? Value,
    Exception? Error);
```

Unexpected infrastructure failures should still reach a supervisor instead of being silently converted into successful values.

## 5. Reliability: Say What the System Guarantees

### Accepted, processed, and durable are different

These are three separate milestones:

| Milestone | What it proves | What it does not prove |
|---|---|---|
| Admitted | A queue or API accepted the item | A worker succeeded |
| Processed | A worker completed its operation | The result survived a crash |
| Durably recorded | Storage or a broker confirmed a commit | Every downstream consumer received it |

**Analogy:** A parcel being accepted at the post office does not mean it reached the recipient. Delivery confirmation does not mean the recipient filed it safely forever.

For business events that must survive restart:

1. Persist the event before acknowledging durable acceptance.
2. Give it a stable identifier.
3. Make processing idempotent so redelivery is safe.
4. Consider an outbox that records the event in the same transaction as the business change.

An in-memory channel and acknowledgment are coordination mechanisms, not durable receipts.

### Recover selectively

A per-item `try/catch` is useful only when it produces a real outcome. A recovery policy should:

1. distinguish requested cancellation from unexpected failure;
2. preserve the original payload and stable ID;
3. confirm that retry or dead-letter scheduling succeeded;
4. stop or alert when the recovery mechanism itself fails.

Do not catch every exception, log it, and report success. That converts an unknown outcome into a misleading success.

Dropping an item is acceptable only when the product explicitly allows it, and it should be counted as a named outcome.

### Acknowledgments need precise meaning

A caller may await a `TaskCompletionSource` stored in an envelope while a worker processes that envelope. The worker must settle the acknowledgment on success, failure, and cancellation. Shutdown must also settle or account for queued envelopes that no worker will process.

An in-memory acknowledgment answers “what happened in this process?” It does not prove a durable commit. A caller can time out after the side effect committed and retry, creating a duplicate.

Use an idempotency key and an atomic deduplication strategy alongside the side effect.

### Dead-letter queues are workflows

A dead-letter record should usually include:

- event ID;
- original or safely protected payload;
- schema version;
- first and most recent failure time;
- attempt count;
- useful error category.

Separate transient retry work from events requiring operator review. Cap retries, preserve attempt counts across redelivery, monitor the count and age of dead letters, and define how corrected events are replayed.

A bounded in-memory dead-letter channel with a drop policy cannot support a “never lose an event” promise. If durable recovery storage is unavailable, retain the original durable message or reject admission according to policy.

### Retry only what is safe to retry

Retry only failures classified as transient and only when repeating the operation is safe. A common policy is capped exponential backoff with jitter, cancellation support, server retry guidance, and an overall time budget.

A circuit breaker typically has these states:

```text
Closed -- repeated failures --> Open
Open -- cooldown expires --> Half-open
Half-open -- successful probe --> Closed
Half-open -- failed probe --> Open
```

It pauses calls while a dependency is unhealthy and permits limited recovery probes. A shared boolean plus a failure counter is not a complete concurrent implementation; use an established resilience library when appropriate.

Avoid stacking independent retries at every layer. One request can otherwise become many downstream attempts.

### Shut down in dependency order

A clean shutdown normally follows this sequence:

1. Stop admission: stop or unsubscribe sources and let producers settle.
2. Complete the first writer.
3. Let downstream stages drain accepted items.
4. Complete each stage output only after its workers finish writing.
5. Await final workers, including recovery writes.
6. Enforce a deadline and report unfinished work if it expires.

Use separate signals for “stop receiving” and “abort processing.” Passing an already-canceled host token through the normal drain path can abandon work immediately.

Cancellation cannot forcibly terminate code that ignores it, so durable recovery is needed for work that may remain in an uncertain state.

## 6. Putting the Pieces Together

### Observable to channel

An observable’s `OnNext` callback is synchronous. It cannot await channel capacity. This is unsafe:

```csharp
public void OnNext(Reading reading)
{
    _ = writer.WriteAsync(reading); // Pending work is untracked.
}
```

Blocking inside `OnNext` can stall the source or deadlock its scheduler. A channel cannot create backpressure if the source cannot pause or acknowledge delivery.

Possible boundary strategies are:

- use an upstream API that supports pause or acknowledgment;
- use an asynchronous pull API;
- persist at the ingress boundary;
- explicitly allow loss and count rejected items.

For disposable display updates, a deliberately lossy adapter can use `TryWrite` and increment a rejection counter when admission fails. The owner must retain and dispose the subscription, observe consumers, and complete the writer during early shutdown.

### Fan-out, fan-in, and ordering

For a broadcast to storage and a live dashboard, give each destination its own queue. Storage may require durable acceptance while the dashboard may use latest-value-wins behavior.

Define the partial failure case: what happens when storage succeeds but the dashboard queue is full? Or when the dashboard succeeds but storage fails?

For multi-stage worker pools, give each stage a coordinator. A stage coordinator completes its output only after all of its workers have finished writing. When a downstream stage fails, cancel blocked upstream writes and production.

Ordering is a product decision. If readings for one sensor must stay ordered, consistently route that sensor to one sequential partition. Bound the number of partitions; an unbounded queue per sensor only moves the memory problem.

### Batching and capacity

Start with bounded concurrency and a measured baseline. Batch when it reduces the cost of the real sink, such as database round trips.

Define:

- maximum batch size;
- maximum batch age;
- what happens to a partial batch on normal completion;
- what happens on error or forced cancellation;
- how timer-triggered and producer-triggered flushes are serialized;
- how a failed batch is retained or durably recovered.

A rough burst-buffer estimate is:

$$
\text{required buffer} \approx
\max(0, \text{arrival rate} - \text{service rate})
\times \text{burst duration}
$$

This is only a starting point. Validate it against actual payload memory and latency. No finite buffer can absorb a permanent rate mismatch.

### A supervised worker pool

The following shape captures the important ownership rules:

```csharp
using System.Threading.Channels;

static async Task RunPipelineAsync(
    Func<Reading, CancellationToken, Task> storeAsync,
    CancellationToken ct)
{
    using CancellationTokenSource abort =
        CancellationTokenSource.CreateLinkedTokenSource(ct);

    Channel<Reading> queue = Channel.CreateBounded<Reading>(
        new BoundedChannelOptions(8)
        {
            FullMode = BoundedChannelFullMode.Wait,
            SingleWriter = true,
            SingleReader = false,
            AllowSynchronousContinuations = false
        });

    async Task ProduceAsync()
    {
        try
        {
            for (int id = 1; id <= 20; id++)
            {
                abort.Token.ThrowIfCancellationRequested();
                await queue.Writer.WriteAsync(
                    new Reading(id, id * 0.5), abort.Token);
            }
        }
        finally
        {
            // The producer owns admission completion.
            queue.Writer.TryComplete();
        }
    }

    async Task ConsumeAsync()
    {
        await foreach (Reading reading in
            queue.Reader.ReadAllAsync(abort.Token))
        {
            await storeAsync(reading, abort.Token);
        }
    }

    Task producer = ProduceAsync();
    Task[] consumers = Enumerable.Range(0, 3)
        .Select(_ => ConsumeAsync())
        .ToArray();

    try
    {
        await Task.WhenAll(consumers.Append(producer));
    }
    catch
    {
        // A real implementation records the originating failure,
        // cancels peers, and rethrows it at this ownership boundary.
        abort.Cancel();
        throw;
    }
}

public sealed record Reading(int Id, double Value);
```

A production version should preserve the first non-cancellation failure, cancel sibling work when a worker fails, unblock a producer waiting on a full queue, and observe every owned task.

The key guarantee is bounded, supervised in-memory processing. It is not durable ingestion. If the process crashes, buffered readings are gone unless they were persisted elsewhere.

## 7. Before Shipping

Use this as a review checklist:

- Every background task has an owner that observes failure and awaits shutdown.
- I/O remains asynchronous.
- CPU parallelism and pending work are bounded.
- Queue capacity, full mode, and permitted loss are explicit.
- `SingleReader` and `SingleWriter` match actual usage.
- Exactly one component owns channel completion.
- Worker failures release blocked producers and peers.
- Observable subscriptions are disposed.
- Observable notifications obey the terminal-event contract.
- Acceptance, processing, and durable commit have separate meanings.
- Retry and dead-letter writes have verified outcomes.
- Retries and redelivery cannot duplicate required side effects.
- Normal shutdown drains accepted work.
- Deadline expiry reports unfinished work.
- Metrics cover rejection, backlog age, processing failures, and recovery lag.
- Tests cover overload, worker failure, cancellation, restart, and duplicate delivery.

## 8. Quick Decision Guide

### I need one result later

Use `Task<T>` and `await` it. Pass cancellation through the operation. Avoid blocking waits.

### I need to run CPU-heavy work away from the caller

Use `Task.Run` deliberately, and bound how much CPU work can run concurrently.

### I need producers and workers to exchange work

Use a bounded `Channel<T>`. Pick `Wait` or a drop policy based on business meaning, not convenience.

### I need every subscriber to receive a notification stream

Use an observable or one queue per destination. A channel with competing readers is a worker pool, not a broadcast mechanism.

### I need to survive process restart

Use durable storage, a broker, or an outbox. An in-memory task, channel, replay buffer, or acknowledgment is not enough.

### I need to handle overload

Decide whether to wait, shed work, batch, reject, or persist. Make the choice visible in metrics and documentation.

### I need reliable retries

Classify failures, make the operation idempotent, cap attempts, add jitter, and preserve a recovery path when retries are exhausted.

The durable design principle is simple: make the boundary, ownership, capacity, failure policy, and guarantee explicit. Once those are explicit, the concurrency primitives become much easier to choose and test.
