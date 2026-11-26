# SOLID Architecture Principles

Source: [CodeProject article](http://www.codeproject.com/Articles/703634/SOLID-architecture-principles-using-simple-Csharp)

## Table of Contents

1. [What SOLID Means](#what-solid-means)
2. [S - Single Responsibility Principle (SRP)](#s---single-responsibility-principle-srp)
3. [O - Open/Closed Principle (OCP)](#o---openclosed-principle-ocp)
4. [L - Liskov Substitution Principle (LSP)](#l---liskov-substitution-principle-lsp)
5. [I - Interface Segregation Principle (ISP)](#i---interface-segregation-principle-isp)
6. [D - Dependency Inversion Principle (DIP)](#d---dependency-inversion-principle-dip)
7. [Consolidated SOLID Recap](#consolidated-solid-recap)
8. [Further Learning](#further-learning)

## What SOLID Means

SOLID is a set of five foundational object-oriented design principles used to build maintainable, testable, and evolvable software.

- `S` stands for SRP (Single Responsibility Principle)
- `O` stands for OCP (Open/Closed Principle)
- `L` stands for LSP (Liskov Substitution Principle)
- `I` stands for ISP (Interface Segregation Principle)
- `D` stands for DIP (Dependency Inversion Principle)

At a practical level, SOLID helps teams avoid rigid classes, fragile dependencies, and ripple-effect changes.

## S - Single Responsibility Principle (SRP)

**Definition:** A class should have one reason to change.

### Problem Setup

Consider this `Customer` class:

```csharp
class Customer
{
    public void Add()
    {
        try
        {
            // Database code goes here
        }
        catch (Exception ex)
        {
            System.IO.File.WriteAllText(@"c:Error.txt", ex.ToString());
        }
    }
}
```

At first glance, this may look fine, but it mixes two responsibilities:

- customer-related behavior
- logging behavior

If logging requirements change (file logger, event viewer logger, email logger), this class must change even though customer behavior did not.

### First Refactor Step

Move logging to a dedicated type:

```csharp
class FileLogger
{
    public void Handle(string error)
    {
        System.IO.File.WriteAllText(@"c:Error.txt", error);
    }
}
```

Use delegation inside `Customer`:

```csharp
class Customer
{
    private FileLogger obj = new FileLogger();

    public virtual void Add()
    {
        try
        {
            // Database code goes here
        }
        catch (Exception ex)
        {
            obj.Handle(ex.ToString());
        }
    }
}
```

### Technical Note: Why This Is Still Not Perfect

This revision improves SRP, but there are still concerns:

- `Customer` still owns `try/catch` (cross-cutting concern)
- `Customer` has a strong concrete dependency on `FileLogger`
- exception formatting is still coupled to caller behavior

A stronger version is to depend on `ILogger` and inject logger implementations. An even cleaner architecture may centralize exception handling in middleware/global handlers so domain classes focus strictly on business behavior.

## O - Open/Closed Principle (OCP)

**Definition:** Software entities should be open for extension, but closed for modification.

### Problem Setup

```csharp
class Customer
{
    private int _CustType;

    public int CustType
    {
        get { return _CustType; }
        set { _CustType = value; }
    }

    public double getDiscount(double TotalSales)
    {
        if (_CustType == 1)
        {
            return TotalSales - 100;
        }
        else
        {
            return TotalSales - 50;
        }
    }
}
```

The hidden issue appears when a new customer type arrives:

- add one more `if`/`else`
- modify existing logic
- re-test prior scenarios
- risk regressions for existing clients

This creates change-amplification: each small feature increases the blast radius.

### Refactor Toward Extension

```csharp
class Customer
{
    public virtual double getDiscount(double TotalSales)
    {
        return TotalSales;
    }
}

class SilverCustomer : Customer
{
    public override double getDiscount(double TotalSales)
    {
        return base.getDiscount(TotalSales) - 50;
    }
}

class GoldCustomer : SilverCustomer
{
    public override double getDiscount(double TotalSales)
    {
        return base.getDiscount(TotalSales) - 100;
    }
}
```

Now new discount behavior is added by new subclasses instead of repeatedly editing stable code.

### Why This Matters

- old, working code remains untouched
- new behavior is isolated and test-focused
- extensions are additive, not disruptive

## L - Liskov Substitution Principle (LSP)

**Definition:** Objects of a subtype must be replaceable for objects of its base type without breaking expected behavior.

### Problem Setup

Suppose `Enquiry` inherits from `Customer`, but enquiries are leads and should not be persisted.

```csharp
class Enquiry : Customer
{
    public override double getDiscount(double TotalSales)
    {
        return base.getDiscount(TotalSales) - 5;
    }

    public override void Add()
    {
        throw new Exception("Not allowed");
    }
}
```

Inheritance view:

```mermaid
classDiagram
    class Customer
    class SilverCustomer
    class GoldCustomer
    class Enquiry

    Customer <|-- SilverCustomer
    Customer <|-- GoldCustomer
    Customer <|-- Enquiry
```


Polymorphic usage:

```csharp
List<Customer> Customers = new List<Customer>();
Customers.Add(new SilverCustomer());
Customers.Add(new GoldCustomer());
Customers.Add(new Enquiry());

foreach (Customer o in Customers)
{
    o.Add();
}
```

Failure during runtime:

![][image2]

### Root Cause

`Enquiry` looks like a `Customer` through inheritance but violates the base contract (`Add()` is not safe).

This means substitution is invalid.

### Refactor With Segregated Contracts

Define separate contracts for separate behaviors:

```csharp
interface IDiscount
{
    double getDiscount(double TotalSales);
}

interface IDatabase
{
    void Add();
}
```

`Enquiry` only implements what it can honor:

```csharp
class Enquiry : IDiscount
{
    public double getDiscount(double TotalSales)
    {
        return TotalSales - 5;
    }
}
```

`Customer` implements both behaviors:

```csharp
class Customer : IDiscount, IDatabase
{
    private MyException obj = new MyException();

    public virtual void Add()
    {
        try
        {
            // Database code goes here
        }
        catch (Exception ex)
        {
            obj.Handle(ex.Message.ToString());
        }
    }

    public virtual double getDiscount(double TotalSales)
    {
        return TotalSales;
    }
}
```

Compiler enforces correct usage boundaries:

![][image3]

## I - Interface Segregation Principle (ISP)

**Definition:** Clients should not be forced to depend on interfaces they do not use.

Assume the existing `Customer` component is widely consumed:

![][image4]

### Problem Setup

A new client asks for `Read()` support. If we add `Read()` directly to an existing broad interface, all old consumers inherit new obligations.

```csharp
interface IDatabase
{
    void Add();
    void Read();
}
```

This forces changes on clients that only require `Add()`, creating unnecessary coupling and migration cost.

### Refactor by Interface Versioning/Segregation

```csharp
interface IDatabase
{
    void Add();
}

interface IDatabaseV1 : IDatabase
{
    void Read();
}
```

Implement the extended contract only where needed:

```csharp
class CustomerWithRead : IDatabaseV1
{
    public void Add()
    {
        Customer obj = new Customer();
        obj.Add();
    }

    public void Read()
    {
        // Implements logic for read
    }
}

IDatabase i = new Customer();
i.Add();

IDatabaseV1 iv1 = new CustomerWithRead();
iv1.Read();
```

### Practical Outcome

- old clients remain stable
- new clients get new behavior
- change is additive, not breaking

## D - Dependency Inversion Principle (DIP)

**Definition:**

- High-level modules should not depend on low-level modules. Both should depend on abstractions.
- Abstractions should not depend on details. Details should depend on abstractions.

### Problem Setup

When `Customer` decides which concrete exception/logger type to construct, it owns object-creation policy and business logic together.

```csharp
class Customer : IDiscount, IDatabase
{
    private IException obj;

    public virtual void Add(int Exhandle)
    {
        try
        {
            // Database code goes here
        }
        catch (Exception ex)
        {
            if (Exhandle == 1)
            {
                obj = new MyException();
            }
            else
            {
                obj = new EmailException();
            }

            obj.Handle(ex.Message.ToString());
        }
    }
}
```

The key smell is internal `new` selection logic controlled by flags. This introduces tight coupling and violates inversion.

### Refactor to Abstraction + Injection

Define abstraction:

```csharp
interface ILogger
{
    void Handle(string error);
}
```

Concrete implementations:

```csharp
class FileLogger : ILogger
{
    public void Handle(string error)
    {
        System.IO.File.WriteAllText(@"c:Error.txt", error);
    }
}

class EventViewerLogger : ILogger
{
    public void Handle(string error)
    {
        // log errors to event viewer
    }
}

class EmailLogger : ILogger
{
    public void Handle(string error)
    {
        // send errors in email
    }
}
```

Inject dependency from the outside:

```csharp
class Customer : IDiscount, IDatabase
{
    private ILogger obj;

    public Customer(ILogger i)
    {
        obj = i;
    }
}

IDatabase i = new Customer(new EmailLogger());
```

### Why This Is Better

- `Customer` focuses on customer concerns
- wiring/configuration moves to composition root/client
- adding logger variants does not require modifying `Customer`

## Consolidated SOLID Recap

- **S / SRP:** A class should have one reason to change.
- **O / OCP:** Favor extension over repeated modification.
- **L / LSP:** Subtypes must honor base behavioral expectations.
- **I / ISP:** Keep interfaces client-specific and minimal.
- **D / DIP:** Depend on abstractions and inject details.

## Further Learning

- [GOF Design Patterns](http://www.codeproject.com/Articles/28309/Design-pattern-FAQ-Part-1-Training)
- [IOC demo video](https://www.facebook.com/photo.php?v=690253231015623&set=vb.341019362605680&type=2&theater)
- [OO design course](http://www.pluralsight.com/courses/table-of-contents/principles-oo-design)

[image2]: <data:image/png;base64,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>

[image3]: <data:image/png;base64,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>

[image4]: <data:image/png;base64,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>






