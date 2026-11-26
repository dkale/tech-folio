# OOPS, SOLID, DRY, DESIGN PATTERNS

**✔ OOPS, SOLID, DRY, DESIGN PATTERNS**

Now, OOP from the most simplistic perspective revolves around inheritance and polymorphism. These should be concepts you're already familiar with. Once you understand these two very core concepts, you can start working toward following some principles and patterns. There's SOLID, DRY, and then there's patterns.

You can get really far with just adopting the SOLID principles.

Briefly, it's

**S: Single Responsibility Principle** (Classes should have only one reason to change)

**O: Open-Closed Principle** (Classes should be closed for modification but open for extension)

**L: Liskov Substitution Principle** (Super classes should be substitutable for sub classes)

**I: Interface Segregation Principle** (Interfaces should convey a focused concept. Those that don't should be refactored into more specialised interfaces)

**D: Dependency Inversion Principle** (Dependencies that classes rely on should be inverted so that that class is no longer tightly coupled to it)

Furthermore from that, there's DRY principle:

DRY: Don't repeat yourself. Each entity in the system should have a dominant representation.

This all sounds pretty simple huh? In reality, it's quite tricky. Getting the hang of constantly thinking about these principles and knowing when to apply them simply takes practice.

In the end, they are just principles though. Eventually (I'm not there yet), it should come natural to you. It's like always doing the same thing until it's easier. It didn't really get easier, but your brain mapped all these little shortcuts to achieve the same results quicker and more accurately. It just takes time and practice.

As for patterns, there's a family of them in each core concept. The concepts are structural, behavioural, and creational.

Briefly,

**Structural: **Methodology for organising classes in a sane and intuitive way. The most popular patterns are: adapter, bridge, decorator, and facade.

**Behavioral: **Provides ways to supply various implementations of algorithms essentially. Popular ones are: state, strategy, template, and observer.

**Creational: **How objects are created in the life of application. Some of these include: factory, builder, singleton, and abstract factory.

All of these help get you closer toward more OO code. Unfortunately, there's just no exact science to it (or if there is, we haven't discovered it yet).

My final thought here is that you should work toward SOLID most of all. With a SOLID foundation, you should refactor existing code to patterns for the most part. When you're comfortable, the factory, facade, observer, decorator, and singleton are going to be the most used tools in your box and with time, you'll know when to implement them without having to have refactored them.

Take it slow though. Try to just implement DRY and the S in SOLID for a while. I know it's a lot to digest.

PS: Try to get to the point where you can almost recite everything I said from memory in your own words. Being able to talk about it like I did in this post really lets you internalise these practices better.

Oh and excuse any inaccuracies too please. As stated, this is from memory. If you want specifics, do some google searches.
