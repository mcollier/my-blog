---
layout: post
title: "Dotnet Run App"
date: 2025-05-30
author: "Michael S. Collier"
tags: [dotnet]
draft: true
---

Have you ever had a moment of inspiration, an idea you wanted to test in C#, but didn’t want to spin up a full project just to run a few lines of code? Same here.

Well, with [.NET 10 Preview 4](https://devblogs.microsoft.com/dotnet/announcing-dotnet-run-app-cs/), that pain is going away. Say hello to the new `dotnet run app.cs` feature.

It’s like C# finally took a page from Python and JavaScript, languages that have long made it easy to run a file with a single command. Now C# joins the party. :tada:

## Try It Out: Instant Console Apps

Let's start simple. We'll create a new _file-based app_ with a little file called `Champions.cs`:

```csharp
Console.Writeline("2025 National Champions . . . The Ohio State Buckeyes!");
```

Just run it with:

```bash
dotnet run Champions.cs
```

And that’s it. No `Program.cs`, no `.csproj` files, no boilerplate. Just your code, running instantly.

## Go Bigger: Run a Minimal API from a Single File

Here's where it gets _really_ cool. You can now run a full **ASP.NET Minimal API** from a single .cs file too. Check this out:

```csharp
#:sdk Microsoft.NET.Sdk.Web

using Microsoft.AspNetCore.Builder;
using Microsoft.Extensions.Hosting;

var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

string[] bigTenTeams = new[]
{
    "Illinois",
    "Indiana",
    "Iowa",
    "Maryland",
    "Michigan",
    "Michigan State",
    "Minnesota",
    "Nebraska",
    "Northwestern",
    "Ohio State",
    "Oregon"
    "Penn State",
    "Purdue",
    "Rutgers",
    "UCLA",
    "USC",
    "Washington",
    "Wisconsin"
};

app.MapGet("/teams", () => bigTenTeams);

app.Run();
```

Save this as `MyApi.cs`, and run:

```bash
dotnet run MyApi.cs
```

Boom :boom: — you’ve got a running local API. No scaffolding, no folders, just fast feedback and working code.

## Shell Scripts

I'm not great at writing bash scripts. Truth be told, I rely on my trusty friend, GitHub Copilot, to help write bash scripts.  With file-based apps in C#, I can write cross-platform C# shell scripts! :boom:

```shell
#!/usr/bin/dotnet run

Console.WriteLine("Hello, Champs!");
```

## Growing Up

When the time comes to "grow up" to a full C# project, you can do that with the `dotnet project convert` command. Nice!

## Why This Matters

This change makes C# a much more approachable language for:

- Rapid prototyping
- Teaching and learning
- Sharing quick samples
- Running minimal tools and APIs on the fly

And honestly… with features like this, I may never need to write another shell script again. (Half joking. Probably. :grin:)

## Ready to Dive In?

Try it out, share your experiments, and let me know what you build! I’m especially curious if you’ve found creative ways to use this in your day-to-day development.

> Check out my _very simple_ sample repo at [https://github.com/mcollier/dotnet-run-demo](https://github.com/mcollier/dotnet-run-demo) (used to help write this post).

You can read more in the official .NET blog post:
:point_right: [Announcing dotnet run app.cs - A simpler way to start with C# and .NET 10](https://devblogs.microsoft.com/dotnet/announcing-dotnet-run-app-cs/)

{{< youtube 98MizuB7i-w >}}
