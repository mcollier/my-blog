---
layout: post
title: "Developing with ARM-based Surface Laptop Copilot+PC"
author: "Michael S. Collier"
date: 2025-07-22
tags: [dotnet, semantic-kernel, ai]
---

Like many in the Microsoft ecosystem, I was excited to get my hands on one of the new ARM-based Surface Laptop Copilot+ PCs. As a developer focused on .NET and Azure, I was eager to see how well this sleek new machine could handle my day-to-day workflow.

While performance and battery life have impressed me so far, I’ve run into a few bumps in the road when it comes to local development—especially in areas that rely on platform-specific tooling or containerized environments. In this post, I’ll highlight a couple early challenges I’ve encountered and what I did to work around them. I’ll update this post as I learn more and adapt my setup.

## :bulb: Why ARM64

First, a quick note on the setup. The Surface Laptop Copilot+ PC is powered by Qualcomm’s Snapdragon X Elite processor, which means it runs Windows on ARM64. For most day-to-day usage, this isn’t a problem—thanks to x64 emulation improvements in Windows 11, a lot of apps “just work.”

But when it comes to software development, especially anything involving Docker, native tooling, or architecture-sensitive packages, the details really matter.

## :construction: Challenge 1: Docker Desktop Didn’t Work (At First)

One of the first roadblocks I hit was Docker. I use Docker heavily in my development environment—whether for running SQL Server containers, API services, or GitHub Codespaces-style dev containers.

Initially, I installed Docker Desktop using my usual Winget configuration file (which I use to quickly configure new machines). But I kept running into runtime issues. Containers wouldn’t start, and I was getting cryptic errors I hadn’t seen before.

Turns out: I had mistakenly installed the standard x64 version of Docker Desktop—not the version built for Windows ARM64.

✅ Fix: Use the ARM64 version of Docker Desktop. You can install it explicitly using:

TODO

## :gear: Challenge 2: Azure Functions Core Tools on ARM64

Another surprise came when I tried to run Azure Functions locally using Azure Functions Core Tools.

When developing in a GitHub Codespace (which is Linux-based), everything worked great—including using a dev container setup for my .NET 8 Functions project. But when I cloned the repo and tried to run the same setup locally inside WSL or even directly in Windows, I hit a wall.

The issue?
The func CLI tools (Azure Functions Core Tools) have limited or no stable support for Linux ARM64. Specifically:

Running inside WSL2 (Ubuntu ARM64) caused runtime errors or missing dependencies.

The stable NPM package (azure-functions-core-tools) didn’t install cleanly due to native bindings.

Dev containers worked in Codespaces, likely due to a hosted x64 Linux container runtime under the hood.

But locally, the ARM64 dev container image failed to start (or build)—likely due to [insert more detail once confirmed].

🧪 Workaround: Use the preview version of Azure Functions Core Tools, which offers limited support for ARM64. Install it via:

```bash
npm i -g azure-functions-core-tools@4 --unsafe-perm true
```

That got me unblocked, though I’m still testing to see if everything works as expected (for example, Durable Functions, blob triggers, etc.).

## :white_check_mark: What’s Working Well

Despite these early hiccups, a lot is working really well:

.NET SDKs run great on ARM64—no issues compiling or running apps.

Visual Studio Code (ARM64) is fast and stable, with extensions working as expected.

GitHub Codespaces and Dev Containers work well remotely—and might be a good fallback strategy if local tooling proves unreliable.

Battery life and performance are seriously impressive.

## :thought_balloon: Final Thoughts (For Now)

ARM64 on Windows is getting better—but it’s not frictionless yet, especially for cloud-native and cross-platform developers. I’m optimistic, though. Each of these hurdles has had a workaround, and I expect the tooling will continue to mature rapidly now that Copilot+ PCs are on the market.

If you’re setting up a Surface Laptop Copilot+ for development—especially in the Azure/.NET world—I hope this post gives you a head start. And if you’ve encountered (or solved!) other challenges, I’d love to hear about them.

:point_right: [Reach out on Twitter/Threads/LinkedIn] and let’s compare notes.
