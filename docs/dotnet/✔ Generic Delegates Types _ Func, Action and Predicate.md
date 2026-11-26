# ✔ Generic Delegates Types _ Func, Action and Predicate

## Table of Contents

- [Content](#content)

## Content
Generic Delegates Types : Func, Action and Predicate
Func<> : Accepts variable number of parameters and delegate has a return type.
Action<> : Accepts variable number of parameters but cannot return any value.
Predicate<> : Does not have any overload, and only returns true or false. Subjects each item in collection to a test.

C# language has observed dramatic transformation from 2.0 to 3.0 and finally with 4.0. C# 2.0 added generics, simplified delegate syntax, nullable types etc. C# 3.0 takes it a level next by adding LINQ, anonymous methods, anonymous types, etc, and finally C# 4.0 adds dynamism, enhances parallel programming by introducing Task Parallel Library. Of all these changes, some are altogether new functionalities in language, some enhance previous features by adding additional capabilities or by just simplifying the syntactic structure. Now we have multiple ways of doing a particular task. Consider iterating through a list of objects. As a developer you can use basic foreach loop, write a LINQ query (either extension method syntax or query based format) or Parallel.ForEach (conditions applied) construct.

Of all the changes one concept that has received high attention is “Delegate”. C# 2.0 simplified the syntactical structure for implementing delegates. C# 3.0 took it to a level next by adding “Lambda Expressions”. Simplified delegate syntax, lambda expression, anonymous types, anonymous methods,  implicit type variables, etc all come together and play a crucial role in LINQ. But among all these myriad features its the small one that goes unnoticed and one such victim are the “Generic Delegate Types” added in System namespace. Following are the important generic delegate types added in .NET 3.5 framework.

	Action<>
	Func<>
	Predicate<>

These generic delegate types are heavily used by the extension method (e.g. Select, SelectMany, etc.) that are added to the various collection classes (e.g. List, Dictionary, etc) or in fact any collection class that implements IEnumerable interface. To give a clear picture, I have provided the frameworks implementation of Any extension method.


```text
public static bool Any<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate)
{
   if (source == null)
   {
       throw Error.ArgumentNull("source");
   }
   if (predicate == null)
   {
       throw Error.ArgumentNull("predicate");
   }
   foreach (TSource local in source)
   {
       if (predicate(local))
       {
           return true;
       }
   }
   return false;
}
```


All that “Any” extension method does is when provided with a collection of items it checks for a condition and returns true if any of the item in collection satisfies the condition otherwise returns false. What condition has to checked is defined via Func<TSource, bool> delegate. The first parameter in the Any method is marked with “this” keyword, which makes it a extension method on any type that implements IEnumerable<T> interface. Here source can be of any type ranging from very basic Int32 till our very own custom defined types.

So lets start with Func<> generic delegate and see how it is used and by what all extension methods it is used.

Func<>
Like I have said before there are multiple overloads for these generic delegate types. Following are the various overloads for Func<> delegate type.

Func<TResult>
Func<T1,TResult>
Func<T1,T2,TResult>
Func<T1,T2,T3,TResult>
Func<T1,T2,T3,T4,TResult>
Func<T1,T2,T3,T4,T5,TResult>
Func<T1,T2,T3,T4,T5,T6,TResult>
Func<T1,T2,T3,T4,T5,T6,T7,TResult>
Func<T1,T2,T3,T4,T5,T6,T7,T8,TResult>

Consider the fourth Func<> delegate definition. It says, the delegate takes three input parameter of type T1, T2, T3 and returns a value of type TResult. Here T1, T2, T3 and TResult can be of any type. Internally, the actual definition of these delegates looks like this (for the 4th delegate i.e. Func<T1, T2, T3, TResult>) :


```text
public delegate TResult Func<in T1, in T2, in T3, out TResult>(T1 arg1, T2 arg2, T3 arg3);
```


Going back to the definition of Any extension method, its second parameter of type Func<TSource, bool>. It corresponds to the second delegate definition and it takes the source as input and returns either true or false.

Various extension methods that use Func<> delegate are : Aggregate, All, Any, Average, Count, First, FirstOrDefault, GroupBy, GroupJoin, Join, Last, LastOrDefault, Max, Min, OrderBy, OrderByDescending, Select, SelectMany, Single, SingleOrDefault, SkipWhile, Sum, TakeWhile, ToLookUp.

Wow the list was pretty long and it shows the versatility of generic Func<> delegate. Following source code highlight as how to use some of these extension methods.


```text
static void Main(string[] args)
{
var fordCars = new List<string> { "GT", "Ikon", "Landau", "Mainline", "Meteor", "Torino" };
var toyotaCars = new List<string> { "Camry", "Prius", "Innova", "Corolla", "Venza", "Yaris" };
var carMakers = new Dictionary<string, List<string>>();
carMakers.Add("Ford", fordCars);
carMakers.Add("Toyota", toyotaCars);
```



```text
var allNameMoreThanFourChars = fordCars.All(item => item.Length > 4); // False
var anyNameStartingWithM = fordCars.Any(item => item[0].Equals('M')); // True
var anyNameLargeThanTenChars = fordCars.FirstOrDefault(item => item.Length > 10); // null(default)
var orderByName = fordCars.OrderBy(item => item); // Sorted List.Since list if of type string so
// frameworks string comparison is used
var unifiedList = carMakers.SelectMany(item => item.Value); // look carefully,I have used Dictionary "carMaker"
var takeCarsTillVenza = toyotaCars.OrderBy(item => item)
.TakeWhile(item => !item.Equals("Venza"));// Sort the list and take cars until you reach Venza
```



```text
Console.ReadLine();
}
```


I haven’t covered all the extension methods in the above example. Also I have used lambda expression, It you are not familiar or comfortable with lambda expressions then I would strongly suggest to read some standard text and get some hands on practice on it.

Action<>
Just like Fun<> delegate, Action<> too has its own set of overloads but the difference is that it doesn’t have a return parameter. That is, unlike Func<> delegate, Action<> delegate cannot return any value. Following are the various overloads of Action<> delegate.

Action
Action<T1>
Action<T1, T2>
Action<T1, T2, T3>
Action<T1, T2, T3, T4>
Action<T1, T2, T3, T4, T5>
Action<T1, T2, T3, T4, T5, T6>
Action<T1, T2, T3, T4, T5, T6, T7>
Action<T1, T2, T3, T4, T5, T6, T7, T8>

As we can see there is no TResult type specified. Action<> delegate is used when a set of action has to be performed on every item in the collection. And the most  obvious place for its use is the “ForEach” loop. Here is a sample code. For each string in list is printed out into console in Uppercase.


```text
static void Main(string[] args)
{
    var fordCars = new List<string> { "GT", "Ikon", "Landau", "Mainline", "Meteor", "Torino" };
    fordCars.ForEach(item => Console.WriteLine(item.ToUpper()));
}
```



Predicate<T>
Predicate is very much different from the other two. It doesn’t have multiple overloads like the others and it can only return true/false corresponding to each item in the collection. Predicate is used by the following extension methods : Exists, Find, FindAll, FindIndex, FindLast, FindLastIndex, RemoveAll, TrueForAll.


```text
static void Main(string[] args)
{
       var fordCars = new List<string> { "GT", "Ikon", "Landau", "Mainline", "Meteor", "Torino" };
       var nameLargeThanFiveChar = fordCars.FindAll(item => item.Length > 5);
}
```


There are numerous examples on internet as on how to use these generic delegates. But one important thing that distinguishes each one of them with the others is the returned value. As it must be clear that each of these delegates take IEnumerable (basically collection of items) as input and return either a collection, scalar value (single item) or nothing. Following listing describes what could be the possible outcomes of these delegates.

Func<> : Func<> has the capability to return value. Returned value can be of any type, irrespective of the type of input source. This makes it the most used generic delegate. When working with Func<> delegate you can get the following possible outcomes:
	- Same type and length as of the input source e.g. OrderBy
	- Same length but different type e.g. Select (fordCars.Select(item=>item.Length))
	- Single value same type e.g. First
	- Single value different type e.g. Average
	- Same type but shortened in length, e.g. TakeWhile, SkipWhile

Action<> : Since action doesn’t return anything, you will always get the output of same length and of same type as that of original (input source) collection.

Predicate<> : Predicate subjects each item in collection to a test and returns true or false. So with predicate you can have either same length or less but the type of output data will always be of same type as that of original (input source) collection.

It is very much likely that I might have missed some possibilities while listing the above items. But all in all when you are working with collection and you know what kind of output you want, you are then pretty much sure what delegate type to use.

Finishing this article with a sample code which lists various directories and the corresponding files in them and their respective file size.


```text
1:  DirectoryInfo[] dirs = new DirectoryInfo(@"G:\My Books").GetDirectories();
  2:             var result = dirs.Select(item => new
  3:             {
  4:                 DirectoryName = item.FullName,
  5:                 Created = item.CreationTime,
  6:                 Files = item.GetFiles().Select(value => new
  7:                 {
  8:                     FileName = value.Name,
  9:                     Length = value.Length,
 10:                 })
 11:             });
 12:  foreach (var item in result)
 13:  {
 14:         Console.WriteLine("Directory : " + item.DirectoryName);
 15:         foreach (var files in item.Files)
 16:              Console.WriteLine(files.FileName + " Length : " + files.Length);
 17:  }
```


I hope it would be clear now as to where in the above code which generic delegate type is used.

[↑ Back to Top](#table-of-contents)

[↑ Back to Top](#table-of-contents)
