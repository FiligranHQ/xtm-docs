---
title: How you validate your defenses in OpenAEV
description: The steps of a security validation in OpenAEV, from the targets and the attack to the results on the home dashboard, and the page that covers each step.
doc_type: explanation
products:
  - openaev
personas: [validation-engineer, soc-manager, exercise-planner]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/evaluate/overview/
---

OpenAEV tells you whether your defenses stop a real attack. You describe what you test and what the attacker does, OpenAEV plays it against your endpoints and your people, and your security tools report what they saw. Every action is scored: was it prevented, detected, did a person react as expected? This page gives the order of the work, from the first agent to the trend on the home dashboard, and the page that covers each step. How the platform and its integrations fit together is in [How OpenAEV works](/openaev/concepts/architecture/).

## The steps of a validation

::diagram{id=openaev-use-workflow}

| Step | What you do | Where | Read |
| --- | --- | --- | --- |
| 1. Prepare the targets | Install agents on the endpoints you test, describe the other assets, gather them in asset groups, and organize the people of tabletop and phishing exercises in teams | **Install simulation agents** in the top bar, :menu[Assets], :menu[Asset groups], :menu[Persons], :menu[Teams] | [How injects reach assets and people](/openaev/use/targets/) |
| 2. Choose what the attacker does | Pick the actions of the attack: commands and payloads of the Threat Arsenal, emails, HTTP requests and the other injects of your injectors | :menu[Threat Arsenal], :menu[Integrations] | [Build the actions of your threat arsenal](/openaev/use/threat-arsenal/), [How injectors deliver your injects](/openaev/use/injectors/) |
| 3. Build the scenario | Chain the injects on a timeline, or as an attack path that follows its own results, with the expectations each one must meet | :menu[Scenarios] | [How an attack becomes injects and expectations](/openaev/concepts/injects-expectations/), [Build a scenario that replays an attack](/openaev/use/scenarios/build/) |
| 4. Run it | Launch the scenario now, schedule it, or repeat it: each run is a simulation. To check one action first, run it alone as an atomic test | :menu[Simulations], :menu[Atomic testings] | [How a scenario becomes a simulation](/openaev/concepts/scenarios-simulations/), [Run a simulation and read its results](/openaev/use/simulations/), [Check one attack technique with an atomic test](/openaev/use/atomic-testing/) |
| 5. Read the results | Check which expectations succeeded, answer the ones a person must judge, and review the findings: CVEs, open ports, credentials | The simulation's tabs, :menu[Findings] | [How expectations score an inject](/openaev/use/expectations/), [Review what your injects discovered](/openaev/use/findings/) |
| 6. Follow your posture | Watch the trend over runs on the home dashboard, the **Statistics** tab of a scenario, your own dashboards and reports | :menu[Home], :menu[Dashboards], :menu[Reporting] | [Build dashboards of your validation results](/openaev/use/dashboards/) |

Steps 1 and 2 are done once and kept up to date: the assets you describe, the agents you install and the actions you import serve every scenario. Steps 3 to 6 repeat for each threat you test.

When the threat comes from OpenCTI, step 3 can start from its security coverage: OpenAEV generates the scenario from the techniques of the threat. See [Turn an OpenCTI security coverage into a scenario](/openaev/use/scenarios/from-opencti/).

## Two kinds of exercises on the same platform

The same steps serve two kinds of work, often in the same scenario:

- **A technical validation** acts on endpoints: an agent runs the attacker's commands, and collectors read the alerts of your security tools to score prevention and detection. It is the daily work of a validation engineer.
- **A human exercise** acts on people: emails, SMS, media pressure, challenges and manual actions reach the persons of your teams, and their reactions are scored. A tabletop or a crisis exercise is built this way, and so is a phishing campaign. See [How a phishing exercise works](/openaev/use/phishing/).

## Read the home dashboard

:menu[Home] opens on **Adversarial exposure overview**, a dashboard of every simulation and atomic test of the period. Choose the period in the list at the top right: **Last 3 months** by default, from **Last 24 hours** to **All time**. Your browser remembers the choice. **Refresh** reloads every widget.

| Widget | What it tells you |
| --- | --- |
| **Exposure command center** | The adversarial exposure score, from 0 to 100: the share of validations your controls did not stop, so a higher score is worse. Around it, the attacks stopped and breached, and your security platforms |
| **Prevention**, **Detection**, **Vulnerability** | The share of successful expectations of each type. **Human Response** appears beside them when expectations of people exist in the period |
| **Performance by security domain** | The same rates by kind of control: endpoint, network, email, cloud and the others. See [How security domains group your results](/openaev/concepts/security-domains/) |
| **Detection coverage** | The MITRE ATT&CK matrix, each technique colored by the detection expectations it passed and failed |
| **Posture radar** and **Missed by security platform** | For each security platform, what it caught, and what it did not detect or prevent |
| **Scenarios**, **Simulations**, **Injects**, **Assets**, **CVEs found**, **Vulnerable endpoints** | Counts of the period |
| **Findings by type**, **Latest findings**, **Total findings**, **Ports open** | What the injects discovered on your assets |
| **Missed injects by week**, **Most detected & prevented TTPs**, **Most undetected TTPs** | Where the gaps are, and whether they shrink |
| **Simulations by week**, **Latest simulations** | How often you test, and the last runs |

Click a bar, a slice or a number to open the list of what it counts: the expectations, findings or simulations behind it.

This dashboard is the **Platform default**. An administrator can open another one on :menu[Home] for the whole platform, and each user can choose their own in their profile: see [Build dashboards of your validation results](/openaev/use/dashboards/).

:::example[Follow Grand Meridian's defenses against Velvet Harbor]
After the intrusion of 9 January 2026, the validation engineer of Grand Meridian Hotels replays Velvet Harbor against the front desks of its French and Spanish hotels:

1. An OpenAEV agent runs on each front-desk workstation, and the asset group **Front desk workstations** gathers them.
2. The scenario **Velvet Harbor replay** chains a fake booking request for the European Hospitality Summit 2026 (T1566.001), a PowerShell loader (T1059.001), the search for guest records on the workstation (T1005) and the HTTPS beacon of the backdoor (T1071.001).
3. It runs every Wednesday at 06:00, and each run is a new simulation.
4. **Meridian EDR** blocks the PowerShell loader on four workstations of five; the **Meridian email gateway** lets the fake booking request through.
5. On :menu[Home], **Most undetected TTPs** lists Data from Local System (T1005) week after week: the gap to bring to the SOC.
:::

## Find what you look for in the lists

Every list of OpenAEV, such as the scenarios, the simulations or the assets, has the same search and filters. **Add filter** offers the attributes of the objects listed; each filter takes an operator, such as **contains**, **starts with** or **is empty**, and its values. Between two filters, the **AND** or **OR** chip switches the logic of the whole list in one click. **Clear filters** removes them all.

::screenshot{id=openaev-lists-filters alt="The Threat Arsenal list with Add filter open on the attributes its actions can be filtered on: action type, author, domains, platforms and status, beside the search field and the quick filters of the left panel."}

Your browser remembers the filters, the search and the page of each list, so you find them again when you come back. They also go into the address of the page, in its `query` parameter: copy the address to share the filtered list with a colleague who has access to the same tenant.

:::xtm-one[AI at each step]
When OpenAEV is registered with XTM One, AI agents help at several steps. The **Scenario assistant** builds a time-based scenario from a threat report, **Generate with AI** designs a chained scenario from an objective, and **Ask Ariane** in the top bar answers questions on your results and turns them into a report or a deck. See [XTM One in OpenAEV](/xtm-one/use/in-openaev/).
:::

## What's next

- [How a scenario becomes a simulation](/openaev/concepts/scenarios-simulations/): what you build and what each run keeps.
- [How an attack becomes injects and expectations](/openaev/concepts/injects-expectations/): what an inject plays and what it measures.
- [How injects reach assets and people](/openaev/use/targets/): prepare what you test.
- [Build dashboards of your validation results](/openaev/use/dashboards/): follow the trend your way.
