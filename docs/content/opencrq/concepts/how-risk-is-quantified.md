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

OpenCRQ expresses each cyber risk as an **Annualized Loss Expectancy** (ALE): the loss, in US dollars, that a risk scenario can be expected to cause over a year, with the probability of exceeding each amount. It answers three questions: how often attacks happen, how often one lands, and what it costs when it does.

## The building blocks

| Block | What it is |
| --- | --- |
| Risk scope | The part of the organization you assess. Its financial profile, such as revenue, scales the losses that depend on it |
| Primary assets | What the business would lose: the services, data and processes a scenario threatens |
| Assets | The estate that supports them: devices, identities, cloud resources, declared by hand or imported by a connector such as Microsoft Defender or OpenAEV |
| Risk scenario | The outcome an attacker seeks and the primary assets it would hit. OpenCRQ weighs every class of attacker in your threats that seeks that outcome |
| Threat Event Frequency | How many attack campaigns a year the scenario faces, as a range: minimum, most likely, maximum. From the catalog, or set by hand |
| Loss drivers | What an impact costs, by loss category, each as a range |
| Controls | Your defenses, each with a declared strength and coverage. OpenAEV results show how they held when tested, and lower a declaration the tests do not bear out |

## How a scenario is priced

::diagram{id=opencrq-scenario-pricing}

1. **Frequency.** The Threat Event Frequency says how many campaigns the scenario faces in a year. It counts attack campaigns, not the exposure of your assets: exposure comes next.
2. **Initial access.** Most campaigns first have to get in, over the whole risk scope. They only use the techniques that apply to your estate: exploiting a public service needs an asset exposed to the internet, and phishing needs enabled user accounts, the people it is sent to. A drive-by compromise needs a workstation, a laptop, a mobile device or a user account. An estate with no exposed service resists those campaigns instead of seeing them move to another technique. Two outcomes also have attacks that need no way in: a denial of service is launched straight at an internet-facing asset, and a small share of financial fraud starts from a spoofed domain or an insider.
3. **The paths after it.** An attack that got in follows the paths toward the primary assets, and meets your controls on the way. Each control prevents or detects a technique as well as its strength and coverage say, less where OpenAEV's tests of that technique held less often.
4. **Outcome and cost.** Each attack ends one of four ways: **blocked**, **evicted** before any impact, **impact detected**, or **impact undetected**. The loss drivers give the loss magnitude, the full cost of one loss event. An eviction costs a response, priced at 15 % of it; a detected impact at 75 %; an undetected impact in full; a blocked attack nothing.

OpenCRQ evaluates this over many worlds. Each world draws the uncertain inputs from the ranges you gave and prices the expected loss of a year. Each world also draws ten simulated years, for the probability of exceeding each annual loss. The result is a distribution, not a single figure: the mean ALE, its p10 to p90 band, and the probability of exceeding each annual loss.

The band is the range the expected annual loss takes in eight worlds out of ten as the uncertain inputs vary. A single year's loss can fall far outside it. The mean also counts the few worlds with a heavy loss, so when attacks rarely land, it can sit above the p90. The means of several scenarios add up; their bands do not.

:::xtm-one[Ask XTM One about a scenario]
OpenCRQ runs with XTM One in both editions: people sign in through it, and XTM One AI agents read your risk scopes and scenarios, for example from an AI client through XTM One's MCP server. An XTM One administrator decides which changes wait for a person's approval. In the Enterprise Edition, the assistant **Ask Ariane** also explains what drives a loss from inside OpenCRQ. That edition comes from a Filigran license installed in XTM One that covers this OpenCRQ as a sub-license. It also allows more than one tenant.
:::

## Where the rest of the XTM platform comes in

- **OpenCTI** keeps the threats current: an OpenCTI Live Stream brings the threat actors, the sectors and countries they target, their techniques and the CVEs they exploit.
- **OpenAEV** shows how your controls held: its detection and prevention results, per ATT&CK technique and security platform, become findings on your assets, and audit each control's declaration technique by technique. A test can lower what you declared, never raise it.
- **CVE sources** (NIST, FIRST EPSS, CISA KEV) say which vulnerabilities are likely or known to be exploited.

## From risk to decisions

A **treatment plan** is the set of controls you would fund, each aimed at a posture. OpenCRQ prices the scope's scenarios again against that posture, so you read the loss a plan removes against what it costs. A **budget review** sets a fiscal year's security spend against the annual loss it avoided, line by line.

## What's next

- [How OpenCRQ's objects connect, from a risk scope to its price](/opencrq/concepts/data-model/): what each building block belongs to, and which prices a change reaches.
- [OpenCRQ overview](/opencrq/), and how it connects to the XTM platform.
- [Journeys across the platform](/workflows/), including a full CTEM cycle.
