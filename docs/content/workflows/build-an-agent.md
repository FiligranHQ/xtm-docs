---
title: Build an AI agent your team can trust
description: Build an XTM One AI agent that knows your policies, reads OpenCTI without changing it, is shared with a team, runs on schedule within a budget, and leaves a trace of every run.
doc_type: journey
journey: build-an-agent
products: [xtm-one, opencti]
personas: [ai-builder, cti-analyst, admin]
owner: docs-suite
last_reviewed: 2026-11-09
sidebar_position: 70
---

At the end of this journey, your team has an AI agent it can rely on: it quotes your own policies, reads OpenCTI with the rights of the person it works for and changes nothing there, is shared with the team that uses it, runs every week and when another tool calls it, stops at a cost you set, and leaves a record of every run. An AI agent builder drives it in XTM One, the CTI team uses the result, and an administrator raises a limit when needed. The steps below are the pages to follow, in order.

::diagram{id=journey-build-an-agent}

## Who takes part

- **The AI agent builder** (Sophie Laurent, the AI engineer of Grand Meridian Hotels) drives every step: the knowledge base, the skill, the agent, its test, its sharing, its triggers and its budget.
- **The CTI analysts** (Camille Roux and the CTI team) use the agent once it is shared, read the weekly brief in the work history, and rate its answers.
- **The administrator** (Hélène Marchetti, the CISO, administrator of XTM One) registered OpenCTI with XTM One, and sets the one budget the builder cannot: a limit above the platform's.

## Why it spans several modules

The agent lives in XTM One; the knowledge it summarizes lives in OpenCTI.

- **XTM One** holds what the agent is made of: its persona, its knowledge bases, its skills, the integrations it acts through, its triggers and its budget. It runs the agent and keeps its history. See [What agents are made of](/xtm-one/concepts/agents/).
- **OpenCTI** holds the reports on Velvet Harbor. Registering OpenCTI with XTM One creates a company-managed integration that stores no credential: every call carries a short-lived token signed for the person the agent works for, so the agent reads only what that person may read, under their markings and organizations. See [What an agent can reach, and with which rights](/xtm-one/use/in-opencti/#what-an-agent-can-reach-and-with-which-rights).

What passes between them: OpenCTI's reports go to the agent, read with the tools that search, get and list; nothing comes back, because the agent keeps no tool that writes. Built in OpenCTI alone, an AI feature has no persona of your own, no policy to quote and no schedule. Built in XTM One without OpenCTI, the agent has nothing to summarize.

## What you need

- **XTM One in Enterprise Edition, with AI switched on.** Agents, the chat, knowledge bases, skills, assignments and webhook triggers are AI areas, and connecting a new integration requires the Enterprise Edition. Sharing and the work history are not: **Operations** stays readable with AI off. The Enterprise Edition comes from a registered OpenCTI in Enterprise Edition or from an XTM One license; the included allowance of a connected platform, 30 agentic executions a day, is sized for trying the agent, not for a team. See [What each edition of XTM One brings](/xtm-one/administer/editions/).
- **OpenCTI registered with XTM One.** A platform engineer sets the registration: see [Connect the modules to each other](/deploy/configure/connect-modules/#register-the-platforms-with-xtm-one).
- **The same email in both modules** for everyone the agent works for. An XTM One account with no match in OpenCTI gets no knowledge.
- **In XTM One**, the **User** role is enough to build, test and share your own agent, knowledge base and skill. The **Administrator** role makes a resource company-managed, and raises a run budget above the platform's. The platform budget itself is set by a platform engineer in the deployment.
- **In OpenCTI**, the **Access knowledge** capability for each analyst who chats with the agent, and for the builder: the agent's scheduled runs act as its owner.

## How it unfolds

The story is the one of the step pages: in April 2026, three months after Velvet Harbor reached the front desks of Grand Meridian Hotels with a fake booking for the European Hospitality Summit 2026, the CTI team wants every new OpenCTI report on the intrusion set summarized, with the escalation their own policy requires.

### Plan the blocks

Sophie reads what an agent is made of: the persona decides, the blocks enable. **Agent Studio** holds the knowledge, prompts, templates and skills; **Tools & Integrations** holds the services the agent acts through. She chooses the fewest tools that do the job, so the agent will read OpenCTI and never write. See [Know what an agent is made of](/xtm-one/concepts/agents/).

### Give it what it must know

She creates the knowledge base **Grand Meridian security policies** with the incident response policy and the crisis communication playbook, turns on **Read-only content** so no agent changes them, and checks with **Test Search** that the question "When must a guest data breach be escalated to the crisis cell?" finds the escalation section. See [Give it your documents](/xtm-one/build/knowledge-bases/).

She then checks the integrations: the registered OpenCTI is there already, **Managed by the platform**, with its read tools and its write tools listed apart. See [Connect it to OpenCTI](/xtm-one/build/integrations/).

A procedure the agent follows only for some tasks goes in a skill rather than in the persona: the skill's description is all the agent sees until a task calls for it. Sophie packages the SOC's Sigma conventions this way, as **Write a Sigma rule**. See [Teach it a procedure](/xtm-one/build/skills/).

### Build it and try it

In the agent wizard, she builds **Velvet Harbor watcher** from scratch: a persona that searches OpenCTI for the reports on Velvet Harbor since a given date and quotes the policy, the OpenCTI integration, the knowledge base, and a guardrail that forbids creating, changing or deleting anything in OpenCTI. On the agent's **Tooling**, she narrows the OpenCTI integration to the tools that match `search_opencti`, `get_opencti` and `list_opencti`. See [Create the agent](/xtm-one/build/agents/).

In the chat, she asks "What is new on Velvet Harbor since 1 April 2026?". The answer lists the new reports with their OpenCTI links and quotes the escalation section; its **Reasoning details** show only the OpenCTI searches and the knowledge base search. See [Try it in the chat](/xtm-one/use/chat/).

### Hand it to the team

She shares the agent with the **CTI team** group, **Read only**: Camille finds it in her **Agents** list and chats with it, and reads OpenCTI with her own rights, not Sophie's: a platform-managed integration lends no credential. See [Share it with the team](/xtm-one/build/sharing/).

Then the agent gets its work. The assignment **Weekly Velvet Harbor brief** runs every Monday at 06:00 UTC, before the team's meeting. A second assignment, **SOAR case to CTI brief**, has a webhook trigger: the SOAR posts each case where the EDR blocked a PowerShell loader on a front-desk workstation, with the secret kept in its credential store. Both run as Sophie, the agent's owner, so they read OpenCTI with her rights. See [Start it from another tool](/xtm-one/use/webhook-triggers/).

### Bound it and review it

The platform budget, $5.00 and 1,000,000 tokens per run, applies to every run. The Monday brief reads every new report and needs $8.00: Sophie's field shows that only an administrator can set a value above the platform budget, and Hélène sets it. See [Cap what it spends](/xtm-one/use/run-budgets/).

On Monday 13 April 2026 at 08:20, Camille opens :menu[Work > Operations], filters on **Velvet Harbor watcher**, reads the summary of the morning run and rates it. Every run, its trigger, its tool calls and its result stay there. See [Review what it did](/xtm-one/use/work-history/).

## The steps

Each step opens its page; the first one starts the journey, and every page of it then shows where you are.

::journey-steps

## Where to go next

- [Govern how AI is used across the platform](/workflows/govern-ai/): make the agent ask before a tool changes something, screen what it reads for prompt injection, and keep one switch to stop it.
- [Let the night shift ask agents from Slack](/workflows/answer-from-slack/): bring an agent to the channel the SOC already works in.
- [Choose which agent answers each AI feature](/xtm-one/administer/intents/): let the watcher write the **Summarize** of OpenCTI's reports.
- [How access works in XTM One](/xtm-one/concepts/access/): the two roles, groups and sharing behind who reaches the agent.
