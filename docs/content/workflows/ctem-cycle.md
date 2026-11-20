---
title: Run a CTEM cycle across the platform
description: Scope what the business could lose in OpenCRQ, bring in the threats from OpenCTI, price each risk, test the controls with OpenAEV, fund what removes the most loss, and follow it in XTM One.
doc_type: journey
journey: ctem-cycle
products:
  - opencrq
  - opencti
  - openaev
  - xtm-one
personas: [risk-manager, validation-engineer, executive]
owner: docs-platform
last_reviewed: 2026-11-09
---

At the end of this journey, the board sees where the organization stands in money each quarter, and what the next dollar of security budget should buy. Continuous threat exposure management (CTEM) is a cycle of five stages: scope what matters, discover what threatens it, prioritize, validate, and mobilize the people who fix it. OpenCRQ scopes the business and prices each risk, OpenCTI brings in the threats, OpenAEV tests the controls against real attacks, and XTM One follows the five stages on one screen. The risk manager drives it, with the validation engineer who runs the tests. The steps below are the pages to follow, in order.

::diagram{id=journey-ctem-cycle}

## Who takes part

- **Risk managers** run the cycle in OpenCRQ: the risk scope, its assets, the risk scenarios, the controls and the treatment plans (steps 1, 2, 4 to 7, 9 and 10).
- An **administrator** with **Manage Security** in OpenCRQ connects OpenCTI and OpenAEV to the tenant once (steps 3 and 9).
- **Security validation engineers** replay the techniques of the threats that weigh most, on the endpoints that carry the business (step 8).
- **CISOs and executives** read the result: the ranking in money, the plan to fund, and the exposure posture of XTM One (step 11).
- The CTI team keeps the intelligence current in OpenCTI, and the SOC closes the gaps the tests reveal.

## Why it spans several products

Each stage of the cycle lives where its data is:

- **Scope and prioritize** in OpenCRQ, which knows what the business would lose and turns each risk into an annualized loss expectancy (ALE): the loss to expect in a year, from how often attacks come, how often one lands, and what it costs.
- **Discover** from OpenCTI, which brings the threat actors and campaigns that target your sector and countries, with their techniques and CVEs; and from Microsoft Defender and OpenAEV, which bring the estate and its flaws.
- **Validate** with OpenAEV, whose simulations replace a declared control strength by a measured one.
- **Mobilize** in OpenCRQ again, where treatment plans compare what each option costs with the loss it removes; XTM One scores the five stages and tells each person what to fix first.

A risk register alone prices assumptions; a list of findings alone does not say which to fix first. In the cycle, threats and test results move the price, and the price orders the work. The model behind it is in [Risk scopes, primary assets and supporting assets](/opencrq/concepts/risk-scopes-and-assets/) and [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/).

## What you need

- **OpenCRQ**, registered with XTM One, with a tenant for your organization. See [Install OpenCRQ](/deploy/install/opencrq/) and [Connect the modules to each other](/deploy/configure/connect-modules/).
- **OpenCTI**, with the intelligence on the threats to your sector, and a user OpenCRQ reads it as.
- **OpenAEV**, with OpenAEV agents on the endpoints to test and a collector that reads your EDR. [Check your defenses against the threats you track](/workflows/threat-informed-validation/) prepares it.
- **XTM One**, with OpenCTI and OpenAEV registered, for the **CTEM Command Center**.
- Capabilities: in OpenCRQ, **Manage Security** to connect OpenCTI and OpenAEV; any member of the tenant creates risk scopes, assets, scenarios, control postures and treatment plans.

## How it unfolds

The story: on the night of 9 January 2026, Velvet Harbor copied guest records from Grand Meridian's booking platform through a shared front-desk account. Hélène Marchetti, the CISO, wants the board to see where the group stands each quarter, in money. The risk manager runs the cycle with the CTI team, the validation engineer and the SOC; the figures are those of 12 April 2026.

### Scope what the business could lose

A CTEM cycle starts from the business, not from the vulnerabilities. The risk manager creates the risk scope Grand Meridian France, sector Hospitality, country France, with its two primary assets: the **Online booking platform** and the **Guest records** ([step 1](/opencrq/concepts/risk-scopes-and-assets/)). The supporting assets an attacker can reach, such as the booking web application, internet-facing and **Very High** in criticality, are linked to the primary assets they carry ([step 2](/opencrq/use/attack-surface/assets/)). The CTI team can state the same scope as a standing question in OpenCTI, a priority intelligence requirement on the hospitality sector in Europe: see [Follow your priority threats with PIRs](/opencti/use/analyze/pir/).

### Discover the threats and the open doors

A live stream of OpenCTI brings the intrusion sets, threat actors and campaigns to the tenant ([step 3](/opencrq/use/sources/opencti/)). In the threat center, Velvet Harbor and Ember Tide arrive with the sectors and countries they target ([step 4](/opencrq/use/threats/threat-center/)). When OpenCTI reports Operation Low Season, a campaign against hotel chains attributed to Ember Tide, on 10 April 2026, it reaches OpenCRQ the same day. The findings come from Microsoft Defender and OpenAEV: each vulnerability is scored from 0 to 100, and the open ones are the doors an attack run uses ([step 5](/opencrq/use/attack-surface/findings/)).

### Prioritize in money

The risk manager creates the scenarios the board worries about, and OpenCRQ prices them: Ransomware on the Booking Platform reads $254K a year, Data Breach on Guest Records $33K ([step 6](/opencrq/use/quantify/risk-scenarios/)). Grand Meridian France carries $287K of the group's $512K on the **CISO** dashboard of [Track cyber risk in money with dashboards](/opencrq/use/decide/dashboards/). Ransomware comes first: the order of the work follows the money, not the count of findings.

### Validate the controls against real attacks

A price rests on how well the controls hold. Declared, a control is an assumption ([step 7](/opencrq/use/defenses/controls/)); tested, it is a measure. The validation engineer replays the techniques of the threats that weigh most on the front desks ([step 8](/openaev/use/simulations/)), and OpenCRQ reads each finished simulation: a technique the EDR did not prevent becomes a finding, and the tests lower what a control was declared to do where it failed ([step 9](/opencrq/use/sources/openaev/)). When the weekly simulation shows Web Protocols (T1071.001) detected but not prevented, the scenarios are priced again; when the SOC closes the gap, the next simulation prices them again.

### Mobilize on the plan that removes the most loss

The risk manager prices one treatment plan per option, with the controls it moves and what it costs ([step 10](/opencrq/use/decide/treatment-plans/)). On 12 April 2026, three plans compete for Grand Meridian France:

| Plan | First-year cost | Annual loss reduction | Return |
| --- | --- | --- | --- |
| Zero-trust remote access for the hotels | $35K | $63K | +80.4% |
| Network intrusion prevention at the head office | $195K | $52K | −73.3% |
| MFA for the front-desk accounts | $115K | $0, already done | −100% |

The remote access plan is the one to fund. MFA for the front desks already brought Data Breach on Guest Records down from $97K to $33K a year by 2 March, which is why its own reduction now reads $0. Once a plan is delivered, its controls are declared at their new level: the scenarios are priced again, and the next cycle starts from there. At the end of the fiscal year, [Review a year's security budget against the loss it avoided](/opencrq/use/decide/budget-reviews/) sets the spend against the loss it avoided.

On the home dashboard of XTM One, the **CTEM Command Center** scores **Scoping**, **Discovery**, **Prioritization**, **Validation** and **Remediation** and averages them into one exposure posture, and its **Exposure briefing** tells each person what to fix first ([step 11](/xtm-one/use/dashboard/)).

## The steps

Each step opens its page; the first one starts the journey, and every page of it then shows where you are.

::journey-steps

## Where to go next

- [Check your defenses against the threats you track](/workflows/threat-informed-validation/), the validation stage on its own, from an OpenCTI report, every week.
- [XTM One in OpenCRQ](/xtm-one/use/in-opencrq/), to ask **Ask Ariane** which risk scenarios cost more than $100K a year.
- [How you validate your defenses in OpenAEV](/openaev/concepts/validation/), for what a simulation measures.
