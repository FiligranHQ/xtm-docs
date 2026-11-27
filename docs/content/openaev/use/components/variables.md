---
title: Send each player their own email with variables
description: Write an email or SMS inject once and let OpenAEV fill in each player's name, the exercise and your own values, such as the crisis bridge link.
doc_type: how-to
products:
  - openaev
personas: [exercise-planner]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/components/variables/
---

A variable is a placeholder, such as `${user.firstname}` or `${crisis_bridge_url}`, that OpenAEV replaces when it sends an inject. You write the email of the crisis once, and every player receives it with their own name, the name of the exercise and the values you defined for this scenario. This page lists the variables OpenAEV provides, creates your own, and uses both in an inject.

## Before you start

- Your role has **Manage assessment**, or your group has a **Manage+Delete** grant on the scenario or simulation.
- The scenario plays its injects on a timeline. A chained scenario has no **Configuration** button, and so no variables of its own.

## How a variable reaches the player

::diagram{id=openaev-variables-resolution}

OpenAEV replaces the variables in the subject and the body of the emails that injects send, and in the text of SMS messages. It does so when the inject runs, for each person who receives it: the same body gives one email per player, each with that player's values.

A variable OpenAEV cannot resolve stays in the text as you wrote it, `${crisis_bridge_url}` included. Test an inject before the exercise: see [Test injects](/openaev/use/injects/tests/).

## The variables OpenAEV provides

**Available variables**, at the top of the **Inject data** section of an inject's form, lists them on its **Builtin variables** tab, and your own on **Custom variables**.

| Variable | Replaced with |
| --- | --- |
| `${user.firstname}`, `${user.lastname}` | The first and last name of the player who receives the inject |
| `${user.email}`, `${user.phone}`, `${user.lang}` | The player's email address, phone number and language |
| `${user.id}`, `${user.pgpKey}` | The player's identifier in OpenAEV and PGP key |
| `${exercise.name}`, `${exercise.description}`, `${exercise.id}` | The name, description and identifier of the simulation |
| `${teams}` | The list of the team names the inject targets |
| `${player_uri}` | The link to the player interface |
| `${challenges_uri}` | The link to the player's challenges page |
| `${lessons_uri}` | The link to the player's lessons learned questionnaire |

Two injects add their own: **Publish a media pressure** adds `${articles}`, the published articles, each with `name` and `uri`, and **Publish challenges** adds `${challenges}` in the same form. Their default email bodies already use them.

The links and `${exercise.*}` exist only when the inject runs in a simulation. Each link carries a token for its player: forwarding the email forwards the access. See [Protect the links of emails](/openaev/administer/url-access-token/).

## Create your own variables

:::steps

1. Open the scenario or the simulation and click **Configuration** in its header. The **Scenario configuration** panel opens, or **Simulation configuration**.
2. Open the **Variables** tab and click **Add variable**.
3. Fill in the **Key**: lowercase letters and underscores only, such as `crisis_bridge_url`. You write it as `${crisis_bridge_url}` in the injects.
4. Fill in the **Value**, the text that replaces it.
5. Optionally, add a **Description**, to tell the other planners what it is for.
6. Click **Create**.

:::

The variable shows in the list with its **Key**, **Description** and **Value**. **Update** and **Delete**, in the menu of its line, change or remove it.

A value is plain text, the same for every player. Avoid the names OpenAEV provides, such as `user`, `teams` or `exercise`: your value would replace OpenAEV's.

A simulation created from a scenario copies the scenario's variables. Changing a variable of the scenario later does not change the simulations already created: change it in the simulation too.

::screenshot{id=openaev-variables-configuration alt="The Variables tab of the Scenario configuration of Velvet Harbor crisis night: the variables crisis_bridge_url and hotel_group, with their description and value."}

## Use variables in an inject

Type the variable where the text should change, in the subject or the body. For a list, such as `${teams}` or `${articles}`, the body uses the FreeMarker list syntax:

```text
Teams on the bridge: <#list teams as team>${team} </#list>
<#list articles as article>- ${article.name}: ${article.uri}<br/></#list>
```

:::example[Call the Grand Meridian crisis cell to the bridge]
On the night of 9 January 2026, Example CERT plays the Velvet Harbor intrusion to the crisis cell of Grand Meridian Hotels. In the scenario **Velvet Harbor crisis night**, the exercise planner creates two variables:

| Key | Value | Description |
| --- | --- | --- |
| `hotel_group` | `Grand Meridian Hotels` | The company the exercise is about. |
| `crisis_bridge_url` | `https://bridge.example.com/meridian-crisis` | The conference bridge of the crisis cell. |

The email inject **Call the crisis cell to the bridge**, at 22:40, reads:

```text
Subject: ${hotel_group}: crisis cell on the bridge now

Hello ${user.firstname},

Meridian EDR blocked a PowerShell loader on a Paris front-desk workstation and guest
records may have left the booking database. Join the bridge now: ${crisis_bridge_url}

Teams called: <#list teams as team>${team} </#list>
```

Hélène Marchetti, the CISO, receives "Hello Hélène," with the bridge link. When Example CERT plays the scenario for another member chain, it changes the two values in the copy, not the emails.
:::

## What's next

- [Publish fake press articles during an exercise](/openaev/use/components/media-pressure/).
- [Test injects](/openaev/use/injects/tests/) to read the email a player will receive.
- [Built-in injectors](/openaev/use/injectors/built-in/) for the email and SMS injects.
