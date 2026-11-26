# ✔ Generics

## Table of Contents

- [Content](#content)

## Content
Generics

Allows you to define Type-Safe classes without compromising Type-Safety, Performance, or Productivity.

In the non-generics era, while developing a general-purpose stack, you would like to use it to store instances of various types. Hence you would have to use an Object-based stack, meaning that the internal data type used in the stack is an amorphous object, and the stack methods interact with Objects.

public class Stack
{
   object[] m_Items;
   public void Push(object item)
   {...}
   public object Pop()
   {...}
}

There are two problems with object-based solutions. The first issue is performance. When using values types, you have to box them in order to push and store them, and unbox the value types when popping them off the stack. Boxing and unboxing incurs a significant performance penalty in their own right, but it also increases pressure on the managed heap, resulting in more garbage collections, which is not great performance either. When using reference types instead of values types, there is still a performance penalty because you have to interact with and incur the casting cost.

The second (and often more severe) problem with the object-based solution is type safety. Because the compiler lets you cast anything to and from Object, you lose compile-time type safety. For example, the following code compiles fine, but raises an invalid cast exception at run-time:

Stack stack = new Stack();
stack.Push(1);
//This compiles, but is not type safe, and will throw an exception:
string number = (string)stack.Pop();

You can overcome these two problems by providing a type-specific (and hence type-safe) performant stack. Thus for integers you can implement and use the IntStack, for string, StringStack, etc.

Unfortunately, solving the performance and type-safety problems this way introduces a third, and just as  serious problem - productivity impact. Writing type-specific data structures is a tedious, repetitive, and error-prone task. When fixing a defect in the data structure, you have to fix it not in one place, but as many places as there are type-specific duplicates of what is essentially the same data structure.

What are Generics?
- Generics allow you to define type-safe classes without compromising type-safety, performance, or productivity.
- Are classes, structures, interfaces, and methods that have placeholders (type parameters) for one or more of the types they store or use.
- Generics allow you to define type-safe data structures, without committing to actual data-types.

Generic Benefits
Generics in .Net let you reuse code and the effort you put into implementing. You can develop, test and deploy your code once, reuse it with any type, including future types, all with full compiler support and type-safety. Because the generic code does not force the boxing and unboxing of value types, performance is greatly improved.

Q: Consider the following code, which is part of a console application:
| static void Main(string[] args){  DeveloperList listOfDevelopers = new DeveloperList();  IntegerList listOfNumbers = new IntegerList();    listOfNumbers.DoSomething(5);  listOfDevelopers.DoSomething(new Developer());}public class Developer{  public string Name { get; set; }  public List<string> Skills { get; set; }}public class DeveloperList{  public void DoSomething(Developer developer)  {    Console.WriteLine("Doing something with developer");  }}public class IntegerList{  public void DoSomething(Int32 number)  {    Console.WriteLine("Doing something with number");  }} |
| --- |

Optimise the code in a way to replace DeveloperList and IntegerList with one class named GenericList that will contain a single doSomething method. Make sure to handle the case when GenericList is instantiated by an unexpected type.

The solution should be similar to this:

| static void Main(string[] args){  GenericList<int> listOfNumbers = new GenericList<int>();  GenericList<Developer> listOfDevelopers = new GenericList<Developer>();  GenericList<string> listOfStrings = new GenericList<string>();    listOfNumbers.DoSomething(5);  listOfDevelopers.DoSomething(new Developer());  listOfStrings.DoSomething("Whats up");}public class Developer{  public string Name { get; set; }  public List<string> Skills { get; set; }}class GenericList<T>{  public void DoSomething(T value){  if (value.GetType() == typeof(Int32))      {          Console.WriteLine("Doing something with ");            return;        }        if (value.GetType() == typeof(Developer))        {            Console.WriteLine("Doing something with developer");            return;        }        Console.WriteLine("I cannot do anything with " + value.GetType().ToString());  }} |
| --- |


[↑ Back to Top](#table-of-contents)

[↑ Back to Top](#table-of-contents)
