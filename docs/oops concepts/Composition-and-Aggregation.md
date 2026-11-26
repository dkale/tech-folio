# Composition and Aggregation

**COMPOSITION AND AGGREGATION**

You already encounter composition every day. It is the way most physical objects are built Consider an office chair for example. In most cases, that chair is built out of seat, a back, some legs, and potentially wheels. Certain variations of chairs may combine some of these elements but, in most cases the chair is not a single discrete object formed from one piece of material. Rather it is a composition of individual elements that have been assembled together to create a chair. This article is about using composition to create your objects.

```text
Inheritance (“is a”)
```

The Single Responsibility Principle of object-oriented programming states that objects should have just one responsibility. For example, a wheel on an office chair should be responsible for facilitating rolling. You wouldn't expect the wheel to also handle reclining, it couldn't be used appropriately on a chair that didn't recline or on a file cabinet, or on another object that required a wheel but didn't recline. Reuse is a major goal of object-oriented design.

Therefore, when you create a new object by inheriting from another, the goal should be to make the new object a more specific version of the original. For example, using inheritance, you could create a new type of wheel that is made of hard plastic and rolls better on carpet. You could also create one that is covered in rubber for use on hardwood floors. These examples are both good uses of inheritance because the resultant object is still a wheel, just a more specific type of wheel. **Inheritance is about creating an "is a" relationship. **When done right, you can say that "the hard plastic wheel is a wheel."

Because SRP, inheritance shouldn't be used to add more responsibilities to an object. Therefore creating a new object that inherits from wheel but adds a jet engine would be poor practice. You can't really say that jet engine is a wheel.

```text
Composition (“has a”)
```

Composition is about expressing relationships between objects. Think about the chair example. A chair "has a" Seat. A chair "has a" back. And a chair "has a" set of legs. The phrase "has a" implies a relationship where the chair owns, or at minimum, uses, another object. It is this "has a" relationship which is the basis for composition.

Consider the class definitions for each of the chair parts and the chair itself below. Note that for simplicity, each of the chair parts is just an empty object. They would each have properties and methods of their own in a real application.

```csharp
package chairs.parts {
public class Seat {
public function Seat() {}
}
}
package chairs.parts {
public class Back {
public function Back() {}
}
}
package chairs.parts {
public class Leg {
public function Leg() {}
}
}
package chairs {
import chairs.parts.Back;
import chairs.parts.Leg;
public class Chair {
protected var back:Back;
protected var seat:Seat;
protected var legs:Array;
public function Chair() {
```

back = new Back();

seat = new Seat();

legs = new Array();

legs.push( new Leg() );

legs.push( new Leg() );

legs.push( new Leg() );

legs.push( new Leg() );

```text
}
}
}
```

In this example, the chair object itself is composed of an instance of the Back class, an instance of the Seat class and 4 instances of the Leg class. Each of those objects has its own responsibilities but they are related together in a composition referred to as Chair.

Using composition does not mean you will never use inheritance. In fact, when you begin composing a new object, you will frequently still need to choose an appropriate superclass. Supposing that you wanted to create an OfficeChair class, you might start by extending Chair and then adding an additional component which allows the seat to swivel freely. It would be appropriate to say the OfficeChair is a Chair but it has a swivel.

```csharp
package chairs.parts {
public class Swivel {
public function Swivel() {}
}
}
package chairs {
import chairs.parts.Swivel;
public class OfficeChair extends Chair {
protected var swivel:Swivel;
public function OfficeChair() {
super();
```

swivel = new Swivel();

```text
}
}
}
```

In the code above you are visibly using both inheritance and composition to create a new object. This is the most common way you will work when building complex objects.

```text
Aggregation ( “object does not instantiate the objects it has” )
```

There is a closely related concept to composition called aggregation. In conversation the differences between composition and aggregation are often ignored. However, for the sake of accuracy, it will be covered here. Just like composition, aggregation occurs when an object is composed of multiple objects. However, with composition, the internal objects (such as Leg, Seat and Back) are owned by the main object (Chair). If you destroy the Chair, you also likely want the Leg, Seat and Back instances destroyed because they are a composition which, together, form a single chair.

However, imagine you make a new type of Chair called DinnerChair. DinnerChair extends Chair but it also defines a property which refers to the person currently sitting in the DinnerChair. You could say that DinnerChair has a Person.

The Chair instance certainly doesn't own the Person and you probably shouldn't assume that the Person is destroyed if the Chair is destroyed. Further, the Person exists independent of the Chair. The Person can leave this chair and sit on another one. This independence makes a great deal of difference so this combination of objects is referred to as an aggregation instead of composition. When designing your applications, it is important to note that difference.

Usually, when using composition, the object instantiates the objects it has. Look at the Chair class above. You can see that Back, Seat, and Leg are instantiated in the Chair class. When using aggregation, the OBJECT DOES NOT INSTANTIATE THE OBJECTS IT HAS. Look at the code below. The DinnerChair class has a Person, but it does not instantiate Person.

```csharp
package humans {
public class Person {
public function Person() {}
}
}
package chairs {
import humans.Person;
public class DinnerChair extends Chair {
public var person:Person;
public function DinnerChair() {
super();
}
}
}
```

**Where to go from here**

Using a balance of inheritance, composition and aggregation throughout your application will provide you with a flexible structure of extensible objects that model the core elements of your application.
