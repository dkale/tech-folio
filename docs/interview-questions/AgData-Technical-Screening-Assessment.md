# AgData Technical Screening Assessment

**Technical Screening Form**

**You are working on a complex software system where you have a base class Vehicle with various derived classes like Car, Truck, and Motorcycle. Each derived class has a unique method called StartEngine. Additionally, you want to implement a method called Drive in the Vehicle class that can operate any type of vehicle. Which C# concept should you employ to ensure that the Drive method works for all derived classes without knowing their specific types?**

1. Use the base keyword to directly call the StartEngine method of each derived class within the Drive method.

1. Implement method overloading for the StartEngine method in the Vehicle class to accommodate different vehicle types.

1. Declare an abstract StartEngine method in the Vehicle class and override it in each derived class as needed.

1. Create an interface called IEngineStart and have each derived class implement this interface, then use polymorphism to call StartEngine in the Drive method.

**In an http://ASP.NET Core application, you have registered three services:**

```csharp
serviceCollection.AddScoped<IServiceA, A>();
serviceCollection.AddTransient<IServiceB, B>();
serviceCollection.AddTransient<IServiceC, C>();
```

**The A class implements IServiceA with an integer property Value initially set to 0.**

**The B class implements IServiceB and sets the Value property of A to 10 in one of its methods.**

**The C class implements IServiceC and accesses the Value property of A.**

**When you execute the code, what will be the value of the Value property in A when accessed by C?**

1. It will be 0.

1. It will be 10.

1. It will result in a runtime error.

1. It will be unpredictable.

**You are developing a bulk upload feature for user profiles in a RESTful API built with an http://ASP.NET Core application. You have a controller named UserProfileController with various actions. Your objective is to determine the correct route path for the action responsible for bulk uploading user profiles. Which route path should you use for this action?**

```csharp
[HttpPost("api/userprofile/uploadbulkprofiles")]
[HttpPost("api/userprofile/bulk/upload")]
[HttpPost("api/userprofile/bulk")]
[HttpPost("api/userprofile")] and controller method is “bulkUpload”
```

**Which HTTP methods are suitable for creating a new article and editing an existing one in a RESTful API for a content management system, along with the appropriate HTTP status codes?**

1. POST method for creating; PUT method for editing.

1. POST method for creating; PATCH method for editing.

1. PUT method for creating; PATCH method for editing.

1. POST method for creating; POST method for editing.

**You are developing a .NET Core application using Entity Framework Core for database access. You need to retrieve a single record from the database based on a primary key. Which method should you use for this scenario?**

1. .FindAsync(Id)

1. .FirstOrDefaultAsync(x => x.Id == Id)

1. .SingleOrDefaultAsync(x => x.Id == Id)

1. .Where(x => x.Id == Id)

**You are practising Test-Driven Development (TDD) for a complex financial calculation module in your .NET Core application. The requirement is to verify that the calculation of compound interest for a savings account works correctly. What would be an appropriate name for the test method, considering the complexity of the calculation?**

1. Test_CompoundInterest_PerformSuccessfulCalculation

1. Verify_CompoundInterest_SuccessfulCalculation

1. CompoundInterestCalculator_WhenValidData_ShouldPerformSuccessfulCalculation

1. Test_InterestCalculation_Accuracy

**You are working on an Angular application that makes HTTP requests to retrieve user data. You want to ensure that if multiple HTTP requests are in progress simultaneously, only the response from the latest request is processed, and previous responses are ignored. Which RxJS operator is most suitable for achieving this behavior?**

1. switchMap

1. mergeMap

1. concatMap

1. exhaustMap

**What is the approach to use in testing to mock your UntypedFormControl and set their values for testing purposes?**

1. Use AbstractControl

1. Use FormGroup and FormControl

1. Use FormArray

1. Use input.dispatchEvent

**For optimising an Angular application's performance with lazy loading and preloading, which approach is efficient?**

1. Use RouterModule with loadChildren for lazy loading and set a custom preloading strategy for certain modules.

1. Using CanPreload gaurd from '@angular/router' to preload the module

1. Manually use import() for all modules and enable automatic preloading

1. Skip lazy loading entirely, preload every module during the initial app load for optimal performance.

**Consider the following C# code snippet: **

```csharp
public async Task ProcessDataAsync()
{
int result = 0;
Task<int> task1 = CalculateAsync(5);
Task<int> task2 = CalculateAsync(10);
Console.WriteLine("A");
```

**  result = await task1;    **

```csharp
if (result > 5)
{
Console.WriteLine("B");
}
else
{
Console.WriteLine("C");
}
```

**  result += await task2; **

```csharp
if (result > 15)
{
Console.WriteLine("D");
}
else
{
Console.WriteLine("E");
}
}
```

**What will be the sequence of Console.WriteLine statements when you execute the ProcessDataAsync method?**

1. "A, B, D"

1. "A, C, E"

1. "A, B, E"

1. "A, C, D"
