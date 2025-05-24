---
layout: post
title: "Microsoft Build 2025 Recap"
date: 2025-05-19T04:56:16Z
categories:
author: "Michael S. Collier"
tags:
comments: true
---

# Build 2025 Recap

The theme at Microsoft Build 2025 was unmistakably clear: **Agents. Agents. Agents communicating with agents.** It's obvious Microsoft is fully invested in showcasing the transformative potential of agents across their platforms. However, it's early days for this technology, and there's plenty of hype, but also a lot of genuine promise. It'll be fascinating to see how much changes by Build 2026, or even just three months from now.

Every time I hear someone talking about agents, I can't help but to picture . . .

{{< figure src="/images/msft-build-2025-recap/chatgpt-agent-smith.png" class="agent-image">}}

Here’s my personal recap of each day, highlighting the sessions I attended.

## Day 1

I could hardly sleep the night before. Excitement? Definitely. Fear of oversleeping? Absolutely. Jet lag from Eastern time? Probably that too. Either way, that's just how I roll.

I got in line about 45 minutes before doors opened at 8am. By 8:45am, Build kicked off with the announcement of the [Imagine Cup](https://imaginecup.microsoft.com/en-us) finalists. Each team and their project was incredibly impressive!  Congratulations to Argus, this year's [world champion](https://techcommunity.microsoft.com/blog/studentdeveloperblog/announcing-the-2025-imagine-cup-world-champion/4414342)!

### Keynote

- **Satya Nadella** opened with a compelling vision for AI’s expanding role in software development.
- The announcement of [GitHub Copilot Coding Agent](https://github.blog/news-insights/product-news/github-copilot-meet-the-new-coding-agent/) (formerly Project Padawan). Currently gated behind [GitHub Copilot Pro+](https://docs.github.com/en/copilot/about-github-copilot/plans-for-github-copilot), but fingers crossed this becomes more accessible soon!
- **Azure AI Foundry** continues to grow with exciting new capabilities.
- Special guests included **Sam Altman (OpenAI CEO)**, who shared valuable insights on developer productivity, and [OpenAI's Codex](https://openai.com/index/introducing-codex/). Unfortunately, **Elon Musk** also made an unsurprisingly incoherent appearance to promote Grok's inclusion in Azure AI Foundry.  Moving on.
- **Kevin Scott (Microsoft CTO)** impressively presented without relying on a teleprompter, introducing the fascinating [NLWeb](https://news.microsoft.com/source/features/company-news/introducing-nlweb-bringing-conversational-interfaces-directly-to-the-web/?msockid=391a5e1ff6be6b070ee04bf9f7c56a98). I definitely need to learn more on this area.

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

My standout was **David Fowler's** and **Stephen Toub's** ["How Microsoft Developers Use AI in Real-World Coding"](https://build.microsoft.com/en-US/sessions/BRK103) session. It resonated deeply, demonstrating Copilot as a valuable assistant rather than a replacement for skilled developers. Key insights included Copilot's help with documentation, ideating on complex problem-solving, and feature implementation, thus **freeing developers to focus on high-impact tasks**.

The takeaway from Day 1 was clear: **GitHub Copilot is pivotal but imperfect**. Developers must thoughtfully review and refine Copilot's outputs. Prompt engineering (or as one mentor aptly put it, "prompt experiments") plays a crucial role. Is the juice worth the squeeze? I think yes.

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

I continue to be impressed and excited by [.NET Aspire](https://learn.microsoft.com/en-us/dotnet/aspire/get-started/aspire-overview).  It was great to hear how other Build attendees are using Aspire and the challenges they're having.  The .NET Aspire team was incredibly receptive to feedback and understanding how people are using Aspire. Aspire's come a long way in the last year.  I'm really excited about what the future holds!

The labs were OK.  They are mostly copy & paste.  If you want, you can complete quickly.  I don't recommend that, as you're likely not learning much.  Much of the **lab content is online, so you can go back and review at your own pace**, which I like.  There were some hiccups in the labs, likely due to the fact that many products used in the labs were updated the week of Build, and thus some of the prepared content didn't match any longer.  The proctors did good to coach participants through those areas.

## Day 3

No keynote today - extra sleep!! :sleeping:

### Sessions

I started day 3 with another lab, followed by several sessions.

- :computer: [Accelerate AI App Development with AI Gateway Capabilities in Azure API Management](https://build.microsoft.com/en-US/sessions/LAB340-R1)
- :microphone: [Building secure AI Agents with Azure Functions](https://build.microsoft.com/en-US/sessions/BRK189)
- :microphone: [Accelerate Azure Development with GitHub Copilot, VS Code & AI](https://build.microsoft.com/en-US/sessions/BRK118)
- :microphone: [Elevating Development with .NET Aspire: AI, Cloud, and Beyond](https://build.microsoft.com/en-US/sessions/BRK106)
- :microphone: [Yet "Another Highly Technical Talk" with Hanselman and Toub](https://build.microsoft.com/en-US/sessions/BRK121)

On Wednesday I was fortunate to attend a customer forum on "Building AI-Enabled Applications".  This was a small session, about 10 customers, with other Build attendees and members of Microsoft's DevDiv product leadership team.  Attendees were able share their challenges in building AI solutions.  It was mostly about **Microsoft product leaders gaining a deeper understanding of customer challenges** to improve product offerings. The Microsoft representatives were very engaging and willing to help customers try to overcome the challenges they had. Loved this session!  

### Microsoft Build and AMD Celebration

:partying_face: After 3 long days, it was time to unwind a bit will fellow Build attendees.  Microsoft and AMD hosted a part at Lumen Field, home of the Seattle Seahawks.  There was plenty of food and drink available from the various concession stands in the stadium.  Attendees could also get on the field for various competitions, including the chance to run a NFL combine style 40 yard dash or kick a field goal.  I tried the field goal . . . let's just say I'm not going to be getting called up to the league any time soon.

{{< gallery "/images/msft-build-2025-recap/build-celebration.jpg">}}

## Day 4

The final day!  There are relatively few sessions, as the event is over by about 12:30pm

### Sessions

I attended two breakout sessions and the closing keynote session today:

- :microphone: [Build the next gen of AI apps with .NET: Models, Data, Agents, & More](https://build.microsoft.com/en-US/sessions/BRK104)
- :microphone: [A Visual Studio Story: How We Build Software Loved by Millions of Developers](https://build.microsoft.com/en-US/sessions/BRK133)
- :microphone: [Scott and Mark Learn to...LIVE](https://build.microsoft.com/en-US/sessions/KEY040)

I found it very interesting to see how the Visual Studio team approaches attempting to understand how developers use Visual Studio, listening to customer feedback. It was standing room only in this session. The Visual Studio session underscored its enduring appeal among .NET developers, contrary to misguided claims otherwise.

As one may expect, Scott Hanselman and Mark Russinovich delivered an entertaining and informative closing keynote.  They demonstrated using a robot, from [hello robot](https://hello-robot.com/stretch-3-product) to deliver Scott a Diet Coke. It worked . . .somewhat. It was entertaining to try to get the robot to follow the instructions.  It underscored the challenges of telling a machine to do things that humans find innate.

Scott & Mark also showed a few examples of prompt injection attacks, underscoring the importance of keeping the human in the loop and ensuring safety guardrails are in place.

A fantastic wrap to another stellar Microsoft Build!
