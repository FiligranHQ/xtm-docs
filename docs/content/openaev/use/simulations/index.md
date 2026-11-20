---
title: Run a simulation and read its results
description: Start a simulation from a scenario or on its own, follow it while it runs, and read which attacks your defenses prevented and detected and how your teams responded.
doc_type: how-to
products:
  - openaev
personas: [validation-engineer, exercise-planner, soc-manager]
owner: docs-openaev
last_reviewed: 2026-11-10
migrated_from:
  - openaev:/usage/simulation/
---

A simulation is one run of an attack or an exercise on your environment: its injects run at their time, your security tools and your teams react, and OpenAEV scores the result on four axes, prevention, detection, vulnerability and human response. Most simulations come from a scenario, launched by hand or on its schedule, so that successive runs can be compared. This page starts a simulation, follows it, and reads its results. To build what it runs, see [Build a scenario that replays an attack](/openaev/use/scenarios/build/).

## Before you start

- Your role has **Launch assessment** to start, pause, stop or reset a simulation, and **Manage assessment** to create or change one. **Access assessment** is enough to read the results.
- Collectors read your security tools, so that detection and prevention can be scored: see [Collectors](/openaev/use/collectors/). When no collector answers that type of expectation for the security platforms an inject expects, its detection or prevention expectation fails at once, and reads **No automated security platform detected**. A collector you **Stop** in :menu[Integrations] no longer counts for the injects that run afterwards.

## Start a simulation

- **From a scenario**: click **Launch** in its header, or let its schedule create the simulations. Each simulation is a copy of the scenario at that moment. See [Launch or schedule it](/openaev/use/scenarios/build/#launch-or-schedule-it).
- **On its own**, for a one-off exercise: open :menu[Simulations], click **Create**, choose **Time-based simulation**, and fill in the same fields as a scenario, plus a **Start date (optional)** in the future. **Chained simulation** requires the Enterprise Edition.
- **From a file**: click **Import a simulation** at the top of :menu[Simulations] and choose an export.

A simulation without a start date is a **Draft**; with one, it is **Scheduled** and starts on its own. **Start now** starts it at the next full minute.

Before you start it, you can still change it without touching the scenario: its injects on the **Injects** tab, and its teams, variables and media pressure articles in **Configuration**. Teams and documents are shared with the scenario: a change to a team applies to both.

## Follow it while it runs

::diagram{id=openaev-simulations-statuses}

| Status | What happens | Actions in the header |
| --- | --- | --- |
| **Draft**, **Scheduled** | Nothing runs yet | **Start now**, **Scheduling** |
| **On-going** | The injects run at their time; collectors and players answer the expectations | **Pause**, **Stop** |
| **Paused** | The injects wait | **Resume**, **Stop** |
| **Finished** | Every inject ran and every expectation is answered or expired | **Reset** |
| **Canceled** | Stopped by hand; the results collected so far are kept | **Reset** |

::screenshot{id=openaev-simulations-execution alt="The simulation Velvet Harbor crisis night On-going, with Pause and Stop in its header, its three injects, one team, three persons, a media pressure and a challenge, and under Results the human response still at 0 % with one validation needed."}

Every action asks for a confirmation. A simulation does not finish when its last inject is sent: it stays **On-going** until each expectation is answered, or expires. By default, a detection, prevention or vulnerability expectation expires after 6 hours, and an expectation of people after 24 hours, set by `openaev.expectation.technical.expiration-time` and `openaev.expectation.human.expiration-time`. See the [configuration reference](/reference/openaev/configuration/#openaev-expectation).

The **Execution** tab follows the run, with a menu on the right:

- **Overview**: the timeline of the injects, with a cursor on the current time, and the injects to come, running and done.
- **Mails**: the emails and SMS sent, by inject, person and team.
- **Validations**: the expectations of people waiting for an evaluator. See [Validate expectations](/openaev/use/expectations/validation/).
- **Simulation logs**: notes the animation team writes during the run.

::screenshot{id=openaev-simulations-mails alt="The Mails menu of the Execution tab: the inject Call the crisis cell to the bridge, sent to three persons, with its five mails, the three sent and the two replies of the crisis cell."}

:::warning
**Reset** deletes the results of the simulation: inject statuses, expectations, findings, lessons learned answers, and its files. **Stop** keeps them. To keep a run's results and run it again, launch a new simulation from the scenario instead.
:::

## Read the results

The **Overview** tab has four parts:

- **Information**: the fields of the simulation and its scenario.
- **Results**: a gauge per axis, **Prevention**, **Detection**, **Vulnerability** and **Human response**. Each gauge reads its **Resilience**: the share of all its expectations that succeeded, pending ones included, so the figure rises while successes come in. Under the ring, the gauge counts its **Success**, **Partial**, **Failed** and **Pending** expectations. Its color goes from red below 25 to green from 75, and stays grey while every expectation is pending.
- **Kill chain results**: the techniques of the injects on the matrix, colored by their results.
- **Injects results**: each inject with its targets and its results; click one to open its details, its execution traces and what each security platform saw. See [Inject results](/openaev/use/injects/results/).

Each expectation counts as **Success** when its score reaches the expected score, **Failed** at 0, **Partial** in between, and **Pending** until it is answered. Per axis, the results read:

| Axis | Success | Partial | Failed |
| --- | --- | --- | --- |
| **Prevention** | Prevented | Partially Prevented | Not Prevented |
| **Detection** | Detected | Partially Detected | Not Detected |
| **Vulnerability** | Not vulnerable | Partially Vulnerable | Vulnerable |
| **Human response** | Successful | Partial | Failed |

An expectation that expires unanswered fails: a technique that no collector reported counts as not detected. Before you conclude that an attack went unseen, check on the inject that the collector of that security platform ran.

The other tabs complete the picture:

- **Injects**, in its **Distribution view**: charts of the injects and their expectations by type and by team. **Distribution of expected total score by inject type** draws the score each inject type gathers over time, technical injects included, each result counted once.
- **Findings**: what the injects found, such as open ports, credentials or vulnerabilities. See [Findings](/openaev/use/findings/).
- **Lessons learned**: the players' answers, when enabled. See [Lessons learned](/openaev/use/components/lessons/).
- **Statistics**: a custom dashboard of the simulation.

**View in scenario** opens the scenario, whose overview compares the runs.

:::example[Read the weekly Velvet Harbor run on the front desks]
On Wednesday 11 February 2026 at 06:00, the scenario `Velvet Harbor replay` creates its weekly simulation: the email **Fake summit booking request** to the two front-desk teams, then three actions on the five front-desk workstations of Grand Meridian Hotels. It stays **On-going** until 12:05, when the last detection expectation of the Meridian EDR collector is answered, then reads **Finished**.

| Inject | Prevention | Detection |
| --- | --- | --- |
| PowerShell loader (T1059.001) | Prevented | Detected |
| Local guest records (T1005) | Prevented | Detected |
| HTTPS beacon (T1071.001) | Not Prevented | Detected |

The **Results** gauges read **Prevention** 67 and **Detection** 100. The validation engineer opens the HTTPS beacon inject: Meridian EDR raised an alert on the command and control traffic but let it through. They send the inject's details to the SOC, which adds a blocking rule; next Wednesday's run shows whether **Prevention** reaches 100.
:::

:::xtm-one[Explain a run with XTM One]
With XTM One connected and the Enterprise Edition, **Ask Ariane** in the top bar reads the simulation you are on, as you. Ask it which techniques were not prevented and why, or to turn the run into a readout or a slide deck for the CISO, which you download as PDF or PowerPoint. See [XTM One in OpenAEV](/xtm-one/use/in-openaev/).
:::

## Find a simulation in the list

:menu[Simulations] lists every simulation of the tenant, those created by scenarios included, the most recently updated first. Each line gives its **Start time**, **Status**, **Target**, **Global score** and **Tags**, so the running ones and the latest scores show at a glance.

- Search by name, or filter on **Scenario**, **Status**, **Severity**, **Start date**, **Kill chain phase**, **Tags** or **Updated at**. A filter on **Scenario** lists the runs of one scenario. The list keeps your filters for your next visit.
- **Export this list** downloads the simulations shown as a CSV file.
- Tick simulations, then **Delete** to delete them at once, with **Delete assessment**. Deleting an **On-going** or **Paused** simulation stops it.

## What's next

- [Share simulation results in a report](/openaev/use/simulations/reports/).
- [Validate expectations](/openaev/use/expectations/validation/), for the answers of people.
- [Check one attack technique with an atomic test](/openaev/use/atomic-testing/), to check a fix without the whole scenario.
- [Scenarios and simulations](/openaev/concepts/scenarios-simulations/), for the model behind them.
