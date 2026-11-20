---
title: How a scenario becomes a simulation
description: What a scenario holds, the two kinds of scenarios, what a simulation copies when it starts, and how runs of the same scenario show your posture over time.
doc_type: explanation
products:
  - openaev
personas: [validation-engineer, exercise-planner]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/scenarios-and-simulations/
---

A scenario describes what you test: a threat, such as an intrusion set or an attack campaign, turned into a sequence of actions called injects, aimed at your assets and your people, each with the results you expect from your defenses. A simulation is one run of that scenario. You launch a scenario as often as you like, by hand or on a schedule, and each launch creates a new simulation with its own results. Comparing the simulations of one scenario shows whether your defenses get better against that threat. Scenarios are under :menu[Scenarios], simulations under :menu[Simulations]; reading them takes the **Access assessment** capability, launching them **Launch assessment**.

## One scenario, many runs

::diagram{id=openaev-scenarios-simulations-runs}

When a simulation starts, it copies the scenario as it is at that moment: its injects with their targets and expectations, its teams with the persons enabled in each, its media pressure articles, its lessons learned questions and its tags. The simulation then lives on its own:

- A change to the scenario, such as a new inject or another target, applies to the next simulations, never to the ones already created.
- A change to a simulation, such as an inject added during the run, stays in that simulation and never goes back to the scenario.

So each simulation is a faithful record of what was played and how your defenses answered at the time. The scenario keeps the link to all of them.

## What a scenario holds

| Part | What it is | Where you set it |
| --- | --- | --- |
| Injects | The actions of the attack or of the exercise: a command on an endpoint, an email, an SMS, a media article, a manual action. See [How an attack becomes injects and expectations](/openaev/concepts/injects-expectations/) | **Injects** tab, or **Logic** tab for a chained scenario |
| Targets | The assets and asset groups an inject acts on, and the teams whose persons receive it. A team must be added to the scenario before an inject can target it | Each inject; the teams in **Configuration**, **Teams** tab. See [How injects reach assets and people](/openaev/use/targets/) |
| Expectations | What must happen for each inject to succeed: prevented, detected, not vulnerable, or a reaction of the targeted persons | Each inject |
| Context | **Variables** reused in inject contents, **Media pressure** articles, challenges and documents that make an exercise believable | **Configuration**; the components of each inject |
| Properties | **Name**, **Category**, **Main focus**, **Severity** and tags. A tag can also add asset groups to the injects by default: see [Set default asset groups by scenario tag](/openaev/configure/asset-rules/) | The scenario's **Update** form |

The categories are Global Crisis, Attack Scenario, Media Pressure, Data Exfiltration, Capture The Flag, Vulnerability Exploitation, Lateral Movement and URL Filtering. They classify your scenarios for filters and dashboards, and change nothing in how a scenario runs.

::screenshot{id=openaev-scenario-header alt="The header of the scenario Velvet Harbor replay: Attack Scenario, severity high, not scheduled, its four injects, two teams, five persons and two asset groups, and its tabs from Overview to Statistics."}

## Time-based and chained scenarios

You choose the kind of scenario when you create it, in **Create a new scenario**:

| | **Time-based scenario** | **Chained scenario** |
| --- | --- | --- |
| How it runs | Each inject starts at a fixed delay after the start of the simulation, on a timeline | Each step triggers the next possible steps, from the results of the previous ones: an attack path |
| Tabs | **Overview**, **Injects**, **Tests**, **Execution**, **Findings**, **Statistics** | **Overview**, **Scope**, **Logic**, **Attack Path**, **Execution**, **Findings**, **Statistics** |
| Use it for | Replaying a known threat step by step, a tabletop or crisis exercise, a phishing campaign | Following an attack across your network as far as it can go, inside a scope you authorize |
| Edition | Every edition | Enterprise Edition |

A time-based inject can still wait for another: it then runs only when the conditions on its parent are met, such as the parent being executed or one of its expectations failing. A chained scenario needs its **Scope** before it can launch. See [How a chained scenario works](/openaev/use/attack-chaining/).

When the scenario enables lessons learned, both kinds get a **Lessons learned** tab, for the questions players answer after the exercise.

## Launch, schedule or run once

A scenario runs in three ways, all from its header:

- **Launch** starts a simulation now, after a confirmation. You land on the new simulation.
- **Scheduling** runs it at a set time: **Once**, **Hourly**, **Daily**, **Weekly** or **Monthly**, from a start date and until an optional end date. Each occurrence creates a new simulation. A recurring scenario is how you follow a threat over time.
- A simulation created directly from :menu[Simulations] with **Create** has no scenario. Use it for a one-off exercise: it cannot be compared with other runs.

A simulation shows its status in the list: **Draft** when it has no start date yet, then **Scheduled**, **On-going**, **Paused**, **Finished** or **Canceled**.

## Compare the runs of a scenario

Two tabs of the scenario read its simulations:

- **Execution** shows the latest simulation as it runs: its timeline and the result of each inject.
- **Statistics** aggregates every simulation of the scenario on a dashboard, so you see the trend: are more steps of the threat detected than last month? The dashboard is the scenario's own, or the platform's default scenario dashboard. See [Build dashboards of your validation results](/openaev/use/dashboards/).

To test one action without building a scenario, use an atomic test: one inject, run at once on the targets you pick, with its own results. See [Check one attack technique with an atomic test](/openaev/use/atomic-testing/).

:::example[Replay Velvet Harbor every week]
The validation engineer of Grand Meridian Hotels creates the **Time-based scenario** `Velvet Harbor replay`, **Category** Attack Scenario, **Severity** High, tag `velvet-harbor`:

- Four injects: **Fake summit booking request**, an email to the teams **Front desk France** and **Front desk Spain** at the start; **PowerShell loader** on the asset group **Front desk workstations** 15 minutes later (T1059.001); **Local guest records** on the same group at 30 minutes (T1005); and **HTTPS beacon** on the same group at 45 minutes (T1071.001).
- **Scheduling**: **Weekly**, every Wednesday at 06:00.

The first simulation, on 14 January 2026, shows the loader blocked by Meridian EDR on four workstations, and neither blocked nor detected on `fd-barcelona-03`. Once the EDR agent of that workstation is restarted, the simulation of 21 January shows it blocked everywhere. The scenario's **Statistics** tab shows both runs side by side, while each simulation keeps the result of its own week.
:::

:::xtm-one[Build a scenario with AI]
When OpenAEV is registered with XTM One, the **Scenario assistant** builds a time-based scenario from a threat report: it proposes the techniques to cover and generates their injects. For a chained scenario, **Generate with AI** lets the orchestrator design the attack path from an objective, templates, agents and a scope. See [XTM One in OpenAEV](/xtm-one/use/in-openaev/).
:::

## What's next

- [Build a scenario that replays an attack](/openaev/use/scenarios/build/): create the scenario and its injects.
- [Run a simulation and read its results](/openaev/use/simulations/): launch it and follow its run.
- [How an attack becomes injects and expectations](/openaev/concepts/injects-expectations/): what each inject must achieve.
- [Turn an OpenCTI security coverage into a scenario](/openaev/use/scenarios/from-opencti/): start from a threat known to OpenCTI.
