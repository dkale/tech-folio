# Scoped lifetimes in a Console App

## Table of Contents

#### Scoped lifetimes in a console app — in one view
1. OS processes isolate app runs
 Running the console app twice gives two separate processes. This isolation is at application level, not inside a single run.
1. Scopes solve an in‑process problem
 A scope defines a lifetime boundary for a logical unit of work inside the same process (job, file, batch, message, command, loop iteration).
1. Scoped ≠ HTTP request
 HTTP requests are just one example of a scope.
 In console apps, you create scopes explicitly using CreateScope().
1. Why Scoped exists (and why Transient/Singleton aren’t enough)
1. Singleton: shared forever → state leaks, concurrency issues
1. Transient: new instance every resolve → no shared unit of work
1. Scoped: shared within one operation, disposed after it
1. Classic example: DbContext
1. One DbContext per unit of work
1. Shared across multiple services inside that work
1. Disposed cleanly when the work ends
 (This is why EF Core defaults to Scoped—even in console apps.)
1. Core benefit


 Scoped lets you isolate work, manage resources correctly, and prevent state leakage within a single long‑running process.
EF Core's DbContext is scoped by design — it tracks entity changes in memory (the "Unit of Work" pattern) and is meant to live for one logical operation, then be disposed. Key reasons:
Change tracking isolation — each pipeline run (one retailer's extraction or construction) gets its own DbContext with its own change tracker. If it were singleton, entity states from Retailer A's run would leak into Retailer B's.
Connection management — a scoped DbContext opens and closes its SQL connection within the scope. A singleton would hold a connection open for the entire app lifetime, risking timeouts and connection pool exhaustion.
Thread safety — DbContext is not thread-safe. The orchestrator runs multiple retailers in parallel (Parallel.ForEachAsync). Each parallel worker creates its own DI scope, getting its own DbContext and IStagingRepository instance. A singleton would be shared across threads and crash.
Memory — the change tracker accumulates entities. A singleton processing thousands of invoices across all retailers would consume growing memory. Scoped instances get GC'd after each scope ends.
The practical flow (with reference to the AgData’s Transmittal ERP project):
Parallel.ForEachAsync (per retailer)
  → new DI scope
    → new StagingDbContext (own connection, own change tracker)
    → new EfStagingRepository (wraps that DbContext)
    → extract/construct pipeline runs
  → scope disposed → DbContext disposed → connection returned to pool
If you registered it as singleton, you'd get a runtime exception: "Cannot consume scoped service 'StagingDbContext' from singleton 'IStagingRepository'." .NET enforces this by default via scope validation

#### One‑line takeaway
Process isolation separates app runs; scopes isolate units of work inside a run, control resource lifetimes, and prevent state and resource leakage within a single process. Scoped lifetimes give correctness, resource control, and clean boundaries that process isolation alone cannot provide.


[↑ Back to Top](#table-of-contents)
