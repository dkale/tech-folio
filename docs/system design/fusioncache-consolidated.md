# FusionCache in .NET and This Workspace

FusionCache is an open-source .NET caching library that provides a hybrid caching model, resiliency features, and coordinated cache access. In this workspace, it is used primarily as a shared RavenDB caching layer backed by Redis and exposed through `IRavenDbCacheService`.

## Table of Contents

- [What Is FusionCache?](#what-is-fusioncache)
- [The Smart Bakery: A Mental Model](#the-smart-bakery-a-mental-model)
- [Core Capabilities](#core-capabilities)
  - [Cache Stampede Protection](#cache-stampede-protection)
  - [Fail-Safe Mode](#fail-safe-mode)
  - [Multi-Node Backplane Synchronization](#multi-node-backplane-synchronization)
- [FusionCache and Redis](#fusioncache-and-redis)
  - [Quick Comparison](#quick-comparison)
- [Basic C# Usage](#basic-c-usage)
  - [Execution Flow](#execution-flow)
- [Cache Invalidation and Update Strategies](#cache-invalidation-and-update-strategies)
  - [Proactive Invalidation or Update](#proactive-invalidation-or-update)
  - [Distributed Backplane Workflow](#distributed-backplane-workflow)
  - [Reactive Expiration](#reactive-expiration)
- [FusionCache in This Workspace](#fusioncache-in-this-workspace)
  - [Where It Is Defined](#where-it-is-defined)
  - [How It Is Configured](#how-it-is-configured)
  - [Shared Abstraction](#shared-abstraction)
  - [Usage in CAT.MDM.Parties](#usage-in-catmdmparties)
  - [Important Distinction: Not All Caching Is FusionCache](#important-distinction-not-all-caching-is-fusioncache)
- [Summary Comparison](#summary-comparison)
- [Key Takeaway](#key-takeaway)

## What Is FusionCache?

**FusionCache** is an open-source, ultra-fast caching library for .NET applications designed to handle high-density, real-time caching scenarios efficiently. It provides advanced resiliency features, multi-level caching (L1/L2), cache stampede protection, and a single API for cache reads, writes, and invalidation.

FusionCache combines:

- In-memory caching for fast local reads.
- Distributed caching for shared cache state across application instances.
- Backplane support so cache invalidations can propagate between nodes.
- A unified API for cache reads, writes, and invalidation.

In this workspace, FusionCache is used as part of the shared RavenDB store layer rather than as a generic cache sprinkled directly throughout every application.

## The Smart Bakery: A Mental Model

To understand FusionCache, think of it as a smart manager at a busy bakery:

1. **The Display Counter (L1 cache - in-memory):** Items kept right in front of the cashier. It takes 0 seconds to hand a croissant to a customer.
2. **The Backroom Storage (L2 cache - distributed, such as Redis):** Extra stock stored in the back room. It takes 5 seconds to retrieve, but it holds more items and is shared across servers.
3. **The Kitchen (database):** Baking a fresh croissant from scratch. It takes 20 minutes.

FusionCache coordinates all three layers:

- It checks the display counter first.
- If the item is missing, it checks the backroom storage and copies the item to the counter.
- If the item is missing there too, it asks the kitchen to prepare a fresh item and fills both cache layers.

## Core Capabilities

Beyond basic caching, FusionCache addresses three major architectural challenges.

### Cache Stampede Protection

A **cache stampede**, also called a **thundering herd**, occurs when a popular item expires during peak traffic and many users request it at almost the same time. Without protection, all 1,000 requests might query the database simultaneously and cause a failure or severe load spike.

FusionCache provides request coalescing. Only one request goes to the database to fetch the data while the other requests wait briefly for that same result.

### Fail-Safe Mode

If the database goes down or becomes slow after a cache entry expires, an ordinary cache may return an error. FusionCache can detect the database problem and serve slightly expired, or stale, data as a fallback until the database recovers.

### Multi-Node Backplane Synchronization

In a multi-server deployment, Server A may update a product price while Server B still has the old value in its local memory cache.

FusionCache can use a messaging channel called a **backplane**, typically Redis or NATS, to notify other servers immediately that an entry changed. The other servers can then clear their local L1 entries and avoid serving outdated data.

## FusionCache and Redis

Redis and FusionCache are different layers in the caching stack:

- **Redis** is a cache server where data can be stored and shared across application instances.
- **FusionCache** is an application library that manages cache behavior and can use memory cache, Redis, and a backplane together.

In short:

- Redis answers where cached data lives.
- FusionCache answers how the application reads, writes, and coordinates cache entries.

In this workspace, they are commonly combined as follows:

- Local memory cache for the fastest reads.
- Redis distributed cache for shared cache state across instances.
- Redis backplane to propagate invalidations between instances.

This is why the shared RavenDB store wiring contains FusionCache configuration that points at Redis.

### Quick Comparison

| Aspect | Redis Cache | FusionCache |
| :--- | :--- | :--- |
| What it is | Distributed data store/cache server | .NET caching library/API |
| Primary role | Stores shared cached data | Orchestrates cache behavior |
| Runs where | External service/process | Inside the application |
| Scope | Shared across instances | Memory plus distributed cache coordination |
| Network hop | Usually yes | Memory path has no network hop; distributed path does |
| Typical use in this workspace | Distributed cache store and backplane | RavenDB document/query caching wrapper |

## Basic C# Usage

Here is a basic FusionCache example:

```csharp
using ZiggyCreatures.Caching.Fusion;

// 1. Create a FusionCache instance
var cache = new FusionCache(new FusionCacheOptions());

// 2. Get or fetch data
string productId = "product-42";

var product = await cache.GetOrSetAsync<Product>(
    key: $"product:{productId}",
    factory: async (context, cancellationToken) =>
    {
        // This code runs only if the item is missing from the cache.
        // It fetches data from the slow database.
        return await database.GetProductByIdAsync(productId);
    },
    duration: TimeSpan.FromMinutes(10) // Cache for 10 minutes
);
```

### Execution Flow

1. **First request:** FusionCache checks L1 memory -> miss -> runs the factory, which queries the database -> saves the result to the cache -> returns the result.
2. **Subsequent requests:** FusionCache checks L1 memory -> hit -> immediately returns the result without touching the database.

## Cache Invalidation and Update Strategies

When data changes in the database, the related cache entry must be updated or invalidated so stale data is not served.

### Proactive Invalidation or Update

Proactive invalidation or update is the recommended strategy when the application knows that data has changed.

#### Option A: Explicit Removal with `RemoveAsync`

This deletes the item from the cache. The next request triggers a fresh database query.

```csharp
public async Task UpdateProductPriceAsync(string productId, decimal newPrice)
{
    // 1. Update the database
    await database.UpdatePriceAsync(productId, newPrice);

    // 2. Invalidate the cache entry
    await cache.RemoveAsync($"product:{productId}");
}
```

#### Option B: Eager Overwrite with `SetAsync`

This directly overwrites the cached value with the new state, avoiding a database hit on the next read.

```csharp
public async Task UpdateProductAsync(Product updatedProduct)
{
    // 1. Update the database
    await database.UpdateAsync(updatedProduct);

    // 2. Overwrite the cache entry directly
    await cache.SetAsync(
        key: $"product:{updatedProduct.Id}",
        value: updatedProduct,
        duration: TimeSpan.FromMinutes(10)
    );
}
```

### Distributed Backplane Workflow

In a multi-server setup, calling invalidation methods can propagate notifications across all instances through a backplane:

```text
                       [ Database Updated ]
                                |
                           (Server A)
                   Executes cache.RemoveAsync()
                                |
             [ Backplane / Redis Message Bus ]
                      /                  \
              Notifies Server B       Notifies Server C
                      |                          |
       Clears Local L1 Memory      Clears Local L1 Memory
```

### Reactive Expiration

Reactive expiration is a fallback for changes that occur outside the primary application, such as direct SQL updates or writes from an external service.

1. **Time-to-live (TTL):** A hard expiration limit, such as 10 minutes. Once reached, the data is automatically refetched on the next request.
2. **Soft expiration/background refresh:** Define a soft TTL, such as 5 minutes, alongside a hard TTL of 10 minutes. Requests arriving after the soft TTL receive stale data immediately while a background process asynchronously refreshes the cache from the database.

## FusionCache in This Workspace

FusionCache in this workspace is primarily a shared RavenDB caching layer implemented in CAT.Foundation and actively used by CAT.MDM.Parties.

### Where It Is Defined

The shared implementation lives in CAT.Foundation, specifically in the RavenDB store package.

Relevant files:

- `Foundation.Stores.RavenDb.csproj`
- `RavenStoreServiceCollectionExtensions.cs`
- `IRavenDbCacheService.cs`
- `RavenDbCacheService.cs`

The package references show that the shared store layer depends on:

- `ZiggyCreatures.FusionCache`
- `ZiggyCreatures.FusionCache.Backplane.StackExchangeRedis`
- `ZiggyCreatures.FusionCache.Serialization.NewtonsoftJson`

### How It Is Configured

FusionCache is registered in the shared service collection extension:

- `RavenStoreServiceCollectionExtensions.cs`

That registration:

- Adds FusionCache to dependency injection.
- Sets a cache key prefix.
- Configures Newtonsoft.Json serialization.
- Uses Redis as the distributed cache store.
- Uses a Redis backplane for cross-instance cache invalidation.
- Registers the shared `IRavenDbCacheService` abstraction.

Consumers do not usually talk to FusionCache directly. They go through the shared RavenDB caching service.

### Shared Abstraction

The main workspace abstraction is `IRavenDbCacheService`:

- `IRavenDbCacheService.cs`

It exposes methods for:

- Loading individual RavenDB documents through the cache.
- Loading multiple documents through the cache.
- Caching RavenDB query results.
- Invalidating specific cache entries.
- Clearing the distributed cache scope.

The implementation is in:

- `RavenDbCacheService.cs`

Internally, that implementation uses FusionCache `GetOrSetAsync`, `RemoveAsync`, and `ClearAsync`.

### Usage in CAT.MDM.Parties

The clearest active usage in this workspace is in CAT.MDM.Parties.

#### Registration

FusionCache-backed RavenDB caching is wired into startup here:

- `ServicesConfigExtensions.cs`
- `ServiceCollectionExtensions.cs`

#### Cache Options

Cache behavior is controlled through:

- `CachingOptions.cs`

This class builds FusionCache entry options and can disable caching completely by skipping:

- Distributed cache reads.
- Distributed cache writes.
- Memory cache reads.
- Memory cache writes.

#### Document-Level Caching

A direct document load is cached in:

- `PartiesClient.cs`

This path loads a `Party` document through `IRavenDbCacheService` instead of hitting RavenDB every time.

#### Query-Level Caching

Shared party queries are cached in:

- `PartySharedService.cs`

These methods use `IRavenDbCacheService.CachedQueryAsync` to cache the results of RavenDB queries.

The read model repository also uses FusionCache-backed calls for:

- Existence checks.
- Selected read-model queries.

Example:

- `PartiesReadModelRepository.cs`

### Important Distinction: Not All Caching Is FusionCache

This workspace also contains a separate Redis-based cache abstraction that is not FusionCache:

- `RedisCacheManager.cs`

For example, CAT.MDM.Parties also uses `ICacheManager` directly in some places for distributed cached values beyond the RavenDB FusionCache wrapper.

Therefore, two different caching patterns are in play:

- FusionCache through `IRavenDbCacheService` for RavenDB document and query caching.
- A separate Redis cache manager for some distributed cache scenarios.

## Summary Comparison

| Feature / Method | Description / Behavior |
| :--- | :--- |
| **L1/L2 caching** | Combines fast in-memory execution with shared distributed backup. |
| **Cache stampede protection** | Coalesces multiple concurrent cache misses into a single database read call. |
| **Fail-safe mode** | Serves stale cache data when database calls fail or time out. |
| **`RemoveAsync()`** | Immediately deletes a cache key and forces a database query on the next read. |
| **`SetAsync()`** | Immediately overwrites a cache key with an updated data object. |
| **Backplane synchronization** | Broadcasts cache updates or invalidations across instances using publish/subscribe messaging. |
| **RavenDB cache service** | Wraps FusionCache behind `IRavenDbCacheService` for document and query caching in this workspace. |
| **Separate Redis cache manager** | Handles some distributed cached values outside the RavenDB FusionCache wrapper. |

## Key Takeaway

FusionCache is not being used here as a random utility cache. It is wrapped behind `IRavenDbCacheService` and forms part of the shared data-access infrastructure.

At a high level, the workspace uses FusionCache for:

- Caching RavenDB document loads.
- Caching RavenDB query results.
- Using local memory for fast reads.
- Using Redis for shared distributed cache state.
- Propagating invalidation across instances with a Redis backplane.
- Providing cache stampede protection and fail-safe behavior where configured.
