# POLYMORPHISM

**POLYMORPHISM**

- **One Name, Multiple Forms**.

- Polymorphism is about an object's **ABILITY TO PROVIDE CONTEXT **when methods or operators are called on the object.

**Operator Overloading**

- **Polymorphism is the characteristic of being able to ASSIGN A DIFFERENT MEANING to a particular symbol or "operator" in different contexts.**

- Allows us to have **OVERLOADING OF OPERATORS**, so that an **operation can exhibit different behaviours in different instances**.

- A common programming example is the use of **'+' OPERATOR** which allows similar or polymorphic treatment or numbers (addition), strings (concatenation), and Lists (attachment).

**Method Overloading**

- Polymorphism allows us to have more than one **FUNCTION WITH THE SAME NAME **in a program.

- Method overloading refers to methods that have the same name but different signatures inside the same class.

- Neither method overloading nor method overriding is by itself an implementation of polymorphism.

- Polymorphism is not the same as **method overloading** and **method overriding **(which is instead known as **ad-hoc polymorphism**).

**Interface based Polymorphism**

- Polymorphism is only concerned with the **application of specific implementations to an interface or a more generic base class**.

- **PRIMARY USAGE OF POLYMORPHISM**, is the ability of objects belonging to different types to respond to methods, fields, or property of the same name, each one according to an appropriate **TYPE-SPECIFIC BEHAVIOUR**. The called (calling code), does not have to know the exact type of callee (called object), thus the **EXACT BEHAVIOUR IS DETERMINED AT RUNTIME** (this is called **LATE-BINDING** or **DYNAMIC-BINDING**).

- **INTERFACES PROVIDE ANOTHER WAY TO ACHIEVE POLYMORPHISM**. Interfaces describe properties and methods like class, but unlike classes, interfaces cannot provide any implementation.

- To achieve polymorphism with interface, you implement an interface in different ways in several classes. Client applications can use either the old or new implementations in exactly the same way.

- The **BIGGEST ADVANTAGE TO INTERFACE BASED POLYMORPHISM**** **is that **YOU DO NOT NEED TO RE-COMPILE EXISTING CLIENT APPLICATIONS**** **to get them to work with new interface implementations. Client modules or classes only depend on the interface/contracts instead of actual implementation.

- **AN INTERFACE IS A PUBLIC PRESENTATION OF A CLASS**, and **IT SHOULD ADVERTISE**, to anyone considering to use it, **what methods and constants are available and accessible from the outside**.

- Interfaces allow classes to share a common design.

- **SO AS ITS NAME SUGGESTS, IT ALWAYS SITS BETWEEN, THE USER AND THE CLASS IMPLEMENTING IT.**

- **Polymorphism allows a client to treat different objects in the same way even if they were created from different classes and exhibit different behaviours.**

**Abstract Class based Polymorphism**

- On the other hand, an **ABSTRACT CLASS** is a tool aimed at helping the 'implementer' of the classes that extend it. It is an infrastructure that can **impose restrictions and guidelines **about what the concrete classes should look like. From a **class design perspective, abstract classes are more important than interfaces**. **Interface is a 'contract' for the code to respect**. When not used this way, it applies more to inheritance than polymorphism.

- **ABSTRACT CLASSES DEFINE 'TYPE'**. And when the concrete subclasses use abstract classes and redefine methods, add new ones, etc, there you see polymorphism in action.

**SubClass Polymorphism**

There are multiple ways to achieve polymorphism when programming. The first is called **subclass polymorphism**, but technically, it is really using the **concept of inheritance**. As an example, look at this simple Pen Class having functions DrawLine() and DrawCircle(). Using inheritance, you can create two new types of Pen : FountainPen and BallPen. In this example you can say that FountainPen is a Pen and BallPen is a Pen. The fact that the compiler  recognizes each of these classes as still being a Pen makes this type of Polymorphism possible. Now, let's say we have a **UsePen() function that accepts a Pen reference** and invokes its DrawLine() and DrawCircle() methods. Depending upon the instance of Pen, respective methods will be invoked. In the most basic sense, this is subclass polymorphism. Pen or any of its subclasses can be used when the code requires a Pen, just as you would be willing to accept any type of Pen from a colleague when you need to sign a document. Subclass polymorphism is one technique that provides flexibility and allows future extension of your code. With subclass polymorphism, you can only substitute subclasses for their superclass. You cannot substitute a super class for one of its subclasses.
