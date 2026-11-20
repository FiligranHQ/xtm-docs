---
title: Follow the threats that target your scopes
description: Read the threat actors, intrusion sets and campaigns OpenCTI streams to OpenCRQ, see which ones weigh on a risk scenario and why, and follow their activity as it moves the price.
sidebar_position: 1
doc_type: how-to
products: [opencrq, opencti]
personas: [risk-manager, cti-analyst]
owner: docs-opencrq
last_reviewed: 2026-11-04
---

OpenCRQ prices a risk scenario against the attackers that pursue it. It does not ask you to name them: it reads the threats your OpenCTI platform streams, keeps the ones its model judges likely to lead to the scenario's outcome, weighs more the ones that target your sector or your country, and weighs each by how recently it was seen acting. The **Threats** screen, the threat center, lists them. This page reads it, explains how a threat gets into a scenario and what it changes there, and shows how a threat's return moves the price. For the model behind the price, read [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/).

## Before you start

- You are a member of the tenant: every member reads the threats.
- An administrator with **Manage Security** has connected OpenCTI with a Live Stream. See [Connect OpenCTI with a Live Stream](/opencrq/use/sources/opencti/). Until then, the threats of **OpenCRQ Default Threats**, groups and campaigns derived from MITRE ATT&CK that every tenant starts with, stand in.
- The risk scopes have a **Sector** and a **Country**: they are what a threat is matched on. See [Risk scopes, primary assets and supporting assets](/opencrq/concepts/risk-scopes-and-assets/).

## What OpenCTI brings

OpenCRQ keeps three kinds of STIX objects as threats: **Threat actor**, **Intrusion set** and **Campaign**. For each one it keeps what pricing needs:

- the countries and sectors it targets, each claim with its confidence;
- the ATT&CK techniques it uses, and the CVEs it targets;
- the threats it is attributed to, such as a campaign attributed to an intrusion set;
- its first and last seen dates, motivations, goals, resource level, sophistication and labels.

You cannot create or edit a threat in OpenCRQ: correct it in OpenCTI, and the stream brings the change. Besides the stream, **Analyze new bundle** imports a STIX bundle file; a threat imported this way, and only that one, can be deleted. The **Source** of a threat says which way it came: **OpenCTI**, **OpenCRQ Default Threats** or **STIX import**.

## Read the threat center

Three figures head the list, for the whole catalogue whatever the filters: **Active threats**, the threats seen in the last 90 days out of all of them; **Strong match with your outcomes**, the threats at 60% or more on one of your attack outcomes; and **Targeted CVEs**, the CVEs the threats target, with how many of them are findings on your estate.

:::steps
1. Go to :menu[Threats > Threats]. Each line is a threat, with its **Type**, its **Top outcome**, its **Observed activity**, the number of **CVE** and **TTP** it carries and its **Source**.

   ::screenshot{id=opencrq-threat-center alt="The threat center of Grand Meridian Hotels, 2 Active threats out of 9: Ember Tide matches Ransomware at 99% and its campaign Operation Low Season at 71%, both Active and last observed on Apr 8, 2026, and Velvet Harbor matches Data Breach & Resale at 71%, Dormant since its last activity on Jan 9, 2026."}

2. Read the **Top outcome**: the attack outcome the threat most likely leads to, with that probability. A model OpenCRQ ships gives each threat a probability for each attack outcome, from what its sources say of it: its description and the reports about it, its declared motivations and goals, its labels, its malware and its techniques. Only an outcome above 50% is named. For a threat of **OpenCRQ Default Threats**, the dataset OpenCRQ publishes can give judged probabilities instead. They replace the model's on the outcomes they cover, and the model fills the others. When the judgment covers every outcome, the threat's **Attack outcomes** card says "Judged by OpenCRQ on" and the date of the judgment. Choose an outcome in **Filter by outcome** to compare every threat on that outcome: the column becomes **Probability**.
3. Read the **Observed activity**:
   - **Active**: last observed less than 90 days ago, with the date beside it. The chip is red under 14 days.
   - **Dormant**: longer ago.
   - **Not dated**: no source gives a date of activity.

   Only observed dates count: the start and stop times of the threat's relationships, its last seen date, and those of the threats attributed to it, such as a campaign attributed to an intrusion set. Such a date climbs through attributions stated at or above the **Confidence gate**, up to three levels, and not from a campaign attributed to more than three threats. The date a report was written does not make a threat active.
4. Select a threat. Its header gives its type, its MITRE ATT&CK group ID when it has one, **Open in OpenCTI**, its last observed activity, the outcome it best matches, how many of your countries and sectors it targets, and its **Resource level**. **Overview** shows its **Description**, the countries and sectors of yours it targets under **This concerns you**, its **Attack outcomes**, each with its **Probability** and **Weighs on** when it is above 50%, its **Profile**, with its **Declared intent**, and its **Attributions**, as a list or a graph. **Locations & Sectors** maps the **Targeted countries** and lists the locations and sectors, marking those that concern you. **CVE** lists the CVEs it targets, filtered **On your estate** or not, and **TTP** its techniques, with how your controls hold against each in **Prevention** and **Detection**. The "i" beside a control's figure opens its **Engine efficacy**. When OpenAEV tested a control on a technique the control does not declare, the engine leaves those tests out. The dialog then offers "Declare on" and the control's name, which links the technique to the control on that function without leaving the threat. Each line links back to OpenCTI when the threat came from the stream.

   ::screenshot{id=opencrq-threat-center-threat alt="The overview of Ember Tide, an intrusion set: Active, last activity observed on Apr 8, 2026, a best match of 99% on Ransomware, three countries and sectors of yours, and its attack outcomes, Ransomware at 99% and Data Theft Extortion at 76% marked Weighs on."}
:::

## How a threat gets into a scenario

A risk scenario names an attack outcome on primary assets of a risk scope, never an attacker. A threat weighs on the scenario when the model gives it a probability above 50% of leading to the scenario's outcome; the further above, the more it weighs. Its sources must also say something of what it is after: a description, a report summary of more than 100 characters, a declared motivation or goal, or a threat it is attributed to that does. Techniques and malware alone are not enough: its **Attack outcomes** then read **Not enough data on this threat to assess its attack outcomes.** A threat whose probabilities the dataset of **OpenCRQ Default Threats** judges needs none of this: the judgment stands as its evidence.

Targeting a scope does not make a threat weigh on its own: it raises the weight of a threat that already weighs. Targeting the risk scope's country multiplies it by 2.5, its sector by 4, both by 10. A country or sector claim whose confidence is below the **Confidence gate** does not count. Open a scenario's **Threat** tab to see the **Relevant Threats** that weigh on it, each with:

- its **Score**, out of 100: the relevance times the threat's activity. A threat with no dated activity counts as active once, **Assumed activity age (days)** ago (365 by default);
- its **Relevance**, **Strong**, **Moderate** or **Weak**, from its probability on the outcome and its targeting;
- its **Observed activity** and **Resource level**;
- the **Countries** and **Sectors** that match your scope.

Click a line to see why the threat weighs: its **Score**, **Relevance** and **Observed activity**. Under **Why the relevance is**, the panel gives the country, the sector and the attack outcome it matches, with the claims behind each. The attack outcome says where its probability comes from: **Assessed by OpenCRQ's internal model**. When the dataset of **OpenCRQ Default Threats** judged it, it says "Judged by OpenCRQ on" and the date instead.

::screenshot{id=opencrq-threat-center-relevant-threats alt="The Relevant Threats of the scenario Ransomware on the Booking Platform, 5 entities by Score: Ember Tide at 74, Strong, observed 4 days ago; Operation Low Season at 13, Moderate, 4 days ago; Velvet Harbor at 5 and Harbor Syndicate at 0, Weak, 3 months ago; each targeting France and the hospitality sector."}

A scenario that no threat weighs on is not priced: OpenCRQ prices it again once a source brings a threat that qualifies. While OpenCRQ weighs the threats of a new or changed scenario, its tabs say **Weighing the threats for this scenario. This page updates on its own once it is done.**

## What a threat changes in the price

A threat does not change how often attacks come: the [threat event frequency](/opencrq/use/threats/threat-event-frequency/) does. It changes who attacks and how far they get.

- **Each threat takes a share of the attacks**, in proportion to its weight. A threat seen recently weighs more: its weight halves every 180 days after its last observed activity, or every 365 days when its sources only date when it was written about. Targeting your country multiplies it by 2.5, your sector by 4, both by 10. And the higher its probability on the outcome, the more it weighs.
- **Each threat plays as its class.** OpenCRQ gives every threat an actor class, from its labels in OpenCTI (state-sponsored, ransomware, hacktivism, crimeware), else from its resource level. For a threat of **OpenCRQ Default Threats**, the dataset can state the class instead. For a campaign, that class can be the one of the groups it is attributed to. The class sets the attacker's capability, patience and stealth in the attack run. The classes are **State sponsored**, **Organized cybercrime** (ransomware and extortion labels), **Hacktivist**, **Opportunistic crime**, and **Unclassified actor** when nothing says. A scenario's **CRQ** tab shows the **Admitted actors** and, in its **Actors** card, the share of attacks each class takes, how often it lands an impact and the loss it accounts for.

The weighting and the gate are set for the tenant in :menu[Settings > Engine > Threat Relevance]: **Confidence gate** (**probably true (60)** by default), **Observed anchor half-life (days)** (180), **Authored anchor half-life (days)** (365) and **Assumed activity age (days)** (365). See [Set how OpenCRQ weighs threats, controls and assets](/opencrq/configure/quantification-engine/).

:::example[Ember Tide is back]
On 10 April 2026, OpenCTI reports a new campaign against hotel chains, Operation Low Season, attributed to Ember Tide. The stream brings it to Grand Meridian Hotels the same day.

In the threat center, the campaign reads **Active**, and so does Ember Tide: its own last observed activity is 5 January 2026, but the campaign attributed to it was observed in April, and that date climbs to Ember Tide. The campaign matches **Ransomware** at 71%, Ember Tide at 99%; Ember Tide's overview lists the campaign under **Attributions**.

On the **Threat** tab of the scenario Ransomware on the Booking Platform (Grand Meridian France), the **Relevant Threats** are the five threats above 50% on **Ransomware**. Ember Tide leads with a **Score** of 74 and a **Strong** relevance, ahead of its campaign Operation Low Season at 13: both were observed 4 days ago. Velvet Harbor, also admitted for Data Breach & Resale, scores 5, and Harbor Syndicate and Velvet Harbour, a second record of Velvet Harbor under another spelling, score 0: their last activity is 3 months old, so they weigh little. On the **CRQ** tab, the **Actors** card splits the attacks between the classes of these threats, with the share each one takes, how often it lands an impact, and the part of the $254K of annual loss it accounts for. The amounts are in your currency.
:::

## Check that it works

- The threat center lists the threats of your OpenCTI stream, with **OpenCTI** as their **Source** and a **Last ingested** date in the **Profile** of their overview.
- Each risk scenario's **Threat** tab lists its **Relevant Threats**, and its **CRQ** tab counts its **Admitted actors**.

## What's next

- [Set the threat event frequency](/opencrq/use/threats/threat-event-frequency/) of a scenario.
- [Review the findings on your assets](/opencrq/use/attack-surface/findings/): a CVE that an active threat cites scores higher.
- [Connect OpenCTI with a Live Stream](/opencrq/use/sources/opencti/).
