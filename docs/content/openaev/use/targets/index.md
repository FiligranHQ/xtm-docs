---
title: How injects reach assets and people
description: The targets of an inject, assets, asset groups, persons and teams, how a scenario makes them available, and why each target gets its own expectations.
doc_type: explanation
products:
  - openaev
personas: [validation-engineer, exercise-planner]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/targets/
---

Every inject acts on targets. A technical inject, such as a command or a vulnerability scan, acts on assets: the hosts, applications and other systems you test. A human inject, such as an email, an SMS or a media article, reaches persons, always through their teams. You prepare the targets once, in their own menus, and every scenario, simulation and atomic test picks from them. This page explains the four kinds of targets, how an inject gets them, and what OpenAEV scores for each.

## The four kinds of targets

::diagram{id=openaev-targets-model}

| Target | What it is | Menu | Read |
| --- | --- | --- | --- |
| Asset | A system an inject acts on: a host where an OpenAEV agent runs, or an asset you describe yourself, such as a web application, a cloud resource, a network device or an AI target | :menu[Assets] | [Add the assets your simulations target](/openaev/use/targets/assets/) |
| Asset group | A set of assets, listed by hand or selected by a rule, such as every Windows host | :menu[Asset groups] | [Add the assets your simulations target](/openaev/use/targets/assets/) |
| Person | Someone who receives emails, SMS, media articles and challenges, and whose reaction is scored | :menu[Persons] | [Organize the people your injects target](/openaev/use/targets/people/) |
| Team | The persons an inject reaches together: the front desk of a hotel, the SOC, the crisis cell | :menu[Teams] | [Organize the people your injects target](/openaev/use/targets/people/) |

Reading these menus takes the **Access assets** capability for assets and asset groups, and **Access teams & persons** for persons and teams.

::screenshot{id=openaev-assets-list alt="The Assets list of Grand Meridian Hotels, by name: the booking database server booking-db-01, then the front-desk workstations fd-barcelona-03, fd-madrid-02, fd-nice-04 and fd-paris-01, each a host."}

## How an inject gets its targets

The type of an inject decides what it can target. An action of the Threat Arsenal runs on assets and asset groups; an email, an SMS, a challenge or a manual action reaches teams. A type that can do both shows both fields. You choose the targets in the inject's form:

- **Assets and asset groups** are picked directly on the inject, among all your assets. The endpoint picker opens filtered on the platforms and the architecture of the action, so it lists the endpoints that can run it; remove a filter to see the others. A scenario tag can also add asset groups to its injects for you: see [Set default asset groups by scenario tag](/openaev/configure/asset-rules/).
- **Teams** must first belong to the scenario or the simulation: add them in the **Configuration** of the scenario, **Teams** tab. The inject then targets some of them under **Targeted teams**, or all of them with **All teams**. When a team joins a scenario, all its persons are enabled; disable the ones who must not receive its injects. An atomic test has no team list of its own: its **Targeted teams** offer every team of :menu[Teams].

An asset group is read when the inject runs: an asset that matches its rule at that moment is targeted, even if it did not exist when you built the scenario. Teams and their enabled persons, on the contrary, are copied into each simulation when it starts. See [How a scenario becomes a simulation](/openaev/concepts/scenarios-simulations/).

## One target, one set of expectations

OpenAEV scores each target on its own. An inject on an asset group creates expectations for the group, for each of its assets and for each agent on them; an inject on a team creates expectations for the team and for each of its enabled persons. That is why a detection can succeed on four workstations and fail on the fifth, and why the **Validation mode** of an expectation says whether one member of a group is enough. See [How an attack becomes injects and expectations](/openaev/concepts/injects-expectations/).

## Security platforms are not targets

:menu[Security platforms] lists your security tools: EDR, XDR, SIEM, SOAR, NDR, email security, vulnerability scanners and others. They are what OpenAEV measures, not what it attacks: the results of detection and prevention name the security platform that answered, and an expectation can require a given kind of platform. Each collector is linked to the security platform it reads; you can also add a platform by hand with **Create**. Reading them takes the **Access security platforms** capability.

:::example[The targets of Velvet Harbor at Grand Meridian Hotels]
To replay Velvet Harbor, the validation engineer of Grand Meridian Hotels prepares:

| Target | Value |
| --- | --- |
| Assets | The front-desk workstations of the French and Spanish hotels, such as `fd-paris-01` and `fd-madrid-02`, each with an OpenAEV agent, and `booking-db-01`, the booking database server |
| Asset group | **Front desk workstations**, by a rule on the hostnames that start with `fd-` |
| Persons | Paul Garnier, the night auditor, and the other front-desk employees |
| Teams | **Front desk France** and **Front desk Spain**, with the front-desk employees of each country |

The fake booking request for the European Hospitality Summit 2026 targets both teams; the PowerShell loader and the search for local guest records target the asset group **Front desk workstations**.
:::

## What's next

- [Add the assets your simulations target](/openaev/use/targets/assets/): register endpoints, describe assets and group them.
- [Organize the people your injects target](/openaev/use/targets/people/): create persons, teams and organizations.
- [Install the OpenAEV agent on your endpoints](/openaev/use/agents/): let your hosts run injects.
