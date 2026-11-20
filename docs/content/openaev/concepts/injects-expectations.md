---
title: How an attack becomes injects and expectations
description: How OpenAEV turns each step of a threat into an inject, what an inject is made of, the three families of expectations it carries, and how their results become your security posture.
doc_type: explanation
products:
  - openaev
personas: [validation-engineer, soc-analyst, exercise-planner]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/injects-and-expectations/
---

A threat is a series of actions: an intrusion set sends a phishing email, runs a script, steals an account, takes data away. OpenAEV replays each action as an inject: a command on an endpoint, an email to a team, a media article, a request to the SOC. Each inject carries expectations, the reactions you expect from your defenses and your people: block the command, raise an alert, report the email. OpenAEV plays the injects, collects what happened, and scores each expectation. The scores, added up, are your security posture against that threat. This page explains the model; how the answers are combined into a score is detailed in [How expectations score an inject](/openaev/use/expectations/).

## From an action to a score

::diagram{id=openaev-injects-expectations-scoring}

## What an inject is made of

| Part | What it decides | Read |
| --- | --- | --- |
| **Type** | What the inject does: an action of the Threat Arsenal run on endpoints, or an inject type of an injector, such as an email, an SMS, an HTTP request, a media article, a challenge or a manual action. The type also decides which targets the inject accepts and which expectations it proposes | [Build the actions of your threat arsenal](/openaev/use/threat-arsenal/), [How injectors deliver your injects](/openaev/use/injectors/) |
| **Targets** | Who or what it acts on: assets and asset groups for an action on endpoints, teams for an inject that reaches people | [How injects reach assets and people](/openaev/use/targets/) |
| **Time** | When it runs: at a delay after the start of the simulation, or once its parent inject meets a condition, such as being executed or one of its expectations failing | [Run injects on conditions and move them between scenarios](/openaev/use/injects/chaining/) |
| **Expectations** | What must happen for the inject to succeed, under **Inject expectations** in its form. The type proposes them; you keep, change or remove them | [How expectations score an inject](/openaev/use/expectations/) |

Who plays the inject depends on its type. An action on endpoints is run by an agent on each targeted host; an email, an SMS or an HTTP request is sent by its injector. Both paths are drawn in [How OpenAEV works](/openaev/concepts/architecture/).

## Three families of expectations

| Family | Expectation types | It checks that | Answered by |
| --- | --- | --- | --- |
| Your controls | **Prevention**, **Detection** | A security tool blocked the action, or raised an alert about it | The collectors of your security tools, which match their alerts and logs to the inject. See [How collectors measure what your security tools saw](/openaev/use/collectors/) |
| Your exposure | **Vulnerability** | The target has none of the vulnerabilities the inject looks for | The output of the inject, such as a vulnerability scan |
| Your people | **Manual**, **Challenge**, and **Automatic - Triggered when target reads the articles** | A team reacted as your procedures require, solved a challenge, read the media articles | A user who judges the reaction by hand, or OpenAEV itself when a person submits a flag or opens an article |

Dashboards add the last family up as **Human Response**. Every expectation ends as a success or a failure, and shows **Pending** while it waits: a result brings a score, and the expectation succeeds when the score reaches its **Success score**. An expectation that gets no answer before its **Expiration time** fails, except a vulnerability expectation, which then counts as not vulnerable: nothing was found.

Each target is scored on its own: every asset and agent of an asset group, every enabled person of a team. The **Validation mode** of an expectation decides whether the whole group must succeed, or one member is enough.

## From expectations to your posture

The results of expectations feed everything OpenAEV tells you about your defenses:

- **The result of each inject** and of each simulation: which steps of the threat were stopped, seen, or neither. See [Run a simulation and read its results](/openaev/use/simulations/).
- **The home dashboard and your dashboards**: success rates by expectation type, by security platform, by MITRE ATT&CK technique and by security domain. See [How security domains group your results](/openaev/concepts/security-domains/).
- **The posture score of each asset**: the share of its expectations that succeeded. See [Add the assets your simulations target](/openaev/use/targets/assets/).
- **The rest of the attack**: in a chain, the next inject can wait for the result of an expectation, so the attack goes on only when the previous step got through.

The expectations an injector proposes can change with its versions. An inject keeps the ones it was created with, and OpenAEV shows when they drift from those of its type, so that you choose whether to realign them.

:::example[Turn Velvet Harbor into injects and expectations]
The validation engineer of Grand Meridian Hotels maps each step of Velvet Harbor's intrusion of 9 January 2026 to an inject of the scenario `Velvet Harbor replay`:

| Step of the threat | Inject | Targets | Expectations |
| --- | --- | --- | --- |
| A fake booking request for the European Hospitality Summit 2026 reaches the night auditor (T1566.001) | **Fake summit booking request**, an email | Teams **Front desk France** and **Front desk Spain** | **Manual**: the front desk reports the email to the SOC |
| The attachment starts a loader in PowerShell (T1059.001) | **PowerShell loader**, an action of the Threat Arsenal | Asset group **Front desk workstations** | **Prevention** and **Detection**, expected from an EDR |
| The operators look for exported guest records on the workstation (T1005) | **Local guest records** | Asset group **Front desk workstations** | **Prevention** and **Detection**, expected from an EDR |
| The backdoor beacons to its command and control over HTTPS (T1071.001) | **HTTPS beacon** | Asset group **Front desk workstations** | **Prevention** and **Detection**, expected from an EDR |

After the first run, on 14 January 2026, the scenario reads: the email reported by two persons of five, the loader prevented by Meridian EDR, the search for guest records not detected. The search is the gap to close first.
:::

## What's next

- [How expectations score an inject](/openaev/use/expectations/): how the answers of agents, assets, persons and groups combine, and when they expire.
- [How a scenario becomes a simulation](/openaev/concepts/scenarios-simulations/): where injects live, and what each run keeps.
- [Add injects to scenarios and simulations](/openaev/use/injects/): create the injects of a scenario.
