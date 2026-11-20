---
title: Keep your detections tested every week
description: Run an attacker's actions through your EDR on an OpenAEV schedule, score each step from its alerts, get warned when the score drops, and have an XTM One agent write the missing rule.
doc_type: journey
journey: validate-detections
products: [openaev, xtm-one]
personas: [validation-engineer, soc-manager, ai-builder]
owner: docs-suite
last_reviewed: 2026-11-09
sidebar_position: 66
---

At the end of this journey, your endpoint detection and response (EDR) tool runs an attacker's actions on your endpoints every week, and every alert it raises scores a step of the attack. A dashboard shows the trend, a notification tells you the day the score drops, and the night shift of your security operations center (SOC) gets a detection rule for the technique that went unseen. The validation engineer drives it in OpenAEV, and an XTM One AI agent writes the rule. The steps below are the pages to follow, in order.

::diagram{id=journey-validate-detections}

## Who takes part

- **The validation engineer** drives the journey in OpenAEV: the executor, the actions, the atomic test, the scenario, the collector, the dashboard and the trigger.
- **The SOC manager** follows the dashboard of each weekly run, and receives the same warning when the score drops.
- **The AI agent builder** gives the night shift's agent the team's method for detection rules, as a skill, at the step [Write the missing rule with an agent](/xtm-one/build/skills/).
- **The SOC analysts** of the night shift ask the agent for the rule, review it and deploy it.

## Why it spans several products

OpenAEV measures what your defenses catch. It runs each action on your endpoints, through an OpenAEV agent or through your EDR, and its collectors read your security tools to mark each action detected or prevented, or not. Scheduled, the same scenario shows the trend run after run. See [How a scenario becomes a simulation](/openaev/concepts/scenarios-simulations/) and [How an attack becomes injects and expectations](/openaev/concepts/injects-expectations/).

A score tells you what was missed, not how to catch it. XTM One closes that gap: an agent bound to the OpenAEV integration reads the run as the analyst who asks, and a skill gives it the team's conventions for detection rules. See [What agents are made of](/xtm-one/concepts/agents/). Without it, the analyst writes the rule alone, from the inject's details, often at night.

## What you need

- **OpenAEV** with the Enterprise Edition: running actions through your EDR, deploying a collector from the catalog, and the **Remediation** tab of an action require it. With the Community Edition, install the OpenAEV agent on the endpoints, and run the collector yourself.
- Your EDR's API key, and the two scripts or packages the executor needs in its console. See [Run actions through CrowdStrike Falcon](/openaev/use/executors/crowdstrike/), or the page of your EDR.
- For a collector deployed from the catalog, a running XTM Composer: see [XTM Composer](/deploy/integrations/composer/).
- **XTM One** with the Enterprise Edition and AI switched on, and OpenAEV registered with it, so that its integration exists. See [Connect the modules to each other](/deploy/configure/connect-modules/#register-the-platforms-with-xtm-one).
- The capabilities, by their label in OpenAEV's role editor:
  - **Manage tenant settings** to deploy the executor and the collector, and to choose the tenant's default dashboards.
  - **Manage threat arsenal** to build the actions.
  - **Manage assessment** and **Launch assessment** to create, launch and schedule the atomic test and the scenario.
  - **Manage dashboards** to build the dashboard. Triggers are personal: they need no capability.
- In XTM One, any user creates a skill and attaches it to an agent they can configure; only an administrator makes it a **Company-managed skill**.

## How it unfolds

After the night of 9 January 2026, Grand Meridian Hotels wants to know each week whether Meridian EDR, its CrowdStrike Falcon deployment, still stops what Velvet Harbor did at the front desks.

**Run through the EDR.** The validation engineer deploys the CrowdStrike Falcon executor. Every 20 minutes, it imports the endpoints of the Falcon host groups they chose, and each group becomes an asset group: `Front desk workstations`. The actions run with SYSTEM rights, as an attacker in full control of the machine, with nothing installed on the endpoints ([Run attacks through your EDR](/openaev/use/executors/)).

**Build the attacker's actions.** In the Threat Arsenal, the engineer builds what Velvet Harbor ran: the PowerShell loader of the booking attachment (T1059.001), the collection of local guest records (T1005) and the HTTPS beacon (T1071.001), each expecting **Prevention** and **Detection** ([Build the attacker's actions](/openaev/use/threat-arsenal/)).

**Check one technique first.** The atomic test `Velvet Harbor loader on the front desks` runs the loader on `Front desk workstations` every day at 03:00. It reads **Prevention** 100 and **Detection** 100. Each run replaces the results of the one before, so the history of the whole attack belongs in a scenario ([Check one technique first](/openaev/use/atomic-testing/)).

**Schedule the scenario.** The time-based scenario `Velvet Harbor replay` plays the techniques one after the other on the same group, every Wednesday at 06:00 from 14 January 2026. Each run is a new simulation, with its own results ([Schedule the scenario](/openaev/use/scenarios/build/)).

**Collect the alerts.** The CrowdStrike collector reads Falcon's detections. An alert whose parent process is the implant of an inject marks that inject **Detected**, or **Prevented** when Falcon killed it. What no alert answers within 6 hours expires as **Not Detected** or **Not Prevented** ([Collect the alerts](/openaev/use/collectors/)).

**Follow the trend and get warned.** The dashboard `Velvet Harbor replay`, with its parameter `Weekly run`, becomes the tenant's **Default simulation dashboard**: each new simulation opens on it. On 21 January, it shows that Data from Local System (T1005) is still not detected ([Follow the trend](/openaev/use/dashboards/)). The live trigger `Velvet Harbor replay score drop` watches for **Score degradation**. One hour after a run ends, OpenAEV compares it with the previous one and, when prevention or detection fell, sends an email through **Default mailer** ([Get warned of a drop](/openaev/configure/notifications/)).

**Write the missing rule.** Sophie Laurent, the AI engineer, attached the skill `Write a Sigma rule` to the agent **Night shift triage**, shared with the group **SOC**. The agent reads OpenCTI and OpenAEV. The night shift asks it for a rule detecting the collection of guest records that the replay of 21 January missed. The trace shows the skill loaded, and the rule follows the team's conventions: its title starts with `GM -`, and it carries the ATT&CK tag of the technique ([Write the missing rule with an agent](/xtm-one/build/skills/)). The SOC reviews and deploys it; the next Wednesday's run shows whether it fires.

## The steps

Each step opens its page; the first one starts the journey, and every page of it then shows where you are.

::journey-steps

## Where to go next

- [Check your defenses against the threats you track](/workflows/threat-informed-validation/): let OpenCTI's threat intelligence build the scenario, and read its scores beside the threat.
- [Test whether your staff fall for the lure](/workflows/test-your-people/): measure the human side of the same intrusion.
- [Run a simulation and read its results](/openaev/use/simulations/): what each gauge and result means.
- [XTM One in OpenAEV](/xtm-one/use/in-openaev/): **Generate with AI** on the **Remediation** tab of an action writes a rule from OpenAEV itself.
