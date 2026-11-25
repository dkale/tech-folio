# RavenDB Indexing

> RavenDB uses indexes to transform documents and efficiently answer queries.

## Overview

RavenDB indexing maps document fields into a searchable index. Static indexes provide stable query paths, while dynamic indexes may be created from ad hoc queries.

## Example

```csharp
public sealed class Products_ByCategory :
    AbstractIndexCreationTask<Product>
{
    public Products_ByCategory()
    {
        Map = products => from product in products
                          select new
                          {
                              product.Category,
                              product.Name
                          };
    }
}
```

!!! note
    RavenDB Indexing should be designed around query requirements and reviewed when document shapes evolve.
