---
title: Put XTM One agents to work on your intelligence
sidebar_label: AI-assisted intelligence
description: Let XTM One agents label and summarize each new OpenCTI report inside a playbook, keep a weekly brief running, and review everything they did, in both modules.
doc_type: journey
journey: ai-assisted-intelligence
products:
  - xtm-one
  - opencti
personas: [cti-lead, ai-builder, cti-analyst]
owner: docs-platform
last_reviewed: 2026-11-09
---

At the end of this journey, XTM One AI agents do part of your CTI team's reading: inside an OpenCTI playbook, one agent labels each new report that is about your sector and another writes its summary for the team, while an assignment writes a brief every week. Every agent works with the rights of a real account, only the agents an administrator bound can be called, and every run leaves a trace in both modules. The CTI team lead drives it, with the AI agent builder who prepares the agents. The steps below are the pages to follow, in order.

::diagram{id=journey-ai-assisted-intelligence}

## Who takes part

- **CTI team leads** own the playbook: they build it, choose the account it runs as, start it and watch its runs (steps 5 and 6).
- **AI agent builders** build the summary agent and its weekly assignment, share it with the CTI team, and, as XTM One administrators, bind it so that playbooks can call it (steps 2 to 4).
- **CTI analysts** receive the labelled and assigned reports and the summaries, and rate the weekly brief (step 7).
- An XTM One administrator registers OpenCTI with XTM One once (step 1).

## Why it spans several modules

OpenCTI holds the intelligence and decides when work starts: a playbook listens to new reports, filters them and changes them. XTM One holds the agents: their instructions, their tools, their language models and who may use them. A playbook step hands its Structured Threat Information Expression (STIX) bundle to an agent and, for a transformation, carries on only with a valid STIX 2.1 bundle back.

Each side keeps its own control. In XTM One, an intent says which job a module asks for and a binding says which agent answers it: **Transform with AI agent** asks for `cti.stix_transformer`, **Send to AI agent** for `cti.stix_consumer`, and the built-in **CTI STIX Transformer** and **CTI STIX Consumer** answer them by default. In OpenCTI, the **Run as** account of the step gives the agent its rights, never rights of its own. Read [XTM One in OpenCTI](/xtm-one/use/in-opencti/) for everything XTM One does in OpenCTI, and [Choose which agent answers each AI feature](/xtm-one/administer/intents/) for how bindings work.

## What you need

- **OpenCTI** 7.260811 or later, with an Enterprise Edition license: playbooks and their AI agent steps require it.
- **XTM One** in the Enterprise Edition, with AI switched on, and OpenCTI registered with it. The AI agent steps only show in the playbook editor once OpenCTI is configured with the address and token of XTM One. See [Connect the modules to each other](/deploy/configure/connect-modules/) and [Configure XTM One](/deploy/configure/xtm-one/).
- A service account the agents work as, such as `playbook-automation`: left empty, **Run as** makes the agent act as the platform's initial administrator. See [Create accounts for integrations](/opencti/administer/service-accounts/).
- Capabilities: in OpenCTI, **Manage Playbooks** for the team lead; in XTM One, an administrator account for the bindings.

## How it unfolds

The story: since Velvet Harbor reached the front desks of Grand Meridian Hotels on 9 January 2026, Example CERT and other partners publish several reports a day on the hospitality sector. By April 2026, Camille Roux, CTI analyst, misses the ones that arrive without the `hotel-sector` label. Sophie Laurent, the AI engineer, builds the agents; the CTI team lead owns the playbook.

### Prepare the agents in XTM One

Once OpenCTI is registered, **Ask Ariane** shows in its top bar and the AI agent steps in its playbook editor ([step 1](/xtm-one/use/in-opencti/)). The labelling needs no new agent: the built-in **CTI STIX Transformer** returns a valid bundle and follows the instructions the playbook gives it. Sophie builds `Hotel report digest`, with the OpenCTI integration narrowed to the tools that search, read and create notes, and gives her **Velvet Harbor watcher** the assignment **Weekly Velvet Harbor brief**, every Monday morning ([step 2](/xtm-one/build/agents/)). She shares the digest agent with the CTI team group, because XTM One refuses to bind a private agent for everyone ([step 3](/xtm-one/build/sharing/)), and binds it to `cti.stix_consumer` for the `cti` vertical ([step 4](/xtm-one/administer/intents/)).

### Hand the triage to the playbook

The team lead builds `Triage of hotel reports` ([step 5](/opencti/use/automate/playbooks/create/)): it listens to new reports, lets the transformer add `hotel-sector` when a report is about hotels, keeps only the labelled reports, sets them in progress and assigns them to Camille, and ends with **Send to AI agent**, which hands the report to `Hotel report digest` for a five-line summary added as a note. Both AI steps run as `playbook-automation`.

When a partner publishes a report on a phishing kit that mimics hotel booking confirmations, without a label, the transformer adds `hotel-sector`, the report is assigned to Camille, and the digest agent writes its summary. When the transformer cannot run, or returns no valid bundle within five minutes, the report leaves by `unmodified` and the branch ends: nothing is lost, and nothing is labelled by mistake.

### Check what the agents did

Each part leaves its trace where it runs. A playbook step calls its agent as a chat message of its **Run as** account, so its record is in OpenCTI: the execution traces of the playbook show the bundle each step received and the output it took ([step 6](/opencti/use/automate/playbooks/monitor/)). The weekly brief is an assignment, so its record is in XTM One: **Operations** lists each of its runs with its result, and Camille rates a result to tell the agent's managers ([step 7](/xtm-one/use/work-history/)). Each run uses the XTM One quota of the account it runs as: the team lead watches the first week of runs before widening the playbook to other sectors.

:::xtm-one[The agents where analysts work]
The same agents answer the analysts outside the playbook. **Ask Ariane**, in OpenCTI, answers questions on the page the analyst is on, such as "Is this IP linked to a known intrusion set, and since when?". In Cursor, Claude or VS Code, the OpenCTI server of XTM One lets Camille search and update the knowledge with her own rights. See [Use XTM One's own MCP servers from other clients](/xtm-one/build/platform-mcp-servers/).
:::

## The steps

Each step opens its page; the first one starts the journey, and every page of it then shows where you are.

::journey-steps

## Where to go next

- [Automate threat intelligence](/workflows/automate-intelligence/), for the playbooks that filter, enrich and share intelligence without an agent.
- [Build an AI agent your team can trust](/workflows/build-an-agent/), to build your own agents with knowledge bases, skills and budgets.
- [Let an agent act later, at a time you choose](/xtm-one/use/work-items/), for work to do once, such as "check the open incidents on Friday at 17:00".
