# Polymorphism and Interfaces

**Polymorphism and Interfaces**

http://www.artima.com/objectsandjava/webuscript/PolymorphismInterfaces1.html

Polymorphism, which etymologically means "many forms," is the ability to treat an object of any subclass of a base class as if it were an object of the base class. A base class has, therefore, many forms: the base class itself, and any of its subclasses.

If you need to write code that deals with a family of types, the code can ignore type-specific details and just interact with the base type of the family. Even though the code thinks it is sending messages to an object of the base class, the object's class could actually be the base class or any one of its subclasses. This makes your code easier for you to write and easier for others to understand. It also makes your code extensible, because other subclasses could be added later to the family of types, and objects of those new subclasses would also work with the existing code.

**To fully realize the wonders of polymorphism, you must send a message to an object without knowing the actual class of the object.** To do this in Java, you just invoke a method defined in a base type on an object referenced by a variable of the base type. As you saw above, the object referred to by a base class reference might be of the base class or any of its subclasses. Therefore, when you write the code to invoke the method, you don't necessarily know the actual class of the object. Likewise, when you compile the code, the compiler doesn't necessarily know the actual class of the object. At run-time, the Java Virtual Machine determines the actual class of the object, is how you realize the full power of polymorphism in your programs.

```java
class CoffeeCup {
private Liquid innerLiquid;
```

void addLiquid(Liquid liq) {

innerLiquid.add(liq);

```text
// Swirl counterclockwise
```

innerLiquid.swirl(false);

```text
}
}
```

Given the above definition of CoffeeCup, you can reap the benefit of polymorphism by invoking addLiquid() with different kinds of liquid.

```text
// This version doesn't take advantage of polymorphism.
class UglyCoffeeCup {
```

Liquid innerLiquid;

void addLiquid(Liquid liq) {

innerLiquid = liq;

```java
if (liq instanceof Milk) {
```

((Milk) innerLiquid).swirlLikeMilk(false);

```java
}
else if (liq instanceof Coffee) {
```

((Coffee) innerLiquid).swirlLikeCoffee(false);

```text
}
else {
```

innerLiquid.swirlLikeGenericLiquid(false);

```text
}
}
}
```

The above code is more difficult to read and less extensible than the code (shown earlier) that took full advantage of polymorphism. If later you added a new type to the Liquid family, say Tea, you would have to add another if-else statement to the above code. You would not, however, need to make any changes to the earlier addLiquid() implementation that took advantage of polymorphism. That implementation of addLiquid() would still work, even though you wrote it before you knew about Tea. Whenever you find yourself writing a series of if- else statements, where the condition is a test of type, you should try and see if your program can't be redesigned to take advantage of polymorphism.

Programming down the path of polymorphism implies that you write code that lets an object figure out how it should behave.

This attitude towards objects fits well with the object-oriented mindset, because the customary mission of a method is to manipulate the internal data of the object. Objects are supposed to know best how to manipulate their own data, which is the reason data is usually private. Keeping data private gives full responsibility for proper manipulation of the internal state of an object to the object's methods. Polymorphism gives you another way in which you can give objects responsibility for their own behavior--the object's behavior matches its class.

## Static versus Dynamic Binding

The underlying mechanism that makes polymorphism possible is dynamic binding. Except for three special cases, all instance methods in Java programs are dynamically bound. The instance method that is invoked at run-time will be determined by the actual class of the object, not by the type of the reference.

## Interfaces

One of the most important benefits of class extension is that you can take advantage of polymorphism.

Sometimes, however, it is difficult to get the polymorphism you want from the singly-inherited hierarchies you can build with class extension. To help you get more polymorphism than you can easily get with single- inheritance, Java supports a restricted form of multiple inheritance through a construct called the "interface."

Although class extension only allows single inheritance (each class can have at most one direct superclass), Java offers a special variation of multiple inheritance through the "interface." An interface is like an abstract class that has only public abstract methods and public static final fields. An interface (the Java construct) represents a pure interface (the object-oriented concept); it has no implementation.

Interfaces in Java allow you to get the benefits of polymorphism without requiring you to build a singly-inherited family of classes. Although a class can extend only one other class, it can "implement" multiple interfaces.

Although interfaces (like abstract classes) cannot be instantiated by themselves, you can create a variable to hold a reference to an interface type:

```csharp
// In Source Packet in file interface/ex6/Example6a.java
class Example6a {
public static void main(String[] args) {
// OK to declare a variable as an interface type
```

Washable wa;

```text
// Can't instantiate an interface by itself.
```

wa = new Washable(); // THIS WON'T COMPILE

```text
}
}
```

Given an object variable of an interface type (such as Washable wa), you can assign a reference to an object of a class that implements the interface (such as class CoffeeCup):

```csharp
// In Source Packet in file interface/ex6/Example6b.java
class Example6b {
public static void main(String[] args) {
```

Washable wa = new CoffeeCup();

wa.wash();

```text
}
}
```

Interfaces allow you to get the benefits of polymorphism without requiring that you fit everything into one singly-inherited family of classes. In the examples above, the Washable interface defines a standard way to do washing, and any class can implement it. Because of interfaces, you can use class extension to model what objects are. You can use interface implementation to get polymorphism based purely on what an object does. Any desired polymorphism that class extension doesn't produce, you can get with interface implementation.
