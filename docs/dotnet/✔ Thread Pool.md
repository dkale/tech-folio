# ✔ Thread Pool

## Table of Contents

- [Content](#content)

## Content
Thread Pool
A thread pool is a collection of threads that can be used to perform a number of tasks in the background. This leaves the primary thread free to perform other tasks asynchronously.

Thread pools are often employed in server applications. Each incoming request is assigned to a thread from the thread pool, so the request can be processed asynchronously, without tying up the primary thread or delaying the processing of subsequent requests.

Once a thread in the pool completes its task, it is returned to the queue of waiting threads, where it can be reused. This reuse enables applications to avoid the cost of creating a new thread for each task.

Thread pools typically have a maximum number of threads. If all the threads are busy, additional tasks are placed in queue until they can be serviced as threads become available.

You can implement your own thread pool, but it is easier to use the thread pool provided by the .Net Framework through the ThreadPool class.

==============

Whenever you start a thread, a few hundred microseconds are spent organising such things as a fresh private local variable stack. Each thread also consumes (by default) around 1 MB of memory. The thread pool cuts those overheads by sharing and recycling threads, allowing multithreading to be applied at a very granular level without a performance penalty. This is useful when leveraging multi-core processors to execute computationally intensive code in parallel in "divide-and- conquer" style.

Thread pooling is the process of creating a collection of threads during the initialization of a multithreading application, and then reusing those threads for new tasks as and when required, instead of creating new threads. Then every process has some fixed number of threads depending on the amount of memory available, those threads are the need of the application but we have freedom to increase the number of threads. Every thread in the pool has a specific given task. The thread returns to the pool and waits for the next assignment when the given task is completed.

Usually thread pool is required when we have a number of threads created to perform a number of tasks, in this organised queue. Typically, we have more tasks than threads. As soon as a thread completes its tasks, it will request the next task from the queue until all tasks have been completed. The thread can then terminate, or sleep until there are new tasks available.

Creating thread pooling
There are a number of ways to create the thread pool :
1. Task Parallel Library
1. Calling ThreadPool.QueueUserWorkItem
1. Asynchronous delegates
1. BackgroundWorker

Entering Thread Pool via TPL
The Task class is the part of .Net Framework 4.0 if you’re familiar with the older constructs, consider the non generic Task class a replacement for ThreadPool.QueueUserWorkItem, and the generic Task<TResult> a replacement for asynchronous delegates. The newer constructs are faster, more convenient, and more flexible than the old.

To use the non generic Task class, call Task.Factory.StartNew, passing in a delegate of the target method:

using System.Threading.Tasks;
using System.Threading;
using System.Diagnostics;
using System;
class Akshay
{
    static void Run()
	{
        Console.WriteLine("Welcome to the C# corner thread pool!");
	}
    static void Main() // The Task class is in System.Threading.Tasks
	{
        Task.Factory.StartNew(Run);
        Console.Read();
	}
}

Task.Factory.StartNew returns a Task object, which you can then use to monitor the task-for instance, you can wait for it to complete by calling its Wait method.

The generic Task<Result> class is a subclass of the non generic Task. It lets you get a return value back from the Task after it finishes executing. In the following example, we download a web page using Task<TResult>:

class Akshay
{
    static void Main()
	{
        // Start the task executing:
        Task<string> task = Task.Factory.StartNew<string>
    	(() => DownloadString("http://www.c-sharpcorner.com/"));
        // We can do other work here and it will execute in parallel:
       //RunSomeOtherMethod();
        // When we need the task's return value, we query its Result property:
        // If it's still executing, the current thread will now block (wait)
        // until the task finishes:
        string result = task.Result;
	}
    static string DownloadString(string uri)
	{
        using (var wc = new System.Net.WebClient())
            return wc.DownloadString(uri);
        Console.Read();
	}
}

Entering Thread Pool using ThreadPool.QueueUserWorkItem
If you are using an older version of the .Net Framework (prior to 4.0) you cannot use TPL. Then you must use one of the older constructs for entering the thread pool: ThreadPool.QueueUserWorkItem and asynchronous delegates.

The ThreadPool.QueueUserWorkItem method allows us to launch the execution of a function on the system thread pool. Its declaration is as follows:

ThreadPool.QueueUserWorkItem(new WaitCallback(Consume), ware);

The first parameter specifies the function that we want to execute on the pool. Its signature must match the delegate WaitCallback.

public delegate void WaitCallback (object state);

In just a few lines of code, I’ve recreated a multithreaded consumer-producer application.

using System;
using System.Threading;
using System.Diagnostics;
public class Akshay
{
      public int id;
      public Akshay(int _id)
      {
          id = _id;
      }
}
class Class1
{
      public int QueueLength;
      public Class1()
  	{
          QueueLength = 0;
  	}
      public void Produce(Akshay ware)
      {
    	  ThreadPool.QueueUserWorkItem(
    	  new WaitCallback(Consume), ware);
    	  QueueLength++;
      }
      public void Consume(Object obj)
      {
          Console.WriteLine("Thread {0} consumes {1}",
          Thread.CurrentThread.GetHashCode(), //{0}
          ((Akshay)obj).id); //{1}
          Thread.Sleep(100);
          QueueLength--;
      }
      public static void Main(String[] args)
  	{
          Class1 obj = new Class1();
          for (int i = 0; i < 100; i++)
          {
               obj.Produce(new Akshay(i));
          }
         Console.WriteLine("Thread {0}",
         Thread.CurrentThread.GetHashCode() ); //{0}
         while (obj.QueueLength != 0)
         {
               Thread.Sleep(1000);
          }
          Console.Read();
     }
}

Synchronization Objects
The previous code contains some rather inefficient coding when the main thread cleans up. I repeatedly test the queue length every second until the queue length reaches zero. This may mean that the process will continue executing for up to a full second after the queues are finally drained. I can’t have that.

The following example uses a ManualResetEvent event object that will signal the main thread to exit.

using System;
using System.Threading;
using System.Diagnostics;
public class Akshay
{
      private bool WaitForComplete;
      private ManualResetEvent Event;
      public int QueueLength;
      public int id;
      public Akshay(int _id)
      {
           id = _id;
      }
      public void Wait()
      {
          if (QueueLength == 0)
          {
               return;
          }
          Event = new ManualResetEvent(false);
          WaitForComplete = true;
          Event.WaitOne();
     }
     public void Consume(Object obj)
     {
          Console.WriteLine("Thread {0} consumes {1}",
          Thread.CurrentThread.GetHashCode(), //{0}
          ((Akshay)obj).id); //{1}
          Thread.Sleep(100);
          QueueLength--;
          if (WaitForComplete)
          {
               if (QueueLength == 0)
               {
                    Event.Set();
               }
          };
     }
}

When the consuming thread finishes consuming a ware and detects that the WaitForComplete is true, it will trigger the event when the queue length is zero. Instead of calling the while block when it wants to exit, the main thread calls the Wait instance method. This method sets the WaitForComplete flag and waits on the Event object.

Why we need thread pooling ?
Thread pooling is essential in multithreaded applications for the following reasons.
1. Thread pooling improves the response time of an application as threads are already available in the thread pool waiting for their next assignment and do not need to be created from scratch.
1. Thread pooling saves the CLR from the overhead of creating an entirely new thread for every short-lived task and reclaiming its resources once it dies.
1. Thread pooling enables us to start several tasks without having to set the properties for each thread.
1. Thread pooling enables us to pass state information as an object to the procedure arguments of the task that is being executed.
1. Thread pooling can be employed to fix the maximum number of threads for processing a particular request.

[↑ Back to Top](#table-of-contents)

[↑ Back to Top](#table-of-contents)
