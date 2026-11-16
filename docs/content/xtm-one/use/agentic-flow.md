---
title: Follow how agents hand work to each other
description: Read the Agentic Flow page of XTM One, the live map of how agents delegate, consult and transfer work to each other, and the schedule of everything that will run and when.
doc_type: how-to
products: [xtm-one]
personas: [ai-builder, soc-manager]
owner: docs-xtm-one
last_reviewed: 2026-11-10
migrated_from:
  - xtm-one:/usage/agentic-flow/
---

AI agents in XTM One rarely work alone: a triage agent asks the CTI agent for context, a briefing agent hands a long analysis to the background, an assistant passes the conversation to a specialist. **Agentic Flow** shows that cooperation for the whole platform. Its **Agent-to-agent map** draws the agents you can see and the handovers between them, with what is running right now; its **Schedule** view lists everything that will start on its own, and when. This page reads both, and changes a handover from the map. How an agent's handovers are set on the agent itself is in [Create an agent](/xtm-one/build/agents/).

::diagram{id=xtm-one-agentic-flow-handovers}

## Before you start

- You see the agents shared with you, and the handovers between them.
- To create, change or remove a handover, you can edit the agent the handover starts from (its owner, or someone it is shared with for editing), and AI is switched on. The arrows of an agent you cannot edit are read-only.

## How agents hand work over

An agent hands work to another agent, called a handover agent, in one of four ways. The type decides what the person in the conversation sees.

| Type | What happens | Use it when |
| --- | --- | --- |
| **Delegate** | The work runs in the background, as a task of the other agent. The first agent goes on, and gets the result when the task ends | The work is long or can run in parallel: the chat shows a background task banner |
| **Consult** | The first agent asks the other one a question, waits for the answer and stays in charge of the conversation | It needs an expert's answer to continue |
| **Transfer** | The conversation moves to the other agent, which answers from then on | The request belongs to a specialist |
| **Mandatory Transfer** | After its own answer, the first agent always hands the conversation to the other one | A step must always be followed by another agent |

An agent configured before handover types existed, or through the API without types, shows each of its handover agents as **All types**: it may delegate to, consult or transfer to each one. Types are set per agent, for all its handover agents at once: when you give one of them a type, the others become **Delegate**, and XTM One names them and asks you to confirm first (**Give each agent a handover type?**). Once an agent's handovers have types, they never switch back to **All types** on their own.

An agent can also delegate to itself: faced with a large workload, it splits the work across copies of itself that run in parallel. This is on by default. To keep an agent from doing it, turn off **Parallel copies of itself** in the **Agent Handover** card of its configuration. An administrator can switch it off for every agent with `AGENT_SELF_FANOUT_ENABLED`: see the [configuration reference](/reference/xtm-one/configuration/).

## Read the map

Open :menu[Orchestration > Agentic Flow]. The page opens on the **Agent-to-agent map**, in its **Graph** view; **Schedule**, next to it, is the [other view](#read-the-schedule). The **i** next to the title, **About Agentic Flow**, says what the page is for.

::screenshot{id=xtm-one-agentic-flow-map alt="The Agent-to-agent map of Sophie Laurent on Mine: the toolbar with Layout, the Mine, Company, Shared and All segments, Forces, Filters, Focus and Legend; the graph of the General Assistant and her three agents, with the Consult arrow from Night shift triage to Velvet Harbor watcher and the Transfer arrow to Crisis briefer; the status line under the graph, then the Activity stream."}

- **The graph** draws each agent, the General Assistant included, and an arrow for each handover, labelled with its type. A running agent pulses, and the arrow of a recent handover lights up. A disabled agent is greyed. Drag the background to pan; the mouse wheel or a pinch zooms where the pointer is. The buttons in the corner are **Zoom in**, **Zoom out**, **Fit to view** and **Reset view**. When the names are too small to read, hover an agent or zoom in.
- **The line under the graph** counts what the toolbar keeps: the **agents**, the **fleets** when there are any, the **handovers** drawn, the **runs in flight**, the **handovers in flight** and the **open work items**.
- **Activity stream**, under the graph, lists the runs, handovers and work items as they happen, with their total next to its title. **Open Operations** opens the full history: see [Review what agents did](/xtm-one/use/work-history/).

Click an agent to open its panel: whether it is enabled, its assignments, the agents it **Hands off to**, its **Capabilities** (knowledge bases, integrations, MCP servers, custom tools, prompts, skills) and its **Recent runs**. `Ctrl`+click an agent (`Cmd`+click on a Mac) to open its page. Drag an agent to move it: it stays where you dropped it, marked with a dot, until you change the layout or click **Unpin agents**.

### Choose what the map draws

The toolbar above the graph narrows and arranges the map. The map keeps your choices for your next visit, and writes them in the page's address: a link to it opens the same view.

| Control | What it does |
| --- | --- |
| **Mine**, **Company**, **Shared**, **All** | Keeps the agents you own, those published to the whole company, those shared with you directly or through a group, or every agent you can see. Each shows how many agents it holds; the map first opens on the first one that holds an agent. The General Assistant is always drawn |
| **Filters** | Keeps an **Agent type** (**Built-in**, **Company-managed**, **Personal**), the **Enabled** or **Disabled** agents, and the agents **With open work items** or **With runs in flight**. Its badge counts the filters on; **Clear filters** removes them |
| **Layout** | Arranges the agents: **Force** (the default), **Top down**, **Left to right**, **Radial** or **Org chart** |
| **Forces** | Sets how the agents spread: **Repulsion**, **Link distance** and **Node spacing** |
| **Fleets** | Shows when an agent belongs to a fleet you can open, and is on: the agents of a fleet are drawn as one node, which opens the fleet when you click it. Turn it off to draw every agent |
| **Legend** | Explains the shapes, colours and arrows of the map |

### Focus on a few agents

On a large platform, type an agent's name in the **Focus** field, or double-click the agent on the graph; double-click a fleet to focus on its agents. Add several to compare them; **Clear** removes them all. Then choose how the focus shows:

- **Show only** (the default) lays out the focused agents alone, with every agent they hand work to or receive work from. Double-click one of those to bring its own handover agents in, one step at a time.
- **Highlight** keeps the whole graph and dims every agent outside the focus.

A focused agent that is no longer on the graph is ignored; when none is, the whole graph shows.

## Change a handover from the map

:::steps
1. To create a handover, press the connect handle at the bottom right of the agent that will hand the work over (**Drag to an agent to start a handover**), and drag it onto the agent that takes it. You can also click the handle, then the target agent; `Esc` cancels. Only an agent you can edit has a handle. Both agents must be on the graph: add the second one to the focus, or switch to **Highlight**.
2. In **Select handover type**, pick **Delegate**, **Consult**, **Transfer** or **Mandatory Transfer**, then click **Save**. The General Assistant cannot take a handover.
3. To change a handover, click the label of its arrow. In **Edit handover type**, pick another type and click **Save**, or click **Delete** to remove the handover.
:::

A handover belongs to the agent it starts from, so the change applies to everyone who uses that agent, in every conversation and assignment. The same handovers are in the **Agent Handover** card of the agent's configuration.

## Read the Schedule

Click **Schedule** above the map. This view answers "what will start, and when?" for the agents the toolbar keeps: the segment, the **Filters** and the **Focus** of the graph apply to it too, so **All** with no filter and no focus covers every agent you can see.

::screenshot{id=xtm-one-agentic-flow-schedule alt="The Schedule view of the Agent-to-agent map, on All: 7 scheduled assignments, no one-off action, the next trigger in 19h 59m for the Weekly Velvet Harbor brief, the Upcoming timeline of six weekly assignments, and the Recurring list, which starts with the Weekly Velvet Harbor brief of Velvet Harbor watcher at 06:00 AM, only on Monday."}

- **The counters**: **Scheduled** (assignments on a schedule), **One-off** (scheduled actions waiting), **Next trigger** (a countdown to the next scheduled run), **In flight** (queued or running), **Next 24h** (expected triggers) and **Event-driven** (assignments that start on a webhook or an integration event).
- **Upcoming** places what will run on a timeline: choose **Next hour**, **Next 24h** or **Next 7d**.
- **The inventory** lists the **One-off** actions people scheduled from a chat, then the assignments by kind: **Recurring**, **Event-driven** and **Manual**. Click a row to open the assignment, or the one-off action with **Cancel** and **Reschedule**.

An agent's **Coming up** card, on its page, opens this view narrowed to that agent; **Show every agent** widens it again. How one-off actions are scheduled is in [Let an agent act later, at a time you choose](/xtm-one/use/work-items/); how an assignment starts on an event, in [Start an agent from another tool with a webhook](/xtm-one/use/webhook-triggers/).

:::example[Let the triage agent ask the CTI agent before it answers]
Sophie Laurent, the AI engineer of Grand Meridian Hotels, wants **Night shift triage** to check with **Velvet Harbor watcher** before telling Paul Garnier whether an alert matches the January intrusion. She opens :menu[Orchestration > Agentic Flow]: on **Mine**, the map draws her three agents and the General Assistant.

| Action | Value |
| --- | --- |
| Drag | The connect handle of **Night shift triage** onto **Velvet Harbor watcher** |
| **Select handover type** | **Consult**, then **Save** |

The arrow `Consult` appears between the two agents. **Night shift triage** already transferred to **Crisis briefer** with **All types**, so XTM One asks **Give each agent a handover type?** and names **Crisis briefer**, which will become **Delegate**. Sophie clicks **Set types**, then changes that arrow to **Transfer**, so the crisis cell talks to **Crisis briefer** directly. The next night, the arrow between the triage and the CTI agent lights up when Paul asks about an alert, and the answer arrives in his conversation without a background banner. In the **Schedule** view, **Weekly Velvet Harbor brief** shows under **Recurring**, next on Monday 13 April 2026 at 08:00.
:::

## What's next

- [Create an agent](/xtm-one/build/agents/), to set an agent's handovers and assignments.
- [Review what agents did](/xtm-one/use/work-history/), to follow a delegated task to its end.
- [Set what one agent run may spend](/xtm-one/use/run-budgets/), to bound a run that delegates a lot.
