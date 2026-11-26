# Understanding Interfaces

**Understanding Interfaces**

**Introduction**

For software to survive in the ever-changing jungle of the production environment, it must have three distinct characteristics: **reusability**, **maintainability**, and **extensibility**. This article will provide a general overview of interface-based programming and examine these characteristics.

Interfaces solve many problems associated with code reuse in object-oriented programming.

A class based reference makes it possible to instantiate and communicate with an object. **The communication between the client and the object takes place through a set of publicly accessible properties and methods that are known as an object's public interface.** The class author must use the public interface to expose the object's functionalities to the client. This is what makes the object useful. Not that the method of the class must continue to supply the same members in order to honour these dependencies built into the client.

One benefit of using classes is that they allow you to reuse the code. Once a class has been written, you can use it in many different places in an application. Classes let you reduce or eliminate redundant code in an application. They also facilitate code maintenance. You can modify or remove any properties or methods that are not publicly visible. You can also change public method implementations as long as the calling syntax of the methods is not altered. When the implementation of a method in a class is improved, any client that uses the class will seamlessly benefit from the changes.

When you modify an existing class definition, you should not change the calling syntax for accessing any public method or property because of the risk of breaking the dependencies that client code has built on the original class definition. **As long as you hold the public interface constant, you can make any modifications to improve your class without breaking any client code**.

To rephrase the key point of the last paragraph: ***Once you public a property or a method signature in a class's public interface, you cannot change or remove it. ***This means that you must properly design the public interface at the beginning of the project and use discipline as the class evolves. This lets you improve and extend object code without having to rewrite any client code. You can maintain a class by fixing bugs and improving method implementations.

The rules for maintaining existing members of the public interface are cut and dried, but what flexibility do you have when you add new functionality to a class? What can you do to safely extend an object in later versions? It's easy and safe to add new public methods and properties to a class. Old clients continue to run as before even though they cannot take advantage of the object's new functionality. However, new clients written after the class has been modified can take advantage of any members added to the public interface. This means you can improve an object safely over time in the production environment.

Problems arise in class design when you change the signature of a public method in a way that breaks an existing client. This commonly happens when you discover that the initial class design was inadequate.

The intuitive solution is to make sure the design of the class's public interface is full-featured and finalized before you write client code against it. But this is not always possible, even for the most experienced class designer. If a class models a real-world entity that never changes, an experienced designer can create a robust, long-lasting design. However, in many cases it's impossible to predict how external changes will affect the requirements for an object's public interface. A designer who is creating classes for an application that is expected to run for years in a rapidly changing business environment cannot possibly predict what is needed. If the business model is constantly changing, your classes must change with it. Therein lies the need for extensible objects.

Software designers have found that the use of class-based references results in a layer of dependencies between clients and classes. You can lessen the impact of these dependencies on maintainability and extensibility through disciplined design and by anticipating future requirements. Don't define a method or property in the public interface unless you are prepared to live with it forever. Most experienced designers make all data properties private and provide access to an object's state through public methods. This prevents any client dependencies to the actual data layout of the class. Be conscientious about which methods you mark as public. Any member you mark as private can be changed or removed as the class implementation evolves. Of course, you have to make some members public or the class will be useless. Designing with class-based references always involves these tradeoffs.

**What is Polymorphism ?**

**Implementation inheritance **offers the implicit reuse of method implementations, which results in greater maintainability through the elimination of duplicate code. There is another powerful OOP feature provided by implementation inheritance known as polymorphism. This is arguably the most important concept in object-oriented programming. **Polymorphism allows a client to treat different objects in the same way even if they were created from different classes and exhibit different behaviours.**

**What is an Interface in C# ?**

*Interface describe a group of related functionalities that can belong to any class or struct.*

An Interface is a named collection of semantically similar abstract members or functions. A class or a structure may follow a behaviour that is prescribed by an interface. The keyword interface is used to create an interface. Note that while defining members inside an interface, there is no need to specify access specifier, as they are public by default.

Syntax :

```csharp
public interface IBooks
{
```

int BookID;

```text
}
```

As a convention interfaces in .NET are prefixes with an "I" in uppercase, as a good programming practice.

Methods inside an interface do not have a body. They only signify that such a method should exist in the implementing class or structure.

Classes that display a common set of features, such as having similar types of member variables or member functions, are designed in such a way that they implement a common interface.

**What is the difference between abstract class and interface ?**

If a class is to **serve the purpose of providing common fields and members to all subclasses, we create an Abstract class**. For creating an abstract class, we make use of the abstract keyword. Such a class cannot be instantiated.

Further, we may define abstract methods within an abstract class (analogous to C++ pure virtual functions) when we wish to define a method that does not have any default implementation. It is then in the hands of the descendant class to provide the details of the method. There may be any number of abstract methods in an abstract class. We define an abstract method using the abstract keyword. If we do not use the abstract keyword, and use the virtual keyword instead, we may provide an implementation of the method that can be used by the child class, but this is not an abstract method.

Remember, abstract class can have an abstract method, that does not have any implementation, for which we use the abstract keyword, OR the abstract class may have a virtual method, that can have an implementation, and can be overridden in the child class as well, using the override keyword.

## Bullet Points

1. A class may inherit only one abstract class, but may implement multiple number of Interfaces. Say a class named Car needs to inherit some basic features of a vehicle, it may inherit from an abstract class named Vehicle. A car may be of any kind, it may be a vintage car, a sedan, a coupe, or a racing car. For these kinds of requirements, say a car needs to have only two seats (means it is a coupe), then the class Cars needs to implement a member from an interface that we make, say ICoupe.

2. Members of an abstract class may have any access modifier, but members of an interface are public by default, and can't have any other access modifier.

3. Abstract class methods may OR may not have an implementation, while methods in an interface only have definition, no implementation.
