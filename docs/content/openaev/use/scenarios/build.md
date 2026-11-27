---
title: Build a scenario that replays an attack
description: Create a scenario, fill it with injects by hand or with the scenario assistant, aim it at your assets and teams, then launch it or schedule it to follow your posture.
doc_type: how-to
products:
  - openaev
personas: [validation-engineer, exercise-planner]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/scenario/
---

A scenario is the attack you want to replay, written once: its injects, the techniques they run, the assets and teams they reach, and when. Each launch copies it into a simulation, so the same scenario run every week shows whether your defenses get better or worse. This page creates a scenario, fills it, and launches or schedules it. How scenarios, simulations and injects relate is in [Scenarios and simulations](/openaev/concepts/scenarios-simulations/).

::diagram{id=openaev-scenarios-build-lifecycle}

## Before you start

- Your role has **Manage assessment** to create and change a scenario, and **Launch assessment** to launch or schedule it. **Delete assessment** deletes it.
- The assets to attack have an OpenAEV agent or another executor, and belong to asset groups: see [Assets](/openaev/use/targets/assets/). For tabletop and crisis exercises, the teams and persons exist: see [People](/openaev/use/targets/people/).

## Create the scenario

:::steps

1. Open :menu[Scenarios] and click **Create**. The **Create a new scenario** panel opens.
2. Choose the type:
   - **Time-based scenario**: each inject runs at a delay after the start. Use it for an attack replay, a phishing exercise or a crisis timeline.
   - **Chained scenario**: each step starts the next according to the result of the previous one, across a scope of assets. It requires the Enterprise Edition. See [Chain attacks across your network](/openaev/use/attack-chaining/).
3. Under **General**, give a **Name**, the only required field. The other fields describe the scenario and filter the list:

   | Field | Default | Purpose |
   | --- | --- | --- |
   | **Category** | **Attack Scenario** | **Global Crisis**, **Attack Scenario**, **Media Pressure**, **Data Exfiltration**, **Capture The Flag**, **Vulnerability Exploitation**, **Lateral Movement**, **URL Filtering** |
   | **Main focus** | **Incident Response** | The defense the scenario tests: **Endpoint Protection**, **Web Filtering**, **Incident Response**, **Standard Operating Procedures**, **Crisis Communication**, **Strategic Reaction** |
   | **Severity** | **High** | **Low**, **Medium**, **High**, **Critical** |
   | **Default kill chain** | **Automatic** | The kill chain the results are shown on |
   | **Description**, **Tags** | Empty | Tags also apply the [default asset rules](/openaev/configure/asset-rules/) of their tag to the injects |

4. Under **Modules**, turn on **Enable lessons learned** to collect the players' feedback after each run. See [Lessons learned](/openaev/use/components/lessons/).
5. For a time-based scenario, check **Emails & SMS**: **Sender email from**, **Reply to**, and the **Messages header** and **Messages footer** that frame every email and SMS, `SIMULATION HEADER` and `SIMULATION FOOTER` by default.
6. To build the injects from techniques right away, select the **Scenario assistant** card at the end of the form.
7. Click **Create**.

:::

The scenario opens, on the scenario assistant when you selected it. A time-based scenario has the tabs **Overview**, **Injects**, **Tests**, **Execution**, **Findings** and **Statistics**, and **Lessons learned** when it is enabled.

::screenshot{id=openaev-scenarios-create alt="The Create a new scenario panel over the scenarios list, where the three Velvet Harbor scenarios show: the choice between a chained scenario and a time-based scenario."}

## Add the injects

Each inject is one action of the attack: a payload run on endpoints, an email, an SMS, a media article, a manual step. Three ways fill a scenario:

- **By hand**: on the **Injects** tab, click **Create** and pick an action from the Threat Arsenal. See [Injects](/openaev/use/injects/).
- **With the scenario assistant**, from techniques: below.
- **From a spreadsheet**: see [Import a timeline of injects from a spreadsheet](/openaev/use/scenarios/import/).

### Generate injects from techniques

The scenario assistant creates one or more injects per MITRE ATT&CK technique you select, with actions that fit the platforms and architectures of your targets. It needs actions in the Threat Arsenal linked to the techniques.

:::steps

1. In the header of the scenario, click **Scenario assistant**.
2. Under **Target**, click **Modify assets** or **Modify asset groups** and choose the endpoints to attack.
3. Under **Configuration**, set the **Number of injects by TTP**, from 1 to 5. With more than one, the assistant picks different actions for the same technique.
4. On the matrix, select the techniques. **With actions** shows only the techniques that have an action; **Search a technique** finds one by ID or name.
5. Click **Create injects**, then **Access these injects**.

:::

For each technique, the assistant first looks for an action that runs on every platform, and gives it all your targets. Otherwise, it groups the targets by platform and architecture, and looks for an action that covers them all, then group by group. A technique also matches the actions of its sub-techniques. A platform and architecture left without an action gets a placeholder, disabled, named `[T1078] Placeholder - Windows x86_64`: its description says which payload is missing. Create the payload in the [Threat Arsenal](/openaev/use/threat-arsenal/), then delete the placeholder and generate again.

:::xtm-one[Find the techniques of a threat report]
With XTM One connected and the Enterprise Edition, **Suggest TTPs with XTM One** in the scenario assistant reads a text you paste, or up to five `.txt` or `.pdf` files, and adds the techniques it finds to your selection. Check them on the matrix before you click **Create injects**. See [XTM One in OpenAEV](/xtm-one/use/in-openaev/).
:::

### Set the teams and variables

For injects that reach people, click **Configuration** in the header. The **Scenario configuration** panel has three tabs: **Teams**, the teams and persons the scenario targets; **Variables**, values that injects reuse; **Media pressure**, the articles injects publish. See [People](/openaev/use/targets/people/) and [Custom variables](/openaev/use/components/variables/).

## Launch or schedule it

- To run it now, click **Launch**, then **Confirm**. OpenAEV creates a simulation from the scenario, starts it and opens it.
- To run it on a schedule, click **Scheduling**. Choose **Once**, **Hourly**, **Daily**, **Weekly** or **Monthly**, the **Time**, the **Day of week** or **Week of month** when they apply, a **Start date** and an **End date (optional)**, then **Save**. The dialog sums up the schedule in a sentence. **Only weekday** skips weekends for hourly and daily runs.

A scheduled scenario reads **Scheduled** in its header and in the list. OpenAEV creates each simulation a minute before its time. **Stop** removes the schedule; **Launch now** runs it once more without changing it.

Each simulation is a copy: changing its injects does not change the scenario, and changing the scenario changes only the simulations to come. Teams and documents are shared, not copied. See [Run a simulation and read its results](/openaev/use/simulations/).

Without the Enterprise Edition, a launch fails when an inject would run through an executor that requires it, such as a third-party EDR.

## Read the overview

The **Overview** shows the results of the latest simulation. Before the first run, it shows sample results, greyed, with **Launch simulation now**.

- **Latest run posture**: a gauge per expectation type, prevention, detection, vulnerability and human response. Its ring splits the expectations into **Success**, **Partial**, **Failed** and **Pending**, and its **Resilience** is the share of successes among all of them, pending ones included, so it matches the green part of the ring. Its number turns green from 75%, amber from 50%, orange from 25%, and red below. While every expectation of a type is still pending, its gauge stays grey: no result yet, not a failure.
- **Posture trend**: the scores of the successive simulations.
- **Kill chain results**: the techniques of the scenario on the matrix, colored by their results.
- **Simulations**: every simulation of the scenario, with its status and scores.
- **Information**: the scenario's fields, and **Open in OpenCTI** when the scenario comes from an OpenCTI security coverage.

The **Statistics** tab shows a custom dashboard of the scenario: **Select a dashboard** to choose it.

## Find a scenario in the list

:menu[Scenarios] lists the scenarios of the tenant, the most recently updated first, with their **Severity**, **Category**, **Type**, **Status**, **Platforms** and **Tags**. **Status** reads **Scheduled** for a scenario with a schedule, **Not planned** otherwise.

- Search by name, or filter on **Category**, **Severity**, **Type**, **Recurrence**, **Platforms**, **Kill chain phase**, **Tags** or **Updated at**. The list keeps your filters for your next visit.
- **Export this list** downloads the scenarios shown as a CSV file.
- Tick scenarios, or the box of the header for all of them, then **Delete** to delete them at once. It needs **Delete assessment**.

## Duplicate, export and import

The menu of the header offers **Update**, **Duplicate**, **Export** and **Delete**.

- **Export** downloads a `.zip`: the scenario and its injects with their files, and, if you tick them, the **Teams**, the **Persons** and the **Variable values**. Variable values can hold the addresses and other values your injects use: tick them only for a file you trust.
- To import it, click **Import a scenario**, the upload icon at the top of :menu[Scenarios], and choose the `.zip`. It can be an export you unzipped, edited and zipped again with any archive tool: OpenAEV recognizes the scenario, its payloads and its files by their names and content. A `.zip` that holds no scenario, simulation or payload is refused.
- **Import from Hub**, at the top of the list, opens the scenario library of XTM Hub when the platform is registered with it: deploy a scenario from XTM Hub and it appears in your list. See [Connect to XTM Hub](/openaev/administer/xtm-hub/).

:::example[Replay Velvet Harbor on the front desks every week]
The validation engineer of Grand Meridian Hotels wants to know each week whether Meridian EDR still stops what Velvet Harbor did on the night of 9 January 2026: a fake booking request opened at the front desk, a PowerShell loader, the shared `night.audit` account, and guest records copied from the booking database server.

| Where | Value |
| --- | --- |
| Type | **Time-based scenario** |
| **Name** | `Velvet Harbor replay` |
| **Category**, **Main focus**, **Severity** | **Attack Scenario**, **Endpoint Protection**, **High** |
| **Tags** | `velvet-harbor` |
| **Scenario assistant**, **Asset groups** | `Front desk workstations`: the five Windows x86_64 workstations of the French and Spanish hotels |
| **Number of injects by TTP** | 1 |
| Techniques | Spearphishing Attachment (T1566.001), PowerShell (T1059.001), Valid Accounts (T1078), Data from Local System (T1005), Web Protocols (T1071.001) |
| **Scheduling** | **Weekly**, **Day of week** Wednesday, **Time** 06:00, from 14 January 2026 |

The assistant creates four injects and one placeholder, `[T1078] Placeholder - Windows x86_64`: no action of the arsenal signs in with a valid account on Windows. Every Wednesday morning, a simulation runs the four techniques on the workstations, and the **Posture trend** shows whether Meridian EDR prevented and detected them.
:::

## What's next

- [Run a simulation and read its results](/openaev/use/simulations/).
- [Check one attack technique with an atomic test](/openaev/use/atomic-testing/), to try an action before you add it.
- [Turn an OpenCTI security coverage into a scenario](/openaev/use/scenarios/from-opencti/), to let the threat intelligence build the scenario.
- [Import a timeline of injects from a spreadsheet](/openaev/use/scenarios/import/).
