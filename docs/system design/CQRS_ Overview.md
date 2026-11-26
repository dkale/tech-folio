# CQRS

## Table of Contents

- [What Problem CQRS Solves](#what-problem-cqrs-solves)
- [How CQRS Works](#how-cqrs-works)
- [C# Implementation Example](#c-implementation-example)
- [Key Benefits](#key-benefits)
- [When Not to Use CQRS](#when-not-to-use-cqrs)
- [How CQRS Helps Avoid Race Conditions](#how-cqrs-helps-avoid-race-conditions)
- [1. Isolating the Write Model](#1-isolating-the-write-model)
- [2. Applying Optimistic Concurrency](#2-applying-optimistic-concurrency)
- [3. Pairing CQRS with Event Sourcing](#3-pairing-cqrs-with-event-sourcing)
- [4. Queueing and Serializing Commands](#4-queueing-and-serializing-commands)
- [Important Caveat: Eventual Consistency](#important-caveat-eventual-consistency)
- [Bottom Line](#bottom-line)

CQRS stands for Command Query Responsibility Segregation. It is an architectural pattern that separates reading data from writing data.

## What Problem CQRS Solves

Traditional application architectures often use the same model and database workflow for both reads and writes. In simple systems that can work well, but in more complex systems it creates several problems:

- Data contention: reads and writes compete for the same tables, which can increase locking and reduce performance.
- Complex mapping: read-heavy screens often need expensive joins and projections that do not fit cleanly into the write-focused domain model.
- Security misalignment: it becomes harder to expose read-only data safely when the same model is also responsible for mutations.
- Scalability mismatch: most applications read data far more often than they write it, but a shared model forces both workloads to scale together.

## How CQRS Works

CQRS separates the application into two distinct paths:

```text
                  ┌───► Commands (Write Model) ───► Write Database
                  │
Client / API ─────┤
                  │
                  └───► Queries  (Read Model)  ───► Read Database
```

- Commands handle writes. These operations change state, such as create, update, and delete. They focus on business rules, validation, and data integrity.
- Queries handle reads. These operations fetch data without changing state. They typically return flat, optimized DTOs tailored to UI or reporting needs.

## C# Implementation Example

In C#, CQRS is often implemented with MediatR so commands and queries remain decoupled from controllers and services.

### Command Side (Write Model)

```csharp
public record CreateProductCommand(string Name, decimal Price) : IRequest<int>;

public class CreateProductHandler : IRequestHandler<CreateProductCommand, int>
{
    private readonly WriteDbContext _context;

    public CreateProductHandler(WriteDbContext context) => _context = context;

    public async Task<int> Handle(CreateProductCommand request, CancellationToken cancellationToken)
    {
        var product = new Product
        {
            Name = request.Name,
            Price = request.Price
        };

        _context.Products.Add(product);
        await _context.SaveChangesAsync(cancellationToken);
        return product.Id;
    }
}
```

### Query Side (Read Model)

```csharp
public record GetProductByIdQuery(int Id) : IRequest<ProductDto>;

public class GetProductByIdHandler : IRequestHandler<GetProductByIdQuery, ProductDto>
{
    private readonly ReadDbContext _context;

    public GetProductByIdHandler(ReadDbContext context) => _context = context;

    public async Task<ProductDto> Handle(GetProductByIdQuery request, CancellationToken cancellationToken)
    {
        return await _context.Products
            .Where(product => product.Id == request.Id)
            .Select(product => new ProductDto(product.Id, product.Name, product.Price))
            .FirstOrDefaultAsync(cancellationToken);
    }
}
```

## Key Benefits

- Optimized performance: the read side can be tuned for query speed, while the write side can stay normalized and focused on correctness.
- Independent scaling: read services can scale horizontally without forcing the write side to do the same.
- Cleaner code: commands and queries each have one responsibility, which makes behavior easier to understand and test.

## When Not to Use CQRS

CQRS is usually a poor fit for simple CRUD applications. It adds boilerplate, increases operational complexity, and often introduces eventual consistency concerns that are not worth the tradeoff for smaller systems.

---

# How CQRS Helps Avoid Race Conditions

CQRS does not eliminate race conditions by itself. What it does provide is a clean separation that makes concurrency problems easier to isolate and solve.

In a CQRS system, all state changes pass through the command side. That means concurrency protections can be concentrated in one place instead of being spread across read and write logic.

## 1. Isolating the Write Model

In a traditional shared model, race conditions often happen when one request reads stale data and then writes back over a newer value.

CQRS reduces that risk by separating responsibilities:

- Queries are read-only. They do not mutate state and generally do not compete with writes in the same pipeline.
- Commands are write-only. All business rules and state transitions happen in a controlled path.

This isolation does not solve concurrency automatically, but it makes it much easier to enforce concurrency rules consistently.

## 2. Applying Optimistic Concurrency

Because commands own all writes, the write model can use optimistic concurrency control with version numbers or timestamps.

If two commands try to update the same record at the same time, the second write can be rejected because its version is no longer current.

```csharp
public class Account
{
    public int Id { get; set; }
    public decimal Balance { get; set; }

    [Timestamp]
    public byte[] RowVersion { get; set; }
}

public async Task Handle(WithdrawMoneyCommand command)
{
    var account = await _writeDbContext.Accounts.FindAsync(command.AccountId);
    account.Balance -= command.Amount;

    try
    {
        await _writeDbContext.SaveChangesAsync();
    }
    catch (DbUpdateConcurrencyException)
    {
        // Another write won the race. Retry, reject, or notify the caller.
    }
}
```

## 3. Pairing CQRS with Event Sourcing

CQRS is often paired with event sourcing for stronger concurrency guarantees.

Instead of storing only current state, the system stores an append-only stream of events such as:

- `AccountOpened`
- `MoneyDeposited`
- `MoneyWithdrawn`

When a command executes:

1. It rebuilds current state from the event stream.
2. It appends a new event with an expected version.
3. If another command has already written a newer event, the append fails.

That expected-version check prevents concurrent writers from silently overwriting each other.

## 4. Queueing and Serializing Commands

Because commands are explicit messages, they can also be queued and processed in order.

For high-contention scenarios, systems often serialize commands per entity or aggregate. For example, all commands for the same account can be processed one at a time through a queue. This turns uncontrolled parallel updates into a predictable sequence and can eliminate many write-side race conditions.

## Important Caveat: Eventual Consistency

CQRS improves control over write-side concurrency, but it introduces a separate tradeoff on the read side: eventual consistency.

For example, a username change may succeed immediately on the write model, while the read model updates a short time later. During that delay, the user may briefly see stale data. That is not a race condition, but it is a consistency lag that the system design must account for.

## Bottom Line

CQRS helps avoid race conditions by making writes explicit and centralized. It does not remove concurrency problems automatically, but it makes it much easier to apply the right protections, such as:

- optimistic concurrency checks
- event-store version enforcement
- serialized command processing
- stricter write-side business rules

The main benefit is not that CQRS magically prevents races. The benefit is that it gives you a clear place to control them.