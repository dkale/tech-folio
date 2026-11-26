# ASP.NET Core - Exception Handling

## Table of Contents

- [Centralized Exception Handling](#centralized-exception-handling)
- [Handling Errors with the Built-In Middleware](#handling-errors-with-the-built-in-middleware)
  - [Create an ErrorDetails Model](#create-an-errordetails-model)
  - [Create an Exception Handler Extension](#create-an-exception-handler-extension)
  - [Register the Handler in Startup](#register-the-handler-in-startup)
- [Handling Errors with Custom Middleware](#handling-errors-with-custom-middleware)
  - [Create the Custom Middleware](#create-the-custom-middleware)
  - [Create the Middleware Extension](#create-the-middleware-extension)
  - [Register the Custom Middleware](#register-the-custom-middleware)
- [Conclusion](#conclusion)

## Centralized Exception Handling

Exception handling logic can be extracted into a centralized location instead of repeating `try-catch` blocks in every API action.

Centralized handling provides several benefits:

- API actions remain shorter and more readable.
- Exception-handling behavior is easier to maintain.
- The same behavior can be reused across the entire project.
- Endpoint code can focus on the requested operation instead of repeated error handling.

Handling exceptions separately in every API endpoint requires repeating `try-catch` blocks in every action that catches unhandled exceptions.

For a complete article, see [Global Error Handling in ASP.NET Core](https://code-maze.com/global-error-handling-aspnetcore/).

[↑ Back to Top](#table-of-contents)

## Handling Errors with the Built-In Middleware

The `UseExceptionHandler` middleware is a built-in middleware that can handle exceptions in an ASP.NET Core Web API application.

### Create an ErrorDetails Model

Create a new `ErrorDetails` class in the `Models` folder:

```csharp
using System.Text.Json;

namespace GlobalErrorHandling.Models
{
    public class ErrorDetails
    {
        public int StatusCode { get; set; }
        public string Message { get; set; }

        public override string ToString()
        {
            return JsonSerializer.Serialize(this);
        }
    }
}
```

This class represents the details included in the JSON error response.

### Create an Exception Handler Extension

Create a folder named `Extensions` and a static class named `ExceptionMiddlewareExtensions.cs` inside it.

```csharp
using GlobalErrorHandling.Models;
using LoggerService;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Diagnostics;
using Microsoft.AspNetCore.Http;
using System.Net;

namespace GlobalErrorHandling.Extensions
{
    public static class ExceptionMiddlewareExtensions
    {
        public static void ConfigureExceptionHandler(
            this IApplicationBuilder app,
            ILoggerManager logger)
        {
            app.UseExceptionHandler(appError =>
            {
                appError.Run(async context =>
                {
                    context.Response.StatusCode =
                        (int)HttpStatusCode.InternalServerError;
                    context.Response.ContentType =
                        "application/json";

                    var contextFeature =
                        context.Features
                            .Get<IExceptionHandlerFeature>();

                    if (contextFeature != null)
                    {
                        logger.LogError(
                            $"Something went wrong: {contextFeature.Error}");

                        await context.Response.WriteAsync(
                            new ErrorDetails
                            {
                                StatusCode =
                                    context.Response.StatusCode,
                                Message =
                                    "Internal Server Error."
                            }.ToString());
                    }
                });
            });
        }
    }
}
```

The extension method registers `UseExceptionHandler`, sets the response status code and content type, logs the exception, and returns a custom `ErrorDetails` JSON response.

### Register the Handler in Startup

For a .NET 5 project, modify the `Configure` method in `Startup.cs`:

```csharp
public void Configure(
    IApplicationBuilder app,
    IWebHostEnvironment env,
    ILoggerManager logger)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

    app.ConfigureExceptionHandler(logger);

    app.UseHttpsRedirection();
    app.UseRouting();
    app.UseAuthorization();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
    });
}
```

For .NET 6 and later projects, register the handler after building the application:

```csharp
var app = builder.Build();

var logger =
    app.Services.GetRequiredService<ILoggerManager>();

app.ConfigureExceptionHandler(logger);
```

The endpoint or action no longer needs a repeated `try-catch` block. This makes the action cleaner and allows the exception-handling behavior to be reused throughout the project.

[↑ Back to Top](#table-of-contents)

## Handling Errors with Custom Middleware

A custom middleware component can also centralize exception handling.

### Create the Custom Middleware

Create a folder named `CustomExceptionMiddleware` and add a class named `ExceptionMiddleware.cs`.

```csharp
public class ExceptionMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILoggerManager _logger;

    public ExceptionMiddleware(
        RequestDelegate next,
        ILoggerManager logger)
    {
        _logger = logger;
        _next = next;
    }

    public async Task InvokeAsync(HttpContext httpContext)
    {
        try
        {
            await _next(httpContext);
        }
        catch (Exception ex)
        {
            _logger.LogError(
                $"Something went wrong: {ex}");

            await HandleExceptionAsync(
                httpContext,
                ex);
        }
    }

    private async Task HandleExceptionAsync(
        HttpContext context,
        Exception exception)
    {
        context.Response.ContentType =
            "application/json";
        context.Response.StatusCode =
            (int)HttpStatusCode.InternalServerError;

        await context.Response.WriteAsync(
            new ErrorDetails
            {
                StatusCode =
                    context.Response.StatusCode,
                Message =
                    "Internal Server Error from the custom middleware."
            }.ToString());
    }
}
```

The `ILoggerManager` service and `RequestDelegate` must be available for dependency injection.

- `ILoggerManager` provides the logging service.
- `RequestDelegate` is a function delegate that processes an HTTP request.

The `InvokeAsync()` method is required for `RequestDelegate`-based middleware to process requests.

If the next middleware and controller action complete successfully, a normal response is returned. If an exception occurs, the `catch` block logs the exception and invokes `HandleExceptionAsync`.

The exception handler sets:

- The response content type to `application/json`.
- The response status code to `500 Internal Server Error`.
- A custom JSON error response.

### Create the Middleware Extension

Add another static method to `ExceptionMiddlewareExtensions`:

```csharp
public static void ConfigureCustomExceptionMiddleware(
    this IApplicationBuilder app)
{
    app.UseMiddleware<ExceptionMiddleware>();
}
```

For .NET 6 and later projects, extend `WebApplication` instead:

```csharp
public static void ConfigureCustomExceptionMiddleware(
    this WebApplication app)
{
    app.UseMiddleware<ExceptionMiddleware>();
}
```

### Register the Custom Middleware

In the `Configure` method of a .NET 5 `Startup` class, leave the built-in handler commented or remove it, and add the custom middleware:

```csharp
// app.ConfigureExceptionHandler(logger);
app.ConfigureCustomExceptionMiddleware();
```

The custom middleware is now implemented and can provide centralized exception handling across the application.

[↑ Back to Top](#table-of-contents)

## Conclusion

The application now has centralized, reusable exception-handling logic. This avoids repeated `try-catch` blocks, makes controller actions more readable, and keeps global error handling consistent throughout the project.

[↑ Back to Top](#table-of-contents)
