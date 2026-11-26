# Race Conditions in Software Programming

## Table of Contents

- [1. What Is a Race Condition?](#1-what-is-a-race-condition)
- [2. Simple Example](#2-simple-example)
- [3. Beginner-Friendly Real-World Analogy](#3-beginner-friendly-real-world-analogy)
- [4. Where Race Conditions Commonly Appear](#4-where-race-conditions-commonly-appear)
  - [4.1 Multithreaded Code](#41-multithreaded-code)
  - [4.2 Async Code with Shared State](#42-async-code-with-shared-state)
  - [4.3 Distributed Systems](#43-distributed-systems)
  - [4.4 UI and Event-Driven Code](#44-ui-and-event-driven-code)
  - [4.5 Database Writes](#45-database-writes)
- [5. Race Conditions in Async Code vs Multithreaded Code](#5-race-conditions-in-async-code-vs-multithreaded-code)
  - [5.1 Async Code Example](#51-async-code-example)
  - [5.2 Multithreaded Code Example](#52-multithreaded-code-example)
- [6. C#-Specific Explanation](#6-c-specific-explanation)
- [7. Common Symptoms](#7-common-symptoms)
- [8. Ways to Avoid or Mitigate Race Conditions](#8-ways-to-avoid-or-mitigate-race-conditions)
  - [8.1 Minimize Shared Mutable State](#81-minimize-shared-mutable-state)
  - [8.2 Use Synchronization Primitives Correctly](#82-use-synchronization-primitives-correctly)
  - [8.3 Use Atomic Operations](#83-use-atomic-operations)
  - [8.4 Prefer Thread-Safe Data Structures](#84-prefer-thread-safe-data-structures)
  - [8.5 Enforce Clear Ownership](#85-enforce-clear-ownership)
  - [8.6 Use Transactions and Isolation in Databases](#86-use-transactions-and-isolation-in-databases)
  - [8.7 Make Operations Idempotent When Possible](#87-make-operations-idempotent-when-possible)
  - [8.8 Use Ordering and Coordination Mechanisms](#88-use-ordering-and-coordination-mechanisms)
  - [8.9 Avoid Check-Then-Act Patterns](#89-avoid-check-then-act-patterns)
  - [8.10 Design for Concurrency Explicitly](#810-design-for-concurrency-explicitly)
  - [8.11 Test Under Concurrency Stress](#811-test-under-concurrency-stress)
  - [8.12 Use Timeouts and Defensive Validation](#812-use-timeouts-and-defensive-validation)
- [9. Practical Rule of Thumb](#9-practical-rule-of-thumb)
- [10. Interview MCQs and Answer Explanations](#10-interview-mcqs-and-answer-explanations)
  - [10.1 Beginner Level](#101-beginner-level)
  - [10.2 Senior Level](#102-senior-level)
  - [10.3 Architect Level](#103-architect-level)
  - [10.4 Harder Round 2](#104-harder-round-2)

## 1. What Is a Race Condition?

A race condition is a bug that happens when the correctness of a program depends on the timing or ordering of concurrent operations, and at least one possible interleaving produces the wrong result.

## 2. Simple Example

Two threads update the same counter:

```text
counter = 5

Thread A reads 5
Thread B reads 5
Thread A writes 6
Thread B writes 6
```

The expected result was `7`, but the actual result is `6`. Both threads raced on shared state.

## 3. Beginner-Friendly Real-World Analogy

Imagine two cashiers updating the stock count for the same product at the same time.

The shelf system says there are `10` items left.

1. Cashier A reads `10`.
2. Cashier B also reads `10`.
3. Cashier A sells one item and updates the count to `9`.
4. Cashier B sells one item and also updates the count to `9`.

The correct count should be `8`, but the system shows `9`. Nobody intended to be wrong, but the updates happened in the wrong order without coordination. That is the essence of a race condition.

## 4. Where Race Conditions Commonly Appear

### 4.1 Multithreaded Code

Multiple threads reading and writing the same memory without correct coordination.

### 4.2 Async Code with Shared State

Asynchronous operations can still race when they mutate the same object, cache, or in-memory state.

### 4.3 Distributed Systems

Competing updates across services, workers, or nodes can create ordering conflicts and lost updates.

### 4.4 UI and Event-Driven Code

Multiple events or callbacks may update the same view model or state store in an unexpected order.

### 4.5 Database Writes

Concurrent reads and writes without correct transaction boundaries can produce conflicting or lost changes.

## 5. Race Conditions in Async Code vs Multithreaded Code

Race conditions can happen in both async code and multithreaded code, but the mechanics differ.

In async code, the problem usually comes from multiple operations yielding and resuming later while sharing mutable state.

In multithreaded code, the problem usually comes from multiple threads executing at the same time on different cores and touching the same memory.

### 5.1 Async Code Example

```csharp
var balance = 100;

async Task WithdrawAsync(int amount)
{
  var current = balance;
  await Task.Delay(10);
  balance = current - amount;
}

await Task.WhenAll(WithdrawAsync(30), WithdrawAsync(50));
```

Both operations may read the same initial value of `balance` before either writes back. That can produce the wrong final balance.

### 5.2 Multithreaded Code Example

```csharp
var counter = 0;

void Increment()
{
  for (var i = 0; i < 100000; i++)
  {
    counter++;
  }
}

var thread1 = new Thread(Increment);
var thread2 = new Thread(Increment);

thread1.Start();
thread2.Start();
thread1.Join();
thread2.Join();
```

The expected result is `200000`, but the actual result may be lower because `counter++` is not atomic.

Atomic means an operation happens as one indivisible step: no other thread can observe it halfway through or interfere in the middle.

`counter++` looks like one statement, but it is usually three separate actions:

1. Read the current value of `counter`
2. Add 1 to that value
3. Write the new value back

If two threads do that at the same time, they can both read the same old value before either one writes the update.

Example:

- `counter` starts at `5`
- Thread A reads `5`
- Thread B reads `5`
- Thread A computes `6` and writes `6`
- Thread B also computes `6` and writes `6`

Two increments happened, but the final value is `6` instead of `7`. One update was lost.

So when people say `counter++` is not atomic, they mean the increment is not guaranteed to happen as one protected, uninterruptible action. It is a read-modify-write sequence, and concurrent threads can interleave inside that sequence.

In C#, if you need an atomic increment for a shared integer, use `Interlocked.Increment`.

## 6. C#-Specific Explanation

In C#, race conditions often appear when multiple threads or tasks access the same shared field, property, collection, or object without coordination.

Common C# sources of race conditions include:

- `Task.WhenAll` over methods that mutate shared state
- `Parallel.ForEach` updating shared variables
- Multiple requests in ASP.NET code touching shared static state
- Background services and timers updating the same resource
- Non-thread-safe collections such as `List<T>` or `Dictionary<TKey, TValue>` used from multiple threads

For example:

```csharp
public class CounterService
{
  private int _count;

  public void Increment()
  {
    _count++;
  }

  public int Count => _count;
}
```

If `Increment` is called concurrently from multiple threads, `_count` can lose updates because `_count++` is read-modify-write, not a single atomic step.

Safer C# approaches include:

- `lock` for critical sections
- `Interlocked.Increment(ref _count)` for simple counters
- `ConcurrentDictionary<TKey, TValue>` instead of `Dictionary<TKey, TValue>` when multiple threads write
- avoiding shared mutable `static` state in web applications

## 7. Common Symptoms

- Intermittent failures
- Data corruption or lost updates
- Behavior that fails only under load
- Bugs that disappear when logging or debugging changes timing

## 8. Ways to Avoid or Mitigate Race Conditions

### 8.1 Minimize Shared Mutable State

Use immutable data, pure functions, or message passing where possible. The fewer things multiple workers can change, the fewer races you can have.

Example:

Instead of two threads updating the same `OrderStatus` object, create a new immutable status record and replace the whole reference in one controlled place.

### 8.2 Use Synchronization Primitives Correctly

Locks, mutexes, semaphores, monitors, and critical sections ensure that only one execution path can modify a protected resource at a time.

Example:

```csharp
private readonly object _lock = new();
private int _count;

public void Increment()
{
  lock (_lock)
  {
    _count++;
  }
}
```

Without the `lock`, concurrent calls can lose updates.

### 8.3 Use Atomic Operations

For simple shared values like counters or flags, atomic increment, compare-and-swap, or atomic exchange is often safer and faster than manual locking.

Example:

```csharp
Interlocked.Increment(ref _count);
```

This performs the increment as one atomic operation.

### 8.4 Prefer Thread-Safe Data Structures

Concurrent collections, synchronized queues, and lock-free structures reduce the chance of incorrect manual coordination.

Example:

If many worker threads add items to a shared collection, use `ConcurrentQueue<T>` instead of `List<T>`. A plain `List<T>` can become corrupted or throw exceptions when written concurrently.

### 8.5 Enforce Clear Ownership

Make one thread, process, or service the sole writer for a piece of state. Others can read or send requests instead of writing directly.

Example:

A background worker owns updates to an in-memory cache. Other threads do not modify the cache directly; they submit requests through a queue.

### 8.6 Use Transactions and Isolation in Databases

Race conditions are not limited to in-memory code. Use transactions, row or version checks, unique constraints, and appropriate isolation levels to prevent lost or conflicting updates.

Example:

Two users edit the same customer record. Use a row version column so the second update fails if the record changed after it was read.

A row version column is a concurrency token in the database. It lets the system detect whether a row changed between the time you read it and the time you try to save it.

The flow works like this:

1. User A reads a customer row.
2. User B reads the same customer row.
3. Both users receive the current `RowVersion` value with the data, for example `42`.
4. User A updates the record first. The database saves the change and also changes `RowVersion`, for example from `42` to `43`.
5. User B now tries to save their update, but they still send the old `RowVersion = 42`.
6. The update statement says, in effect: update this row only if the row version is still `42`.
7. The database sees the row is now version `43`, so the update affects `0` rows.
8. The application treats that as a concurrency conflict and tells User B the record changed since they loaded it.

Conceptually, the SQL looks like this:

```sql
UPDATE Customer
SET Name = @Name
WHERE Id = @Id
  AND RowVersion = @OriginalRowVersion
```

If another update already changed the row, the `WHERE` clause no longer matches, so the second write is rejected.

Why this helps:

- It prevents silent overwrites.
- It does not block readers with long locks.
- It is an optimistic concurrency approach: assume conflicts are rare, but detect them when they happen.

### 8.7 Make Operations Idempotent When Possible

In distributed systems, retries happen. If the same operation can safely run multiple times, timing issues become less dangerous.

Example:

If a payment API receives the same request twice, use an idempotency key so the second request returns the first result instead of charging the customer again.

### 8.8 Use Ordering and Coordination Mechanisms

Examples include job queues, actor models, event serialization, and single-threaded executors for sensitive workflows.

Example:

Instead of letting five workers update the same account balance directly, place balance updates into a single queue and process them one at a time.

### 8.9 Avoid Check-Then-Act Patterns

This is a classic race:

- Check whether a value exists
- Assume it still has the same state later

Another thread may change it in between. Combine the check and update into one atomic or transactional operation.

Example:

Bad pattern:

```csharp
if (!_users.ContainsKey(id))
{
  _users[id] = user;
}
```

Another thread might add the same key between the check and the write. A safer approach is `TryAdd` on `ConcurrentDictionary<TKey, TValue>`.

### 8.10 Design for Concurrency Explicitly

Document invariants, define what can run in parallel, and identify which state needs protection. Race conditions are often design bugs, not just coding mistakes.

Example:

Write down that only the subscription processing service is allowed to transition a subscription from `Pending` to `Active`. That prevents multiple services from racing to perform the same state change.

### 8.11 Test Under Concurrency Stress

Use repeated parallel tests, fuzzing, stress tests, and tools like thread sanitizers or race detectors. These bugs often do not show up in normal unit tests.

Example:

Run a test that starts 100 tasks, each incrementing the same counter 10,000 times. If the final result varies across runs, the code likely has a race.

### 8.12 Use Timeouts and Defensive Validation

These do not prevent races directly, but they help contain damage and make failures observable instead of silent.

Example:

If two services both try to process the same message, a timeout plus duplicate-message detection can stop the system from hanging forever and can surface the conflict in logs or metrics.

## 9. Practical Rule of Thumb

If multiple execution paths can read and write the same state, assume a race is possible until you can explain exactly why it is safe.

## 10. Interview MCQs and Answer Explanations

This section provides 30 interview-style multiple-choice questions for race conditions and concurrency. Some questions are code-based. Each question is worth 1 point.

### 10.1 Beginner Level

1. What is the main problem in a race condition?
   - A. Code runs too slowly
   - B. Memory usage grows unexpectedly
   - C. Program correctness depends on timing of concurrent operations
   - D. The compiler cannot optimize the code
   - Correct answer: C
   - Explanation: A race condition is about correctness depending on how concurrent operations interleave, not about performance or compilation.

2. In `counter++`, which step happens first?
   - A. Write the new value
   - B. Read the current value
   - C. Lock the variable
   - D. Flush CPU cache
   - Correct answer: B
   - Explanation: The increment starts by reading the current value. The write happens later, which is why other threads can interfere in the middle.

3. Which C# API provides an atomic increment for an `int`?
   - A. `Task.Run`
   - B. `lockfree++`
   - C. `Monitor.Pulse`
   - D. `Interlocked.Increment`
   - Correct answer: D
   - Explanation: `Interlocked.Increment` performs the increment as one atomic operation for supported primitive values.

4. What is the best plain-English meaning of atomic?
   - A. It happens as one indivisible step
   - B. It always runs on one CPU core
   - C. It uses `async` or `await` internally
   - D. It is faster than all alternatives
   - Correct answer: A
   - Explanation: Atomic means the operation cannot be observed halfway through and cannot be interrupted as separate visible sub-steps.

5. Which shared structure is unsafe for concurrent writes unless protected?
   - A. `ConcurrentQueue<T>`
   - B. `ImmutableArray<T>`
   - C. `List<T>`
   - D. `string`
   - Correct answer: C
   - Explanation: `List<T>` is not safe for concurrent modification. It can lose updates or become corrupted when multiple threads write to it.

6. Code question: What is the likely issue here?

```csharp
private int _count;
public void Inc() { _count++; }
```

   - A. It leaks memory
   - B. It can lose updates if called concurrently
   - C. It always throws `NullReferenceException`
   - D. It deadlocks immediately
   - Correct answer: B
   - Explanation: `_count++` is a read-modify-write sequence. Concurrent calls can interleave and overwrite one another.

7. Which symptom often hints at a race condition?
   - A. The program fails every time in the same place
   - B. The program only fails when offline
   - C. The code does not compile
   - D. Intermittent failures under load
   - Correct answer: D
   - Explanation: Race conditions often appear only under certain timing conditions, so they tend to be intermittent and more visible under load.

8. What does a `lock` primarily do?
   - A. Allows only one execution path into a critical section at a time
   - B. Makes all operations async
   - C. Removes all bugs automatically
   - D. Prevents the garbage collector from running
   - Correct answer: A
   - Explanation: A `lock` provides mutual exclusion so only one thread can execute the protected code at a time.

9. Code question: Two tasks both read `balance = 100`, `await`, then write back updated values. What kind of bug is this?
   - A. Syntax error
   - B. Type mismatch
   - C. Lost update race condition
   - D. Memory leak
   - Correct answer: C
   - Explanation: Both tasks can read the same original value and then overwrite each other's change, producing a lost update.

10. Which approach best reduces race-condition risk?
  - A. Use more global mutable state
  - B. Minimize shared mutable state
  - C. Add random delays
  - D. Rename variables clearly
  - Correct answer: B
  - Explanation: The less mutable state that is shared across execution paths, the fewer opportunities there are for conflicting updates.

### 10.2 Senior Level

11. Which pattern is most vulnerable to a race without coordination?
  - A. Pure function over local values
  - B. Read-only cache lookup
  - C. Immutable record creation
  - D. Check-then-act on shared state
  - Correct answer: D
  - Explanation: Check-then-act is dangerous because another thread can change the state after the check but before the write.

12. Code question: Which replacement is safest for a shared increment-heavy counter?
  - A. `count = count + 1`
  - B. `Interlocked.Increment(ref count)`
  - C. `count += 1` inside `Task.Run`
  - D. `volatile int count`
  - Correct answer: B
  - Explanation: `Interlocked.Increment` gives atomic increments. `volatile` improves visibility but does not make compound operations atomic.

13. In optimistic concurrency, why does the second update fail?
  - A. The concurrency token no longer matches the original read
  - B. The database forbids all concurrent reads
  - C. The second transaction always deadlocks
  - D. The API deletes stale requests
  - Correct answer: A
  - Explanation: The second update is rejected because the row changed after it was read, so the saved token or row version no longer matches.

14. What does `volatile` guarantee for a field in C#?
  - A. Atomic read-modify-write
  - B. Mutual exclusion
  - C. Visibility and ordering constraints, not compound atomicity
  - D. Prevention of context switches
  - Correct answer: C
  - Explanation: `volatile` helps ensure threads see the latest value, but it does not make `x++` or other multi-step operations atomic.

15. Code question: What is the flaw here?

```csharp
if (!_users.ContainsKey(id))
{
  _users[id] = user;
}
```

  - A. It is too slow to compile
  - B. Check and write are separate, so another thread can interleave
  - C. `Dictionary` always rejects writes
  - D. The `if` statement is illegal in C#
  - Correct answer: B
  - Explanation: Another thread can insert the same key after the `ContainsKey` check and before the assignment. That makes the pattern race-prone.

16. Which test strategy is best for surfacing races?
  - A. Only single-threaded unit tests
  - B. Snapshot tests of HTML
  - C. One manual run in debug mode
  - D. Repeated concurrency stress tests
  - Correct answer: D
  - Explanation: Repetition under concurrency increases the chance of exposing timing-sensitive failures that normal tests may miss.

17. What is the strongest architectural benefit of immutable data in concurrent code?
  - A. It removes write-write races on that data
  - B. It guarantees zero allocations
  - C. It makes locks faster
  - D. It forces database serialization
  - Correct answer: A
  - Explanation: If data cannot be changed after creation, threads cannot race to modify the same instance.

18. Code question: `await Task.WhenAll(UpdateA(), UpdateB());` Both methods mutate the same shared object. What should you suspect first?
  - A. SQL injection
  - B. Namespace collision
  - C. Async race on shared mutable state
  - D. JIT compiler bug
  - Correct answer: C
  - Explanation: `async` does not remove concurrency risk. If both operations touch the same mutable object, interleaving can still break correctness.

19. Which choice best fits when many threads enqueue work but one worker should own state changes?
  - A. Let every thread write directly
  - B. Queue-based single-writer ownership
  - C. Add `Thread.Sleep` between writes
  - D. Use a static `List<T>`
  - Correct answer: B
  - Explanation: A queue plus a single writer simplifies coordination and removes multi-writer races on the protected state.

20. Which statement about locks is most accurate?
  - A. Locks are always unnecessary if code is async
  - B. Locks fix distributed concurrency automatically
  - C. Locks only matter for databases
  - D. Locks can protect critical sections but must be scoped carefully
  - Correct answer: D
  - Explanation: Locks are useful within a process, but they must guard the right shared state and be used carefully to avoid contention or deadlocks.

### 10.3 Architect Level

21. For a high-write distributed workflow, what most directly prevents duplicate side effects on retries?
  - A. Longer request timeouts
  - B. More logging
  - C. Idempotency keys
  - D. Larger thread pools
  - Correct answer: C
  - Explanation: Idempotency keys let the system recognize duplicate requests and return the original outcome instead of repeating the side effect.

22. Which ownership model most cleanly reduces races around a hot aggregate?
  - A. Single writer per aggregate or partition
  - B. Any node may update directly
  - C. Every service keeps its own mutable copy
  - D. Retry until it works
  - Correct answer: A
  - Explanation: Restricting mutation to one owner per aggregate or partition greatly reduces coordination complexity.

23. Code or system question: Multiple services update the same customer row. What is the best first control at the persistence boundary?
  - A. Random backoff only
  - B. Cache the writes client-side
  - C. Disable concurrent requests globally
  - D. Concurrency token or row-version check
  - Correct answer: D
  - Explanation: A concurrency token detects conflicting writes where they matter most: at the database write boundary.

24. In an event-driven system, what reduces ordering races for sensitive entity updates?
  - A. Fan out to more consumers per key
  - B. Per-key ordered processing
  - C. Ignore duplicate events
  - D. Prefer shared static state
  - Correct answer: B
  - Explanation: If updates for the same key are processed in order by a single stream or partition, ordering conflicts are much easier to control.

25. Which design review question best exposes concurrency risk early?
  - A. Which color should the UI use?
  - B. Can we reduce class count?
  - C. Who is allowed to mutate this state, and when?
  - D. Should we rename the repository?
  - Correct answer: C
  - Explanation: Concurrency bugs usually center on shared mutable state, so identifying ownership and mutation rules early is critical.

26. When conflicts are rare but overwrites must be detected, which strategy fits best?
  - A. Optimistic concurrency
  - B. No validation at save time
  - C. Shared mutable static cache
  - D. Polling every millisecond
  - Correct answer: A
  - Explanation: Optimistic concurrency assumes most updates will not conflict, but it validates at save time so conflicting writes are rejected.

27. Code or system question: A read model is eventually consistent. What should writers avoid assuming?
  - A. That immutable messages help
  - B. That retries can happen
  - C. That idempotency matters
  - D. That a just-written value will be instantly visible everywhere
  - Correct answer: D
  - Explanation: Eventual consistency means propagation takes time. Writers and callers must not depend on immediate global visibility.

28. Which database feature helps reject conflicting inserts for a supposed-unique business key?
  - A. Lower isolation level
  - B. Unique constraint or unique index
  - C. More application threads
  - D. Manual delay before save
  - Correct answer: B
  - Explanation: A unique constraint enforces the rule at the database level, preventing duplicates even when concurrent requests race.

29. What is the main reason to separate command ownership from query access in concurrent systems?
  - A. To maximize shared writable state
  - B. To eliminate all need for validation
  - C. To constrain mutation paths and reduce coordination complexity
  - D. To force synchronous APIs only
  - Correct answer: C
  - Explanation: Fewer mutation paths make it easier to reason about consistency, ownership, and ordering.

30. Which architecture pattern best fits strict sequential updates for the same entity key?
  - A. Actor or partitioned single-threaded processor per key
  - B. Unbounded parallel writes to one row
  - C. Shared in-memory static dictionary across services
  - D. Client-side polling loop
  - Correct answer: A
  - Explanation: Actor-style or per-key single-threaded processing ensures updates for the same key are handled one at a time in a predictable order.

### 10.4 Harder Round 2

This second round is slightly harder and focuses more on subtle concurrency behavior, persistence boundaries, and architectural controls.

1. Which statement about race conditions is most precise?
   - A. They only happen in multithreaded code
   - B. They occur when correctness depends on an unsafe interleaving of concurrent operations
   - C. They are caused only by missing locks
   - D. They require distributed systems
   - Correct answer: B
   - Explanation: A race condition is fundamentally about unsafe interleavings. Threads, tasks, callbacks, and distributed workers can all produce them.

2. Code question: Why is `volatile int count; count++;` still unsafe?
   - A. `volatile` disables writes
   - B. `volatile` only works for reference types
   - C. `volatile` makes reads slower
   - D. `volatile` does not make read-modify-write atomic
   - Correct answer: D
   - Explanation: `volatile` helps with visibility and ordering, but `count++` is still multiple steps, so two writers can still collide.

3. In optimistic concurrency with a row version, what does a 0-row update usually mean?
   - A. The row changed since it was read or no longer matches the original token
   - B. The database lost the transaction log
   - C. The query always deadlocked
   - D. The ORM skipped `SaveChanges`
   - Correct answer: A
   - Explanation: The row version no longer matches the version seen during the read, so the update condition fails and no rows are updated.

4. Which bug pattern is this: read shared value, await I/O, then write based on stale local copy?
   - A. Guaranteed deadlock
   - B. Heap fragmentation
   - C. Lost update from stale read
   - D. Compile-time ambiguity
   - Correct answer: C
   - Explanation: The write is based on an old snapshot of state. Another operation may have already changed the shared value.

5. Code question: Which is the safest insertion API for concurrent add-if-absent semantics?
   - A. `Dictionary.ContainsKey` plus indexer set
   - B. `ConcurrentDictionary.TryAdd`
   - C. `List.Add` inside `Task.Run`
   - D. `HashSet.Add` without coordination
   - Correct answer: B
   - Explanation: `TryAdd` combines the existence check and insertion into one concurrency-safe operation.

6. If adding more logging makes the bug disappear, what should you suspect?
   - A. The compiler is broken
   - B. The bug is impossible now
   - C. The GC is corrupting memory
   - D. Timing-sensitive concurrency behavior
   - Correct answer: D
   - Explanation: Logging changes timing. If that changes the result, a race condition or similar timing-sensitive bug is a strong suspect.

7. Which design most reduces contention for a hot account balance aggregate?
   - A. Route updates for that key through a single owner or partition
   - B. Let all services update it directly with retries
   - C. Cache writes client-side only
   - D. Use `Thread.Sleep` before each update
   - Correct answer: A
   - Explanation: Constraining writes for one hot key to one owner simplifies ordering and consistency.

8. Code question: Two threads both execute `if (state == Open) state = Closed;`. What is the core issue?
   - A. `state` comparisons are illegal
   - B. `Closed` is not a valid enum
   - C. Check-then-act on shared mutable state
   - D. Assignment to enums is always atomic and safe
   - Correct answer: C
   - Explanation: The check and the update are separate steps. Another thread can change `state` between them.

9. Which database feature best prevents duplicate rows for a unique business identifier under concurrency?
   - A. Read committed isolation alone
   - B. Client-side validation only
   - C. Retry loops without constraints
   - D. Unique constraint or unique index
   - Correct answer: D
   - Explanation: The database must enforce uniqueness at the final write boundary to close the race completely.

10. Which testing approach gives the best chance of reproducing a race?
  - A. One careful manual run
  - B. High-repeat parallel stress with assertions on invariants
  - C. Only static code formatting checks
  - D. Snapshot tests of UI markup
  - Correct answer: B
  - Explanation: Repetition plus concurrency plus invariant checks exposes timing-sensitive correctness failures more reliably.

11. In a distributed system, what is idempotency primarily protecting against?
  - A. Cache eviction
  - B. CPU starvation
  - C. Duplicate side effects from retries or replays
  - D. Null references
  - Correct answer: C
  - Explanation: Idempotency ensures that replaying the same request does not apply the same effect multiple times.

12. Code question: What is the strongest reason `Interlocked.CompareExchange` is useful?
  - A. It can perform conditional atomic updates without a full lock
  - B. It serializes all database writes
  - C. It replaces `async` and `await`
  - D. It guarantees fairness across threads
  - Correct answer: A
  - Explanation: Compare-and-swap lets you update a value only if it still equals the expected old value, atomically.

13. What is the main concurrency risk of exposing mutable static state in a web app?
  - A. It cannot be read by requests
  - B. It only affects one user session
  - C. It improves isolation too much
  - D. Many concurrent requests may mutate the same shared object
  - Correct answer: D
  - Explanation: Static mutable state is shared process-wide, so many requests can race on it at once.

14. Which statement about locks in async code is most accurate?
  - A. `lock` can safely contain awaited work
  - B. Synchronous locks protect only in-process critical sections and should not wrap awaits
  - C. `lock` is required for every async method
  - D. `async` automatically makes `lock` unnecessary
  - Correct answer: B
  - Explanation: `lock` is synchronous and should not span `await`. Async coordination usually needs different primitives or design choices.

15. Code question: Why is a queue plus single consumer often safer than many direct writers?
  - A. It makes reads impossible
  - B. It removes the need for any validation
  - C. It centralizes mutation order and ownership
  - D. It guarantees zero latency
  - Correct answer: C
  - Explanation: One consumer creates a clear serialization point for mutations and simplifies reasoning about order.

16. What does eventual consistency mean for readers?
  - A. A recent write may not be visible everywhere immediately
  - B. Every read is strongly serialized
  - C. Reads always fail during writes
  - D. Concurrency bugs cannot happen
  - Correct answer: A
  - Explanation: Eventual consistency trades immediate global visibility for propagation over time.

17. Which choice best describes a compare-and-swap loop?
  - A. A logging retry pattern
  - B. A database replication algorithm
  - C. A garbage collector optimization
  - D. Re-read and retry atomic conditional update until it succeeds or aborts
  - Correct answer: D
  - Explanation: A CAS loop repeatedly reads the value and retries the conditional atomic update if another writer changed it first.

18. Code question: If two handlers mutate the same in-memory cache entry in response to separate events, the first architectural question is:
  - A. Which color is the dashboard?
  - B. Who owns writes to this cache entry?
  - C. Can we rename the event?
  - D. Should we switch ORMs?
  - Correct answer: B
  - Explanation: Ownership is the first question because unclear write ownership is a root cause of many concurrency bugs.

19. Why are immutable messages helpful in concurrent pipelines?
  - A. They make the network faster
  - B. They eliminate retries
  - C. Consumers cannot race to mutate the same message instance
  - D. They guarantee FIFO delivery
  - Correct answer: C
  - Explanation: If messages are immutable, every consumer sees the same content and cannot accidentally alter shared message state.

20. What is the key purpose of defining concurrency invariants explicitly?
  - A. To state what must always remain true despite interleavings
  - B. To reduce source-control history
  - C. To increase CPU usage predictably
  - D. To avoid writing tests
  - Correct answer: A
  - Explanation: Invariants are the correctness rules that must hold no matter how operations interleave.

21. Which symptom most strongly suggests an ordering race in event-driven code?
  - A. Consistent failure on startup
  - B. Compile error in handler signature
  - C. Memory leak after hours
  - D. Correct final state depends on which callback finishes last
  - Correct answer: D
  - Explanation: If the result changes depending on callback completion order, ordering is part of correctness and may be unsafe.

22. Code question: Which persistence strategy best detects conflicting updates without long-lived locks?
  - A. Blind overwrite `UPDATE` by `Id` only
  - B. `UPDATE` by `Id` plus original version token
  - C. Remove the `WHERE` clause
  - D. Save to cache first and hope
  - Correct answer: B
  - Explanation: Matching on the original version token lets the database detect whether someone else changed the row since it was read.

23. What is the biggest flaw in using `Thread.Sleep` to 'fix' a race?
  - A. It strengthens correctness guarantees
  - B. It converts the code to single-threaded
  - C. It changes timing without establishing correctness
  - D. It makes code atomic
  - Correct answer: C
  - Explanation: Sleep may reduce the chance of failure temporarily, but it does not create any actual correctness guarantee.

24. For per-entity strict ordering, which partitioning strategy is best?
  - A. Partition by entity key so related updates land on the same ordered lane
  - B. Random partitioning each message
  - C. Broadcast each write to every worker
  - D. Keep all writes client-side
  - Correct answer: A
  - Explanation: Partitioning by key keeps related updates together and makes per-entity ordering enforceable.

25. Code question: Why can `x = x + 1` fail under concurrency just like `x++`?
  - A. `+` is unsupported on ints
  - B. Assignment always deadlocks
  - C. It is optimized away
  - D. It is still a read-modify-write sequence
  - Correct answer: D
  - Explanation: The syntax changed, but the behavior is still read current value, compute new value, write new value.

26. What is the primary reason a unique constraint belongs in the database even if the app validates uniqueness?
  - A. It improves variable naming
  - B. It closes the race at the final write boundary
  - C. It replaces all application logic
  - D. It prevents reads
  - Correct answer: B
  - Explanation: Application checks can race. The database constraint is the final authority that prevents conflicting writes.

27. Which architectural smell most increases race-condition risk?
  - A. Short methods
  - B. Immutable DTOs
  - C. Many writers with unclear ownership over shared mutable state
  - D. Separate query models
  - Correct answer: C
  - Explanation: Multiple uncontrolled writers create ambiguity around sequencing, protection, and authority.

28. Code question: Why might a lock-free design still be correct?
  - A. It can rely on atomic primitives and clear ownership instead of mutual exclusion
  - B. Lock-free means no shared state exists by definition
  - C. It never needs testing
  - D. The runtime guarantees fairness
  - Correct answer: A
  - Explanation: Correctness can come from atomic operations, immutability, partitioning, and ownership boundaries rather than locks alone.

29. In a CQRS-style system, why separate commands from queries for concurrency control?
  - A. To make every read transactional
  - B. To eliminate eventual consistency
  - C. To make all data immutable forever
  - D. To narrow mutation paths and reason about write coordination more clearly
  - Correct answer: D
  - Explanation: Separating reads from writes helps concentrate coordination logic around mutation paths.

30. Which statement best distinguishes synchronization from atomicity?
  - A. They are identical terms
  - B. Atomicity is about indivisible operations; synchronization is about coordinating access and ordering
  - C. Synchronization only matters in databases
  - D. Atomicity only matters in UI code
  - Correct answer: B
  - Explanation: Atomicity describes one operation's indivisibility. Synchronization is the broader coordination of multiple operations and actors.