# Asynchronous Programming in C#

**Core Concepts:** Asynchrony · Parallelism · Concurrency · Multithreading

**Reference:** [Concurrency vs Parallelism vs Async - StackOverflow](https://stackoverflow.com/questions/4844637/what-is-the-difference-between-concurrency-parallelism-and-asynchronous-methods)

---

## Table of Contents

1. [Core Concepts](#core-concepts)
   - [Concurrency vs Parallelism](#concurrency-vs-parallelism)
   - [Asynchronous vs Multithreading](#asynchronous-vs-multithreading)
   - [When Operations Don't Need Threads](#when-operations-dont-need-threads)

2. [Async/Await Fundamentals](#asyncawait-fundamentals)
   - [Overview](#overview)
   - [How Await Works](#how-await-works)
   - [Does Await Create Threads?](#does-await-create-threads)
   - [Advantages Over Callbacks](#advantages-over-callbacks)

3. [CPU-Bound vs I/O-Bound Operations](#cpu-bound-vs-io-bound-operations)
   - [Core Definitions](#core-definitions)
   - [Async/Await Suitability Matrix](#asyncawait-suitability-matrix)
   - [When to Use Task.Run](#when-to-use-taskrun)

4. [Task Parallel Library (TPL)](#task-parallel-library-tpl)
   - [Benefits and Overview](#benefits-and-overview)
   - [Task as Universal Abstraction](#task-as-universal-abstraction)
   - [Exception Handling in Tasks](#exception-handling-in-tasks)

5. [Continuation Patterns](#continuation-patterns)
   - [ContinueWith (Traditional)](#continuewith-traditional)
   - [Async/Await (Modern)](#asyncawait-modern)
   - [Readability Comparison](#readability-comparison)

6. [Task.Run in Depth](#taskrun-in-depth)
   - [What Task.Run Does](#what-taskrun-does)
   - [Thread Pool Integration](#thread-pool-integration)
   - [Synchronization Context](#synchronization-context)

7. [Advanced Patterns](#advanced-patterns)
   - [Parallel.ForEach](#parallelforeach)
   - [ConfigureAwait](#configureawait)
   - [ASP.NET Considerations](#aspnet-considerations)

8. [Code Examples](#code-examples)
   - [Image Processing Workflow](#image-processing-workflow)
   - [CPU-Bound vs I/O-Bound Implementations](#cpu-bound-vs-io-bound-implementations)

9. [Asynchronous Web API Endpoint Benefits](#asynchronous-web-api-endpoint-benefits)
  - [Why Async Endpoints Matter in .NET Core](#why-async-endpoints-matter-in-net-core)
  - [Example: Async Controller Action](#example-async-controller-action)
  - [Practical Guidance](#practical-guidance)

---

## Core Concepts

### Concurrency vs Parallelism

**Concurrency:** Execution of multiple tasks is *interleaved* through context switching, not necessarily simultaneous.
- Single processing resource managing many tasks
- Achieved on single-core systems via OS thread scheduling
- Creates the *appearance* of simultaneous execution

**Parallelism:** Tasks execute *truly simultaneously* on multiple CPU cores.
- Multiple processing resources executing different tasks at the same time
- Requires multi-core hardware to be effective
- Actual parallel execution of work

**Key Distinction:** Concurrency is about *task management*; parallelism is about *task execution*.

### Asynchronous vs Multithreading

| Aspect | Asynchronous | Multithreading |
|--------|--------------|---|
| **Definition** | Calling code doesn't wait; execution proceeds without blocking | Using multiple threads for parallelism/responsiveness |
| **Threading** | Doesn't necessarily use multiple threads | Explicitly uses multiple threads |
| **Overhead** | Minimal when used for I/O | Higher context-switching overhead |
| **Use Case** | I/O operations, responsive UI/services | CPU-intensive work across cores |
| **CPU Utilization** | Good for I/O-bound work | Essential for CPU-bound work |

**Simple Analogy:**
- **Asynchronous:** Like a waiter in a restaurant. Takes an order, passes it to the kitchen, then serves the next customer without waiting for the food. When the food is ready, they deliver it.
- **Multithreading:** Like multiple cooks in the kitchen. Many cooks work on multiple orders simultaneously. One order can be worked on by multiple cooks at once, or multiple cooks handle different orders in parallel.

**Another Analogy (Recruitment):**
- **Asynchronous:** Like an HR recruiter. They tell candidates "We'll get back to you," then move on to interview the next candidate. Candidates don't sit in the waiting room blocking the process. When HR has news, they call back.
- **Synchronous:** Candidates sit in the waiting room blocking the next person from being interviewed until the first candidate's interview is complete.

### When Operations Don't Need Threads

**I/O Operations require no thread blocking:**
- I/O hardware (disk, network) handles the operation asynchronously
- CPU simply initiates I/O and moves to other work
- Hardware interrupts CPU when complete; OS delivers event to application
- **Result:** Async I/O without extra threads

**Higher-level APIs often hide this:**
- Many APIs don't expose OS-level async capabilities
- May force thread usage even for I/O (when only synchronous methods available)
- But native async APIs (HttpClient, FileStream) leverage true async I/O

**CPU-Bound Operations require threads:**
- CPU must actively do work
- Cannot proceed asynchronously without another execution unit
- Requires separate thread to achieve true asynchrony

---

## Async/Await Fundamentals

### Overview

The async/await model provides a simple, readable way to write asynchronous code:

```csharp
// I/O-bound: await naturally async operation
await DownloadImage(url);

// CPU-bound: use Task.Run to move to thread pool
await Task.Run(() => ExpensiveComputation());
```

**Core objects:**
- **Task** — Represents a single asynchronous operation
- **Task<T>** — Asynchronous operation returning value of type T
- **async** keyword — Marks method as containing await expressions
- **await** keyword — Pauses execution non-blockingly until operation completes

### How Await Works

When you use `await` on a Task:

1. **Yield control** — Rest of method becomes a continuation
2. **Return immediately** — Control returns to caller of async method
3. **Non-blocking** — Current thread is freed to handle other work
4. **Resume automatically** — When Task completes, continuation resumes

**Critical:** The thread is **NOT blocked** during the await. The await pauses the *method's* execution but releases the *thread* to do other work.

### Does Await Create Threads?

**No.** The `await` keyword alone does not create threads:
- `await` simply yields control and sets up a continuation
- The **Task Scheduler** manages actual execution
- May execute on same thread, different thread, or thread pool thread depending on context
- No thread allocation per await expression

**Example:** When awaiting `HttpClient.GetStringAsync()`, the actual network I/O happens on hardware; no thread sits waiting.

**Understanding Blocking vs Non-Blocking:**

*Blocking code:* The thread sits and waits, completely occupied until the operation finishes
```csharp
// BLOCKING: Current thread does absolutely nothing for 1 second
System.Threading.Thread.Sleep(1000);

// BLOCKING: Current thread waits synchronously for result
var result = task.Result;  // or task.Wait();
var data = httpClient.GetString(url);  // synchronous call
```

*Non-blocking code:* The thread is freed to do other work during the wait
```csharp
// NON-BLOCKING: Current thread is released, method pauses
await Task.Delay(1000);

// NON-BLOCKING: Current thread handles other requests while I/O happens
var data = await httpClient.GetStringAsync(url);
```

The key difference: **Blocking exhausts thread resources; non-blocking frees them for other work.**

### Advantages Over Callbacks

**Before: Callback Hell (ContinueWith)**
```csharp
DownloadImage(url).ContinueWith(task1 => {
  var bytes = task1.Result;
  SaveImage(bytes).ContinueWith(task2 => {
    BlurImage(path).ContinueWith(task3 => {
      // Deep nesting, hard to follow
    });
  });
});
```

**After: Clean Linear Code (Async/Await)**
```csharp
var bytes = await DownloadImage(url);
await SaveImage(bytes);
await BlurImage(path);
```

**Benefits:**
1. **Exception handling** — Normal try/catch works; exceptions auto-unwrapped
2. **Readability** — Code flows top-to-bottom like synchronous code
3. **Maintainability** — Easier to debug and understand control flow
4. **Conciseness** — Less boilerplate and punctuation

**Why This Matters:** With callbacks (ContinueWith), each operation adds a level of indentation and complexity. Your code becomes pyramid-shaped, making it hard to follow the actual logic. Async/await flattens this, making code read like a normal sequential sequence of operations — even though it's running asynchronously.

#### Await from the Caller's Perspective

When you call an async method **without awaiting it**, the method returns immediately before its work completes:

```csharp
void OnButtonClick() {
  Console.WriteLine("button clicked");
  DownloadAndBlur("https://...jpg");  // NOT awaited!
  Console.WriteLine("about to show dialog");
  ShowDialog("Success!");
  Console.WriteLine("dialog shown");
}

async Task DownloadAndBlur(string url) {
  Console.WriteLine("about to download");
  await DownloadImage(url);
  Console.WriteLine("finished downloading, about to blur");
  await BlurImage();
  Console.WriteLine("finished blurring, about to save");
  await SaveImage();
  Console.WriteLine("finished saving");
}

// Output:
// button clicked
// about to download
// about to show dialog
// dialog shown
// finished downloading, about to blur
// finished blurring, about to save
// finished saving
```

**Notice:** The dialog shows before the image download completes! This is because `DownloadAndBlur()` was called without `await`.

**Best Practice:** Always await your async methods:
```csharp
// ✅ CORRECT: Wait for async operation to complete
await DownloadAndBlur("https://...jpg");

// ⚠️ ACCEPTABLE: "Fire and forget" when you truly don't care about completion
// (But consider logging exceptions in ContinueWith for debugging)
DownloadAndBlur("https://...jpg").ContinueWith(t => {
  if (t.IsFaulted) Console.WriteLine($"Error: {t.Exception}");
});
```

---

## CPU-Bound vs I/O-Bound Operations

### Core Definitions

**CPU-Bound Code:**
- The processor is the bottleneck
- CPU actively doing calculations/processing
- High CPU utilization required
- Examples: Complex math, data processing, image manipulation, encryption
- *If you use async/await without Task.Run, the thread still waits (wasteful)*

**I/O-Bound Code:**
- Data transfer rate is the bottleneck
- CPU is mostly idle waiting for I/O
- I/O hardware doing the actual work (disk controller, network interface, etc.)
- Examples: Network requests, file operations, database queries
- *The CPU doesn't have to wait—hardware interrupts when done, OS delivers the event*

**The Fundamental Rule:** 
- **I/O-bound operations don't need threads** — hardware handles them asynchronously
- **CPU-bound operations must have threads** — CPU work can't be "offloaded" to hardware

### Why This Matters for Async/Await

Async/await is designed for I/O-bound work. When you await an I/O operation:
- No thread is sitting around wasting CPU core capacity
- The OS/hardware handles the actual I/O
- Your thread is freed to handle other work
- Result: Better scalability, especially in web scenarios

If you use async/await for CPU-bound work without Task.Run:
- You're still blocking the thread with computation
- No scalability benefit — you just made the code harder to read
- Actually worse than synchronous code because of async overhead

If you use Task.Run for CPU-bound work:
- You move the heavy computation to a thread pool thread
- Main thread stays responsive
- Only suitable for scenarios where responsiveness matters (UI, ASP.NET doesn't benefit)

### Async/Await Suitability Matrix

| Operation Type | Async/Await | Task.Run + Async | Notes |
|---|---|---|---|
| **I/O-Bound** | ✅ Perfect | ❌ Wasteful | Use native async APIs directly |
| **CPU-Bound** | ❌ Not suitable | ✅ Good | Offload to thread pool |
| **Mixed** | ⚠️ Partial | ✅ Good | Use Task.Run for CPU parts |

**Why this matters:**

*Async/await for I/O-bound:* No thread is wasted sitting idle waiting for I/O. The hardware/OS handles the operation; CPU is free for other work. Perfect scalability.

*Async/await for CPU-bound:* The thread is still blocked doing CPU work. Async overhead with no benefit. The CPU still can't work on other tasks.

*Task.Run for CPU-bound:* Moves work to background thread, freeing UI thread. Good for responsiveness. For servers (ASP.NET), this reduces overall throughput since you have a fixed thread pool.

*Task.Run for I/O:* Unnecessarily creates threads that sit idle waiting for I/O. Wasteful of precious thread resources.

### When to Use Task.Run

**DO use Task.Run for:**
- CPU-intensive calculations in a responsive UI
- Offloading synchronous CPU-bound methods to background threads
- Preventing UI thread blocking from heavy processing

**DON'T use Task.Run for:**
- I/O operations in ASP.NET (reduces scalability)
- Operations with native async APIs available
- Library/reusable code (only in calling code, not implementations)

**The Key Principle:** Task.Run is for taking synchronous CPU-bound methods and making them "asynchronous" by running them on a thread pool thread. You should NOT use Task.Run for I/O operations (which are already asynchronous at the OS level) or in ASP.NET (where threading hurts scalability).

**Why avoid Task.Run for I/O in ASP.NET:**
- One thread per request model means threads are your most precious resource
- Task.Run reduces threads available for new incoming requests
- Scalability drops dramatically with many concurrent requests
- No responsiveness benefit anyway (user waits for page load)
- Better approach: Use native async I/O APIs throughout your stack

**Best practices:**

```csharp
// ❌ DON'T: CPU-bound as synchronous method
void DoWork() { /* heavy computation */ }

// ✅ DO: Call synchronous CPU-bound method with Task.Run
await Task.Run(() => DoWork());

// ✅ DO: I/O-bound with async/await directly
var data = await httpClient.GetStringAsync(url);

// ❌ DON'T: I/O-bound with Task.Run (wasteful)
await Task.Run(() => httpClient.GetString(url));
```

---

## Task Parallel Library (TPL)

### Benefits and Overview

The TPL provides a powerful abstraction layer over raw threading:

**Advantages over direct threading:**
- Auto-scales to multicore processors
- Auto-scales LINQ queries to multicore
- Handles work partitioning and thread pool management
- Simple, unified API for diverse async scenarios
- Built-in support for cancellation (CancellationToken) and progress (IProgress<T>)
- Reduces complexity of manual thread management

**Why TPL matters:** Before TPL, developers had to manually manage threads, locks, and synchronization. TPL abstracts these details away, letting you focus on the logic of what you want to accomplish rather than how to orchestrate threads.

### Task as Universal Abstraction

A **Task** represents any asynchronous operation your application must wait for:
- Async I/O operation (network, disk)
- CPU-bound work on thread pool
- Multiple disparate operations aggregated into one
- External events or delays

This unified abstraction allows composing diverse operations seamlessly.

**Key Insight:** Asynchronous programming and multithreaded programming are not the same thing:
- **Asynchronous programming** deals with *latency* — something your application has to wait for
- **Multithreaded programming** deals with *parallelization* — multiple things your application has to do simultaneously

A Task elegantly bridges both concerns: it abstracts away whether you're waiting on I/O (no thread) or CPU work (background thread) — you use the same pattern either way.

### Exception Handling in Tasks

**Task encapsulates exceptions completely:**
- Exception in task code does NOT interrupt application
- Must inspect task state to detect faults

**With ContinueWith (problematic):**
```csharp
if (task.Status == TaskStatus.Faulted && task.Exception != null) {
  foreach (var ex in task.Exception.InnerExceptions) {
    Console.WriteLine($"Exception: {ex}");
  }
}
```

**With await (simple):**
```csharp
try {
  await MyAsyncMethod();
} catch (SpecificException ex) {
  // Exception automatically unwrapped
  Console.WriteLine($"Exception: {ex}");
}
```

---

## Continuation Patterns

### Why Async/Await Was Introduced

Many applications require waiting for operations (processes, calculations, web requests, I/O). Doing this synchronously blocks the UI or reduces server scalability. The standard solution before async/await was callbacks, but callbacks have serious problems:

**Callback Problems:**
1. **Pyramid of Doom** — Each callback adds nesting and indentation
2. **Error Handling** — Exceptions don't propagate normally; must check task status manually
3. **Hard to Follow** — Control flow becomes hard to trace
4. **Verbose Boilerplate** — Lots of extra code for simple operations

**Async/Await Solution:**
- Code reads like normal sequential code (even though it's asynchronous)
- Natural exception handling with try/catch
- No deep nesting
- Minimal boilerplate

### ContinueWith (Traditional)

```csharp
public void DownloadAndProcess(string url) {
  DownloadImage(url).ContinueWith(task1 => {
    var imageBytes = task1.Result;
    SaveImage(imageBytes).ContinueWith(task2 => {
      BlurImage(path).ContinueWith(task3 => {
        Console.WriteLine("Done!");
      });
    });
  });
}
```

**Issues:**
- Deep nesting (callback pyramid)
- Exception handling is verbose
- Hard to read and maintain
- Result extraction via .Result requires careful null checking

### Async/Await (Modern)

```csharp
public async Task DownloadAndProcessAsync(string url) {
  var imageBytes = await DownloadImage(url);
  await SaveImage(imageBytes);
  await BlurImage(path);
  Console.WriteLine("Done!");
}
```

**Advantages:**
- Linear, top-to-bottom code flow
- Natural exception handling with try/catch
- Cleaner and more maintainable
- Result automatically unwrapped

### Readability Comparison

The async/await version is not just easier to read—it's easier to write correctly. The same method using ContinueWith requires significantly more code and careful error handling.

---

## Task.Run in Depth

### What Task.Run Does

```csharp
await Task.Run(() => ExpensiveComputation());
```

Task.Run:
1. Takes a delegate (Action or Func<T>)
2. Queues it to thread pool for execution
3. Returns a Task representing the ongoing operation
4. Allows you to await synchronous work

### Thread Pool Integration

**Thread pool management:**
- Creates pool of worker threads (usually = # of CPU cores)
- Queues work efficiently
- Reuses threads for multiple operations
- Auto-scales based on workload

**Synchronization context:**
- `await` captures current SynchronizationContext
- By default, resumes on original thread/context
- Critical for UI applications (must resume on UI thread)
- For library code, use `ConfigureAwait(false)` to allow thread pool thread

**Simple meaning:** SynchronizationContext is the rule that decides *where the rest of your async method continues running* after an `await`.

Think of it like a message dispatcher:
- In a UI app, it says: "When the async work finishes, come back to the UI thread before touching controls."
- In server or console code, there may be little or no special thread affinity, so continuation can run wherever the runtime allows.

**Why it exists:** Some environments have thread rules.
- WinForms/WPF UI controls must be updated on the UI thread
- Some application models need continuations to resume in a specific context

**Simple example:**

```csharp
var data = await FetchDataAsync();
UpdateUi(data);
```

By default, `await` captures the current context so `UpdateUi(data)` can safely run back on the original UI thread.

**Plain-language takeaway:** SynchronizationContext is basically the mechanism that says: "After this async work finishes, continue in the right place."

### Example

```csharp
async Task Main() {
  Console.WriteLine("Before: " + Thread.CurrentThread.ManagedThreadId);
  
  await Task.Run(() => {
    Console.WriteLine("During: " + Thread.CurrentThread.ManagedThreadId);
    for (int i = 0; i < 1000000000; i++) { /* work */ }
  });
  
  Console.WriteLine("After: " + Thread.CurrentThread.ManagedThreadId);
  // "Before" and "After" likely same thread; "During" different thread
}
```

---

## Advanced Patterns

### Parallel.ForEach

Process collection elements in parallel:

```csharp
int[] numbers = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

Parallel.ForEach(numbers, num => {
  Console.WriteLine($"Processing {num} on thread {Task.CurrentId}");
  // Process each element concurrently
});

Console.WriteLine("Processing complete.");
```

**When to use:**
- Substantial work per element
- No order dependency
- CPU-bound collection processing

**When NOT to use:**
- Minimal work per element (overhead exceeds benefit)
- Operations depend on processing order
- Shared resource contention likely

### ConfigureAwait

```csharp
await MyAsync().ConfigureAwait(false);
```

**Purpose:** `ConfigureAwait(false)` tells `await` **not** to resume on the original context after the awaited work finishes.

Without it:

```csharp
await SomeAsyncMethod();
```

the continuation usually tries to return to the captured context.

With it:

```csharp
await SomeAsyncMethod().ConfigureAwait(false);
```

the continuation may run on any available thread.

**What it does:**
- Tells await not to resume on current context
- Allows continuation on any thread pool thread
- Avoids expensive context switching

**Simple way to think about it:** It means, "I don't need to come back to the same thread after this await. Just continue wherever it is convenient."

**When to use:**
- Library code (not modifying UI)
- Don't need original thread after await
- Want to improve performance

**Common good use cases:**
- Reusable library code that should not assume a UI thread
- Helper/service methods that only process data and do not touch UI objects
- Code paths where avoiding context capture helps performance or avoids deadlock risks

**When to avoid:**
- UI event handlers and UI code that must update controls after `await`
- Code that depends on resuming in a specific context
- Request-handling code where you still need ambient context immediately after the await

**UI example:**

```csharp
async void Button_Click(object sender, EventArgs e)
{
  var data = await FetchAsync();
  textBox.Text = data;
}
```

This is fine because execution resumes on the UI context by default.

If you wrote this instead:

```csharp
async void Button_Click(object sender, EventArgs e)
{
  var data = await FetchAsync().ConfigureAwait(false);
  textBox.Text = data;
}
```

you may end up resuming on a non-UI thread, which can break UI updates.

**Library example:**

```csharp
public async Task<string> LoadDataAsync(string url)
{
  var response = await httpClient.GetAsync(url).ConfigureAwait(false);
  return await response.Content.ReadAsStringAsync().ConfigureAwait(false);
}
```

This is a good fit because the method does not need to get back onto a UI thread.

**Practical rule of thumb:**
- App code that needs UI or request context: usually avoid `ConfigureAwait(false)`
- General-purpose library code: usually prefer `ConfigureAwait(false)`

**Short summary:** Use `ConfigureAwait(false)` when you do not care about returning to the original context. Avoid it when the code after `await` must run in that original context.

**Example:**
```csharp
// Library code: enable parallelism
public async Task<string> GetDataAsync(string url) {
  var response = await httpClient.GetAsync(url).ConfigureAwait(false);
  var content = await response.Content.ReadAsStringAsync().ConfigureAwait(false);
  return ProcessData(content);  // Runs on any thread pool thread
}
```

### ASP.NET Considerations

**DON'T use Task.Run in ASP.NET:**
- One thread per request model
- Thread pool threads limited
- Task.Run reduces threads available for new requests
- Scalability drops with concurrent load
- No responsiveness benefit (user waits for page anyway)

**DO use async/await for I/O:**
```csharp
// ✅ GOOD: Let I/O complete without thread
var data = await _db.GetUserAsync(userId);

// ❌ BAD: Task.Run wastes thread
var data = await Task.Run(() => _db.GetUser(userId));
```

**Strategy:**
- Use async APIs for database, HTTP, file operations
- Keep CPU-bound work synchronous (don't offload to threads)
- Use async/await throughout your call stack

---

## Code Examples

### Image Processing Workflow

Process multiple images: download → save → blur

```csharp
async Task ProcessImages(List<string> urls) {
  var processImageTasks = new List<Task>();
  
  foreach (var url in urls) {
    processImageTasks.Add(ProcessSingleImage(url));
  }
  
  await Task.WhenAll(processImageTasks);
  Console.WriteLine("All images processed");
}

async Task ProcessSingleImage(string url) {
  // I/O-bound: download directly
  var fileName = Path.GetFileName(url);
  var imageBytes = await DownloadImage(url);
  var imagePath = Path.Combine(ImagePath, fileName);
  
  // Save
  await SaveImage(imageBytes, imagePath);
  
  // CPU-bound: blur on thread pool
  var blurredBytes = await Task.Run(() => BlurImage(imagePath));
  
  // Save blurred result
  var blurredPath = Path.Combine(ImagePath, $"{Path.GetFileNameWithoutExtension(fileName)}_blurred.jpg");
  await SaveImage(blurredBytes, blurredPath);
}

// I/O-bound methods: use native async APIs
async Task<byte[]> DownloadImage(string url) {
  using var client = new HttpClient();
  return await client.GetByteArrayAsync(url);
}

async Task SaveImage(byte[] bytes, string imagePath) {
  using var stream = new FileStream(imagePath, FileMode.Create);
  await stream.WriteAsync(bytes, 0, bytes.Length);
}

// CPU-bound method: wraps synchronous work
async Task<byte[]> BlurImage(string imagePath) {
  return await Task.Run(() => {
    var image = Image.Load(imagePath);
    image.Mutate(ctx => ctx.GaussianBlur());
    using (var ms = new MemoryStream()) {
      image.SaveAsJpeg(ms);
      return ms.ToArray();
    }
  });
}
```

### CPU-Bound vs I/O-Bound Implementations

**CPU-Bound: Use Task.Run**
```csharp
// Library method: synchronous only
void ComputeComplex(int[] data) {
  for (int i = 0; i < data.Length; i++) {
    data[i] = ExpensiveCalculation(data[i]);
  }
}

// Calling code: offload with Task.Run
async Task ProcessData() {
  var data = new int[1000000];
  await Task.Run(() => ComputeComplex(data));
  Console.WriteLine("Computation complete");
}
```

**I/O-Bound: Use native async APIs**
```csharp
// Always async for I/O operations
async Task<User> FetchUser(int userId) {
  using var db = new AppDbContext();
  return await db.Users.FindAsync(userId);  // Native async method
}

async Task ProcessUser() {
  var user = await FetchUser(123);  // No Task.Run needed
  Console.WriteLine(user.Name);
}
```

---

## Asynchronous Web API Endpoint Benefits

### Why Async Endpoints Matter in .NET Core

Making ASP.NET Core Web API endpoints asynchronous improves scalability, throughput, and resource usage, especially for I/O-heavy workloads.

**Key benefits:**
1. **Improved scalability** — Request threads are released while waiting on database, file, or network I/O.
2. **Higher throughput** — More concurrent requests can be handled with the same hardware.
3. **Better resource efficiency** — Thread pool threads are not blocked during I/O waits.
4. **Non-blocking execution** — Long I/O operations do not stall server request processing.
5. **Better responsiveness under load** — Reduced thread starvation during traffic spikes.

### Example: Async Controller Action

```csharp
using Microsoft.AspNetCore.Mvc;
using System;
using System.Threading.Tasks;

[ApiController]
[Route("api/[controller]")]
public class TrashProductsController : ControllerBase
{
  private readonly TrashProductService _trashProductService;

  public TrashProductsController(TrashProductService trashProductService)
  {
    _trashProductService = trashProductService;
  }

  [HttpGet]
  public async Task<IActionResult> GetTrashProducts(Guid catalogId, string originalProductHash = null)
  {
    var products = await _trashProductService.FetchTrashProductsAsync(catalogId, originalProductHash);
    return Ok(products);
  }
}
```

**What happens here:**
- `Task<IActionResult>` signals an asynchronous action method.
- `await` pauses method execution without blocking the request thread.
- While awaiting I/O, ASP.NET Core can reuse that thread to serve other requests.

### Practical Guidance

- Use async end-to-end for I/O paths (`Controller -> Service -> Repository -> DB/HTTP`).
- Return `Task` or `Task<T>` from async methods (TAP pattern).
- Avoid `Task.Run` for I/O in ASP.NET Core.
- Use standard `try/catch` around awaited calls for exception handling.

---

## Summary

| Concept | Key Point |
|---------|-----------|
| **Async/Await** | Enables responsive applications without dedicated threads per operation |
| **I/O-Bound** | Use async APIs directly; don't wrap with Task.Run |
| **CPU-Bound** | Use Task.Run to offload to thread pool |
| **Task** | Universal abstraction for any asynchronous operation |
| **ConfigureAwait** | Use in library code to avoid unnecessary context switches |
| **ASP.NET** | Focus on I/O async; avoid Task.Run for CPU work |
| **Threads** | Await doesn't create threads; task scheduler manages execution |
