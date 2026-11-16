---
title: How OpenCRQ quantifies risk
description: How OpenCRQ prices a risk scenario as an annualized loss, from attack
  frequency, the attack run against your controls, and loss drivers.
doc_type: explanation
products:
  - opencrq
personas: [risk-manager]
owner: docs-opencrq
last_reviewed: 2026-10-31
---

OpenCRQ expresses each cyber risk as an **Annualized Loss Expectancy** (ALE): the loss, in your currency, that a risk scenario can be expected to cause over a year, with the probability of exceeding each amount. It follows the Factor Analysis of Information Risk (FAIR) model: how often attacks happen, how often one lands, and what it costs when it does.

## The building blocks

| Block | What it is |
| --- | --- |
| Risk scope | The part of the organization you assess. Its financial profile, such as revenue, scales the losses that depend on it |
| Primary assets | What the business would lose: the services, data and processes a scenario threatens |
| Assets | The estate that supports them: devices, identities, cloud resources, declared or imported from Microsoft Defender |
| Risk scenario | A class of attacker, the outcome they seek, and the primary assets it would hit |
| Threat Event Frequency | How many attack campaigns a year the scenario faces, as a range: minimum, most likely, maximum. From the catalog, or set by hand |
| Loss drivers | What an impact costs, by loss category, each as a range |
| Controls | Your defenses, each with a declared strength and coverage. OpenAEV results show how they held when tested, and lower a declaration the tests do not bear out |

## How a scenario is priced

::diagram{id=opencrq-scenario-pricing}

1. **Frequency.** The Threat Event Frequency says how many campaigns the scenario faces in a year. It counts attack campaigns, not the exposure of your assets: exposure comes next.
2. **Initial access.** Each campaign first has to get in, over the whole risk scope. It only uses the techniques that apply to your estate: exploiting a public service needs an asset exposed to the internet, phishing needs workstations or user accounts. An estate with no exposed service resists those campaigns instead of seeing them move to another technique.
3. **The paths after it.** An attack that got in follows the paths toward the primary assets, and meets your controls on the way. Each control prevents or detects a technique as well as its strength and coverage say, less where OpenAEV's tests of that technique held less often.
4. **Outcome and cost.** Each attack ends one of four ways: **blocked**, **evicted** before any impact, **impact detected**, or **impact undetected**. An eviction costs a response, a detected impact costs less than an undetected one, and a blocked attack costs nothing. The loss drivers give the amounts.

OpenCRQ evaluates this over many simulated years, each drawn from the ranges you gave. The result is a distribution, not a single figure: the mean ALE, and the probability of exceeding each annual loss.

:::xtm-one[Ask XTM One about a scenario]
With XTM One connected, agents read your risk scopes and scenarios from the chat panel and explain what drives a loss. Any change they propose waits for a person to approve it. XTM One also unlocks the Enterprise Edition of OpenCRQ.
:::

## Where the rest of the suite comes in

- **OpenCTI** keeps the threats current: an OpenCTI Live Stream brings the threat actors, the sectors and countries they target, their techniques and the CVEs they exploit.
- **OpenAEV** shows how your controls held: its detection and prevention results, per ATT&CK technique and security platform, become findings on your assets, and audit each control's declaration technique by technique. A test can lower what you declared, never raise it.
- **CVE sources** (NIST, FIRST EPSS, CISA KEV) say which vulnerabilities are likely or known to be exploited.

## From risk to decisions

A **treatment plan** is the set of controls you would fund, each aimed at a posture. OpenCRQ prices the scope's scenarios again against that posture, so you read the loss a plan removes against what it costs. A **budget review** sets a fiscal year's security spend against the annual loss it avoided, line by line.

## What's next

- [OpenCRQ overview](/opencrq/), and how it connects to the suite.
- [Journeys across the suite](/workflows/), including a full CTEM cycle.
