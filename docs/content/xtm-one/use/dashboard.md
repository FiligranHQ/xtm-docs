---
title: Read the home dashboard
description: Read the exposure posture of the XTM suite on the XTM One home dashboard, your briefing, what to act on first, the stages and what agents did, switch vertical, and arrange the widgets your way.
doc_type: how-to
products: [xtm-one, opencti, openaev]
personas: [executive, soc-manager, cti-lead]
owner: docs-xtm-one
last_reviewed: 2026-11-12
migrated_from:
  - xtm-one:/usage/home-dashboard/
---

The home dashboard of XTM One, the **CTEM Command Center**, opens from :menu[Home] at the top of the left navigation. It says at a glance how exposed the organization is across the XTM suite. It also says what to act on first, and what the AI agents already did about it. It scores each stage of continuous threat exposure management (CTEM), from the attack surface in scope to the fixes, from what OpenCTI and OpenAEV hold. It averages them into one exposure posture out of 100. This page reads each part, top to bottom, and arranges the dashboard your way. How OpenCRQ opens the same command center is in [XTM One in OpenCRQ](/xtm-one/use/in-opencrq/).

::diagram{id=xtm-one-dashboard-posture}

## What you see depends on the platform

| Platform state | The dashboard shows |
| --- | --- |
| No OpenCTI or OpenAEV platform connected yet | A welcome page with **Connect OpenCTI** and **Connect OpenAEV**. An administrator connects them once, with a registration token: see [Set up the platform in Settings](/xtm-one/administer/settings/) |
| Community Edition, a platform connected | A one-time presentation of what the Enterprise Edition adds, then the command center with its figures, without AI: see [When AI cannot run](#when-ai-cannot-run) |
| Enterprise Edition, a platform connected | The full command center |
| AI switched off by an administrator | A page that says so: see [Run XTM One with AI switched off](/xtm-one/administer/running-without-ai/) |

The top of the command center says which edition the platform runs, and each part says which connected product its figures come from. A part whose product is not connected shows no figure, and a button to connect it, such as **Connect OpenAEV**.

Above it, notices say what waits for a person, and go away once it is done:

- the work items waiting for you, naming the one waiting longest: **Review** opens it, or the **Waiting for me** view of :menu[Work > Operations] when several wait;
- a missing default model or AI provider, with **Configure** or **Configure AI Models** for an administrator;
- knowledge base documents that failed to index, or that are being indexed again, with **View Docs**;
- an integration of yours that shares your personal OAuth credentials: **Stop sharing** opens it, and its cross, **Keep sharing**, closes the notice on every device;
- in the Community Edition, **AI is not activated - this deployment runs Community Edition**, with **Start free trial**, **Contact sales** and, for an administrator, **Activate**.

The first time an administrator signs in, while the organization profile was never saved, **Set up your organization profile** proposes to fill in the profile that every agent and fleet reads before it acts. **Set up now** opens :menu[Administration > Settings > Organization] in edit mode, with the **Draft** menu to start **From the platforms** or **With AI**; nothing is saved before **Save**. **Later** and **Dismiss** close it. The first answer is kept for the whole platform: the proposal never comes back, for anyone. See [Set up the platform in Settings](/xtm-one/administer/settings/#organization).

### When AI cannot run

In the Community Edition, or while no default model is configured, the command center keeps its exposure figures and leaves out what needs AI:

- Where **Start chatting** would be, a sentence says why. An administrator also gets the way forward: **Start free trial** and **Activate License** in the Community Edition, **Configure a model** while no default model is set. Anyone else reads whom to ask, such as "Ask your administrator to activate a license."
- The posture line leaves out the agents watching. **Exposure briefing** keeps its greyed sample, and the **AI assessment** of each stage page shows the same sentence.
- **Act now** leaves out the rows that lead to the chat. The row of a gap in detection coverage opens the **Discovery** stage.
- **Add a widget** offers two exposure figures of the vertical in place of **Agents** and **Agentic executions**: **Critical CVEs** and **ATT&CK exercised** for **Cyber**, **Channels monitored** and **Open incidents** for **FIMI**, **Leaked credentials** and **Containment** for **Fraud**.

## Read the exposure posture

The **Posture overview** at the top gives the exposure posture out of 100 and how many CTEM stages are covered, the number of agents watching and running, and a ribbon of the stages. Click the posture to read **How the Exposure Posture is computed**: a weighted average of the five stage scores, renormalized over the stages that have data, so a stage with no connected product never lowers the score. The weights depend on the vertical.

The exposure data is collected once a day. **Last synced** says when; **Force sync** fetches it again now, for you. **Start chatting** opens the chat. Under it, the pills of OpenCTI, OpenAEV, OpenCRQ and the XTM Hub give each product's edition, health and version; click one to open the product. OpenCRQ shows **Soon** until an OpenCRQ platform registers.

## Read your briefing

**Exposure briefing**, under the overview, is written for you by the **Exposure Briefing Analyst** agent, scoped to what you can see in OpenCTI and OpenAEV. It is generated every month for each person. It gives:

- a headline, a summary and a **Cross-product insight**;
- the top three **Key findings**, each with its action;
- the top three **Next steps**, by priority: **Follow up in chat** hands a step to the analyst agent, in a conversation;
- the **Recommended focus**.

**Show more** draws every finding and step. The refresh button, **Generate my briefing**, writes yours now; **View full briefing** opens it on its own page. Until your first briefing is written, the widget shows a greyed sample marked **Sample**: generate yours, or wait for the monthly run.

While the organization profile was never saved, your briefing says **This briefing is generic**: the analyst cannot rank findings by your sector, regions, crown jewels or tech stack. An administrator gets **Set up organization context**, then generates the briefing again; anyone else reads that an administrator can set it up. See [Set up the platform in Settings](/xtm-one/administer/settings/#organization).

## Act on what comes first

**Act now** ranks what to do, the most urgent first, and marks the first row **Do first**. Under its title, each row says why it is there, and gives its priority. The queue holds:

- the work items waiting for you, and the runs waiting for your approval;
- what the connected products found, one row per stage: known-exploited exposures to triage, failed OpenAEV expectations, a stage with a connected product but no data, a gap in detection coverage;
- the products to connect and, for administrators, **Wire in your security stack**.

When failed expectations of several types share the same count, their row names the types in one phrase.

The button of a row opens what to act on: the waiting work items, the executions, the page of its stage, or the connection steps. **Triage**, on the known-exploited row, opens the **Prioritization** stage, beside the top exposures. **Review**, on a failed expectation, opens the **Validation** stage, with the failed expectations per type and their simulations. Five rows show; **Show all** opens the others. The queue leaves the briefing's next steps to the briefing. With nothing to act on, it says "No recommended action yet".

When the organization profile names a **Tech stack** that OpenCTI knows as software, the known-exploited row counts only the known-exploited vulnerabilities of that software. Its reason then names the products of your tech stack it affects, up to three, in the order of the profile. Otherwise, it counts every known-exploited vulnerability in OpenCTI.

**Ask AI**, on the rows of what the connected products found and on the setup rows, opens the chat with the row in the composer. The message gives the row's title, why it is there, its stage, and asks the agent for step-by-step guidance. Read it, then send it yourself. **Ask AI** is not offered when AI cannot run.

While none of these rows is tailored to the organization profile, **These actions are generic** says they come from module-wide figures. An administrator gets **Set up organization context**, or **Review organization context** once a profile exists. It opens :menu[Administration > Settings > Organization] in edit mode, as **Set up now** does. Anyone else reads that an administrator can set it up.

## Read the stages

**Exposure lifecycle** gives one chip per stage, **Scoping**, **Discovery**, **Prioritization**, **Validation** and **Remediation**, and says how many stages are covered. Each chip shows:

- the stage's score, in the color of its band, and its trend over 30 days, or **No trend yet**;
- the stage's key figure;
- a dot when a recommended action for the stage waits.

A stage with no connected product says **No data**, with a button such as **Connect OpenCTI**. A stage with too little data says **Not enough data to score**. **Export CSV** downloads the posture, the stages and their figures as a CSV file, once the posture is computed from a connected platform. When the known-exploited count of your tech stack is computed, the file also gives **Known-exploited in your tech stack** and **Tech stack products affected**.

Click a chip to open the stage's page: its score and change, its figures, and its **History** over **30 days** or **90 days**. The **AI assessment** there is the stage analyst agent's reading of the stage; regenerate it for you from its button. **Back to Command Center** brings you back.

The last chip, **Governance & Risk**, is the risk side of exposure, quantified by OpenCRQ. Once an OpenCRQ platform is registered, **Open in OpenCRQ** opens it; until then, the chip says **OpenCRQ not connected**.

## Read the coverage and the stats

**Stats and coverage** groups four tiles:

| Tile | What it gives |
| --- | --- |
| **Coverage** | A heat map of the stages against OpenCTI, OpenAEV and OpenCRQ: which product feeds which stage, in the color of its band. Click a stage to open it |
| **Exposure stats** | **Stages improved** and **Stages declined** over the period, and the change of the posture and of each stage |
| **Findings** | The three headline figures of the vertical. For **Cyber**: **Known-exploited** (actively exploited vulnerabilities, KEV), **Detection coverage** (the share of ATT&CK techniques with a detection) and **Validation success** (prevention and detection in OpenAEV). **Known-exploited** counts those of your tech stack when the organization profile names one. Click a figure to open the page of its stage |
| **ATT&CK exercised** | The share of ATT&CK techniques your OpenAEV simulations **Tested**, and the share **Untested**, with the **Validation success** |

In **Cyber**, while **Known-exploited** is not tailored to your tech stack, **These figures are module-wide** says it counts every such vulnerability in OpenCTI. An administrator gets the same button as in **Act now**; anyone else reads that an administrator can set it up.

## See what was done for you

**Done for you** places what ran over the last **7 days** or **30 days** on a timeline, one lane per source:

| Lane | What it holds |
| --- | --- |
| **Modules** | The events of the connected products, such as a sync or a registration |
| **Automations** | The runs of the agents' assignments, and the digests |
| **Fleets** | The runs of the fleets, only when a fleet ran in the period |
| **Outcomes** | What a run produced: the actions it took, a digest published |

Click a point to open its run. When a lane holds more than three points on one day, they become a count: click it to list them, then **Open run** on one. **All executions** opens every run: see [Review what agents did](/xtm-one/use/work-history/).

**Fleets**, at the bottom, gives one line per CTEM fleet, a team of AI agents: whether it is on, and when it last ran. Click a line to open the fleet; **View all fleets** opens them all. While every CTEM fleet is off, **Add agentic work to this page** says what each fleet would add, and about how many runs a month it would take. An administrator clicks **Set up** to open the fleet: nothing runs until it is turned on there (see [Run agents as a team with fleets](/xtm-one/build/fleets/#switch-its-automations-on)). Close the invitation to keep a single sentence instead. In Community Edition, the row says "Fleets run with Enterprise Edition".

## Switch the vertical

The switcher at the top right moves the whole dashboard between three verticals: **Cyber** (the default), **FIMI** (foreign information manipulation and interference, disinformation) and **Fraud**. Each vertical has its own headline figures, weights and assessments: a **Fraud** view counts phishing assets, not known-exploited vulnerabilities. Your choice is kept for your next visit.

## Arrange the dashboard

:::steps
1. Click **Customize**, next to the vertical switcher. The widgets stay on screen, framed, and a bar lists what you can do.
2. Drag a widget to move it; drag its bottom right corner to resize it. A smaller widget shows fewer details, the most important ones staying.
3. Click the cross of a widget to remove it. **Add a widget** lists the widgets not shown: the ones you removed, and the ones the default layout leaves out.
4. Click **Done**. While you customize, the widgets open nothing.

::screenshot{id=xtm-one-dashboard-command-center alt="The CTEM Command Center of the Cyber vertical greeting Hélène: an exposure posture of 0 out of 100 across 5 CTEM stages, 69 agents watching, Start chatting, and OpenCTI, OpenAEV and OpenCRQ connected in Enterprise Edition with their versions. Customize sits next to the Cyber and FIMI switcher, and the Exposure briefing starts below."}
:::

Each change is saved to your profile at once, so the dashboard looks the same on every device. Each vertical keeps its own layout, and your layout changes nobody else's. **Reset to default** puts the default layout back. In a narrow window, the widgets stack in the order of your layout. When the default layout gains a widget, a layout you saved keeps its widgets and gets the new one below them.

While you customize, each widget shows its name. Two differ from their title: the stage chips are **Lifecycle band**, and the fleets line is **Fleets row**. **Add a widget** also offers the widgets the default layout leaves out:

- the single figures: **Exposure posture**, the three headline figures of the vertical, **Agents**, **Agentic executions** and **Value band**;
- the stage cards: **Exposure lifecycle**, one card per stage and **Governance & Risk**, each stage card with its **AI assessment**;
- the lists: **Recommended actions**, **Live Activity**, **Latest Results & Digests**, **Fleet activity** and **Fleet timeline**;
- the tiles on their own: **Coverage** and **Exposure stats**.

:::example[Hélène's Monday look at Grand Meridian's exposure]
On Monday 13 April 2026, Hélène Marchetti, CISO of Grand Meridian Hotels, opens `https://xtm-one.grand-meridian.example` before the crisis cell's meeting. OpenCTI and OpenAEV are connected, the vertical is **Cyber**. She reads:

| Where | What she checks |
| --- | --- |
| **Posture overview** | The exposure posture, across 5 of 5 CTEM stages |
| **Exposure briefing** | **Recommended focus**: the PowerShell loader still undetected on two front-desk workstations |
| **Act now** | The **Do first** row, which she sends to Julien Bernard's SOC |
| **Exposure lifecycle** | The **Validation** chip: whether the weekly Velvet Harbor replay in OpenAEV scores better than last month |
| **Done for you** | On the **Automations** lane, the **Weekly Velvet Harbor brief** run of 08:00 |

She clicks **Customize**, moves **Act now** above **Exposure briefing** and removes **Fleets row**: her dashboard now opens on what to do, on her laptop and her phone.
:::

## What's next

- [Review what agents did](/xtm-one/use/work-history/), behind **All executions**.
- [Get answers from an agent in the chat](/xtm-one/use/chat/), behind **Start chatting** and **Follow up in chat**.
- [XTM One in OpenCTI](/xtm-one/use/in-opencti/) and [XTM One in OpenAEV](/xtm-one/use/in-openaev/), for the products the posture is computed from.
