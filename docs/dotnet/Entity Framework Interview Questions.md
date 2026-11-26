# Entity Framework Interview Questions

This interview guide covers Entity Framework change tracking, query execution, transactions, bulk operations, concurrency, pagination, and related performance concepts.

## Table of Contents

1. [Change Tracking and Tracking Options](#1-change-tracking-and-tracking-options)
2. [Eager Loading With Include](#2-eager-loading-with-include)
3. [Transactions](#3-transactions)
4. [Bulk Operations](#4-bulk-operations)
5. [Single and First Query Methods](#5-single-and-first-query-methods)
6. [Deferred Execution and ToList](#6-deferred-execution-and-tolist)
7. [SaveChanges Workflow](#7-savechanges-workflow)
8. [Shadow Properties](#8-shadow-properties)
9. [Concurrency Handling](#9-concurrency-handling)
10. [Dataset Pagination](#10-dataset-pagination)

## 1. Change Tracking and Tracking Options

What are the key differences between `AsNoTracking()`, `AsNoTrackingWithIdentityResolution()`, and `TrackGraph()`? When would you use each?

### Concepts Tested

- Entity Framework change tracking
- Performance optimization
- Detached entities

### Hint

- `AsNoTracking()` improves performance by disabling change tracking.
- `AsNoTrackingWithIdentityResolution()` ensures that the same entity appears only once in the query result.
- `TrackGraph()` allows custom change tracking when reattaching entities.

[↑ Back to Top](#table-of-contents)

## 2. Eager Loading With Include

What are potential issues with using `Include()` for eager loading in Entity Framework, and how can you mitigate them?

### Concepts Tested

- Lazy loading
- Eager loading
- Performance optimization
- Circular references

### Hint

- Using multiple `Include()` calls can lead to a Cartesian explosion because too many rows are returned.
- Use projection with `Select()` or use filtered `Include()` and `ThenInclude()` calls to limit the data.

[↑ Back to Top](#table-of-contents)

## 3. Transactions

How does Entity Framework handle transactions internally, and when should you manually use `DbContext.Database.BeginTransaction()`?

### Concepts Tested

- Automatic transactions
- Explicit transactions
- Rollback scenarios

### Hint

- Entity Framework automatically wraps `SaveChanges()` in a transaction.
- Use a manual transaction for batch updates or cross-context operations.

[↑ Back to Top](#table-of-contents)

## 4. Bulk Operations

How can you optimize bulk operations in Entity Framework?

### Concepts Tested

- Performance
- Entity Framework bulk insert and update operations
- `SaveChanges()` optimization

### Hint

- Use `AddRange()` and `SaveChanges(false)` to batch inserts.
- Consider third-party libraries such as EFCore.BulkExtensions for performance.

[↑ Back to Top](#table-of-contents)

## 5. Single and First Query Methods

What are the differences among `Single()`, `SingleOrDefault()`, `First()`, and `FirstOrDefault()`? Which one should you prefer?

### Concepts Tested

- Query optimization
- Exception handling
- Performance trade-offs

### Hint

- `Single()` throws an exception if multiple matching results exist.
- `First()` is generally preferred for performance.

[↑ Back to Top](#table-of-contents)

## 6. Deferred Execution and ToList

Why should you avoid using `ToList()` inside LINQ queries?

### Concepts Tested

- Deferred execution
- Premature execution
- Performance issues

### Hint

- `ToList()` forces query execution early, making subsequent filtering inefficient.
- Keep queries deferred until their data is needed.

[↑ Back to Top](#table-of-contents)

## 7. SaveChanges Workflow

What happens when you call `SaveChanges()`? Explain the internal Entity Framework workflow.

### Concepts Tested

- Change tracking
- Database transactions
- Command batching

### Hint

1. Entity Framework detects changes in the `DbContext`.
2. It generates SQL commands for inserts, updates, and deletes.
3. It executes commands inside a transaction.
4. It resets change tracking after a successful commit.

[↑ Back to Top](#table-of-contents)

## 8. Shadow Properties

What are shadow properties in Entity Framework, and why are they useful?

### Concepts Tested

- Entity Framework model configuration
- Hidden fields
- Database schema flexibility

### Hint

- Shadow properties are properties that are not defined in the C# entity class but exist in the Entity Framework model.
- They are useful for storing audit data, such as `CreatedBy` and `UpdatedOn`.

[↑ Back to Top](#table-of-contents)

## 9. Concurrency Handling

How does Entity Framework Core handle concurrency, and how can you resolve concurrency conflicts?

### Concepts Tested

- Optimistic concurrency
- Row versioning
- Database locking

### Hint

- Use a row version (`byte[]`) or concurrency tokens marked with `IsConcurrencyToken()` to detect changes.
- Handle `DbUpdateConcurrencyException` in application code.

[↑ Back to Top](#table-of-contents)

## 10. Dataset Pagination

How can you efficiently paginate large datasets in Entity Framework?

### Concepts Tested

- Query optimization
- Performance
- LINQ best practices

### Hint

- Use `Skip()` and `Take()`.
- Consider keyset pagination instead of offset-based pagination.

[↑ Back to Top](#table-of-contents)
