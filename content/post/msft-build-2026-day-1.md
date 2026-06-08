---
layout: post
title: "Microsoft Build 2026 - Day 1"
date: 2026-06-02
categories: ["Microsoft Build"]
author: "Michael S. Collier"
tags: [AI, agents, microsoft-build]
comments: true
---
 
I attended Microsoft Build in person on June 2, 2026. Below are my observations from day one. I’ll post a day-two update, as well as a wrap-up after the event.

<!--more-->

![Microsoft Build 2026](/images/msft-build-2026-day-1/build-1.jpg)

## The Venue

Build was smaller this year — about 2,700 attendees. The venue, Fort Mason in San Francisco, is much smaller than the Seattle Convention Center.

![Alcatrez](/images/msft-build-2026-day-1/alcatrez-2.jpg)

Fort Mason is scenic: you can see Alcatraz and the Golden Gate Bridge from the piers. Microsoft used several buildings: the Gateway and Festival Pavilions hosted the keynote, many breakouts, table talks, and demos. A few breakouts and labs were in buildings A–D. The buildings are close, so it’s easy to move between them, and Microsoft provided social areas and light refreshments (which were very good).

Logistics fell short. With so much outdoors, the good weather helped. It would have been worse in the rain. Inside the pavilions the setup didn’t work for Build’s scale. Speakers in the breakout rooms were hard to hear - so much so that Microsoft handed out headphones. Theater sessions were packed and screens were small and hard to see past the front rows. Table talks only held five to seven people, which limited interaction.

I support a smaller, more interactive Build — but this layout missed the mark.

## The Content

Issues with the venue logistics aside, I really enjoyed the day 1 content.  The theme from day 1 focused on some really interesting developer hardware, questionable consumer hardware, and ways to make AI agents production / enterprise grade.

### Keynote

Highlights:

- The new [Microsoft Surface RTX DevBox](https://www.microsoft.com/en-us/surface/devices/surface-rtx-spark-dev-box). No doubt, the hardware is impressive!  The form factor is a bit unconventional but looks really nice in person.  I’m game!  No word on pricing, but I’m expecting it to be quite expensive. ![Surface RTX Spark DevBox](/images/msft-build-2026-day-1/surface-devbox-1.jpg)
> I was hoping there would be an announcment that all attendees would get a DevBox to take home, like Microsoft did a few times at the PDC events.  Not so much.
- [Windows Developer Config](https://github.com/microsoft/WindowsDeveloperConfig) looks great! I love the easy, getting started setup, theming, and Homebrew support.  I’m looking forward to giving this a try.
- OpenClaw for Windows. I haven’t gotten into OpenClaw, yet.  I’m intrigued though.  I feel a bit more comfortable with some of the safety controls Microsoft unveiled for OpenClaw on Windows.  I’m looking forward to learning more. ![OpenClaw on Windows](/images/msft-build-2026-day-1/openclaw-1.jpg)
- The [GitHub Copilot App](https://aka.ms/GitHubCopilotApp) looks really nice!  I like the idea of driving multiple sessions from the app, getting more done with less context switching.  Cassidy Williams did a great job showing off the app and making it fun. ![GitHub Copilot App](/images/msft-build-2026-day-1/github-copilot-app-1.jpg)
- 7 new Microsoft AI Models.  I’m not surprised Microsoft is pushing their own AI models more.  I’m really looking forward to giving these a try, especially MAI Code-1-Flash and MAI Thinking-1. ![MAI Models](/images/msft-build-2026-day-1/mai-1.jpg)

#### Project Solara and consumer hardware

I’m less convinced about Project Solara and Microsoft making yet another consumer hardware push.  You’ll not find a bigger Microsoft fanboy than me – I’ve had nearly all Microsoft hardware at some point (Windows Phone, Zune, Band, Xbox, Surface Laptops, etc.)  The only ones that are still anything – Xbox and Surface Laptop.  For whatever reason, Microsoft doesn’t have a good track record of consumer hardware. The Project Solara prototype hardware didn’t impress me; nothing made me go “gotta have it”.  The desktop-like device looked like a cheaper Amazon Echo and I’m still not sure why I need it.  The keycard device could be neat . . . I guess . . . but I was wondering why I need that instead of using my iPhone or Apple Watch.  Maybe Project Solara will get better as the hardware and associated software evolves.  Or, maybe we won’t be talking about this at Build 2027.

![Project Solara](/images/msft-build-2026-day-1/project-solara-1.jpg)

### Breakouts

I was able to attend three breakout sessions today and a few theater sessions.  Well, I’ll say I partially attended the theater sessions, because of the aforementioned problems with seating and A/V.

The theme for the breakouts, even for the ones I didn’t attend in person (but will be catching up with via the recordings), was definitely about how to make AI agents ready for production.  Last year, Build was about showing off what’s possible with AI and the early days of agents.  This year, it’s about how to manage key capabilities to make AI agents truly viable long term in an organization – security, scalability, managing costs, and observability.  At the heart of all of this sits Microsoft Foundry.  It’s clear that Microsoft is positioning Foundry as the center of their AI ecosystem.

Sessions I attended:

- [Observe and control agents across any framework with open source tools](https://build.microsoft.com/en-US/sessions/BRK250): This was a great session that focused on ways an agent can fail, and Microsoft's collaboration with the broader community to develop techniques to help improve agent relibabilty and safety.  Microsoft is releasing [ASSERT](https://github.com/responsibleai/ASSERT), a spec-driven eval and testing framework, and [Agent Control Specification (ACS)](https://commandline.microsoft.com/agent-control-specification-runtime-governance/), an approach for applying governance to AI agents.  ACS is part of [Microsoft's Agent Governance Toolkit](https://github.com/microsoft/agent-governance-toolkit)
- [Why your AI code doesn’t ship](https://build.microsoft.com/en-US/sessions/BRK200): Focused on the GitHub Copilot App and productivity tips I’ll try.
- [Deploy. Observe. Learn. Reinforcement learning for production agents](https://build.microsoft.com/en-US/sessions/BRK231): The best session I attended. The presenters explained why tuning models (distillation and related techniques) is now essential, especially given frontier-model economics. I’ll rewatch this and try the methods.
- [From observability to ROI for AI agents on any framework](https://build.microsoft.com/en-US/sessions/BRK252): A Foundry session showing logs, evaluations, and a new Rubric evaluation feature. Foundry Toolkit (MCP server and skills) looked useful. They demoed `azd ai agent optimize` for tuning prompts, skills, tools, and models, and a private-preview ROI evaluation that uses token and tool costs to compare agent versions. That ROI capability could help teams choose components that deliver real business value.

## Day 1 Snapshot

Despite the venue issues, day one at Build 2026 was valuable. Microsoft Foundry introduced several enhancements I want to try. The GitHub Copilot App looks promising for improving productivity, and I plan to explore OpenClaw further. Overall, day one delivered useful tools and insights — I’m looking forward to day 2.
