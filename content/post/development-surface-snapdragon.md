---
layout: post
title: "Developing with ARM-based Surface Laptop Copilot+PC"
author: "Michael S. Collier"
date: 2025-07-23
tags: [Windows, ARM, WSL2, Dev Containers, Azure Functions]
---

This week I was excited to _finally_ get my hands on a new PC for my personal use.  I purchased one of the [new ARM-based Surface Laptop Copilot+ PCs](https://www.microsoft.com/en-us/store/configure/surface-laptop-15-inch/8mzbmmcjzqc4). As a developer focused on .NET and Azure, I was eager to see how well this sleek new machine could handle my day-to-day workflow.

While performance and battery life have impressed me so far, I’ve run into a few bumps in the road when it comes to local development, especially in areas that rely on platform-specific tooling or containerized environments. In this post, I’ll highlight a couple of early challenges I’ve encountered and what I did to work around them. I’ll update this post as I learn more and adapt my setup.

## :bulb: Why ARM64

First, a quick note on the setup. The Surface Laptop Copilot+ PC is powered by Qualcomm’s Snapdragon X Elite processor, which means it runs Windows on ARM64. For most day-to-day usage, this isn’t a problem in Windows 11. Many of my apps “just work.”

But when it comes to software development, I've had a few relatively minor challenges.

## :construction: Challenge 1: Docker Desktop Didn’t Work (At First)

One of the first roadblocks I hit was Docker. I use Docker heavily in my development environment. The first thing I set up on any new project is a Visual Studio Code dev container. I love the ability to have a known-good environment that I can use locally or in a GitHub Codespace.

Initially, I installed [Docker Desktop](https://www.docker.com/products/docker-desktop/) using my usual [winget](https://learn.microsoft.com/en-us/windows/package-manager/winget/) configuration file (which I use to quickly configure new machines). But I kept running into runtime issues. While Docker Desktop did install, it continually failed to start, and I was getting errors I hadn't seen before - "wsl.exe --mount on ARM64 requires Windows version 27653 or newer."

![Docker Desktop error](/images/development-surface-snapdragon/docker-desktop-arm64-wsl-error-sm.png)

Even after applying all the Windows updates, my new PC is running Windows 11 version 24H2, OS Build 26100.4652.

Turns out I had installed the standard x64 version of Docker Desktop, not the version built for Windows ARM64.

✅ Fix: Use the ARM64 version of Docker Desktop.

## :gear: Challenge 2: Azure Functions Core Tools on ARM64

Another surprise came when I tried to install Azure Functions Core Tools in my dev container.  I've been using the Azure Functions Core Tools dev container feature from [https://github.com/jlaundry/devcontainer-features/tree/main/src/azure-functions-core-tools](https://github.com/jlaundry/devcontainer-features/tree/main/src/azure-functions-core-tools) for some time, and it almost always works flawlessly.  

```json
"features": {
    "ghcr.io/jlaundry/devcontainer-features/azure-functions-core-tools:1": {}
}
```

On my new PC, when using the dev container, I continually received errors when the Azure Functions Core Tools feature was being installed.

On a hunch, I tried running in a GitHub Codespace, using the same dev container configuration. It worked!  Dev containers worked in Codespaces, likely due to a hosted x64 Linux container runtime under the hood.

The issue?  The Azure Functions Core Tools currently has limited support for Linux ARM64. After much searching, I found this issue - [https://github.com/Azure/azure-functions-core-tools/issues/4279](https://github.com/Azure/azure-functions-core-tools/issues/4279)

:sparkles: Workaround: Use the preview version of Azure Functions Core Tools, which offers limited support for ARM64. Install it via:

```bash
npm install -g azure-functions-core-tools@4.0.7332-preview1
```

That got me unblocked!  I updated my dev container configuration to install node and the Azure Functions Core Tools (preview) via NPM.

## :white_check_mark: What’s Working Well

Despite these early hiccups, a lot is working really well:

- .NET SDKs run great on ARM64. No issues compiling or running apps.
- Visual Studio Code is wicked fast and stable, with extensions working as expected.
- GitHub Codespaces might be a good fallback strategy if local tooling proves unreliable.

## :thought_balloon: Final Thoughts (For Now)

ARM64 on Windows is really good, but it’s not frictionless yet. I’m optimistic, though. Each of these hurdles has had a workaround, and I expect the tooling will continue to mature rapidly now that Copilot+ PCs are on the market.

If you’re setting up a Surface Laptop Copilot+PC for development, especially in the Azure/.NET world, I hope this post gives you a head start. And if you’ve encountered (or solved!) other challenges, I’d love to hear about them.

:point_right: Reach out and let’s compare notes!
