# Dependency Injection

> Dependency Injection separates object creation from object behavior and improves testability.

## Overview

In C# and ASP.NET Core, Dependency Injection supplies dependencies through constructors or service providers instead of requiring application code to construct them directly.

## Example

```csharp
public interface IProductService
{
    Task<Product?> GetByIdAsync(int id);
}

public sealed class ProductService : IProductService
{
    public Task<Product?> GetByIdAsync(int id)
    {
        // Implemented by the data access layer.
        return Task.FromResult<Product?>(null);
    }
}

builder.Services.AddScoped<IProductService, ProductService>();
```

## Lifetimes

| Lifetime | Behavior |
| --- | --- |
| Singleton | One instance for the application lifetime |
| Scoped | One instance per request scope |
| Transient | One instance per resolution |

## Best Practices

- Use constructor injection for required dependencies.
- Keep registrations explicit.
- Avoid injecting the service container itself.
- Avoid captive dependencies where a short-lived service depends on a longer-lived service.
