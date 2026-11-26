# ASP.NET Core Interview Questions

## Table of Contents

- [Middleware and Routing](#middleware-and-routing)
- [Dependency Injection and Configuration](#dependency-injection-and-configuration)
- [Application Startup and Hosting](#application-startup-and-hosting)
- [ASP.NET Core Fundamentals and Architecture](#aspnet-core-fundamentals-and-architecture)
- [Razor Views and Static Files](#razor-views-and-static-files)

## Middleware and Routing

### What is the use of the `Map` extension while adding middleware to the ASP.NET Core pipeline?

The `Map()` method branches the pipeline based on request-path matching. If the request path starts with the given path, middleware on that branch executes.

```csharp
public void Configure(IApplicationBuilder app)
{
    app.Map("/path1", Middleware1);
    app.Map("/path2", Middleware2);
}
```

### What is routing in ASP.NET Core?

Routing is functionality that maps incoming requests to the route handler. The route can have values (extract them from the URL) that are used to process the request. Using the route, routing can find a route handler based on the URL. All the routes are registered when the application is started. There are two types of routing supported by ASP.NET Core.

- Conventional routing
- Attribute routing

The Routing uses routes to map incoming requests with the route handler and Generates URL that is used in response. Mostly, the application has a single collection of routes and this collection is used for the process of the request. The RouteAsync method is used to map incoming requests (that match the URL) available in route collection.

### How to enable Session in ASP.NET Core?

The middleware for the session is provided by the `Microsoft.AspNetCore.Session` package. To use session in an ASP.NET Core application, add this package to the `.csproj` file and add the session middleware to the request pipeline.

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        // ...
        services.AddSession();
        services.AddMvc();
    }

    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        // ...
        app.UseSession();
        // ...
    }
}
```

### How to add a custom middleware in ASP.NET Core?

[TutorialsTeacher: How to add custom middleware](https://www.tutorialsteacher.com/core/how-to-add-custom-middleware-aspnet-core)

### What are the different ways of creating Middleware in .NET Core?

Middleware in ASP.NET Core can be created using different approaches. Here are some of the common ways to create middleware:

**Class-based middleware:** Define a class with a method that takes `HttpContext` and a `Func<Task>` representing the next middleware in the pipeline.

Implement the next middleware in the pipeline.

#### Example

```csharp
public class CustomMiddleware
{
    private readonly RequestDelegate _next;

    public CustomMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // Middleware logic before the next middleware
        await _next(context);
        // Middleware logic after the next middleware
    }
}
```

Register the middleware in `Startup.cs`:

```csharp
app.UseMiddleware<CustomMiddleware>();
```

#### Inline/Anonymous Middleware

Define the middleware logic directly in the `app.Use` or `app.Run` method.

#### Example

```csharp
app.Use(async (context, next) =>
{
    // Middleware logic before the next middleware
    await next();
    // Middleware logic after the next middleware
});
```

#### Map and MapWhen

Use the `Map` and `MapWhen` methods to conditionally branch the middleware pipeline based on request path or other criteria.

#### Example

```csharp
app.Map("/branch1", branch =>
{
    // Middleware specific to "/branch1"
    branch.UseMiddleware<Branch1Middleware>();
});

app.MapWhen(
    context => context.Request.Query.ContainsKey("param"),
    paramBranch =>
    {
        // Middleware for requests with a specific query parameter
        paramBranch.UseMiddleware<ParamMiddleware>();
    });
```

#### Endpoint Middleware

Use endpoint middleware to run logic based on the selected endpoint.

#### Example

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.Map("/endpoint1", context =>
    {
        // Middleware logic for "/endpoint1"
        return context.Response.WriteAsync("Hello from Endpoint1!");
    });
});
```

#### Convention-based Middleware

Use conventions to apply middleware based on specific conditions or criteria.

#### Example

```csharp
app.Use(next => context =>
{
    if (context.Request.Path.StartsWithSegments("/api"))
    {
        // Apply middleware only for requests under "/api"
        // Middleware logic for "/api" requests
    }

    return next(context);
});
```

[↑ Back to Top](#table-of-contents)

## Dependency Injection and Configuration

### What are service lifetimes in ASP.NET Core?

.NET Core supports a design pattern called ‘Dependency Injection’ which helps in the implementation of IoC(Inversion of Control). During registration, dependencies require their lifetime to be defined. The lifetime of service decides under what condition the instance of the service will be created and till what time it will be live.

ASP.NET Core allows us to specify the lifetime for registered services. The service instance gets disposed of automatically based on a specified lifetime. So, we do not care about cleaning these dependencies, it will take care of the ASP.NET Core framework. There are three types of lifetimes:

#### Singleton

#### Transient

#### Scoped

#### Singleton

ASP.NET Core will create and share a single instance of the service throughout the application life. The service can be added as a singleton using the `AddSingleton()` method of `IServiceCollection`. ASP.NET Core creates a service instance at the time of registration, and subsequent requests use this instance. Here, we do not need to implement the Singleton design pattern; ASP.NET Core maintains the single instance.

#### Example

```csharp
services.AddSingleton<IHelloWorldService, HelloWorldService>();
```

#### Transient

ASP.NET Core creates a new service instance each time it is requested. The service can be registered as transient using the `AddTransient` method of `IServiceCollection`. This lifetime can be used for stateless, lightweight services.

When a service is registered with a transient lifetime in the DI container, a new instance of that service is created every time it's requested from the container. In other words, a new object is returned for each dependency injection request, regardless of how many times the service is requested during the application's lifetime.

#### Example

```csharp
services.AddTransient<IHelloWorldService, HelloWorldService>();
```

#### Scoped

ASP.NET Core creates and shares one service instance per request, creating a new instance for each request. The service can be registered as scoped using the `AddScoped` method of `IServiceCollection`. Take care when using a scoped service in middleware: inject it into the `Invoke` or `InvokeAsync` method rather than through the constructor.

#### Example

```csharp
services.AddScoped<IHelloWorldService, HelloWorldService>();
```

#### Let's break down the highlighted statement

"We need to take care while the service is registered via Scoped in middleware..."

This implies that the service is registered with a Scoped lifetime in the DI container. Scoped services in ASP.NET Core are created once per request and are disposed of when the request is completed.

```csharp
// Register a service with a scoped lifetime.
services.AddScoped<IScopedService, ScopedService>();
```

"...and inject the service in the Invoke or InvokeAsync methods."

When you inject a scoped service directly into the `Invoke` or `InvokeAsync` methods of middleware, the service instance is scoped to the current request:

```csharp
public class MyMiddleware
{
    private readonly RequestDelegate _next;

    public MyMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(
        HttpContext context,
        IScopedService scopedService)
    {
        // Use scopedService within the current request.
        // It will be disposed of after the request is completed.
        await _next(context);
    }
}
```

"If we inject dependency via the Constructor, it behaves like a singleton."

If a scoped dependency is injected into the constructor of a middleware component, it behaves like a singleton within the scope of the application rather than the scope of the current request.

```csharp
public class MyMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IScopedService _scopedService;

    public MyMiddleware(
        RequestDelegate next,
        IScopedService scopedService)
    {
        _next = next;
        _scopedService = scopedService;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // This constructor-injected scoped service is shared by the middleware.
        await _next(context);
    }
}
```

In summary, when using scoped services in middleware, it's crucial to understand that injecting them directly into the Invoke or InvokeAsync methods ensures they are scoped to the current request. However, if injected into the constructor, they behave like singletons within the application scope, potentially leading to unintended consequences. Always consider the intended lifetime and scope of your dependencies when working with middleware and dependency injection in ASP.NET Core.

### Problem Statement: In the context of an ASP.NET Core application, there is a registration of a singleton service (Class A) and a scoped service (Class B), where Class A depends on Class B. The question is whether this configuration might lead to issues due to the differing lifetimes of the two classes. (AgData Interview)

In ASP.NET Core, when you register a service as a singleton, it means that there will be only one instance of that service for the entire lifetime of the application. On the other hand, when a service is registered as scoped, it means that a new instance will be created for each scope (usually corresponding to a request in a web application).

Now, if you have registered a singleton dependency (Class A) and a scoped dependency (Class B) where Class A has a dependency on Class B, it can potentially lead to issues related to the differing lifetimes of the two classes.

#### Here's what happens in this scenario

#### Singleton Dependency (Class A)

There is only one instance of Class A throughout the application's lifetime.

#### Scoped Dependency (Class B)

A new instance of Class B is created for each scope (e.g., each HTTP request in a web application).

#### Dependency Relationship (Class A depends on Class B)

When Class A is instantiated, it will have a reference to the scoped instance of Class B.

The issue arises because Class B is scoped, but Class A (which holds a reference to Class B) has a singleton lifetime. This means that Class A may end up holding a reference to a scoped instance of Class B that is no longer valid outside the scope where it was created.

In general, it's a good practice to avoid having a singleton depend directly on a scoped service, as it can lead to unintended consequences and potential issues related to the different lifetimes.

#### To resolve this, consider one of the following approaches

#### Make Class B Singleton

If it makes sense for Class B to have a singleton lifetime, you can register it as a singleton service.

```csharp
services.AddSingleton<ClassA>();
services.AddSingleton<ClassB>();
```

#### Inject Class B Lazily

Rather than injecting Class B directly into the constructor of Class A, you can use lazy initialization to ensure that the scoped instance of Class B is accessed only when needed.

```csharp
public class ClassA
{
    private readonly Lazy<ClassB> _classB;

    public ClassA(Lazy<ClassB> classB)
    {
        _classB = classB;
    }

    // ...
}
```

Choose the approach that best fits your application's requirements and design.

### Resolving service instances with ASP.NET Core Dependency Injection

The `IServiceCollection` interface is used to build a dependency injection container. After it is built, it is composed into an `IServiceProvider` instance that can resolve services. You can inject an `IServiceProvider` into any class. The `IApplicationBuilder` and `HttpContext` classes also expose a service provider through their `ApplicationServices` and `RequestServices` properties, respectively.

#### `IServiceProvider` defines a `GetService(Type type)` method to resolve a service

```csharp
var service = (IFooService)serviceProvider.GetService(typeof(IFooService));
```

#### There are several convenience extension methods available, like

```csharp
serviceProvider.GetService<IFooService>();
```

Add a `using Microsoft.Extensions.DependencyInjection;` directive to use this extension method.

### What is the Options pattern in ASP.NET Core?

The options pattern uses classes to provide strongly typed access to groups of related settings. The `Configure` method on `IServiceCollection` registers these options with the ASP.NET Core dependency injection service container; `IOptions<T>` has a singleton lifetime.

Once configured, a strongly typed options class can be injected into any service or controller through dependency injection using one of the generic options interfaces: `IOptions<T>`, `IOptionsSnapshot<T>`, or `IOptionsMonitor<T>`.

When you are accessing the configuration settings with the isolated classes, the app should adhere to these two principles.

**Interface Segregation Principle (ISP) or Encapsulation:** A class that depends on configuration should depend only on the settings it uses.

**Separation of Concerns:** Settings for different classes should not be related to or dependent on one another.

#### Let’s consider the below configuration

```jsonc
{
  // Other settings omitted.
  "AllowedHosts": "*",
  "DashboardSettings": {
    "Title": "C# Corner - A Community for Software Developers",
    "Header": {
      "IsSearchBoxEnabled": true,
      "BannerTitle": "Breaking News",
      "IsBannerSliderEnabled": false
    }
  }
}
```

#### `IOptions<TOptions>`

The `IOptions` service binds a strongly typed options class to a configuration section and registers it in the ASP.NET Core dependency injection service container with a singleton lifetime. Its `Value` property contains the configured options instance.

Create a class called `DashboardHeaderConfiguration` under the `Models/Configuration` folder, with properties that bind to matching configuration keys:

`DashboardHeaderConfiguration.cs`:

```csharp
public class DashboardHeaderConfiguration
{
    public bool IsSearchBoxEnabled { get; set; }
    public string BannerTitle { get; set; }
    public bool IsBannerSliderEnabled { get; set; }
}
```

Now, we’ll consume this options class to read settings inside our controller.

`Dashboard2Controller.cs`:

```csharp
public class Dashboard2Controller : Controller
{
    private readonly DashboardHeaderConfiguration dashboardHeaderConfig;

    public Dashboard2Controller(
        IOptions<DashboardHeaderConfiguration> options)
    {
        dashboardHeaderConfig = options.Value;
    }

    public IActionResult Index()
    {
        // dashboardHeaderConfig.BannerTitle returns "Breaking News".
        // dashboardHeaderConfig.IsBannerSliderEnabled returns false.
        // dashboardHeaderConfig.IsSearchBoxEnabled returns true.
        return Content(
            JsonConvert.SerializeObject(dashboardHeaderConfig));
    }
}
```

In the above code, we use the `IOptions` interface instead of the `IConfiguration` interface. `IOptions` is in the `Microsoft.Extensions.Options` namespace and is a generic interface that takes `TOptions`; here, the options type is `DashboardHeaderConfiguration`. When `Dashboard2Controller` is executed, the required `IOptions` instance is injected from the ASP.NET dependency injection container. Its `Value` property provides the `DashboardHeaderConfiguration` instance and its mapped settings.

The options class has been created and consumed in the controller, but the configuration is not yet bound. To bind it, call the `Configure` method of `IServiceCollection` from the `ConfigureServices` method in the startup class and register the configuration section against which the options class will bind.

`Startup.cs`:

```csharp
public class Startup
{
    private readonly IConfiguration _configuration;

    public Startup(IConfiguration configuration)
    {
        _configuration = configuration;
    }

    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllersWithViews();
        services.Configure<DashboardHeaderConfiguration>(
            _configuration.GetSection("DashboardSettings:Header"));
    }

    // ...
}
```

Using the Options pattern, we can access the options wherever configurations are needed through our dependency injection - and this can be very useful when the same configuration values need to be consumed across multiple parts of the application.

[↑ Back to Top](#table-of-contents)

## Application Startup and Hosting

### Explain the basic structure of an ASP.NET Core project Startup class

The ‘Startup’ class in an ASP.NET Core project plays a crucial role in configuring the application’s services and defining the request processing pipeline. Here’s an overview of the basic structure of an ASP.NET Core ‘Startup’class:

#### Constructor

The `Startup` class typically has a constructor that accepts an `IConfiguration` parameter. This provides access to settings from sources such as `appsettings.json`, environment variables, or command-line arguments.

```csharp
public class Startup
{
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    public IConfiguration Configuration { get; }
}
```

#### ConfigureServices method

The `ConfigureServices` method registers services with the dependency injection (DI) container. The runtime calls it to configure the application's services.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Add services to the DI container.
    services.AddMvc();
    services.AddScoped<IMyService, MyService>();
    // ...
}
```

#### Configure method

The `Configure` method defines the request-processing pipeline by setting up middleware components that handle HTTP requests and responses.

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
        app.UseDatabaseErrorPage();
    }
    else
    {
        app.UseExceptionHandler("/Home/Error");
        app.UseHsts();
    }

    app.UseHttpsRedirection();
    app.UseStaticFiles();
    app.UseCookiePolicy();
    app.UseMvc(routes =>
    {
        routes.MapRoute(
            name: "default",
            template: "{controller=Home}/{action=Index}/{id?}");
    });
}
```

### How to use multiple environments in ASP.NET Core?

ASP.NET Core uses environment variables to configure application behavior based on the runtime environment. The `launchSettings.json` file sets `ASPNETCORE_ENVIRONMENT` to `Development` on the local machine.

### How to create ASP.NET Core in a Class Library and use the library in a ASP.NET Core web project?

Below is the `TestController` GET API from the class library:

```csharp
using Microsoft.AspNetCore.Mvc;
using System;
using System.Collections.Generic;

namespace PkrUni.SMS.Api
{
    [ApiController]
    [Produces("application/json")]
    [Route("api/[controller]")]
    public class TestController : ControllerBase
    {
        public IEnumerable<string> Get()
        {
            return new string[] { "Chris", "Hadfield" };
        }
    }
}
```

And below is our solution structure where we have referenced our above class library in our main project.

#### Then install the `Microsoft.AspNetCore.Mvc` package in the Razor class library. In `Startup.cs`, use

```csharp
services.AddMvc()
    .AddApplicationPart(
        Assembly.Load(new AssemblyName("PkrUni.SMS.Area.Api")));
```

### What is the purpose of `WebHostBuilder()`?

It is used to build the HTTP pipeline by chaining `webHostBuilder.Use()` calls and then calling `WebHostBuilder.Build()`, following the builder pattern. It is available in the `Microsoft.AspNet.Hosting` namespace. The `Build` method builds the required services and a `Microsoft.AspNetCore.Hosting.IWebHost` that hosts a web application.

### What Is The Purpose Of `UseIISIntegration`?

`UseIISIntegration` configures the port and base path on which the server listens when running behind AspNetCoreModule. The app is also configured to capture startup errors. `WebHostBuilder` uses `UseIISIntegration` when hosting in IIS and IIS Express.

### Explain Startup process in ASP.NET Core.

Everything starts from `Program.cs`:

```csharp
public static void Main(string[] args)
{
    BuildWebHost(args).Run();
}

public static IWebHost BuildWebHost(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup()
        .Build();
```

The `CreateDefaultBuilder` extension method creates a default configuration. It first looks in `appSettings.json` files, then environment variables, and finally command-line arguments.

This also sets up default logger sources (Debug and Console) and loads logging settings from `appsettings.json`.

After `CreateDefaultBuilder` finishes, the `Startup` class is executed. First, its constructor runs. Services are then added to the DI container through the `AddServices` method in the `Startup` class, and the middleware order for handling incoming requests is set up.

### What is the role of `Program.cs` in an ASP.NET Core application, specifically the `CreateWebHostBuilder` method?

#### Entry Point

`Program.cs` serves as the entry point of an ASP.NET Core application. It contains the `Main` method, which is the starting point when the application is executed.

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateWebHostBuilder(args).Build().Run();
    }

    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .UseStartup<Startup>();
}
```

#### WebHostBuilder

The `CreateWebHostBuilder` method creates an `IWebHostBuilder`, which is used to configure and build a `WebHost`. The `WebHost` hosts the ASP.NET Core application.

#### Default Configuration

`WebHost.CreateDefaultBuilder(args)` sets up the web host's default configuration, including environment variables, command-line arguments, and `appsettings.json` files. It also configures logging and the content root.

#### UseStartup

The `UseStartup<Startup>()` method specifies the application's startup class. The `Startup` class configures services, middleware, and the request-processing pipeline.

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        // Configure services (dependency injection)
    }

    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        // Configure the request processing pipeline
    }
}
```

#### Build and Run

After configuring the web host, calling `.Build()` builds the `WebHost` with the specified configuration. Calling `.Run()` starts the web host and makes the application listen for incoming HTTP requests.

The `Run` method is a blocking call, which keeps the application running and listening for requests until it is manually stopped.

In summary, `Program.cs` is the entry point for an ASP.NET Core application. It configures a `WebHostBuilder` using `CreateWebHostBuilder`, sets up default configuration, specifies the `Startup` class, and starts the web host. The `Startup` class configures the application's services and request-processing pipeline.

| Word | Frequency |
|---|---:|
| net | 101 |
| core | 91 |
| asp | 80 |
| class | 69 |
| application | 41 |
| public | 41 |
| service | 41 |
| configuration | 32 |
| request | 31 |
| middleware | 30 |

[↑ Back to Top](#table-of-contents)

## ASP.NET Core Fundamentals and Architecture

### Differentiate between .NET and .NET Core

| .NET Core Framework | .NET Framework |
|---|---|
| It is open-source | Some of its components are open-source |
| Runs across multiple operating systems- Windows, macOS, and Linux, i.e. it is cross-platform | Runs only on Windows, i.e. is not cross-platform |
| Has better support | Relies heavily on command line tools |
| Is flexible in installation, i.e. you can install components on a needed basis. | Must be installed as a whole package |
| Is faster, high-performance, and easier to scale | Less effective in comparison to .NET Core |
| .NET Core ships in smaller NuGet packages | .NET Framework ships as a whole package |

### What is ASP.NET Core, and how does it differ from previous versions of ASP.NET?

ASP.NET Core is an open-source, cross-platform framework for building modern, cloud-based, and internet-connected applications. Below are the key differences:

- **Cross-platform support**
- **Modular and lightweight**
- **Unified MVC and Web API framework:** ASP.NET Core combines MVC and Web API into a single framework for building web applications and APIs. This simplifies development and promotes consistency.
- **Built-in dependency injection**
- **New configuration system:** ASP.NET Core supports configuration from sources such as JSON files, environment variables, and Azure Key Vault.
- **Middleware pipeline:** Developers can compose the request-processing pipeline by adding or removing middleware components.
- **.NET Core and .NET Standard:** ASP.NET Core is built on the cross-platform .NET Core runtime and adheres to .NET Standard for compatibility with other .NET implementations.

### Explain the main components of the ASP.NET Core Architecture

- **Hosting:** ASP.NET Core applications are hosted by a web server, with Kestrel as the default lightweight, cross-platform server. Kestrel can run independently or with servers such as IIS or Nginx for production deployments.
- **Request processing pipeline:** A sequence of middleware components handles HTTP requests. Components are configured in the `Configure` method of the `Startup` class and composed into the desired request-handling flow.
- **Hosting environment:** Provides information about the environment, such as Development, Staging, or Production, so the application can tailor its behavior.
- **Configuration:** Settings can come from sources such as JSON files, environment variables, and command-line arguments. `appsettings.json` is commonly used.
- **Dependency injection (DI):** ASP.NET Core includes built-in DI. The container is configured in the `ConfigureServices` method of the `Startup` class.
- **Routing:** Maps incoming URLs to controllers and actions. ASP.NET Core supports convention-based and attribute routing to define URL patterns.
- **Model-View-Controller (MVC):** Supports scalable, maintainable web applications. Controllers handle requests, models represent data and business logic, and views render the user interface.
- **Middleware:** Pipeline components handle tasks such as authentication, authorization, logging, and serving static files. Developers can use existing middleware or create custom components.
- **ASP.NET Core Identity:** Provides membership features such as user registration, password recovery, and role-based authorization.
- **Entity Framework Core:** A lightweight, extensible, cross-platform object-relational mapper (ORM) for database access.

### How does ASP.NET Core support cross-platform development?

#### Cross-Platform Support in ASP.NET Core

**.NET Core Runtime:**

ASP.NET Core is built on top of the .NET Core runtime, which is a cross-platform and open-source framework. The .NET Core runtime enables ASP.NET Core applications to run on various operating systems, including Windows, Linux, and macOS.

#### Kestrel Web Server

ASP.NET Core includes the Kestrel web server, a lightweight and cross-platform server that is the default server for ASP.NET Core applications. Kestrel is designed to be fast and efficient and can be used on its own or in conjunction with other servers like IIS or Nginx.

#### Platform-Agnostic Development

ASP.NET Core allows developers to write platform-agnostic code by targeting the .NET Standard. The .NET Standard is a set of APIs that are common across different .NET implementations, ensuring code portability across platforms.

#### Command-Line Interface (CLI)

ASP.NET Core applications can be created, built, and run using the cross-platform .NET CLI (Command-Line Interface). The CLI provides commands for tasks such as project creation, compilation, and running applications, making it easy to manage projects across different environments.

#### Cross-Platform Development Tools

Visual Studio Code, a lightweight and cross-platform code editor, is a popular choice for developing ASP.NET Core applications on different operating systems. While Visual Studio is available on Windows, Visual Studio Code provides a consistent development experience across platforms.

#### Containerization

ASP.NET Core applications can be easily containerized using technologies like Docker. Containers encapsulate an application and its dependencies, ensuring consistent behavior across different environments. This is especially beneficial in cross-platform development and deployment scenarios.

#### NuGet Package Management

NuGet, the package manager for .NET, is used to manage dependencies in ASP.NET Core projects. NuGet packages are platform-agnostic, allowing developers to easily include and manage third-party libraries regardless of the target operating system.

#### Cross-Browser Compatibility

ASP.NET Core produces HTML that is compatible with various browsers, promoting cross-browser compatibility. The framework emphasises adherence to web standards, ensuring a consistent user experience across different browsers.

#### Runtime and Framework Independence

ASP.NET Core applications can run on the .NET Core runtime or, if necessary, on the full .NET Framework. This flexibility allows developers to choose the appropriate runtime for their application's requirements.

In summary, ASP.NET Core's cross-platform support is achieved through its foundation on the .NET Core runtime, the use of a lightweight and cross-platform web server (Kestrel), cross-platform development tools, and support for containerization. These features enable developers to build and deploy ASP.NET Core applications on a variety of operating systems and environments.

[↑ Back to Top](#table-of-contents)

## Razor Views and Static Files

### What are Tag Helpers in ASP.NET Core? Provide an example.

Tag Helpers are a feature in ASP.NET Core that enables server-side code to participate in creating and rendering HTML elements. They provide a more natural syntax to work with server-side logic directly within HTML markup, making the code more readable and maintainable.

#### Key Characteristics

- **Readable and intuitive syntax:** Tag Helpers use HTML-like syntax to make server-side logic easier to read and maintain in views.
- **Built-in Tag Helpers:** ASP.NET Core includes helpers for common tasks such as form creation and URL generation.

#### Extensibility

Developers can create custom Tag Helpers to encapsulate reusable components or simplify complex server-side logic within HTML. Custom Tag Helpers are implemented as C# classes.

#### Example of a built-in tag helper

Consider a scenario where you want to create a form in an ASP.NET Core view. The `<form>` Tag Helper can simplify the markup.

**Before using a Tag Helper:**

```html
<form method="post" action="/Home/SubmitForm">
    <!-- Form content goes here -->
    <input type="text" name="username" />
    <input type="password" name="password" />
    <button type="submit">Submit</button>
</form>
```

**With a Tag Helper:**

```html
<form asp-action="SubmitForm" method="post">
    <!-- Form content goes here -->
    <input type="text" asp-for="Username" />
    <input type="password" asp-for="Password" />
    <button type="submit">Submit</button>
</form>
```

#### Custom Tag Helper Example

Let’s imagine you want a custom Tag Helper to create a Bootstrap-styled alert.

```csharp
[HtmlTargetElement("bootstrap-alert")]
public class BootstrapAlertTagHelper : TagHelper
{
    public string AlertType { get; set; }

    public override void Process(
        TagHelperContext context,
        TagHelperOutput output)
    {
        output.TagName = "div";
        output.Attributes.SetAttribute("class", $"alert alert-{AlertType}");
        output.Content.SetHtmlContent(
            "This is a Bootstrap-styled alert.");
    }
}
```

#### Usage in a view

```html
<bootstrap-alert alert-type="success"></bootstrap-alert>
```

Tag Helpers provide a powerful way to blend server-side logic seamlessly into HTML, resulting in cleaner and more maintainable views.

### What is the purpose of the `wwwroot` folder in an ASP.NET Core project?

The `wwwroot` folder in an ASP.NET Core project stores web assets and static files. Its purpose includes:

#### Publicly Accessible Content

The `wwwroot` folder is the default location for static files intended to be publicly accessible over the web. These include client-side assets such as HTML, CSS, JavaScript, images, and fonts that the browser can download directly.

#### Static File Middleware

ASP.NET Core includes Static File Middleware that serves files from `wwwroot` directly to clients without additional processing. This middleware is configured automatically in the default project templates.

#### Default Root for Web Assets

When a client requests a static file, Static File Middleware looks for it in `wwwroot` by default. This provides an organized way to structure web assets and distinguish them from server-side code.

#### Client-Side Resources

Web assets stored in `wwwroot` can be referenced directly in HTML or Razor views. For example, if you have a CSS file at `wwwroot/css/style.css`, reference it like this:

```html
<link rel="stylesheet" href="/css/style.css" />
```

#### Security Considerations

Files within `wwwroot` are meant to be publicly accessible, so sensitive information or files that should not be exposed publicly should not be placed there. Security measures should be taken to protect sensitive data.

#### Default Location for Client-Side Libraries

Client-side libraries, such as JavaScript or CSS frameworks, are often placed in `wwwroot/lib` to indicate they are intended for client-side use and to make them easy to reference in views.

In summary, `wwwroot` is the designated location for static files in an ASP.NET Core project. It provides a central repository for client-side assets that clients can access over the web, supporting simplicity, organization, and easy access to static content.

[↑ Back to Top](#table-of-contents)
