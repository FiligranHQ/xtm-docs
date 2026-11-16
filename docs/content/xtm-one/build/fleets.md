---
title: Run agents as a team with fleets
description: Group XTM One agents into a fleet that works in one workspace, read what it does on its page, switch its automations on or pause them, and choose which of its actions wait for a person.
doc_type: how-to
products: [xtm-one]
personas: [ai-builder, admin]
owner: docs-xtm-one
last_reviewed: 2026-11-09
edition: ee
---

A fleet is a team of agents. Its agents hand work to each other, and it works in one workspace, which keeps everything they produce. This page creates a fleet from agents you built, reads what its page shows, switches its automations on, and chooses which of its actions wait for a person's approval. The platform also ships ready-made fleets: you turn them on the same way.

::screenshot{id=xtm-one-build-fleets-list alt="The Fleets page: Create with a prompt and New fleet at the top right, the search, filters and sort above the cards, and one card per fleet, among them the built-in Threat-informed exposure response with its orchestrator, its members and its workspace."}

## Before you start

- The platform runs the Enterprise Edition. Otherwise **Fleets** only says that fleets need the agentic AI license.
- AI is switched on, for the fleet's agents to run. With AI off, you still open and read the fleets.
- The agents exist, and you manage them: see [Create an agent](/xtm-one/build/agents/). An agent has one home fleet, so an agent already in a fleet cannot join another one as a member.
- Any user can create a fleet: it is private to its creator until shared. Only an administrator makes a fleet **Company-managed fleet**, seen by every user.

## Create a fleet

Open :menu[Orchestration > Fleets]. The page lists the fleets you can see: yours, the ones shared with you, and the fleets the platform ships, marked **Built-in**. Search them, filter them by owner, workspace or orchestrator, and sort them by **Name**, **Status**, **Last run** or **Net value (30d)**.

:::steps
1. Click **New fleet**.
2. Fill **Name**, as people and agents will read it in the fleet list, and **Description (optional)**, what the fleet is for in a sentence or two.
3. In **Workspace**, keep **A new workspace named after the fleet**, or pick a workspace you manage. Everything the fleet's agents produce is kept there: its work items, shared state and knowledge.
4. In **Orchestrator**, pick the member that receives work from other fleets and hands it on. It is optional, and always exposed to other fleets.
5. In **Members**, pick the other agents whose home is this fleet. Only the agents you manage that are in no fleet are listed.
6. Optionally, in **Connected agents**, add agents outside the fleet that it works with. For each one, choose in **Conversation shared** what it receives with each task: **Nothing** (the default, the task only), **A summary** or **The full conversation**. Then write in **Collaboration instruction** when the fleet calls it and what to expect, for example `Ask for a review before any screen change ships.`
7. In **Exposed to other fleets**, pick the home members other fleets may call directly. Left on **Only the orchestrator**, they reach the orchestrator only.
8. Click **Create fleet**. The fleet's page opens, on **About**: XTM One writes the fleet's documentation from its members, in the background.
:::

::screenshot{id=xtm-one-build-fleets-new alt="The New fleet dialog filled by Sophie Laurent: Velvet Harbor response in Name, A new workspace named after the fleet in Workspace, Night shift triage as Orchestrator, and the Members and Connected agents fields below."}

A new fleet runs nothing yet: its members' automations stay as they were until you enable the fleet.

:::xtm-one[Set up a fleet in a chat interview]
**Create with a prompt** opens the chat on an assistant that manages fleets, and starts a setup interview at once. The assistant asks one question at a time: what the fleet is for, which agents it needs, its orchestrator, its workspace, then the integrations and knowledge its agents need. It proposes the whole configuration, and creates the fleet only once you confirm, as you, with the same checks as the dialog. Then it walks you through each prerequisite still missing, such as an integration to connect. Leave the chat and ask again later: it resumes at the next step the fleet needs. On a fleet's page, **Modify with AI** opens the same chat beside the page, to change that fleet.
:::

## Read a fleet's page

Under the fleet's name, the header shows its orchestrator, its workspace, and a status:

| Status | Means |
| --- | --- |
| **Not set up** | None of the fleet's automations is on. The built-in fleets start there, and a fleet [set back to not set up](#start-the-setup-over) returns there. |
| **Partially on** | Some of its automations are on. |
| **On** | All its automations are on. |
| **Paused** | Someone paused the fleet: none of its automations starts until it is resumed. |

**Archived** shows on an archived fleet, and a copy says which fleet it was duplicated from.

::screenshot{id=xtm-one-build-fleets-page alt="The page of the built-in fleet Threat-informed exposure response, as an administrator sees it: its orchestrator, status and workspace under its name, Modify with AI, Duplicate, Settings and Enable fleet at the top right, the About, Graph, Data flow, Members, Approval, Schedule, Operations and ROI tabs, and About opening on Written by Filigran, shipped with the platform."}

| Tab | What it is for |
| --- | --- |
| **About** | What the fleet does, how its members work together, what it needs before it runs and about how many runs a month it takes. A built-in fleet's is written by Filigran; yours is generated by AI, and **Regenerate documentation** brings it up to date after a change. A generation keeps running if you leave the page. **Version history** and **Export PDF** are beside it. |
| **Graph** | The members and the handovers between them, with the agents of other fleets they work with, and what runs now. Whoever manages the fleet draws a handover between two members, or drags a member onto another to make it report to it. |
| **Data flow** | What starts each member, and where its work goes. |
| **Members** | One row per member and connected agent, with its settings: inherited from the fleet, or its own. |
| **Approval** | Which of the fleet's actions wait for a person: see [Choose what waits for a person](#choose-what-waits-for-a-person). |
| **Schedule** | What the fleet will do, and when. |
| **Operations** | The fleet's runs, delegations, work items, shared state and worksets. |
| **ROI** | What the fleet's work is worth against what it costs, for the people who manage it. See [Follow what agents do and what they cost](/xtm-one/administer/observability/). |

## Switch its automations on

A fleet is opt-in: opening it, even a built-in one, switches nothing on. The fleet's owner and the people it is shared with for editing enable it; a built-in fleet, only an administrator.

:::steps
1. On the fleet's page, click **Enable fleet**. When some automations are already on, the button reads **Enable N more**.
2. In **Run as**, keep yourself, or, as an administrator, pick another user, such as a service account. The automations use that user's connected modules, model and memory.
3. Untick the automations to leave off. An automation marked **Needs OpenCTI**, or another product, needs a module the user has not connected: connect it from **Open Integrations**, run as another user, or leave the automation out.
4. Check the estimate, from the fleet's documented triggers, of how many runs a month it takes, and click **Enable N automations**.
:::

**Pause all** switches off every automation of the fleet that is on; the runs already started finish. **Resume** switches back on exactly those. On the Fleets page, select several fleets you manage to pause or resume them at once.

:::warning
**Pause every fleet automation**, on the Fleets page, is for administrators: it pauses the automations of every fleet at once, and the runs in flight continue. To stop every agent and every run now, use the emergency stop: [Stop every agent at once](/xtm-one/administer/emergency-stop/).
:::

### Start the setup over

A fleet still being built can read **Paused** or **Partially on** after a few tries. To start over, ask the assistant of **Modify with AI** to set the fleet back to not set up. It says how many automations it would switch off, and acts only once you confirm. Every automation is then off and the fleet reads **Not set up** again, but nothing is deleted: **Enable fleet** switches them back on. The runs already started finish, and an automation an emergency stop holds stays as it is. The same people as for **Pause all** may do it: on a built-in fleet, an administrator only.

## Choose what waits for a person

On **Approval**, **Human approval** sets which of the fleet's actions wait for a person before they run. The tab lists those actions, and which wait at each level.

| Level | What waits |
| --- | --- |
| **Full human gates** | Every action that writes outside XTM One waits for approval. Drafts and reads need none. The level of every fleet you create. |
| **Medium** | Low-impact writes run without approval; launches and deployments still wait. |
| **Low** | Only destructive actions and actions on production assets wait. The level the built-in fleets ship with. |
| **Full autonomous** | Nothing waits for approval. Every action is still logged and reported in **Operations**. |

Whoever manages the fleet can make it more supervised; only an administrator makes it more autonomous. To set the level of several fleets at once, select them on the Fleets page and click **Human approval...**.

## Work with other fleets

- Another fleet reaches yours through its orchestrator, and through the members you add to **Exposed to other fleets**. A handover from another fleet's agent to a member you do not expose is refused.
- A connected agent stays in its own fleet, or in none: your fleet calls it on the terms of its **Conversation shared** and **Collaboration instruction**. The **Members** tab sets these for several connected agents at once.
- On **Graph**, the agents of other fleets are drawn apart from your members, and a dotted line marks a handover to an agent its fleet does not expose.

## Change, copy or delete a fleet

- **Settings** opens **Fleet settings**: the name, description, icon and colour, the workspace, the orchestrator, **Exposed to other fleets**, **Visibility and sharing**, and **Run defaults**, what one run of a member may spend when the agent sets no limit of its own (the agent's own limit wins, see [Set what one agent run may spend](/xtm-one/use/run-budgets/)).
- **Duplicate** makes a copy you own and can change, of any fleet you see, built-in ones included. Duplicate a built-in fleet to adapt it: the built-in one stays as Filigran ships it.
- **Archive the fleet**, at the bottom of **Fleet settings**, hides it from the Fleets page and freezes its members. Turn on **Show archived** in the page's filters to find it, then **Restore the fleet** brings it back.
- **Delete the fleet** deletes no agent: the members go back to being in no fleet, with their handovers and assignments, and the workspace is kept.

:::example[Put the Velvet Harbor agents in one fleet]
Sophie Laurent, the AI engineer of Grand Meridian Hotels, built three agents around Velvet Harbor: **Night shift triage** consults **Velvet Harbor watcher** and passes a crisis to **Crisis briefer**. She wants them to work as one team, in one place.

In :menu[Orchestration > Fleets] she clicks **New fleet** and names it **Velvet Harbor response**, described as "Triages the SOC's Velvet Harbor alerts, keeps the CTI team informed and briefs the crisis cell." She keeps **A new workspace named after the fleet**, picks **Night shift triage** as **Orchestrator**, and adds **Velvet Harbor watcher** and **Crisis briefer** in **Members**. She clicks **Create fleet**.

The fleet stays at **Full human gates**: whatever its agents would write outside XTM One waits for a person. Its **Operations** tab now gathers the runs of the three agents, and the CTI team's Monday brief keeps running as before.
:::

## What's next

- [Follow how agents hand work to each other](/xtm-one/use/agentic-flow/), fleets included.
- [Share a workspace with your team](/xtm-one/use/workspaces/): the fleet's workspace keeps what its agents produce.
- [Read the home dashboard](/xtm-one/use/dashboard/): the **Fleets** row says which CTEM fleets are on.
