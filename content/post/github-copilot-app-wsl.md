---
layout: post
title: "Enabling WSL Support for the GitHub Copilot App"
date: 2026-08-27
categories: ["Software Development"]
author: "Michael S. Collier"
tags: [AI, agents, github, github-copilot]
comments: true
---
 
I've been intrigued by the [GitHub Copilot App](https://gh.io/app) ever since it first released at Build a few months ago.  It's a different way of working for me, but it's one that I'm getting more and more comfortable with and really enjoying.  However, one thing that held me back a bit was its lack of support for working with repos in WSL. That has now changed.

<!--more-->

I do the vast majority of my personal and professional work in WSL2 using dev containers. I find that setup is usually very efficient and gives me the performance and sandbox environment that I desire. While this setup works great for me, it didn't work with with the GitHub Copilot App.  With the GitHub Copilot App installed on Windows, the app didn't work with repos in WSL.

## Enabling WSL Support

Starting with v1.1.4, the GitHub Copilot App includes an "preview" experimental feature to connect to and create sessions inside WSL. Your first step will be to enable the "WSL hosts (Preview)" feature in the Experimental settings of the app. You'll need to restart the app after doing so.

![Enable WSL host preview in GitHub Copilot experimental settings](/images/github-copilot-app-wsl/wsl-experimental-enable-wsl-host.png)

> It's experimental _and_ preview . . . is that another way of saying "this may not work just right yet, but try it out and us know what you think"? 😉

After restarting, return to the Settings menu in the GitHub Copilot App.  You'll now notice a new Environments section (immediately below Experimental).  Your next step will be to connect to one of the detected WSL distribution on your machine. I run Ubuntu on WSL2, so I clicked the little **+** button to add that distribution to the app.

![Selecting a WSL host in the GitHub Copilot App](/images/github-copilot-app-wsl/github-copilot-app-wsl-hosts.png)

Now that I've connected the app to my Ubuntu instance, it's time to add the projects. Click the **+ Add projects** button to open a small dialogue to browse to the folder on my WSL instance where the project is that I want to add to GitHub Copilot App.  Select the folder and then the green **Use this folder** button.  The app should validate the folder and display a "Path looks good. Will register on host." message.  Finally, click the green "Add project" button to finish adding the project folder.

![Adding projects from the WSL host in the GitHub Copilot App](/images/github-copilot-app-wsl/github-copilot-app-add-project.png)

At that point, the project should get added to the list of projects on the left side of the app.

## It works . . . mostly

I've had a relatively good experience so far with the new WSL support.  It is "experimental" and "preview", and I did notice a few bugs.

### Unable to select a branch to start a new session

I did notice that I'm unable to select a branch when starting a new session.  For repos hosted on Windows, I can click the **Create from** branch button next to the project name to open a dialogue from which I can pick a branch for a new session.  For repos on WSL, clicking **Create from** did nothing.  See [https://github.com/github/app/issues/3246](https://github.com/github/app/issues/3246).

### Can't add projects of the same name

I had a few projects which I'd previously cloned in Windows (I know . . . violating my earlier rule).  I cloned the repo in WSL and added it to the GitHub Copilot App.  However, even though the app indicated the project was registered, nothing happened. A new project didn't get added to the project list.  There was no warning or other indicator that I may be replacing an existing project or unable to add the project because one of the same name existed.  There was nothing.  See [https://github.com/github/app/issues/3247](https://github.com/github/app/issues/3247)

## Summary

I'm using the GitHub Copilot App more often.  The lack of support for WSL-hosted projects was holding me back.  Starting in GitHub Copilot App v1.1.4, a new experimental "preview" feature enables support for WSL-hosted projects.  Despite a couple of bugs, the new feature seems to be working well.  I'm looking forward to continuing to use this new functionality in the GitHub Copilot App.
