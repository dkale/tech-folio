# Dependency Injection & Inversion of Control

## Table of Contents

- [Dependency Injection (DI)](#dependency-injection-di)
- [Inversion of Control (IoC)](#inversion-of-control-ioc)
- [Inversion of Control](#inversion-of-control)
- [Dependency Injection (DI)](#dependency-injection-di)
- [What is dependency?](#what-is-dependency)
- [What is dependency injection?](#what-is-dependency-injection)
- [Constructor Dependency](#constructor-dependency)
- [Dependency Injection Pros & Cons](#dependency-injection-pros-cons)
- [Pros](#pros)
- [Cons](#cons)
- [Summary](#summary)

The main goal of Inversion of Control and Dependency Injection is to remove dependencies of an application. This makes the system more decoupled and maintainable.

Dependency Injection (DI) and Inversion of Control (IoC) are related concepts often used together, but they refer to different aspects of software design.

## Dependency Injection (DI)

Definition: Dependency Injection is a design pattern in which the dependencies of a class or a module are injected from the outside rather than being created or instantiated within that class or module.

Purpose: The main purpose of Dependency Injection is to achieve loose coupling between components by allowing dependencies to be provided externally. This makes code more modular, testable, and maintainable.

Mechanism: Dependencies can be injected through constructor injection, method injection, or property injection.

[↑ Back to Top](#table-of-contents)

## Inversion of Control (IoC)

Definition: Inversion of Control is a broader design principle where the control of the flow of a program is inverted or moved from the application to a framework of a container.

Purpose: IoC aims to promote the separation of concerns and modularization by allowing a framework or container to manage the flow and lifecycle of components.

Mechanism: IoC can be implemented through various techniques, with Dependency Injection being one of the most common forms. Other mechanisms include service locators and factories.

In summary, Dependency Injection is a specific technique used to achieve Inversion of Control. IoC is a more general principle that involves delegating control and responsibilities to an external framework or container. Dependency Injection is often the means by which IoC is implemented, but IoC encompasses a broader set of concepts and techniques.

[↑ Back to Top](#table-of-contents)

## Inversion of Control

Sometimes referred to as the Dependency Inversion Principle (DIP). The principle states that high level or low level modules should not depend upon each other, instead they should depend upon abstractions.

Specific implementations (object instances) are deferred to a higher level of abstraction of control. Example: Parent class, a container.

Referred to as the “Hollywood Principle” - Don’t call us, we will call you.

Inversion of control means the program delegates control to someone else who will drive the flow. For instance if we make the chat application event based then the flow of the program will go something as below:

1. End user sends a chat message.

2. User continues with his work ahead.

3. Application listens to events. If a message arrives, the event is activated and the message is received and displayed.

If you see the program flow it's not sequential, its event based. So now the control is inverted. So rather than the internal program controlling the flow, events drive the program flow. Event flow approach is more flexible as there is no direct invocation which leads to more flexibility.

This does not mean IoC is implemented only by events. You can delegate the control flow by callback delegates, observer pattern, events, DI (Dependency Injection) and a lot other ways.

IoC (Inversion of Control) is a general parent term while DI(Dependency Injection) is a subset of IoC. IoC is a concept where the flow of application is inverted. So for example, rather than the caller calling the method.

SomeObject.Call();

Will get replaced with an event based approach as shown below:

SomeObject.WhenEvent += Call();

In the above code the caller is exposing an event and when that event occurs he is taking action. It's based on the Hollywood principle, "Don't call us, we will call you". In Hollywood when artists used to give auditions the judges would say them, "Don't call us, we will call you".

The above approach makes code more flexible as the caller is not aware of the object methods and the object is not aware of the caller program flow.

[↑ Back to Top](#table-of-contents)

## Dependency Injection (DI)

[↑ Back to Top](#table-of-contents)

## What is dependency?

When we use another object to perform a certain task, we are creating a dependency on that object. That object might use other objects, too, so you are implicitly depending on those other objects, and they too might depend on other objects to do their work, and so on.

What problems do dependencies create?

- Code is tightly coupled.

- Difficult to isolate when testing.

- Difficult to maintain.

[↑ Back to Top](#table-of-contents)

## What is dependency injection?

The ability to supply (inject) an external dependency into a software component. Dependency Injection provides objects that an object needs. So rather than the dependencies construct themselves they are injected by some external means.

[↑ Back to Top](#table-of-contents)

## Constructor Dependency

public class CustomerService : ICustomerService

{

```csharp
private ICustomerRepository repository;
```

```csharp
private ICustomerDTOMapper mapper;
```

public CustomerService(ICustomerRepository repository, ICustomerDTOMapper mapper)

{

this.repository = repository;

this.mapper = mapper;

}

}

Injecting a ICustomerRepository and a ICustomerDTOMapper through the constructor.

[↑ Back to Top](#table-of-contents)

## Dependency Injection Pros & Cons

[↑ Back to Top](#table-of-contents)

## Pros

- Loosely Coupled

- Increases Testability (a lot!)

- Separates components cleanly

- Allows for use of Inversion of Control Containers

[↑ Back to Top](#table-of-contents)

## Cons

- Increases code complexity

- Some Jr.Developers find it difficult to understand.

- Can complicate debugging at first.

[↑ Back to Top](#table-of-contents)

## Summary

Inversion of Control: It's a generic term and implemented in several ways(events, delegates, etc)

Dependency Injection: DI is a subtype of IoC and is implemented by constructor injection, setter injection or method injection.

[↑ Back to Top](#table-of-contents)
