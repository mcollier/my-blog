---
title: "Prompt Agent Harness"
date: 2026-10-28
draft: true
---

Today Microsoft unveiled an exciting new capability for Microsoft Foundry prompt agents, [Microsoft Copilot harness in Foundry Agent Service](https://devblogs.microsoft.com/foundry/microsoft-copilot-harness-in-foundry/).  This new capability greatly enhances what Foundry prompt agents can do.  With the new harness (built on what was [previously known as the GitHub Copilot SDK](https://blogs.microsoft.com/blog/2026/10/28/introducing-the-microsoft-copilot-harness-unlocking-the-potential-of-truly-multi-model-agentic-work/)), prompt agents now have the ability to read & write files in the agent workspace, perform web searches, run shell commands in a sandboxes session, spawn , and add standard agent skills (via a SKILLS.md or the [Foundry Skills Catalog](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/private-skill-catalog)).

<!-- more -->

## Why a Harness Matters

An **agent harness** is the runtime that turns an AI model into an agent capable of performing work. It coordinates model and tool calls, manages conversation state and context, and can keep the agent progressing through multiple steps.

A robust harness is especially important for reasoning-heavy agents that handle more complex scenarios. It supports moving beyond a fixed script: an agent can break a goal into steps, access knowledge and tools through various connectors, inspect results, and adjust when a step fails or a request changes. The harness provides the context, execution capabilities, and controls that help sustain this work through completion.

## Harnesses in Other Microsoft AI Services

Microsoft has been adding more powerful harness to other services.  For instance, earlier this summer, [Microsoft Agent Framework released support for new harness capabilities](https://devblogs.microsoft.com/agent-framework/the-microsoft-agent-framework-harness-is-now-released/) for custom agents.  Most recently, in August, Microsoft [announced a new GitHub Copilot harness for Copilot Studio](https://techcommunity.microsoft.com/blog/copilot-studio-blog/more-powerful-agents-and-workflows-for-autonomous-business-processes-introducing/4542969).

Now, Foundry is getting an improved harness too!

## Microsoft Copilot harness in Foundry Agent Service

When I'm creating a Foundry agent and deciding between a prompt agent and a (custom code) hosted agent, I tend to think of it like this:

![foundry prompt vs. hosted agent comparison](/images/microsoft-copilot-harness-prompt-agent/foundry-prompt-vs-hosted-agent.png)

Now, when I look at the capabilities with the new Microsoft Copilot harness, there is a solution that sits in the middle between a traditional prompt agent and custom code agents.  Let's get started!

## Getting Started

Let's walk through creating a new Foundry prompt agent using the new Microsoft Copilot harness.

### Create a Foundry resource, project, and model

First, you'll need to have a Foundry resource and associated project.  If you don't yet, [follow the quickstart](https://learn.microsoft.com/en-us/azure/foundry/tutorials/quickstart-create-foundry-resources?tabs=azurecli) to get a Foundry resource and project created.

The same quickstart will guide you through deploying a model too. Deploy a model such as **gpt-5-mini** or **gpt-5.6-sol**, but any available model should work.

### Create an agent

With the Foundry resource, project, and model in place, it's time to create your new Foundry prompt agent.  From the Foundry portal, you'll want to select **Build** from the upper navigation menu and then **Agents** from the left-side navigation menu.  That'll get you to the Agents page to view existing agents and create new ones.

Next, click the **New agent** button to, well, create a new agent (bet you didn't see that coming :wink:).  Select the option to **Build an agent**.

Now you'll provide your agent with an awesome name and select **Text** for the interaction mode.

Check the **Use the Microsoft Copilot harness** checkbox to use the new harness.

![Select option to use the Microsoft Copilot harness on a new Foundry prompt agent](/images/microsoft-copilot-harness-prompt-agent/create-prompt-agent-with-copilot-harness-2.png)

Finally, click on the **Create agent and open playground** button to complete and start adding agent capabilities via the agent playground.

### Add agent capabilities

The Foundry prompt agent playground is where you can select your agent's model, provide instructions, add a knowledge base (e.g., using data in Azure AI Search to inform your agent's response), manage safety and security guardrails, and set the tools your agent has access to use.  Nearly all this is the same (as standard prompt agents), with the exception of the tools.

When using the Microsoft Copilot harness, you'll notice the agent has access to several built-in harness tools:

- File system read
- File system write
- Web
- Shell
- Subagents

Additionally, using the Microsoft Copilot harness also adds the ability for using a code interpreter and adding skills.  With skills, you can write a new skill from scratch (set the name, description and instructions), or upload an existing SKILL.md FILE.  You can also reference an existing skill from the new Foundry Skill Catalog.

![write or upload a skill](/images/microsoft-copilot-harness-prompt-agent/foundry-skill-catalog-2.png)  
![select a skill from the Foundry Skill Catalog](/images/microsoft-copilot-harness-prompt-agent/select-a-skill-2.png)

I think you'll agree there are a lot of capabilities here that will make Foundry prompt agents a lot more compelling and powerful to use!

With the new Microsoft Copilot harness, there is now an option that sits between traditional prompt agents and code-first hosted agents.

![middle ground for prompt agents](/images/microsoft-copilot-harness-prompt-agent/foundry-three-agent-options.png)

### Let's do something

One of the first things I did with a new prompt agent using the Microsoft Copilot harness was to try out the web search and subagent capabilities.  I was curious how it would work to spawn several subagent to execute a provided tasks, and have each carry out a web search.

The idea (granted relatively trivial for this early experiment) was to have an agent create a table showing the population of each U.S. state using the official data from the most recent census.  The prompt was a follows:

``` text
Create a table showing the population of each state in the United States.  Use the most recent official United States government census data.  Use subagents to split up the work.
```

![]()

## Wrapping up

I'm looking forward to building more agents with the new Microsoft Copilot harness for Foundry prompt agents.  The new capabilities should make prompt agents a much more capable alternative (potentially negating some needs to elevate to a code-first agent via Foundry hosted agents and Microsoft Agent Framework).
