---
title: Set how OpenCRQ weighs threats, controls and assets
description: Choose which threat intelligence counts, how fast it ages and how undated threats rank, then tune the simulation, exposure, stacked controls, phishing and test results for a tenant.
sidebar_position: 2
doc_type: how-to
products:
  - opencrq
personas: [risk-manager, admin]
owner: docs-opencrq
last_reviewed: 2026-11-04
---

Every price OpenCRQ gives rests on a few assumptions: which threat intelligence is reliable enough to read, how fast an actor's weight fades when it goes quiet, how a threat with no dated activity ranks, how many simulated years a price draws, how much stacked controls add, and what an asset that declares nothing counts as. **Settings > Engine** holds them for the whole tenant. This page explains each assumption, its default and what moves when you change it. Read [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/) first if the attack run is new to you.

## Before you start

- You are a member of the tenant. The two pages need no capability: anyone in the tenant can read and change them, so agree in your team who owns them.
- The settings apply to every risk scope of the tenant. A change reprices every scenario it touches, so note the current values before you change one: the pages keep no history.

## Choose which threats count

**Threat Relevance** decides which intelligence about threat actors OpenCRQ reads and how much each actor weighs. OpenCRQ's internal model gives each threat a probability of leading to each attack outcome, from what its intelligence says of it: its description, motivations and goals, malware, techniques and reports. For each scenario, OpenCRQ admits the actors whose probability for the scenario's attack outcome is above 50 %, weighs each one more when it targets the risk scope's country (×2.5), its sector (×4) or both (×10), then weighs the ATT&CK techniques of the scenario by the actors that use them. The confidence gate and the half-lives act on that mixture of attackers.

**Confidence gate.** Each relationship that OpenCTI streams, such as "Ember Tide targets the hospitality sector" or "Velvet Harbor uses Phishing", carries the confidence its producer gave it. OpenCRQ reads a relationship only when its confidence reaches the gate. A relationship below it is kept, not read: lowering the gate brings it back without importing anything again. The attack outcome probabilities of OpenCRQ's model do not depend on the gate.

| Level | Reads |
| --- | --- |
| **confirmed (100)** | Only relationships confirmed by their producer |
| **confirmed by other sources (80)** | Relationships at 80 or more |
| **probably true (60)** | Relationships at 60 or more. The default |
| **possibly true (40)** | Relationships at 40 or more |
| **doubtful (20)** | Relationships at 20 or more |
| **improbable (0)** | Every relationship |

A gate too high leaves scenarios with no admitted actor, and a scenario no admitted actor pursues stays unpriced: its **Relevant Threats** panel then says that no threat passes the gate and links back to this setting. A gate too low lets weak claims shape your prices.

**Activity half-lives.** An actor weighs less as its last known activity ages: after one half-life, its weight is halved. OpenCRQ dates that activity from two kinds of anchor:

| Field | Anchored on | Default |
| --- | --- | --- |
| **Observed anchor half-life (days)** | The start or stop date reported for an activity | `180` |
| **Authored anchor half-life (days)** | Only the date the intelligence relationship was written | `365` |

An authored date says less about when the actor was active, so it decays more slowly. Each half-life is a whole number of days from 7 to 3,650, and the observed one cannot exceed the authored one.

**Threats Without a Dated Activity.** A scenario's **Relevant Threats** ranks the admitted actors by their relevance and their activity. An actor whose intelligence carries no observed activity date still needs an activity to rank on: **Assumed activity age (days)** scores it as if it had a single activity that many days old. The default is `365`, a whole number of days from 7 to 3,650. A lower value ranks undated threats higher, next to the recently active ones; a higher value pushes them down. The setting moves the ranking and the threat score it shows, not the attacker mixture a scenario is priced with.

:::steps
1. Open :menu[Settings > Engine]. **Threat Relevance** opens first; **CRQ Indicators** is the other entry of the menu on the right.

   ::screenshot{id=opencrq-engine-threat-relevance alt="The Threat Relevance settings of the Grand Meridian Hotels tenant: the Confidence gate set to probably true (60), the observed and authored anchor half-lives at 180 and 365 days, and the start of the Threats Without a Dated Activity section."}

2. Choose the **Confidence gate**.
3. Set the **Observed anchor half-life (days)** and the **Authored anchor half-life (days)**.
4. Set the **Assumed activity age (days)** of **Threats Without a Dated Activity**. The field is required.
5. Click **Save changes**. OpenCRQ confirms with **Threat relevance settings updated successfully.**
:::

## Tune the simulation and the scores

**CRQ Indicators** holds the parameters of the simulation and of the finding scores. Each section of the page is one assumption:

| Section and field | What it changes | Default and values |
| --- | --- | --- |
| **Monte Carlo** > **Iterations** | How many simulated years each price draws. More years make the distribution steadier and each run slower | `1000`; a whole number up to 2,000, above which OpenCRQ refuses the value |
| **Exposure Weights** > **Internet facing**, **Internal** | The multipliers the finding score applies by exposure, to how reachable a flaw is and how much an asset matters for each technique | `1` and `0.4`; any number above 0 |
| **Unknown exposure** > **Counts as** | The exposure of an asset that declares none: on its TTP tab, in the finding score, and in the simulation at initial access and along the paths | **Internal**, or **Internet-facing** |
| **Control aggregation** > **Diminishing returns factor (γ)** | How several controls on the same technique combine. The strongest counts fully, and each additional one is discounted by γ. At `1` every control counts fully; at `0` only the strongest counts | `0.7`, from 0 to 1 |
| **Phishing simulations** > **People caught (%)**, **Campaign measured on** | The share of your people your last phishing simulation caught, which replaces the default rate for phishing at initial access | Empty: the engine reads 7 %, from published click rates. Both fields are set together or cleared together with **Clear the declaration**; the date cannot be after today (UTC) |
| **Validity of a test result** > **Result stays valid for (days)** | When a control's test result stops counting as a validation of the control, and when a declared phishing simulation gives way to the default rate | Empty: a result counts however old it is |
| **Unknown criticality** > **Counts as** | The criticality of an asset that declares none, in the finding score and in the weight of the assets an attacker moves through | **Low**; **Medium**, **High** or **Very High** |

Two choices deserve a second look on a young estate:

- **Unknown exposure** and **Unknown criticality** decide how every undeclared asset counts. Leaving them at **Internal** and **Low** prices a half-documented estate as if it were mostly safe; counting unknown assets as **Internet-facing** is the cautious choice until your inventory says otherwise.
- **People caught (%)** is read as measured under the awareness training you declare today. When the training changes, run a new campaign and declare it again.

:::steps
1. Open :menu[Settings > Engine], then **CRQ Indicators** in the menu on the right.

   ::screenshot{id=opencrq-engine-crq-indicators alt="The CRQ Indicators settings at their defaults: 1000 iterations, exposure weights of 1 for internet-facing and 0.4 for internal assets, unknown exposure counted as Internal, and a diminishing returns factor of 0.7."}

2. Change the fields of the assumptions you revisit. To declare a phishing campaign, fill in both **People caught (%)** and **Campaign measured on**.
3. Click **Save changes**. OpenCRQ confirms with **CRQ indicator settings updated successfully.**
:::

:::example[Declare Grand Meridian's phishing campaign and age old test results]
After the multi-factor rollout of March 2026, Grand Meridian's security team ran a phishing simulation on the front desks of its French hotels. The risk manager declares it and decides that a control test older than a year no longer proves anything:

- **People caught (%)**: `4`.
- **Campaign measured on**: `2026-03-16`.
- **Result stays valid for (days)**: `365`.

Every other field keeps its default. The engine now reads 4 % instead of 7 % wherever an attack starts with phishing, until 16 March 2027, when the campaign expires and the default applies again. A control whose last test is more than a year old counts on what its declaration claims, not on that test.
:::

## What happens after you save

OpenCRQ keeps its figures current without a manual recompute: the background jobs that score findings, refresh the threat lists and price scenarios run again when the settings they read change. A new confidence gate also refreshes what the threats list reads of each threat, and the attackers weighed in each scenario. Follow the runs in :menu[Settings > Jobs]; [Background processes](/reference/opencrq/processes/) lists what each job reads.

To see the effect on one scenario at once, open it and run its simulation: [Build and price a risk scenario](/opencrq/use/quantify/risk-scenarios/).

:::xtm-one[Change the settings with XTM One]
XTM One agents read and update both pages, the threat relevance settings and the CRQ indicator settings, as the person who asks them. Ask Ariane, the assistant, needs the Enterprise Edition; an XTM One administrator can make the update tools wait for your approval before they run. See [XTM One in OpenCRQ](/xtm-one/use/in-opencrq/).
:::

## What's next

- [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/), for where each assumption enters the attack run.
- [Explore the ATT&CK, CVE and CWE catalogues](/opencrq/configure/attack-and-vulnerability-catalogues/), to see the techniques the threats are weighed on.
- [Read the risk catalogues and manage asset tags](/opencrq/configure/risk-taxonomies/).
- [Background processes](/reference/opencrq/processes/), for the jobs that apply a change.
