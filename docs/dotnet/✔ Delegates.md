# ✔ Delegates

## Table of Contents
- [Declaration - Define delegate](#declaration-define-delegate)
- [Instantiation - Create Instance of delegate](#instantiation-create-instance-of-delegate)
- [Invocation - Invoke delegate](#invocation-invoke-delegate)
- [Define](#define)
- [Create instance](#create-instance)
- [Invoke delegate](#invoke-delegate)
DELEGATES
­ http://www.akadia.com/services/dotnet_delegates_and_events.html
- A delegate is a reference to a method.
- All delegates inherit from the System.Delegate type.
- Delegates provide a foundation for event handling.
- Types of Delegate : Unicast & Multicast

1. Delegate is a reference type used to reference a method.
1. Delegates are similar to function pointers, but with a few differences. Delegates are type safe than function pointers. Every method has a physical location in memory and associated memory address. This address is assigned to the delegate. Therefore they can invoke a method to which they refer to (it is different from function pointers which could refer to any method).
1. - Delegate types cannot be inherited and delegate objects are immutable.
1. - A delegate can refer to multiple methods. It can invoke either static methods or instance methods of an object.
1. - Delegates are mainly used for event handlers and callback methods.
1. - A DELEGATE INSTANCE CAN BE PASSED AS A PARAMETER TO OTHER METHODS. OR DELEGATES ALLOW METHODS TO BE PASSED AS PARAMETERS.

1. Using a delegate allows the programmer to encapsulate a reference to a method inside a delegate object.
1. The delegate object can then be passed to code which can call the referenced method, without having to know at compile time which method will be invoked.
1. Sometimes, however, we don't want to call a function directly - we'd like to be able to pass it to somebody else so that they can call it.
1. An interesting and useful property of a delegate is that it does not know or care about the class of the object that it references. Any object will do; all that matters is that the method's argument types and return type match the delegates. This makes delegates perfectly suited for "anonymous" invocation.
1. The signature of a single cast delegate is shown below:
delegate result-type identifier ( parameters );
1. A delegate will allow us to specify what the function we'll be calling looks like without having to specify which function to call.
1. The declaration for a delegate looks just like the declaration for a function, except that in this case, we're declaring the signature of functions that this delegate can reference.
1. There are three steps in defining and using delegates:
            	Declaration > Instantiation > Invocation

UNICAST (Single Cast) DELEGATES
A single cast delegate can refer to and call only one method.

MULTICAST DELEGATES
Multicast delegates can point to more than one method. A multicast delegate maintains a list of functions. All the functions are called when the delegate is invoked.

Delegates are used to indirectly call methods. For example, a delegate refers to a method. When a delegate is invoked, it calls the method it refers to. After completion, this method returns some value to the delegate. This value is then passed to the calling program by the delegate.

Following is the step-wise usage of a delegate:
* When a delegate is declared, it defines a class that is derived from the class System.Delegate. Declaration of single cast and multicast delegate is the same.
* When an instance of a delegate is created, it can encapsulate one method or a list of methods.
* When a delegate instance is invoked with a set of arguments, all the methods to which delegate refer is invoked with a given set of arguments.
* MulticastDelegate is a special class. A MulticastDelegate has a linked list of delegates, called an invocation list, consisting of one or more elements. When a multicast delegate is invoked, the delegates in the invocation list are called synchronously in order in which they appear. If an error occurs during execution of the list then an exception is thrown.

Unicast (Single Cast) Delegate

[↑ Back to Top](#table-of-contents)
## Declaration - Define delegate
delegate string ReverseDelegate(string str);

[↑ Back to Top](#table-of-contents)
## Instantiation - Create Instance of delegate
ReverseDelegate revDel = new ReverseDelegate(myObj.ReverseString);

[↑ Back to Top](#table-of-contents)
## Invocation - Invoke delegate
string revstr = revDel.Invoke("Hello World");  	OR     string revstr = revDel("Hello World");

Multicast Delegate
- A multicast delegate derives from System.MulticastDelegate class
- Provides a synchronous way of invoking the methods in the invocation list.
- Generally multicast delegates have void as return type.
- It’s important to note that a multicast delegate will call its subscribers in an unpredictable order. DO NOT assume they will be called in any specific order.
-


[↑ Back to Top](#table-of-contents)
## Define
delegate void strDelegate(string str);

[↑ Back to Top](#table-of-contents)
## Create instance
strDelegate DelegateObj;
strDelegate UpperDel = new strDelegate(obj.UpperCaseStr);
strDelegate LowerDel = new strDelegate(obj.LowerCaseStr);
DelegateObj = UpperDel;
DelegateObj += LowerDel;

[↑ Back to Top](#table-of-contents)
## Invoke delegate
DelegateObj("GoodMorning"):

Asynchronous Delegates
Note that the call to the invocation list methods is done by the same thread which called the delegate object. We refer to this as a synchronous call. But when making an asynchronous call to a method, the caller thread blocks while the call is active. While the thread is blocked, it could create other threads, despite the fact that the CPU may be idle. So even when these threads might not be consuming CPU, they are wasting resources, which is not cost-effective. When a thread makes an asynchronous call to a method, the call returns immediately. The caller thread is not blocked; it is free to perform some other task. The .NET infrastructure obtains a thread for the thread for the method invocation and delivers the in parameters passed by the calling code. The async thread can then run the method in parallel to the calling thread. If the method generates some data and returns this value, the calling thread must be able to access this data. The .NET asynchronous feature supports two mechanisms: that calling thread can either ask for the results, or the infrastructure can deliver the results to the calling thread when the results are ready.
Asynchronous delegates provide the ability to call a synchronous method in an asynchronous manner. When you call a delegate synchronously, the Invoke method calls the target method directly on the current thread. If the compiler supports asynchronous delegates, then it will generate the Invoke method and the BeginInvoke and EndInvoke methods. If the BeginInvoke method is called, the common language runtime will queue the request and return immediately to the caller. The target method will be called on a thread from the thread pool. The original thread, which submitted the request, is free to continue executing in parallel to the target method, which is running on a threadpool thread. If a callback has been specified on the BeginInvoke, it will be called when the target method returns. In the callback, the EndInvoke can be used on the original thread that submitted a request.
http://www.codeproject.com/Articles/10311/What-s-up-with-BeginInvoke

ANONYMOUS METHOD
- Called as an inline delegate.
- Is a block of code that is used as the parameter for the delegate.
- An anonymous method can be used anywhere a delegate is used and is defined inline, without a method name, with optional parameters and a method body. The scope of the parameters of an anonymous method is the anonymous-method-block. An anonymous method can use generic parameter types like any other method.

delegate string strDelegate(string str);

public static void Main()
{
    	strDelegate UpperStr = delegate(string s) { return s.ToUpper(); };
}

Example:

namespace OopsDemo
{
	delegate void StringOperationsDelegate(string str);

	class Program
	{
    	string reverse, uppercase, lowercase;

    	static void Main(string[] args)
    	{
        	Program progObj = new Program();
        	StringOperationsDelegate strOpsDel = progObj.ReverseString;
            strOpsDel += progObj.UpperCase;
            strOpsDel += progObj.LowerCase;
            strOpsDel("Dayanand");
    	}

    	void ReverseString(string str)
    	{
        	reverse = string.Join("", str.Reverse());
    	}
    	void UpperCase(string str)
  	  {
            uppercase = str.ToUpper();
    	}
    	void LowerCase(string str)
    	{
            lowercase = str.ToLower();
    	}
	}
}





[↑ Back to Top](#table-of-contents)
