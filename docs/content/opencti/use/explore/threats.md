---
title: Explore threats
description: Follow a threat in the OpenCTI Threats menu - who it targets, who is behind it, what it uses, how its techniques change - and judge how solid each piece of that knowledge is.
sidebar_position: 8
doc_type: how-to
products:
  - opencti
personas: [cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/exploring-threats/
---

The **Threats** menu is where you follow an adversary over time: the threat actors, the intrusion sets they run and the campaigns they carry out. This page shows how to find the threats that target you, read their profile, check what an attribution rests on, and follow how their techniques change. How to read the tabs every entity page shares is explained once in [Read an entity page](/opencti/use/explore/#read-an-entity-page).

## Before you start

- Your role has the **Access knowledge** capability to read threats.
- To create a threat, change it or link it, your role also needs **Create / Update knowledge**; to delete one, **Delete knowledge**. See [Create knowledge manually](/opencti/use/ingest/manual/).

## How OpenCTI represents a threat

| Entry | Use it for |
| --- | --- |
| **Threat actors (group)** | The people behind attacks, as an organization: a state service, a crime syndicate, hacktivists |
| **Threat actors (individual)** | A real person behind attacks, described by personal attributes |
| **Intrusion sets** | A consistent set of tactics, tools and infrastructure, tracked whoever operates it |
| **Campaigns** | A series of attacks over a period, against a defined set of victims |

The four types connect through `attributed-to` relationships: a campaign is attributed to an intrusion set, an intrusion set to a threat actor. Track a threat as an intrusion set while you do not know who runs it, then attribute it when you do. Feeds do not all draw the line in the same place: some publish as an intrusion set what others call a threat actor group.

Attribution carries knowledge upward. When the inference rules **Usage propagation via attribution** and **Targeting propagation via attribution** are on, the actor inherits what its intrusion set uses and targets, as inferred relationships. See [How OpenCTI infers new relationships](/opencti/use/enrich/inferences/).

## Find the threats that target you

:::steps

1. Open :menu[Threats > Intrusion sets], or the entry for the kind of threat you follow. Each card shows the aliases (**Known as**), **Used malware**, **Targeted countries** and **Targeted sectors**.
2. Search a name or an alias, or filter: **In regards of** keeps the threats linked to an entity, such as a sector you serve.
3. Click **Bookmark this item** on the threats you follow. Bookmarks come first in the list and feed the bookmarks widget of your dashboards.

:::

:::example[List the intrusion sets that target the hospitality sector]
Example CERT serves hotel chains. In :menu[Threats > Intrusion sets], add the filter **In regards of**, operator **Equals**, **Relationship type** `targets`, **Entity** **Hospitality**. The **Velvet Harbor** card remains, known as FIN99 and Harbor Mantis, with Velvet Loader, Harbor Stealer and HarborGate backdoor as used malware. Bookmark it. Repeat in :menu[Threats > Campaigns]: Operation Night Desk and Operation Checkout Rush target the sector too.
:::

To be alerted when a new threat targets your sector, instead of checking the list, create a [priority intelligence requirement](/opencti/use/analyze/pir/).

## Read the profile of a threat

:::steps

1. Click a card. **Overview** opens: **Details** describes the threat, **Basic information** says who states it and how sure they are.
2. Open **Knowledge**. Its **Overview** counts the reports, indicators and relationships of the threat, and draws the **Diamond**: adversary, infrastructure, victimology and capabilities.
3. Go deeper from the side menu. **Victimology** says who it targets; **Malware**, **Tools** and **Vulnerabilities** what it uses; **Attack patterns** how it operates. **Indicators** and **Observables** give what to detect, **Campaigns** its operations.

:::

::screenshot{id=opencti-threats-intrusion-set-diamond alt="The Knowledge tab of an intrusion set opened on the diamond model: its aliases, its last used infrastructures, its last targeted countries and sectors, and its last used attack patterns and malware."}

The **Details** of a threat hold what the sources say about the adversary itself:

| Field | What it means | On |
| --- | --- | --- |
| **First seen**, **Last seen** | When the threat was first and last active, as the sources report it, not when the data arrived | All four types |
| **Resource level** | The means behind it, from an individual to a government | Intrusion sets, threat actors |
| **Primary motivation**, **Secondary motivations** | Why it attacks: financial gain, espionage, ideology | Intrusion sets, threat actors |
| **Goals** | What it wants to achieve, entered one per line in **Goals (1 / line)** of the edit form | Intrusion sets, threat actors |
| **Threat actor types**, **Sophistication**, **Roles** | What kind of actor it is, its skill, and its role in an operation | Threat actors |
| **Personal motivations** | What drives the person | Threat actors (individual) |
| **Objective** | What the campaign aims at | Campaigns |

:::only{opencti=">=7.261030"}
Intrusion sets and threat actor groups also have a **Score**, from 0 to 100, which your team or the source sets, for example to rank the threats that matter most to you. OpenCTI does not compute it.
:::

:::example[Profile Velvet Harbor]
Open **Velvet Harbor**. **Details** gives an activity from March 4 to December 18, 2025, the resource level **organization** and the primary motivation **financial-gain**. Its two goals are to steal payment card data and to resell guest records. **Basic information** shows the author Example CERT, the marking TLP:AMBER and a confidence level of 75.

In **Knowledge**, **Victimology** lists the Hospitality sector, France, Spain and Grand Meridian Hotels, down to the hotel chain's booking database server, its night auditor Paul Garnier and the European Hospitality Summit its lures posed as. **Malware** lists Velvet Loader, Harbor Stealer and HarborGate backdoor. **Attack patterns** places Phishing, Spearphishing Attachment, Valid Accounts, PowerShell and Web Protocols on the matrix. You can now brief the hotel chains: phishing of front-desk staff, then their accounts used against the booking platform.
:::

## Check what an attribution rests on

A profile is only as good as the relationships behind it. Each relationship has its own author, confidence and sources, so check them before you repeat a claim.

:::steps

1. In **Knowledge**, open **Attribution** to see who the threat is attributed to, or any entry such as **Malware**.
2. Switch to **Relationships view**: each line shows the **Confidence** and the author of the link.
3. Click a line to open the relationship. Its **Description** says why the source states the link, and **Latest containers about the object** lists the reports it comes from.
4. Back on the threat, **Distribution of reports**, in the knowledge **Overview**, shows how many authors report on it: one author means one source.

:::

A line with a wand icon is inferred by a rule, not stated by a source: its page shows the **Inference explanation**. A link whose other end shows **Restricted** points to an object your markings hide from you. [The page of a relationship](/opencti/use/explore/#the-page-of-a-relationship) describes every field.

:::example[Weigh the links of Velvet Harbor before a briefing]
In **Knowledge** > **Attribution**, Velvet Harbor is attributed to Harbor Syndicate with a confidence of 60, "2 - Probably True" on the default Admiralty scale. In **Malware**, **Relationships view**, the link to Harbor Stealer has a confidence of 30, "4 - Doubtful". Its description reads "A single source links Harbor Stealer to Velvet Harbor", and one report backs it. In the briefing, present the attribution as probable and Harbor Stealer as unconfirmed.
:::

[Reliability and confidence](/opencti/concepts/reliability-confidence/) explains the scales. When a new source confirms a link, raise its confidence with **Edit** on the relationship page.

## Follow how its techniques change

:::steps

1. In **Knowledge**, open **Attack patterns**. **Matrix view** places the techniques of the threat on the MITRE ATT&CK matrix; **Display only used techniques** hides the others.
2. In the knowledge **Overview**, open **Timeline**, click the settings icon and set **Date reference** to **Functional date**. Each relationship is now dated by its **Start time**, the day the source says it began: the timeline follows what the threat did, not when you learned it.
3. Click a technique in the timeline to open its relationship: **Start time** and **Stop time** give the period the threat used it.
4. To see which of these techniques your defenses miss, go back to **Matrix view** and pick your security platforms in **Compare with my security posture**.

:::

::screenshot{id=opencti-threats-intrusion-set-attack-patterns alt="The Attack patterns entry of an intrusion set in Matrix view, showing only the techniques it uses: Phishing with its sub-technique Spearphishing Attachment, Valid Accounts, PowerShell and Web Protocols."}

:::example[See how Velvet Harbor changed its entry point]
A hotel chain asks whether blocking booking attachments is still enough. In **Velvet Harbor** > **Knowledge**, the **Timeline** on **Functional date** dates Spearphishing Attachment March 4, 2025, Valid Accounts October 1 and PowerShell October 15. The relationship to Spearphishing Attachment has a **Stop time** of September 30. Answer: since October, Velvet Harbor logs in with stolen front-desk accounts; add multi-factor authentication to the booking platform.
:::

**Courses of action view** lists the mitigations of each technique, to hand to defenders. Every view and option is described in [Attack patterns](/opencti/use/explore/#attack-patterns).

## Read a threat actor as a person

A threat actor (individual) adds two cards to its **Overview** when they hold data:

- **Demographic Information**: **Country of Residence**, **Citizenship**, **Nationality**, **Place of Birth**, **Date of Birth**, **Gender**, **Marital Status**, **Ethnicity** and **Job Title**.
- **Biographic Information**: **Eye Color**, **Hair Color**, **Height** and **Weight**. Height and weight can hold several measures, each with the date it was observed.

Country of residence, citizenship and nationality are relationships to countries, not text. **Add country of residence**, **Add citizenship** and **Add nationality**, in the card, link the person to a country, so the link shows in **Knowledge** and in graphs. The other fields are edited in the **Demographics** and **Biographics** tabs of the edit panel.

To say that a person works for a group or an organization, create the relationship with **Create Relationship**. Pictures of threat actors and intrusion sets are managed in **Pictures Management**, in the **Data** tab.

## When the same threat appears twice

Sources name threats differently, and OpenCTI only recognizes a threat it already has when a name or an alias matches exactly: see [Deduplication](/opencti/concepts/deduplication/#names-and-aliases). Search a few letters of the name to spot near-identical cards. In the demo data, `Velvet` finds Velvet Harbor from Example CERT, Velvet Harbour from Saltmarsh CERT and Velvet Harbor Group from Bluefin Labs: one intrusion set reported under three names. Merge them so that its knowledge sits in one place: see [Merge duplicates](/opencti/use/enrich/merge/), which requires **Merge knowledge**.

## Summarize a threat

**AI Insights**, next to the tabs of a threat, rates its recent activity, digests the reports about it and forecasts how it may evolve. It requires the Enterprise Edition: see [Read the home dashboard and AI summaries](/opencti/use/explore/insights/). Intrusion sets and campaigns also offer **Add Security coverage**, to test with a simulation whether your defenses detect the threat: see [Measure your defenses against a threat](/opencti/use/analyze/security-coverage/).

::::only{opencti=">=7.260811"}
:::xtm-one[Summarize a threat with XTM One]
When XTM One is connected, an XTM One AI agent writes the **AI Insights** of a threat, for example a digest of last month's reports on Velvet Harbor for your weekly briefing. Check its answer against the reports it cites before you share it. See [XTM One in OpenCTI](/xtm-one/use/in-opencti/).
:::
::::

## What's next

- [Explore arsenal](/opencti/use/explore/arsenal/) for the malware, tools and vulnerabilities threats use.
- [Explore techniques](/opencti/use/explore/techniques/) for their attack patterns.
- [Pivot and investigate](/opencti/use/analyze/pivot/) from a threat to the objects around it.
- [Follow your priority threats with PIRs](/opencti/use/analyze/pir/) to follow the threats that matter to you.
- The [data model reference](/reference/opencti/data-model/#threats) for the attributes and relationships of each type of the Threats menu.
