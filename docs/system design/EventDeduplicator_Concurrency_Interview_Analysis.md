# EventDeduplicator Concurrency Interview Analysis

An interview question analysis of a C# `EventDeduplicator` class that processes high-throughput,
multi-threaded, at-least-once-delivery events (~50K events/sec across 20 threads), including the
race condition bug, secondary problems, and alternative fixes discussed.

## Table of Contents

- [1. The Original Code](#1-the-original-code)
- [2. The Primary Bug: Check-Then-Act Race Condition](#2-the-primary-bug-check-then-act-race-condition)
- [3. Secondary Problems (Detailed, with Examples)](#3-secondary-problems-detailed-with-examples)
  - [3.1 `List<string>.Contains()` is O(n) and gets slower over time](#31-liststringcontains-is-on-and-gets-slower-over-time)
  - [3.2 Reading the `List<T>` while another thread writes to it is undefined behavior](#32-reading-the-listt-while-another-thread-writes-to-it-is-undefined-behavior)
  - [3.3 Unbounded memory growth](#33-unbounded-memory-growth)
  - [3.4 The single global lock serializes all threads at the busiest point](#34-the-single-global-lock-serializes-all-threads-at-the-busiest-point)
- [4. The Minimal Fix](#4-the-minimal-fix)
- [5. Alternative Design: `ConcurrentDictionary<string, string>`](#5-alternative-design-concurrentdictionarystring-string)
  - [5.1 Naive version — compute, then store](#51-naive-version--compute-then-store)
  - [5.2 Tempting but broken: `GetOrAdd` with a factory](#52-tempting-but-broken-getoradd-with-a-factory)
  - [5.3 Best of both worlds: `Lazy<T>` values](#53-best-of-both-worlds-lazyt-values)
    - [5.3.1 What is `Lazy<T>`?](#531-what-is-lazyt)
  - [5.4 Comparison Summary](#54-comparison-summary)
- [6. Key Takeaways](#6-key-takeaways)

---

## 1. The Original Code

```csharp
public class Program
{
    public static void Main()
    {
        var dedup = new EventDeduplicator();
        var threads = new List<Thread>();
        for (int t = 0; t < 20; t++)
        {
            var thread = new Thread(() =>
            {
                foreach (var evt in GetIncomingEvents())   // events arrive at ~50K/sec
                {
                    // The SAME event_id can show up on multiple threads (at-least-once delivery)
                    dedup.ProcessEvent(evt.EventId, evt.Payload);
                }
            });
            threads.Add(thread);
            thread.Start();
        }

        threads.ForEach(t => t.Join());
    }

    private static IEnumerable<(string EventId, string Payload)> GetIncomingEvents()
    {
        yield break;
    }
}

public class EventDeduplicator
{
    private readonly List<string> _processedEvents = new();  // stores event_ids we've seen
    private readonly List<string> _results = new();          // stores processed results
    private readonly object _lock = new();

    public bool ProcessEvent(string eventId, string payload)
    {
        // Check if we've already processed this event
        if (_processedEvents.Contains(eventId))
            return false;  // duplicate

        // Simulate processing work
        string result = Transform(payload);

        lock (_lock)
        {
            _processedEvents.Add(eventId);
            _results.Add(result);
        }

        return true;
    }

    private static string Transform(string payload)
    {
        Thread.Sleep(1);  // simulate work
        return payload.ToUpper();
    }
}
```

**Context:** 20 threads pull events from a queue where the *same* `event_id` can legitimately be
delivered more than once (at-least-once delivery semantics), at a combined rate of ~50,000
events/sec. `EventDeduplicator` is supposed to guarantee each unique `event_id` is only processed
once.

---

## 2. The Primary Bug: Check-Then-Act Race Condition

The dedup **check** and the dedup **write** are not atomic — they're two separate steps, and only
the write is inside the lock:

```csharp
if (_processedEvents.Contains(eventId))   // read — NOT locked
    return false;

string result = Transform(payload);        // work happens outside any lock

lock (_lock)
{
    _processedEvents.Add(eventId);         // write — locked, but too late
    _results.Add(result);
}
```

**What goes wrong:** Two threads can both call `ProcessEvent` with the same `eventId` at nearly
the same time. Both read `Contains(eventId) == false` (neither has added it yet), so both proceed
past the guard, both run `Transform`, and both then acquire the lock and add the id + result. The
lock only protects the *write*, not the *check-plus-write* as a unit — so the deduplicator fails
at the one thing it exists to do. With 20 threads hammering it at 50K events/sec, duplicate
processing (and duplicate entries in `_results`) will happen constantly, not as a rare edge case.

---

## 3. Secondary Problems (Detailed, with Examples)

### 3.1 `List<string>.Contains()` is O(n) and gets slower over time

`_processedEvents.Contains(eventId)` doesn't jump straight to the answer. It walks the list **one
item at a time** from the start until it finds a match (or reaches the end and gives up).

**Example:** Imagine the list already has 1,000,000 event ids in it (easy to reach at 50K
events/sec — that's less than a minute of traffic). Every single new event now requires checking,
on average, half a million string comparisons just to answer "have I seen this before?" As the
list keeps growing to 5 million, 10 million... every check gets proportionally slower.

Compare that to a `HashSet<string>` or `ConcurrentDictionary<string, T>`, where the lookup is a
hash computation + direct jump to a bucket — roughly the same speed whether there are 100 items or
100 million. That's the difference between **O(n)** (List) and **O(1)** (Dictionary/HashSet).

**Symptom you'd actually see:** the app looks healthy for the first few seconds, then throughput
visibly drops minute over minute, and CPU usage climbs even though the incoming event rate stayed
constant at 50K/sec.

### 3.2 Reading the `List<T>` while another thread writes to it is undefined behavior

The read (`Contains`) happens with **no lock**. The write (`Add`) happens **inside** the lock.
That means it's entirely possible for:

- Thread A to be in the middle of `_processedEvents.Add(eventId)` — which internally might be
  **resizing the array** that backs the list (copying all elements to a bigger array), while...
- Thread B is simultaneously calling `_processedEvents.Contains(...)` on that same list.

`List<T>` was never designed to tolerate that. It's not just "might return a stale answer" (which
would be tolerable) — it's genuinely undefined: Thread B could throw an
`IndexOutOfRangeException`, see a partially-copied array, or in rare cases get stuck/corrupt
internal state.

**Example:** Picture the list resizing its internal array from 1,000,000 slots to 2,000,000 slots.
That resize takes real, non-zero time to copy every element over. If another thread reads `Count`
or iterates the list *during* that copy, it can read a length that no longer matches the actual
data — that's the kind of race that produces a crash that seems to happen "randomly" once every
few million requests and is very hard to reproduce.

### 3.3 Unbounded memory growth

Every event that comes in adds one entry to `_processedEvents` and one to `_results`. **Nothing
ever removes anything.** These lists only grow, forever, for as long as the process runs.

**Example:** At 50,000 events/sec:

| Duration | Entries added |
|---|---|
| 1 minute | 3,000,000 |
| 1 hour | 180,000,000 |
| 24 hours | ~4.3 billion |

Even if each `eventId` string is small (say, 40 bytes with object overhead), that's ~172 GB of
strings added per hour just for the ids — before counting `_results`. No real process has
unlimited RAM, so this **will** eventually crash with an `OutOfMemoryException`, or get killed by
the OS/container orchestrator for exceeding its memory limit, likely within hours, not days.

**Why the original design has no way around this:** it's a "remember every event id forever"
cache with no expiration policy — no TTL, no LRU eviction, no windowing. A real fix needs a bound,
e.g. "only remember event ids seen in the last 5 minutes" (since at-least-once delivery systems
typically redeliver within a bounded window, not days later).

### 3.4 The single global lock serializes all threads at the busiest point

Even *if* the check-then-act race were fixed by moving everything inside the lock, you'd still have
this problem: **only one thread at a time can be inside `lock (_lock) { ... }`.** With 20 threads
all calling `ProcessEvent` simultaneously at a combined 50,000 events/sec, they all have to queue
up single-file to get through that lock, one at a time.

**Example:** Imagine 20 people trying to walk through a single-file turnstile to record results
already computed and ready to record. It doesn't matter that 20 people can *compute* their answer
in parallel — they still funnel through one gate to write it down, so the gate itself becomes the
bottleneck. If the lock body takes even a tiny amount of time (memory allocation for `Add`, list
resizing, etc.), and you're pushing 50K/sec through it, that lock is doing 50,000
lock-acquire/release cycles per second no matter how many CPU cores you have — the other 19
threads spend time waiting instead of processing.

**Why `ConcurrentDictionary` avoids this:** internally it uses **striped/partitioned locking** —
it splits its internal storage into multiple independent segments, each with its own lock, so two
threads updating different segments (different event ids, most of the time) don't block each
other at all. That's why swapping the lock+List design for `ConcurrentDictionary.TryAdd` isn't
just "cleaner code" — it removes the single-lane bottleneck entirely.

**Where exactly is this "single global lock" in the code?** It's the `lock (_lock)` block inside
`ProcessEvent`:

```csharp
public bool ProcessEvent(string eventId, string payload)
{
    if (_processedEvents.Contains(eventId))
        return false;

    string result = Transform(payload);

    lock (_lock)              // <-- the single global lock
    {
        _processedEvents.Add(eventId);
        _results.Add(result);
    }

    return true;
}
```

`_lock` is a single `readonly object` field on the `EventDeduplicator` instance:

```csharp
private readonly object _lock = new();
```

Since `Main()` creates only **one** `dedup` instance and all 20 threads call
`dedup.ProcessEvent(...)` on that same instance, every thread contends for the **same `_lock`
object** — regardless of whether their `eventId`s are even related. Whichever thread reaches
`lock (_lock)` first "wins," and every other thread must wait until that thread exits the block.
That's what "global" means here: one lock shared by every thread, guarding every event, rather
than being scoped per-event or per-partition.

---

## 4. The Minimal Fix

Make the "have I seen this?" check and the "mark as seen" step one atomic operation using a
concurrent set, so only the winning thread does the work:

```csharp
public class EventDeduplicator
{
    private readonly ConcurrentDictionary<string, byte> _processedEvents = new();
    private readonly ConcurrentQueue<string> _results = new();

    public bool ProcessEvent(string eventId, string payload)
    {
        if (!_processedEvents.TryAdd(eventId, 0))
            return false; // already claimed by another thread — true duplicate

        string result = Transform(payload);
        _results.Enqueue(result);
        return true;
    }

    private static string Transform(string payload)
    {
        Thread.Sleep(1);
        return payload.ToUpper();
    }
}
```

`TryAdd` is atomic, so exactly one thread "wins" per `eventId` and only that thread runs
`Transform`/records the result — no lock needed at all. This also fixes the O(n) lookup
(dictionary is O(1)) and removes the lock contention bottleneck.

**Trade-off:** this design loses the `eventId → result` association (`_results` is just a flat
queue of results, same as the original `_results` list). If that mapping needs to be preserved,
see the `ConcurrentDictionary<string, string>` alternative below.

(Unbounded growth would still need a TTL/eviction strategy — e.g., an LRU or time-windowed store —
if that matters for a production system / interview follow-up.)

---

## 5. Alternative Design: `ConcurrentDictionary<string, string>`

**Question raised:** Why not use a `ConcurrentDictionary<string, string>` where the value is the
result, instead of a separate marker dictionary + results queue?

This is a valid alternative — simpler in some ways — but it has one important gotcha depending on
*how* it's used.

### 5.1 Naive version — compute, then store

```csharp
private readonly ConcurrentDictionary<string, string> _processedEvents = new();

public bool ProcessEvent(string eventId, string payload)
{
    string result = Transform(payload);
    return _processedEvents.TryAdd(eventId, result); // atomic claim
}
```

This is correct (no duplicate ever gets stored — `TryAdd` is atomic), but **every racing thread
still runs `Transform` before finding out it lost the race**. If `Transform` is cheap that's fine;
if it's expensive/slow (as the `Thread.Sleep(1)` here simulates — 1ms × wasted duplicate races at
50K/sec adds up), you're burning CPU on throwaway work.

### 5.2 Tempting but broken: `GetOrAdd` with a factory

```csharp
_processedEvents.GetOrAdd(eventId, id => Transform(payload));
```

This *looks* like it guarantees the factory runs once per key, but it doesn't —
`ConcurrentDictionary.GetOrAdd`'s value-factory delegate is explicitly documented as **not
guaranteed to run only once** under contention; multiple threads can invoke it concurrently for a
new key, and all but one result is discarded. This is a common footgun.

### 5.3 Best of both worlds: `Lazy<T>` values

If "compute exactly once" + the `eventId → result` mapping in one structure is needed, wrap the
value in `Lazy<T>`:

#### 5.3.1 What is `Lazy<T>`?

`Lazy<T>` is a built-in .NET wrapper type that **delays computing a value until the first time
it's actually accessed**, and then **caches** that value so subsequent accesses just return the
cached result instead of recomputing it.

```csharp
var lazyValue = new Lazy<string>(() => ExpensiveComputation());

// At this point, ExpensiveComputation() has NOT run yet.

string result = lazyValue.Value; // NOW it runs, computes once, and caches the result
string again = lazyValue.Value;  // returns the cached result, does NOT recompute
```

- The constructor takes a **factory delegate** (`Func<T>`) — code that knows *how* to produce the
  value, but it isn't invoked yet.
- The first time `.Value` is read, the factory runs and the result is stored internally.
- Every subsequent `.Value` read just returns the stored result — the factory never runs again.

**Thread-safety modes** control what happens when multiple threads access `.Value` concurrently
before it's been computed (set via the constructor's `LazyThreadSafetyMode` overload):

| Mode | Behavior |
|---|---|
| `None` | No thread safety — fastest, but unsafe if accessed concurrently. |
| `PublicationOnly` | Multiple threads may run the factory concurrently, but only one result is "published" (kept); the others are discarded. Same footgun as `ConcurrentDictionary.GetOrAdd`. |
| `ExecutionAndPublication` (default) | Only **one thread ever runs the factory**. Every other thread calling `.Value` concurrently **blocks and waits** for that execution to finish, then all threads get the same cached result. |

`ExecutionAndPublication` is the one that gives a true "compute exactly once, everyone gets the
same answer" guarantee — which plain `ConcurrentDictionary.GetOrAdd` cannot promise on its own
(see [5.2](#52-tempting-but-broken-getoradd-with-a-factory)).

**How this applies to the dedup example below:** `GetOrAdd` itself might still create more than
one `Lazy<string>` **wrapper object** if two threads race for a brand-new `eventId` — but that's
cheap (it just wraps a delegate, doesn't run it). The actual expensive work (`Transform(payload)`)
only happens when `.Value` is read, and `Lazy<T>` guarantees only **one** thread ever executes that
factory delegate for the `Lazy` instance that ends up stored in the dictionary. So if Thread A's
wrapper wins the `GetOrAdd` race, both Thread A and Thread B then call `.Value` on that *same*
stored `Lazy<string>` — only one of them actually runs `Transform`, the other blocks until it's
done, and both get the identical result.

```csharp
private readonly ConcurrentDictionary<string, Lazy<string>> _processedEvents = new();

public string ProcessEvent(string eventId, string payload)
{
    var lazyResult = _processedEvents.GetOrAdd(eventId,
        _ => new Lazy<string>(() => Transform(payload), LazyThreadSafetyMode.ExecutionAndPublication));

    return lazyResult.Value; // only the winning thread ever runs Transform; others block and get the same result
}
```

`ExecutionAndPublication` makes the `Lazy<T>` itself do the locking, so exactly one thread executes
`Transform` per `eventId`, everyone else waits and gets the same cached result — and a single
dictionary answers both "have we seen it" and "what was the result."

### 5.4 Comparison Summary

| Design | Wasted computation on races? | Keeps `eventId → result` mapping? | Complexity |
|---|---|---|---|
| `ConcurrentDictionary<string, byte>` marker + `ConcurrentQueue<string>` results (Section 4) | No — claims slot before computing | No — results are a flat queue | Low |
| `ConcurrentDictionary<string, string>` naive (compute then `TryAdd`) | Yes — every racer computes | Yes | Low |
| `ConcurrentDictionary<string, string>` with `GetOrAdd(factory)` | Yes (factory not guaranteed single-run — common misconception) | Yes | Low, but incorrect assumption risk |
| `ConcurrentDictionary<string, Lazy<string>>` | No — `Lazy` guarantees single execution | Yes | Medium |

Which one is "right" depends on whether `Transform` is cheap/idempotent (naive version is fine) or
expensive/side-effecting (use the `Lazy<T>` version).

---

## 6. Key Takeaways

1. **Root cause:** the dedup check (`Contains`) and the dedup commit (`Add`) are not atomic — a
   classic check-then-act (TOCTOU) race. Fix by making "check + claim" a single atomic operation
   (`ConcurrentDictionary.TryAdd`).
2. **`List<T>` is not thread-safe** for concurrent reads during writes — even ignoring the dedup
   logic bug, this is independently undefined behavior.
3. **O(n) `Contains` on an ever-growing list** degrades throughput over time; use a hash-based
   structure for O(1) lookups.
4. **Unbounded growth** of `_processedEvents`/`_results` is a memory leak at high throughput; a
   real system needs eviction (TTL/LRU/time-window).
5. **A single global lock serializes all threads** at the busiest point; `ConcurrentDictionary`'s
   internal striped locking avoids this bottleneck.
6. **`GetOrAdd`'s factory can run more than once** under contention — don't rely on it for
   "exactly once" side-effecting work; use `Lazy<T>` with `ExecutionAndPublication` instead.
