---
title: Read the risk catalogues and manage asset tags
description: Read the attack outcomes, business resources, loss drivers, asset categories, sectors and countries OpenCRQ prices risk with, and rename, merge or delete the tags of your assets.
sidebar_position: 3
doc_type: how-to
products:
  - opencrq
personas: [risk-manager, admin]
owner: docs-opencrq
last_reviewed: 2026-11-04
---

OpenCRQ prices risk with catalogues it ships: what an attacker is after (attack outcomes), what a primary asset holds (business resources), what an impact costs (loss drivers), what kind of asset an attack goes through (supporting asset categories), and where a risk scope operates (sectors and countries). **Settings > Taxonomies** shows each catalogue as your tenant reads it, with what it means for your own scenarios, so you can justify a choice to an auditor or a board. Only the tags of your assets are yours to change: this page also renames, merges and deletes them.

## Before you start

- You are a member of the tenant. Reading the catalogues and managing tags need no capability.
- Every tenant gets the catalogues when the platform starts, and a new release of OpenCRQ updates them. Nobody edits them in the interface: a business resource says so on its page, **Business resources are platform reference data and cannot be edited.**
- The ATT&CK, CVE and CWE catalogues are on the same menu: see [Explore the ATT&CK, CVE and CWE catalogues](/opencrq/configure/attack-and-vulnerability-catalogues/).

## What each catalogue is for

Open :menu[Settings > Taxonomies], then a catalogue in the menu on the right. **Business Resources** opens first.

| Catalogue | What it holds | Where your scenarios use it | Full list |
| --- | --- | --- | --- |
| **Attack Outcomes** | The seven ends an attacker pursues, such as **Ransomware** or **Data Breach & Resale** | A risk scenario names one. It decides which threats are admitted and how the attack ends | [Attack outcomes](/reference/opencrq/attack-outcomes/) |
| **Business Resources** | The ten kinds of value a primary asset holds, such as **Personal data** or **Payment card data** | A primary asset holds one or more, and a scenario takes those of its primary assets when it is created or its primary assets change. With the outcome, the scenario's business resources decide which loss drivers OpenCRQ recommends | [Business resources](/reference/opencrq/business-resources/) |
| **Loss Drivers** | What an impact costs, grouped by loss category, such as **Extortion** or **Business Interruption (own operations)** | A scenario prices its loss drivers as ranges | [Loss drivers](/reference/opencrq/loss-drivers/) |
| **Supporting Asset Categories** | The thirteen kinds of asset, such as **Host**, **Identity**, **Web application** or **Source code** (the repositories, pipelines and packages your teams build), with the number of ATT&CK techniques that apply to each (**TTP count**). **Security platform** holds your detection and prevention tools (EDR, SIEM, email security): no technique applies to it | An attack only uses the techniques that apply to the assets it goes through. A security platform is inventoried but never priced as a target of loss | Open a category for its techniques |
| **Sectors** | The 22 sectors, such as **Hospitality** | A risk scope has one. A threat that targets it weighs four times more in the scope's scenarios, and the threat event frequency catalog reads it | |
| **Countries** | The 250 countries, with their **ISO Alpha-2** and **ISO Alpha-3** codes | A risk scope has one. A threat that targets it weighs 2.5 times more, ten times when it also targets the sector | |
| **Tags** | The tags of your assets, set by hand or reported by a connector | The labels an asset carries. Tags do not enter a price | |

The generated references list the full catalogues of this version, with what each entry covers. The screens add what only your tenant knows.

## Read an attack outcome against your scenarios

An attack outcome's page shows what it costs you, which threats pursue it and which controls reduce it.

:::steps
1. Open :menu[Settings > Taxonomies > Attack Outcomes], and click an outcome.

   ::screenshot{id=opencrq-taxonomies-attack-outcome alt="The Ransomware attack outcome of the Grand Meridian Hotels tenant: a reference dwell of 3 days, an exposure of $254K from its one priced scenario, loss drivers of $8.1M, and the top matching threats led by Ember Tide at 99% and Operation Low Season at 71%."}

2. Read **Overview**:
   - **Reference dwell** and **Criteria reached**: how long an attacker usually stays before the outcome, and which of confidentiality, integrity, availability and traceability it breaches.
   - **Exposure**: the annual loss of every risk scenario that names the outcome, summed. Each scenario is priced on its own, so the total is not the cost of all of them landing together. **Loss drivers** below it adds up what one occurrence costs, from the ranges recorded on those scenarios: a different measure, never compared with the exposure.
   - **Threat in the wild** and **Top matching threats**: the threats that lead to this outcome, ranked by their **Probability**. OpenCRQ's internal model gives each threat a probability of leading to each outcome, and a threat appears once it is above 50 %. Click a threat to open its page. See [Set how OpenCRQ weighs threats, controls and assets](/opencrq/configure/quantification-engine/).
   - **Controls that reduce this outcome**: the controls whose simulated upgrade removes the most annual loss across the outcome's scenarios, first. The order ranks; it does not price a set of controls together.
3. Open **TTP** for the techniques that mark the outcome, the reference every threat is matched against, in a matrix or a list.
:::

To see when a loss driver applies, open :menu[Settings > Taxonomies > Loss Drivers], expand a loss category and click a driver. **When this driver applies** crosses each attack outcome with each business resource: **This driver applies**, **Realistic pairing, driver not called**, or **Pairing not held realistic**. A recommendation never restricts: a scenario can price any loss driver.

:::example[Explain the ransomware exposure to the board]
Before the budget review, Grand Meridian's risk manager opens :menu[Settings > Taxonomies > Attack Outcomes] and clicks **Ransomware**. On 12 April 2026, the page reads:

- **Reference dwell**: 3 days. **Criteria reached**: confidentiality, availability, and integrity partly; traceability is not reached.
- **Exposure**: $254K, **1 of 1 priced**: Ransomware on the Booking Platform, in Grand Meridian France.
- **Top matching threats**: Ember Tide, an intrusion set, at a **Probability** of 99%, then the campaign Operation Low Season at 71%.
- **Controls that reduce this outcome**: Secure Remote Access (VPN / ZTNA), Patch Management and Network Intrusion Prevention, first.

The board hears one sentence per line: the ransomware risk costs $254K a year in expectation, Ember Tide is why, and remote access and patching are where the money goes first.
:::

## Manage the tags of your assets

A tag labels assets, such as all the assets of the booking platform. Tags come from three sources: a person who tags an asset (**Manual**), a push from XTM One (**XTM One**), and a connector that reports the tags of its own tool, such as OpenAEV. OpenCRQ counts as an XTM One push every tag written through its API with a service account's token. A person's tags and a push's tags never replace each other: each only replaces its own. The same idea often arrives under several names; merge them into one.

:::steps
1. Open :menu[Settings > Taxonomies > Tags]. Each tag shows its **Source** (**Manual**, **XTM One** or the connector), the names connectors report it under, the **Merged tags** it holds and its number of **Assets**. Search with **Search tags...** and filter with **Filter by source**.

   ::screenshot{id=opencrq-taxonomies-tags alt="The four Tags of the Grand Meridian Hotels tenant, all Manual: booking-platform on 2 assets, front-desk on 3, pci-dss and remote-access on 1 each."}

2. Open the menu at the end of a tag's row, and choose what to do:
   - **Rename**: give the tag a new **Name**, and click **Save**. The names connectors report stay listed under **Name in** followed by the connector, so the connector keeps finding the tag. If the name is taken, OpenCRQ offers to merge the two tags instead.
   - **Merge into…**: choose the tag to keep under **Merge into**, and click **Merge**. Every asset of the merged tag carries the chosen one.
   - **Undo a merge…**: shown on a tag that holds merged tags. Click **Restore** next to a merged tag to separate it again.
   - **Delete**: confirm to remove the tag from every asset. Tags merged into it are deleted too, and a connector that still reports the tag creates it again at its next sync.
:::

A merge keeps the merged tag's name: renaming or deleting the tag you kept never loses where its assets came from, and **Undo a merge…** brings it back as it was.

## What's next

- [Explore the ATT&CK, CVE and CWE catalogues](/opencrq/configure/attack-and-vulnerability-catalogues/), the techniques and flaws behind the attack outcomes.
- [Set how OpenCRQ weighs threats, controls and assets](/opencrq/configure/quantification-engine/).
- [Build and price a risk scenario](/opencrq/use/quantify/risk-scenarios/), with an attack outcome and loss drivers.
- [Attack outcomes](/reference/opencrq/attack-outcomes/), [Business resources](/reference/opencrq/business-resources/) and [Loss drivers](/reference/opencrq/loss-drivers/), the full catalogues of this version.
