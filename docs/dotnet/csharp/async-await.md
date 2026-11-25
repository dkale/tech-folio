# Async/Await

> Use async/await to write asynchronous operations with readable control flow.

## Overview

The `async` and `await` keywords in C# make promise-like operations easier to compose without blocking a thread while work is pending.

## Key Concepts

- `async` marks a method that can contain `await` expressions.
- `await` pauses the method until the awaited task completes.
- Prefer `Task` or `Task<T>` for asynchronous APIs.
- `ConfigureAwait(false)` can be useful in library code.

## Example

```csharp
public async Task<string> DownloadAsync(HttpClient client, string url)
{
    using HttpResponseMessage response =
        await client.GetAsync(url).ConfigureAwait(false);

    response.EnsureSuccessStatusCode();
    return await response.Content.ReadAsStringAsync().ConfigureAwait(false);
}
```

## Best Practices

- Avoid blocking with `.Result` or `.Wait()`.
- Propagate cancellation tokens when appropriate.
- Dispose resources deterministically.
- Do not use async void except for event handlers.

!!! warning
    Awaiting a task does not necessarily complete work on the same thread. Preserve synchronization context only when required by the application model.
