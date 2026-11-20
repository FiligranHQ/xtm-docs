---
title: Run agents as a team with fleets
description: Group XTM One agents into a fleet that works in one workspace, read what it does on its page, switch its automations on or pause them, and choose which of its actions wait for a person.
doc_type: how-to
products: [xtm-one]
personas: [ai-builder, admin]
owner: docs-xtm-one
last_reviewed: 2026-11-19
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

Open :menu[Orchestration > Fleets]. The page lists the fleets you can see: yours, the ones shared with you, and the fleets the platform ships, marked **Built-in**, [Platform Health's fleet](#read-platform-health-s-fleet) among them. Search them, filter them by owner, workspace or orchestrator, and sort them by **Name**, **Status**, **Last run** or **Net value (30d)**.

The page opens on **Mine** when you own a fleet. Under **Company**, the built-in fleets are hidden once your organization has company-managed fleets of its own, and the tooltip of **Company** says how many are hidden. Tick **Built-in** in the **Properties** filter to list them again: your browser remembers the choice for this list. A search that only matches built-in fleets lists them anyway. At the right of the toolbar, a line counts the agents in no fleet, and its link **Show in Agents** lists them.

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
**Create with a prompt** opens the chat on an assistant that manages fleets, and starts a setup interview at once. The assistant asks one question at a time: what the fleet is for, which agents it needs, its orchestrator, its workspace, then the integrations and knowledge its agents need. It proposes the whole configuration, and creates the fleet only once you confirm, as you, with the same checks as the dialog. In the same step, it can give each member its role and the member it reports to, and create the fleet at the [human approval level](#choose-what-waits-for-a-person) you asked for rather than **Full human gates**: check it on **Approval** once the fleet exists. Then it walks you through each prerequisite still missing, such as an integration to connect. Leave the chat and ask again later: it resumes at the next step the fleet needs. On a fleet's page, **Modify with AI** opens the same chat beside the page, to change that fleet.
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
| **About** | What the fleet does, how its members work together, what it needs before it runs and about how many runs a month it takes. A built-in fleet's is written by Filigran; yours is generated by AI, and **Regenerate documentation** brings it up to date after a change. A generation keeps running if you leave the page. **Version history** and **Export PDF** are beside it. Whoever manages the fleet also sees its suggestions there: see [Act on the fleet's suggestions](#act-on-the-fleet-s-suggestions). |
| **Graph** | The members and the handovers between them, with the agents of other fleets they work with, the partners they work with outside the platform ([See its A2A partners](#see-its-a2a-partners)), and what runs now. Whoever manages the fleet draws a handover between two members, or drags a member onto another to make it report to it. |
| **Data flow** | What starts each member, and where its work goes. |
| **Members** | One row per member and connected agent, with its settings: inherited from the fleet, or its own. |
| **Approval** | Which of the fleet's actions wait for a person: see [Choose what waits for a person](#choose-what-waits-for-a-person). |
| **Schedule** | What the fleet will do, and when. |
| **Operations** | The fleet's runs, delegations, work items, shared state and worksets. |
| **ROI** | What the fleet's work is worth against what it costs, for the people who manage it. To someone who is not an administrator, when fewer people than the **Smallest group shown apart** set in Settings used the fleet in the period, the tab shows only their own use of it, or says that the totals are hidden. See [Follow what agents do and what they cost](/xtm-one/administer/observability/). |

### Read Platform Health's fleet

**Platform Health** is the fleet of the platform's built-in supervisor: its orchestrator, also named Platform Health, and the Memory Steward, Quality Analyst, Flow Lab and Experimenter, working in a company-managed workspace also named Platform Health. Every user sees it, and nobody changes it, administrators included. Its header shows **Read-only**, whose tooltip reads "Platform Health is built in and read-only.", and **Settings** is its only action: no **Modify with AI**, no **Duplicate**, no **Enable fleet** and no **ROI** tab. **Fleet settings** open with every field locked, and **About** lists the fleet's prerequisites without a fix or a **Set up**. The platform writes the fleet's whole definition again at every start, so a change would not last anyway.

Its automations start from Platform Health's own cycles, never from this page: an administrator turns on **Enable Platform Health**, off by default, and sets its schedules and budget in :menu[Administration > Settings > Agents and automations > Platform Health]. See [Set up the platform in Settings](/xtm-one/administer/settings/#agents-and-automations).

::screenshot{id=xtm-one-build-fleets-platform-health alt="The page of Platform Health's built-in fleet, as an administrator sees it: the Built-in and Read-only badges under its name, Settings as the only action at the top right, no Duplicate, Modify with AI or Enable fleet, and the tabs up to Operations, without ROI."}

## Switch its automations on

A fleet is opt-in: opening it, even a built-in one, switches nothing on. The fleet's owner and the people it is shared with for editing enable it; a built-in fleet, only an administrator; Platform Health's fleet, nobody from its page.

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
| **Full human gates** | Every action that writes outside XTM One waits for approval. Drafts and reads need none. The level of every fleet created with **New fleet**. |
| **Medium** | Low-impact writes run without approval; launches and deployments still wait. |
| **Low** | Only destructive actions and actions on production assets wait. The level the built-in fleets ship with. |
| **Full autonomous** | Nothing waits for approval. Every action is still logged and reported in **Operations**. |

Whoever manages the fleet can make it more supervised; only an administrator makes it more autonomous. To set the level of several fleets at once, select them on the Fleets page and click **Human approval...**.

Under **Waiting for approval now**, the tab lists the runs of the fleet's members paused until a person decides. Click a row, or its **Review**, to decide there; **Open run** in the decision panel opens the whole run.

## Act on the fleet's suggestions

Whoever manages the fleet sees, in the hero of **About**, a chip that counts its suggestions. The platform checks good practices each time the page opens, without calling a model, and adds what the AI noted while writing the fleet's documentation. A fleet without an orchestrator gets no good-practice suggestion.

| Suggestion | What it checks |
| --- | --- |
| **Member cannot consult the orchestrator** | A member has no Consult handover to the orchestrator, for a route, scope or context question. |
| **Member open to chat** | A member other than the orchestrator takes chats: people usually reach a fleet through its orchestrator. |
| **Work items without an inbox** | A member owns open work items, and none of its automations is triggered by its work items. |
| **Orchestrator without listeners** | The orchestrator has no automation started when a work item completes, or a run fails or is orphaned. |
| **No working knowledge base** | No knowledge base is written by the orchestrator and read by every member. |
| **Orchestrator cannot write the knowledge base** | The whole fleet reads a knowledge base the orchestrator cannot write. |
| **Member without the working knowledge base** | A member does not read the knowledge base the orchestrator writes for the fleet. |
| **Knowledge base with several writers** | Members write the fleet's knowledge base besides the orchestrator. |

:::steps
1. On the fleet's page, open **About** and click the suggestions chip. **Suggestions for this fleet** lists each one with the member it concerns and a suggested fix.
2. Click **Apply** to make the change in one click: it shows what changes, and you confirm with **Apply**. Three suggestions offer it: a missing Consult handover, write access for the orchestrator, and a member to bind, read-only, to the fleet's knowledge base. The change is recorded in the member's version history.
3. For another suggestion, click **Fix with AI** to ask the assistant in the chat, or **Open** to go to the agent it concerns. **Fix all with AI** sends every suggestion in one request.
4. For a suggestion that needs nothing, click **Dismiss**, pick a **Reason (optional)**: **Standing decision**, **False positive**, or **Other reason** with one line of your own. **Dismiss all** dismisses every suggestion with the same reason.
:::

A dismissal holds while the member concerned is unchanged, for 90 days at most. **Show dismissed (N)** lists who dismissed what, when and why, and **Restore** brings a suggestion back. A suggestion you fix goes away the next time the page opens.

::screenshot{id=xtm-one-build-fleets-suggestions alt="Suggestions for this fleet on the built-in fleet Threat-informed exposure response: each suggestion names its kind and member, the finding and the suggested fix, with Fix with AI or Apply, Open and Dismiss below, and Show dismissed, Fix all with AI and Dismiss all at the top."}

### Let the orchestrator check the fleet's health every week

Every fleet orchestrator gets an automation named "Weekly fleet health check", switched off until you enable the fleet: **Enable fleet** lists it with the others. It runs on Mondays at 07:00 UTC, and only when the fleet has a health finding, so a healthy fleet costs no model call. The findings include a member whose latest run failed, a schedule silent for 14 days, an overdue work item, a shared state entry over its size limit, and a member that used more than three times its daily tokens.

The orchestrator releases the work items still claimed by runs that ended, on its own. For everything else, it sends the fleet's owner one advisory, through the channel its automation names or else as a work item, and does not repeat a finding already sent. When a finding needs nothing, such as a cost spike you accept, tell the assistant of **Modify with AI**, with your reason: it dismisses the finding, and the weekly check no longer counts it. A finding dismissed alone comes back when its evidence changes.

To do without the weekly check, delete its automation, or ask the assistant of **Modify with AI** to switch the fleet's health check off: either way, XTM One does not create it again. The assistant can also make an automation of one of the fleet's members its health check instead, or bring the weekly check back.

## Work with other fleets

- Another fleet reaches yours through its orchestrator, and through the members you add to **Exposed to other fleets**. A handover from another fleet's agent to a member you do not expose is refused.
- A connected agent stays in its own fleet, or in none: your fleet calls it on the terms of its **Conversation shared** and **Collaboration instruction**. The **Members** tab sets these for several connected agents at once.
- On **Graph**, the agents of other fleets are drawn apart from your members, and a dotted line marks a handover to an agent its fleet does not expose.

### See its A2A partners

An agent of the fleet can work with an agent outside the platform through an agent-to-agent (A2A) integration. On **Graph**, these partners are drawn outside the fleet, each marked **Inbound A2A partner** or **Outbound A2A partner**, and a pink dash-dot line joins each one to its agent:

- **Tasks** runs from an inbound partner to the agent: the partner sends the agent work.
- **Delegates** runs from the agent to an outbound partner: the agent hands work to that partner agent. An outbound integration gets one node per partner agent it lists, or a single node while it lists none.

A partner's tooltip names its integration and the addresses of its agent cards. Click a partner to open its integration, and **Back to the fleet** to come back. These lines are not handovers: you cannot draw, change or remove one on the graph. They come from binding the integration to the agent, on the agent's **Configuration** > **Tooling** or on the integration's page: see [Connect agents to your tools with integrations](/xtm-one/build/integrations/). The graph shows only the integrations you can see, and no partner while AI is off.

## Change, copy or delete a fleet

- **Settings** opens **Fleet settings**: the name, description, icon and colour, the workspace, the orchestrator, **Exposed to other fleets**, **Visibility and sharing**, and **Run defaults**, what one run of a member may spend when the agent sets no limit of its own (the agent's own limit wins, see [Set what one agent run may spend](/xtm-one/use/run-budgets/)).
- To share the fleet and its workspace in one step, ask the assistant of **Modify with AI**: name the people, by email, or the groups, and whether they view or edit. It adds them to both, and can also remove people, replace the whole list, or say who has access. Only an administrator, or the owner of the fleet's workspace, shares this way. A company-managed fleet that every user sees needs no sharing.
- An administrator can also ask the assistant of **Modify with AI** to make the fleet company-managed, or personal again. It refuses while a home member or the fleet's workspace is personal, and names them: make them company-managed first.
- **Hide activity from non-admins**, in **Fleet settings**, is for administrators, on a company-managed fleet: the runs, traces, work items and shared state of its members and of its workspace become visible to administrators only. An administrator can still change it on a built-in fleet, whose other settings are read-only, except on Platform Health's fleet.
- **Duplicate** makes a copy you own and can change, of any fleet you see, built-in ones included, except Platform Health's. Duplicate a built-in fleet to adapt it: the built-in one stays as Filigran ships it.
- **Archive the fleet**, at the bottom of **Fleet settings**, hides it from the Fleets page and freezes its members. Turn on **Show archived** in the page's filters to find it, then **Restore the fleet** brings it back.
- **Delete the fleet** deletes no agent: the members go back to being in no fleet, with their handovers and assignments, and the workspace is kept.

:::example[Put the Velvet Harbor agents in one fleet]
Sophie Laurent, the AI engineer of Grand Meridian Hotels, built three agents around Velvet Harbor: **Night shift triage** consults **Velvet Harbor watcher** and passes a crisis to **Crisis briefer**. She wants them to work as one team, in one place.

In :menu[Orchestration > Fleets] she clicks **New fleet** and names it **Velvet Harbor response**, described as "Triages the SOC's Velvet Harbor alerts, keeps the CTI team informed and briefs the crisis cell." She keeps **A new workspace named after the fleet**, picks **Night shift triage** as **Orchestrator**, and adds **Velvet Harbor watcher** and **Crisis briefer** in **Members**. She clicks **Create fleet**.

The fleet stays at **Full human gates**: whatever its agents would write outside XTM One waits for a person. Its **Operations** tab now gathers the runs of the three agents, and the CTI team's Monday brief keeps running as before.

Once XTM One has written the documentation, **About** shows a suggestions chip. **Member cannot consult the orchestrator** says that Crisis briefer cannot consult Night shift triage: she clicks **Apply**, reads that Crisis briefer gets a Consult handover, and confirms. **Member open to chat** says that Velvet Harbor watcher takes chats: the CTI analysts ask it questions directly, so she clicks **Dismiss** with the reason **Standing decision**.
:::

## What's next

- [Follow how agents hand work to each other](/xtm-one/use/agentic-flow/), fleets included.
- [Share a workspace with your team](/xtm-one/use/workspaces/): the fleet's workspace keeps what its agents produce.
- [Read the home dashboard](/xtm-one/use/dashboard/): the **Fleets** row says which CTEM fleets are on.
