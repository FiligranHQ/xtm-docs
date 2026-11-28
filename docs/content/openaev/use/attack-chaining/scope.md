---
title: Set the targets and limits of a chained scenario
description: Fill the allow and deny lists of a chained scenario, add the variables its actions reuse, and bound the run with a time out and a rate limit.
doc_type: how-to
products:
  - openaev
personas: [validation-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
edition: ee
migrated_from:
  - openaev:/usage/attack-chaining/scope-definition/
---

The scope of a chained scenario says what its run may touch and how far it may go. A chained run starts actions on its own, as its events hold, so the scope is what keeps it on the endpoints and networks you are authorized to test: an allow list of targets, a deny list that always wins, variables the actions reuse, and a time out and a rate limit. This page fills the **Scope** tab. If you are new to chained scenarios, read [How a chained scenario works](/openaev/use/attack-chaining/) first.

## Before you start

- The Enterprise Edition is active, and the scenario was created as a **Chained scenario**.
- Your role has **Manage assessment**. Picking assets and asset groups from the inventory takes **Access assets**; picking teams and persons takes **Access teams & persons**.
- The endpoints the actions run on have an OpenAEV agent. See [OpenAEV agents](/openaev/use/agents/).

## How the scope decides what is reached

::diagram{id=openaev-attack-chaining-scope-targets}

- The allow list holds the targets of the run. Technical actions (commands, scans) use its assets, asset groups, IP addresses, subnets and domains; tabletop actions (emails, SMS) use its teams and persons.
- The deny list removes targets from the run, even those inside an allowed asset group or subnet: an entry in both lists is never reached.
- A value an action discovers, such as the IP address of a host it scanned, feeds the next steps only when the scope allows it.
- A scenario cannot be launched with an empty allow list. Its launch button stays disabled with **A chained scenario requires a defined scope.**, and a health check **Scope definition is required** links back to this tab. The same check blocks a launch when the logic has technical actions and the allow list has no asset, IP address or domain, or tabletop actions and no team or person.

::screenshot{id=openaev-attack-chaining-scope alt="The Scope tab of the chained scenario Velvet Harbor path - front desk to booking database, before its scope is set: the empty Allow list and Deny list with their Define buttons, the Variables, and the Simulation time out, on at one hour, and the Simulation rate limit, off."}

## Fill the allow list

:::steps
1. Open the scenario and its **Scope** tab. Four cards show: **Allow list**, **Deny list**, **Variables**, and the time out and rate limit.
2. On the **Allow list** card, click **Define**. The **Define allowlisted scope** drawer opens.
3. Select the targets on the **Assets**, **Asset groups**, **Teams** and **Persons** tabs.
4. To add targets that are not in the inventory, type IP addresses, subnets in CIDR notation or host names in the box under the tabs, separated by commas, and press Enter. A subnet larger than /24 may not be expanded into its hosts.
5. To add many at once, click **CSV template** to download the expected format, fill it, then click **Add Bulk CSV**. Each row is `type,value`, where the type is `domain`, `ip` or `ip_subnet`; a header row `type,value` is skipped, duplicates are removed, and each invalid row is reported with its row number.
6. Click **Define scope**. The card lists the entries, grouped by kind.
:::

## Carve out the deny list

:::steps
1. On the **Deny list** card, click **Define**. The **Define denylisted scope** drawer opens, with the same tabs and the same manual and CSV entry.
2. Select or type what the run must never touch: a production server inside an allowed subnet, an executive inside an allowed team.
3. Click **Define scope**.
:::

## Add variables

A variable is a value you set once and that actions read instead of a typed value: an account name, a share path, a port. It matters most when no action produces that value, since an event or an argument can then only get it from a variable. Variables are optional.

:::steps
1. On the **Variables** card, click **Add variable**.
2. In **Create a new variable**, set the **Key**, the **Type**, the **Value** and, optionally, a **Description**. The type is what links the variable to the actions: an argument of type **Username** can read a variable of type **Username**. The value must fit the type, such as a valid IP address for **IPv4**, or the dialog shows an error. Two variables cannot share both key and type.
3. Click **Create**.
:::

How actions read variables is in [Choose the actions of a chained scenario](/openaev/use/attack-chaining/actions/#feed-an-argument-from-earlier-steps).

## Bound the run

| Setting | What it does | Default |
| --- | --- | --- |
| **Simulation time out** | Stops the whole run after this duration, set in **Hours** (0 to 23) and **Minutes**, whatever is still running. The time a simulation spends paused does not count. Turned off, the run ends only when no action is left to run or when you stop it | On, 1 hour, for a new scenario |
| **Simulation rate limit** | Caps how many injects of the run start within a sliding window: **Max Attempts** (1 to 99) per number of **Minutes** (1 to 59). An action due once the cap is reached is delayed until older ones leave the window, not dropped | Off. Turned on, 1 attempt per 30 minutes until you change the values |

Turn each one on with its switch, then choose the values. A rate limit models a slow, quiet attacker, or keeps a password test under the lockout threshold of your directory.

## Check the scope

- The **Allow list** and **Deny list** cards show the count of their entries, and each entry its name.
- The health checks of the scenario no longer show **Scope definition is required**, and its launch button is enabled.
- After a launch, the simulation keeps the scope it started with. A banner on its **Scope** tab tells you when an asset of that scope changed in the inventory since then.

To reuse the scenario on another platform, export it from its menu: **Export the scenario** offers **Scope definition** for a chained scenario, to carry the allow and deny lists with the **Actions & Events**.

:::example[Bound a night test of the front desk]
After Velvet Harbor's loader ran on a front-desk workstation of Grand Meridian Hotels on 9 January 2026, the validation team prepares a chained scenario that starts from that workstation and tries to reach the booking database server, at night, without touching the payment gateway that sits in the same server network.

| Card | Value |
| --- | --- |
| **Allow list** | Asset `fd-paris-01` (the Paris front-desk workstation, with its OpenAEV agent), and the subnet `10.20.8.0/24` of the booking servers, imported from a CSV row `ip_subnet,10.20.8.0/24` |
| **Deny list** | `10.20.8.40`, the payment gateway, typed by hand |
| **Variables** | Key `front_desk_account`, type **Username**, value `night.audit`, description "Shared night auditor account" |
| **Simulation time out** | On, 4 hours: the test runs between 23:00 and 03:00 |
| **Simulation rate limit** | On, 5 **Max Attempts** per 30 **Minutes**, under the lockout threshold of the Meridian directory |

The run may scan and query the hosts of the booking network, but no value pointing at `10.20.8.40` ever reaches an action, and the run stops by 03:00 at the latest.
:::

:::xtm-one[Let the AI ask for the scope]
An autonomous run of a chained scenario takes its own allow and deny lists in its launch drawer. Left empty, the XTM One orchestrator asks you which targets are in scope before it attacks anything. See [Let AI run a penetration test](/openaev/use/attack-chaining/autonomous/).
:::

## What's next

- [Choose the actions of a chained scenario](/openaev/use/attack-chaining/actions/)
- [Build the logic of a chained scenario](/openaev/use/attack-chaining/logic/)
- [Read how far a chained simulation got](/openaev/use/attack-chaining/attack-path-map/)
