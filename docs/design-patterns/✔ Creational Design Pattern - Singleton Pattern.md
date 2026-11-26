# Singleton Design Pattern

## Table of Contents

- [When to Use:](#when-to-use)
- [Motivation](#motivation)
- [Intent](#intent)

[http://www.oodesign.com/singleton-pattern.html](http://www.oodesign.com/singleton-pattern.html)

Ensure that only one instance of a class is created and provide a global access point to the object.

NOTE: CONSTRUCTOR ALWAYS NEED TO BE PRIVATE. IMPLEMENTATIONS ONLY VARY ON HOW WE CREATE AND RETURN THE INSTANCE.

Note: Making a constructor private also ensures that the class cannot be inherited. This ensures that users cannot create an instance from a derived class.

![https://lh6.googleusercontent.com/g8u5Cze_WP22PumvL7TZwFtqE0s3LrSTSR8p6fSbs0O78m-SvMn_efWVY7ZKKzcyggGoxeGXK1zRnCPRjXeQBm-82KX67rYtCZUmT6OeqF8ac9m-YAo](data:image/png;base64,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)

## When to Use:

Singleton pattern should be used when we must ensure that only one instance of a class is created and the instance must be available through all the code. A special care should be taken in multithreading environments when multiple threads must access the same resources through the same singleton object.

Common Usage:

Logger Classes:   
The Singleton pattern is used in the design of logger classes. To provide a global logging access point in all the application components without being necessary to create an object each time a logging operation is performed.

Configuration Classes:   
The singleton pattern is used to design the classes which provide the configuration settings for an application. By implementing configuration classes as Singleton, not only do we provide a global access point, but we also keep the instance we use as a cache object. When the class is instantiated (or when a value is read) the singleton will keep the values in its internal structure. If the values are read from the database or from files this avoids reloading the values each time the configuration parameters are used.

Accessing resources in shared mode:   
It can be used in the design of an application that needs to work with the serial port.

Factories implemented as Singletons:

Let’s assume that we design an application with a factory to generate new objects (Account, Customer, Site, Address objects) with their ids, in a multithreading environment. If the factory is instantiated twice in two different threads then it is possible to have two overlapping ids for two different objects. If we implement the Factory as a singleton we avoid this problem.

[↑ Back to Top](#table-of-contents)

## Motivation

Sometimes it’s important to have only one instance for a class. For example, in a system there should be only one window manager (or only a file system or only a print spooler). Usually singletons are used for centralized management of internal or external resources and they provide a global point of access to themselves.

The singleton pattern is one of the simplest design patterns: it involves only one class which is responsible to instantiate itself, to make sure it creates not more than one instance; in the same time it provides a global access to that instance. In this case the same instance can be used from everywhere, being impossible to invoke directly the constructor each time.

[↑ Back to Top](#table-of-contents)

## Intent

Ensure that only one instance of a class is created.

Provide a global point of access to the object.

Implementation Ways: (public static Method | public static Property | public static readonly Property)

The implementation involves a static member in the “Singleton” class, a private constructor and a static public method that returns a reference to the static member. The above UML diagram shows the static structure of this pattern. The diagram is surprisingly simple because Singleton consists of a simple class that holds a reference to a simple instance of itself.

Example 1 : PUBLIC STATIC METHOD

```csharp
class Emp  
{  
   public int No { get; set; }  
   public string Name { get; set; }  
   public string Sal { get; set; }  
 
   // Private static object can access only inside the Emp class.  
   private static Emp emp;  
 
   // Private empty constructor to restrict end use to deny creating the object.  
   private Emp()  
   {  
   }  
 
   // A public method to access outside of the class to create an object.  
   public static Emp CreateObject()  
   {  
       // If the object is null for first time, instantiate it once.  
       if (emp == null)  
       {  
           emp = new Emp();  
       }  
 
       // Return the emp object, when the user request for create an instance.  
       return emp;  
   }  
}  
 
class Program  
{  
   static void Main(string[] args)  
   {  
       Emp emp1 = Emp.CreateObject();  
       emp1.No = 10;  
       emp1.Name = "Sai";  
       emp1.Sal = "10000";  
       Console.WriteLine("Employee 1 Details:
                  \n No: " + emp1.No + "\n Name: " + emp1.Name + "\n Sal: " + emp1.Sal);  
 
       Emp emp2 = Emp.CreateObject();  
       emp2.Name = "Sri";  
       Console.WriteLine("Employee 2 Details:
                  \n No: " + emp2.No + "\n Name: " + emp2.Name + "\n Sal: " + emp2.Sal);  
 
       Emp emp3 = Emp.CreateObject();  
       emp1.Sal = "5000";  
       Console.WriteLine("Employee 2 Details:
                  \n No: " + emp3.No + "\n Name: " + emp3.Name + "\n Sal: " + emp3.Sal);  
       Console.ReadLine();  
   }  
}
```

Example 2 : PUBLIC STATIC PROPERTY

```csharp
public class Singleton  
{  
   // Private static object can access only inside the Emp class.  
   private static Singleton instance;  
 
   // Private empty constructor to restrict end use to deny creating the object.  
   private Singleton()  
   {  
   }  
 
   // A public property to access outside of the class to create an object.  
   public static Singleton Instance  
   {  
       get  
       {  
           if (instance == null)  
           {  
               instance = new Singleton();  
           }  
           return instance;  
       }  
   }  
}
```

Example 3 : PUBLIC STATIC READONLY PROPERTY

```csharp
public sealed class SealedSingleton  
{  
   private static readonly SealedSingleton instance = new SealedSingleton();  
 
   private SealedSingleton()  
   {  
   }  
 
   public static SealedSingleton Instance  
   {  
       get  
       {  
           return instance;  
       }  
   }  
}
```

Example 4: THREAD SAFE SINGLETON CLASS

```csharp
public class Singleton 
{ 
    private static Singleton _instance = null; 

    private static Object _mutex = new Object();
 
    private Singleton(object arg1, object arg2) 
    { 
        // whatever
    } 
 
    public static Singleton GetInstance(object arg1, object arg2)
    { 
        if (_instance == null) 
        { 
          lock (_mutex) // now I can claim some form of thread safety...
          {
              if (_instance == null) 
              { 
                  _instance = new Singleton(arg1, arg2);
              }
          } 
        }

        return _instance;
    }
}
```

Example 5: GENERIC THREAD SAFE SINGLETON CLASS

```csharp
/// <summary> Generic singleton with double check pattern and with instance parameter </summary>
/// <typeparam name="T"></typeparam>
public class SingleObject<T> where T : class, new()
{
    /// <summary> Lock object </summary>
    private static readonly object _lockingObject = new object();

    /// <summary> Instance </summary>
    private static T _singleObject;

    /// <summary> Protected ctor </summary>
    protected SingleObject()
    {
    }

    /// <summary> Instance with parameter </summary>
    /// <param name="param">Parameters</param>
    /// <returns>Instance</returns>
    public static T Instance(params dynamic[] param)
    {
        if (_singleObject == null)
        {
            lock (_lockingObject)
            {
                if (_singleObject == null)
                {
                    _singleObject = (T)Activator.CreateInstance(typeof(T), param);
                }
            }
        }
        return _singleObject;
    }
}
```

[↑ Back to Top](#table-of-contents)
