# How to prevent duplicate entity creation in micro-services architecture?

## Table of Contents
- [Centralized Deduplication with Idempotency Key](#1-centralized-deduplication-with-idempotency-key)
- [Distributed Lock Mechanism](#2-distributed-lock-mechanism)
- [Eventual Consistency via Party Service](#3-eventual-consistency-via-party-service)
- [Database-Driven Deduplication](#4-database-driven-deduplication)
- [Choosing the Right Approach](#choosing-the-right-approach)
- [What Is Idempotency Key?](#what-is-idempotency-key)
- [Scenario Recap](#scenario-recap)
- [Approach Using Idempotency Keys](#approach-using-idempotency-keys)
- [Optional Enhancements](#optional-enhancements)
- [Key Considerations](#key-considerations)

I have a Sales service which receives a distinct list of (string [] SourceIDs). For each sourceId, the sales service sends out a command to create a Party to the Party service. I also have an Inventory service which also receives a distinct list of (string [] SourceIDs). For each sourceId, the inventory service sends out a command to create a Party to the Party service. Both Sales and Inventory service processes each of the sourceIds in parallel. There might be a scenario where Sales and Inventory both have the same sourceId. In this case we should not be creating two parties. How can we prevent such a thing from happening?

To prevent duplicate creation of parties when both Sales and Inventory services send commands for the same sourceId, you can implement a distributed lock mechanism or a deduplication strategy. Here's how you can approach this:

## 1. Centralized Deduplication with Idempotency Key

### How it works

Use a centralized database or cache (e.g., Redis, SQL, etc.) to store the sourceId as a key before creating a party.

When either Sales or Inventory tries to create a party for a sourceId, they first check if the sourceId exists in the store:

If it exists, skip the party creation.

If it doesn’t exist, insert the sourceId atomically and proceed with party creation.

### Implementation

Both services send a command to the Party service with an idempotency key (the sourceId).

The Party service ensures that party creation for a given sourceId is performed only once.

```csharp
public async Task<bool> TryCreatePartyAsync(string sourceId, Party partyData)
{
    // Check if the sourceId already exists
    var exists = await _cacheService.ExistsAsync(sourceId); // Redis or Database check
    if (exists) return false;

    // Atomically add sourceId and create party
    var added = await _cacheService.AddIfNotExistsAsync(sourceId, TimeSpan.FromHours(1));
    if (!added) return false;

    // Proceed to create the party
    await _partyRepository.CreateAsync(partyData);
    return true;
}
```

## 2. Distributed Lock Mechanism

### How it works

Use a distributed lock system (e.g., Redis, Consul, or Zookeeper) to ensure that only one process (either Sales or Inventory) can handle a specific sourceId at a time.

Both services attempt to acquire a lock on the sourceId:

If the lock is acquired, proceed with party creation and release the lock after completion.

If the lock cannot be acquired (e.g., another service is already processing it), skip the operation.

### Implementation

Redis is commonly used for distributed locks using the SET NX command.

```csharp
public async Task CreatePartyWithLockAsync(string sourceId, Party partyData)
{
    // Attempt to acquire a distributed lock
    var lockKey = $"party-lock-{sourceId}";
    var lockAcquired = await _lockService.TryAcquireLockAsync(lockKey, TimeSpan.FromSeconds(30));
    if (!lockAcquired) return;

    try
    {
        // Check if the party already exists
        if (await _partyRepository.ExistsAsync(sourceId)) return;

        // Create the party
        await _partyRepository.CreateAsync(partyData);
    }
    finally
    {
        // Release the lock
        await _lockService.ReleaseLockAsync(lockKey);
    }
}
```

## 3. Eventual Consistency via Party Service

### How it works

Both Sales and Inventory services enqueue events (e.g., CreatePartyCommand) into a message queue or event stream (e.g., RabbitMQ, Kafka).

The Party service processes the commands sequentially, ensuring deduplication by checking the sourceId before creating a party.

### Implementation

Sales and Inventory publish CreatePartyCommand events.

The Party service consumes these events and handles deduplication.

```csharp
public async Task HandleCreatePartyCommandAsync(CreatePartyCommand command)
{
    if (await _partyRepository.ExistsAsync(command.SourceId)) return;

    // Create the party
    await _partyRepository.CreateAsync(command.PartyData);
}
```

## 4. Database-Driven Deduplication

### How it works

Use the database itself to ensure uniqueness by enforcing a unique constraint on the sourceId.

If both Sales and Inventory services attempt to insert a party with the same sourceId concurrently, one will succeed while the other fails with a unique constraint violation.

### Implementation

Add a unique index on the sourceId column in the Party table.

Handle the unique constraint violation gracefully.

```sql
CREATE TABLE Party (
    Id INT PRIMARY KEY,
    SourceId VARCHAR(255) UNIQUE,
    ... -- other columns
);
```

```csharp
public async Task CreatePartyAsync(Party party)
{
    try
    {
        await _partyRepository.AddAsync(party);
    }
    catch (SqlException ex) when (ex.Number == 2627) // Unique constraint violation
    {
        // Handle duplicate party creation gracefully
    }
}
```

## Choosing the Right Approach

For high performance: Use Redis for idempotency checks or distributed locks.

For strong consistency: Use the Database-Driven Deduplication method with unique constraints.

For event-driven systems: Use Eventual Consistency via Party Service with a message queue.

## What Is Idempotency Key?

An idempotency key is a unique identifier that ensures a specific operation (like creating a Party) is executed only once, even if the request is repeated multiple times. It's commonly used in distributed systems to prevent duplicate actions caused by retries or concurrent requests.

How It Works:

Generate a Key:

Use a unique value as the idempotency key, such as the sourceId in your scenario.

This key represents the specific operation.

Store the Key:

When the request to create a Party is received, the Party service checks if the idempotency key already exists in a centralized store (e.g., database, Redis, etc.).

If it exists, the operation is skipped or treated as a duplicate.

Perform the Operation:

If the key doesn't exist, the operation is performed, and the idempotency key is stored to indicate that the action has been processed.

Ensure Atomicity:

The insertion of the idempotency key and the creation of the Party must be atomic, ensuring that they either both succeed or both fail.

### Benefits

Prevents duplicate processing of the same operation.

Enables safe retries in the event of network issues or failures.

Simplifies concurrency handling by centralizing deduplication logic.

### Example with Redis

```csharp
public async Task<bool> TryCreatePartyAsync(string sourceId, Party partyData)
{
    // Check if the sourceId (idempotency key) already exists
    bool isDuplicate = await _cache.ExistsAsync(sourceId);
    if (isDuplicate) return false; // Skip duplicate

    // Atomically add idempotency key and process the party creation
    bool added = await _cache.SetIfNotExistsAsync(sourceId, "processed", TimeSpan.FromHours(1));
    if (!added) return false;

    // Proceed to create the party
    await _partyRepository.CreateAsync(partyData);
    return true;
}
```

### Idempotency Key in APIs

Many APIs use idempotency keys in headers to ensure that retrying a request doesn’t result in duplicate actions. For example:

Stripe API: Clients send an Idempotency-Key header with the request. If a request with the same key has already been processed, the server ignores subsequent retries.

Would you like a deeper dive into how this could be applied to your services?

Yes

Let’s break it down step-by-step to see how idempotency keys can be applied to your scenario and integrated with your Sales and Inventory services to prevent duplicate Party creation.

### Scenario Recap

Both Sales and Inventory services receive distinct lists of SourceIDs (e.g., string[] SourceIDs).

For each sourceId, they send a command to the Party service to create a Party.

There may be overlap in sourceId between the two services, but you want only one Party to be created per unique sourceId.

### Approach Using Idempotency Keys

Step 1: Define the Idempotency Key

Use the sourceId as the idempotency key, as it uniquely identifies the operation of creating a Party for a given source.

Step 2: Centralized Store for Idempotency Keys

Use a centralized, fast-access storage mechanism, such as Redis or a relational database table, to track processed sourceIds.

Implementation Example:

### Party Service with Idempotency Logic

This service will handle incoming requests and ensure deduplication.

```csharp
public class PartyService
{
    private readonly IRedisCacheService _cache; // Or other distributed store
    private readonly IPartyRepository _partyRepository;

    public PartyService(IRedisCacheService cache, IPartyRepository partyRepository)
    {
        _cache = cache;
        _partyRepository = partyRepository;
    }

    public async Task<bool> TryCreatePartyAsync(string sourceId, Party partyData)
    {
        // Step 1: Check if the sourceId has already been processed
        var isDuplicate = await _cache.ExistsAsync(sourceId);
        if (isDuplicate)
        {
            Console.WriteLine($"Party creation skipped for duplicate sourceId: {sourceId}");
            return false; // Skip duplicate processing
        }

        // Step 2: Atomically set idempotency key in cache
        var added = await _cache.SetIfNotExistsAsync(sourceId, "processed", TimeSpan.FromDays(1));
        if (!added)
        {
            Console.WriteLine($"Another process is already handling sourceId: {sourceId}");
            return false;
        }

        // Step 3: Create the Party
        Console.WriteLine($"Creating Party for sourceId: {sourceId}");
        await _partyRepository.CreateAsync(partyData);

        return true; // Successfully created
    }
}
```

### Redis Cache Implementation

You can use Redis for storing idempotency keys efficiently.

```csharp
public class RedisCacheService : IRedisCacheService
{
    private readonly IConnectionMultiplexer _redis;

    public RedisCacheService(IConnectionMultiplexer redis)
    {
        _redis = redis;
    }

    public async Task<bool> ExistsAsync(string key)
    {
        var db = _redis.GetDatabase();
        return await db.KeyExistsAsync(key);
    }

    public async Task<bool> SetIfNotExistsAsync(string key, string value, TimeSpan expiration)
    {
        var db = _redis.GetDatabase();
        return await db.StringSetAsync(key, value, expiration, when: When.NotExists);
    }
}
```

### Sales and Inventory Services

Both services call the TryCreatePartyAsync method of the Party service for each sourceId.

```csharp
public class SalesService
{
    private readonly IPartyService _partyService;

    public SalesService(IPartyService partyService)
    {
        _partyService = partyService;
    }

    public async Task ProcessSourceIdsAsync(string[] sourceIds)
    {
        var tasks = sourceIds.Select(async sourceId =>
        {
            var partyData = new Party { SourceId = sourceId, Name = "Sales Party" };
            await _partyService.TryCreatePartyAsync(sourceId, partyData);
        });

        await Task.WhenAll(tasks);
    }
}
```

Similarly, the InventoryService would work the same way, sending its own Party data.

## Optional Enhancements

## 1. Database-Backed Idempotency

If Redis isn't an option, you can use a database table with a unique constraint on sourceId.

```sql
CREATE TABLE IdempotencyKeys (
    SourceId VARCHAR(255) PRIMARY KEY,
    CreatedAt DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

When inserting a new sourceId, handle the unique constraint violation to ensure atomicity.

## 2. Eventual Consistency via Message Queue

If the Sales and Inventory services publish events to a message queue (e.g., RabbitMQ, Kafka), you can ensure deduplication within the Party service's event consumer.

Sales and Inventory publish CreatePartyCommand events.

The Party service listens to these events and deduplicates based on sourceId.

```csharp
public class PartyEventConsumer
{
    private readonly IPartyService _partyService;

    public PartyEventConsumer(IPartyService partyService)
    {
        _partyService = partyService;
    }

    public async Task HandleEventAsync(CreatePartyCommand command)
    {
        await _partyService.TryCreatePartyAsync(command.SourceId, command.PartyData);
    }
}
```

## Key Considerations

Concurrency: The idempotency store (Redis or database) must support atomic operations (e.g., SET NX in Redis or a unique constraint in a database).

Performance: Redis is faster and more suitable for high-throughput scenarios compared to a database.

TTL for Idempotency Keys: Set a reasonable expiration time for idempotency keys to clean up old records and save storage.

Would you like detailed code for database-based deduplication or event-driven processing?

[↑ Back to Top](#table-of-contents)
