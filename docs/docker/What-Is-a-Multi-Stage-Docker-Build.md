# What Is a Multi-Stage Docker Build?

Multi-stage build means:

> Use one Docker image to build the app, and another smaller image to run the app.

Only the final, small image is kept.

## Table of Contents

- [A Simple Analogy: Making Tea](#a-simple-analogy-making-tea)
- [Why Do We Need It?](#why-do-we-need-it)
- [Simple Example (.NET)](#simple-example-net)
- [What Docker Does Automatically](#what-docker-does-automatically)
- [Benefits](#benefits)
- [One-Line Summary](#one-line-summary)
- [Further Exploration](#further-exploration)

---

## A Simple Analogy: Making Tea

Imagine making tea ☕:
1. You use:
- Stove
- Gas
- Big utensils
2. But when you serve tea to someone:
- You only give the cup of tea
- You don’t give the stove and gas
Multi-stage build works the same way:
- **Build stage** = kitchen (tools needed to prepare)
- **Run stage** = tea cup (only the final result)

[↑ Back to Top](#table-of-contents)

## Why Do We Need It?

When building apps (like .NET, Java, Node.js):
- We need heavy tools (SDK, compiler, npm, Maven, etc.) to build
- But we don’t need those tools to run the app
If we include everything:
- Image becomes very large
- Slower to download
- Less secure

[↑ Back to Top](#table-of-contents)

## Simple Example (.NET)

### Step 1: Build the App (Needs SDK)

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY . .
RUN dotnet publish -c Release -o /app
```

This stage:

- Compiles the code
- Produces final files (`.dll`)

### Step 2: Run the App (No SDK Needed)

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app .
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

This stage:

- Takes only the compiled output
- Runs the app
- No build tools included

[↑ Back to Top](#table-of-contents)

## What Docker Does Automatically

- Builds both stages
- Copies files from build stage to run stage
- Throws away the build stage
- Final image = only runtime + app

[↑ Back to Top](#table-of-contents)

## Benefits

- Smaller image
- Faster download & deploy
- More secure (fewer tools inside)
- Cleaner Dockerfile

[↑ Back to Top](#table-of-contents)

## One-Line Summary

> Multi-stage build is a Docker optimization where you build the app using big tools, then run it using a small clean image by keeping only the final output.

[↑ Back to Top](#table-of-contents)

## Further Exploration

The source document ends with these suggested follow-ups:
- Explain with a Node.js example
- Draw an ASCII diagram
- Compare single-stage vs multi-stage visually
Just tell me 👍

[↑ Back to Top](#table-of-contents)
