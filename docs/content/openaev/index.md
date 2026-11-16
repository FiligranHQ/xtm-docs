---
title: OpenAEV
description: Use OpenAEV to replay real attacks against your endpoints and your people, see what your
  defenses prevented, detected and answered, and follow that posture from run to run.
doc_type: landing
hide_table_of_contents: true
sidebar_label: Overview
products:
  - openaev
sidebar_position: 0
---

:::product-hero{eyebrow="Adversarial exposure validation" title="Replay the attacks that target you, and see what your defenses stop" primary-label="Start here" primary-to="#start-here" secondary-label="Install OpenAEV" secondary-to="/deploy/install/docker-compose/" shot=openaev-injects-list alt="The injects of the Velvet Harbor replay scenario on its timeline: a fake booking request emailed at the start, then a PowerShell loader at 15 minutes, a search for local guest records at 30 and an HTTPS beacon at 45, each with its domain, its platform and its tag."}
OpenAEV plays a real attack, the one a threat group ran last month or the one your threat intelligence expects, against your own endpoints and your own people. An agent runs the attacker's commands, emails and fake press articles reach your teams, and your security tools report what they saw. Every action is scored: prevented, detected, answered in time. Run the same scenario every week and the scores show whether your defenses get better or worse.
:::

## Start here

Pick your role: each tab gives your goal and the pages to read first, in order.

::::tabs
:::tab[Validation engineers]
You test whether your security tools stop and see the techniques of the threats you face.

1. [How you validate your defenses in OpenAEV](/openaev/concepts/validation/): the steps of a validation, from the first agent to the trend on the home dashboard.
2. [Install the OpenAEV agent on your endpoints](/openaev/use/agents/): the agent that runs the attacker's commands on the hosts you test.
3. [Build a scenario that replays an attack](/openaev/use/scenarios/build/): the injects, their timeline, their targets and their expectations.
4. [Run a simulation and read its results](/openaev/use/simulations/): launch the scenario and read what was prevented and detected.
5. [How collectors measure what your security tools saw](/openaev/use/collectors/): how an alert of your EDR or SIEM becomes a score.
:::
:::tab[Exercise planners]
You rehearse a crisis or a phishing campaign with the people who would live it.

1. [How a scenario becomes a simulation](/openaev/concepts/scenarios-simulations/): one scenario written once, played as many times as you need.
2. [Organize the people your injects target](/openaev/use/targets/people/): the players, their teams and their organizations.
3. [Publish fake press articles during an exercise](/openaev/use/components/media-pressure/): the media pressure a crisis cell faces.
4. [Give your players challenges to solve](/openaev/use/components/challenges/): questions scored as players answer them.
5. [How a phishing exercise works](/openaev/use/phishing/): the lure, the landing page and who opened, clicked and submitted.
6. [Collect lessons learned after an exercise](/openaev/use/components/lessons/): the questionnaire that closes the exercise.
:::
:::tab[SOC managers]
You want to know whether your detection holds against the threats of the moment, and show it.

1. [How expectations score an inject](/openaev/use/expectations/): prevention, detection and human response, and how each is decided.
2. [Turn an OpenCTI security coverage into a scenario](/openaev/use/scenarios/from-opencti/): test the threats your CTI team tracks.
3. [Build dashboards of your validation results](/openaev/use/dashboards/): the trend of your scores over runs.
4. [Share simulation results in a report](/openaev/use/simulations/reports/): a run summed up for the people who were not there.
:::
:::tab[Administrators]
You give people access, connect the security tools and set the platform up for your organization.

1. [How access works in OpenAEV](/openaev/concepts/access/): users, groups, roles and the grants on each scenario.
2. [Configure single sign-on](/openaev/administer/sso/): sign in with your identity provider.
3. [Choose between the OpenAEV agent and your EDR](/openaev/use/executors/): run the attacks through CrowdStrike, Defender, SentinelOne and others (Enterprise Edition, except Caldera).
4. [Give each organization its own tenant](/openaev/administer/multi-tenancy/): separate subsidiaries or customers on one platform.
:::
:::tab[Platform engineers]
You deploy OpenAEV beside the suite and keep it running.

1. [How OpenAEV works](/openaev/concepts/architecture/): the platform, the agents, the collectors and the injectors, and what each one needs.
2. [Install with Docker Compose](/deploy/install/docker-compose/): a running platform behind your reverse proxy.
3. [Connect the modules to each other](/deploy/configure/connect-modules/): OpenAEV with OpenCTI, OpenCRQ and XTM One.
4. [OpenAEV configuration](/reference/openaev/configuration/): every key, its variable and its default.
:::
::::

## Start from the starter pack

A new tenant is not empty. OpenAEV loads a starter pack into it, so you can launch a first simulation before you build anything:

| What | In the starter pack |
| --- | --- |
| Scenarios | **Tabletop Exercise - Akira Ransomware Crisis**, a crisis exercise by email; **[SALT TYPHOON] CVE & TTPs validation**, the techniques and vulnerabilities of an intrusion set, on endpoints; **EASM Scenario**, a discovery of internet-facing services |
| Assets | The agentless endpoint `honey.scanme.sh`, an internet host offered for scanner tests, which the EASM scenario scans; the dynamic asset group **All endpoints**, every endpoint of the tenant |
| Dashboards | **Technical home**, **Technical scenario** and **Technical simulation**, set as the tenant's defaults, and their **Tabletop Exercice** counterparts |

The first time you sign in from a browser, OpenAEV opens :menu[Getting Started], at the bottom of the left menu. Its scenarios come from the starter pack: **Try this scenario** opens one, and **Launch** in its header runs it. Every inject of the starter pack that targets asset groups targets **All endpoints**, and those that take assets only target `honey.scanme.sh`. Before you launch a technical scenario, aim its injects at your own test endpoints, and replace `honey.scanme.sh` with your own internet-facing assets: a simulation plays real attacker behavior, and should reach production endpoints only once your security team has approved it.

The platform engineer can turn the starter pack off with `openaev.starterpack.enabled`: see the [configuration reference](/reference/openaev/configuration/#openaev-starterpack).

## How a validation runs

You describe once what you test and what the attacker can do: the assets and agents, the people and their teams, the actions of the Threat Arsenal. A scenario chains those actions on a timeline, each with the result you expect from your defenses. Each launch is a simulation, scored action by action, and the scores of every run build the trend of your posture.

::diagram{id=openaev-use-workflow}

[How you validate your defenses in OpenAEV](/openaev/concepts/validation/) gives the page that covers each step, and [How an attack becomes injects and expectations](/openaev/concepts/injects-expectations/) explains the scoring.

## Rehearse a crisis with your people

::::split
:::column
The same scenarios reach people instead of endpoints: emails to the crisis cell, articles in a fake newspaper, documents to read, challenges to solve. Players answer from their own mailbox and from the player interface, and each reaction you expect is scored like a detection. A phishing campaign works the same way, with a lure, a landing page and the results of each person.

:::cards{cols=2}
  ::card[The press articles and media outlets of an exercise.]{icon=FileText title="Media pressure" href="/openaev/use/components/media-pressure/"}
  ::card[Lures, landing pages and who clicked.]{icon=Users title="Phishing" href="/openaev/use/phishing/"}
:::
:::
:::column
::screenshot{id=openaev-crisis-injects-list alt="The injects of the Velvet Harbor crisis night: the email that calls the crisis cell to the bridge at 40 minutes, the challenge Which account was used? at 50, and the Hotel Business Daily article at 1 hour 30."}
:::
::::

## Let the attack find its own path

::::split
:::column
A chained scenario does not follow a fixed timeline: each action reads what the previous one found, such as a user name, a host or an open port, and the next one aims at it. You set the targets it may reach and the limits it must keep, and the attack path map shows how far it got. With XTM One, an agent can choose the next action itself. Chained scenarios are part of the Enterprise Edition.

:::cards{cols=2}
  ::card[Actions that follow their own results.]{icon=Workflow title="Chained scenarios" href="/openaev/use/attack-chaining/"}
  ::card[Let AI run a penetration test.]{icon=Bot title="Autonomous attacks" href="/openaev/use/attack-chaining/autonomous/"}
:::
:::
:::column
::diagram{id=openaev-attack-chaining-cycle}
:::
::::

:::xtm-one[AI agents in OpenAEV]
**Ask Ariane** in the top bar answers questions on your scenarios and results. XTM One agents also generate the commands of an action from a technique, the detection rule that would have caught it, a phishing landing page from a brand, and drive autonomous attacks. See [XTM One in OpenAEV](/xtm-one/use/in-openaev/).
:::

## Find anything

**Search the platform**, in the top bar, looks for a word in the names and descriptions of scenarios, simulations, assets (security platforms included), asset groups, teams and organizations, and in the email addresses of persons. Press Enter to open **Search across OpenAEV**, its results grouped by **Categories**. It lists only what your role lets you access.

## Browse OpenAEV

::section-overview{id=openaev}
