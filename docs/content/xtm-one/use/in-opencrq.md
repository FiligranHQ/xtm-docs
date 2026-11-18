---
title: XTM One in OpenCRQ
description: What XTM One does in OpenCRQ, from Ask Ariane and its approvals to the CTEM Command Center and the MCP server, with whose rights an agent works, and what each edition brings.
doc_type: explanation
products:
  - xtm-one
  - opencrq
personas: [risk-manager, admin, ai-builder]
owner: docs-opencrq
last_reviewed: 2026-11-04
---

XTM One is the AI of the XTM platform, and OpenCRQ cannot run without it: OpenCRQ registers with XTM One when it starts, and the XTM license XTM One returns decides its edition. In OpenCRQ, XTM One answers your questions about risk in the **Ask Ariane** assistant, asks your approval before an agent changes something when a tool requires it, opens the **CTEM Command Center** of the XTM platform, and lets your own AI client work with OpenCRQ through its MCP server. Each time, an XTM One AI agent works with OpenCRQ as you. This page explains how the pieces connect, what each edition brings, and what an agent can reach.

::diagram{id=xtm-one-in-opencrq}

## What the AI does for your work

| Your goal | Where | Edition |
| --- | --- | --- |
| Ask why a scenario costs what it costs, or draft a new one | **Ask Ariane**, in the top bar | Enterprise |
| Let an agent change OpenCRQ, after your approval | The approval prompt of **Ask Ariane** | Enterprise |
| See the exposure of the whole XTM platform, beyond risk in money | **CTEM Command Center**, in the top bar | Community |
| Query or update OpenCRQ from Cursor, Claude Desktop or your own agent | The **XTM One MCP server** card of your profile | Community |

Every answer is written by a language model and can be wrong. Check a figure an agent gives against the scenario's own page before you put it in front of a board.

## Ask Ariane, the assistant

:::note
Ask Ariane requires the Enterprise Edition of OpenCRQ, which comes from the XTM license XTM One returns for your platform. Without it, the **Ask Ariane** button does not show.
:::

**Ask Ariane**, in the top bar, opens a chat with an XTM One agent about the risk data of your tenant. The panel opens at the side of the page, where you can widen it, or floating. It suggests a few questions to start from: **Summarize our compliance posture**, **Which controls have the biggest maturity gaps?**, **Help me draft a new risk scenario** and **Explain the financial impact of this risk**.

What the assistant knows when you ask:

- **The page you are on.** OpenCRQ sends the address of the page, without its query, so "Explain the financial impact of this risk" on a scenario is about that scenario.
- **The files you attach.** They go to XTM One with your message.

Everything else, the assistant reads from OpenCRQ as you, with the tools of its agent. An answer can also produce a slide deck, a report or a video, which you download from its card. Your browser never talks to XTM One directly: OpenCRQ relays each message, with a short-lived token it signs for you.

The agent that answers is the one XTM One binds to the assistant of OpenCRQ. When OpenCRQ registers, it declares its tasks, and an XTM One administrator binds agents to each; the one with the highest priority answers first. OpenCRQ declares the assistant (`global.assistant`), the text tasks every platform shares (`global.summarize`, `global.explain`, `global.fix_spelling` and others), and two risk tasks: `grc.control_gap_analysis` and `grc.risk_scenario_generator`. In this version, **Ask Ariane** is the screen of OpenCRQ that calls an agent.

### Approve what the assistant does

An XTM One administrator can make tools ask before they run, such as every tool that changes OpenCRQ. When the agent wants to run one, the answer stops, and the panel shows the tool and the values it would use, with what each one means. Nothing runs until you answer, in one of three ways:

| Answer | Effect |
| --- | --- |
| Approve it once | The tool runs. The next call asks again |
| Decline it | The tool does not run. Give a reason: the agent receives it and adapts |
| Approve it from now on | The tool runs, and stops asking you, including in runs nobody watches, such as scheduled ones |

Reloading the page keeps the question: the panel reads again what the agent is waiting for. OpenCRQ tells XTM One it can ask you only while it runs the Enterprise Edition. In the Community Edition there is no panel to ask in, and an XTM One administrator decides alone whether the tools of OpenCRQ run.

## What an agent can reach, and with which rights

An agent never holds an OpenCRQ account of its own. Each call carries your identity:

:::steps
1. When you send a message, OpenCRQ signs a short-lived token that names your email address, and passes your message to XTM One with it.
2. XTM One runs the agent. When the agent needs data, XTM One calls the OpenCRQ API with a token it signs for the same email.
3. OpenCRQ checks the signature against the keys XTM One publishes, finds the account with that email, and applies its tenants and capabilities.
:::

The consequences:

- **An agent sees what you see**, in the tenants you belong to.
- **An agent changes only what you could change.** Its tools read and change risk scopes, primary and supporting assets, risk scenarios and their loss estimates, controls and their postures, and the engine settings; they run a simulation; they read treatment plans, budget reviews, findings and threats; and, for an administrator, they manage users, groups, roles, invitations and tenants. A tool your capabilities do not allow fails as you would.
- **Your email must be the same in both products.** An account with no match in OpenCRQ gets no data. Use the same email for the first administrator of OpenCRQ as for the administrator of XTM One: see [Install OpenCRQ](/deploy/install/opencrq/#create-the-first-administrator).
- **What leaves OpenCRQ** is your message, the address of the page, the files you attach, and what the agent reads through the API. XTM One sends them to the language models its administrator configured.

## CTEM Command Center

**CTEM Command Center**, the radar icon of the top bar, opens XTM One in a new tab, on the exposure posture across the products of the XTM platform. It works in the Community Edition, as soon as OpenCRQ is configured with XTM One.

## Use OpenCRQ from your own AI client

XTM One exposes OpenCRQ to any client of the Model Context Protocol (MCP), such as Cursor, Claude Desktop or an agent your team builds. The client works with frameworks, controls, assessments and risk scenarios, as you. It works in the Community Edition, and nothing is installed in OpenCRQ: the server exists as soon as the platform is registered.

On your profile, the **XTM One MCP server** card gives the address of XTM One, with **Copy MCP endpoint URL**. XTM One serves the OpenCRQ tools at that address followed by `/mcp/opencrq`: the card shows `/mcp/opengrc`, which XTM One does not answer, so write `/mcp/opencrq` in the client. The client authenticates with a personal API key you create in XTM One, passed as a bearer token. **Manage in XTM One** opens the MCP page of your XTM One profile: the connection status and a client configuration to copy.

::screenshot{id=opencrq-xtm-one-mcp-card alt="The XTM One MCP server card of the OpenCRQ profile, with the MCP endpoint URL of XTM One, its copy button and Manage in XTM One."}

:::example[Rank Grand Meridian's scenarios from Cursor]
Claire Morel, Grand Meridian's cyber risk manager, prepares the budget review in Cursor. She creates a personal API key in XTM One, keeps it in the environment variable `XTM_ONE_API_KEY`, and adds OpenCRQ to `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "opencrq": {
      "url": "https://xtm-one.example.com/mcp/opencrq",
      "headers": {
        "Authorization": "Bearer ${env:XTM_ONE_API_KEY}"
      }
    }
  }
}
```

Asked "Which risk scenarios cost us more than $100K a year?", Cursor calls the OpenCRQ tools with her rights, and lists Ransomware on the Booking Platform, $254K in Grand Meridian France, and Data Breach on Front-Desk Card Payments, $225K in Grand Meridian Spain. Data Breach on Guest Records, at $33K, stays below the line.
:::

## Govern AI on the platform

This part is for administrators: what XTM One requires of OpenCRQ, and who decides what agents do.

**XTM One is required.** OpenCRQ does not start without the address of XTM One, the registration token and its own public URL. It registers when it starts and every 5 minutes. When XTM One stops answering, OpenCRQ keeps working, warns every user, and **Ask Ariane** is unavailable until XTM One answers again. A platform engineer sets the connection: see [Configure OpenCRQ](/deploy/configure/opencrq/#xtm-one).

**The edition follows the XTM license.** OpenCRQ holds no license of its own. At each registration, XTM One returns the XTM license that covers the platform, if any, and OpenCRQ verifies it: the Enterprise Edition brings **Ask Ariane** and more than one tenant. A platform with no such license runs the Community Edition, with the CTEM Command Center and the MCP server.

**In XTM One**, its administrators decide which agents answer the tasks of OpenCRQ, in which order, whether tools ask for approval, and how many agent runs each person may spend. [XTM One in OpenCTI](/xtm-one/use/in-opencti/#govern-ai-on-the-platform) describes these settings for OpenCTI; they work the same way for every registered platform.

## What's next

- [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/), to judge what an agent says about a price.
- [Install OpenCRQ](/deploy/install/opencrq/), with its registration with XTM One.
- [XTM One in OpenCTI](/xtm-one/use/in-opencti/), the same AI in the threat intelligence platform.
- [XTM One](/xtm-one/), to build and govern the agents.
