# Interview MCQs and Answer Explanations — Race Conditions

## Table of Contents

- [1. Beginner Level](#1-beginner-level)
- [2. Senior Level](#2-senior-level)
- [3. Architect Level](#3-architect-level)
- [4. Harder Round 2](#4-harder-round-2)

This section provides interview-style multiple-choice questions for race conditions and concurrency. Some questions are code-based. Each question is worth 1 point.

## 1. Beginner Level

1. What is the main problem in a race condition?
   - Correct answer: Program correctness depends on timing of concurrent operations
   - Explanation: A race condition is about correctness depending on how concurrent operations interleave, not about performance or compilation.

2. In `counter++`, which step happens first?
   - Correct answer: Read the current value
   - Explanation: The increment starts by reading the current value. The write happens later, which is why other threads can interfere in the middle.

3. Which C# API provides an atomic increment for an `int`?
   - Correct answer: `Interlocked.Increment`
   - Explanation: `Interlocked.Increment` performs the increment as one atomic operation for supported primitive values.

4. What is the best plain-English meaning of atomic?
   - Correct answer: It happens as one indivisible step
   - Explanation: Atomic means the operation cannot be observed halfway through and cannot be interrupted as separate visible sub-steps.

5. Which shared structure is unsafe for concurrent writes unless protected?
   - Correct answer: `List<T>`
   - Explanation: `List<T>` is not safe for concurrent modification. It can lose updates or become corrupted when multiple threads write to it.

6. Code question: What is the likely issue here?

```csharp
private int _count;
public void Inc() { _count++; }
```

   - Correct answer: It can lose updates if called concurrently
   - Explanation: `_count++` is a read-modify-write sequence. Concurrent calls can interleave and overwrite one another.

7. Which symptom often hints at a race condition?
   - Correct answer: Intermittent failures under load
   - Explanation: Race conditions often appear only under certain timing conditions, so they tend to be intermittent and more visible under load.

8. What does a `lock` primarily do?
   - Correct answer: Allows only one execution path into a critical section at a time
   - Explanation: A `lock` provides mutual exclusion so only one thread can execute the protected code at a time.

9. Code question: Two tasks both read `balance = 100`, `await`, then write back updated values. What kind of bug is this?
   - Correct answer: Lost update race condition
   - Explanation: Both tasks can read the same original value and then overwrite each other's change, producing a lost update.

10. Which approach best reduces race-condition risk?
   - Correct answer: Minimize shared mutable state
   - Explanation: The less mutable state that is shared across execution paths, the fewer opportunities there are for conflicting updates.

11. What is the main difference between a process and a thread?
   - Correct answer: A process has its own memory address space; threads within a process share that memory
   - Explanation: A process owns an isolated memory address space and OS resources. Threads are the units of execution scheduled within a process, and they share that process's memory, handles, and address space — which is exactly why unsynchronized shared-state access between threads can cause race conditions.

12. Why does the .NET ThreadPool exist?
   - Correct answer: To reuse a managed set of worker threads and avoid the overhead of creating/destroying threads repeatedly
   - Explanation: Creating and tearing down OS threads is expensive. The ThreadPool maintains a pool of reusable worker threads and dynamically manages its size (with heuristics like the "hill-climbing" algorithm) so work items can be scheduled without constant thread creation overhead.

13. In C#, what best describes the relationship between `Task` and `Thread`?
   - Correct answer: A `Task` represents an asynchronous unit of work that may run on a thread-pool thread, no thread, or complete synchronously
   - Explanation: `Task` is a higher-level abstraction over a unit of asynchronous work. For I/O-bound `async` operations, a `Task` often completes without occupying any thread at all while waiting (via I/O completion ports), whereas `Thread` always maps directly to an OS thread.

14. What is the primary purpose of `async`/`await` in C#?
   - Correct answer: To write non-blocking asynchronous code that reads like sequential synchronous code
   - Explanation: `async`/`await` is syntactic sugar over continuations/state machines. It lets you write asynchronous, non-blocking code (especially for I/O) without manually chaining callbacks, while keeping a linear, readable control flow. It does not, by itself, provide parallelism or thread-safety.

15. Which scenario best describes a deadlock?
   - Correct answer: Two or more threads are each waiting on a resource the other holds, so neither can proceed
   - Explanation: A deadlock is a circular wait: each participant holds a resource the other needs and refuses to release it until it gets what it's waiting for, so the whole set of threads freezes permanently (until an external factor like a timeout intervenes).

16. Which workload is best described as I/O-bound rather than CPU-bound?
   - Correct answer: Waiting on a network call to a remote database
   - Explanation: I/O-bound work spends most of its time waiting on external operations (network, disk, database) rather than consuming CPU cycles. This is precisely the case where `async`/`await` shines, because the thread can be released back to the pool while waiting instead of blocking.

17. What does the C# `lock` keyword compile down to under the hood?
   - Correct answer: A call to `Monitor.Enter`/`Monitor.Exit` wrapped in a try/finally
   - Explanation: `lock (obj) { ... }` is compiler sugar for `Monitor.Enter(obj)` followed by the block, wrapped in a `try/finally` that calls `Monitor.Exit(obj)`, guaranteeing the lock is released even if an exception is thrown.

18. How does a `Mutex` in .NET differ from the `lock` keyword (`Monitor`)?
   - Correct answer: A `Mutex` can be named and used to synchronize across process boundaries, while `lock`/`Monitor` only works within one process
   - Explanation: A named `Mutex` is backed by an OS kernel object visible system-wide, so unrelated processes can coordinate on it. `Monitor`/`lock` is purely an in-process, managed construct and is generally cheaper because it avoids a kernel-mode transition in the uncontended case.

19. What does it mean for a piece of code to be "thread-safe"?
   - Correct answer: It behaves correctly when accessed by multiple threads concurrently, without corrupting shared state
   - Explanation: Thread-safety means invariants and correctness are preserved regardless of how concurrent callers interleave — no lost updates, no corrupted structures, no visibility problems — typically achieved via synchronization, immutability, or confinement.

20. In JavaScript/TypeScript (relevant to Angular apps), how does the runtime typically handle concurrency for async operations like HTTP calls?
   - Correct answer: It uses a single-threaded event loop with a callback/microtask queue to handle async operations
   - Explanation: JS runtimes (browser and Node) execute application code on a single main thread. Async operations (HTTP, timers, I/O) are delegated to the runtime/OS, and their callbacks/promise continuations are queued (macrotask/microtask queues) and run one at a time on the event loop — true parallel execution requires Web Workers or worker threads.

21. What is the purpose of `SemaphoreSlim` in .NET?
   - Correct answer: It limits the number of threads/tasks that can access a resource concurrently to a fixed count
   - Explanation: `SemaphoreSlim` maintains a count of available "slots." Callers call `WaitAsync()`/`Wait()` to acquire a slot and `Release()` to give it back, making it ideal for throttling concurrency (e.g., limiting concurrent outbound HTTP calls) rather than pure mutual exclusion.

22. Why would you prefer `Interlocked.Increment` over wrapping `counter++` in a `lock`?
   - Correct answer: `Interlocked` provides lock-free atomic operations for simple primitives, avoiding the overhead of a full lock
   - Explanation: `Interlocked` methods use CPU-level atomic instructions (e.g., `lock xadd` on x86) to perform simple operations like increment/decrement/compare-exchange without acquiring a full monitor lock, making them much cheaper for single-variable updates.

23. Which of these is a classic example of a race condition?
   - Correct answer: Two threads incrementing a shared, unsynchronized instance field at the same time
   - Explanation: Local variables are confined to a single call stack/thread and can't race. Read-only constants can't race because nothing mutates them. A shared, mutable instance field updated by multiple threads without synchronization is the textbook race condition setup.

24. What is `ConcurrentDictionary<TKey,TValue>` designed for?
   - Correct answer: A thread-safe dictionary designed for concurrent reads and writes without external locking
   - Explanation: `ConcurrentDictionary` uses internal fine-grained locking/lock-free techniques so multiple threads can safely read and write concurrently, and it exposes atomic helper methods like `TryAdd`, `TryUpdate`, and `AddOrUpdate` to avoid check-then-act races.

25. What does `Parallel.For`/`Parallel.ForEach` do in the Task Parallel Library (TPL)?
   - Correct answer: It partitions loop iterations across multiple threads to run independent iterations concurrently
   - Explanation: `Parallel.For`/`ForEach` partitions the iteration range across ThreadPool worker threads to exploit multiple CPU cores for CPU-bound work. It does **not** make the loop body thread-safe — shared mutable state accessed inside the loop still needs synchronization.

26. How does a `CancellationToken` cancel an asynchronous operation in .NET?
   - Correct answer: It cooperatively signals that an operation should stop, and the operation must check/observe it
   - Explanation: Cancellation in .NET is cooperative, not preemptive. Code must periodically check `token.IsCancellationRequested` or call `token.ThrowIfCancellationRequested()`, or pass the token to APIs (like `Task.Delay` or stream reads) that observe it internally.

27. What role does `SynchronizationContext` play with `async`/`await` in a UI application (e.g., WPF)?
   - Correct answer: It captures the current context (e.g., UI thread) so a continuation after `await` can be posted back to it
   - Explanation: By default, `await` captures the current `SynchronizationContext` (e.g., the UI context) and marshals the continuation back onto it, which is why code after `await` in a WPF/WinForms event handler can safely touch UI controls.

28. What is the effect of calling `.ConfigureAwait(false)` on an awaited task?
   - Correct answer: It tells the awaiter not to try to resume on the original captured context, improving library performance and avoiding deadlocks
   - Explanation: `ConfigureAwait(false)` skips capturing/restoring the original `SynchronizationContext`, letting the continuation run on whatever thread completed the task. This is commonly used in library code to avoid unnecessary context-switch overhead and to prevent classic UI-thread deadlocks caused by blocking on async code.

29. What is the role of `zone.js` in a classic (pre-zoneless) Angular application?
   - Correct answer: It patches async browser APIs so Angular knows when to run change detection after async work completes
   - Explanation: `zone.js` monkey-patches async APIs (`setTimeout`, promises, DOM events, XHR, etc.) to create an execution "zone" that Angular hooks into, so it automatically knows when an async task finished and triggers change detection — without you manually telling it to refresh the view.

30. Which situation is a textbook cause of a deadlock?
   - Correct answer: Thread A locks resource X then waits for Y; Thread B locks Y then waits for X
   - Explanation: This is the classic "circular wait" pattern — inconsistent lock ordering across threads. The standard fix is to always acquire multiple locks in a consistent, globally-agreed order (or avoid holding multiple locks at once).

31. Which mechanism is classically used to coordinate a producer thread with a consumer thread waiting for new items?
   - Correct answer: `Monitor.Wait`/`Monitor.Pulse` (or `BlockingCollection`) to signal between producer and consumer threads
   - Explanation: `Monitor.Wait`/`Pulse` (or the higher-level `BlockingCollection<T>`, built on similar signaling) lets a consumer thread block efficiently until a producer signals that new work is available, avoiding busy-waiting/polling.

32. What does `ManualResetEventSlim` do?
   - Correct answer: It is a signaling construct that blocks threads until it is explicitly set, then lets all waiters through
   - Explanation: `ManualResetEventSlim` starts unsignaled; any thread calling `Wait()` blocks until another thread calls `Set()`, at which point **all** waiting threads are released (unlike `AutoResetEvent`, which releases only one waiter and resets automatically).

33. What is a key practical difference between `Task.Run(...)` and `Task.Factory.StartNew(...)`?
   - Correct answer: `Task.Run` uses sensible defaults (thread pool, no parent-child) while `Task.Factory.StartNew` has more configurable but easier-to-misuse defaults
   - Explanation: `Task.Run` is effectively a simplified wrapper that always schedules to the ThreadPool and correctly unwraps `Task<Task<T>>`. `Task.Factory.StartNew` exposes more options (custom `TaskScheduler`, `TaskCreationOptions`) but its defaults (like not auto-unwrapping nested tasks) are easy to get wrong.

34. What does setting a `Thread`'s `Priority` property actually guarantee?
   - Correct answer: Thread priority is only a scheduling hint to the OS and does not guarantee ordering or prevent race conditions
   - Explanation: `Thread.Priority` is merely a hint that influences OS scheduling preference; it provides no correctness guarantees, no strict ordering, and does nothing to prevent race conditions on shared state.

35. What does the Angular `async` pipe do in a template like `{{ data$ | async }}`?
   - Correct answer: It automatically subscribes/unsubscribes to an Observable or Promise in the template and updates the view
   - Explanation: The `async` pipe subscribes to the bound Observable/Promise, marks the component for check when a new value arrives, and automatically unsubscribes when the component is destroyed — eliminating manual subscription management and a common source of memory leaks.

36. What is a fundamental difference between a JavaScript `Promise` and an RxJS `Observable`?
   - Correct answer: A Promise represents a single future value and cannot be cancelled directly; an Observable can emit multiple values over time and supports unsubscription/cancellation
   - Explanation: A `Promise` resolves/rejects exactly once and has no built-in cancellation. An `Observable` is a stream that can emit zero, one, or many values over time, is lazy (doesn't execute until subscribed), and supports cancellation via `unsubscribe()`.

37. In .NET, is a simple reference assignment like `_current = newInstance;` atomic?
   - Correct answer: Simple reference (and most primitive) assignments are atomic in .NET, but compound operations like increment are not
   - Explanation: The CLR guarantees that reads/writes of reference types and most primitive types (int, bool, reference, etc. — but not `long`/`double` on 32-bit) are atomic as single operations. However, atomicity of a single assignment does not imply visibility guarantees across threads (that's what `volatile`/memory barriers address), and it does not make multi-step operations like `x++` atomic.

38. What is "false sharing" in the context of multi-threaded performance?
   - Correct answer: Two threads accessing unrelated variables that happen to share the same CPU cache line, causing performance-degrading cache invalidation
   - Explanation: CPU caches operate on cache lines (commonly 64 bytes). If two threads modify logically unrelated variables that physically reside on the same cache line, each write invalidates the other core's cached copy, causing costly cache-coherency traffic even though there's no real data race — a pure performance problem, not a correctness one.

39. Why are immutable objects considered inherently thread-safe for reads?
   - Correct answer: Immutable objects can be freely shared and read across threads without synchronization, since their state never changes after construction
   - Explanation: Since an immutable object's state cannot change after construction, there is no write to race against — any number of threads can safely read it concurrently without locks, making immutability one of the simplest and most robust concurrency strategies.

40. What is the key difference between `Task.WaitAll(tasks)` and `await Task.WhenAll(tasks)`?
   - Correct answer: `Task.WaitAll` blocks the calling thread until all tasks complete; `Task.WhenAll` returns a Task you can `await` without blocking
   - Explanation: `Task.WaitAll` is a synchronous, blocking call — it ties up the calling thread (risky on a UI thread or in ASP.NET, can cause deadlocks). `Task.WhenAll` returns a `Task` that completes when all inputs complete, so `await`-ing it frees the thread to do other work in the meantime.

## 2. Senior Level

1. Which pattern is most vulnerable to a race without coordination?
   - Correct answer: Check-then-act on shared state
   - Explanation: Check-then-act is dangerous because another thread can change the state after the check but before the write.

2. Code question: Which replacement is safest for a shared increment-heavy counter?
   - Correct answer: `Interlocked.Increment(ref count)`
   - Explanation: `Interlocked.Increment` gives atomic increments. `volatile` improves visibility but does not make compound operations atomic.

3. In optimistic concurrency, why does the second update fail?
   - Correct answer: The concurrency token no longer matches the original read
   - Explanation: The second update is rejected because the row changed after it was read, so the saved token or row version no longer matches.

4. What does `volatile` guarantee for a field in C#?
   - Correct answer: Visibility and ordering constraints, not compound atomicity
   - Explanation: `volatile` helps ensure threads see the latest value, but it does not make `x++` or other multi-step operations atomic.

5. Code question: What is the flaw here?

```csharp
if (!_users.ContainsKey(id))
{
  _users[id] = user;
}
```

   - Correct answer: Check and write are separate, so another thread can interleave
   - Explanation: Another thread can insert the same key after the `ContainsKey` check and before the assignment. That makes the pattern race-prone.

6. Which test strategy is best for surfacing races?
   - Correct answer: Repeated concurrency stress tests
   - Explanation: Repetition under concurrency increases the chance of exposing timing-sensitive failures that normal tests may miss.

7. What is the strongest architectural benefit of immutable data in concurrent code?
   - Correct answer: It removes write-write races on that data
   - Explanation: If data cannot be changed after creation, threads cannot race to modify the same instance.

8. Code question: `await Task.WhenAll(UpdateA(), UpdateB());` Both methods mutate the same shared object. What should you suspect first?
   - Correct answer: Async race on shared mutable state
   - Explanation: `async` does not remove concurrency risk. If both operations touch the same mutable object, interleaving can still break correctness.

9. Which choice best fits when many threads enqueue work but one worker should own state changes?
   - Correct answer: Queue-based single-writer ownership
   - Explanation: A queue plus a single writer simplifies coordination and removes multi-writer races on the protected state.

10. Which statement about locks is most accurate?
   - Correct answer: Locks can protect critical sections but must be scoped carefully
   - Explanation: Locks are useful within a process, but they must guard the right shared state and be used carefully to avoid contention or deadlocks.

11. Why is `async void` considered dangerous in C#, outside of event handlers?
   - Correct answer: Exceptions thrown in an `async void` method cannot be caught by the caller with try/catch and can crash the process
   - Explanation: `async void` methods have no `Task` for the caller to await or observe, so exceptions escape onto the `SynchronizationContext`/thread pool and typically become unhandled, which can crash the process. `async Task` should be used everywhere except UI event handlers, which require `void` by signature.

12. Why can calling `task.Result` synchronously from an ASP.NET (classic) request thread cause a deadlock?
   - Correct answer: Calling `.Result` or `.Wait()` on a task from a context with a captured `SynchronizationContext` can deadlock because the continuation needs that same blocked thread
   - Explanation: The awaited task's continuation is scheduled to resume on the captured `SynchronizationContext` (e.g., the single ASP.NET request context). If the same thread is blocked waiting via `.Result`/`.Wait()`, the continuation can never run on that context, and the thread waits forever — the classic "sync-over-async" deadlock.

13. What is `System.Threading.Channels.Channel<T>` primarily used for?
   - Correct answer: An async-friendly, high-performance producer/consumer queue for passing data between tasks
   - Explanation: `Channel<T>` provides a modern, allocation-conscious, fully async producer/consumer pipeline (`Writer`/`Reader`) with backpressure support, ideal for coordinating work between producer and consumer tasks without manual locking.

14. What does a memory barrier (fence) do in the context of multi-threaded/multi-core programming?
   - Correct answer: It prevents certain CPU/compiler instruction reordering and enforces visibility of writes across threads/cores
   - Explanation: Modern CPUs and compilers may reorder instructions or cache writes locally for performance. A memory barrier forces pending writes to be flushed/visible and prevents reordering across the barrier, which is essential for correctness in lock-free algorithms.

15. Why might a hot-path async API return `ValueTask<T>` instead of `Task<T>`?
   - Correct answer: It can reduce heap allocations for frequently-completed-synchronously async methods, at the cost of stricter usage rules (e.g., don't await twice)
   - Explanation: `ValueTask<T>` avoids a heap allocation when the operation completes synchronously (common in caching scenarios), but it comes with restrictions — it generally must not be awaited multiple times or awaited concurrently, unlike `Task<T>` which can be safely awaited multiple times.

16. What problem does `ThreadLocal<T>` (or `[ThreadStatic]`) solve?
   - Correct answer: It gives each thread its own independent copy of a value, eliminating cross-thread contention for that data
   - Explanation: `ThreadLocal<T>` provides per-thread storage — each thread sees and mutates its own isolated copy, so there's no shared mutable state to race on and no synchronization overhead for that data (e.g., per-thread `Random` instances, per-thread buffers).

17. In RxJS, what does the `switchMap` operator do that makes it useful for cancelling stale in-flight requests (e.g., type-ahead search)?
   - Correct answer: It cancels/unsubscribes the previous inner observable when a new source value arrives and switches to the new one
   - Explanation: `switchMap` unsubscribes from the previous inner observable as soon as a new outer value arrives, effectively cancelling stale in-flight work — exactly what you want for a search box where only the latest keystroke's result matters (contrast with `mergeMap`, which keeps all in flight, and `concatMap`, which queues them in order).

18. Why would you use `NgZone.runOutsideAngular()` in an Angular application?
   - Correct answer: It runs code outside Angular's zone so frequent async work (e.g., animations, high-frequency events) doesn't trigger unnecessary change detection cycles
   - Explanation: Work scheduled inside Angular's zone triggers a change-detection pass whenever the patched async APIs complete. For high-frequency operations (scroll listeners, `requestAnimationFrame`, WebSocket streams) that don't need to update bindings on every tick, running outside the zone avoids costly, unnecessary change-detection cycles — you re-enter the zone (`ngZone.run(...)`) only when you actually need to update the view.

19. What is the key trade-off of using a `SpinLock` / `SpinWait` instead of a normal blocking lock?
   - Correct answer: A spinlock busy-waits (burning CPU) instead of blocking, which can be efficient for very short critical sections but wasteful for long ones
   - Explanation: A spinlock avoids the OS context-switch cost of blocking by looping ("spinning") while checking the lock, which pays off only when contention is brief. Held for longer periods, or on a single-core system, spinning wastes CPU cycles that a blocking lock would have yielded to other work.

20. What advantage does `IAsyncEnumerable<T>` (with `await foreach`) provide over returning `Task<List<T>>`?
    - Correct answer: It allows asynchronously streaming a sequence of items one at a time via `await foreach`, without loading the whole collection into memory first
    - Explanation: `IAsyncEnumerable<T>` lets a producer yield items asynchronously as they become available (e.g., paged database results, streaming API responses), so the consumer can begin processing early items before the rest are fetched — reducing memory footprint and latency to first result compared to materializing a full `List<T>` first.

21. What does PLINQ (`.AsParallel()`) provide over standard LINQ-to-Objects?
    - Correct answer: It parallelizes LINQ query execution across multiple threads via `.AsParallel()`, useful for CPU-bound data processing
    - Explanation: PLINQ partitions the source sequence and processes elements across multiple threads/cores, which can significantly speed up CPU-bound in-memory transformations, at the cost of losing guaranteed ordering (unless `.AsOrdered()` is used) and adding overhead for small/cheap workloads.

22. What benefit does `ReaderWriterLockSlim` provide over a plain `lock` for read-heavy/write-rare shared data?
    - Correct answer: It allows multiple concurrent readers OR one exclusive writer, improving throughput for read-heavy workloads over a plain exclusive lock
    - Explanation: A plain `lock` is always exclusive — only one thread at a time, even for read-only access. `ReaderWriterLockSlim` distinguishes read locks (many concurrent readers allowed) from write locks (exclusive), which greatly improves throughput when reads vastly outnumber writes.

23. In the JavaScript event loop, what is the ordering relationship between microtasks (Promise callbacks) and macrotasks (setTimeout)?
    - Correct answer: Microtasks (e.g., Promise callbacks) are drained completely before the event loop proceeds to the next macrotask (e.g., `setTimeout`)
    - Explanation: After each macrotask (or synchronous script execution), the JS engine fully drains the microtask queue (Promises, `queueMicrotask`, `MutationObserver`) before moving to the next macrotask (`setTimeout`, `setInterval`, I/O callbacks) — this is why `Promise.resolve().then(...)` always fires before a `setTimeout(fn, 0)` scheduled around the same time.

24. What is the key behavioral difference between RxJS `combineLatest` and `forkJoin`?
    - Correct answer: `combineLatest` emits whenever any source emits (once all have emitted at least once); `forkJoin` waits for all sources to complete and emits their final values once
    - Explanation: `combineLatest` re-emits a combined array/tuple every time *any* source emits a new value (after each has emitted at least once) — great for reactive forms/live dashboards. `forkJoin` behaves like `Promise.all` — it waits for every source Observable to *complete* and then emits one final combined value, ideal for "wait for N parallel HTTP calls to finish."

25. What does `await Task.Yield()` do?
    - Correct answer: It asynchronously yields control back to the caller/scheduler, allowing other queued work to run before continuing, useful for avoiding thread starvation
    - Explanation: `Task.Yield()` forces an asynchronous continuation point — the method returns control immediately and the rest resumes later via the scheduler/ThreadPool. It's used to break up long synchronous chunks of async work so other queued items get a chance to run, avoiding starving other work.

26. Why must background threads use `Dispatcher.Invoke`/`BeginInvoke` to update WPF UI elements?
    - Correct answer: UI elements have thread affinity — only the thread that created them (the UI/Dispatcher thread) may touch them directly; other threads must marshal via `Dispatcher.Invoke`/`BeginInvoke`
    - Explanation: WPF (and WinForms) UI objects are not thread-safe and are bound to the thread that created them. Touching them from another thread throws an `InvalidOperationException`. `Dispatcher.Invoke`/`BeginInvoke` marshals the call onto the correct UI thread's message loop.

27. What is a compare-and-swap (CAS) retry loop typically used for in lock-free programming?
    - Correct answer: A pattern that repeatedly reads a value, computes a new value, and uses `Interlocked.CompareExchange` to apply it only if unchanged, retrying if another thread interfered
    - Explanation: A CAS loop reads the current value, computes the desired new value, then atomically swaps it in only if the value hasn't changed since the read (via `Interlocked.CompareExchange`); if another thread beat it to the update, it retries from the read — giving lock-free correctness without blocking.

28. What is `BlockingCollection<T>` designed for?
    - Correct answer: It is a thread-safe bounded/unbounded producer-consumer collection built on top of concurrent collections, supporting blocking Add/Take semantics
    - Explanation: `BlockingCollection<T>` wraps a concurrent collection (default `ConcurrentQueue<T>`) and adds blocking/bounding semantics — `Take()` blocks when empty, `Add()` can block when a bounded capacity is full — making it a ready-made producer/consumer pipeline supporting multiple producers and consumers.

29. What guarantee does the `volatile` keyword provide regarding instruction reordering and visibility in C#?
    - Correct answer: `volatile` prevents the compiler/JIT/CPU from caching the value in a register and constrains certain reorderings, ensuring other threads see up-to-date values
    - Explanation: Marking a field `volatile` enforces "acquire/release" semantics — reads get an acquire fence, writes get a release fence — preventing the compiler/JIT/CPU from reordering surrounding memory operations across it and from caching a stale value in a register, so other threads observe the latest write in a timely manner (though it still doesn't make compound operations atomic).

30. How does Angular's `ChangeDetectionStrategy.OnPush` interact with async data emissions to improve performance?
    - Correct answer: OnPush skips change detection for a component subtree unless its inputs change by reference, an event fires within it, or an Observable bound via async pipe emits
    - Explanation: `OnPush` tells Angular to skip re-checking a component (and its subtree) unless one of a few specific triggers occurs: an `@Input()` reference changes, a DOM event originates from within the component, or an Observable/Promise bound with the `async` pipe emits a new value (which explicitly marks the component dirty) — reducing unnecessary change-detection work in large apps.

31. What is `TaskCompletionSource<T>` used for?
    - Correct answer: It is used to manually create and control a `Task` whose completion is driven by external code (e.g., wrapping a callback-based API into `async`/`await`)
    - Explanation: `TaskCompletionSource<T>` gives you a `Task<T>` whose completion (`SetResult`/`SetException`/`SetCanceled`) is driven manually from arbitrary code — the classic use case is bridging an old event/callback-based API (e.g., a hardware SDK callback) into the `async`/`await` world.

32. Why does `Task.WaitAll` (or `Parallel.For`) often throw an `AggregateException` instead of the original exception directly?
    - Correct answer: It wraps one or more exceptions thrown by parallel/child operations (e.g., `Task.WaitAll`, `Parallel.For`) into a single exception with an `InnerExceptions` collection
    - Explanation: Because multiple parallel operations can fail independently and simultaneously, .NET wraps *all* of their exceptions into a single `AggregateException.InnerExceptions` collection rather than picking just one — note that `await`-ing a faulted task instead unwraps and rethrows only the first exception, which is a common source of confusion between the two styles.

33. Why is `SemaphoreSlim.WaitAsync()` preferred over `lock` when you need to protect a critical section that contains `await`?
    - Correct answer: `lock` cannot safely wrap `await` because it's synchronous and tied to thread identity; `SemaphoreSlim.WaitAsync()` supports asynchronous waiting without blocking a thread
    - Explanation: `Monitor`/`lock` is thread-affine — the thread that enters must be the one that exits, which breaks under `await` (the continuation may resume on a different thread), and the compiler outright disallows `await` inside a `lock` block. `SemaphoreSlim.WaitAsync()` is designed for async code — it asynchronously waits for a slot without blocking any thread.

34. What do `SharedArrayBuffer` and the `Atomics` API enable in JavaScript that plain objects/postMessage do not?
    - Correct answer: They provide true shared-memory concurrency between Web Workers, with `Atomics` offering atomic operations similar to `Interlocked` in .NET
    - Explanation: Normally, Web Workers communicate by copying/structured-cloning messages (`postMessage`), with no shared memory. `SharedArrayBuffer` provides a real shared memory region across workers/threads, and `Atomics` provides atomic read/write/compare-exchange/wait-notify operations on it — bringing genuine low-level shared-memory concurrency (and its race-condition risks) to JavaScript.

35. What is a common pitfall when using `Task.ContinueWith` instead of `await` in UI or context-sensitive code?
    - Correct answer: By default, `ContinueWith` does not capture the `SynchronizationContext` and may run on an arbitrary thread pool thread unless a `TaskScheduler` is specified, unlike `await` which does by default
    - Explanation: `await` implicitly captures and restores the current `SynchronizationContext`/`TaskScheduler` (unless you use `ConfigureAwait(false)`). `ContinueWith` does *not* do this by default — you must explicitly pass `TaskScheduler.FromCurrentSynchronizationContext()` to get the continuation back on the UI thread, and forgetting this is a classic source of "why isn't my UI updating" or cross-thread-exception bugs.

36. What is the key difference between an RxJS `Subject` and a `BehaviorSubject`?
    - Correct answer: A `Subject` has no initial/current value and only emits to subscribers active at emission time; a `BehaviorSubject` holds a current value and immediately emits it to new subscribers
    - Explanation: A plain `Subject` has no memory of past values — a late subscriber misses everything emitted before it subscribed. A `BehaviorSubject` requires an initial value, always retains the latest emitted value, and immediately replays that current value to any new subscriber, which is useful for representing "current state" (e.g., logged-in user, current filter).

37. In ASP.NET Core dependency injection, why is it risky to capture a scoped service and use it inside an unawaited `Task.Run(...)` background operation from a controller?
    - Correct answer: A scoped service instance is created once per request/scope; using it across parallel `Task.Run` fire-and-forget work without a new scope can cause race conditions or disposed-object errors
    - Explanation: Scoped services (like `DbContext`) are tied to the request's `IServiceScope` and are disposed when the request ends. If you fire off background work with `Task.Run` without awaiting it, the HTTP request can complete and dispose the scope (and the `DbContext`) while the background task is still using it, causing `ObjectDisposedException`s or corrupted state — the fix is to create a new scope (e.g., via `IServiceScopeFactory`) inside the background work.

38. What does the RxJS `debounceTime` operator do, and why is it commonly paired with type-ahead search inputs?
    - Correct answer: It emits a value only after a specified silence period has passed since the last emission, discarding intermediate rapid-fire emissions (e.g., keystrokes)
    - Explanation: `debounceTime(ms)` waits for a pause of at least `ms` milliseconds of silence before emitting the most recent value, discarding everything emitted during rapid bursts. Paired with `switchMap`, it's the standard pattern for type-ahead search: wait until the user pauses typing, then fire (and auto-cancel stale) HTTP requests.

39. What is the "double-checked locking" pattern typically used for, and what must you be careful about when implementing it manually in C#?
    - Correct answer: It checks a condition, acquires a lock, then re-checks the condition inside the lock before doing expensive/one-time initialization, reducing lock overhead for the common case
    - Explanation: The pattern checks "is it already initialized?" without a lock (fast path), only acquires the lock if not, then re-checks inside the lock (in case another thread initialized it while waiting) before doing the expensive work once. Implemented manually, it's notoriously easy to get wrong without proper memory barriers/`volatile` on the reference being published — which is exactly why `Lazy<T>` exists to handle this correctly for you.

40. How does `Lazy<T>` in .NET help avoid race conditions during expensive one-time initialization?
    - Correct answer: `Lazy<T>` supports configurable thread-safety modes (e.g., `ExecutionAndPublication`) so the factory function runs safely exactly once even under concurrent first access
    - Explanation: `Lazy<T>` implements the double-checked-locking / one-time-initialization pattern correctly and safely internally. Its `LazyThreadSafetyMode` (e.g., `ExecutionAndPublication`, the default) ensures that even if multiple threads race to access `.Value` for the first time, the factory delegate executes exactly once and all threads observe the same correctly-published result — sparing you from re-implementing double-checked locking by hand.

## 3. Architect Level

1. For a high-write distributed workflow, what most directly prevents duplicate side effects on retries?
   - Correct answer: Idempotency keys
   - Explanation: Idempotency keys let the system recognize duplicate requests and return the original outcome instead of repeating the side effect.

2. Which ownership model most cleanly reduces races around a hot aggregate?
   - Correct answer: Single writer per aggregate or partition
   - Explanation: Restricting mutation to one owner per aggregate or partition greatly reduces coordination complexity.

3. Code or system question: Multiple services update the same customer row. What is the best first control at the persistence boundary?
   - Correct answer: Concurrency token or row-version check
   - Explanation: A concurrency token detects conflicting writes where they matter most: at the database write boundary.

4. In an event-driven system, what reduces ordering races for sensitive entity updates?
   - Correct answer: Per-key ordered processing
   - Explanation: If updates for the same key are processed in order by a single stream or partition, ordering conflicts are much easier to control.

5. Which design review question best exposes concurrency risk early?
   - Correct answer: Who is allowed to mutate this state, and when?
   - Explanation: Concurrency bugs usually center on shared mutable state, so identifying ownership and mutation rules early is critical.

6. When conflicts are rare but overwrites must be detected, which strategy fits best?
   - Correct answer: Optimistic concurrency
   - Explanation: Optimistic concurrency assumes most updates will not conflict, but it validates at save time so conflicting writes are rejected.

7. Code or system question: A read model is eventually consistent. What should writers avoid assuming?
   - Correct answer: That a just-written value will be instantly visible everywhere
   - Explanation: Eventual consistency means propagation takes time. Writers and callers must not depend on immediate global visibility.

8. Which database feature helps reject conflicting inserts for a supposed-unique business key?
   - Correct answer: Unique constraint or unique index
   - Explanation: A unique constraint enforces the rule at the database level, preventing duplicates even when concurrent requests race.

9. What is the main reason to separate command ownership from query access in concurrent systems?
   - Correct answer: To constrain mutation paths and reduce coordination complexity
   - Explanation: Fewer mutation paths make it easier to reason about consistency, ownership, and ordering.

10. Which architecture pattern best fits strict sequential updates for the same entity key?
   - Correct answer: Actor or partitioned single-threaded processor per key
   - Explanation: Actor-style or per-key single-threaded processing ensures updates for the same key are handled one at a time in a predictable order.

11. What does the CAP theorem state about distributed systems, and how is it usually applied in practice?
   - Correct answer: Under a network partition, a distributed system must trade off between Consistency and Availability; Partition tolerance is generally non-negotiable for a truly distributed system
   - Explanation: CAP says that during a network partition, you must choose between Consistency (every read gets the latest write) and Availability (every request gets a response). Since partitions are a fact of life in real distributed systems, Partition tolerance is effectively mandatory, so the practical trade-off architects make is CP vs. AP.

12. What problem does a distributed lock (e.g., Redlock, ZooKeeper lock) solve that an in-process `lock` cannot?
   - Correct answer: A coordination mechanism (e.g., via Redis, ZooKeeper, or a database) that ensures only one node/process across a cluster can hold a resource at a time
   - Explanation: An in-process `lock`/`Monitor` only coordinates threads within one process's memory. When multiple separate service instances/machines need mutual exclusion over a shared external resource, they need an externally visible coordination point — a distributed lock backed by Redis, ZooKeeper, or a database row — with careful handling of lease expiry and fencing tokens to avoid split-brain issues.

13. What is the Saga pattern used for in microservice architectures?
   - Correct answer: A sequence of local transactions across services, each with a compensating action, used to maintain eventual consistency without distributed transactions
   - Explanation: Since distributed two-phase-commit transactions across microservices are impractical, a Saga breaks a business process into a series of local transactions, each service performing its part and publishing an event; if a later step fails, previously completed steps are undone via explicit compensating actions rather than a database rollback.

14. What is "backpressure" in a streaming/message-processing architecture, and why does it matter?
   - Correct answer: A mechanism for a slow consumer to signal a fast producer to slow down, preventing unbounded memory growth or overload
   - Explanation: Without backpressure, a fast producer can overwhelm a slower consumer, causing unbounded queue growth, memory exhaustion, or dropped messages. Backpressure mechanisms (bounded channels, reactive streams' `request(n)` protocol, TCP flow control) let the consumer signal capacity back to the producer to keep the system stable under load.

15. In a partitioned/sharded event stream (e.g., Kafka), how does partitioning by key help with concurrency and ordering?
   - Correct answer: Sharding by a consistent key ensures related events/updates for that key land in the same partition, enabling ordered, single-writer processing per key
   - Explanation: Kafka (and similar systems) guarantee ordering only *within* a partition. By hashing a consistent key (e.g., customer ID) to choose the partition, all events for that key are processed in order by a single consumer at a time, giving per-key ordering without requiring costly global ordering across the whole topic.

16. What problem do consensus algorithms like Raft or Paxos solve in distributed systems?
   - Correct answer: A distributed system pattern for getting a set of nodes to agree on a single value/state despite failures (e.g., Raft, Paxos)
   - Explanation: Consensus algorithms let a cluster of nodes agree on a single consistent value or sequence of operations (e.g., who is the leader, or the next entry in a replicated log) even in the presence of node failures or message delays, forming the backbone of systems like etcd, ZooKeeper, and distributed databases.

17. At an architectural level, how do pessimistic and optimistic concurrency control strategies fundamentally differ?
   - Correct answer: Pessimistic concurrency acquires locks upfront to prevent conflicts; optimistic concurrency allows concurrent access and detects conflicts at commit time via versioning
   - Explanation: Pessimistic concurrency assumes conflicts are likely, so it locks resources before use (e.g., `SELECT ... FOR UPDATE`), trading throughput for safety. Optimistic concurrency assumes conflicts are rare, letting operations proceed freely and only checking for conflicts (e.g., via a row version/ETag) at commit time, rejecting the write if it detects a conflicting change — better throughput when contention is genuinely low.

18. What is an "idempotent consumer" pattern designed to protect against in a message-driven architecture with at-least-once delivery?
   - Correct answer: A message consumer designed so processing the same message multiple times (due to at-least-once delivery/retries) produces the same end result as processing it once
   - Explanation: Most message brokers guarantee at-least-once delivery, meaning duplicates are possible (due to retries, redelivery after a crash, etc.). An idempotent consumer tracks processed message IDs (or designs operations to be naturally idempotent, e.g., "set balance to X" instead of "add X") so reprocessing the same message doesn't cause duplicate side effects.

19. What trade-off does an eventually consistent architecture make compared to a strongly consistent one?
   - Correct answer: It sacrifices immediate global visibility of writes in exchange for higher availability and partition tolerance/scalability
   - Explanation: Eventual consistency lets replicas/read-models lag briefly behind the latest write in exchange for lower latency, higher availability, and better horizontal scalability — at the cost of readers sometimes seeing stale data and needing explicit conflict-resolution or reconciliation strategies.

20. How does the Circuit Breaker pattern help protect a system under high concurrent load against a failing downstream dependency?
    - Correct answer: It prevents cascading failures by temporarily stopping calls to a failing dependency, protecting shared thread/connection pools from exhaustion under concurrent load
    - Explanation: When a downstream dependency starts failing or timing out, continuing to send concurrent requests can exhaust thread pools, connections, or queues, taking down the calling service too. A circuit breaker detects the failure rate, "opens" to fail fast without calling the dependency, then periodically probes to see if it has recovered — protecting shared resources from pileup under concurrent load.

21. In an event-sourced system, how is optimistic concurrency typically enforced when appending events to an aggregate's stream?
    - Correct answer: By appending events with an expected version number, so a concurrent writer appending against a stale version is rejected, detecting conflicting concurrent writes to the same aggregate stream
    - Explanation: Each append specifies the expected current version/sequence number of the stream (based on what the writer last read). If another writer already appended an event in the meantime, the version no longer matches, the append is rejected, and the caller must reload and retry — the event-sourcing analog of a row-version check.

22. What problem do vector clocks (or logical clocks) solve in distributed systems?
    - Correct answer: A mechanism to track causal ordering/happened-before relationships between events across distributed nodes without relying on synchronized wall-clock time
    - Explanation: Physical clocks across machines are never perfectly synchronized, so timestamp comparison alone can't reliably establish "what happened first." Vector clocks (a set of per-node counters) let a system determine causal ("happened-before") relationships and detect concurrent, potentially conflicting updates without depending on wall-clock accuracy.

23. Why is leader election important in a clustered/distributed system with multiple redundant nodes?
    - Correct answer: It ensures exactly one node in a cluster is designated to perform certain coordinated actions (e.g., scheduling, writes), avoiding conflicting concurrent actions from multiple nodes
    - Explanation: Some responsibilities (e.g., running a scheduled job exactly once, coordinating writes) must not be performed by multiple nodes simultaneously. Leader election protocols (built on consensus like Raft) select a single active leader, and provide failover so a new leader is elected if the current one fails, avoiding the "split brain" scenario of multiple nodes acting as if they're in charge.

24. What is the two-phase commit (2PC) protocol, and what is its major architectural weakness that leads many systems to prefer Sagas instead?
    - Correct answer: A protocol where a coordinator asks all participants to 'prepare' (vote) and only commits if all agree, but it can block indefinitely if the coordinator or a participant fails mid-protocol
    - Explanation: In 2PC, a coordinator collects "prepared/ready to commit" votes from all participants before telling everyone to commit. If the coordinator crashes after some participants have prepared but before sending the final decision, those participants can be left blocked holding locks indefinitely — a poor fit for the loosely-coupled, independently-deployable nature of microservices, which is why Sagas (local transactions + compensations) are favored instead.

25. Under high concurrency, how can database connection pool exhaustion occur, and why does blocking/sync-over-async code make it worse?
    - Correct answer: Too many concurrent requests holding onto pooled connections longer than necessary (e.g., due to blocking calls or leaks) can exhaust the pool, causing new requests to queue or time out
    - Explanation: A connection pool has a finite max size. Under high concurrent load, if requests hold connections for longer than necessary — e.g., blocking a thread synchronously while waiting on I/O instead of releasing it back to do other work — connections get tied up disproportionately, and new requests queue up waiting for a free connection or time out, amplifying the effect of any slowdown.

26. What architectural purpose does rate limiting (e.g., token bucket) serve alongside concurrency control?
    - Correct answer: It caps the number of concurrent or per-time-window requests a client/service can make, protecting shared downstream capacity from being overwhelmed
    - Explanation: Rate limiting caps the request rate (e.g., via a token bucket or sliding window) to keep load within the capacity the system/downstream dependency can safely handle, complementing (not replacing) concurrency controls like bulkheads and circuit breakers that handle failures once overload happens.

27. What is a "cache stampede" (a.k.a. thundering herd), and why is it a concurrency concern?
    - Correct answer: When a popular cache entry expires, many concurrent requests simultaneously miss the cache and hit the backing store/database at once, potentially overwhelming it
    - Explanation: When a hot cache key expires, if there's no coordination, every concurrent request that misses the cache goes straight to the origin store at the same instant, potentially spiking load enough to cause a cascading outage. Mitigations include request coalescing/single-flight (only one request recomputes, others wait), staggered expirations, or probabilistic early refresh.

28. How do multi-region active-active systems typically handle concurrent writes to the same record from different regions?
    - Correct answer: By using conflict resolution strategies (e.g., last-write-wins, CRDTs, or application-level merge logic) since network partitions/latency make strict global locking impractical
    - Explanation: Coordinating a global lock across regions separated by real network latency is prohibitively slow and fragile. Instead, active-active systems typically accept concurrent regional writes and reconcile conflicts after the fact using strategies like last-write-wins (with timestamps), Conflict-free Replicated Data Types (CRDTs), or custom merge logic tailored to the domain.

29. What is the purpose of the Bulkhead pattern in a highly concurrent, multi-dependency system?
    - Correct answer: It isolates resources (thread pools, connections) per dependency/component so that one overloaded or failing dependency cannot exhaust resources needed by unrelated parts of the system
    - Explanation: Named after ship bulkheads that contain flooding to one compartment, this pattern partitions resources (e.g., separate thread pools or connection pools per downstream dependency) so that if one dependency becomes slow or unresponsive under load, it can't consume all available threads/connections and starve unrelated, healthy parts of the system.

30. What race condition/consistency problem does the Transactional Outbox pattern solve for a service that must both update its database and publish an event?
    - Correct answer: It writes the state change and the outgoing event/message to the same local database transaction, then a separate relay publishes it, avoiding the dual-write race between DB commit and message publish
    - Explanation: Writing to the database and then separately publishing a message to a broker is not atomic — a crash between the two steps can leave the DB updated but the event never published (or vice versa). The Outbox pattern writes the event into an "outbox" table as part of the same local DB transaction as the state change, and a separate relay process/CDC mechanism reliably publishes it afterward, eliminating that dual-write race.

31. What is a Conflict-free Replicated Data Type (CRDT) designed to guarantee?
    - Correct answer: A data structure designed so concurrent updates from different replicas can always be merged deterministically into a consistent result without central coordination
    - Explanation: CRDTs (e.g., G-Counters, OR-Sets) are designed with merge operations that are commutative, associative, and idempotent, so replicas can accept concurrent local writes independently and later merge their states in any order and still converge to the same final result — no coordination or conflict resolution logic required.

32. What is a "hot partition" (hot key) problem in a sharded/partitioned system?
    - Correct answer: A single partition/shard/key receives disproportionately high traffic, becoming a bottleneck even though the overall system has capacity elsewhere
    - Explanation: Sharding distributes load assuming a roughly even key distribution. If one key (e.g., a celebrity user ID, a popular product) receives far more traffic than others, that single partition becomes a concurrency/throughput bottleneck even though the cluster overall has plenty of spare capacity elsewhere.

33. Which set of techniques is commonly used to mitigate a hot-partition/hot-key problem?
    - Correct answer: Techniques like key salting/sub-sharding, request batching, caching, or local aggregation before write to spread load off the single hot key
    - Explanation: Common mitigations include salting the key to spread it across sub-partitions (then merging results), batching/aggregating writes locally before committing, adding a caching layer in front of the hot key, or using write-behind buffering — all aimed at reducing the concurrent pressure that lands directly on the single hot key.

34. Why is distributed tracing (e.g., OpenTelemetry) particularly valuable for diagnosing concurrency-related issues in a microservice architecture?
    - Correct answer: It propagates a correlation/trace ID across service boundaries so you can reconstruct the causal path and timing of a request across concurrent, asynchronous service calls
    - Explanation: Concurrency bugs in distributed systems often manifest as ordering or timing issues across many asynchronous service calls. Distributed tracing propagates a trace/span context across service boundaries, letting engineers reconstruct the actual causal and temporal sequence of concurrent operations after the fact — invaluable for diagnosing races that only manifest under specific interleavings in production.

35. How can garbage collector (GC) behavior interact with perceived concurrency/latency issues in a high-throughput .NET service?
    - Correct answer: A stop-the-world garbage collection pause can momentarily halt all application threads, which can appear as latency spikes or false-positive timeouts under concurrent load
    - Explanation: Even with the mostly-concurrent Server GC, certain collection phases can briefly pause managed threads. Under high concurrent load with high allocation rates, these pauses can manifest as intermittent latency spikes, timeouts, or even cascading failures if downstream systems interpret the pause as an unresponsive node — something architects must account for via tuning, workstation vs. server GC choice, and allocation-reduction strategies.

36. What does Amdahl's Law tell architects about the limits of scaling a workload through parallelism?
    - Correct answer: It shows that the maximum speedup from parallelizing a workload is limited by the proportion of the workload that must remain sequential
    - Explanation: Amdahl's Law formalizes that if a fraction of a workload is inherently sequential (cannot be parallelized), that fraction caps the maximum achievable speedup no matter how many additional processors/threads you throw at the parallelizable part — a key reminder that reducing the sequential portion often matters more than adding raw parallel capacity.

37. How does the Actor Model (e.g., Akka.NET, Orleans) avoid classic race conditions on an actor's internal state, and what role does supervision play?
    - Correct answer: Each actor processes messages sequentially from its own mailbox (avoiding shared-state races), and a supervisor hierarchy handles failures by restarting/stopping child actors
    - Explanation: Each actor owns its private state and processes incoming messages from its mailbox strictly one at a time, so there's never concurrent access to that state — eliminating the need for locks on actor-internal data. Supervisor hierarchies additionally provide fault isolation: when a child actor fails, its supervisor decides whether to restart, stop, or escalate, containing failures without corrupting shared state.

38. How should a REST API be designed to prevent lost-update race conditions when multiple clients might PUT/PATCH the same resource concurrently?
    - Correct answer: Requiring an `If-Match`/ETag or version header on PUT/PATCH requests so the server rejects updates based on stale client-side data, preventing lost updates
    - Explanation: This is HTTP's built-in optimistic concurrency mechanism: the server returns an `ETag` with a `GET`, and the client must send it back via `If-Match` on the subsequent `PUT`/`PATCH`. If the resource has changed since the client last read it (ETag mismatch), the server responds `412 Precondition Failed` rather than silently overwriting a concurrent update — directly preventing the classic lost-update race.

39. In a CQRS architecture, why must client applications be designed to tolerate temporary staleness when reading from a materialized view/read model right after a write?
    - Correct answer: Because materialized read views/projections are updated asynchronously from the write side, so there's a window where a query can return stale data relative to the latest write
    - Explanation: In CQRS, the write model persists the change and publishes an event; the read model/projection is updated asynchronously in response to that event. Between the write completing and the projection catching up, a client reading the materialized view may see stale data — UIs commonly work around this with optimistic UI updates, polling, or "read-your-writes" techniques.

40. Why is chaos engineering particularly relevant to validating concurrency and resiliency assumptions in a distributed architecture?
    - Correct answer: It deliberately injects failures/latency/partitions into a running system to validate that concurrency controls (retries, circuit breakers, timeouts, idempotency) actually behave correctly under real-world fault conditions
    - Explanation: Many concurrency and resiliency mechanisms (retries, idempotency, circuit breakers, timeouts, failover) only really get exercised under actual failure conditions, which are hard to reproduce in normal testing. Chaos engineering deliberately injects faults (killing nodes, adding latency, partitioning networks) in a controlled way to verify these mechanisms behave as designed rather than just as intended on paper.

## 4. Harder Round 2

This second round is slightly harder and focuses more on subtle concurrency behavior, persistence boundaries, and architectural controls.

1. Which statement about race conditions is most precise?
   - Correct answer: They occur when correctness depends on an unsafe interleaving of concurrent operations
   - Explanation: A race condition is fundamentally about unsafe interleavings. Threads, tasks, callbacks, and distributed workers can all produce them.

2. Code question: Why is `volatile int count; count++;` still unsafe?
   - Correct answer: `volatile` does not make read-modify-write atomic
   - Explanation: `volatile` helps with visibility and ordering, but `count++` is still multiple steps, so two writers can still collide.

3. In optimistic concurrency with a row version, what does a 0-row update usually mean?
   - Correct answer: The row changed since it was read or no longer matches the original token
   - Explanation: The row version no longer matches the version seen during the read, so the update condition fails and no rows are updated.

4. Which bug pattern is this: read shared value, await I/O, then write based on stale local copy?
   - Correct answer: Lost update from stale read
   - Explanation: The write is based on an old snapshot of state. Another operation may have already changed the shared value.

5. Code question: Which is the safest insertion API for concurrent add-if-absent semantics?
   - Correct answer: `ConcurrentDictionary.TryAdd`
   - Explanation: `TryAdd` combines the existence check and insertion into one concurrency-safe operation.

6. If adding more logging makes the bug disappear, what should you suspect?
   - Correct answer: Timing-sensitive concurrency behavior
   - Explanation: Logging changes timing. If that changes the result, a race condition or similar timing-sensitive bug is a strong suspect.

7. Which design most reduces contention for a hot account balance aggregate?
   - Correct answer: Route updates for that key through a single owner or partition
   - Explanation: Constraining writes for one hot key to one owner simplifies ordering and consistency.

8. Code question: Two threads both execute `if (state == Open) state = Closed;`. What is the core issue?
   - Correct answer: Check-then-act on shared mutable state
   - Explanation: The check and the update are separate steps. Another thread can change `state` between them.

9. Which database feature best prevents duplicate rows for a unique business identifier under concurrency?
   - Correct answer: Unique constraint or unique index
   - Explanation: The database must enforce uniqueness at the final write boundary to close the race completely.

10. Which testing approach gives the best chance of reproducing a race?
   - Correct answer: High-repeat parallel stress with assertions on invariants
   - Explanation: Repetition plus concurrency plus invariant checks exposes timing-sensitive correctness failures more reliably.

11. In a distributed system, what is idempotency primarily protecting against?
   - Correct answer: Duplicate side effects from retries or replays
   - Explanation: Idempotency ensures that replaying the same request does not apply the same effect multiple times.

12. Code question: What is the strongest reason `Interlocked.CompareExchange` is useful?
   - Correct answer: It can perform conditional atomic updates without a full lock
   - Explanation: Compare-and-swap lets you update a value only if it still equals the expected old value, atomically.

13. What is the main concurrency risk of exposing mutable static state in a web app?
   - Correct answer: Many concurrent requests may mutate the same shared object
   - Explanation: Static mutable state is shared process-wide, so many requests can race on it at once.

14. Which statement about locks in async code is most accurate?
   - Correct answer: Synchronous locks protect only in-process critical sections and should not wrap awaits
   - Explanation: `lock` is synchronous and should not span `await`. Async coordination usually needs different primitives or design choices.

15. Code question: Why is a queue plus single consumer often safer than many direct writers?
   - Correct answer: It centralizes mutation order and ownership
   - Explanation: One consumer creates a clear serialization point for mutations and simplifies reasoning about order.

16. What does eventual consistency mean for readers?
   - Correct answer: A recent write may not be visible everywhere immediately
   - Explanation: Eventual consistency trades immediate global visibility for propagation over time.

17. Which choice best describes a compare-and-swap loop?
   - Correct answer: Re-read and retry atomic conditional update until it succeeds or aborts
   - Explanation: A CAS loop repeatedly reads the value and retries the conditional atomic update if another writer changed it first.

18. Code question: If two handlers mutate the same in-memory cache entry in response to separate events, the first architectural question is:
   - Correct answer: Who owns writes to this cache entry?
   - Explanation: Ownership is the first question because unclear write ownership is a root cause of many concurrency bugs.

19. Why are immutable messages helpful in concurrent pipelines?
   - Correct answer: Consumers cannot race to mutate the same message instance
   - Explanation: If messages are immutable, every consumer sees the same content and cannot accidentally alter shared message state.

20. What is the key purpose of defining concurrency invariants explicitly?
   - Correct answer: To state what must always remain true despite interleavings
   - Explanation: Invariants are the correctness rules that must hold no matter how operations interleave.

21. Which symptom most strongly suggests an ordering race in event-driven code?
   - Correct answer: Correct final state depends on which callback finishes last
   - Explanation: If the result changes depending on callback completion order, ordering is part of correctness and may be unsafe.

22. Code question: Which persistence strategy best detects conflicting updates without long-lived locks?
   - Correct answer: `UPDATE` by `Id` plus original version token
   - Explanation: Matching on the original version token lets the database detect whether someone else changed the row since it was read.

23. What is the biggest flaw in using `Thread.Sleep` to 'fix' a race?
   - Correct answer: It changes timing without establishing correctness
   - Explanation: Sleep may reduce the chance of failure temporarily, but it does not create any actual correctness guarantee.

24. For per-entity strict ordering, which partitioning strategy is best?
   - Correct answer: Partition by entity key so related updates land on the same ordered lane
   - Explanation: Partitioning by key keeps related updates together and makes per-entity ordering enforceable.

25. Code question: Why can `x = x + 1` fail under concurrency just like `x++`?
   - Correct answer: It is still a read-modify-write sequence
   - Explanation: The syntax changed, but the behavior is still read current value, compute new value, write new value.

26. What is the primary reason a unique constraint belongs in the database even if the app validates uniqueness?
   - Correct answer: It closes the race at the final write boundary
   - Explanation: Application checks can race. The database constraint is the final authority that prevents conflicting writes.

27. Which architectural smell most increases race-condition risk?
   - Correct answer: Many writers with unclear ownership over shared mutable state
   - Explanation: Multiple uncontrolled writers create ambiguity around sequencing, protection, and authority.

28. Code question: Why might a lock-free design still be correct?
   - Correct answer: It can rely on atomic primitives and clear ownership instead of mutual exclusion
   - Explanation: Correctness can come from atomic operations, immutability, partitioning, and ownership boundaries rather than locks alone.

29. In a CQRS-style system, why separate commands from queries for concurrency control?
   - Correct answer: To narrow mutation paths and reason about write coordination more clearly
   - Explanation: Separating reads from writes helps concentrate coordination logic around mutation paths.

30. Which statement best distinguishes synchronization from atomicity?
   - Correct answer: Atomicity is about indivisible operations; synchronization is about coordinating access and ordering
   - Explanation: Atomicity describes one operation's indivisibility. Synchronization is the broader coordination of multiple operations and actors.

31. Scenario: Double-clicking 'Transfer' fires two requests; both read balance=$500, each debit $400, leaving -$300 instead of rejecting the second. Best fix?
   - Correct answer: Wrap the balance check and debit in a single database transaction with row locking or an optimistic version check, so concurrent transfers can't both read the same starting balance
   - Explanation: The root cause is a check-then-act race on the balance. Wrapping the read-check-debit sequence in one transaction with row-level locking (`SELECT ... FOR UPDATE`) or an optimistic concurrency check (conditional `UPDATE ... WHERE balance >= @amount`) ensures the second request either waits or is rejected based on the true post-first-debit balance, rather than a stale read.

32. Scenario: The last unit of a popular item gets oversold because multiple servers read qty=1 and each independently decides it's OK to sell. Best fix at the persistence layer?
   - Correct answer: Use an atomic conditional update at the database level (e.g., `UPDATE Stock SET qty = qty - 1 WHERE id = @id AND qty > 0`) and check rows affected, instead of read-then-write in application code
   - Explanation: The bug is a classic check-then-act race across multiple servers/processes, which application-level checks cannot prevent. Pushing the check and decrement into a single atomic conditional `UPDATE` (with a `WHERE qty > 0` guard) lets the database itself guarantee only one caller succeeds in decrementing the last unit — checking `RowsAffected == 0` tells you the sale should be rejected.

33. Scenario: Double-clicking an Angular submit button fires two POST requests before the first response returns, creating duplicate records. Best client-side mitigation?
   - Correct answer: Disable the submit button immediately on first click (and/or use a loading flag with `switchMap`/take(1)) so a second click before the request completes cannot fire another submission
   - Explanation: Disabling the button (or gating the click handler behind an `isSubmitting` flag, and using `switchMap` so a new click cancels/ignores the prior in-flight request) prevents the duplicate client-side trigger. Note this is defense-in-depth only — the server should still enforce idempotency (e.g., via an idempotency key) since client-side guards can be bypassed.

34. Scenario: An ASP.NET Core singleton service has a mutable field set per-request; under load, one user sees another user's data. Correct architectural fix?
   - Correct answer: The singleton service should not hold per-request mutable state at all; move that state to a scoped/transient service or pass it as method parameters instead
   - Explanation: A singleton instance is shared across all concurrent requests for the app's lifetime, so any mutable field it holds is a shared-state race waiting to happen — one request can overwrite the field before another reads it, causing exactly this kind of cross-user data leak. The fix is architectural: per-request data belongs in a scoped/transient service or should simply be passed through method parameters, not stored on the singleton.

35. Scenario: After an update invalidates a cache entry, users still briefly see the old value because another request repopulated it with stale data mid-update. What race caused this?
   - Correct answer: A thread reads stale data from the DB after a concurrent writer updated the DB but before the cache was invalidated, then repopulates the cache with the now-stale value
   - Explanation: This is a read-after-invalidate race: Thread A starts reading the old value from the DB just before Thread B writes the new value and invalidates the cache; Thread A then finishes and repopulates the cache with its now-stale read, overwriting the invalidation. Mitigations include cache versioning/write-through updates, short TTLs, or invalidate-then-recheck patterns.

36. Scenario: A nightly batch job is deployed on 3 load-balanced instances, and all 3 run it simultaneously at midnight, causing duplicate processing. Standard fix?
   - Correct answer: Use a distributed lock (e.g., Redis lock or a database-based lease) that only one instance can acquire before running the scheduled job, so other instances skip execution
   - Explanation: Since all instances are identical and share the same schedule, only a coordination mechanism external to any single instance can guarantee single execution. A distributed lock/lease (Redis `SETNX` with expiry, a database row-based lease, or a scheduler-aware library) lets exactly one instance "win" the lock and run the job while the others detect it's already held and skip.

37. Scenario: A `Parallel.ForEach` loop shares one injected `DbContext` across all iterations and randomly throws concurrency exceptions. Root cause and fix?
   - Correct answer: `DbContext` is not thread-safe; each parallel task must use its own `DbContext` instance (e.g., via a new scope or `DbContextFactory`), not share one instance across concurrent operations
   - Explanation: EF Core's `DbContext` explicitly documents that it is not thread-safe — it tracks a single unit of work and its internal state (change tracker, connection) cannot be safely accessed concurrently. The fix is to give each parallel task its own `DbContext` instance, typically via `IDbContextFactory<T>.CreateDbContext()` or a new DI scope per task, rather than sharing one instance.

38. Scenario: 5 simultaneous HTTP calls all get 401 at once, and each independently refreshes the auth token, invalidating tokens from 'losing' calls. Standard client-side fix?
   - Correct answer: Use a single in-flight refresh promise/task (a mutex or 'refresh-in-progress' flag with subscribers) so concurrent 401s trigger only one token refresh, and all pending requests await that single result
   - Explanation: This is a classic "thundering herd on refresh" race. The standard Angular HTTP interceptor pattern uses a shared `BehaviorSubject`/flag: the first 401 triggers the refresh call and sets an "in progress" state; subsequent concurrent 401s see the in-progress refresh and simply wait on (subscribe to) its result instead of calling refresh themselves, then all retry their original request with the single new token.

39. Scenario: A WebSocket client reconnects from both a 'closed' event and a manual retry click nearly simultaneously, opening two duplicate connections. Best fix?
   - Correct answer: Guard reconnect logic with a state flag/mutex so only one reconnect attempt is in flight at a time, and ignore/cancel duplicate concurrent reconnect triggers
   - Explanation: This is a check-then-act race on connection state triggered by two independent event sources. Introducing a connection-state guard (e.g., an `isConnecting`/`isConnected` flag checked and set atomically before initiating a new connection) ensures only the first trigger actually opens a new socket, and the second is a no-op.

40. Scenario: `Parallel.ForEach` calls `results.Add(item)` on a shared `List<T>` from each iteration, occasionally throwing/corrupting the list. Correct fix?
    - Correct answer: Replace the shared plain `List<T>` with a thread-safe collection (e.g., `ConcurrentBag<T>`) or aggregate via thread-local storage, instead of calling `.Add()` on a shared `List<T>` from multiple threads
    - Explanation: `List<T>` is not thread-safe for concurrent writes — its internal array resize/index bookkeeping can corrupt state when multiple threads call `.Add()` simultaneously. The correct fix is a thread-safe alternative (`ConcurrentBag<T>`, `ConcurrentQueue<T>`) or, for best performance, the `Parallel.ForEach` overload that accumulates into thread-local storage and merges results at the end, avoiding contention on a single shared collection entirely.

41. Scenario: A Web API becomes very slow under moderate load because many actions call `.Result` on async DB calls. What is happening?
    - Correct answer: Blocking calls (`.Result`/`.Wait()`) tie up worker threads while waiting, so under load the pool can't grow fast enough, causing cascading slowdowns
    - Explanation: Each blocked thread sits idle waiting on I/O instead of being released back to the pool. As load increases, the ThreadPool must inject new threads slowly (its growth algorithm adds threads gradually, roughly one every ~1-2 seconds under sustained demand) to keep up, causing request queuing and increasing latency — the classic ThreadPool starvation from sync-over-async. The real fix is to use `await` all the way through instead of blocking.

42. Scenario: A service creates `new HttpClient()` per call, and under load starts throwing socket exhaustion errors. What is the fix?
    - Correct answer: Reuse a single `HttpClient` (or use `IHttpClientFactory`) instead of creating a new one per request, which under load exhausts sockets
    - Explanation: Each `HttpClient` instance owns its own connection pool/handler, and disposing it doesn't immediately release the underlying TCP sockets (they linger in `TIME_WAIT`). Creating one per request under concurrent load exhausts available ports/sockets. The fix is to reuse a single long-lived instance or, preferably, use `IHttpClientFactory`, which manages pooled `HttpMessageHandler`s correctly while still handling DNS changes.

43. Scenario: `forkJoin([callA$, callB$, callC$])` loads dashboard data; if just one call fails, the whole dashboard shows nothing. Why?
    - Correct answer: `forkJoin` immediately errors out and unsubscribes from all other inner observables the moment any one errors, so already-succeeded results are lost
    - Explanation: `forkJoin` propagates the first error immediately and unsubscribes from the remaining sources, discarding any values already received from calls that had succeeded. To show partial data gracefully, each inner observable should `catchError` and return a fallback/sentinel value so `forkJoin` always completes successfully with a mix of real and fallback results.

44. Scenario: After an outage, all clients retry at the exact same fixed 1-second interval, and the synchronized spike prevents recovery. Standard fix?
    - Correct answer: Add exponential backoff with jitter to retries, so failed clients don't all retry at synchronized intervals and overwhelm the recovering service again
    - Explanation: Fixed-interval retries across many clients synchronize into repeated traffic spikes ("thundering herd"), which can prevent a recovering service from ever staying up long enough to drain its backlog. Exponential backoff spreads retries out over increasing intervals, and adding random jitter desynchronizes clients from each other, smoothing the retry load into something the recovering service can actually absorb.

45. Scenario: `for (var i=0;i<3;i++){ setTimeout(()=>console.log(i),100); }` logs `3,3,3` instead of `0,1,2`. What causes this?
    - Correct answer: Using `var` in the loop creates one shared binding captured by all closures, so by the time callbacks run, the variable holds its final value
    - Explanation: `var` is function-scoped, not block-scoped, so there's a single `i` variable shared by all three closures. By the time the `setTimeout` callbacks actually run (after the loop has finished and `i` reached 3), they all read that same final value. Switching to `let` (block-scoped, a fresh binding per iteration) fixes it — this is a well-known async/closure gotcha, not a true race condition, but it's often confused with one in interviews.

46. Scenario: A component manually `.subscribe()`s in `ngOnInit` but never unsubscribes; after navigating away/back repeatedly, the app slows and errors. Root cause?
    - Correct answer: The component subscribes to a long-lived Observable without unsubscribing on destroy, so the subscription keeps running and leaking memory after the component is gone
    - Explanation: Angular does **not** automatically unsubscribe from manual `.subscribe()` calls (only the `async` pipe does this automatically). Each navigation creates a new component and a new subscription, but the old ones keep running, holding references to destroyed components (preventing garbage collection) and potentially executing logic against a torn-down view — the standard fixes are `takeUntil(this.destroy$)`, the `async` pipe, or Angular's `takeUntilDestroyed()`.

47. Scenario: A page-view counter is a static in-memory variable per instance behind a load balancer, so the true total is undercounted. Right fix?
    - Correct answer: Use an atomic increment on a centralized store (e.g., Redis `INCR`) or a CRDT counter, rather than each instance keeping its own in-memory counter
    - Explanation: `Interlocked.Increment` and in-memory statics only coordinate within a single process — they have no visibility across separate load-balanced instances/machines, so each instance's counter reflects only the traffic it personally received. Centralizing the counter in a shared, atomically-updatable store (Redis `INCR`, a database sequence, or a CRDT-based counter for eventual consistency) gives a correct global total.

48. Scenario: A `CanActivate` guard checks an auth flag not yet loaded from an async call, briefly showing a protected page before redirecting. Underlying issue?
    - Correct answer: The guard's async auth check resolves after navigation already started rendering; ensure the guard's Observable/Promise is awaited before navigation proceeds
    - Explanation: If the guard reads a flag that's still being populated by an in-flight async call (rather than returning/awaiting the actual Observable/Promise that resolves the auth check), the router may proceed with a stale "not yet determined" value. `CanActivate` supports returning an `Observable<boolean>`/`Promise<boolean>` — the guard should properly wait on the real async source of truth (e.g., `authService.isAuthenticated$.pipe(take(1))`) rather than racing against it.

49. Scenario: An endpoint loops over 100 orders, lazy-loading `order.Customer.Name` each time, causing 101 total DB queries. Best fix?
    - Correct answer: Lazy-loading each related entity in a loop triggers a separate DB round-trip per item; fix by eager-loading (`Include`) or batching the query upfront
    - Explanation: This is the classic N+1 query problem — 1 query to fetch orders, then N additional queries as each `order.Customer` is lazily loaded on access. The fix is to eager-load the related data upfront with `.Include(o => o.Customer)` so EF Core issues a single (or few) efficient joined queries instead of N round-trips — parallelizing the N queries with `Task.Run` would just add concurrency overhead and potential `DbContext` thread-safety violations on top of the same underlying inefficiency.

50. Scenario: A flaky network causes a client-side payment timeout+retry, but the original request succeeded server-side, double-charging the customer. Best fix?
    - Correct answer: Require the client to send a unique idempotency key; the server stores/checks it so a retried request returns the original result instead of charging twice
    - Explanation: The client genuinely cannot distinguish "request never reached the server" from "response never reached the client" after a timeout, so retries are unavoidable and even desirable for reliability. The correct architectural fix is idempotency keys: the client generates a unique key per logical operation, sends it with every retry, and the server recognizes a repeat key and returns the original result instead of re-executing the charge — this is the standard approach used by real payment gateways (e.g., Stripe's `Idempotency-Key` header).

51. Scenario: A Kafka consumer group occasionally reprocesses the same messages after a consumer restarts or scales. Root cause and fix?
    - Correct answer: During a consumer group rebalance, a partition can be reassigned before the prior consumer commits its offset, causing reprocessing; design consumers to be idempotent
    - Explanation: Kafka's default delivery semantics are at-least-once: if a consumer crashes or a rebalance reassigns a partition after messages were processed but before the offset commit was acknowledged, the new owner starts from the last committed offset and reprocesses those messages. Since rebalances and restarts are a normal part of operating a consumer group, consumers must be designed to be idempotent (or use transactional/exactly-once semantics where supported) rather than assuming single delivery.

52. Scenario: A user occasionally misses the first notification right after joining a SignalR group. Likely cause?
    - Correct answer: Join-group and send-to-group calls can race if membership isn't awaited/coordinated, so a client misses a message sent just after joining
    - Explanation: `Groups.AddToGroupAsync` is itself asynchronous, and if the server-side flow that triggers a group send doesn't happen-after the join completing (e.g., the join and an almost-simultaneous broadcast are handled on different connections/timings), the client can miss the message. The fix is to ensure any "welcome"/catch-up data is sent only after the join truly completes (await it), and to consider a fetch-current-state-after-join pattern for critical data rather than relying purely on the timing of live pushes.

53. Scenario: A GraphQL query resolving 50 authors, each independently resolving `books`, triggers 51 DB calls per request. Standard fix?
    - Correct answer: Use a batching/caching loader (DataLoader pattern) that coalesces per-field resolver calls within a request into one batched query
    - Explanation: GraphQL's per-field resolver model makes N+1 queries especially easy to introduce, since each author's `books` resolver fires independently with no awareness of its siblings. The DataLoader pattern batches all the individual keys requested during a single event-loop tick into one combined query (e.g., `WHERE authorId IN (...)`) and caches results per request, collapsing 51 queries down to 2.

54. Scenario: A feature flag toggled mid-day causes a multi-step request to apply old logic in step 1 but new logic in step 3. Root cause and fix?
    - Correct answer: A request may read the flag's old value early and new value later if re-checked mid-flow, causing inconsistency; read the flag once per request/transaction
    - Explanation: Feature flag evaluation can change mid-flight for any request that spans multiple checks over a non-trivial duration, since the flag's backing store can be updated by an operator or another process at any time. The fix is to snapshot the flag's value once at the start of the logical unit of work (request/transaction/saga) and pass that single evaluated value through all subsequent steps, rather than re-evaluating the live flag at each step.

55. Scenario: After migrating to zoneless Angular, the UI stops updating after HTTP responses via manual subscriptions. Why, and what's the fix?
    - Correct answer: In zoneless Angular, change detection must be explicitly triggered (signals, `markForCheck`, etc.) since there's no zone.js patching to auto-detect async completions
    - Explanation: Zone.js's whole job was patching async APIs so Angular automatically knew when to re-render. Removing it (zoneless mode) means Angular no longer gets that automatic signal — components must use Signals (which notify Angular reactively), the `async` pipe, or explicit `ChangeDetectorRef.markForCheck()` calls after manual subscriptions complete, otherwise the view silently goes stale even though the underlying data updated correctly.

56. Scenario: During a network partition, a cache cluster's two halves each promote their own primary and keep accepting writes. What is this called?
    - Correct answer: A network partition causes two cache nodes to each believe they are primary and accept writes independently, leading to diverging data
    - Explanation: This is the classic "split-brain" problem: without a reliable way to distinguish "the other node is down" from "I'm just cut off from it," each partition can independently conclude it should take over as primary. Both sides then accept writes in isolation, and when the partition heals, the divergent histories must be reconciled (often requiring manual intervention or CRDT-style automatic merge) — which is exactly why quorum-based consensus (majority voting) is used to prevent more than one side from becoming primary in the first place.

57. Scenario: `while (!_stopRequested) { DoWork(); }` where `_stopRequested` is a plain `bool` set from another thread, and it sometimes never stops. Why?
    - Correct answer: Without `volatile` (or a memory barrier), the JIT/CPU may cache the flag in a register or reorder instructions, so the worker may never observe it being set
    - Explanation: Without a `volatile` modifier or another synchronization primitive, the JIT compiler is free to optimize the loop by caching `_stopRequested` in a CPU register (since, from its single-thread-only analysis, the value "never changes" inside the loop body), so the worker thread may never re-read the actual memory location where another thread wrote `true`. Marking the field `volatile` (or using `Interlocked`/a lock) ensures the write is properly published and the read is not stale.

58. Scenario: A polling component uses `interval(5000).subscribe(...)`; after visiting 10 dashboards, 10 concurrent refreshes fire every 5s. Root cause?
    - Correct answer: The RxJS `interval` subscription keeps emitting and executing its callback forever unless explicitly unsubscribed, even after the component is destroyed
    - Explanation: `interval()` has no inherent lifecycle awareness of Angular components — a manual `.subscribe()` keeps running indefinitely until explicitly unsubscribed. Each new component instance (from re-navigating) creates a brand-new, independent subscription, and none of the old ones are ever torn down, so they all keep firing concurrently. The standard fix is `takeUntil(this.destroy$)` (emitting in `ngOnDestroy`) or Angular's `takeUntilDestroyed()` to bind the subscription's lifetime to the component's.

59. Scenario: An optimistic 'liked' UI update stays stuck showing 'liked' when the server call actually fails. What should the design include?
    - Correct answer: Apply the change to local UI state immediately, but keep the prior state to roll back to if the server request ultimately fails
    - Explanation: Optimistic updates trade correctness-on-failure for perceived responsiveness — the whole pattern requires a rollback plan. The client should snapshot the prior state before applying the optimistic change, and if the server call's error handler fires, revert to that snapshot (and typically show an error toast), rather than leaving the UI permanently showing a state the server never actually confirmed.

60. Scenario: Service A calls B calls C; when C becomes slow, A's thread pool eventually exhausts. What design principle was missing?
    - Correct answer: Set explicit per-call timeouts shorter than the caller's own SLA, combined with circuit breakers, so a slow downstream call can't cascade upward
    - Explanation: Without a sensible, tightly-scoped timeout on the B→C call, a slowdown in C ripples upward: B's threads/connections pile up waiting on C, then A's threads/connections pile up waiting on B, and the failure cascades through the whole call chain, potentially taking down unrelated functionality in A. Setting per-call timeouts (each shorter than the caller's own budget) plus circuit breakers to fail fast once a threshold of slow/failed calls is detected contains the blast radius instead of letting one slow dependency exhaust resources all the way up the chain.
