# Encapsulation

**Encapsulation**

The **wrapping** up of **data and functions **into a single unit (called **class**) is known as encapsulation. Containing and hiding information about an object, such as internal data structures and code.

It is a language mechanism for **restricting access to some of the object's components**.

It is a language construct that facilitates the **bundling of data with the methods **(or other functions) operating on that data.

It is an **information hiding mechanism**.  Encapsulation means that the internal representation of an object is generally hidden from view of the object's definition. Typically, only the object's own methods can directly inspect or manipulate its fields. Benefit of encapsulation is that it can **reduce system complexity **and thus **increase robustness**, by allowing the developer to limit the interdependencies between software components.

*Encapsulation is one of the four fundamentals of OOP. Encapsulation is to hide the variables or something inside a class, **preventing unauthorised parties **from using it. So the public methods like getter and setter access it and the other classes call these methods for accessing.

Implementation of abstract data types offer a similar form of encapsulation.

Encapsulation **serves to separate the contractual interface of an abstraction and its implementation**.

The purpose is to achieve potential for change: the internal mechanisms of the component can be improved without impact on other components, or the component can be replaced with a different one that supports the same public interface. Encapsulation also **protects the integrity of the component**, by preventing users from setting the internal data of the component into an **invalid or inconsistent state**. Another benefit of encapsulation is that it **reduces system complexity **and thus **increases robustness**, by limiting the interdependencies between software components.

Encapsulating software behind an interface allows the construction of objects that mimic the behaviour and interactions of objects in the real world. For example, a **simple digital alarm clock **is a real-world object that a person can use and understand. They can understand what the alarm clock does, and how to use it through the provided interface (buttons and screen), without having to understand every part inside the clock. Similarly, if you replaced the clock with a different model, the person could continue to use it in the same way, provided that the interface works the same.

Thus encapsulation is used to mean either an **information hiding mechanism**, a **bundling mechanism, **or the combination of the two.

Put simply, **encapsulation is about hiding complexity**. In the real world, objects frequently hide their information and how they work. You don't need to know the internal details in order to use an object. For example, you don't need to understand how a gasoline-powered internal combustion engine works in order to drive a car. By extension, you should be able to drive a car that uses another fuel source or an electric motor without knowing how each of those work either.

When you create an object in an object-oriented language, you can **hide the complexity of the internal workings of the object**. As a developer, there are two main reasons why you would choose to hide complexity.

The first reason is to **provide a simplified and understandable way to use your object without the need to understand the complexity inside**. As mentioned above, a driver doesn't need to know how to provide fuel, how to stop the car, and how to turn off the engine. You know to use the key, the shifter (and possible clutch), the gas pedal and the brake pedal to accomplish each of these  operations. These **basic operations form an interface for the car**. *Think of an interface as the collection of things you can do to the car without knowing how each of those things works.*

Hiding the complexity of the car from the user allows anyone, not just the mechanic, to drive a car. In the same way, **hiding the complex functionality **of your object from the user **allows anyone to use it and to find ways to reuse it **in the future regardless of their knowledge of the internal workings. This concept of keeping implementation details hidden from the rest of the system is key to object-oriented design.

**Information Hiding**

Information Hiding is the principle of segregation of the design decisions in a computer program that are most likely to change, thus protecting other parts of the program from extensive modification if the design decision is changed. The protection involves **providing a stable interface **which **protects the remainder of the program from the implementation **(the details that are most likely to change).

Information hiding is the ability to **prevent certain aspects of a class or software component from being accessible to it's clients**, using either programming language features(like private variables).

One may think **information hiding as being the principle **and **encapsulation being the technique**.

Take a look at the CombustionEngine class below. Notice that it has only two public methods: start() and stop(). Those public methods can be called from outside of the object. All of the other functions are private, meaning that they are not publicly visible to the rest of the application and **cannot be called from outside of the object.**

- Encapsulation is also called **protection **or **information hiding**.

- **Definition** : Encapsulation is the **inclusion within a program object **of all the **resources needed **for the object to function - basically, the **method **and the **data**. The object is said to "**publish its interfaces**". Other objects adhere to these interfaces to use the object without having to be concerned with how the object accomplishes it. The idea is **"don't tell me how to do it; just do it."** An object can be thought of as a self-contained atom. The object interface consists of public methods and instantiate data.

- **Protection **and **information hiding **are techniques used to accomplish encapsulation of an object. **Protection **is when you **limit the use **of class data or methods. **Information **hiding is when you remove data, methods or code from a **class's public interface **in order to refine the scope of an object.

- It is important to note that **information hiding is not the same as encapsulation**. Just because you protect or hide methods or data, does not mean you are encapsulating an object. But the ability to protect or hide methods or data, provide the ability to encapsulate an object. You might say that encapsulating is the proper use of protection and information hiding. As an example, if I used information hiding to hide members that should clearly be in the public interface, then I am using information hiding techniques, but I am not encapsulating the class. In fact, I am doing the exact opposite (un-encapsulating the class). Do not get the idea that encapsulation is only information hiding. **Encapsulation is a lot more**. **Protection is another way of encapsulating a class**. Protection is about adding methods and data to a class. When you add methods or data to a class, then you are protecting the methods or data from use without first having an object of the class.
