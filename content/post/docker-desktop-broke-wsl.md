---
layout: post
title: "Docker Desktop 4.75.0 Broke My WSL"
date: 2026-06-04
categories: [ "docker" ]
author: "Michael S. Collier"
tags: [docker, docker-desktop, wsl]
comments: true
---

I try to keep my Docker Desktop (on my personal dev machine) up-to-date.  It updates frequently and I'm generally a fan of keeping my dev software on the latest version.  So when I noticed an available update for Docker Desktop, I didn't hesitate to update.  That was a mistake - 4.75.0 broke my WSL integration.

<!--more-->

## The Problem

I tend to do all my development in WSL2 and dev containers.  Thus, Docker (more so the Docker CLI) is a key part of my environment. After updating to Docker Desktop 4.75.0 I started to get this error:

``` text
failed to connect to the docker API at unix:///var/run/docker.sock; check if the path is correct 
and if the daemon is running: dial unix /var/run/docker.sock: connect: no such file or directory
```

![What just happened](https://media2.giphy.com/media/v1.Y2lkPTc5MGI3NjExdzdkaHFzeDdkdGs5dzVjMW5iemN1cnRldmF2eXJiZWY0MWZ3ZmpuZiZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/WuS5qFGLQKp2M/giphy.gif)

Just fantastic!  I restarted Docker Desktop - same problem.  I rebooted - same problem.  I uninstalled Docker Desktop, rebooted and installed Docker Desktop - same problem. :angry:

After questioning some life choices, I finally found the right thread which led me to a solution. A [GitHub issue](https://github.com/docker/for-win/issues/14686#issuecomment-4550664184) pointed me to the [Docker Desktop release notes](https://docs.docker.com/desktop/release-notes/#for-windows-1) which contained a subtle statement under "Bug Fixes and Enhancements":

``` text
WSL integration with the default distribution has been disabled. To change this, visit Settings.
```

I'm not sure how that is a "bug fix" or an "enhancement", but sure . . . whatever.

## The Fix

The fix is relatively simple.  Within Docker Desktop, navigate to **Settings** and then the **Resources** section.  The **Enable integration with my default WSL distro** needs to be checked (it was unchecked after the update).

![Docker Desktop WSL integration option checked](/images/docker-desktop-broke-wsl/docker-desktop-wsl-integration-markup.png)

Once I did that, I was back in business!
