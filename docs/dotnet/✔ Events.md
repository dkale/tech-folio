# ✔ Events

## Table of Contents

- [Content](#content)

## Content
EVENTS

" An event in C# is a way for a class to provide automatic notifications to clients of that class when some interesting thing happens to an object.
" An event is a way for a class to allow clients to give it delegates to methods that should be called when the event occurs. When the event occurs, the delegate(s) given to it by its clients are invoked.
" Events enable a class or object to notify other classes or objects when something of interest occurs. " The class that sends (or raises) the event is called the publisher and the classes that receive (or handle) the event are called subscribers.
" For example, a word processor might maintain a list of the open documents. Whenever this list changes, many different objects in the word processor might need to be notified so that the user interface could be updated. By using events, the code that maintains the list of documents doesn’t need to know who needs to be notified – once the list of documents is changed, the event is automatically invoked and every object that needs to be notified is correctly notified. By using events, the modularity of the program is increased.
" The class containing the event is used to publish the event. This is called the publisher class. Some other class that accepts this event is called the subscriber class. Events use the publisher-subscriber model.
" A publisher is an object that contains the definition of the event and the delegate. The event-delegate association is also defined in this object. A publisher class object invokes the event and it is notified to other objects.
" A subscriber is an object that accepts the event and provides an event handler. The delegate in the publisher class invokes the method (event handler) of the subscriber class.
" An event is built upon a delegate.

Event Overview
" The publisher determines when an event is raised; the subscribers determine what action is taken in response to the event.
" An event can have multiple subscribers. A subscriber can handle multiple events from multiple publishers.
" Events that have no subscribers are never raised.
" Events are typically used to signal user actions such as button clicks or menu selections in graphical user interfaces.
" When an event has multiple subscribers, the event handlers are invoked synchronously when an event is raised.

The basic foundation behind the event programming model is the idea of "publisher and subscribers". In this model, you have publishers who will do some logic and publish an 'event'. Publishers will then send out their event only to subscribers who have subscribed to receive the specific event.
In C#, any object can publish a set of events to which other applications can subscribe. When the publishing class raises an event, all the subscribed applications are notified. The figure shows this mechanism.

/

Conventions:
The following important conventions are used with events:
- Event Handlers in the .Net framework return void and take two parameters.
- The first parameter is the source of the event; that is the publishing object.
- The second parameter is an object derived from EventArgs.
- Events are properties of the class publishing the event.
- The keyword event controls how the event property is accessed by the subscribing classes.

Conclusion:
The publishers and subscribers are DECOUPLED by the delegate. This is highly desirable as it makes more flexible and robust code and makes it easier to maintain.


In a typical Windows environment, the user wants to perform various actions by raising events. Not necessarily by users, but events are also raised by program code or system. Delegates are used to bind events to methods which handle the event. Such a method is called an event handler. In some cases multicast delegates are used for event handling. Another use of delegate is that it allows raising many events though they are bonded to the same event handling method.

Tip : The event handler notification system provides a way to isolate different parts of your program from excessive changes.

Working of an event
In event handling mechanism, first an event handler for an event is registered by an object. When the event occurs, all registered handlers are called. These registered handlers are the Event handlers which are represented by delegates.
The event handling methods may belong to the same class or different classes. Suppose we define three classes namely AccountAccessDetails, PrintSlip and BankMenu. These classes define the methods required to handle the balance underflow event.
In a program, delegate and event can be defined as:

delegate void AccountHandler();
. . .
public event AccountHandler BalanceUnderflowEvent;

Event handlers for these three methods can be registered as:
AccountAccessDetails accDetails = new AccountAccessDetails();
PrintSlip printObj = new  PrintSlip();
BankMenu menuObj = new menuObj();
BalanceUnderflowEvent += printObj.PrintUnderflowBalance;
BalanceUnderflowEvent += menuObj.StartUpMenu;
BalanceUnderflowEvent += accDetails.StoreAccessDetails;

Example:

```text
namespace OopsDemo
{
	public delegate void EventHandler();
```



```text
	class Program
	{
    	public static event EventHandler SendNotification;
```



```text
    	static void Main(string[] args)
    	{
        	SendNotification += Dog;
        	SendNotification += Cat;
        	SendNotification += Mouse;
```



```text
            SendNotification();
    	}
```



```text
    	static void Cat() { Console.WriteLine("Cat"); }
    	static void Dog() { Console.WriteLine("Dog"); }
    	static void Mouse() { Console.WriteLine("Mouse"); }
	}
}
```


Event Handlers Using Arguments




[↑ Back to Top](#table-of-contents)

[↑ Back to Top](#table-of-contents)
