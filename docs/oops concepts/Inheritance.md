# Inheritance

**Inheritance**

**Inheritance Points**

* When the derived class object is created, implicitly constructors from highest parent to the derived class are invoked in order of their hierarchy.

* If the parent class constructor is private, you will not be able to derive a child out of it.

* A derived class is a specialisation of the base class.

* Using inheritance, only if the base class has a method implementation or declared as abstract, you will be able to use a reference of that class to invoke that method, which gives you a polymorphic behaviour when instantiated with a derived type.

Inheritance, encapsulation, abstraction, and polymorphism are four fundamental concepts of object-oriented programming. Inheritance enables new classes to receive - or inherits - the properties and methods of existing classes. In previous articles, you learned that an object is a self-contained component that contains properties and methods needed to make a certain type of data useful. You also learned that a class is a blueprint or template to build a specific type of object and that every object is built from a class. **Inheritance is a way to express a relationship between classes.** It’s a way of saying: ***I want to build a new object that is similar to one that already exists, and instead of creating a new class from scratch, I want to reference the existing class and simply indicate what's different*****.**

Using concepts of inheritance, subclassing (making a new class based on a previous one) and overriding (changing how a previous class works), you can **organise your objects into a hierarchy**. Using inheritance to make this hierarchy often creates easier to understand code, but most importantly it allows you to **reuse and organise code more effectively**.

In object-oriented programming, inheritance enables new objects to take on the properties of existing objects. A class that is used as the basis for inheritance is called a **super class or base class**. A class that inherits from a super class is called a **subclass or derived class**. The terms parent class and child class are also acceptable terms to use respectively. A **child inherits visible properties and methods **from its parent while adding additional properties and methods of its own.

Subclasses and super classes can be understood in terms of the **"is a" relationship**. **A subclass "is a" more specific instance of a superclass. **For ex. on orange "is a" citrus fruit, which "is a" fruit. A shepherd "is a" dog, which "is a" animal. A clarinet "is a" woodwind instrument, which "is a" musical instrument. If the "is a" relationship does not exist between a subclass and superclass, you should not use inheritance. An orange "is a" fruit; so it is okay to write an Orange class that is a subclass of a Fruit class. As a contrast, a kitchen "has a" sink. It would not make sense to say a kitchen "is a" sink or that a sink "is a" kitchen. The **"has a" relationship indicates composition** rather than inheritance.

Look at the ChocolateCake class below:

```csharp
public class ChocolateCake
{
public function ChocolateCake() {  }
public function bake():void{
trace("The chocolate cake is baking");
}
public function frost():void{
trace("The chocolate cake now has frosting");
}
public function putCandlesOnCake(numberOfCandles:int):void{
trace("Putting " + numberOfCandles + " candles on the cake. ");
}
}
```

It's a simple class with three methods: bake(), frost(), and putCandlesOnCake(). You would have a good chocolate cake that you could put candles on after instantiating this class. As fun it might be, do you really want or need candles on every chocolate cake you make? Probably not. Typically, candles only go on birthday cakes. Classes should only include the properties and methods that they need. Not every instance of ChocolateCake need candles, so the class shouldn't include that method. Instead, you should write a seperate BirthdayCake class that can have candles. To keep this first example simple, BirthdayCake will inherit from ChocolateCake. Look at a revised version of ChocolateCake:

```csharp
public class ChocolateCake
{
public function ChocolateCake(){}
public function bake():void{
trace("The chocolate cake is baking");
}
public function frost():void{
trace("The chocolate cake now has frosting");
}
}
```

The **putCandlesOnCake**() method has been removed from ChocolateCake. Look at BirthdayCake below. It inherits the properties and methods of Chocolate Cake and adds the ability to put candles on the cake.

```csharp
public class BirthdayCake extends ChocolateCake
{
public function BirthdayCake(){}
public function putCandlesOnCake(numberOfCandles:int):void{
trace("Putting " + numberOfCandles + " candles on the birthday cake.");
}
}
```

Notice BirthdayCake does not explicitly include a method named bake(), but you can still call that method on the instance of BirthdayCake because a subclass inherits all the public methods of its superclass. Since bake() is a public method in ChocolateCake, bake() is also a method in BirthdayCake. And the result of the method - The Chocolate cake is baking - is the same regardless of the instance on which you call the method.

**Multiple Subclasses of a single superclass**

Remember, you can say that a subclass "is a" superclass. In this example, a birthday cake "is a" chocolate cake. That logic makes sense; however, it also means that every birthday cake is always a chocolate cake. With the code examples so far, you can never have any other flavor for your birthday cake. But think about "is a" logic again for this cake-based example. A chocolate cake "is a" cake. A birthday cake "is a" cake. It makes sense to have a Cake class that gets extended by many different types of cakes. One of the common features of all cakes is flavor, so the Cake class has a flavor property, which is set in the constructor. Look at the code below to see Cake and two speciality subclasses **BirthdayCake** and **WeddingCake**.

BirthdayCake and WeddingCake

```csharp
package cakes {
public class Cake {
protected var flavor:String;
public function Cake(flavor:String) {
this.flavor=flavor;
}
public function bake():void{
trace("The cake is baking and it is " + flavor);
}
public function frost():void{
trace("The cake now has frosting");
}
}
}
package cakes {
public class BirthdayCake extends Cake {
public function BirthdayCake(flavor:String){
super(flavor);
}
public function putCandlesOnCake(numberOfCandles:int):void{
trace("Putting " + numberOfCandles + " candles on the birthday cake.");
}
}
}
package cakes {
public class WeddingCake extends Cake {
public function WeddingCake(flavor:String) {
super(flavor);
}
public function makeCakeTiers(numberOfTiers:int):void{
trace("This wedding cake has " + numberOfTiers + " tiers.");
}
}
}
```

You could use the classes in the following way:

```javascript
public var cake:Cake = new Cake("yellow");
public var bDayCake:BirthdayCake = new BirthdayCake("strawberry");
public var wedCake:WeddingCake = new WeddingCake("lemon chiffon");
```

cake.bake();

bDayCake.bake();

bDayCake.putCandlesOnCake(36);

wedCake.bake();

wedCake.makeCakeTiers(4);

```text
//output
```

The cake is baking and it is yellow

The cake is baking and it is strawberry

Putting 36 candles on the birthday cake.

The cake is baking and it is lemon chiffon

This wedding cake has 4 tiers.

Again notice that neither BirthdayCake nor WeddingCake includes a method named bake(), but can still call it on instances of those classes because it is a public method of their superclasses Cake. Both BirthdayCake and WeddingCake have the properties and methods of Cake plus their own methods specific to that type of cake.

Birthday cakes have candles.  Wedding cakes have tiers. Is there any special feature of a chocolate cake that distinguishes it from any other cake? Not really. Therefore, you do not need a seperate class to make a chocolate cake. Simply instantiate that Cake class setting the flavor property to chocolate:

```javascript
public var chocCake:Cake = new Cake("chocholate");
```

**The is operator :**

The **is operator** test whether a variable or expression is **compatible with a given data type**. It examines the inheritance hierarchy and can be used to **check whether an object is an instance of a particular class** or a child (or grandchild, great grandchild, great great grandchild, and so on) of a particular class. It can also **check whether an object is an instance of a class that implements a particular interface**. The following example creates an instance of the Sprite class named mySprite. Then it uses the operator to test whether mySprite is an instance of the Sprite and DisplayObject classes, and whether it implements the IEventDispatched interface.

```actionscript
var mySprite:Sprite = new Sprite();
trace(mySprite is Sprite);	// true
trace(mySprite is DisplayObject);   // true
trace(mySprite is IEventDispatcher);	// true
trace(mySprite is Number);	// false
```

The is operator checks the inheritance hierarchy and properly reports that mySprite is a Sprite and a descendent of the DisplayObject classes. The is operator also checks whether mySprite inherits from any classes that implements the IEventDispatcher interface. Because the Sprite class inherits from the EventDispatcher class, which implements the IEventDispatcher interface, the is operator correctly reports that mySprite implements the same interface. The is operator correctly reports that mySprite is not a Number.

Using the cake example from above, you can check whether instances of Cake, BirthdayCake, and WeddingCake classes are compatible with Cake:

```actionscript
var cake:Cake = new Cake("yellow");
var bDayCake:BirthdayCake = new BirthdayCake("strawberry");
var wedCake:WeddingCake = new WeddingCake("lemon chiffon");
trace(cake is Cake);  //true
trace(bDayCake is Cake);  //true
trace(wedCake is Cake);  //true
trace(cake is Sprite);  //false
```

Because cake is not an instance of Sprite, the is operator correctly returns false.

**Inheritance and controlling access**

You control access to variables, classes, functions, and methods using access control specifiers. When using inheritance, understanding how access control works is important.

**Overriding methods**

One of the advantages of inheritance is that a subclass inherits all of the public and protected methods of its superclass. Sometimes though, a subclass needs to change the functionality defined in the superclass. Overriding is the technique of redefining an inherited method. When you override a method, use the override keyword. The redefinition must have the same number of parameters as the original (inherited) definition and its return type must be the same as the original return type.

Look at the Pen class below:

```csharp
package writing {
public class Pen {
public function Pen() {}
public function line():void {
trace("Pen drew a line.");
}
public function circle():void {
trace("Pen drew a circle.");
}
}
}
```

Then Pen class has two methods: one for drawing a line and one for drawing a circle. Using inheritance, you can create two new types of Pen: FountainPen and BallPen. The FountainPen class extends the Pen class, add one additional method named refill(), and overrides the line() and circle() functions:

```csharp
package writing {
public class FountainPen extends Pen {
```

override public function line():void {

```actionscript
trace("FountainPen drew a line... and a splotch.");
}
```

override public function circle():void {

```csharp
trace("FountainPen almost drew a circle. Need more ink.");
}
public function refill():void {
trace("all full");
}
}
}
```

The BallPen also extends the Pen class and overrides the line() and circle() methods:

```csharp
package writing {
public class RollerBallPen extends Pen {
```

override public function line():void {

```actionscript
trace("RollerBallPen drew a super smooth line.");
}
```

override public function circle():void {

```actionscript
trace("RollerBallPen easily drew a circle.");
}
}
}
```

When you call an overridden method on an instance of the subclass, the overridden version of the method will run. You would use the above classes in the following way:

```javascript
var pen:Pen = new Pen();
```

pen.line();

pen.circle();

```javascript
var fountainPen:FountainPen = new FountainPen();
```

fountainPen.line();

fountainPen.circle();

```javascript
var rbPen:RollerBallPen = new RollerBallPen();
```

rbPen.line();

rbPen.circle();

```text
//output
```

Pen drew a line.

Pen drew a circle.

FountainPen drew a line... and a splotch.

FountainPen almost drew a circle. Need more ink.

RollerBallPen drew a super smooth line.

RollerBallPen easily drew a circle.

## Where to go from here

The benefits of inheritance - defining relationships between classes, organizing classes into groups, and overriding inherited methods - makes it one of the fundamental concepts of object-oriented programming. Good use of inheritance is critical to developing effective object-oriented solutions.
