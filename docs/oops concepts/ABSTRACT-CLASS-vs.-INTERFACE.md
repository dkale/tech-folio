# ABSTRACT CLASS vs. INTERFACE

## Table of Contents
- [Background](#background)
- [What is Abstract Class ?](#what-is-abstract-class)
- [What is an Interface ?](#what-is-an-interface)

**Abstract Class:**

- Abstract class provides a set of rules to be implemented by the implementation class.

- Rules will be provided through abstract methods.

- Abstract method does not contain any definition.

- While inheriting abstract class all abstract methods must be overridden.

- If a class contains at least one abstract method then it must be declared as an 'Abstract Class'.

- Abstract classes cannot be instantiated (i.e. we cannot create objects), but a reference can be created.

Reference depends on the child class object's memory.

**Interface:**

- **·****         ****Interface supports Multiple Inheritance.**

- In interface all methods are public abstract by default.

- Interfaces cannot be instantiated, but a reference can be created.

**Abstract Classes vs. Interfaces**

©  If a class is to serve the purpose of providing common fields and members to all subclasses, we create an Abstract class.

©  An Interface is a collection of semantically related abstract members. An interface expresses through the members it defines, the behaviours that a class needs to support.

The choice of whether to design your functionality as an Interface of an Abstract class can sometimes be a difficult one. An **abstract class **is a class that cannot be instantiated, but must be inherited from. An abstract class may be fully implemented, but is more usually partially implemented or not implemented at all, thereby encapsulating common functionality for inherited classes.

An **interface**, by contrast, is a totally abstract set of members that can be thought of as defining a contract for conduct. The implementation of an interface is left completely to the developer.

Both **interfaces and abstract classes are useful for component interaction. ****If a method requires an interface as an argument, then any object that implements that interface can be used in the argument**. For example:

```csharp
public void Spin (IWidget widget) {}
```

This **method could accept any object** that implemented IWidget as the widget argument, even though the implementations of IWidget might be quite different.

Abstract classes also allow for this kind of polymorphism, but with a few caveats:

* Classes may inherit from only one base class, so if you want to use abstract classes to provide polymorphism to a group of classes, they must all inherit from that class.

* Abstract classes may also provide members that have already been implemented. Therefore, **you can ensure a certain amount of identical functionality with an abstract class, but cannot with the interface.**

**Here are some important recommendations to help you to decide whether to use an interface or an abstract class to provide polymorphism for your components:**

* If you anticipate creating multiple versions of your components, create an abstract class. **Abstract classes provide a simple and easy way to version your components. By updating the base class, all inheriting classes are automatically updated with the change.** Interfaces, on the other hand, cannot be changed once created. If a new version of an interface is required, you must create a whole new interface.

* If the functionality you are creating will be useful across a wide **range of disparate objects, use an interface**.

*** Abstract classes** should be used primarily for **objects that are closely related**, whereas **Interfaces **are best suited for providing **common functionality to unrelated classes**.

* If you are designing small, **concise bits of functionality, use interfaces**. If you are designing **large functional units, use an abstract class**.

* If you want to provide **common, implemented functionality **among all implementations of your components, **use an abstract class**. Abstract classes allow you to partially implement your class, whereas interfaces contain no implementation for any members.

----------------------------------------------------------------------------------------------------------------------------

**Abstract Class vs. Interface**

http://www.codeproject.com/Articles/11155/Abstract-Class-versus-Interface

## Background

An Abstract class without any implementation just looks like an Interface; however there are a lot of differences than similarities between an Abstract class and an Interface. Let’s explain both concepts and compare their similarities and differences.

## What is Abstract Class ?

An abstract class is a special kind of class that cannot be instantiated. So the question is why we need a class that cannot be instantiated? An abstract class is only to be sub-classed (inherited from). In other words, it only allows other classes to inherit from it but cannot be instantiated. The advantage is that it enforces certain hierarchies for all the subclasses. In simple words, it is a kind of contract that forces all the subclasses to carry on the same hierarchies or standards.

## What is an Interface ?

An interface is not a class. It is an entity that is defined by the word Interface. An interface has no implementation; it only has the signature or in other words, just the definition of the methods without the body. As one of the similarities to Abstract class, it is a contract that is used to define hierarchies for all subclasses or it defines a specific set of methods and their arguments. The main difference between them is that a class can implement more than one interface but can only inherit from one abstract class. Since C# doesn’t support multiple inheritance, interfaces are used to implement multiple inheritance.

Both Together

When we create an interface, we are basically creating a set of methods without any implementation that must be overridden by the implemented classes. The advantage is that it provides a way for a class to be a part of two classes: one from inheritance hierarchy and one from the interface.

When we create an abstract class, we are creating a base class that might have one or more completed methods but at least one or more methods are left uncompleted and declared abstract. If all the methods of an abstract class are uncompleted then it is the same as an interface. The purpose of an abstract class is to provide a base class definition for how a set of derived classes will work and then allow the programmers to fill the implementation in the derived classes.

| **Feature** | **Interface** | **Abstract class** |
| --- | --- | --- |
| Multiple inheritance | A class may inherit several interfaces. | A class may inherit only one abstract class. |
| Default implementation | An interface cannot provide any code, just the signature. | An abstract class can provide complete, default code and/or just the details that have to be overridden. |
| Access Modifiers | An interface cannot have access modifiers for the subs, functions, properties etc everything is assumed as public | An abstract class can contain access modifiers for the subs, functions, properties |
| Core VS Peripheral | Interfaces are used to define the peripheral abilities of a class. In other words both Human and Vehicle can inherit from an IMovable interface. | An abstract class defines the core identity of a class and there it is used for objects of the same type. |
| Homogeneity | If various implementations only share method signatures then it is better to use Interfaces. | If various implementations are of the same kind and use common behaviour or status then abstract class is better to use. |
| Speed | Requires more time to find the actual method in the corresponding classes. | Fast |
| Adding functionality (Versioning) | If we add a new method to an Interface then we have to track down all the implementations of the interface and define implementation for the new method. | If we add a new method to an abstract class then we have the option of providing default implementation and therefore all the existing code might work properly. |
| Fields and Constants | No fields can be defined in interfaces | An abstract class can have fields and constraints defined |

[↑ Back to Top](#table-of-contents)
