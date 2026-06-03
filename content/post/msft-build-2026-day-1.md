---
layout: post
title: "Microsoft Build 2026 - Day 1"
date: 2026-06-02
categories:
author: "Michael S. Collier"
tags: [AI, agents, microsoft-build]
comments: true
---

I was fortunate to attend the Microsoft Build conference in person again this year. Today, June 2nd, was the first day of the event.  I’d like to share my observations from today.  I’ll try to post a similar update after day two, and a wrap-up post as well.

<!--more-->

![Microsoft Build 2026](/images/msft-build-2026-day-1/build-1.jpg)

## The Venue

Build is smaller this year than in previous years – around 2,700 attendees listed in the attendee directory.  The venue, Fort Mason in San Franscisco, is also significantly smaller than the Seattle Convention Center, home of last year’s Build conference.

![Alcatrez](/images/msft-build-2026-day-1/alcatrez-2.jpg)

Fort Mason itself is cool.  One can see Alcatraz and the Golden Gate Bridge from the piers.  Build is making use of several buildings, with the keynote, many breakouts, table talks, and theater / demo sessions being in the Gateway and Festival Pavilions, while a few breakouts and labs seem to be in buildings A – D.  The buildings are relatively close, so it’s relatively easy to move between them.  Microsoft did a nice job with having social areas and some light refreshments outside.

However, while Fort Mason itself was cool, the other logistics left much to be desired in my opinion.  With so much being outside, I’m thankful the weather was good.  It would have been less than awesome if it had been raining.

The venue set up inside the Gateway and Festival Pavilions was not conducive to a event of Build’s size (even if smaller this year) or content.  It was very hard to hear in speakers in the breakout rooms – so much so that Microsoft handed out headphones to people.  Theater sessions were cram packed – again, couldn’t hear the speaker and the screens were really small and difficult to see the content if more than a few rows away (either that or I have worse hearing and eyesight than I thought).  The table talk sessions were a joke – only 5-7 people could get into each.

I’m all for a slightly smaller event with more opportunities to interact with attendees, presenters, and sponsors, but this setup didn’t work.

## The Content

Issues with the venue logistics aside, I really enjoyed the day 1 content overall.  The theme from day 1 seemed to focus on some really interesting developer hardware, questionable consumer hardware, and ways to make AI agents production / enterprise grade.

### Keynote

The most exciting things for me from the keynote were:

- The new [Microsoft Surface RTX DevBo](https://www.microsoft.com/en-us/surface/devices/surface-rtx-spark-dev-box). No doubt, the hardware is impressive!  The form factor is a bit unconventional but looks really nice in person.  I’m game!  No word on pricing, but I’m expecting it to be quite expensive. ![Surface RTX Spark DevBox](/images/msft-build-2026-day-1/surface-devbox-1.jpg)
- [Windows Developer Config](https://github.com/microsoft/WindowsDeveloperConfig) looks great! I love the easy, getting started setup, theming, and Homebrew support.  I’m looking forward to giving this a try.
- OpenClaw for Windows. I haven’t gotten into OpenClaw, yet.  I’m intrigued though.  I feel a bit more comfortable with some of the safety controls Microsoft unveiled for OpenClaw on Windows.  I’m looking forward to learning more. ![](/images/msft-build-2026-day-1/openclaw-1.jpg)
- The [GitHub Copilot App](https://aka.ms/GitHubCopilotApp) looks really nice!  I like the idea of driving multiple sessions from the app, getting more done with less context switching.  Cassidy Williams did a great job showing off the app and making it fun. ![GitHub Copilot App](/images/msft-build-2026-day-1/github-copilot-app-1.jpg)
- 7 new Microsoft AI Models.  I’m not surprised Microsoft is pushing their own AI models more.  I’m really looking forward to giving these a try, especially MAI Code-1-Flash and MAI Thinking-1. ![MAI Models](/images/msft-build-2026-day-1/mai-1.jpg)

#### Project Solara and AI Hardware?

I’m less convinced about Project Solara and Microsoft making yet another consumer hardware push.  You’ll not find a bigger Microsoft fanboy than me – I’ve had nearly all Microsoft hardware at some point (Windows Phone, Zune, Band, Xbox, Surface Laptops, etc.)  The only ones that are still anything – Xbox and Surface Laptop.  For whatever reason, Microsoft doesn’t have a good track record of consumer hardware. The Project Solera prototype hardware didn’t impress me; nothing made me go “gotta have it”.  The desktop-like device looked like a cheaper Amazon Echo and I’m still not sure why I need that thing.  The keycard device could be neat . . . I guess . . . but I was wondering why I need that instead of using my iPhone or Apple Watch.  Maybe Project Solera will get better as the hardware and associated software evolves.  Or, maybe we won’t be talking about this at Build 2027.

![Project Solara](/images/msft-build-2026-day-1/project-solara-1.jpg)

### Breakouts

I was able to attend three breakout sessions today and a few theater sessions.  Well, I’ll say I partially attended the theater sessions, because of the aforementioned problems with seating and A/V.
The theme for the breakouts, even for the ones I didn’t attend in person (but will be catching up with via the recordings), was definitely about how to make AI agents ready for production.  Last year, Build was about showing off what’s possible with AI and the early days of agents.  This year, it’s about how to manage key capabilities to make AI agents truly viable long term in an organization – security, scalability, managing costs, and observability.  At the heart of all of this sits Microsoft Foundry.  It’s clear that Microsoft is positioning Foundry as the center of their AI ecosystem.

As for the sessions I attended:

- [Why your AI code doesn’t ship](https://build.microsoft.com/en-US/sessions/BRK200): Closing the gap to production. This session was mostly about the new GitHub Copilot App and how to use it to be more productive.  Definitely a few tips and techniques I’m looking forward to applying.
- [Deploy. Observe. Learn. Reinforcement learning for production agents](https://build.microsoft.com/en-US/sessions/BRK231).  This was the most interesting session I attended.  I hadn’t done anything with fine tuning a model, mostly because I wasn’t sure why I needed it and concerns over cost in doing so. I may have been incorrect.  The presenters did an excellent job in showing why and how to tune a model, centering on how the economics of the frontier models make tuning using distillation and other techniques more of a necessity now.  I’m definitely going to watch this again and try the approaches presented.  Of course, all this is possible because of features in Microsoft Foundry.
- [From observability to ROI for AI agents on any framework](https://build.microsoft.com/en-US/sessions/BRK252). This was another Microsoft Foundry session, and the presenters did a great job.  I’m looking forward to diving more into the logs and evaluation features in Foundry, including the new Rubric evaluation functionality.  Foundry Toolkit, with the included MCP server and skills are also something to get familiar with quickly.  The presenters also showed how to use the Azure Developer CLI’s `azd ai agent optimize` command to tune system prompt, skills, tools, and model selection.  Finally, they demonstrated the new ROI evaluation capabilities in Foundry (currently in private preview).  With the ROI evaluation, one can use token and tool costs to calculate the ROI for an agent over different versions of the agent.  I think that can be really powerful – helping teams to determine the right selection of components that will provide meaningful business value, not just something that is cool or fun.
