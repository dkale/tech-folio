# ASP.NET Core - CORS

## What Is the Same-Origin Policy?

Two URLs have the same origin when they use the same URL scheme, hostname (domain name), and port number.

Examples of scheme, host, or port differences include:

- HTTP versus HTTPS
- `example.com` versus `example.net`
- Different application ports

The same-origin policy is a browser security control that limits communication between documents from different origins. For example, a client application hosted at `https://example.com` cannot communicate directly with a server application hosted at `https://example.net` unless cross-origin access is explicitly permitted.

This policy isolates potentially malicious scripts and prevents them from interacting with documents or resources from unrelated origins.

## What Is Cross-Origin Resource Sharing?

Cross-Domain scripting is sometimes required, and the open web makes cross-origin communication useful in many applications.

**Cross-Origin Resource Sharing (CORS)** is a mechanism that allows a server to grant specific external clients permission to communicate across origins. The server does so by adding CORS-related HTTP response headers that tell the browser which origins, methods, and headers are allowed.

For a complete article on configuring CORS in ASP.NET Core, see [Enabling CORS in ASP.NET Core](https://code-maze.com/enabling-cors-in-asp-net-core/).
