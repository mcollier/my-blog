---
layout: post
title: "Microsoft Build 2025 Recap"
date: 2025-05-23
categories:
author: "Michael S. Collier"
tags: [azure-openai, AI, agents]
comments: true
---

Microsoft Build 2025 centered on the future of intelligent agent-based applications. This post recaps my four-day experience in Seattle attending Build, covering keynotes, developer sessions, GitHub Copilot updates, .NET Aspire insights, and hands-on labs. Get a firsthand look at emerging trends and tools shaping how we build AI-powered solutions in the Microsoft ecosystem.

<!--more-->

# Build 2025 Recap

**Agents. Agents. Agents communicating with agents.**

That was the unmistakable theme at Microsoft Build 2025. It’s clear Microsoft is all-in on the agentic future, positioning agents as the new foundation for how apps are built, interact, and scale.

But let’s be honest: it’s still early days. The tooling is fragmented, the use cases are emerging, and the hype is . . . palpable. Still, you can feel something powerful forming. The next few months, not just the next year, will be telling.

And every time someone mentioned "agents", this images comes to my mind:

{{< figure src="/images/msft-build-2025-recap/chatgpt-agent-smith.png" class="agent-image">}}

Let's dive into my day-by-day recap, from keynotes to labs.

## Day 1

Sleep? Not so well. Between jet lag and excitement, I was up early and in line 45 minutes before the doors opened.  Bring on the coffee! :coffee:

I got in line about 45 minutes before doors opened at 8am. By 8:45am, Build kicked off with the announcement of the [Imagine Cup](https://imaginecup.microsoft.com/en-us) finalists. Each team and their project was incredibly impressive!  Congratulations to Argus, this year's [world champion](https://techcommunity.microsoft.com/blog/studentdeveloperblog/announcing-the-2025-imagine-cup-world-champion/4414342)!

### Keynote

- **Satya Nadella** opened with a compelling vision for AI’s expanding role in software development.
- The announcement of [GitHub Copilot Coding Agent](https://github.blog/news-insights/product-news/github-copilot-meet-the-new-coding-agent/) (formerly Project Padawan). Currently gated behind [GitHub Copilot Pro+](https://docs.github.com/en/copilot/about-github-copilot/plans-for-github-copilot), but fingers crossed this becomes more accessible soon!
- **Azure AI Foundry** continues to grow with exciting new capabilities.
- Special guests included **Sam Altman (OpenAI CEO)**, who shared valuable insights on developer productivity, and [OpenAI's Codex](https://openai.com/index/introducing-codex/). Unfortunately, **Elon Musk** also made an unsurprisingly incoherent appearance to promote Grok's inclusion in Azure AI Foundry.  Moving on. 🤷‍♂️
- **Kevin Scott (Microsoft CTO)** introduced [NLWeb](https://news.microsoft.com/source/features/company-news/introducing-nlweb-bringing-conversational-interfaces-directly-to-the-web/?msockid=391a5e1ff6be6b070ee04bf9f7c56a98), speaking unscripted, with energy, and clarity. 

If missed it, watch the keynote [here](https://build.microsoft.com/en-US/sessions/KEY010?source=/schedule)

{{< gallery "/images/msft-build-2025-recap/build-day1-keynote.jpg" "/images/msft-build-2025-recap/build-day1-imagine-cup.jpg" "/images/msft-build-2025-recap/build-day1-keynote-coding-agent.jpg" "/images/msft-build-2025-recap/build-day1-keynote-stack.jpg" "/images/msft-build-2025-recap/build-day1-keynote-foundry.jpg" "/images/msft-build-2025-recap/build-day1-keynote-agentic-web.png" >}}

### Sessions

After the keynote, it was time to get to the sessions.  There are so many good sessions, it's impossible to attend them all.  I'll watch several sessions in the coming weeks.

As primarily a .NET developer wanting to learn more about how to better build & architect the next generation of intelligent apps, I focused my attention on those sessions.

Sessions I attended:

- :microphone: [Reimagining Software Development and DevOps with Agentic AI](https://build.microsoft.com/en-US/sessions/BRK100)
- :microphone: [Develop, Build and Deploy LLM Apps using GitHub Models and Azure AI Foundry](https://build.microsoft.com/en-US/sessions/BRK107)
- :microphone: [The Agent Awakens: Collaborative Development with GitHub Copilot](https://build.microsoft.com/en-US/sessions/BRK113)
- :microphone: [Event-Driven Architectures: Serverless Apps That Slay at Scale](https://build.microsoft.com/en-US/sessions/BRK187)
- :microphone: [How Microsoft Developers Use AI in Real-World Coding](https://build.microsoft.com/en-US/sessions/BRK103)

:mag: **Favorite session:** My standout was **David Fowler's** and **Stephen Toub's** ["How Microsoft Developers Use AI in Real-World Coding"](https://build.microsoft.com/en-US/sessions/BRK103) session. It resonated deeply, demonstrating Copilot as a valuable assistant rather than a replacement for skilled developers. Key insights included Copilot's help with documentation, ideating on complex problem-solving, and feature implementation, thus **freeing developers to focus on high-impact tasks**.

:bulb: **Key takeaway:**
The takeaway from Day 1 was clear: **GitHub Copilot is pivotal but imperfect**. Developers must thoughtfully review and refine Copilot's outputs. Prompt engineering (or as one mentor aptly put it, "prompt experiments") plays a crucial role. Is the juice worth the squeeze? In the right situations, yes.

Additionally, in typical Microsoft fashion, keynote and session demos are very polished. I noticed many demos were live, which is great!  However, with AI, live creates an interesting dynamic in that it may not always work or produce the anticipated results (the probabilistic nature of generative AI).  Presenters often caveated their demos with "this is live . . . let's see what it does".

> Check out the [Microsoft Build 2025 Book of News](https://news.microsoft.com/build-2025-book-of-news/) for a full list announcements and news from Build.

## Day 2

### Keynote

Up early again (yep, I'm crazy), I snagged a great seat four rows from the stage for the keynote:

- **Jay Parikh** opened, followed by an engaging **Charles LaManna**, who showcased AI innovations in the Power Platform.
- **Scott Guthrie** offered a breathtaking view of Microsoft's latest AI datacenter scale.
- Demos were heavily scripted. Too scripted in my opinion. Unfortunately, there wasn't any .NET content (in either keynote :disappointed: ) or much of Scott Guthrie, who I always appreciate hearing from.

You can watch the day 2 keynote [here](https://build.microsoft.com/en-US/sessions/KEY020).

{{< gallery "/images/msft-build-2025-recap/build-day2-keynote-1.jpg" "/images/msft-build-2025-recap/build-day2-keynote-2.jpg" "/images/msft-build-2025-recap/build-day2-keynote-3.jpg" "/images/msft-build-2025-recap/build-day2-keynote-4.jpg" "/images/msft-build-2025-recap/build-day2-keynote-5.jpg">}}

### Sessions

For day 2, I attended a few in-person labs.  This was a great way to hands-on experience in a few areas I wanted to explore.

- [Conversations: Let's Talk .NET Aspire](https://build.microsoft.com/en-US/sessions/COMM411?source=/schedule)  This session was not recorded!
- :computer: [Build your code-first agent with Azure AI Foundry in .NET](https://build.microsoft.com/en-US/sessions/LAB321)
- :computer: [Building GenAI Apps in C#: AI Templates, GitHub, Azure OpenAI & More](https://build.microsoft.com/en-US/sessions/LAB307-R1)
- :microphone: [Best practices for building agentic apps with Azure AI Foundry](https://build.microsoft.com/en-US/sessions/BRK152)

:mag: **Favorite session:**  I continue to be impressed and excited by [.NET Aspire](https://learn.microsoft.com/en-us/dotnet/aspire/get-started/aspire-overview).  It was great to hear how other Build attendees are using Aspire and the challenges they're having.  The .NET Aspire team was receptive to feedback and understanding how people are using Aspire. Aspire's come a long way in the last year.  I'm really excited about what the future holds!

:bulb: **Key takeaway:**
The labs were OK.  They are mostly copy & paste.  If you want, you can complete quickly.  I don't recommend that, as you're likely not learning much.  Much of the **lab content is online, so you can go back and review at your own pace**, which I like.  There were some hiccups in the labs, likely due to the fact that many products used in the labs were updated the week of Build, and thus some of the prepared content didn't match any longer.  The proctors did good coaching participants through those areas.

## Day 3

No keynote today - extra sleep!! :sleeping:

### Sessions

I started day 3 with another lab, followed by several sessions.

- :computer: [Accelerate AI App Development with AI Gateway Capabilities in Azure API Management](https://build.microsoft.com/en-US/sessions/LAB340-R1)
- :microphone: [Building secure AI Agents with Azure Functions](https://build.microsoft.com/en-US/sessions/BRK189)
- :microphone: [Accelerate Azure Development with GitHub Copilot, VS Code & AI](https://build.microsoft.com/en-US/sessions/BRK118)
- :microphone: [Elevating Development with .NET Aspire: AI, Cloud, and Beyond](https://build.microsoft.com/en-US/sessions/BRK106)
- :microphone: [Yet "Another Highly Technical Talk" with Hanselman and Toub](https://build.microsoft.com/en-US/sessions/BRK121)

:mag: **Favorite session:**  On Wednesday I was fortunate to attend a small group discussion on building AI-enabled apps with Microsoft DevDiv leaders. Fantastic interaction. Attendees were able share their challenges in building AI solutions.  It was mostly about **Microsoft product leaders gaining a deeper understanding of customer challenges** to improve product offerings. The Microsoft representatives were very engaging and willing to help customers try to overcome the challenges they had.  

:bulb: **Key takeaway:**
I'm bullish on the prospects of building intelligent apps with .NET Aspire. Combining the Aspire-powered dev experience by with the possibilities in AI-assisted development and empowered applications is exciting.  The developer landscape is changing incredibly fast, and probably will not settle for a little while yet.  That's OK, as opportunities abound!

### Microsoft Build and AMD Celebration

:partying_face: After 3 long days, it was time to unwind a bit will fellow Build attendees.  Microsoft and AMD hosted a part at Lumen Field, home of the Seattle Seahawks.  There was plenty of food and drink available from the various concession stands in the stadium.  Attendees could also get on the field for various competitions, including the chance to run a NFL combine style 40 yard dash or kick a field goal.  I tried the field goal . . . let's just say I'm not going to be getting called up to the league any time soon.

{{< gallery "/images/msft-build-2025-recap/build-celebration.jpg" "/images/msft-build-2025-recap/build-celebration-2.jpg">}}

## Day 4

The final day!  There are relatively few sessions, as the event is over by about 12:30pm

### Sessions

I attended two breakout sessions and the closing keynote session today:

- :microphone: [Build the next gen of AI apps with .NET: Models, Data, Agents, & More](https://build.microsoft.com/en-US/sessions/BRK104)
- :microphone: [A Visual Studio Story: How We Build Software Loved by Millions of Developers](https://build.microsoft.com/en-US/sessions/BRK133)
- :microphone: [Scott and Mark Learn to...LIVE](https://build.microsoft.com/en-US/sessions/KEY040)

:eyes: I found it very interesting to see how the Visual Studio team approaches attempting to understand how developers use Visual Studio, listening to customer feedback. It was standing room only in this session. The **Visual Studio session underscored its enduring appeal among .NET developers**, contrary to misguided claims otherwise.

:robot: As one may expect, **Scott Hanselman** and **Mark Russinovich** delivered an entertaining and informative closing keynote.  They demonstrated using a robot, from [hello robot](https://hello-robot.com/stretch-3-product) to deliver Scott a Diet Coke. It worked . . .somewhat. It was entertaining to try to get the robot to follow the instructions.  It underscored the challenges of telling a machine to do things that humans find innate.  They also showed a few entertaining real-world examples of **prompt injection attacks**.

The overarching theme: **keep a human in the loop**. Agents are helpful—but trust, safety, and intention must be core to every AI solution.

A fantastic wrap to another stellar Microsoft Build!

## Final thoughts

Build 2025 was intense, exciting, and inspiring, with just the right amount fun along the way. Microsoft’s vision is clear: **agents are the future. But the path is still forming.**

Personally, I walked away inspired, particularly by the Aspire sessions and the thoughtful pro-code sessions on building and using agentic applications.

Will AI agents replace developers? No. But they will reshape how we build.  Buckle up!
