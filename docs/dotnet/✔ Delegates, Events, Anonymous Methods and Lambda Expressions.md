# ✔ Delegates, Events, Anonymous Methods and Lambda Expressions

## Table of Contents

- [Content](#content)

## Content
Delegates, Events, Anonymous Methods and Lambda Expressions

DELEGATE: A delegate is an object which represents a method and, optionally, the “this” object associated with that method. When the delegate is invoked, the corresponding method is invoked. Delegates contain all that is needed to allow a method, with a specific signature and return type, to be invoked by your code. A delegate can be passed to methods, and a reference to it can be stored in a structure or class. A delegate is used when, at design time, you do not know which method you need to call, and the information to determine this is available only at runtime.

Another scenario is when the code calling a method is being developed independently of the code that will supply the method to be called. The classic example is a Windows Forms control. If you create a control, you are unlikely to know what method should be called in the application when the control raises an event, so you must provide a delegate to allow the application to hook up a handler to the event. When other developers use your control, they will typically decide when they are adding control (through the designer or programmatically) and which method should be called to handle the event published by the control.


EVENTS: Events are specialised delegate types primarily used for message or notification passing. Events can only be invoked from the type they are published from and are typically based on the EventHandler delegate with an object representing the sender of the event and a System.EventArgs derived class holding data about the event.


ANONYMOUS METHODS: Anonymous Methods are expressions that can be converted to delegates. They are a feature of the C# compiler and not a CLR type. An anonymous method is ultimately created as a delegate instance by the compiler, but the syntax for declaring an anonymous method can be more concise than declaring a regular delegate. Anonymous methods also permit you to capture variables in the same scope.


LAMBDA EXPRESSIONS: Lambda expressions are functions with a different syntax that enables them to be used in an expression context instead of the usual object-oriented method of being a member of a class. This means that with a single syntax, we can express a method definition, declaration, and the invocation of a delegate to execute it, just as anonymous methods can, but with a more terse syntax.



[↑ Back to Top](#table-of-contents)

[↑ Back to Top](#table-of-contents)
