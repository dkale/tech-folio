# Behavioral Design Pattern - Mediator Pattern

## Table of Contents

- [Overview](#overview)
- [Using MediatR](#using-mediatr)
- [Mediator Pattern vs. Pub-Sub Pattern](#mediator-pattern-vs-pub-sub-pattern)

## Overview

Explained in simple words:

MediatR is like a middleman in a conversation. Imagine you have a group of friends, and you want to ask a question to the group, but you're not sure which friend has the answer. Instead of asking each friend individually, you ask the group leader (the mediator) to pass your question to the right friend who knows the answer.

### Main concepts

**Requests:** These are like questions or tasks you want to perform in your application.

**Handlers:** These are like friends who know how to answer or handle specific questions or tasks.

**Mediator:** This is like the group leader who receives your questions and passes them to the right friend (handler) to handle them.

So, instead of directly calling specific functions to perform tasks, you send requests to the mediator, and the mediator figures out which handler should handle the request. This helps keep your code organized, makes it easier to add new features, and reduces the complexity of your application.

[↑ Back to Top](#table-of-contents)

## Using MediatR

MediatR is an open-source library for .NET that implements the mediator pattern. The mediator pattern is a behavioral design pattern that helps to reduce coupling between components of a system by allowing them to communicate through a mediator object, rather than directly with each other.

### MediatR components

**Requests:** Actions or commands that are sent to the mediator.

**Handlers:** Components responsible for processing requests and producing responses.

**Mediator:** The central object that receives requests from clients and delegates them to appropriate handlers.

### Request flow

1. You define request and response classes to represent the actions you want to perform in your application.

2. You create handler classes that contain the logic for handling these requests.

3. When a request needs to be processed, you send it to the mediator, which then locates the appropriate handler and delegates the request to it.

4. The handler processes the request and returns a response.

5. The mediator passes the response back to the client.

MediatR simplifies the implementation of the mediator pattern in .NET applications and promotes better separation of concerns by decoupling the code responsible for handling requests from the code that sends them. It is commonly used in CQRS (Command Query Responsibility Segregation) and other architectural patterns to improve the maintainability and scalability of applications.

### Example

```csharp
// Define a request class
public class GetCustomerQuery : IRequest<Customer>
{
    public int CustomerId { get; set; }
}

// Define a handler for the request
public class GetCustomerQueryHandler : IRequestHandler<GetCustomerQuery, Customer>
{
    public async Task<Customer> Handle(GetCustomerQuery request, CancellationToken cancellationToken)
    {
        // Logic to fetch customer data from database or another source
        return await dbContext.Customers.FindAsync(request.CustomerId);
    }
}

// Somewhere in your application, you send the request to the mediator
var customer = await mediator.Send(new GetCustomerQuery { CustomerId = 123 });
```

In this example, GetCustomerQuery is a request class representing a query to retrieve customer data, GetCustomerQueryHandler is a handler responsible for processing this query, and mediator.Send() sends the request to the mediator for handling.

[↑ Back to Top](#table-of-contents)

## Mediator Pattern vs. Pub-Sub Pattern

The mediator pattern and the pub-sub (publish-subscribe) pattern are both used to facilitate communication between components in a software system, but they have different characteristics and use cases:

### Mediator Pattern

- In the mediator pattern, components (or objects) communicate with each other through a mediator object.

- The mediator encapsulates the communication logic between components, reducing direct dependencies between them.

- Components do not need to be aware of each other; they only communicate through the mediator.

- The mediator can be thought of as a central hub that facilitates communication and coordination among components.

### Pub-Sub Pattern

- In the pub-sub pattern, publishers and subscribers are loosely coupled.

- Publishers publish messages (events) to a topic or channel without needing to know who or how many subscribers there are.

- Subscribers express interest in specific topics or channels and receive messages published to those topics.

- There is no central mediator in the pub-sub pattern; instead, communication happens through the pub-sub infrastructure.

### Key Differences

**Communication style:**

- In the mediator pattern, communication is typically one-to-one or one-to-many through a mediator object.

- In the pub-sub pattern, communication is typically one-to-many or many-to-many, where publishers broadcast messages to multiple subscribers.

**Direct vs. indirect communication:**

- Mediator pattern involves direct communication between components through a mediator object.

- Pub-sub pattern involves indirect communication, where publishers and subscribers communicate through a pub-sub infrastructure without direct knowledge of each other.

**Decoupling:**

- Mediator pattern promotes loose coupling between components by abstracting communication through a mediator.

- Pub-sub pattern also promotes loose coupling, but it's often used in scenarios where components are even more loosely coupled, as publishers and subscribers do not need to know about each other.

### Use Cases

**Mediator Pattern:**

- Useful for coordinating complex interactions between multiple components in a system, such as in GUI frameworks or enterprise systems.

- Helps manage dependencies between components and reduces complexity by centralizing communication logic.

**Pub-Sub Pattern:**

- Commonly used for event-driven architectures, message brokers, and distributed systems.

- Facilitates decoupled communication between various parts of a system, enabling scalability and flexibility.

In summary, while both patterns facilitate communication between components in a system, the mediator pattern focuses on direct communication through a mediator object, while the pub-sub pattern enables indirect communication through a message-passing infrastructure. Each pattern has its own strengths and use cases, and the choice between them depends on the specific requirements and architecture of the system.

[↑ Back to Top](#table-of-contents)
