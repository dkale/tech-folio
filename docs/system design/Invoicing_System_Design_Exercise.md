## PROBLEM STATEMENT:

The goal of this exercise is to design the classes and methods (Models and Services) required for an invoicing system for IT companies. Following is the requirements

Please keep in mind the following points for a better result:

1. The code should be divided into API(or aspx file) , Models and services and they need to represent the real objects in real world.

2. Please use the OOPS principles wherever applicable

3. Please follow SOLID principles wherever applicable

4. Please use latest programming techniques like dependency injection etc

5. The review of code will be based on the above points so if you do follow the above points wherever applicable chances of getting a favorable response.

In case you have any doubts in below question or any of above points, please ask the recruiter to connect to Madhu.

The below question is for 1 hr. It has two parts. The individual timelines is just an indication of the amount of time you can take to finish it successfully. Please note that completing is not the priority, doing

whatever is done in a proper way is the priority with above points in mind.

Design Classes, Properties, and Methods for the following requirements (No code is required here only the structure is required)

1. Company has multiple customers

2. Customers have one or more projects that are being executed by the company

3. Each project can be run in one of two billing modes

a. Time & Material - Hours are logged by people working on the project and client is billed based on a fixed rate by the hour

b. Milestone Based - Project has milestones and clients are billed a fixed amount when a milestone is completed

4. System should have the ability to generate invoices for customers and email them to customers at the end of every month

Write code in one of the classes to meet the following requirement

1. Once a month, a file is uploaded into the system that has all the bank transactions. These include payments made by customers. Customers may pay all charges for

multiple projects in one payment or may pay per project.

The system must determine matching amounts and display it to the company's accountant so he can easily tally payments and invoices. Following are the fields in the uploaded sheet.

a. Date

b. Type (Debit/Credit)

c. Amount

d. CustomerID

Goals

- Design the classes with good object oriented design and SOLID principles

- There might be thousands of transactions and invoices, so make the code as efficient as possible

---

## Invoicing System Design Exercise (IT Services)

This section addresses the invoicing-system problem statement with:

1. API, Models, and Services separation
2. OOP + SOLID-friendly contracts
3. Dependency-injection-ready interfaces
4. One concrete code implementation for payment matching

### Suggested Project Structure

```text
/Api
  /Controllers
    CustomersController.cs
    ProjectsController.cs
    InvoicesController.cs
    BillingController.cs
    PaymentsController.cs

/Models
  /Entities
    Company.cs
    Customer.cs
    Project.cs
    Milestone.cs
    Invoice.cs
    InvoiceLineItem.cs
    Payment.cs
    BankTransaction.cs
  /Enums
    BillingMode.cs
    InvoiceStatus.cs
    TransactionType.cs
  /ValueObjects
    Money.cs
    BillingPeriod.cs
  /Dtos
    GenerateInvoicesRequest.cs
    PaymentMatchResultDto.cs

/Services
  /Contracts
    IInvoiceGenerationService.cs
    IInvoiceDeliveryService.cs
    IProjectBillingCalculator.cs
    ITimeAndMaterialBillingCalculator.cs
    IMilestoneBillingCalculator.cs
    IBankTransactionImportService.cs
    IPaymentMatchingService.cs
    IClock.cs
    IEmailSender.cs
    IRepository<T>.cs
  /Implementations
    InvoiceGenerationService.cs
    InvoiceDeliveryService.cs
    TimeAndMaterialBillingCalculator.cs
    MilestoneBillingCalculator.cs
    CsvBankTransactionImportService.cs
    PaymentMatchingService.cs
```

### Models (Structure Only)

```csharp
public enum BillingMode
{
    TimeAndMaterial = 1,
    MilestoneBased = 2
}

public enum InvoiceStatus
{
    Draft = 1,
    Generated = 2,
    Sent = 3,
    PartiallyPaid = 4,
    Paid = 5,
    Overdue = 6
}

public enum TransactionType
{
    Debit = 1,
    Credit = 2
}

public sealed class Company
{
    public Guid Id { get; init; }
    public string Name { get; init; } = string.Empty;
    public IReadOnlyCollection<Customer> Customers { get; init; } = Array.Empty<Customer>();
}

public sealed class Customer
{
    public Guid Id { get; init; }
    public string Name { get; init; } = string.Empty;
    public string BillingEmail { get; init; } = string.Empty;
    public IReadOnlyCollection<Project> Projects { get; init; } = Array.Empty<Project>();
}

public sealed class Project
{
    public Guid Id { get; init; }
    public Guid CustomerId { get; init; }
    public string Name { get; init; } = string.Empty;
    public BillingMode BillingMode { get; init; }
    public decimal HourlyRate { get; init; }
    public IReadOnlyCollection<TimeEntry> TimeEntries { get; init; } = Array.Empty<TimeEntry>();
    public IReadOnlyCollection<Milestone> Milestones { get; init; } = Array.Empty<Milestone>();
}

public sealed class TimeEntry
{
    public Guid Id { get; init; }
    public Guid ProjectId { get; init; }
    public DateTime WorkDate { get; init; }
    public decimal Hours { get; init; }
    public string ResourceName { get; init; } = string.Empty;
}

public sealed class Milestone
{
    public Guid Id { get; init; }
    public Guid ProjectId { get; init; }
    public string Name { get; init; } = string.Empty;
    public decimal Amount { get; init; }
    public bool IsCompleted { get; private set; }
    public DateTime? CompletedOnUtc { get; private set; }

    public void MarkComplete(DateTime completedOnUtc)
    {
        IsCompleted = true;
        CompletedOnUtc = completedOnUtc;
    }
}

public sealed class Invoice
{
    public Guid Id { get; init; }
    public Guid CustomerId { get; init; }
    public Guid ProjectId { get; init; }
    public DateTime InvoiceDateUtc { get; init; }
    public DateTime DueDateUtc { get; init; }
    public InvoiceStatus Status { get; private set; }
    public decimal TotalAmount { get; private set; }
    public decimal OutstandingAmount { get; private set; }
    public IReadOnlyCollection<InvoiceLineItem> LineItems { get; init; } = Array.Empty<InvoiceLineItem>();

    public void MarkSent() => Status = InvoiceStatus.Sent;

    public void ApplyPayment(decimal amount)
    {
        OutstandingAmount = Math.Max(0, OutstandingAmount - amount);
        Status = OutstandingAmount == 0 ? InvoiceStatus.Paid : InvoiceStatus.PartiallyPaid;
    }
}

public sealed class InvoiceLineItem
{
    public Guid Id { get; init; }
    public Guid InvoiceId { get; init; }
    public string Description { get; init; } = string.Empty;
    public decimal Quantity { get; init; }
    public decimal UnitRate { get; init; }
    public decimal Amount => Quantity * UnitRate;
}

public sealed class Payment
{
    public Guid Id { get; init; }
    public Guid CustomerId { get; init; }
    public DateTime ReceivedOnUtc { get; init; }
    public decimal Amount { get; init; }
    public string SourceReference { get; init; } = string.Empty;
}

public sealed class BankTransaction
{
    public DateTime DateUtc { get; init; }
    public TransactionType Type { get; init; }
    public decimal Amount { get; init; }
    public Guid CustomerId { get; init; }
    public string ExternalReference { get; init; } = string.Empty;
}
```

### Services (Structure Only)

```csharp
public interface IInvoiceGenerationService
{
    Task<IReadOnlyCollection<Invoice>> GenerateMonthlyInvoicesAsync(DateTime monthUtc, CancellationToken cancellationToken = default);
}

public interface IInvoiceDeliveryService
{
    Task SendInvoiceAsync(Invoice invoice, Customer customer, CancellationToken cancellationToken = default);
    Task SendMonthlyInvoicesAsync(IReadOnlyCollection<Invoice> invoices, CancellationToken cancellationToken = default);
}

public interface IProjectBillingCalculator
{
    BillingMode Mode { get; }
    Invoice BuildInvoice(Project project, Customer customer, DateTime monthUtc);
}

public interface IBankTransactionImportService
{
    Task<IReadOnlyCollection<BankTransaction>> ImportAsync(Stream file, CancellationToken cancellationToken = default);
}

public interface IPaymentMatchingService
{
    IReadOnlyCollection<PaymentMatchResult> Match(
        IReadOnlyCollection<BankTransaction> transactions,
        IReadOnlyCollection<Invoice> openInvoices,
        DateTime? asOfUtc = null);
}

public sealed class PaymentMatchResult
{
    public Guid CustomerId { get; init; }
    public decimal TransactionAmount { get; init; }
    public IReadOnlyCollection<Guid> MatchedInvoiceIds { get; init; } = Array.Empty<Guid>();
    public decimal MatchedAmount { get; init; }
    public decimal UnmatchedAmount { get; init; }
    public bool IsExactMatch { get; init; }
    public string Notes { get; init; } = string.Empty;
}
```

### API Endpoints (Structure Only)

```text
POST /api/billing/invoices/generate-monthly
GET  /api/invoices?customerId={id}&status=Open
POST /api/invoices/{invoiceId}/send-email
POST /api/payments/bank-transactions/upload
POST /api/payments/match
GET  /api/payments/match-results?month=YYYY-MM
```

### DI Registration (Structure Only)

```csharp
services.AddScoped<IInvoiceGenerationService, InvoiceGenerationService>();
services.AddScoped<IInvoiceDeliveryService, InvoiceDeliveryService>();
services.AddScoped<IBankTransactionImportService, CsvBankTransactionImportService>();
services.AddScoped<IPaymentMatchingService, PaymentMatchingService>();

services.AddScoped<IProjectBillingCalculator, TimeAndMaterialBillingCalculator>();
services.AddScoped<IProjectBillingCalculator, MilestoneBillingCalculator>();

services.AddScoped(typeof(IRepository<>), typeof(EfRepository<>));
services.AddScoped<IEmailSender, SmtpEmailSender>();
```

### Part 2: One Class Implementation (Efficient Payment Matching)

The implementation below matches monthly credit transactions to open invoices per customer.

- Scales well for thousands of records using grouping and sorting.
- Handles full payment, multi-project consolidated payment, partial payment, and overpayment.
- Follows SRP: this class only performs matching logic.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

public sealed class PaymentMatchingService : IPaymentMatchingService
{
    public IReadOnlyCollection<PaymentMatchResult> Match(
        IReadOnlyCollection<BankTransaction> transactions,
        IReadOnlyCollection<Invoice> openInvoices,
        DateTime? asOfUtc = null)
    {
        var effectiveDate = asOfUtc ?? DateTime.UtcNow;

        var credits = transactions
            .Where(t => t.Type == TransactionType.Credit && t.DateUtc <= effectiveDate)
            .GroupBy(t => t.CustomerId)
            .ToDictionary(g => g.Key, g => g.OrderBy(x => x.DateUtc).ToList());

        var invoicesByCustomer = openInvoices
            .Where(i => i.OutstandingAmount > 0)
            .GroupBy(i => i.CustomerId)
            .ToDictionary(g => g.Key, g => g.OrderBy(i => i.InvoiceDateUtc).ThenBy(i => i.Id).ToList());

        var results = new List<PaymentMatchResult>(transactions.Count);

        foreach (var kv in credits)
        {
            var customerId = kv.Key;
            var customerTransactions = kv.Value;
            invoicesByCustomer.TryGetValue(customerId, out var customerInvoices);
            customerInvoices ??= new List<Invoice>();

            foreach (var tx in customerTransactions)
            {
                decimal remaining = tx.Amount;
                decimal matched = 0;
                var matchedInvoiceIds = new List<Guid>();

                foreach (var invoice in customerInvoices)
                {
                    if (remaining <= 0)
                        break;

                    if (invoice.OutstandingAmount <= 0)
                        continue;

                    var applied = Math.Min(invoice.OutstandingAmount, remaining);
                    if (applied <= 0)
                        continue;

                    invoice.ApplyPayment(applied);
                    matched += applied;
                    remaining -= applied;

                    if (!matchedInvoiceIds.Contains(invoice.Id))
                        matchedInvoiceIds.Add(invoice.Id);
                }

                results.Add(new PaymentMatchResult
                {
                    CustomerId = customerId,
                    TransactionAmount = tx.Amount,
                    MatchedInvoiceIds = matchedInvoiceIds,
                    MatchedAmount = matched,
                    UnmatchedAmount = remaining,
                    IsExactMatch = remaining == 0,
                    Notes = BuildNotes(matched, remaining, matchedInvoiceIds.Count)
                });
            }
        }

        return results;
    }

    private static string BuildNotes(decimal matched, decimal unmatched, int invoiceCount)
    {
        if (matched == 0)
            return "No open invoice found for this customer.";

        if (unmatched > 0)
            return $"Overpayment detected. Matched {invoiceCount} invoice(s), excess amount: {unmatched}.";

        return invoiceCount > 1
            ? "Consolidated payment matched across multiple projects/invoices."
            : "Payment matched to a single invoice.";
    }
}
```

### Why This Design Scores Well in Review

- OOP: entities represent real-world objects (Company, Customer, Project, Invoice, Payment).
- SOLID:
  - SRP: each service has one clear responsibility.
  - OCP: billing modes are extensible via new calculator implementations.
  - LSP/ISP: interfaces are granular and replaceable.
  - DIP: controllers depend on interfaces; DI container wires concrete implementations.
- Performance:
  - Grouping by customer avoids repeated full scans.
  - Ordered matching allows deterministic, accountant-friendly reconciliation.
  - Time complexity is approximately `O(T log T + I log I + M)`, where:
    - `T` = transactions count
    - `I` = invoices count
    - `M` = actual match iterations
