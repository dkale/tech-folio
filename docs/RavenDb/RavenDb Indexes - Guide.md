# RavenDB Indexes — Complete Guide

## Table of Contents

1. [The Core Philosophy (Beginner)](#1-the-core-philosophy-beginner)
   - [The "Stale" Concept](#the-stale-concept)
2. [Auto Indexes vs. Static Indexes (Beginner)](#2-auto-indexes-vs-static-indexes-beginner)
3. [Understanding Map Indexes (Intermediate)](#3-understanding-map-indexes-intermediate)
4. [Map-Reduce Indexes (Intermediate)](#4-map-reduce-indexes-intermediate)
5. [Summary Checklist for Beginners](#summary-checklist-for-beginners)
6. [Multi-Map Indexes (Polymorphic Queries)](#6-multi-map-indexes-polymorphic-queries)
7. [Indexing Deeply Nested Data & Fanout Guardrails](#7-indexing-deeply-nested-data--fanout-guardrails)
8. [Storing Fields vs. Indexing Fields (Projections)](#8-storing-fields-vs-indexing-fields-projections)
9. [Advanced Search: Full-Text Analysis & Term Vectors](#9-advanced-search-full-text-analysis--term-vectors)
10. [Expert Architectural Summary](#expert-architectural-summary)
11. [JavaScript Indexes — Overview](#11-javascript-indexes--overview)
12. [Indexing Related Documents (`LoadDocument`)](#12-indexing-related-documents-loaddocument)
13. [Production Deployment & Administration](#13-production-deployment--administration)
14. [JavaScript Indexes — Deep Dive](#14-javascript-indexes--deep-dive)
    - [The Core Anatomy of a JS Index](#1-the-core-anatomy-of-a-js-index)
    - [Indexing Arrays and Nested Logic](#2-advanced-pattern-indexing-arrays-and-nested-logic)
    - [The JS Map-Reduce Pattern](#3-the-js-map-reduce-pattern)
    - [Cross-Document Referencing (`load`)](#4-cross-document-referencing-load)
    - [When to Prefer JS Indexes over LINQ](#when-should-you-prefer-js-indexes-over-linq)

---

## 1. The Core Philosophy (Beginner)

In traditional SQL databases, if you search for a user by email and don't have an index, the database scans the entire table line-by-line (a table scan), which slows down your app.

RavenDB takes a strict stance: **Table scans are completely forbidden.** To achieve massive speeds, RavenDB handles indexing asynchronously in the background.

1. **You save a document:** RavenDB writes it to the database instantly and tells your app "Done!"
2. **Background worker triggers:** In the background, RavenDB picks up that new/changed document, processes it through an index, and updates its internal search list.

### The "Stale" Concept

Because indexing happens in the background, there is a tiny window of time (usually milliseconds) where the index is catching up. If you query the database exactly at that millisecond, RavenDB will still answer you immediately, but it will flag the result as **Stale**. RavenDB firmly believes it is better to give you a fast, slightly stale answer than to lock up your application or freeze your server.

---

## 2. Auto Indexes vs. Static Indexes (Beginner)

You don't always have to write indexes yourself. RavenDB divides indexes into two main categories:

1. **Auto Indexes:** When you write a simple dynamic query in your code (e.g., "Find all Employees where FirstName is 'Robert'"), RavenDB looks to see if an index for that exists. If it doesn't, **RavenDB creates an Auto Index for you on the fly**.
2. **Static Indexes:** These are indexes you write explicitly in your code or via the RavenDB Management Studio. You need these when your queries get complex, or when you want to optimize how data is searched.

---

## 3. Understanding Map Indexes (Intermediate)

A **Map Index** is the most common index type. Its job is simple: take a document, extract specific fields you want to search by, and create a lookup list.

Imagine you have a collection of `Employee` documents that look like this:

```json
// Employees/1-A
{
    "FirstName": "Robert",
    "LastName": "King",
    "Address": {
        "Country": "UK"
    }
}
```

### Step 1: Define the Index

To search employees efficiently by their last name or country, you define a static index class using C# LINQ syntax:

```csharp
public class Employees_ByNameAndCountry : AbstractIndexCreationTask<Employee>
{
    // 1. Define what the output "index entry" looks like
    public class IndexEntry
    {
        public string LastName { get; set; }
        public string FullName { get; set; } // We can combine fields!
        public string Country { get; set; }
    }

    public Employees_ByNameAndCountry()
    {
        // 2. Write the Map function
        Map = employees => from employee in employees
              select new IndexEntry()
              {
                  LastName = employee.LastName,
                  FullName = employee.FirstName + " " + employee.LastName,
                  Country = employee.Address.Country
              };
    }
}
```

*Note how we created a computed field `FullName` by combining `FirstName` and `LastName`. The index computes this ahead of time so searching it is instant!*

### Step 2: Deploy the Index

Before querying, you register the index with your document store when your application starts:

```csharp
new Employees_ByNameAndCountry().Execute(store);
```

### Step 3: Query the Index

Now you can search the database using the index. You can do this using standard strongly-typed LINQ or Raven Query Language (RQL):

**Using LINQ syntax:**

```csharp
IList<Employee> employees = session
    .Query<Employees_ByNameAndCountry.IndexEntry, Employees_ByNameAndCountry>()
    .Where(x => x.LastName == "King" && x.Country == "UK")
    .OfType<Employee>() // Tell RavenDB to return the full Employee documents
    .ToList();
```

**Using Raven Query Language (RQL):**

```sql
from index 'Employees/ByNameAndCountry'
where LastName = 'King' and Country = 'UK'
```

> **Crucial Intermediate Concept:** A common mistake is thinking an index works like a SQL View (returning only the columns you specified). In RavenDB, when you query an index, RavenDB uses the index to find the matching IDs, but **it returns the full, original raw documents** to your application unless you explicitly ask for a "Projection" (subset of fields).

---

## 4. Map-Reduce Indexes (Intermediate)

What if you don't just want to *find* documents, but you want to *aggregate* data—like calculating the total sales per country or counting how many employees live in each city? This is where **Map-Reduce** indexes shine.

A Map-Reduce index works in two phases:

1. **Map:** Selects the data fields from the documents.
2. **Reduce:** Groups the mapped data together and performs calculations (like Sum or Count).

Because RavenDB does this in the background, **the math is computed ahead of time**. If you have 10 million orders, a Map-Reduce query doesn't compute the sum when you ask for it; it outputs the pre-calculated answer instantly.

### Map-Reduce Example: Count Employees per Country

```csharp
public class Employees_CountByCountry : AbstractIndexCreationTask<Employee, Employees_CountByCountry.Result>
{
    public class Result
    {
        public string Country { get; set; }
        public int Count { get; set; }
    }

    public Employees_CountByCountry()
    {
        // 1. Map: Break down documents into basic country blocks, starting with a count of 1
        Map = employees => from employee in employees
              select new Result
              {
                  Country = employee.Address.Country,
                  Count = 1
              };

        // 2. Reduce: Group by Country and sum up the counts
        Reduce = results => from result in results
                 group result by result.Country into g
                 select new Result
                 {
                     Country = g.Key,
                     Count = g.Sum(x => x.Count)
                 };
    }
}
```

**Querying the Map-Reduce results:**

```csharp
// Get the total number of employees in the UK instantly
Result ukStats = session.Query<Employees_CountByCountry.Result, Employees_CountByCountry>()
    .Where(x => x.Country == "UK")
    .FirstOrDefault();

Console.WriteLine($"UK has {ukStats.Count} employees.");
```

---

## Summary Checklist for Beginners

- **Background Magic:** RavenDB indexes process data in the background, keeping your application writes incredibly fast.
- **Staleness is normal:** Don't panic if an index is briefly "stale"; it just means the background worker is processing updates.
- **Auto-Pilot:** RavenDB generates Auto Indexes if you don't write them.
- **Use Static Map Indexes** when you need custom searchable fields or want to combine data fields (like `FullName`).
- **Use Map-Reduce Indexes** when you need instant aggregation, counting, or mathematical summaries across millions of documents.

---

## 6. Multi-Map Indexes (Polymorphic Queries)

In the real world, you often need to search across different document collections simultaneously. For instance, you might have a unified search bar in an enterprise app where a user types a name, and you need to search across both `Employees` and `Companies`.

A **Multi-Map Index** allows you to define multiple `Map` functions within a single index, routing data from different collections into a standardized index entry layout.

### The Example Scenario

We want to create an autocomplete global search that scans both `Employees` (searching by `FirstName` + `LastName`) and `Companies` (searching by `Name`).

```csharp
public class GlobalSearch : AbstractIndexCreationTask<GlobalSearch.IndexEntry>
{
    public class IndexEntry
    {
        // This is our standardized field that queries will hit
        public string SearchTerm { get; set; }
        public string Type { get; set; }
    }

    public GlobalSearch()
    {
        // Map 1: Process Employees
        AddMap<Employee>(employees => from employee in employees
             select new IndexEntry
             {
                 SearchTerm = employee.FirstName + " " + employee.LastName,
                 Type = "Employee"
             });

        // Map 2: Process Companies
        AddMap<Company>(companies => from company in companies
             select new IndexEntry
             {
                 SearchTerm = company.Name,
                 Type = "Company"
             });
    }
}
```

### Querying the Multi-Map Index

When you query this, RavenDB merges the results. Because you are querying multiple collections, you use `project into` or dynamic objects to handle the mixed return types:

```csharp
var results = session.Query<GlobalSearch.IndexEntry, GlobalSearch>()
    .Where(x => x.SearchTerm.StartsWith("Ro"))
    .ToList();
// Returns a mix of index entries pointing to both Employees and Companies
```

---

## 7. Indexing Deeply Nested Data & Fanout Guardrails

Documents in document databases are often deeply nested (arrays within arrays). Indexing these requires understanding how RavenDB maps arrays to Lucene fields, and how to avoid the catastrophic **Fanout Index** trap.

### Indexing Collections

Suppose an `Order` document contains an array of `Lines`:

```json
// Orders/1-A
{
    "CustomerId": "customers/5",
    "Lines": [
        { "ProductName": "Laptop", "Quantity": 1 },
        { "ProductName": "Mouse", "Quantity": 2 }
    ]
}
```

If you want to search for any order that contains a specific product, you index it by targeting the collection directly in LINQ:

```csharp
public class Orders_ByProduct : AbstractIndexCreationTask<Order>
{
    public Orders_ByProduct()
    {
        Map = orders => from order in orders
              select new
              {
                  // LINQ automatically projects this array into a flat index field
                  ProductNames = order.Lines.Select(x => x.ProductName)
              };
    }
}
```

### The Fanout Danger (Expert Warning)

By default, RavenDB outputs **one index entry per document**.

However, if you write a map function using `from... from...` (a SelectMany operation), you create a **Fanout Index**:

```csharp
// DANGER: Avoid doing this unless strictly necessary
Map = orders => from order in orders
                from line in order.Lines // Creates an entry for EVERY line item
                select new
                {
                    ProductName = line.ProductName
                };
```

If an order contains 500 line items, RavenDB creates 500 index entries for that single document. If you update that document, RavenDB has to clean up and re-write all 500 entries. This leads to massive index sizes, high CPU utilization, and system resource exhaustion.

> **Expert Rule:** Keep your map outputs flat. If you need to index arrays, map them as a collection projection (`order.Lines.Select(x => x.ProductName)`) rather than splitting them into separate entries.

---

## 8. Storing Fields vs. Indexing Fields (Projections)

By default, an index acts as an address book: it stores a list of terms and the document IDs they belong to. When you run a query, RavenDB has to fetch the document from the main storage engine to show it to you.

If you have a massive high-traffic dashboard, loading full documents just to display a simple table is an unnecessary I/O cost. You can bypass document loading entirely using **Stored Fields**.

### Step 1: Configure Storage in the Index

```csharp
public class Employees_QuickSearch : AbstractIndexCreationTask<Employee>
{
    public Employees_QuickSearch()
    {
        Map = employees => from employee in employees
              select new
              {
                  employee.LastName,
                  employee.FirstName
              };

        // Instruct RavenDB to keep these values directly inside the Lucene index structure
        Store(x => x.LastName, FieldStorage.Yes);
        Store(x => x.FirstName, FieldStorage.Yes);
    }
}
```

### Step 2: Querying via Projections (`ProjectInto`)

When you query, you use `Select` or `ProjectInto`. RavenDB notices that the requested fields are already saved directly in the index, so it extracts them instantly **without ever opening the raw document storage**.

```csharp
var dashboardLines = session.Query<Employee, Employees_QuickSearch>()
    .Where(x => x.LastName == "King")
    .Select(x => new
    {
        x.FirstName,
        x.LastName
    }) // This query executes at memory-cache speeds!
    .ToList();
```

---

## 9. Advanced Search: Full-Text Analysis & Term Vectors

At an expert level, you will need to implement elastic-style search features like fuzzy matching, stemming, or search suggestions. RavenDB handles this by passing text fields through **Analyzers** (like standard Lucene tokenizers).

```csharp
public class Articles_Search : AbstractIndexCreationTask<Article>
{
    public Articles_Search()
    {
        Map = articles => from article in articles
              select new
              {
                  article.Title,
                  article.Content
              };

        // Turn on Full-Text Search for the Content field
        Analyze(x => x.Content, "Lucene.Net.Analysis.Standard.StandardAnalyzer");

        // Enable Term Vectors if you intend to use MoreLikeThis (recommendation engines)
        TermVector(x => x.Content, FieldTermVector.WithPositionsAndOffsets);
    }
}
```

With `Analyze` turned on, a search for "running" can automatically match documents containing "run" or "runs", depending on the analyzer chosen, and you can perform complex queries using search syntax operators like `Search()`:

```csharp
IList<Article> results = session.Query<Article, Articles_Search>()
    .Search(x => x.Content, "RavenDB database AND optimization")
    .ToList();
```

---

## Expert Architectural Summary

| Feature | Intermediate Use Case | Expert Optimization |
|---|---|---|
| **Data Scope** | Single collection mapping. | **Multi-Map** to index disjointed structures into unified query models. |
| **Nested Data** | Relying on nested structures blindly. | Strict avoidance of **Fanout Indexes** by mapping arrays to collections rather than splitting rows. |
| **I/O Efficiency** | Fetching full documents back to the client. | Using **Stored Fields** and Projections to run queries purely in-memory from index components. |
| **Search Intent** | Exact string matching (`==`). | Leveraging Lucene **Analyzers** and **Term Vectors** for intelligent lexical search. |

---

## 11. JavaScript Indexes — Overview

While C# LINQ syntax is popular, RavenDB actually compiles LINQ indexes into **JavaScript** under the hood. You can write your indexes directly in JavaScript. This is incredibly useful if you are working in a Node.js environment, or if your index needs to perform dynamic manipulations that are difficult to express in LINQ.

### Example: A JavaScript Map Index

```javascript
// Map definition written as a JS function
map("Employees", function (employee) {
    return {
        LastName: employee.LastName,
        FullName: employee.FirstName + " " + employee.LastName,
        Country: employee.Address.Country
    };
});
```

---

## 12. Indexing Related Documents (`LoadDocument`)

In a document database, you try to keep documents self-contained. However, sometimes data is normalized. For example, an `Order` document contains a `CustomerId`, but you want to search for Orders by the **Customer's Name** (which lives in a completely separate `Customers` document).

RavenDB allows an index to dynamically pull data from another document using `LoadDocument`.

```csharp
public class Orders_ByCustomerName : AbstractIndexCreationTask<Order>
{
    public Orders_ByCustomerName()
    {
        Map = orders => from order in orders
              // Load the related Customer document on the fly during indexing!
              let customer = LoadDocument<Customer>(order.CustomerId)
              select new
              {
                  CustomerName = customer.Name,
                  OrderDate = order.CreatedAt
              };
    }
}
```

> **Expert Note:** RavenDB tracks this dependency. If the Customer's name changes in their own document, RavenDB automatically marks the related `Order` index entries as stale and re-indexes them.

---

## 13. Production Deployment & Administration

When updating an index in a production environment with millions of documents, re-indexing can take time and consume a lot of CPU resources. To handle this like an expert, you should learn about:

- **Rolling Index Deployments:** In a clustered environment (multiple server nodes), RavenDB can deploy an index change to one node at a time. This ensures your other nodes remain completely active and unaffected while the index rebuilds.
- **Side-by-Side Indexing:** When you update an index definition, RavenDB builds a new hidden version of it in the background. The old version keeps serving your app's queries until the new index is 100% caught up. Once ready, RavenDB swaps them seamlessly with zero downtime.

---

## 14. JavaScript Indexes — Deep Dive

In RavenDB, **JavaScript Indexes** are treated as first-class citizens. While C# developers love LINQ, JavaScript indexes are incredibly powerful because they allow you to utilize standard JS patterns—like loops, conditional logic, and string manipulations—directly inside the indexing pipeline.

### 1. The Core Anatomy of a JS Index

A JavaScript index consists of one or more string-based scripts written using an ECMAScript-like syntax. Instead of inheriting from `AbstractIndexCreationTask<T>`, you inherit from `AbstractJavaScriptIndexCreationTask`.

Here is the baseline pattern for a simple **Map Index**:

```csharp
public class Employees_ByDetails_JS : AbstractJavaScriptIndexCreationTask
{
    public Employees_ByDetails_JS()
    {
        // Define the target collection and the mapping function
        Maps = new HashSet<string>
        {
            @"map('Employees', function (employee) {
                return {
                    // Fields we want to index
                    LastName: employee.LastName,
                    FullName: employee.FirstName + ' ' + employee.LastName,
                    Country: employee.Address.Country
                };
            })"
        };
    }
}
```

### 2. Advanced Pattern: Indexing Arrays and Nested Logic

Where JavaScript indexes truly shine is handling nested arrays and dynamic objects without needing complex LINQ `SelectMany` or null-coalescing expressions.

Suppose you have a `Product` document with an array of `Tags`. You want to clean up the tags (lowercase them) and handle missing data gracefully.

```csharp
public class Products_Search_JS : AbstractJavaScriptIndexCreationTask
{
    public Products_Search_JS()
    {
        Maps = new HashSet<string>
        {
            @"map('Products', function (product) {
                // JavaScript arrays map beautifully to Lucene multi-valued fields
                var processedTags = [];

                // Safe checking using standard JS logic
                if (product.Tags && Array.isArray(product.Tags)) {
                    for (var i = 0; i < product.Tags.length; i++) {
                        processedTags.push(product.Tags[i].toLowerCase().trim());
                    }
                }

                return {
                    Name: product.Name,
                    Tags: processedTags,
                    HasDiscounts: product.Price > 100 ? 'Yes' : 'No' // Ternary logic
                };
            })"
        };
    }
}
```

### 3. The JS Map-Reduce Pattern

Writing Map-Reduce in JavaScript requires two distinct components: the `map` function and the `groupby` / `aggregate` functions. RavenDB exposes a special global `groupBy` method for this.

Let's rebuild the "Count Employees per Country" example using JS:

```csharp
public class Employees_CountByCountry_JS : AbstractJavaScriptIndexCreationTask
{
    public Employees_CountByCountry_JS()
    {
        // 1. Map Phase
        Maps = new HashSet<string>
        {
            @"map('Employees', function (employee) {
                return {
                    Country: employee.Address.Country,
                    Count: 1
                };
            })"
        };

        // 2. Reduce Phase
        Reduce = @"groupBy(x => x.Country)
            .aggregate(g => ({
                Country: g.key,
                Count: g.values.reduce((sum, x) => sum + x.Count, 0) // Standard JS array reduce
            }))";
    }
}
```

### 4. Cross-Document Referencing (`load`)

Just like `LoadDocument` in LINQ, JavaScript indexes have a built-in global `load` function. This allows your index to hop to another collection to fetch reference data.

```csharp
public class Orders_WithCustomerDetails_JS : AbstractJavaScriptIndexCreationTask
{
    public Orders_WithCustomerDetails_JS()
    {
        Maps = new HashSet<string>
        {
            @"map('Orders', function (order) {
                // Fetch the customer document dynamically
                var customer = load(order.CustomerId, 'Customers');

                return {
                    OrderDate: order.CreatedAt,
                    // Fallback using standard JS short-circuiting if customer document is missing
                    CustomerName: customer ? customer.Name : 'Anonymous',
                    CustomerRating: customer ? customer.Rating : 0
                };
            })"
        };
    }
}
```

### When Should You Prefer JS Indexes over LINQ?

- **Polyglot Teams:** If your backend stack uses Node.js or Python alongside .NET, writing indexes in JS ensures that *everyone* can read and modify the index definitions in the RavenDB Studio.
- **Complex Data Scrubbing:** If you need to perform heavy text transformations, string splits, regex parsing, or deep conditional branches, standard JavaScript is often significantly cleaner to read than multi-line LINQ statements.
- **Dynamic Document Fields:** If your documents store arbitrary JSON keys (like a dynamic `Metadata` dictionary) that you need to loop through and index programmatically, JS loops make it trivial.
