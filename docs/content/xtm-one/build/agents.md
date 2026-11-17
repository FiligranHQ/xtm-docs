---
title: Create an agent
description: Build an XTM One AI agent end to end with the wizard - persona, guardrails, model, read-only OpenCTI tools, a knowledge base - then share it with a team and test it in the chat.
doc_type: how-to
products: [xtm-one, opencti]
personas: [ai-builder]
owner: docs-xtm-one
last_reviewed: 2026-11-16
edition: ee
migrated_from:
  - xtm-one:/usage/agents/
---

An AI agent is an assistant focused on one job: its persona says what it does and how, its model does the reasoning, and the tools, integrations and knowledge bases bound to it decide what it can read and change. This page builds one end to end with the agent wizard, narrows its OpenCTI access to reading, shares it with a team and tests it in the chat. What an agent is made of is in [What agents are made of](/xtm-one/concepts/agents/).

::diagram{id=xtm-one-build-agents-watcher}

## Before you start

- AI is switched on and the platform runs the Enterprise Edition. Otherwise **Create New Agent** shows why it is unavailable.
- Any user can create an agent: it is private to its creator until shared. Only an administrator can make an agent **Company-managed agent** (open to every user, editable by administrators only).
- The integrations the agent acts through are connected. An OpenCTI registered with XTM One is already an integration, named after the platform, for example **OpenCTI - Cyber Threat Intelligence Platform**: see [Connect agents to your tools with integrations](/xtm-one/build/integrations/).
- The knowledge bases it searches exist and hold their documents: see [Give agents your documents with knowledge bases](/xtm-one/build/knowledge-bases/).

## Choose how to build it

Open :menu[Orchestration > Agents] and click **New Agent**, or click **Create an agent** on the :menu[Orchestration] or :menu[Agent Studio] page. **Create New Agent** offers three ways:

| Card | Use it when |
| --- | --- |
| **Wizard** | You build an agent step by step, with an explanation at each step and a canvas that draws the agent as you go. This page uses it. |
| **Advanced** | You know what you want: pick a template or **From Scratch**, then fill the persona and the required integrations in one form. |
| **Import** | You have an agent file: an XTM One native bundle, or a Microsoft AgentSchema document, in JSON or YAML. See [Import or copy an agent](#import-or-copy-an-agent). |

## Build it with the wizard

The wizard creates nothing until its last click: each step fills a draft, and the canvas on the right shows the agent taking shape. The **Tutorial** button of each step explains it. If you leave the wizard, for example to connect a missing integration, the draft is kept for your session and restored when you come back (**Your unsaved draft was restored.**).

:::steps
1. Pick a starting point. **From Scratch** gives an empty agent; a template gives a persona, default skills and pre-configured assignments, and may require an integration of a given type, shown **required** until you bind one.
2. **Identity**: fill **Agent name**, pick an icon, and write **Description (optional)**. The name and icon show in the agent list, the chat picker, the traces and the audit log, and the description helps colleagues choose the right agent. Under the description, the step may suggest a unit of work, the catalog's closest match to the name and description: click **Use this unit** to count one unit per finished run in ROI once the agent exists, or leave it and pick one later (see [Count what an agent produces](#count-what-an-agent-produces)). Click **Next**.
3. **Persona**: write **Persona / System Prompt**, the standing instructions every chat and run starts from. Say who the agent is, its scope, what it must produce, and when to use each tool and knowledge base. **Generate with AI** drafts a first version from the name and your notes, and reads **Improve with AI** once the field holds text, to rework it; **Prompt library** copies a saved prompt's text into the persona. Click **Next**.

   ::screenshot{id=xtm-one-build-agents-wizard-persona alt="Step 2 of 6 of the agent wizard, Persona: the system prompt of the Velvet Harbor watcher in Persona / System Prompt, Improve with AI above it now that the field holds text, and the Prompt library below."}

4. **Tools**: in **Tools the agent can call**, pick the built-in and custom tools the job needs. Pick nothing to start with the platform's default tool set. Click **Next**.
5. **Integrations**: click each connected integration the agent acts through. Its chip turns to the accent color when bound. An integration gives the agent all its tools until you narrow them on the agent's page. Click **Next**.
6. **Knowledge bases**: in **Knowledge bases to search**, pick the collections the agent should search when it answers. Click **Next**.
7. **Review & create** sums up the agent. Under **Advanced options**, open the ones you need; each returns to the review with **Done**:
   - **Model**: keep **Platform default**, the chat model your administrator chose, which the agent follows when it changes, or search for a model to pin. Pin one only when the agent needs it: a stronger reasoner, or a cheaper model for high-volume work.
   - **Guardrails**: hard rules added after the persona with high priority, such as what the agent must never write. Keep them short and absolute.
   - **Memory**: the four tiers (**Short-term Memory**, **Medium-term Memory**, **Persistent Memory**, **User Memory**) are on by default. Turn them off for an agent that must keep nothing between runs.
   - **Skills**, **Handovers** and **Budget**: procedures the agent loads on demand ([skills](/xtm-one/build/skills/)), the agents it may pass a conversation to, and caps on what one run spends ([run budgets](/xtm-one/use/run-budgets/)). Empty budget fields inherit the platform limits.
8. Click **Create the agent**. The button stays disabled, with the steps still missing, until the agent has a name and a persona and the template's required integrations and knowledge bases are bound. The agent's page opens on **Overview**.
:::

## Give it only the OpenCTI tools it needs

An integration bound to an agent brings all its tools: for OpenCTI, the tools that search and read knowledge, but also those that create, change and delete it. An agent that only reads should get only the reading tools.

The registered OpenCTI integration is managed by the platform: every call carries a token of the person the agent works for. In a chat, that is the person chatting; in a scheduled run, the person the run acts as, the agent's owner by default. The agent therefore never reads more in OpenCTI than that person may. Narrowing its tools limits what it may try, whoever uses it.

:::steps
1. On the agent's page, open **Configuration**, then **Tooling**.
2. In the **Integrations** card, click **Edit**. **OpenCTI - Cyber Threat Intelligence Platform** is marked as bound.
3. Click **All tools** under it. The **Tool Scope — OpenCTI - Cyber Threat Intelligence Platform** dialog lists every tool of the integration.
4. Click **None**, then type `search_opencti` in **Search tools…** and click **Select matching**. Do the same with `get_opencti` and `list_opencti`, so only the tools that search, get and list stay selected.
5. Click **Apply**, then **Save** in the card. The button under the integration now reads the number of tools selected.
:::

::screenshot{id=xtm-one-build-agents-tool-scope alt="The Tool Scope dialog of OpenCTI - Cyber Threat Intelligence Platform for the Velvet Harbor watcher, opened on All tools: 63 tools (all), each with its checkbox and description, the search, the All tools and None buttons, and Apply."}

To make the integration read-only for this agent instead, ask the General Assistant in the chat, for example "Make the OpenCTI - Cyber Threat Intelligence Platform integration read-only on the Velvet Harbor watcher agent". A read-only integration keeps its tools that list, get, search and download, and loses every other one, including tools added to it later. XTM One enforces it when the agent runs: a write tool is not offered to the agent, and a call that names one is refused, whatever the prompt says. The same integration can stay writable on another agent. Removing the integration from the agent removes the mark.

By default, an agent that may write to OpenCTI does not change the knowledge directly: its changes wait in an OpenCTI [draft](/opencti/use/ingest/drafts/) that a person reviews and approves. The integration decides it with its own Live writes setting, off by default (see [Connect agents to your tools with integrations](/xtm-one/build/integrations/)), and each agent can override it. In the **Integrations** card, click **Edit**, then choose **Writes** under each bound OpenCTI integration:

- **Inherit**: the agent follows the integration, as the line under the field says, or each person's own integration when the agent uses everyone's.
- **Live**: the agent's changes go straight to the knowledge, without review, and the agent also gets `apply_opencti_draft` and `create_opencti_label`. Only an administrator or the person who manages the integration can choose it; for an agent that uses each person's own integration, only an administrator.
- **Draft only**: the agent's changes always wait in a draft for review.

Out of edit mode, the card reads **Writes: Live** or **Writes: Draft only** under the integration when the agent does not inherit. A read-only integration never writes, whatever **Writes** says.

Knowledge bases follow the same rule: under **Configuration** > **Knowledge**, each bound knowledge base reads **Read-only (search only)** until you turn on writing for it.

## Choose what it knows about your organization

An agent can start each chat and run with a short summary of your organization profile: its name, sector, regions, size and risk appetite, then one line per category of the profile, with the `get_organization_context` tool to read the rest. Your administrator sets the default for every agent, on unless changed, in **Organization context in agents**, under :menu[Administration > Settings > Organization] (see [Choose what every agent receives](/xtm-one/administer/organization-profile/#choose-what-every-agent-receives)). Each agent can override it, and each assignment can override its agent.

:::steps
1. On the agent's page, open **Configuration** > **General**, and click **Edit**.
2. In **Organization context**, choose **Inherit** to follow the platform, **Always** to give the agent the summary whatever the platform says, or **Off** to remove the summary and the `get_organization_context` tool. The line under the field says what is in effect and where it comes from, such as "On, from the platform".
3. Unless you chose **Off**, choose in **Preloaded organization categories** the categories loaded in full into the prompt after the summary: **Inherit** follows the platform's list, and **Choose** sets the agent's own among **Crown jewels**, **Tech stack**, **AI systems inventory**, **Priority intelligence requirements (PIRs)** and **Tracked organizations**. An empty list preloads nothing: every category stays one `get_organization_context` call away.
4. Click **Save**.
:::

Give the summary to an agent that ranks, scopes or estimates for your company, and turn it off for one whose job does not depend on it, to keep its prompt short. The **Advanced** form of **Create New Agent** asks it as one switch, **Organization context**, on by default: turned off, the agent is created with **Off**. On an assignment's page, the **Parameters** card of **Overview** has the same two fields; set to **Inherit**, the assignment follows the agent, then the platform.

## Share it with a team

A new agent is private. To let a group use it:

:::steps
1. Open **Configuration** > **General**, and find the sharing card under **Guardrails**.
2. Turn on **Share with people and groups**, then type in **Search people or groups…** and pick the group.
3. Choose its level: **Read only** (view and use, without editing the configuration), **Can configure** (also edit it), or **Full access (incl. chats)** (also read the other users' chats with the agent and its traces).
4. If XTM One shows **Before you save**, read who newly gets what, and confirm.
:::

The members find the agent in their **Agents** list and can chat with it at once. Sharing lends what the agent carries: they run it with its integrations, MCP servers and custom tools. With a platform-managed integration such as the registered OpenCTI, each member still reads only what their own OpenCTI rights allow. Personal integrations on a shared agent, levels, company-managed agents and the confirmation are explained in [Share agents and resources](/xtm-one/build/sharing/).

On a company-managed agent, an administrator can also turn on **Hide activity from non-admins** in **Configuration** > **General**: the agent's runs, traces, work items and shared state are then visible only to administrators and to its owner. The other users still use the agent and see the runs made as them.

When the agent's owner loses access to a bound integration, for example because its share was revoked, the sharing card warns that the owner no longer has access to it. The agent's runs then refuse that integration's tools until the owner gets access back. Whoever can edit the agent clicks **Remove binding** to unbind it, or opens the integration from its name.

## Test it in the chat

:::steps
1. On the agent's page, click **Chat with Agent**. The chat opens with the agent selected.
2. Ask what the agent is for, the way its users will ask it. While it works, the chat shows **Using tools…**.
3. Hover the answer and click the icon next to its time. **Reasoning details** counts the iterations and the tool calls, and lists each tool called: open one to see what it was given and what it returned. Check that the agent searched the knowledge base when the question needed it, and called only the tools you expected. An administrator also gets **View trace**, the whole trace of the run, when traces are recorded.
4. Rate the answer with **Helpful response** or **Not helpful**. The ratings of everyone who uses the agent gather in its **Feedback** tab.
5. Change the persona from **Configuration** > **General** (**Edit**, then **Save**) and ask again until the answers hold.
:::

::screenshot{id=xtm-one-build-agents-chat-test alt="Camille Roux tests the Velvet Harbor watcher: asked what changed in Velvet Harbor's infrastructure this winter, it names the two reports of Example CERT, the look-alike booking domains of 16 December 2025 and the new hosting of 23 December, and suggests blocking the domains at the mail gateway."}

Every saved change of the persona is a version in **Versioning**, to compare and roll back. The general settings also hold the **Platform routing model** choice: a lightweight model your administrator sets, suited to an agent that takes a narrow, repeated decision such as classifying or routing.

## Give it recurring work with an assignment

An assignment is a task the agent runs on its own: a prompt, and a trigger that says when. An agent can carry several, each with its own history of runs.

:::steps
1. On the agent's **Overview**, click **Add Assignment**. The **New Assignment** panel opens.
2. Fill **Assignment name**.
3. Choose the trigger:
   - **schedule**: on a cron expression. Pick a preset, such as **Monday at 9 AM**, or type it in **Custom cron**. The expression is read in Coordinated Universal Time (UTC), not in your time zone: Monday 08:00 in Paris is `0 6 * * 1` in summer and `0 7 * * 1` in winter.
   - **webhook**: when another tool calls the assignment's own URL, generated after creation. See [Start an agent from another tool with a webhook](/xtm-one/use/webhook-triggers/).
   - **Event**: when a bound integration reports an event, such as a new email, a Slack message, or a Jira issue created, assigned or moved to another status. Offered only when one of the agent's integrations sends events: pick it in **Integration**, then the event in **Event Type**. XTM One reads Jira every minute, from the moment the assignment exists, and runs once per change. Jira can also call XTM One as soon as an issue changes: see [Advanced tuning](/reference/xtm-one/configuration/#advanced-tuning) in the configuration reference.
   - **manual**: only when someone clicks **Run Now**. Manual input fields, shown in a form at each run, are added later under **Input Parameters**. A **Select** field lists its choices in **Options**: press Enter after each one. Its **Default value** is one of them, or **No default**.
   - **Work item**: the agent's inbox, which picks up the work items due for this agent. See [Give agents recurring work](/xtm-one/use/work-items/).
4. In **Action prompt**, write what the agent must do at each run. It comes on top of the persona: say the period, the output and who it is for.
5. On a company-managed agent, an administrator chooses in **Run as** whose identity the automatic runs use: their integrations, model and memory. A private agent's automatic runs act as its owner. A manual run always acts as the person who clicks **Run Now**.
6. Click **Create Assignment**. Its page opens from the **Assignments** list of the agent's **Overview**.
:::

On the assignment's page, **Overview** holds the trigger, the run-as identity and the parameters. Under **Output visibility**, **Show as notification** (off by default) sends a bell notification when a run completes, and **Show in latest results** lists the result in the dashboard digest. Under **Safety & reliability**, **Simulate without executing actions** stubs every tool call, so the runs only describe their plan: use it to try an assignment that writes, and turn it off once the plan is right. **Prompt & Bindings** narrows the integrations and knowledge bases the assignment uses, **Channels & Bots** posts its result to a messaging channel, and **Activity** lists its runs. **Run Now** starts a run at once; **Pause** and **Resume** stop and restart its trigger.

::screenshot{id=xtm-one-build-agents-assignment alt="The Overview of the assignment Weekly Velvet Harbor brief, shared by Sophie Laurent: Active, Scheduled on 0 6 * * 1 with 2 runs, an AI-generated label turned on, and every knowledge base of the agent read through RAG."}

## Count what an agent produces

To count an agent's work in ROI, declare the unit of work its runs deliver, such as one threat report triaged. Its finished runs then count in the **ROI** section of the agent's **Value** tab, on the [Value page](/xtm-one/use/value/) and in the **Outcomes** tab of [Observability](/xtm-one/administer/observability/). The catalog gives each unit the minutes it takes a person by hand and the hourly rate of the role: [How XTM One computes the value of agents' work](/xtm-one/concepts/value-methodology/) explains the figures. Declaring a unit and reading the figures start no run and call no model.

:::steps
1. On the agent's page, open **Configuration**, then **Outcomes**. Until a unit is declared, the card says that the agent is not counted in ROI.
2. Click **Pick a unit**, then search **Unit of work**: the catalog's **Security units** come first, then the **Custom** units of your organization. An administrator also gets **Manage units**, to change the catalog in [Settings](/xtm-one/administer/settings/#organization).
3. Choose the **Counting mode**:
   - **Per run**: each finished run counts **Units per run**, a whole number from 1 to 10,000.
   - **Per output item**: each item of the list at **Output path** in the run's JSON output counts one unit, for example `result.contracts`. Offered only to an agent that answers with a JSON output schema; otherwise it reads **Needs a JSON output schema.**
   - **Per work item**: each work item the run completes counts one unit.
4. Click **Save outcomes**.
:::

The card then shows the unit and the counting mode, with **Edit** and **Stop counting**; only the people who can edit the agent see them. An agent in a fleet counts the fleet's unit, marked as inherited from the fleet, until you click **Customize**; **Reset to inherited** returns to the fleet's. A built-in agent ships with its own unit, of **Origin** **Default**: an administrator can **Customize** it, and **Reset to default** brings it back. An assignment counts the agent's unit unless it declares its own, in the same card under **Prompt & Bindings** on the assignment's page.

On the agent's **Value** tab, the **ROI** section comes first, above the run budget. It reads its figures over **Last 7 days**, **Last 30 days** (the default) or **Last 90 days**:

| Card | What it says |
| --- | --- |
| **Hours saved** | The time the counted runs and chat deliverables spared people, with how many of each. |
| **Net value** | What that time is worth minus what the agent's model calls cost over the window, chat included, with the cost under it. |
| **Review rate** | The share of the counted runs a person reviewed. Click it to list the ones nobody reviewed yet under the agent's **Activity**, and review them as in [Review what agents did](/xtm-one/use/work-history/). |

Under the cards, **How this is computed** gives the formula with the agent's own numbers, and **Assumptions** the minutes and rates behind them. While some of them are still the catalog's defaults, a note says the figures are estimated, with a link to **Units and rates**. When the window counted nothing, **Widen the window** moves to a longer one. An agent without a unit shows **Pick a unit**, which opens **Outcomes**, to the people who can manage it; the others are told to ask its owner. What the agent finished in a chat, such as a deck, counts in the same figures: **How this is computed** shows it as the **Chat (estimated)** line, and the **From chat, estimated** card under the section details it. See [Count what agents finished in a chat](/xtm-one/use/value/#count-what-agents-finished-in-a-chat).

An administrator always reads the agent's totals. Anyone else reads them once enough people used the agent over the window: as many as **Smallest group shown apart**, five by default, which an administrator sets in [Settings](/xtm-one/administer/settings/#organization) under **Rates by role**. Below that, the section shows only your own use, under **Your use of this agent**, without the review link and the assumptions; if you did not use the agent, it says that the totals are hidden.

::screenshot{id=xtm-one-build-agents-roi alt="The ROI section of the Velvet Harbor watcher's Value tab over the last 30 days: the hours its weekly brief saved, its net value after the model cost, its review rate, the note that some minutes and rates are catalog defaults, and how the figures are computed."}

In the list view of :menu[Orchestration > Agents], the **Net value (30d)** column gives each agent's figure over the last 30 days, or **Value (30d)** when the deployment hides costs. An agent without a unit reads "-", with **Declare a unit** for the people who can edit it. An agent whose totals are hidden from you also reads "-": its tip gives your own use, and the row sorts after the figures.

:::example[Sophie Laurent builds the Velvet Harbor watcher]
At Grand Meridian Hotels, Camille Roux and the CTI team want a summary of every new OpenCTI report on Velvet Harbor, the intrusion set behind the fake summit booking of 9 January 2026. Sophie Laurent, the AI engineer, builds the agent on 12 April 2026. In the wizard she picks **From Scratch**, then:

| Step | Field | Value |
| --- | --- | --- |
| **Identity** | **Agent name** | `Velvet Harbor watcher` |
| | **Description (optional)** | `Summarizes new OpenCTI reports on Velvet Harbor for the CTI team.` |
| **Persona** | **Persona / System Prompt** | `You are the Velvet Harbor watcher of Grand Meridian Hotels' CTI team. When asked about Velvet Harbor, search OpenCTI for the reports on the intrusion set published since the date given, read each one, and write a summary: what is new, the techniques and indicators named, and whether our hotels in France and Spain are named or exposed. Check the incident response policy in the knowledge base to say whether the finding must be escalated, and quote the policy section. Link each report in OpenCTI. Say clearly when you found nothing new.` |
| **Tools** | **Tools the agent can call** | Nothing picked: the default set |
| **Integrations** | | **OpenCTI - Cyber Threat Intelligence Platform** |
| **Knowledge bases** | **Knowledge bases to search** | **Grand Meridian security policies** |
| **Model** | | **Platform default** |
| **Guardrails** | | `Never create, change or delete anything in OpenCTI. Never copy guest personal data into a summary.` |

After **Create the agent**, she narrows **OpenCTI - Cyber Threat Intelligence Platform** to the tools matching `search_opencti`, `get_opencti` and `list_opencti`, and shares the agent with the **CTI team** group in **Read only**. In **Chat with Agent** she asks "What is new on Velvet Harbor since 1 April 2026?". The answer lists the new reports with their OpenCTI links, quotes the escalation section of the incident response policy, and **View trace** shows calls to `search_opencti` and the knowledge base search only. Camille Roux now finds **Velvet Harbor watcher** in her **Agents** list.

Last, Sophie clicks **Add Assignment** and creates **Weekly Velvet Harbor brief**: trigger **schedule**, **Custom cron** `0 6 * * 1` (Monday 08:00 in Paris in summer time), **Action prompt** `Summarize what is new on Velvet Harbor in OpenCTI over the last seven days for the CTI team, with the escalation the incident response policy requires.` She turns on **Show as notification** and posts the result to the CTI team's channel from **Channels & Bots**. Every Monday morning, the brief runs as Sophie and the CTI team reads it before the weekly meeting.

To show what the brief is worth, Sophie opens **Configuration** > **Outcomes**, clicks **Pick a unit** and picks **Threat report triaged for relevance** in **Unit of work**, with **Per run** and **Units per run** `1`, then **Save outcomes**. From the next brief on, the **ROI** section of the agent's **Value** tab counts each run: 20 minutes of an analyst's work by hand, minus 3 of review, at the catalog's rate of $88.73 an hour, about $25 a week before the model cost.
:::

:::xtm-one[Build or change an agent from the chat]
The General Assistant builds agents too. Describe the agent in the chat: it creates it with its model, tools, read-only integrations and knowledge bases in one step, lists the integrations it bound, and runs it once on a prompt so you see an answer before anything is scheduled. It can also share an agent with a group or with named colleagues who have an account. It acts as you, with the same checks as the screens.
:::

## The agent's page

:menu[Orchestration > Agents] opens on **Mine** when you have agents of your own, otherwise on the first of **Company**, **Shared** and **All** that lists agents. **Company** holds the company-managed agents and the built-in agents of the platform. Once your organization has company-managed agents of its own, **Company** hides the built-in ones: its count leaves them out, and its tooltip says how many are hidden. To list them, check **Built-in** in the **Properties** filter; XTM One keeps your choice for your next visits. A search that matches only built-in agents lists them anyway. Click an agent to open its page:

| Tab | What it is for |
| --- | --- |
| **Overview** | What the agent does and what is bound to it, at a glance, and its assignments. Its **Fleet** card names the [fleet](/xtm-one/build/fleets/) the agent belongs to, the agent it reports to and the fleets that connect it, with **Open the fleet** and how many of its automations are on; an agent in no fleet says **In no fleet: this agent is in the playground.** |
| **Configuration** | **General** (persona, guardrails, model, organization context, sharing), **Tooling** (built-in tools, integrations, MCP servers, custom tools), **Knowledge** (knowledge bases, memory, prompts, skills), **Channels & Bots**, which lets the agent answer in Slack, Microsoft Teams and other messaging apps ([Talk to agents from your messaging apps](/xtm-one/administer/channels/)), and **Outcomes**, the unit of work its runs count in ROI. A built-in tool this deployment switched off shows **Unavailable**. |
| **Value** | What the agent returns, then what one run may spend. Its **ROI** comes first ([Count what an agent produces](#count-what-an-agent-produces)), then the caps on a run's spend. Lowering a limit is open to whoever edits the agent; raising one above the platform's own is an administrator's. See [Limit what an agent run may spend](/xtm-one/use/run-budgets/). A link to the former **Budget** tab, or to the ROI of the **Overview**, opens **Value**. |
| **Versioning** | Every version of the persona and the tracked settings, to compare and roll back. |
| **Memory** | What the agent learned, for the people who can manage it. |
| **Chats**, **Delegations** | Its conversations, and the work it sent to or received from other agents. A conversation opens read-only, drawn as in the chat, with **Back to Chats** above it. |
| **Feedback** | Every rating and comment given on its answers and runs, for the people who can manage it. |
| **Activity** | Its run history, to spot failures. |

Another agent can call this one, to consult it or hand it a background task, and ask for the answer in a format of its own. By default such a call is refused before the agent runs, so the agent always answers in its configured format. To accept it, turn on **Accept a format requested by the caller** in the **Pipeline** card of **Configuration** > **General**: the caller's format then replaces the agent's for that call. Only the people who can manage the agent change it.

A disabled agent stays listed, dimmed: its chat is limited and its assignments do not run. **Disable agent** and **Enable agent** are in the page header.

Deleting an agent, from the bin icon of the page header, removes it and all its assignments for good; its conversations are kept with their whole history. On a narrow screen, **Duplicate**, **Export** and **Delete agent** are under **More actions**. In **Continue its conversations with**, pick an enabled agent to take them over: the conversations of the people who can use it continue with that agent, which reads the earlier answers. With **No agent**, each owner can continue a conversation with an agent of theirs from its menu. Click **Delete**; for a shared agent, click **Continue**, then type its name in **Delete Shared Agent** to confirm.

## Import or copy an agent

- **Duplicate** makes a private copy you can change without touching the original. When you copy someone else's agent, what you could not bind yourself (their integrations, MCP servers, knowledge bases, tools, skills, prompts) is left out, and the confirmation says how many of each. Copied assignments start disabled.
- **Export** downloads the agent as an **XTM One native bundle** (everything, for backups and moving between XTM One platforms) or a **Microsoft AgentSchema** document (an open format other platforms read; what it cannot hold is kept in an XTM One section of its metadata and listed before the download). A credential in an MCP server's URL or arguments is masked, and variables come without their values.
- **Import** recognizes the format itself. Integration bindings, knowledge bases and links to other agents are not imported, a custom tool from an AgentSchema document arrives without its code, an MCP server as a placeholder, and assignments start disabled. The report says what to fill in before enabling the agent.

## What's next

- [Start an agent from another tool with a webhook](/xtm-one/use/webhook-triggers/).
- [Share agents and resources](/xtm-one/build/sharing/).
- [Teach agents a procedure with skills](/xtm-one/build/skills/).
- [Let an OpenCTI playbook call the agent](/opencti/use/automate/playbooks/create/), or make it answer inside OpenCTI: [Choose which agent answers each AI feature](/xtm-one/administer/intents/).
