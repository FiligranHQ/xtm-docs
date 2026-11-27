---
title: XTM One
description: The AI layer at the center of the XTM platform. Build agents that read what OpenCTI knows,
  what OpenAEV measures and what OpenCRQ prices, put them to work, and govern how AI is used.
doc_type: landing
hide_table_of_contents: true
sidebar_label: Overview
products:
  - xtm-one
sidebar_position: 0
---

:::product-hero{eyebrow="AI for the XTM platform" title="Put AI agents to work on your security, under your rules" primary-label="Start here" primary-to="#start-here" secondary-label="Deploy the XTM platform" secondary-to="/deploy/install/docker-compose/" shot=xtm-one-agent-overview alt="The Overview of the agent Velvet Harbor watcher, built by Sophie Laurent: two active assignments, three runs, two chat queries, one connected integration and its last activity on 12 April, then what it does next under Coming up, with the Weekly Velvet Harbor brief of Monday morning."}
XTM One is where the AI of the XTM platform lives. Its agents read what OpenCTI knows, what OpenAEV measures and what OpenCRQ prices, and they act: they summarize a week of reports, triage an alert with the night shift, draft a crisis briefing, write the detection rule a simulation showed missing. You talk to them in the chat or from Slack and Teams, give them recurring work, or call them from inside the other modules. Each agent reaches only the data and tools you give it, and every run leaves a trace.
:::

## Start here

Pick your role: each tab gives your goal and the pages to read first, in order.

::::tabs
:::tab[CTI and SOC analysts]
You hand an agent the reading, the triage and the first draft, and keep the judgment.

1. [Start with XTM One](/xtm-one/use/getting-started/): sign in, find the agents shared with you, ask your first question.
2. [Get answers from an agent in the chat](/xtm-one/use/chat/): attach a report, follow what the agent does, approve what it proposes.
3. [Let an agent act later, at a time you choose](/xtm-one/use/work-items/): a message posted tomorrow at 9:00, a summary sent on Friday.
4. [XTM One in OpenCTI](/xtm-one/use/in-opencti/): **Ask Ariane**, the AI of playbooks and what agents may change in OpenCTI.
:::
:::tab[AI agent builders]
You design the agents of your teams: what they know, what they may do, how you test them.

1. [What agents are made of](/xtm-one/concepts/agents/): instructions, model, knowledge, skills, tools and integrations.
2. [Give agents your documents with knowledge bases](/xtm-one/build/knowledge-bases/): your policies and playbooks, searched at each answer.
3. [Connect agents to your tools with integrations](/xtm-one/build/integrations/): OpenCTI, OpenAEV, OpenCRQ and your other tools, read-only where it must.
4. [Create an agent](/xtm-one/build/agents/): from the instructions to the first test in the chat.
5. [Share agents and resources](/xtm-one/build/sharing/): who may use, configure or read the chats of an agent.
:::
:::tab[Administrators]
You decide who uses AI, on which models, with which data, and how to stop it.

1. [How access works in XTM One](/xtm-one/concepts/access/): administrators, users, groups and what sharing grants.
2. [Configure single sign-on](/xtm-one/administer/sso/): sign in with your identity provider.
3. [Protect your data and approve what agents do](/xtm-one/administer/ai-governance/): the limits and the approvals agents work under.
4. [Talk to agents from your messaging apps](/xtm-one/administer/channels/): Slack, Microsoft Teams and the others.
5. [Stop every agent at once](/xtm-one/administer/emergency-stop/): the switch for the day something goes wrong.
:::
:::tab[Platform engineers]
You deploy the XTM platform, which always runs XTM One with the other modules, and keep XTM One healthy and up to date.

1. [How XTM One works](/xtm-one/concepts/architecture/): the platform, the worker, the database and what the other modules send it.
2. [Install with Docker Compose](/deploy/install/docker-compose/): the whole XTM platform, XTM One with every other module, behind your reverse proxy.
3. [Configure XTM One](/deploy/configure/xtm-one/): finish its setup with a language model and its edition, then the modules it connects and the security headers.
4. [XTM One configuration](/reference/xtm-one/configuration/): every environment variable, its default and what it does.
:::
:::tab[CISOs and executives]
You see what AI does for the teams and keep it within bounds.

1. [What each edition of XTM One brings](/xtm-one/administer/editions/): what the Enterprise Edition adds to the XTM platform.
2. [Read the home dashboard](/xtm-one/use/dashboard/): the CTEM Command Center, the exposure of the organization across the XTM platform and what its agents do about it.
3. [Get answers about the platform's own activity](/xtm-one/use/platform-insights/): who uses AI, for what, at which cost.
:::
::::

## Agents that work on their own

::::split
:::column
An agent does not wait to be asked. An assignment runs it on a schedule, on a webhook from another tool or on an event of an integration, and its result lands where your team reads it: a brief in your inbox every Monday, a message in the SOC's channel, a file in OpenCTI. Several agents can hand work to each other, and the Agentic Flow shows them working together.

:::cards{cols=2}
  ::card[Schedules, webhooks and events.]{icon=Clock title="Assignments" href="/xtm-one/build/agents/"}
  ::card[Every run, its tools and its cost.]{icon=History title="Work history" href="/xtm-one/use/work-history/"}
:::
:::
:::column
::diagram{id=xtm-one-overview}
:::
::::

:::info[You keep the control]
An agent acts with the rights it was given, never more. A tool can be set to wait for a person's approval before it runs, prompts can be screened for injection, and an administrator can stop every agent at once. See [Protect your data and approve what agents do](/xtm-one/administer/ai-governance/).
:::

## Browse XTM One

::section-overview{id=xtm-one}
