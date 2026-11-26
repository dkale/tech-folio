# OOPS - Interview Questions

## Table of Contents
- [OOP Fundamentals](#oop-fundamentals)
- [Types, Arrays & Collections](#types-arrays-collections)
- [Classes, Inheritance & C# Language Features](#classes-inheritance-c-language-features)
- [Interfaces & Abstract Classes](#interfaces-abstract-classes)
- [Abstraction, Encapsulation & Object Relationships](#abstraction-encapsulation-object-relationships)
- [Polymorphism](#polymorphism)



## OOP Fundamentals



**OOPS Interview Questions**

### What is OOP?

OOP is a technique to develop logical modules, such as classes that contain properties, methods, fields, and events. An object is created in the program to represent a class. Therefore, an object encapsulates all the features, such as data and behaviour that are associated to a class. OOP allows developers to develop modular programs and assemble them as software. Objects are used to access data and behaviours of different software modules, such as classes, namespaces, and shareable assemblies.

### What is a class?

A class describes all the **attributes of objects**, as well as the **methods that implement the behaviour** of member objects. It is a comprehensive data type, which represents a **blueprint of objects**. It is a **template of an object**.

A class can be defined as the primary building block of OOP. It also serves as a template that **describes the properties, state, and behaviours** common to a particular group of objects.

A class contains data and behaviours of an entity. For example, the aircraft class can contain data, such as model number, category, and colour and behaviour, such as duration of flight, speed, and number of passengers. A class inherits the data members and behaviours of other classes by extending from them.

### What is an object?

They are instances of classes. It is a basic unit of a system. An object is an entity that has attributes, behaviour, and identity. Attributes and behaviour of an object are defined by the class definition.

### Explain the basic features of OOPS.

The following are the basic features of OOPS:

- **Abstraction** – Refers to the process of exposing only the relevant and essential data to the users without showing unnecessary information.

- **Polymorphism -** Allows you to use an entity in multiple forms.

- **Encapsulation** – Prevents the data from unwanted access by binding code and data in a single unit called object.

- **Inheritance **– Promotes the reusability of code and eliminates the use of redundant code. It is the property through which a child class obtains all the features defined in its parent class. When a class inherits the common properties of another class, the class inheriting the properties is called a derived class and the class that allows inheritance of its common properties is called a base class.

### What is the difference between Procedural and Object-Oriented programming?

Procedural programming is based upon the modular approach in which the larger programs are broken into procedures. Each procedure is a set of instructions that are executed one after another.

On the other hand, OOP is based upon objects. An object consists of various elements, such as methods and variables.

Access modifiers are not used in procedural programming, which implies that the entire data can be accessed freely anywhere in the program. In OOP, you can specify the scope of a particular data by using access modifiers – public, private, internal, protected, and protected internal.



[↑ Back to Top](#table-of-contents)



## Types, Arrays & Collections



### What is the difference between arrays and collection?

**Array**:

- You need to specify the size of an array at the time of its declaration. It cannot be resized dynamically.

- The members of an array should be of the same data type.

**Collection:**

- The size of a collection can be adjusted dynamically, as per the user’s requirement. It does not have a fixed size.

- Collections can have elements of different types.

### What are collections and generics?

A collection can be defined as a group of related items that can be referred to as a single unit. The System.Collections namespace provides you with many classes and interfaces. Some of them are – ArrayList, List, Stack, ICollection, IEnumerable, and IDictionary. Generics provide the type-safety to your class at the compile time. While creating a data structure, you never need to specify the data type at the time of declaration. The System.Collections.Generics namespace contains all the generics collections.

### What is the difference between a Class and a Structure?

Class:

- A class is a reference type.

- While instantiating a class, CLR allocates memory for its instance in heap.

- Classes support inheritance.

- Variables of a class can be assigned as NULL.

- Class can contain constructor/destructor.

Structure:

- A structure is a value type.

- In structure, memory is allocated on the stack.

- Structures do not support inheritance.

- Structure members cannot have null values.

- Structure does not require a constructor/destructor and members can be initialised automatically.

### What are the similarities between Class and a Structure?

Structures and classes are the two most important data structures that are used by programmers to build modular programs by using OOP languages. The following are some of the similarities between a class and a structure:

- Access specifiers, such as public, private, and protected, are identically used in structures and classes to restrict the access of their data and methods outside their body.

- The access level for class members and struct members, including nested classes and structs, is private by default. Private nested types are not accessible from outside the containing type.

- Both can have constructors, methods, properties, fields, constants, enumerations, events, and event handlers.

- Both structures and classes can implement interfaces to use multiple-inheritance in code.

- Both structures and classes can have constructors with parameters.

- Both structures and classes can have delegates and events.

### Define an array.

An array is defined as a homogeneous collection of elements, stored at contiguous memory locations, which can be referred to by the same variable name. All the elements of an array variable can be accessed by index values. An index value specifies the position of a particular element in an array variable.

### What are properties in C# and their advantages?

In C#, a property is a way to expose an internal data element of a class in a simple and intuitive manner. In other words, it is a simple extension of data fields. You can create a property by defining an externally available name and the writing the set and get property accessors. The get property accessor is used to return the property value. The set property accessor is used to assign a new value to the property.

### What is a Hashtable?

Hashtable is a data structure that **implements the IDictionary interface**. It is used to store multiple items and each of these items is associated with a** unique string key**. Each item can be accessed using the key associated with it. In short, a hashtable is an object holding the key-value pairs.



[↑ Back to Top](#table-of-contents)



## Classes, Inheritance & C# Language Features



### Is it possible for a class to inherit the constructor of its base class?

A derived class can inherit the constructor of its base class using the **base(<parameters…>) **syntax in the constructor. This syntax explicitly calls the constructor of the base class, passing the necessary parameter.

When you create an instance of DerivedClass, it automatically invokes the constructor of the base class first and then executes the constructor code of the derived class.

### Can you declare an overridden method to be static if the original method is not static?

No, two virtual methods must have the same signature.

### Why is the virtual keyword used in code?

The virtual keyword is used while defining a class to specify that the methods and the properties of that class can be overridden in derived classes.

### Explain the different types of inheritance.

Single Inheritance

Hierarchical Inheritance

Multiple Inheritance

Multi-level Inheritance

### You have defined a destructor in a class that you have developed by using the C# programming language, but the destructor never executed. Why did the destructor not execute?

The runtime environment automatically invokes the destructor of a class to release the resources that are occupied by variables and methods of an object. However, in C#, programmers cannot control the timing for invoking destructors, as Garbage Collector is only responsible for releasing the resources used by an object. Garbage Collector automatically get information about unreferenced objects from .NETs runtime environment and then invokes the Finalize() method.

Although it is not preferable to force Garbage Collector to perform garbage collection and retrieve all inaccessible memory, programmers can use the Collect() method of the Garbage Collector class to forcefully execute Garbage Collector.

### What are static constructor? When is a static constructor called?

Static constructors are introduced with C# to initialise the static data of a class. CLR (automatically)calls the static constructor before the first instance is created and before any static members of the class are accessed or any static methods are called. It’s called once and only one, ensuring that the initialization logic in the static constructor is executed before the class is used.

The static constructor has the following features:

- No access specifier is required to define it.

- You cannot pass parameters in a static constructor.

- A class can have only one static constructor.

- It can access only static members of the class.

- It is invoked only once, when the programming execution begins.

Initialization Timing: A static constructor is called automatically before any static members (fields, properties, methods) of the class are accessed or before any static methods are called.

Implicit Invocation: You do not explicitly call a static constructor; it is invoked automatically by the runtime.

Single Execution: The static constructor is guaranteed to be called only once, regardless of how many instances of the class are created or how many times the class is accessed.

No parameters: Static constructors cannot take any parameters, and they don’t have an access modifier(public, private, etc.) or an explicit return type.

Initialization of Static Members: Static constructors are often used to initialise static members of the class, perform one-time initialization tasks, or set up resources that are shared among all instances of the class.

Initialization Order: If a class is derived from another class, the base class’s static constructor is called before the derived class’s static constructor.

### What is Shadowing ?

Shadowing is a concept of polymorphism usage in Object Oriented Programming. This is a concept related to overriding functions at run-time or making a shadow of the object's methods in the inherited classes. Though it is not much used in programming, it may be used sometimes and its usage is very restrictive and unique.



[↑ Back to Top](#table-of-contents)



## Interfaces & Abstract Classes



### State the features of an Interface.

An interface is a template that contains only the signature of methods. The signature of a method consists of the numbers of parameters, the type of parameter (value, reference, or output), and the order of parameters. An interface has no implementation on its own because it cannot instantiate an interface. The various features of an interface are as follows:

- An interface is used to implement multiple inheritances in code. This feature of an interface is quite different from that of abstract classes because a class cannot derive the features of more than one class but can easily implement multiple interfaces.

- It defines a specific set of methods and their arguments.

- Variables in interface must be declared as public, static, and final while methods must be public and abstract.

- A class implementing an interface must implement all of its methods.

- An interface can derive from more than one interface.

### What are abstract classes? What are the distinct characteristics of an abstract class?

- An abstract class is a class that cannot be instantiated and is always used as a base class. The following are the characteristics of an abstract class:

- You cannot instantiate an abstract class directly. This implies that you cannot create an object of the abstract class; it must be inherited.

- You can have abstract as well as non-abstract members in an abstract class.

- You must declare at least one abstract member in an abstract class.

- An abstract class is always public.

- An abstract class is declared using the abstract keyword.

- The basic purpose of an abstract class is to provide a common definition of the base class that multiple derived classes can share.

### Differentiate between an abstract class and an interface.

| **Abstract Class** | **Interface** |
| --- | --- |
| A class can extend only one abstract class. | A class can implement several interfaces. |
| The members of an abstract class can be private as well as protected. | An interface can only have public members. |
| Abstract classes should have subclasses. | Only an interface can extend another interface. |
| Methods in abstract class can be abstract as well as concrete. | All methods in an interface should be abstract. |
| The class extending the abstract class must implement all its abstract methods. | All methods of interface need to be implemented by a class implementing that interface. |
| An abstract class can implement methods. | Interfaces cannot contain the body of any of its methods. |



[↑ Back to Top](#table-of-contents)



## Abstraction, Encapsulation & Object Relationships



### What do you mean by data encapsulation?

Data encapsulation is a concept of binding data and code in a single unit called object and hiding all the implementation details of a class from the user. It prevents unauthorised access of data and restricts the user to use the necessary data only.

### What is Abstraction ?

- Abstraction refers to the act of representing essential features without including the background details or explanations.

- Abstraction defines a way to abstract or hide your data and members from the outside world.

- This can be achieved by means of access modifiers.

- Abstraction means we bother about what the object can do but don’t bother about how it does.

- Data abstraction in OOP is the concept of abstracting the implementation to the user. It gives the functional description but not the implementation part. Take an abstract part of an interface for example, it gives the user some method descriptions, but not the internal implementation.

### Understanding Association, Aggregation, Composition and Dependency Relationship

http://www.c-sharpcorner.com/UploadFile/shivprasadk/3-important-concepts-association-aggregation-and-composi/

A relationship defines the connection between objects. This explains how objects are connected to each other’s and how they will behave.

**Association**

It represents a relationship between two or more objects where all objects have their own lifecycle and there is no owner. The name of association specifies the nature of relationship between objects. This is represented by a solid line.

Let’s take an example of the relationship between Teacher and Student. Multiple students can associate with a single teacher and a single student can associate with multiple teachers. But there is no ownership between the objects and both have their own lifecycle. Both can be created and deleted independently.

![Original embedded image](data:image/png;base64,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)

**Aggregation**

It is a specialized form of Association where all objects have their own lifecycle but there is ownership. This represents a “whole-part or a-part-of” relationship. This is represented by a hollow diamond followed by a line.

![Original embedded image](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAMYAAAAaCAYAAADsZyMJAAACjElEQVR4Ae2azSsEYRzHf2u9C9vaECd5Ky25UBS5KEk5Somb8ge5cVBKysWBiwMHIVJataVISrHrJXaFdlsvz9Rss2MennmM8bDf5zIzv3meZ37PZ+bbfOeZx/P6XggFBEAgg0BOxhEOQAAENAIQBh6EP08gmXLe9EAYf/6xyN4BMEEs7Udpav6IFvei9Jx8cQyGB98YjrFERy4S2D+L0/zOJV3Fk+mr+kvyaLSzirrqy9Ix2R0IQ5Yc2v0KgUgsQXNblxQ6f9CuXxcoooGgn9bCt3QcfdJiwdoSGu+qphpfgXSOEIY0OjR0kwCzTcsHV7QSuiG2X5zvpZGOSupr9pE3x6OlsnF0926pIhR/TmmxwdYKGm4PUGGe/S8GCMPNu4trSREw2yYmhpGOKiot9H7o7zGR0r43mEhSL68ka68gjA9oEVCFgJVtmuiupobKoi9TPL1+0iyXrL2CML5EjApuExCxTaI5ydorCEOUMOq5QmD3NEYLu5H0bNNntkk0IRl7JSSMsZmwaA6oBwKOEZjsraHeJp9j/W2fxGh6/TzdX3+Ln5g1syq5VkHEQEAFArObF3Rxn5CeWdLHYLRmLGac0dLrmLdCbwxzIxyDwE8SkLE+vHzszGgZ+4AwjDSwrxSB78wsfWdGi0GAMJR6FJCMFQE7M0tG28T7EWh1DXMMwjATwbGSBETslaxtshowhGFFBTFlCVjZq6G2AK0e3mSsnxL9EcgbKITBI4O40gSM9kpP1DjbpK+f0s/Z3UIYdomhvjIEjPaqp7Gcu35KJmEIQ4Ya2ihFgAmEvS2cLBCGkzTR178hYH+h+r8ZOgYCAnwCEAafDc5kMQEII4tvPobOJwBh8NngTBYTeAOFyWIxfG2gegAAAABJRU5ErkJggg==)

Let’s take an example of relationship between the Department and Teacher. A Teacher may belong to multiple departments. Hence Teacher is a part of multiple departments. But if we delete a Department, the Teacher Object will not be destroyed.

![Original embedded image](data:image/png;base64,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)

**Composition**

It is a specialized form of Aggregation. It is a strong type of Aggregation. In this relationship child objects do not have their lifecycle without Parent object. If a parent object is deleted, all its child objects will also be deleted. This represents the “death” relationship. This is represented by a solid diamond followed by a line.

![Original embedded image](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAMQAAAAeCAYAAABzA7EiAAACbklEQVR4Ae2bzUsCQRjG382+pRKTjDpFRRAFXQoMgi5dunSUIOrc/9QliiDoEvQHdCoqhOoQBIEEQWoFYVLiR+UspJvN7M7a5k7uM6f1nXHmfX+7D/vsrGofpUZoIAACOoEmcAABEKgQgCAqLHAEAgRB4CIAAQMBCMIAA4cgAEHgGvjXBPJFZ/eEIIh/fTl4N3kmhL1Yita3r2n3LEXZ/LsjMDRsuzrCEZPUkUDs9oW2ju/pMVMorxr0t9DyTJgiw93lWC0HEEQt1PAdVwgk0znaPErQ5V1GuP7EoJ9WI/00EGgTjjHrgCDM6KBPCQLMHu2fP9DBxRMV3s2eGVifRr4mjRYne2lpKkTtLfaeCiAIJU45khAR4Nkj0djqeC02CoKopojPShCQsUeyidqxURCELFWMqwsBeXskm449GwVByHLFuD8ncBpP085J4tvukdOLWtkoU0GsbFw5nQ/mAwElCCyMB2lttv9HLs0/IgiAgIsEutp99JItOp6BVpqRmafOVh9Fp/tofizAXcP0DsH9BoIg8IcEXnNF/c3z4fUzFU23WOWS0EpK+PrHDxNBdDpMTHSiBkGIyCDuKoH445v+Eu4m9fbrPIZCHbo9GunrsJwLgrBEhAFuEmB3it2zpC0bxbNH7GWdTIMgZChhjKsEZG2UXXvEKwqC4FFBTEkCMjbKjj3iFQlB8KggpjQBno0y7h7J2iNekRAEjwpiyhMw2qi50R7L3SPZgiAIWVIYpyQBJgx2d3CqQRBOkcQ8DUHA3o/FG6JkFAECYgIQhJgNejxIAILw4ElHyWICEISYDXo8SACC8OBJR8liAp+ACfOXmf9hJAAAAABJRU5ErkJggg==)

Let’s take an example of the relationship between House and Rooms. House can contain multiple rooms, there is no independent life of room and any room cannot belong to two different houses. If we delete the house, the room will automatically delete.

![Original embedded image](data:image/png;base64,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)

Let’s take another example of the relationship between Questions and options. Single questions can have multiple options and options cannot belong to multiple questions. If we delete questions, options will be automatically deleted.

**Dependency**

It represents a relationship between two or more objects where an object is dependent on another object(s) for its specification or implementation. This is represented by a dashed arrow.

![Original embedded image](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAALQAAAASCAYAAADyiPTBAAABN0lEQVRoBWP8DwQMo2A0BIZJCDANE3+MemM0BMAhMJqgRxPCsAqB0QQ9rKJz1DOjCXo0DQyrEBhN0MMqOkc9M5qgR9PAoAyBDeffMBy69YHh7z/SBuFGE/SgjM5RR736/Ith1qFnDGVr7pKUsEcT9GjaGdQh8PITaQmbEdvESsyca3g9uSRFC6v8qL7RcMGWMMhNL9jMEudjY/A3EGGwVuFnYGZixFAyWkJjBAl1Be6/+U5dA0e4aZ9//GV48+U3w88//7CGBNYSGqvKUcHREKBjCIDaz6BOIQxwsTEzeOgIgTGIjQuw4JIYFR8NgcEQAsQmZJhbRxM0LCRG6UEVAlxsTAxBRqIES2R0R482OdBDZJQ/pENgtFM4pKNv1PHoITCaoNFDZJQ/pENgNEEP6egbdTx6CAAAqiVpLu/WikoAAAAASUVORK5CYII=)

Let’s take an example of the relationship between client and service. A client is dependent on the service for implementing its functionalities.

![Original embedded image](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAALQAAAASCAYAAADyiPTBAAABN0lEQVRoBWP8DwQMo2A0BIZJCDANE3+MemM0BMAhMJqgRxPCsAqB0QQ9rKJz1DOjCXo0DQyrEBhN0MMqOkc9M5qgR9PAoAyBDeffMBy69YHh7z/SBuFGE/SgjM5RR736/Ith1qFnDGVr7pKUsEcT9GjaGdQh8PITaQmbEdvESsyca3g9uSRFC6v8qL7RcMGWMMhNL9jMEudjY/A3EGGwVuFnYGZixFAyWkJjBAl1Be6/+U5dA0e4aZ9//GV48+U3w88//7CGBNYSGqvKUcHREKBjCIDaz6BOIQxwsTEzeOgIgTGIjQuw4JIYFR8NgcEQAsQmZJhbRxM0LCRG6UEVAlxsTAxBRqIES2R0R482OdBDZJQ/pENgtFM4pKNv1PHoITCaoNFDZJQ/pENgNEEP6egbdTx6CAAAqiVpLu/WikoAAAAASUVORK5CYII=)

Let’s take another example of the relationship between a client and a supplier. A client is dependent on the supplier for supplying products. If the supplier will not supply the products, clients cannot use those products.



[↑ Back to Top](#table-of-contents)



## Polymorphism



### What are the different types of polymorphism ?

Compile Time / Static / Early Binding : Operator overloading ~ Function Overloading

Runtime / Dynamic / Late Binding : Virtual, Interfaces, Abstract classes, Base Class



[↑ Back to Top](#table-of-contents)
