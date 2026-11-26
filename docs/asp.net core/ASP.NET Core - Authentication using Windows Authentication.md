# ASP.NET Core - Authentication using Windows Authentication

## Table of Contents

- No section headings found.

**ASP.NET Core - Authentication using Windows Authentication**
```csharp
Using Windows authentication makes sense when you are building an **internal company website** (an **intranet site**) and you want your users to be able to use their standard Windows usernames and passwords when accessing the website. If you are building an **outwards facing website** (an **Internet website**) consider using **Forms authentication** instead.
```
**Enabling Windows Authentication**
When creating a new ASP.NET MVC application, Windows authentication is not enabled by default. Forms authentication is the default authentication type enabled for MVC applications. You must enable Windows authentication by modifying your MVC application’s web configuration (web.config) file. Find the <authentication> section and modify it to use Windows instead of Forms authentication like this:
<authentication mode="Windows">
</authentication>
When you enable Windows authentication, your server becomes responsible for authenticating users. Typically, there are two different types of web servers that you use when creating and deploying an ASP.NET MVC application.
While developing an MVC application, you use the ASP.NET Development Web Server included with Visual Studio. By default, the ASP.NET Development Web Server executes all pages in the context of the current Windows account(whatever account you used to log into Windows).
ASP.NET Development Web Server also supports NTLM authentication.
Steps to enable NTLM authentication: Right-click project name in VS > Properties > Web Tab > Check the NTLM checkbox.
For production web applications, on the other hand, you use IIS as your web server, IIS supports several types of authentication.

[↑ Back to Top](#table-of-contents)