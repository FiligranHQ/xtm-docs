---
title: Read how far a chained simulation got
description: Follow a chained run live, then read its attack path graph to see which endpoints an attack reached, what it found, and which endpoint to fix first.
doc_type: how-to
products:
  - openaev
personas: [validation-engineer, soc-manager]
owner: docs-openaev
last_reviewed: 2026-11-07
edition: ee
migrated_from:
  - openaev:/usage/attack-chaining/attack-path-map/
---

Once a chained simulation runs, its **Attack Path** tab rebuilds what really happened: which endpoints the chain reached, in what order, what it found on each, and which finding triggered the next step. It answers the question a chained scenario asks, how far an attacker got, and points at the one endpoint to fix that closes the most routes. This page follows a run on the **Execution** tab, then reads the graph. The chain it reads is the one built in [Build the logic of a chained scenario](/openaev/use/attack-chaining/logic/).

## Before you start

- The Enterprise Edition is active, and the simulation is a chained one that has started. The **Attack Path** tab shows only for chained simulations.
- Your role has **Access assessment** to read the simulation.
- The simulation ran enough steps to have reached endpoints and produced findings.

## Follow the run on the Execution tab

While the chain runs, the **Execution** tab is where it happens. It shows:

- A header with the elapsed time and the counts of **Processed injects**, **Pending injects**, **Execution errors** and **Pending validations**.
- An **Attack timeline** of when each inject fired.
- A board where injects move from **Up next** to **In flight** to **Completed** as the engine readies and resolves them. Unlike a time-based simulation, the full list is not known in advance.

::screenshot{id=openaev-attack-chaining-execution alt="The Execution tab of the live run Velvet Harbor path - booking web front to database: 2 processed injects out of 4, the attack timeline of the OpenAEV implant, and the board with the discovery and the export Completed on one endpoint and still In flight on the other."}

If an action you expect never shows, its event's conditions are not met yet (see [Build the logic of a chained scenario](/openaev/use/attack-chaining/logic/)), or the run hit its time out or rate limit (see [Set the targets and limits of a chained scenario](/openaev/use/attack-chaining/scope/)). Once enough actions have run, open the **Attack Path** tab.

## Read the attack path graph

::diagram{id=openaev-attack-chaining-attack-path-map-run}

::screenshot{id=openaev-attack-chaining-attack-path alt="The Attack Path tab of the same run: 2 targets reached, 1 captured credential, 1 detected CVE, 2 port scans and 1 top chokepoint; the graph runs from the discovery action through booking-web-01 to its four findings, the finding 10.20.8.15:5432 triggering Booking database reachable and the export action, booking-db-01 in green and booking-web-01 in orange."}

The graph is built from what really executed. It joins:

| Shape | What it is |
| --- | --- |
| Square (**Injector**) | An executed action. Click it for the inject it ran and its results |
| Dashed circle | A cluster of endpoints reached at that point. Click to expand into single endpoints |
| Pill | A cluster of findings. Click to expand |
| Circle | A single finding, or a single endpoint once expanded |

Colors give the verdict at a glance:

| Color | Meaning |
| --- | --- |
| Green | Prevented on every endpoint |
| Orange | Detected, or prevented or detected on only some of the endpoints |
| Red | Neither prevented nor detected |
| Highlighted node | A chokepoint, the most exposed endpoint |
| Highlighted edge | An **Event link**: a finding that triggered the next action |
| Primary color path | The path you selected by clicking a node or a table row |

A **Legend** at the bottom left recalls the shapes and colors. It starts collapsed and expands on click.

## Find what to fix first

The summary cards above the graph count what the run found: **Targets reached**, **Discovered Shares**, **Captured Files**, **Captured Credentials**, **Discovered Users**, **Detected CVEs**, and a card for any other finding type the run produced. The **Top chokepoints** card ranks the most exposed endpoints.

:::steps
1. Narrow the graph with the search box and the filters: kill chain phase, finding type, criticality.
2. Open the **Top chokepoints** card and pick the top endpoint. The graph highlights every action, event and finding that led to it.
3. Follow its path and open the **Injector** node of the action that reached it, to start fixing the cause: patch the host, change a control, add a detection rule.
4. To work from a finding instead, pick one, such as a captured credential: the graph highlights the path that produced it, and the action that produced it.
:::

A chokepoint is scored by its finding count weighted by the endpoint's asset criticality: **VERY_HIGH** counts four times a **LOW** or unset one, **HIGH** three times, **MEDIUM** twice. Because many paths cross a chokepoint, fixing it closes the most routes at once. The card holds the same explanation.

Switch between the **Graph** and the **Table** view at the top right. **Export as PNG** downloads the whole graph, whatever is on screen, for a report. From a **Finding** node, the **Findings** tab of the simulation holds every finding across the run: see [Findings](/openaev/use/findings/).

## What you see when it worked

- The graph shows at least one action, the endpoints it reached and the findings it produced.
- A red or orange action is one your controls did not stop: the attacker got that far.
- The **Top chokepoints** card names the endpoint whose findings, weighted by criticality, are the heaviest.

:::example[How far Velvet Harbor's path reached the booking database]
On 12 April 2026, the validation engineer of Grand Meridian launches `Velvet Harbor path - booking web front to database`, scoped to the two booking servers. On the **Attack Path** tab, a few minutes later:

- `Discover the booking web front` ran on `booking-web-01` and on `booking-db-01`, the two Linux endpoints of the allow list. On `booking-web-01` it found two **Ports scan** findings, among them `10.20.8.15:5432`, the PostgreSQL port of the booking database, a **Credentials** finding, the account of the booking application, and the CVE `CVE-2021-42013` of its Apache. `booking-web-01` is orange: Meridian EDR detected the action there without blocking it.
- Because **Booking database reachable** held on `10.20.8.15:5432`, an **Event link** leads to `Export the booking database`. It ran on both servers; on `booking-db-01` Meridian EDR blocked it, and the endpoint is green.
- `booking-web-01` is the **chokepoint**: its four findings, weighted by its criticality, carry the whole path. Hélène Marchetti's team reads it as the gap to close first: patch its Apache, move the database password out of its configuration, and extend the prevention of the booking database to the web front.
:::

## What's next

- [Findings](/openaev/use/findings/): every finding of the run, across simulations.
- [Build the logic of a chained scenario](/openaev/use/attack-chaining/logic/): adjust the chain from what you saw.
- [Set the targets and limits of a chained scenario](/openaev/use/attack-chaining/scope/): the scope and limits that bounded the run.
- [Let AI run a penetration test](/openaev/use/attack-chaining/autonomous/): let an AI find paths for you.
