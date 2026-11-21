---
title: Start with XTM One
description: Sign in to XTM One, find your way around its navigation, ask an agent your first question and see where its work is recorded, in about fifteen minutes.
doc_type: tutorial
products: [xtm-one]
personas: [cti-analyst, soc-analyst, ai-builder]
owner: docs-xtm-one
last_reviewed: 2026-11-10
migrated_from:
  - xtm-one:/usage/getting-started/
no_screenshot: "The tour ends on a first conversation with an agent, which needs a recorded chat; its screens are captured in a next version."
---

XTM One is the AI layer of the XTM platform: AI agents that read what OpenCTI knows and what OpenAEV measures, answer your questions, write summaries and drafts, and run work on their own when something happens. You do not need to be an AI specialist to use them. In about fifteen minutes, this page signs you in, walks you through the navigation, has an agent answer a first question, and shows where its work is recorded. How the platform is built is in [How XTM One works: the platform, the worker and the XTM platform](/xtm-one/concepts/architecture/).

::diagram{id=xtm-one-getting-started-navigation}

## What you need

- The address of your XTM One platform, such as `https://xtm-one.grand-meridian.example`, and an account on it. An administrator creates accounts, or lets you sign in with your company account: see [Configure single sign-on](/xtm-one/administer/sso/).
- For the chat and the agents: the Enterprise Edition, with AI switched on. Without it, the AI areas of the navigation carry a lock, and the dashboard says how to activate it. See [What each edition of XTM One brings](/xtm-one/administer/editions/).
- At least one AI agent shared with you. Every platform comes with ready-made assistants, such as the **CTEM Assistant**, which answers as the **General Assistant**.

## Step 1: Sign in

:::steps
1. Open the address of your platform. The sign-in page shows the methods your administrator set up.
2. Sign in with your company account, which sends you to your identity provider and back, or with your email and password.
3. If you set up two-factor authentication, type the six-digit code of your authenticator app.

<!-- shot: xtm-one-getting-started-sign-in -->
:::

You land on **Home**, the home dashboard. When no OpenCTI or OpenAEV platform is connected yet, it shows how to connect one instead: an administrator does that once for everyone.

## Step 2: Find your way around

The left navigation reads from top to bottom like a sentence: the agents you talk to and how they work together, what they produce, what they are made of and act through. A group, such as **Orchestration** or **Work**, opens a page of cards when you click its name, and unfolds its entries with its arrow.

| Entry | What you do there | Read |
| --- | --- | --- |
| :menu[Home] | See the exposure posture of the XTM platform and what agents did lately | [Read the home dashboard](/xtm-one/use/dashboard/) |
| :menu[Chat] | Ask an agent in plain language | [Get answers from an agent in the chat](/xtm-one/use/chat/) |
| :menu[Orchestration] | **Agents** (open an agent: its instructions, tools, assignments and activity), **Fleets** (agents that run as a team), **Agentic Flow** (how agents hand work to each other, and what runs when) and, for administrators, **Intents** (which agent answers each AI feature) | [Create an agent](/xtm-one/build/agents/), [Run agents as a team with fleets](/xtm-one/build/fleets/), [Follow how agents hand work to each other](/xtm-one/use/agentic-flow/) |
| :menu[Work] | **Operations** (the runs), **Value** (what the agents' work is worth), **Presentations** (the decks) and **Files** (everything agents produced or received) | [Review what agents did](/xtm-one/use/work-history/), [Measure what your agents' work is worth](/xtm-one/use/value/) |
| :menu[Agent Studio] | What agents know and say: knowledge, prompts, templates, skills | [What agents are made of](/xtm-one/concepts/agents/) |
| :menu[Tools & Integrations] | What agents act through: MCP servers, tools, integrations, and **Connections** (browser profiles, variables, relays) | [Connect agents to your tools with integrations](/xtm-one/build/integrations/) |

To create an agent, click **New Agent** on the :menu[Orchestration > Agents] list, or **Create an agent** on the page that :menu[Orchestration] opens. Typing `create` in the search field of the top bar finds **Create an agent** too, under **Pages**.

Administrators also see the **Administration** section at the bottom, which folds open: **Observability** (model usage, outcomes, traces and sessions), **AI Adoption**, **Users and groups**, **Objects**, **Audit logs** and **Settings**. A group manager sees only **Model usage** there, the AI usage of their team. An entry you do not see is not open to your account. An entry with a lock needs the Enterprise Edition or AI switched on; hover it to read why. **Fleets** and **Value** lock only on the Community Edition: they stay open when AI is switched off.

The top bar holds the **Search the platform** field, your **Notifications**, the **AI Catalog** when an administrator turned it on (ready-made agents and resources to add to your account), and your user menu: **My Profile**, **Emergency stop** and **Sign out**.

Press `Ctrl K` (`Cmd K` on a Mac), or `/` outside a text field, to place the cursor in the search field. Type at least two characters: the list under the field shows the agents, fleets, conversations, documents and other objects you can see whose name matches, three per type, then the matching **Pages** of the navigation. **See all results**, or `Enter`, opens the **Search** page: every match, ranked by how well its name matches, with the list of types at its left to narrow them to one type. Its address keeps the search, so you can bookmark it or send it to a colleague, who sees only what they may read. Start with `>` to run a command, such as creating an agent, or with `@` to jump to a section of the page you are on. See [Find anything on the platform by its name](/xtm-one/use/search/).

## Step 3: Ask your first question

:::steps
1. Open :menu[Chat]. The **General Assistant** is selected, unless you chose another default agent in your profile.
2. Click **New conversation**.
3. Type a question about your own work, and send it.
4. Read the answer as the agent writes it. When it used tools (a search in OpenCTI, a knowledge base), the answer says what it found where.
:::

Each answer counts as one agentic execution of your quota: the lightning icon of the composer counts them.

## Step 4: See where the work is recorded

:::steps
1. Open :menu[Work > Operations]. The **Executions** tab lists the runs of the agents you can see: the work agents did on their own, on a schedule or on an event.
2. Open :menu[Work > Files]: the files you attached and those the agents produced are listed there, wherever they were made.
3. Back on :menu[Home], **Live Activity** shows the latest runs as they happen.
:::

:::example[Camille's first question to XTM One]
On 12 April 2026, Camille Roux, CTI analyst at Grand Meridian Hotels, signs in to `https://xtm-one.grand-meridian.example` with her company account. She opens :menu[Chat], keeps the **General Assistant**, clicks **New conversation** and asks:

> What does OpenCTI know about the intrusion set Velvet Harbor? Give me the three techniques it used against hotels in January 2026.

The **CTEM Assistant** answers for the General Assistant: it reads the intrusion set in OpenCTI with Camille's rights and lists the fake booking request, the PowerShell loader and the theft of guest records, each with its MITRE ATT&CK technique. In :menu[Work > Operations], she then finds the runs of **Velvet Harbor watcher**, the agent Sophie Laurent built to summarize every new report on Velvet Harbor for the CTI team.
:::

## What you learned

- XTM One agents answer in the chat, and also work on their own: [What agents are made of](/xtm-one/concepts/agents/).
- What an agent reaches depends on who asks: [Who sees what: containers, scopes and nesting](/xtm-one/concepts/scopes/).
- Everything agents do is recorded and can be reviewed: [How XTM One keeps what agents know and do](/xtm-one/concepts/data-model/).
