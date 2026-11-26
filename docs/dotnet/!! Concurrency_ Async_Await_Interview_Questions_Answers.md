# Async/Await Interview Questions & Answers

## Table of Contents

- [Junior (0-2 Years)](#junior-0-2-years)
  - [1. What is async and await in C#?](#1-what-is-async-and-await-in-c)
  - [2. Synchronous vs Asynchronous code](#2-synchronous-vs-asynchronous-code)
  - [3. What happens when an async method is called?](#3-what-happens-when-an-async-method-is-called)
  - [4. Task, Task\<T> and ValueTask](#4-task-taskt-and-valuetask)
  - [5. Task.Run() vs async/await](#5-taskrun-vs-asyncawait)
  - [6. await Task.Delay() vs Thread.Sleep()](#6-await-taskdelay-vs-threadsleep)
  - [7. Why avoid .Result and .Wait()?](#7-why-avoid-result-and-wait)
  - [8. How does async/await avoid blocking threads?](#8-how-does-asyncawait-avoid-blocking-threads)
  - [9. async void vs async Task vs async Task\<T>](#9-async-void-vs-async-task-vs-async-taskt)
  - [10. What happens if you don't await a Task?](#10-what-happens-if-you-dont-await-a-task)
- [Mid (2-5 Years)](#mid-2-5-years)
  - [1. Task.WhenAll vs Task.WhenAny vs Task.WhenEach](#1-taskwhenall-vs-taskwhenany-vs-taskwheneach)
  - [2. Handling exceptions in async code](#2-handling-exceptions-in-async-code)
  - [3. Implementing a timeout for an async operation](#3-implementing-a-timeout-for-an-async-operation)
  - [4. CancellationToken in async methods](#4-cancellationtoken-in-async-methods)
  - [5. Avoiding deadlocks in async code](#5-avoiding-deadlocks-in-async-code)
  - [6. IAsyncEnumerable\<T> vs IEnumerable\<T>](#6-iasyncenumerablet-vs-ienumerablet)
  - [7. CPU-bound vs I/O-bound operations](#7-cpu-bound-vs-io-bound-operations)
  - [8. Optimizing performance in an async method](#8-optimizing-performance-in-an-async-method)
  - [9. Purpose of ValueTask](#9-purpose-of-valuetask)
  - [10. How async/await works under the hood](#10-how-asyncawait-works-under-the-hood)
  - [11. What happens when you omit async on a Task-returning method?](#11-what-happens-when-you-omit-async-on-a-task-returning-method)
- [Senior (5+ Years)](#senior-5-years)
  - [1. ThreadPool and async/await](#1-threadpool-and-asyncawait)
  - [2. ConfigureAwait(true) vs ConfigureAwait(false)](#2-configureawaittrue-vs-configureawaitfalse)
  - [3. Designing a highly scalable async API](#3-designing-a-highly-scalable-async-api)
  - [4. Pitfalls of using async over sync](#4-pitfalls-of-using-async-over-sync)
  - [5. Handling backpressure in an async stream](#5-handling-backpressure-in-an-async-stream)
  - [6. async/await in ASP.NET Core request pipeline](#6-asyncawait-in-aspnet-core-request-pipeline)
  - [7. Diagnosing async performance issues](#7-diagnosing-async-performance-issues)
  - [8. IAsyncDisposable and await using](#8-iasyncdisposable-and-await-using)
  - [9. Retries with exponential backoff](#9-retries-with-exponential-backoff)
  - [10. ExecutionContext flow with async/await](#10-executioncontext-flow-with-asyncawait)

---

## Junior (0-2 Years)

### 1. What is async and await in C#?

`async` marks a method as asynchronous, enabling the use of `await` inside it. `await` suspends execution of the method until the awaited task completes, freeing the calling thread to do other work.

```csharp
public async Task<string> GetDataAsync()
{
    var result = await httpClient.GetStringAsync("https://api.example.com/data");
    return result;
}
```

### 2. Synchronous vs Asynchronous code

**Synchronous** code blocks the calling thread until the operation completes. **Asynchronous** code yields control back to the caller while waiting, allowing the thread to service other work (e.g., handle UI events or other HTTP requests).

```
Synchronous:     [Thread blocked ██████████████████] → result
Asynchronous:    [Start ██] → [thread free] → [Resume ██] → result
```

### 3. What happens when an async method is called?

It runs synchronously until it hits the first `await` on an incomplete task. At that point, it returns an incomplete `Task` to the caller. When the awaited operation finishes, the continuation resumes (on the captured `SynchronizationContext` or thread pool).

```
Caller → async Method()
           ↓ runs synchronously
         await someTask
           ↓ returns incomplete Task to caller
         [caller continues]
           ...
         [someTask completes → continuation resumes]
```

### 4. Task, Task\<T> and ValueTask

| Type | Description |
|------|-------------|
| `Task` | Represents an async operation with **no return value** |
| `Task<T>` | Represents an async operation that returns a value of type `T` |
| `ValueTask` / `ValueTask<T>` | Lightweight **struct-based** alternative that avoids heap allocation when the result is already available synchronously |

### 5. Task.Run() vs async/await

`Task.Run()` queues work onto the **thread pool** (offloads CPU-bound work). `async/await` is a **language pattern** for composing asynchronous operations — it doesn't inherently create new threads; it frees the current thread during I/O waits.

```csharp
// Task.Run — offloads CPU work to thread pool
await Task.Run(() => HeavyComputation());

// async/await — no new thread, frees current thread during I/O
await httpClient.GetStringAsync(url);
```

### 6. await Task.Delay() vs Thread.Sleep()

| | `await Task.Delay()` | `Thread.Sleep()` |
|---|---|---|
| **Blocking** | Non-blocking — frees the thread | Blocks the current thread entirely |
| **Thread usage** | Thread returned to pool | Thread held hostage |
| **Risk** | None | Thread starvation |

### 7. Why avoid .Result and .Wait()?

They **block the calling thread** synchronously, which can cause **deadlocks** (especially on UI or ASP.NET Classic contexts with a single-threaded `SynchronizationContext`). They also negate the benefits of async by holding a thread hostage.

```csharp
// ❌ DEADLOCK RISK
public string GetData()
{
    return GetDataAsync().Result; // blocks thread, captures context → deadlock
}

// ✅ CORRECT
public async Task<string> GetData()
{
    return await GetDataAsync();
}
```

### 8. How does async/await avoid blocking threads?

When `await` encounters an incomplete task, the compiler-generated **state machine** registers a continuation and returns. The thread is released back to the pool. When the I/O completes, the continuation is scheduled and execution resumes — no thread was blocked during the wait.

```
Thread 1: [Start method] → [await (incomplete)] → [return to pool]
                                                         ↓
Thread N: ← [I/O completes] → [continuation runs] → [method finishes]
```

#### Does the Same Thread Resume After `await`?

No — `await` does **not** guarantee the same thread resumes. It captures the current **SynchronizationContext** (not the thread). When the awaited task completes, the continuation is posted back to that captured context — which *may or may not* be the original thread.

| Environment | SynchronizationContext | Same thread resumes? |
|---|---|---|
| WPF / WinForms | UI `SynchronizationContext` | **Yes** — continuation is marshaled back to the UI thread |
| ASP.NET Core | **None** (no `SynchronizationContext`) | **No** — any thread pool thread can pick up the continuation |
| Console app | **None** | **No** — any thread pool thread resumes |
| ASP.NET (legacy) | `AspNetSynchronizationContext` | Same *context*, but not necessarily the same thread |

```csharp
// ASP.NET Core / Console — different thread resumes
async Task DoWork()
{
    Console.WriteLine(Thread.CurrentThread.ManagedThreadId); // e.g. Thread 5
    await Task.Delay(1000);
    Console.WriteLine(Thread.CurrentThread.ManagedThreadId); // e.g. Thread 9 — different!
}

// WPF — UI thread resumes because SynchronizationContext is tied to it
async void Button_Click(object sender, EventArgs e)
{
    // Thread 1 (UI thread)
    await Task.Delay(1000);
    // Thread 1 again — SynchronizationContext marshals back to UI thread
    label.Text = "Done"; // safe to touch UI
}
```

> **Key distinction:** `await` restores the **context**, not the **thread**. In UI apps the context *is* tied to a specific thread, so it appears the same thread resumes. In server/console apps there is no such binding. Use `ConfigureAwait(false)` to opt out of context capture entirely.

### 9. async void vs async Task vs async Task\<T>

| Signature | Exception Handling | Use Case |
|---|---|---|
| `async Task` | Exceptions propagate via the returned Task | General async methods |
| `async Task<T>` | Same, but returns a result | Async methods returning data |
| `async void` | Exceptions **crash the process** (unobserved) | **Only** for event handlers |

> **Rule:** Never use `async void` except for event handlers. It makes error handling impossible and breaks composability.

### 10. What happens if you don't await a Task?

The task runs in the background. Exceptions are **silently swallowed** (unobserved). The caller continues without waiting for completion, which can cause race conditions, resource leaks, or missed errors.

```csharp
// ❌ Fire-and-forget — exception lost
SomeAsyncMethod(); // no await

// ✅ Proper
await SomeAsyncMethod();
```

---

## Mid (2-5 Years)

### 1. Task.WhenAll vs Task.WhenAny vs Task.WhenEach

| Method | Completes When | Use Case |
|---|---|---|
| `Task.WhenAll` | **All** tasks finish | Parallel batch operations |
| `Task.WhenAny` | **First** task finishes | Timeouts, racing, fastest-wins |
| `Task.WhenEach` (.NET 9+) | Yields tasks **in completion order** as `IAsyncEnumerable<Task<T>>` | Process results as they arrive |

```csharp
// WhenAll — parallel fan-out
var results = await Task.WhenAll(task1, task2, task3);

// WhenAny — timeout pattern
var winner = await Task.WhenAny(operationTask, Task.Delay(5000));
if (winner != operationTask) throw new TimeoutException();

// WhenEach — process in completion order (.NET 9+)
await foreach (var task in Task.WhenEach(tasks))
{
    var result = await task;
    Process(result);
}
```

### 2. Handling exceptions in async code

- Wrap `await` calls in `try/catch`.
- For `Task.WhenAll`, catch the exception and inspect individual `Task.Exception` properties.
- Never use `async void` (exceptions go unobserved).

```csharp
try
{
    await Task.WhenAll(task1, task2, task3);
}
catch (Exception)
{
    // Only first exception is thrown; inspect all:
    foreach (var t in new[] { task1, task2, task3 })
        if (t.IsFaulted)
            Log(t.Exception!.InnerException!);
}
```

### 3. Implementing a timeout for an async operation

```csharp
// Approach 1: CancellationTokenSource with timeout
using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(5));
await SomeOperationAsync(cts.Token); // throws OperationCanceledException on timeout

// Approach 2: Task.WhenAny race
var operationTask = SomeOperationAsync();
var completed = await Task.WhenAny(operationTask, Task.Delay(TimeSpan.FromSeconds(5)));
if (completed != operationTask)
    throw new TimeoutException();
return await operationTask;
```

### 4. CancellationToken in async methods

A **cooperative cancellation** mechanism. The caller creates a `CancellationTokenSource`, passes its `.Token` into async methods, which periodically check for cancellation. The caller calls `cts.Cancel()` to signal.

```csharp
public async Task ProcessAsync(CancellationToken ct)
{
    foreach (var item in items)
    {
        ct.ThrowIfCancellationRequested(); // check periodically
        await SaveAsync(item, ct);         // pass to framework APIs
    }
}

// Caller
using var cts = new CancellationTokenSource();
cts.CancelAfter(TimeSpan.FromMinutes(2));
await ProcessAsync(cts.Token);
```

### 5. Avoiding deadlocks in async code

1. **Always use `await`** — never `.Result` / `.Wait()`.
2. **Use `ConfigureAwait(false)`** in library code to avoid capturing `SynchronizationContext`.
3. **Don't mix sync-over-async** in contexts with a single-threaded sync context.

```csharp
// ❌ Deadlock in UI/ASP.NET Classic
public string Get() => GetAsync().Result;

// ✅ Async all the way down
public async Task<string> Get() => await GetAsync();

// ✅ Library code — don't capture context
var data = await GetAsync().ConfigureAwait(false);
```

### 6. IAsyncEnumerable\<T> vs IEnumerable\<T>

| Feature | `IEnumerable<T>` | `IAsyncEnumerable<T>` |
|---|---|---|
| Iteration | `foreach` | `await foreach` |
| MoveNext | Blocking `MoveNext()` | Async `MoveNextAsync()` |
| Thread | Blocks during each yield | Frees thread during each yield |
| Use case | In-memory collections | Streaming from DB, API, file I/O |

```csharp
// Dapper — QueryUnbufferedAsync returns IAsyncEnumerable<T> (truly streaming)
public async IAsyncEnumerable<Order> GetOrdersAsync(
    [EnumeratorCancellation] CancellationToken ct = default)
{
    await using var conn = new SqlConnection(connString);
    await foreach (var order in conn.QueryUnbufferedAsync<Order>("SELECT * FROM Orders", ct))
        yield return order;
}

// Dapper — QueryAsync returns Task<IEnumerable<T>> (buffered, NOT streaming)
public async IAsyncEnumerable<Order> GetOrdersBufferedAsync(
    [EnumeratorCancellation] CancellationToken ct = default)
{
    await using var conn = new SqlConnection(connString);
    var orders = await conn.QueryAsync<Order>("SELECT * FROM Orders"); // all rows loaded into memory
    foreach (var order in orders)
        yield return order; // yields from an in-memory list — defeats streaming purpose
}

// EF Core — AsAsyncEnumerable() returns IAsyncEnumerable<T>
public async IAsyncEnumerable<Order> GetOrdersEfAsync(
    [EnumeratorCancellation] CancellationToken ct = default)
{
    await foreach (var order in _dbContext.Orders
        .Where(o => o.Status == "Active")
        .AsAsyncEnumerable()
        .WithCancellation(ct))
    {
        yield return order;
    }
}
```

> **Key distinction:** Dapper's `QueryAsync` buffers all rows into memory before returning — use `QueryUnbufferedAsync` for true streaming. EF Core uses `AsAsyncEnumerable()` to stream rows one at a time from the database.

### 7. CPU-bound vs I/O-bound operations

| | CPU-bound | I/O-bound |
|---|---|---|
| **Example** | Image processing, encryption | HTTP calls, DB queries, file reads |
| **Approach** | `await Task.Run(() => ...)` | `await SomeIoAsync()` directly |
| **Thread** | Needs a thread pool thread | No thread needed during wait |
| **Anti-pattern** | — | Wrapping I/O in `Task.Run` |

### 8. Optimizing performance in an async method

- **Elide async/await** when just passing through a single task (avoid state machine overhead).
- **Use `ValueTask`** for hot paths that often complete synchronously.
- **Use `ConfigureAwait(false)`** in libraries.
- **Batch with `Task.WhenAll`** instead of sequential awaits.
- **Pool buffers** with `ArrayPool<T>` — avoid allocations in tight loops.

```csharp
// Elision — no state machine overhead
public Task<int> GetCountAsync() => _repo.CountAsync(); // no async/await needed

// Batching — parallel instead of sequential
var (users, orders) = (await Task.WhenAll(GetUsersAsync(), GetOrdersAsync()));
```

### 9. Purpose of ValueTask

`ValueTask<T>` avoids **heap allocation** when the result is already available (cache hit, buffered read). Use it on high-throughput hot paths.

| | `Task<T>` | `ValueTask<T>` |
|---|---|---|
| Allocation | Always heap-allocated | Stack-allocated when synchronous |
| Await multiple times | ✅ Safe | ❌ Undefined behavior |
| Store for later | ✅ Safe | ❌ Must consume once |
| General use | ✅ Preferred | ⚠️ Only for hot paths |

### 10. How async/await works under the hood

The compiler rewrites the method into a **state machine** (`IAsyncStateMachine`). Each `await` is a state transition.

```
Source code:                      Compiled:
                                  struct <Method>d__0 : IAsyncStateMachine
async Task Method()               {
{                                     int <>1__state;
    var x = await A();                AsyncTaskMethodBuilder <>t__builder;
    var y = await B(x);               ...
    return y;                         void MoveNext()
}                                     {
                                          switch (<>1__state)
                                          {
                                              case 0: goto AfterA;
                                              case 1: goto AfterB;
                                          }
                                          // state 0: call A, register continuation
                                          // state 1: call B, register continuation
                                      }
                                  }
```

The `AsyncMethodBuilder` manages Task creation and completion. Awaiters implement `ICriticalNotifyCompletion` to register the continuation callback.

### 11. What happens when you omit async on a Task-returning method?

If you omit the async keyword on a function returning a [Task](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/async-return-types), the method runs normally and directly returns the inner `Task` object without creating a compiler state machine. This is completely valid and often more efficient if you just pass along a task from another async call. [1] 

#### What Happens Code-Wise

- No State Machine: The C# compiler does not generate extra allocation overhead to track state.
- Direct Return: The `Task` returned by an inner call is handed straight to your caller.
- Synchronous Execution Upfront: Any synchronous code at the start of your function still runs immediately on the calling thread until it hits that inner task.
- No `await` Allowed: You cannot use the `await` keyword inside this function because `async` is required to enable `await`.

#### When to Omit `async`

- Pass-Through Methods: When your method only calls another async method and returns its `Task` directly.
- Performance Tuning: To save memory allocations by avoiding unnecessary state machine generation.

---

#### What is the State Machine?
Think of the compiler's state machine as a hidden "bookmarking system" for your code.
When you use the `async` keyword, the C# compiler secretly rewrites your simple function into a complex structure. This structure acts like a project manager tracking a multi-step job.

#### How the State Machine Works

- Pausing: When code hits an `await`, the state machine takes a snapshot of where it is.
- Saving: It remembers local variables and parameters.
- Yielding: It frees up the current thread to do other work while waiting.
- Resuming: Once the background task finishes, the state machine reads the snapshot and jumps right back to where it left off. [2, 3]

#### The Problem: It Costs Resources
Building this bookmarking system requires memory and processing power. If your function doesn't actually need to pause or do extra work after the task is done, creating this state machine is a waste of resources.
By omitting `async` and just returning the `Task` directly, you skip building this system entirely. You hand the task straight to the caller and let *their* code deal with the bookmarking.
If you want, share a snippet of your function code, and I can tell you if it is safe to remove the `async` keyword or if it will change how exceptions are handled.

---

## Senior (5+ Years)

### 1. ThreadPool and async/await

The thread pool provides threads for **continuations**. When an I/O completes, the IOCP (I/O Completion Port) thread posts the continuation to the thread pool's work queue. The pool dynamically adjusts thread count (hill-climbing algorithm).

**Key insight:** async/await doesn't *create* threads — it *returns* them during waits and *borrows* them for continuations.

```
[Request Thread] → await I/O → [thread returned to pool]
                                        ↓
[IOCP notification] → [ThreadPool picks up continuation] → [resume method]
```

Monitor with `dotnet-counters`:
```bash
dotnet-counters monitor System.Runtime --counters threadpool-thread-count,threadpool-queue-length
```

### 2. ConfigureAwait(true) vs ConfigureAwait(false)

| | `ConfigureAwait(true)` (default) | `ConfigureAwait(false)` |
|---|---|---|
| Context | Resume on captured `SynchronizationContext` | Resume on any thread pool thread |
| Overhead | Context switch back to original thread | No context switch overhead |
| Deadlock risk | Higher in single-threaded contexts | Lower |

**Guidelines:**
- **Libraries:** Always use `ConfigureAwait(false)` — don't assume a context.
- **Application code (UI/Blazor):** Use default (`true`) when touching UI elements.
- **ASP.NET Core:** No `SynchronizationContext` exists — `ConfigureAwait(false)` is harmless but unnecessary.

### 3. Designing a highly scalable async API

1. **Fully async top to bottom** — no sync-over-async anywhere.
2. **`CancellationToken` on every public async method.**
3. **Stream large results** with `IAsyncEnumerable<T>`.
4. **Avoid thread pool starvation** — no `Task.Run` wrapping I/O.
5. **Connection/resource pooling** — `HttpClientFactory`, DB connection pools.
6. **Apply backpressure** — bounded `Channel<T>`, `SemaphoreSlim` for concurrency limiting.
7. **Avoid capturing large closures** in continuations.

```csharp
public async IAsyncEnumerable<OrderDto> GetOrdersAsync(
    OrderFilter filter,
    [EnumeratorCancellation] CancellationToken ct = default)
{
    await using var conn = await _pool.GetConnectionAsync(ct);
    await foreach (var order in conn.QueryStreamAsync(sql, filter, ct))
        yield return MapToDto(order);
}
```

### 4. Pitfalls of using async over sync

"Async over sync" means wrapping a **synchronous/blocking** method in an async shell — making it *look* async without actually being non-blocking.

```csharp
// ❌ Async over sync — wraps blocking call in Task.Run
public async Task<string> GetDataAsync()
{
    return await Task.Run(() => File.ReadAllText("data.txt")); // still blocks a thread pool thread
}

// ✅ Truly async — no thread blocked during I/O
public async Task<string> GetDataAsync()
{
    return await File.ReadAllTextAsync("data.txt");
}
```

**Why it's a pitfall:** A thread pool thread is still held hostage doing nothing during the blocking call. Callers assume `async` = non-blocking, but under the hood you pay for the async state machine overhead *and* a blocked thread. In server apps, this burns threads instead of freeing them — negating the whole point of async.

**The rule:** If the underlying API is synchronous, keep your wrapper synchronous too. Only use `async` when calling genuinely async APIs (I/O completion port-based). Don't slap `Task.Run` on a blocking call and call it async — that just moves the blocking to a different thread.

#### How is `File.ReadAllTextAsync` truly async?

`File.ReadAllTextAsync` is truly async because it ultimately calls the OS kernel's **overlapped I/O** (Windows) or **epoll/kqueue** (Linux/macOS) — no thread is blocked at any point during the disk read.

Here's the chain:

```
File.ReadAllTextAsync
  → StreamReader.ReadToEndAsync
    → FileStream.ReadAsync  (opened with useAsync: true / FileOptions.Asynchronous)
      → OS kernel overlapped I/O (Win32 ReadFile with OVERLAPPED struct)
        → I/O Completion Port (IOCP) callback when disk read finishes
          → ThreadPool picks up continuation
```

**What makes it different from `Task.Run(() => File.ReadAllText(...))`:**

| | `File.ReadAllTextAsync` | `Task.Run(() => File.ReadAllText(...))` |
|---|---|---|
| During disk read | **Zero threads blocked** — OS handles it | **One thread pool thread blocked**, sitting idle |
| Notification | IOCP fires when I/O completes | Thread wakes up when blocking call returns |
| Thread pool impact | No thread consumed | Thread held hostage |
| Scalability | Scales to thousands of concurrent reads | Thread pool exhausted under load |

**The critical detail:** `FileStream` must be opened with `FileOptions.Asynchronous` (which `ReadAllTextAsync` does internally). Without this flag, even `ReadAsync` falls back to blocking I/O on a thread pool thread — making it effectively "async over sync."

```csharp
// This is what .NET does internally (simplified)
var fs = new FileStream(path, FileMode.Open, FileAccess.Read, 
    FileShare.Read, bufferSize: 4096, 
    options: FileOptions.Asynchronous);  // ← this flag enables true OS-level async I/O

// With this flag: ReadAsync → OS overlapped I/O → IOCP → no thread blocked
// Without this flag: ReadAsync → ThreadPool.QueueUserWorkItem → blocking read → thread wasted
```

> **Key takeaway:** True async I/O means the **OS kernel** manages the wait, not a .NET thread. The IOCP thread pool thread only runs briefly to post the continuation — it doesn't sit idle during the I/O operation.

| Pitfall | Description |
|---|---|
| **Deadlocks** | `.Result`/`.Wait()` in sync-context-bound environments |
| **State machine allocations** | Overhead on hot paths (mitigate with `ValueTask`) |
| **Unobserved exceptions** | Fire-and-forget tasks silently swallow errors |
| **Thread pool starvation** | Blocking calls inside async methods exhaust the pool |
| **Debugging complexity** | Stack traces are fragmented across continuations |
| **Sync context capture** | Unnecessary context switches hurt throughput |

### 5. Handling backpressure in an async stream

```csharp
// Bounded Channel — producer naturally throttled when full
var channel = Channel.CreateBounded<Event>(new BoundedChannelOptions(1000)
{
    FullMode = BoundedChannelFullMode.Wait // producer awaits when full
});

// Producer
await channel.Writer.WriteAsync(item, ct); // blocks when buffer full

// Consumer
await foreach (var item in channel.Reader.ReadAllAsync(ct))
    await ProcessAsync(item, ct);
```

**Other approaches:**
- **`SemaphoreSlim`** — limit concurrent operations.
- **Token-bucket / leaky-bucket** rate limiter.
- **`IAsyncEnumerable`** — inherently pull-based (consumer controls pace).

### 6. async/await in ASP.NET Core request pipeline

ASP.NET Core has **no** `SynchronizationContext` — continuations run on any thread pool thread. Each request starts on a thread pool thread, and `await` returns it to the pool during I/O.

**Implications:**
- `ConfigureAwait(false)` is unnecessary (but harmless).
- The middleware pipeline is fully async → high concurrency with a small thread pool.
- `HttpContext` is **not thread-safe** — don't pass it to background tasks.

```csharp
app.Use(async (context, next) =>
{
    // Thread A
    await next(context);    // thread returned during downstream I/O
    // Thread B (possibly different) — but HttpContext is still valid
    context.Response.Headers.Add("X-Custom", "value");
});
```

### 7. Diagnosing async performance issues

| Tool | Purpose |
|---|---|
| `dotnet-counters` | Monitor thread pool queue length, starvation events |
| `dotnet-trace` | Capture ETW traces for thread pool and async analysis |
| PerfView | Analyze thread time, GC pressure, async state machine allocations |
| Async analyzers | Detect sync-over-async, missing `ConfigureAwait` at compile time |
| `Activity` / `DiagnosticSource` | Distributed tracing across async boundaries |

```bash
# Detect thread pool starvation
dotnet-counters monitor System.Runtime \
  --counters threadpool-thread-count,threadpool-queue-length,threadpool-completed-items

# Capture async trace
dotnet-trace collect --providers Microsoft-DotNETCore-SampleProfiler
```

### 8. IAsyncDisposable and await using

`IAsyncDisposable` exposes `DisposeAsync()` for resources requiring **async cleanup** (flushing buffers, closing connections).

```csharp
public class AsyncDbConnection : IAsyncDisposable
{
    public async ValueTask DisposeAsync()
    {
        await _connection.CloseAsync();
        GC.SuppressFinalize(this);
    }
}

// Usage — deterministic async disposal
await using var conn = new AsyncDbConnection(connectionString);
await conn.ExecuteAsync(query);
// DisposeAsync called automatically here
```

Use `await using` whenever the resource's cleanup involves I/O (network, file, stream flush).

### 9. Retries with exponential backoff

```csharp
public async Task<T> RetryAsync<T>(
    Func<CancellationToken, Task<T>> operation,
    int maxRetries = 3,
    CancellationToken ct = default)
{
    int delay = 100; // ms
    for (int attempt = 0; ; attempt++)
    {
        try
        {
            return await operation(ct);
        }
        catch (TransientException) when (attempt < maxRetries)
        {
            var jitter = Random.Shared.Next(0, 50);
            await Task.Delay(delay + jitter, ct);
            delay *= 2; // 100 → 200 → 400 → ...
        }
    }
}
```

**Recursive approach** — public wrapper with a private recursive core:

```csharp
// Public wrapper — clean API surface, no state machine (async elided)
public Task<T> RetryAsync<T>(
    Func<CancellationToken, Task<T>> operation,
    int maxRetries = 3,
    CancellationToken ct = default)
{
    return RetryCore(operation, maxRetries, delayMs: 100, attempt: 0, ct);
}

// Private recursive implementation
private async Task<T> RetryCore<T>(
    Func<CancellationToken, Task<T>> operation,
    int maxRetries,
    int delayMs,
    int attempt,
    CancellationToken ct)
{
    try
    {
        return await operation(ct);
    }
    catch (TransientException) when (attempt < maxRetries)
    {
        var jitter = Random.Shared.Next(0, 50);
        await Task.Delay(delayMs + jitter, ct);
        return await RetryCore(operation, maxRetries, delayMs * 2, attempt + 1, ct);
    }
}
```

**Or with Polly:**
```csharp
var retryPolicy = Policy
    .Handle<HttpRequestException>()
    .WaitAndRetryAsync(3, attempt =>
        TimeSpan.FromMilliseconds(100 * Math.Pow(2, attempt))
        + TimeSpan.FromMilliseconds(Random.Shared.Next(0, 50)));
```

### 10. ExecutionContext flow with async/await

`ExecutionContext` (carrying `AsyncLocal<T>`, security context, etc.) **automatically flows** across `await` boundaries — the state machine captures and restores it.

```csharp
private static AsyncLocal<string> _correlationId = new();

public async Task HandleRequest()
{
    _correlationId.Value = Guid.NewGuid().ToString();

    await StepA(); // _correlationId flows automatically
    await StepB(); // still accessible here
}
```

**Key behaviors:**
- `AsyncLocal<T>` values are **copy-on-write** per async fork — each `Task.Run` or `Task.WhenAll` branch gets its own copy.
- Use `ExecutionContext.SuppressFlow()` to prevent flow in rare perf-sensitive scenarios.
- This is how correlation IDs, ambient transactions, and culture flow across async boundaries without thread-static issues.

```csharp
// Suppressing flow (advanced, rare)
using (ExecutionContext.SuppressFlow())
{
    Task.Run(() => { /* no ExecutionContext here */ });
}
```
