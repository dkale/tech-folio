# `ParallelForEachAsync<T>` for `IEnumerable<T>`

## Table of Contents

- [Extension Method](#extension-method)
- [Explanation](#explanation)
- [Parameters](#parameters)
- [Inner Function](#inner-function)
- [Partitioning the Source](#partitioning-the-source)
- [Processing Partitions in Parallel](#processing-partitions-in-parallel)
- [Waiting for All Tasks](#waiting-for-all-tasks)
- [Return Value](#return-value)
- [Example Usage](#example-usage)
- [Thread and Task Behavior](#thread-and-task-behavior)
- [Execution with 100 Items and DOP 10](#execution-with-100-items-and-dop-10)

## Extension Method

```csharp
using System;
using System.Collections.Concurrent;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

public static Task ParallelForEachAsync<T>(
    this IEnumerable<T> source,
    int dop,
    Func<T, Task> body)
{
    async Task AwaitPartition(IEnumerator<T> partition)
    {
        using (partition)
        {
            while (partition.MoveNext())
            {
                await body(partition.Current);
            }
        }
    }

    return Task.WhenAll(
        Partitioner
            .Create(source)
            .GetPartitions(dop)
            .AsParallel()
            .Select(p => AwaitPartition(p)));
}
```

[↑ Back to Top](#table-of-contents)

## Explanation

The `ParallelForEachAsync` method is an extension method designed to process elements in an `IEnumerable<T>` concurrently using a specified degree of parallelism.

[↑ Back to Top](#table-of-contents)

## Parameters

- `source`: The input collection of items of type `T` to be processed.
- `dop`: The degree of parallelism, or how many partitions/tasks are allowed to be active concurrently.
- `body`: An asynchronous function (`Func<T, Task>`) that specifies the operation to perform on each element.

[↑ Back to Top](#table-of-contents)

## Inner Function

The `AwaitPartition` inner function processes a single partition:

- Its input is an `IEnumerator<T>` representing a partition of the source.
- A `using` statement ensures that the enumerator is disposed after use.
- A `while` loop iterates over the elements in the partition.
- For each element, it invokes `body(participant.Current)` asynchronously.

[↑ Back to Top](#table-of-contents)

## Partitioning the Source

`Partitioner.Create(source)` splits the input collection into multiple partitions.

`.GetPartitions(dop)` creates a collection of enumerators with the number of partitions equal to `dop`, the requested degree of parallelism.

[↑ Back to Top](#table-of-contents)

## Processing Partitions in Parallel

- `AsParallel()` converts the collection of partitions into a form that supports parallel operations.
- `Select(p => AwaitPartition(p))` applies `AwaitPartition` to each partition.

This schedules processing for each partition concurrently.

[↑ Back to Top](#table-of-contents)

## Waiting for All Tasks

`Task.WhenAll(...)`:

- Collects all tasks returned by `AwaitPartition` for each partition.
- Waits until all partitions complete before the returned task completes.

[↑ Back to Top](#table-of-contents)

## Return Value

The method returns a `Task` that completes after all elements in the source have been processed.

**Summary of execution:**

1. The source collection is split into `dop` partitions.
2. Each partition is processed concurrently by invoking `body` asynchronously for each element.
3. The method waits until all elements across all partitions have been processed.

[↑ Back to Top](#table-of-contents)

## Example Usage

```csharp
var numbers = Enumerable.Range(1, 100);

await numbers.ParallelForEachAsync(4, async number =>
{
    Console.WriteLine(
        $"Processing {number} on Thread {Thread.CurrentThread.ManagedThreadId}");

    await Task.Delay(100); // Simulate work
});
```

In this example:

- `numbers` is divided into four partitions.
- Each partition is processed concurrently with asynchronous tasks.
- Console output can show multiple numbers being processed simultaneously.

[↑ Back to Top](#table-of-contents)

## Thread and Task Behavior

### Does This Method Process Each Element on a Separate Thread?

No. `ParallelForEachAsync` does not necessarily process each element on a separate thread. It processes elements concurrently while respecting the specified degree of parallelism (`dop`).

### Concurrency versus Parallelism

The method uses `Task.WhenAll` to execute multiple asynchronous tasks concurrently. This does not imply that each task runs on a separate thread.

Tasks are scheduled on the `ThreadPool`, and actual execution depends on thread management and task scheduling for the system.

### Degree of Parallelism

- The input collection is divided into `dop` partitions.
- Each partition is processed independently and can run in parallel.
- If `dop` is `4`, at most four partitions and therefore four tasks run concurrently.
- Not every element receives its own thread.

### Task Scheduler and Threads

Asynchronous methods release a thread back to the `ThreadPool` while awaiting an incomplete operation. The thread is reused when execution resumes.

The actual number of threads used depends on the work performed in `body` and the system's scheduling.

### Partition Processing

Each partition is processed sequentially. For example, if partition 1 contains `{1, 2, 3}`, those elements are processed one by one within that partition. Multiple partitions can run concurrently.

### Example: How Threads Are Used

```csharp
await source.ParallelForEachAsync(4, async item =>
{
    Console.WriteLine(
        $"Processing {item} on Thread {Thread.CurrentThread.ManagedThreadId}");

    await Task.Delay(100); // Simulate an asynchronous operation
});
```

Possible output, assuming four tasks run in parallel:

```text
Processing 1 on Thread 10
Processing 2 on Thread 12
Processing 3 on Thread 14
Processing 4 on Thread 16
Processing 5 on Thread 10
Processing 6 on Thread 12
...
```

This output demonstrates:

- A maximum of four threads can be active at once when `dop` is `4`.
- Threads are reused as tasks complete.
- Asynchronous work, such as network calls or database queries, does not block threads and allows the system to optimize thread usage.

### Summary

- The method processes up to `dop` elements/partitions in parallel.
- It does not assign a separate thread to every element.
- Threads are managed by the `ThreadPool` and depend on the system's task scheduler.
- The method uses threads efficiently, especially when `body` contains asynchronous operations.

[↑ Back to Top](#table-of-contents)

## Execution with 100 Items and DOP 10

When the source enumerable contains 100 items and `dop` is `10`, the method executes as follows.

### Partition Creation

`Partitioner.Create(source)` divides the 100 items into 10 partitions, one for each degree of parallelism.

Each partition has approximately 10 items, depending on how the partitioner divides the source:

```text
Partition 1:  {1, 2, ..., 10}
Partition 2:  {11, 12, ..., 20}
Partition 3:  {21, 22, ..., 30}
...
Partition 10: {91, 92, ..., 100}
```

### Task Creation

For each partition, the method creates a task to process that partition's items sequentially.

This creates 10 tasks, one per partition.

### Concurrent Processing

The 10 tasks run concurrently, limited by system thread availability and task scheduling.

Each task processes its assigned items one by one, while tasks themselves run in parallel.

### Execution Flow

At any given time, up to 10 tasks are active.

Each task processes items sequentially within its partition:

```text
Task 1 processes {1, 2, ..., 10}
Task 2 processes {11, 12, ..., 20}
...
Task 10 processes {91, 92, ..., 100}
```

### Completion

The method waits for all tasks to complete using `Task.WhenAll`.

The total execution time depends on:

- The longest-running task or the partition containing the most expensive items.
- Whether `body` performs synchronous or asynchronous work.

### CPU-Bound Example

```csharp
await source.ParallelForEachAsync(10, async item =>
{
    Console.WriteLine($"Processing {item}");
    Thread.Sleep(100); // Simulate blocking synchronous work
});
```

For a synchronous workload:

- Each task processes 10 items sequentially.
- Each item takes approximately 100 milliseconds.
- Total time per task is approximately `10 × 100 ms = 1 second`.
- Since 10 tasks run in parallel, the total execution time can be approximately 1 second.

### Asynchronous I/O-Bound Example

```csharp
await source.ParallelForEachAsync(10, async item =>
{
    Console.WriteLine($"Processing {item}");
    await Task.Delay(100); // Simulate non-blocking asynchronous work
});
```

For an asynchronous workload:

- Each task processes 10 items sequentially but asynchronously.
- `Task.Delay(100)` releases the thread while the delay is pending.
- Other tasks continue executing.
- Depending on task scheduling, total execution time can approach 1 second.
- The system uses threads more efficiently than with blocking work.

### Final Summary

- **Partitions:** 100 items are divided into 10 partitions of approximately 10 items each.
- **Tasks:** Each partition is processed by a separate task.
- **Concurrency:** Up to 10 tasks run concurrently, as specified by `dop`.
- **CPU-bound execution time:** Determined primarily by the slowest task.
- **I/O-bound execution time:** Depends on the work in `body` and thread reuse.

[↑ Back to Top](#table-of-contents)
