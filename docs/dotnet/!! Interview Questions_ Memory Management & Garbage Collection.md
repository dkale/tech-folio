# !! Interview Questions_ Memory Management & Garbage Collection

## Table of Contents
- [The Garbage Collector Groups Objects into Generations](#the-garbage-collector-groups-objects-into-generations)
- [How Generation help the garbage collector run more efficiently](#how-generation-help-the-garbage-collector-run-more-efficiently)
- [Garbage Collection of objects with Finalizers](#garbage-collection-of-objects-with-finalizers)
- [How does Garbage Collector in .NET CLR manage object lifetime? What is non-deterministic finalization?](#how-does-garbage-collector-in-net-clr-manage-object-lifetime-what-is-non-deterministic-finalization)
- [When to Implement a Finalizer](#when-to-implement-a-finalizer)
- [You Shouldn’t Explicitly Force Garbage Collection](#you-shouldnt-explicitly-force-garbage-collection)
- [Accessing the Original Object Referenced through a WeakReference](#accessing-the-original-object-referenced-through-a-weakreference)
- [Destruction vs. Collection](#destruction-vs-collection)
- [How to Tell if an Object has been Garbage Collected ?](#how-to-tell-if-an-object-has-been-garbage-collected)
- [The Garbage Collector](#the-garbage-collector)
- [Nondeterministic Destruction and Object Finalization](#nondeterministic-destruction-and-object-finalization)
- [Use a Destructor to Free Unmanaged Resources](#use-a-destructor-to-free-unmanaged-resources)
- [Use the Dispose Pattern for Deterministic Destruction](#use-the-dispose-pattern-for-deterministic-destruction)
- [A Dispose Pattern Example](#a-dispose-pattern-example)
- [The using Statement Automates Invocation of the Dispose Method](#the-using-statement-automates-invocation-of-the-dispose-method)
- [How is the using() pattern useful? What is IDisposable? How does it support deterministic finalization?](#how-is-the-using-pattern-useful-what-is-idisposable-how-does-it-support-deterministic-finalization)
- [Difference between destructor, dispose and finalize method ?](#difference-between-destructor-dispose-and-finalize-method)
Memory Management & Garbage Collection Interview Questions


[↑ Back to Top](#table-of-contents)
## The Garbage Collector Groups Objects into Generations
Referenced-typed objects in an application have different lifetimes. The application will use some objects as long as the application is running.  Others are referenced only during execution of a single method.
If the garbage collector always examined every object whenever it did a garbage collection pass, it would spend a lot of time reexamining longer-living objects that can’t yet be garbage collected.  The garbage collector can perform more efficiently by looking at only a subset of all objects during each pass.  It does this by grouping objects into generations:
1. Generation 0 – Objects that have been created since the last GC pass  (newest objects)
1. Generation 1 – Objects that have survived one pass of the GC
1. Generation 2 – All other objects  (oldest objects)
The garbage collector examines and collect objects in generation 0, moving to higher generations only if it needs additional memory.
Objects are promoted to the next generation only if a GC pass is done on the generation in which they are located.


[↑ Back to Top](#table-of-contents)
## How Generation help the garbage collector run more efficiently
Objects allocated on the managed heap are grouped into generations by the garbage collector:
1. Generation 0 – Newest objects
1. Generation 1 – Objects that have survived one GC pass
1. Generation 2 – Objects that have survived more than one GC pass
When the garbage collector performs a GC pass, it looks only at objects in generation 0, releasing memory for any that are no longer reachable. In this way, the garbage collector is more efficient, because it is only looking at a portion of the managed heap.
If the garbage collector has collected Gen 0, but the application requires more memory, the garbage collector can then garbage collect generation 1. If the application still requires memory after collecting generation 1, it can move on to generation 2.
This scheme relies on the fact that if an object survives one GC pass, it likely has a longer lifetime and will survive future passes.


[↑ Back to Top](#table-of-contents)
## Garbage Collection of objects with Finalizers
The purpose of the garbage collector is to discover objects on the managed heap that are no longer reachable and to reclaim the memory occupied by these objects.
Once an object is discovered to be no longer reachable, the garbage collector checks to see if the object has a finalizer. If the object doesn’t have a finalizer, it is marked for collection and its memory is reclaimed by the garbage collector.
If an object has a finalizer, it will not be marked for garbage collection, but instead be marked for finalization. It will not get garbage collected during the current pass of the garbage collector. Instead, its finalizer will be run at some point after garbage collection finishes. It will then be eligible for collection during the next pass of the garbage collector.

* Reclaiming memory for objects with finalizers therefore requires at least two passes of the garbage collection.


[↑ Back to Top](#table-of-contents)
## How does Garbage Collector in .NET CLR manage object lifetime? What is non-deterministic finalization?
The hugely simplistic version is that every time it garbage-collects, it starts by assuming everything to be garbage, then goes through and builds a list of everything reachable. Those become not-garbage; everything else doesn't, and gets thrown away. What makes it generational is that every time an object goes through this process and survives, it is noted as being a member of an older generation (up to 2, right now). When the garbage-collector is trying to free memory, it starts with the lowest generation (0) and only works up to higher ones if it can't free up enough space, on the grounds that shorter-lived objects are more likely to have been freed than longer-lived ones.

Non-deterministic finalization implies that the destructor (if any) of an object will not necessarily be run (nor its memory cleaned up, but that's a relatively minor issue) immediately upon its going out of scope. Instead, it will wait until first the garbage collector gets around to finding it, and then the finalization queue empties down to it; and if the process ends before this happens, it may not be finalized at all. (Although the operating system will usually clean up any process-external resources left open - note the usually there, especially as the exceptions tend to hurt a lot.)

If one object loses all its references, we can no longer use that object. These types of objects are called as garbage and they are subjected to Garbage Collection by CLR.

Because of the non-deterministic nature of the Garbage Collection, we can never predict when this object will be collected by the CLR. This is generally done at the time of requirement of memory or when the application terminates.


[↑ Back to Top](#table-of-contents)
## When to Implement a Finalizer
In C#, you implement a finalizer using the destructor (~) syntax.  The finalizer is called by the system when the object is being destroyed.

In general, you rarely need to implement a finalizer.  Rules of thumb for implementing a finalizer include:

1. Implement a finalizer only when the object has unmanaged resources to clean up (e.g. file handles)
1. Do not implement a finalizer if you don’t have unmanaged resources to clean up
1. The finalizer should release all of the object’s unmanaged resources
1. Implement the finalizer as part of the dispose pattern, which allows for deterministic destruction
1. The finalizer should only concern itself with cleanup of objects owned within the class where it is defined
1. The finalizer should avoid side-effects and only include cleanup code
1. The finalizer should not add references to any objects, including a reference to the finalizer’s own object
1. The finalizer should not call methods in any other objects


[↑ Back to Top](#table-of-contents)
## You Shouldn’t Explicitly Force Garbage Collection
For purposes of understanding what happens when objects get garbage collected, you can force garbage collection to happen by calling the GC.Collect method.  Calling this method in production code is almost always a bad idea.
The garbage collector is finely tuned for maximum performance and when you call GC.Collect explicitly, you disturb that tuning and make the garbage collector less efficient.
When you feel that you need to call GC.Collect in order to explicit release objects, it’s generally a sign of poor design in some area of your code.  (E.g. Not making proper use of a Dispose method on an object when you are done using it).


[↑ Back to Top](#table-of-contents)
## Accessing the Original Object Referenced through a WeakReference
You can create a WeakReference instance that refers to an object and then use the WeakReference to determine whether the object has been garbage collected.
You can also refer back to the original object referenced by the WeakReference using its Target property.

```text
Dog dog = new Dog("Bowser");
WeakReference dogRef = new WeakReference(dog);
Dog origDog = (Dog)dogRef.Target;
```



[↑ Back to Top](#table-of-contents)
## Destruction vs. Collection
Automatic memory management in C# means that the memory for a heap-based object is automatically released when there is no longer any user-written code that references the object.  This is done by the garbage collector.
An object goes through two distinct phases when it is no longer referenced–destruction and collection.
When an object is no longer referenced by any user-written code, other than code in a destructor, the object becomes eligible for destruction.
Once the object is eligible for destruction, it may be destructed at some point in the future, i.e. its destructor is called.
After the object is destructed and it is no longer being referenced by any code, including code in a destructor, the object becomes eligible for collection.
Once the object is eligible for collection, the garbage collector may at some point in the future release memory for the object.


[↑ Back to Top](#table-of-contents)
## How to Tell if an Object has been Garbage Collected ?
You normally can’t tell whether an object has been garbage collected by using some reference to the object–because once you have a reference to the object, it won’t be garbage collected.
You can instead create a weak reference to an object using the WeakReference object.  The weak reference is one that won’t be counted as a reference, for purposes of garbage collection.
In the code below, we check before and after garbage collection to show that a Dog object is garbage collected.

```text
Dog dog = new Dog("Bowser");
```



```text
WeakReference dogRef = new WeakReference(dog);
Console.WriteLine(dogRef.IsAlive);
```



```text
dog = null;
GC.Collect();
```



```text
Console.WriteLine(dogRef.IsAlive);
```


/


[↑ Back to Top](#table-of-contents)
## The Garbage Collector
In C#, you don’t explicitly delete heap-based objects.  Instead, the CLR will reclaim memory from objects that are no longer used, by running a Garbage Collector.
The garbage collector (GC) can release memory only for managed objects created on the heap.  It’s only job is to release memory.  It can’t clean up or release other resources, like file handles or database connections.
The GC will typically perform garbage collection when the amount of available physical memory is becoming too low or when the amount of virtual memory consumed by your application is becoming too large.
You can’t predict when garbage collection is going to happen, or even if it’s going to happen, prior to an application terminating.


[↑ Back to Top](#table-of-contents)
## Nondeterministic Destruction and Object Finalization
In .NET, the CLR (Common Language Runtime) is responsible for reclaiming memory for objects that are no longer being referenced, through garbage collection.
Garbage collection can be done on an object as soon as it’s no longer being referenced by any other objects.  However, the garbage collector may not free memory for an object immediately.  It can choose to free up memory for an object whenever it likes.
Every object has a Finalize method that you can override, which will be called when the object is being destroyed.  This method allows cleaning up any unmanaged resources the object might hold, before the object is destroyed.
The CLR implements nondeterministic destruction–you can’t explicitly force an object’s Finalize method to be called and you’d can’t predict when it is called.  There’s also no guarantee that the finalizer will ever be called–an application might crash before the finalizer is called.


[↑ Back to Top](#table-of-contents)
## Use a Destructor to Free Unmanaged Resources
When an object is no longer referenced by other objects, its memory may be reclaimed by the garbage collector in the CLR.  If you have any unmanaged resources that you need to clean up explicitly, you can implement a finalizer for your class.
A finalizer is the implementation of the Object.Finalize method for your class, which the garbage collector calls when it’s ready to release memory for an object.  In C#, you implement a finalizer using the destructor syntax–a method defined as a tilde (~), followed by the name of the class.

```text
public class Dog
{
    ~Dog()
    {
        Trace.WriteLine("This dog is on the way out..");
    }
```

}

When you look at the IL for this class, you’ll see that the compiler has overridden the Finalize method.

/
You’ll notice that this method is now called when the garbage collector is releasing the object.



[↑ Back to Top](#table-of-contents)
## Use the Dispose Pattern for Deterministic Destruction
You can implement a finalizer in a class to explicitly release unmanaged resources before an object’s memory is freed.  Drawbacks of this approach include:
1. You don’t control when the object’s resources are cleaned up
1. A finalizer can’t make use of any other managed objects  (they may have already been finalized)
Another pattern for releasing unmanaged resources is for a class to implement the IDisposable interface by providing a Dispose method.  Dispose is meant to clean up the object’s resources when it’s called.  A client of the class can decide exactly when it wants to release the object’s resources (deterministic destruction).  Benefits of this pattern include:
1. Can release resources earlier, not waiting for finalization
1. Dispose logic can make use of other managed resources


```text
public class Dog : IDisposable
{
    public void Dispose()
    {
        // Release unmanaged resources here
```



```text
        // Note: need logic to prevent double disposal, etc.
    }
```

}


[↑ Back to Top](#table-of-contents)
## A Dispose Pattern Example
If you want to control when an object’s unmanaged resources are released, you can follow the dispose pattern, implementing a Dispose method.
Here’s a complete example.  We create a method to release resources that is called either when a client invokes Dispose directly or when the CLR is finalizing the object.


```text
public class Dog : IDisposable
{
    // Prevent dispose from happening more than once
    private bool disposed = false;
```



```text
    // IDisposable.Dispose
    public void Dispose()
    {
        // Explicitly dispose of resources
        DoDispose(true);
```



```text
        // Tell GC not to finalize us--we've already done it manually
        GC.SuppressFinalize(this);
    }
```



```text
    // Function called via Dispose method or via Finalizer
    protected virtual void DoDispose(bool explicitDispose)
    {
        if (!disposed)
        {
            // Free some resources only when invoking via Dispose
            if (explicitDispose)
                FreeManagedResources();   // Define this method
```



```text
            // Free unmanaged resources here--whether via Dispose
            //   or via finalizer
            FreeUnmanagedResources();
```



```text
            disposed = true;
        }
    }
```



```text
    // Finalizer
    ~Dog()
    {
        DoDispose(false);
    }
}
```



[↑ Back to Top](#table-of-contents)
## The using Statement Automates Invocation of the Dispose Method
A class may implement the IDisposable interface, providing a Dispose method, to allow for deterministic destruction.  Client code calls Dispose when it is done using the object, telling it to release resources that it might be holding.
It can be hard to ensure that you always call Dispose when appropriate, especially when exceptions occur.
The using statement in C# specifies the scope in which you want to use an object and guarantees that the Dispose method will be called when the object goes out of scope.

```text
using (StreamWriter writer =
    new StreamWriter(@"D:\Remember.txt"))
{
    writer.Write("RIP Steve Jobs, 1955-2011");
}
```


If we look at the IL generated for the using statement, we can see that it is converted to a try/finally block and the Dispose method of the StreamWriter object is called in the finally block.

/

Implementing Finalize and Dispose to Clean Up Unmanaged Resources
http://msdn.microsoft.com/en-us/library/vstudio/b1yfkh5e(v=vs.100).aspx
Class instances often encapsulate control over resources that are not managed by the runtime, such as window handles (HWND), database connections, and so on. Therefore, you should provide both an explicit and an implicit way to free those resources. Provide implicit control by implementing the protected Finalize on an object (destructor syntax in C#). The garbage collector calls this method at some point after there are no longer any valid references to the object.

In some cases, you might want to provide programmers using an object with the ability to explicitly release these external resources before garbage collector frees the object. If an external resource is scarce or expensive, better performance can be achieved if the programmer explicitly releases resources when they are no longer being used. To provide explicit control, implement the Dispose provided by IDisposable. The consumer of the object should call this method when it is finished using the object. Dispose can be called even if other references to the object are alive.

Note that even when you provide explicit control using Dispose, you should provide implicit cleanup using the Finalize method. Finalize provides a backup to prevent resources from permanently leaking if the programmer fails to call Dispose.

For more information about implementing Finalize and Dispose to clean up unmanaged resources, see Garbage Collection. The following example illustrates the basic design pattern for implementing Dispose:


```text
// Design pattern for a base class.
public class Base: IDisposable
{
    private bool disposed = false;

    //Implement IDisposable.
    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }

    protected virtual void Dispose(bool disposing)
    {
        if (!disposed)
        {
            if (disposing)
            {
                // Free other state (managed objects).
            }
            // Free your own state (unmanaged objects).
            // Set large fields to null.
            disposed = true;
        }
    }

    // Use C# destructor syntax for finalization code.
    ~Base()
    {
        // Simply call Dispose(false).
        Dispose (false);
    }
}

// Design pattern for a derived class.
public class Derived: Base
{
    private bool disposed = false;

    protected override void Dispose(bool disposing)
    {
        if (!disposed)
        {
            if (disposing)
            {
                // Release managed resources.
            }
            // Release unmanaged resources.
            // Set large fields to null.
           // Call Dispose on your base class.
            disposed = true;
        }
        base.Dispose(disposing);
    }
    // The derived class does not have a Finalize method
    // or a Dispose method without parameters because it inherits
    // them from the base class.
}
```


Implementing a Dispose Method
http://msdn.microsoft.com/en-us/library/vstudio/fs2xkftw(v=vs.100).aspx




#### Explain Object.Finalize()     ##### RESEARCH THIS TOPIC THROUGH THE LINKS #####
http://msdn.microsoft.com/en-us/library/system.object.finalize(v=vs.110).aspx

How Finalization Works
The Object class provides no implementation for the Finalize method, and the garbage collector does not mark types derived from Object for finalization unless they override the Finalize method.

If a type does override the Finalize method, the garbage collector adds an entry for each instance of the type to an internal structure called the finalization queue. The finalization queue contains entries for all the objects in the managed heap whose finalization code must run before the garbage collector can reclaim their memory. The garbage collector then calls the Finalize method automatically under the following conditions:
1. After the garbage collector has discovered that an object is inaccessible, unless the object has been exempted from finalization by a call to the GC.SuppressFinalize method.
1. During shutdown of an application domain, unless the object is exempt from finalization, During shutdown, even objects that are still accessible are finalized.

Finalize operations have the following limitations:
1. The exact time when the finalizer executes is undefined. To ensure deterministic release of resources for instances of your class, implement a Close method or provide a IDisposable.Dispose implementation.
1. The finalizers of two objects are not guaranteed to run in any specific order, even if one object refers to the other. That is, if Object A has a reference to Object B and both have finalizers, Object B might have already been finalized when the finalizer of Object A starts.
1. The thread on which the finalizer runs is unspecified.

Notes for implementers
You should override Finalize for a class that uses unmanaged resources such as file handles or database connections that must be released when the managed object that uses them is discarded during garbage collection.
The Object.Finalize method does nothing by default, but you should override Finalize only if necessary, and only to release unmanaged resources. Reclaiming memory tends to take much longer if a finalization operation runs, because it requires at least two garbage collections. In addition, you should override the Finalize method for reference types only. The common language runtime only finalizes reference types. It ignores finalizers on value types.

Every implementation of Finalize in a derived type must call its base type’s implementation of Finalize. This is the only case in which application code is allowed to call Finalize.

NOTE: The C# compiler does not allow you to override the Finalize method. Instead, you provide a finalizer by implementing a destructor for your class. A C# destructor automatically calls the destructor of its base class.

Because the garbage collection is non-deterministic, you do not know precisely when the garbage collector performs finalization. To release resources immediately, you can also choose to implement the dispose pattern and the IDisposable interface. The IDisposable.Dispose implementation can be called by consumers of your class to free unmanaged resources, and you can use the Finalize method to free unmanaged resources in the event that the Dispose method is not called.

The SafeHandle alternative
Creating reliable finalizers is often difficult, because you cannot make assumptions about the state of your applications, and because unhandled system exceptions such as OutOfMemoryException and StackOverflowException terminate the finalizer. Instead of implementing a finalizer for your class to release unmanaged resources, you can use an object that is derived from the System.Runtime.InteropServices.SafeHandle class to wrap your unmanaged resources, and then implement the dispose pattern without a finalizer. The .Net Framework provides the following classes in the Microsoft.Win32 namespace that are derived from System.Runtime.InteropServices.SafeHandle :

1. SafeFileHandle is a wrapper class for a file handle.
1. SafeMemoryMappedFileHandle is a wrapper class for memory-mapped file handles.
1. SafeMemoryMappedViewHandle is a wrapper class for a pointer to a block of unmanaged memory.

#### What is the difference between Finalize() and Dispose()?   ###
Finalize
It is called by Garbage Collector internally to clean up the unmanaged resources. It is a destructor and when object is not in a scope (no longer in use) it is automatically called by GC to destroy that object.
We could say it is non-deterministic function. To clear the memory it is uncertain when Garbage Collector will call Finalize() method. It belongs to the Object class. Comparatively slow method of clean up unmanaged resources, as called by GC when the object goes out of the scope (usually at the end of the program)

Dispose
User explicitly calls Dispose method using IDisposable interface again to clean up the unmanaged resources. When we use Dispose method using IDisposable interface, we need to use GC.SuppressFinalize() so that system will not call Finalize method again to clean up the objects. However, it is deterministic function as Dispose() method is explicitly called by the user. As previously said it belongs to the IDisposable interface. Faster method for instant disposal of the objects as user executes it using customer code.
---------------------
http://www.dotnetobject.com/Thread-What-is-the-difference-between-Finalize-and-Dispose
Finalize() is called by Garbage Collector implicitly to free unmanaged resources. The garbage collector calls this method at some point after there are no longer valid references to the object. There are some resources like windows handles, database connections which cannot be collected by the garbage collector. Therefore the programmer needs to call Dispose() method of IDisposable interface.
Dispose() of IDisposable interface is called by the programmer to explicitly release resources when they are no longer being used. Dispose() can be called even if other references to the object are alive.


[↑ Back to Top](#table-of-contents)
## How is the using() pattern useful? What is IDisposable? How does it support deterministic finalization?
The “using” statement provides convenient syntax that ensures the correct use of IDisposable objects. Within the using block, the object is read-only and cannot be modified or reassigned.
To use “using” statement for particular type it must implement IDisposable interface. Then you use “using” statement.
* When you wrap a IDisposable object with using statement, compiler converts it to try-finally block and object gets disposed in finally block.* Read more about it here.
The reason for the "using" statement is to ensure that the object is always disposed correctly, and it doesn't require explicit code to ensure that this happens.
The using block helps manage resources. Conceptually it protects the whole system's resources by specifying the scope of the usage of the resource. The Type used in a using statement must have implemented IDisposable interface.
http://msdn.microsoft.com/en-us/library/yh598w02(v=vs.80).aspx

IDisposable
Defines a method to release allocated/unmanaged resources.
The primary use of this interface is to release unmanaged resources. The garbage collector automatically releases the memory allocated to a managed object when that object is no longer used. However, it is not possible to predict when garbage collection will occur. Furthermore, the garbage collector has no knowledge of unmanaged resources such as window handles, or open files and streams.
Use the Dispose method of this interface to explicitly release unmanaged resources in conjunction with the garbage collector. The consumer of an object can call this method when the object is no longer needed.


[↑ Back to Top](#table-of-contents)
## Difference between destructor, dispose and finalize method ?
Destructor implicitly calls the Finalize method, they are technically same. Dispose is available with those object which implements IDisposable interface.

The destructor implicitly calls Finalize on the base class of the object.

Example from the same link:


```text
class Car
{
~Car() // destructor
{
// cleanup statements...
}
}
```

Destructor code is implicitly translated to the following code :

```text
protected override void Finalize()
{
try
{
// Cleanup statements...
}
finally
{
base.Finalize();
}
}
```

The programmer has no control over when the destructor is called because this is determined by the garbage collector.



[↑ Back to Top](#table-of-contents)
