---
layout: post
title: "Microsoft Build 2026 - Day 2"
date: 2026-06-04
categories: ["Microsoft Build"]
author: "Michael S. Collier"
tags: [AI, agents, microsoft-build]
comments: true
---
 
Microsoft Build is a two day event this year (after being three days in prior years).  That's in line with the overall smaller Build - fewer attendees, smaller venue, and fewer days.  There was no day 2 keynote either.  For me, day 2 was full of breakout sessions, a hands-on lab, and great conversations with Microsoft and GitHub employees.

<!--more-->

![Microsoft Build overview](/images/msft-build-2026-day-2/msft-build-high.jpg)

## No Day 2 Keynote

At a large conference like Microsoft Build, it's hard to keep track of, or at least have awareness of, all the announcements from the various product teams.  Microsoft is conference-driven, and as such many teams target conferences such as Build for their more significant updates and product announcements.  In prior years Build had a day 1 and day 2 keynote, where one of those was focused more on developers and associated product announcements.  I found that helpful as an awareness engine.

![Microsfot Build entrance](/images/msft-build-2026-day-2/msft-build-entrance.jpg)

With no day 2 keynote, it is a bit harder to be aware of all the announcements.  I'll definitely be catching up via the post-Build email blasts from Microsoft, and various product team blogs.

## Breakout sessions

I was able to attend several breakout sessions today.  Thankfully, most were in real rooms with good A/V, so it was easier to hear today than on day 1.

I attended the following sessions:

- [From CLI to PR: Automating the path to merged code](https://build.microsoft.com/en-US/sessions/BRK203): I loved this session!  The speakers were exceptional - sharing their knowledge on GitHub Copilot and the new GitHub Copilot App, strategies for using GitHub Copilot, along with an engaging and entertaining style.  This was mostly a live demo session, which worked very well.  They shared several good tips for GitHub Copilot.  One nugget I thought was interesting was that 60% of code it GitHub Copilot App is tests - that high degree of tests is the team's strategy for being effective with AI agents in software engineering.
- [Agent Supervision is the new Senior Engineer Skill](https://build.microsoft.com/en-US/sessions/BRK244):  Unfortunately the speaker was over 15 minutes late getting started (for a 45-minute session). But, he made up for it once he got going!  The session focused on the rapid pace of change and evolution in the coding models and how software engineering is changing.  The SDLC is dead - the IDE is dying/dead, RIP the PR, code reviews are dead, etc.  I'm not sure I buy into all that just yet, but I can understand the perspective.  Something to think more deeply about for sure.
- [Future of Developer Productivity: Microsoft’s EngThrive Framework in Practice](https://build.microsoft.com/en-US/sessions/BRK210): This was one of the better sessions I attended at Build.  I found it really interesting to learn how Microsoft's engineering teams have approached using AI and how they're metriced on outcomes/value instead of activity.  I'd had similar conversations in my firm about how to measure value and using AI engineering tools to deliver better outcomes.  I've also had conversations about "token maxing" and, in my opinion, the problems that presents.
- [Aspire for agents: Transform how you build and deploy distributed apps](https://build.microsoft.com/en-US/sessions/BRK205): As I state below, I'm a major fan of Aspire.  If I'm doing a .NET project (want to try with a Typescript project soon), it'll be using Aspire.  This was mostly an introductory session on the value of Aspire and the AppHost.  I still learned several things, mostly about the [new features in Aspire 13.4](https://aspire.dev/whats-new/aspire-13-4/), using `.withBrowserLogs()` to get the web browser's console logs into the Aspire dashboard, and the new Aspire agent skills.  Very cool stuff!

## Hands-On Lab

One of the really nice things about being at Build is the opportunity to try new tech in a relatively controlled manner, with experts ready to assist as needed.  Microsoft does a nice job with the lab setup - a controlled environment loaded with the necessary tech stack.  Easy to get going.

I joined the standby line (because I failed to register in advance to secure a spot) for the ["From data to context: Agent-ready knowledge with Foundry IQ"](https://build.microsoft.com/en-US/sessions/LAB532-R2).  Thankfully, several people failed to show up and I got in!

I had not done anything with any of the IQ workloads (of which there are now four - Fabric IQ, Work IQ, Foundry IQ, and, new at Build, Web IQ), so this was all new to me. :thumbsup:

The lab session started with a brief presentation on the various IQ workloads and then moved into the lab scenario. The lab itself was a series of Jupyter notebooks with pre-filled content.  That made it easy to "play" each cell to see the results.  A good approach to get familiar with the concepts, which is what I wanted at this point.

## Conversations

Another great perk of attending Build in person is the conversations you can have with Microsoft product group (PG) members.  They're always eager to hear how customers are using the products and any challenges.  I spent a good amount of time on day 2 talking with various PG members (so much that I missed a breakout session, but I'll catch up via the recording):

- **GitHub**: I spent about nearly an hour talking with two GitHub employees about a few challenges I was having with GitHub (lack of ability to select a GitHub Copilot policy when getting GitHub Copilot from multiple organizations, and problems with the new AI Credit billing model).  I really appreciated their time in listening to my feedback and providing me with references that I can follow-up on.
- **Durable Agents & Workflows**: I'm a huge fan of Azure Functions, the [Durable Task Scheduler](https://learn.microsoft.com/en-us/azure/durable-task/scheduler/durable-task-scheduler), and the [Durable Agent](https://techcommunity.microsoft.com/blog/appsonazureblog/bulletproof-agents-with-the-durable-task-extension-for-microsoft-agent-framework/4467122) and [Durable Workflow](https://devblogs.microsoft.com/dotnet/durable-workflows-in-microsoft-agent-framework/) extension support for [Microsoft Agent Framework](https://learn.microsoft.com/en-us/agent-framework/).  I had a great conversation with the team to share some early use feedback on Durable Agents and Durable Workflows.  I'm really excited about this stack and the direction it is heading!
- **Aspire**: I'm a big Aspire fan. However, I recently ran into a problem with the Service Bus emulator integration, specifically running the emulator on Linux ARM64 (using WSL on my Surface Laptop 7 with Snapdragon processor).  I was able to convey my scenario to the Aspire PMs, who were fantastic at understanding the scenario and trying to help me find a solution.  The solution may take a bit of time, but they're aware and working on it.  I appreciate that.

## Conclusion

Overall, I had a great time at Microsoft Build this year. I'm grateful that I was able to attend.  I'm looking forward to sharing more with my colleagues when I return home, and with the broader tech community via user groups, conferences, and this blog!
