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
