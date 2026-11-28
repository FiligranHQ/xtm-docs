---
title: Follow your priority threats with PIRs
description: Turn what your organization most needs to know into an OpenCTI PIR that flags, scores and follows the threats targeting your locations and industries.
sidebar_position: 3
doc_type: how-to
products:
  - opencti
edition: ee
personas: [cti-lead, cti-analyst, risk-manager]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/pir/
---

A priority intelligence requirement (PIR) states what your organization most needs to know, such as "which threats target European hotels?". OpenCTI keeps the answer up to date: it flags every threat that matches, scores how closely it matches, and logs what changes. Your team gets a standing priority, your analysts a list to work from, and your executives a one-screen answer. This page shows how to scope a PIR, create it, read it, and act when it changes.

## Before you start

- Your platform has an Enterprise Edition license.
- Your role has **Access PIR** to read PIRs, and **Create / Update / Delete PIR** to create and manage them.

## How a PIR finds its threats

A PIR does not search the knowledge base once. The PIR manager reads the stream of changes and checks each relationship against the PIR:

::diagram{id=opencti-pir-flagging}

- **What gets flagged.** A campaign, intrusion set, threat actor or malware that **targets** one of the PIR locations or industries, through a relationship whose confidence reaches the PIR minimum. Relationships inferred by the rules engine count, so a threat that targets Spain also matches a PIR on Europe.
- **The score** is the share of criteria a threat matches. With two criteria, Europe and the energy sector, a malware that targets the energy sector only scores 50; one that targets both scores 100.
- **Threats leave** when the relationships that matched are deleted or stop matching. The score drops first, then the threat leaves the PIR when nothing keeps it in.
- **The rescan period** sets how far back in the stream the PIR starts reading at creation. A relationship older than that is ignored until it changes again.

## Decide the scope

You cannot change the criteria of a PIR after creation, so settle them with the people who asked the question:

| Choice | What it changes for the team |
| --- | --- |
| **Targeted locations** and **Targeted industries** | Each one is a criterion. A threat that matches any of them enters the PIR, and the score tells how many it matches. Add the countries you care most about next to their region: threats that target them score higher than those that target the region only |
| **Minimum confidence of the relationship** | Leaves out relationships from sources you trust less. Too high, and the PIR misses early, single-source warnings; too low, and the analysts chase claims nobody confirmed |
| **Rescan period (days)** | How much history the PIR starts with. **6 months** gives a first picture for a quarterly review; **No rescan** follows only what happens from now on |

A PIR is visible to everyone who has **Access PIR**, and only its creator can change or delete it. To let others manage it, or to keep it to some people, click **Manage access restriction** in its header. The button shows only to a user who can manage the PIR and whose role also has **Manage credentials**.

## Create a PIR

:::steps

1. Open :menu[PIR] and click **Create PIR**.
2. Under **General settings**, enter a **Name** and a **Description** that states the question the PIR answers.
3. Choose the **Rescan period (days)**: **No rescan**, **1 day**, **1 month** (the default) or **6 months**. If the stream holds less history than the period, OpenCTI warns you and reads what the stream holds. Click **Next**.

   ::screenshot{id=opencti-pir-create alt="The Create priority intelligence requirement dialog on its General settings step, with the Name, Description and Rescan period fields."}

4. Under **Entities selection**, pick the **Targeted locations** and the **Targeted industries**. You need at least one location or one industry.
5. Set the **Minimum confidence of the relationship**.
6. Click **Create**. The PIR starts flagging threats from the rescan date onward.

:::

After creation, you can change the name and the description only, with **Update**. To change the criteria, create a new PIR.

:::only{opencti=">=7.261127"}
A script that creates a PIR through the API gets an error when its filters use a key the PIR manager cannot check on the stream of changes, such as `Stix filtering is not compatible with the provided filter key`. The filters the form builds always pass.
:::

:::example[Follow the threats to European hotels]
The CISO of a hotel group asks Example CERT for a standing answer to "which threats target hospitality in Europe, and France first of all?". Create the PIR with:

- **Name**: `Threats to European hospitality`
- **Description**: `Intrusion sets, campaigns and malware that target hotels in Europe.`
- **Rescan period (days)**: **6 months**, to pick up the campaigns reported since the summer
- **Targeted locations**: `Europe` and `France`
- **Targeted industries**: `Hospitality`
- **Minimum confidence of the relationship**: `50`, to leave out single-source claims

The PIR has three criteria. Velvet Harbor, Copper Lantern, Harbor Syndicate, Operation Night Desk and the malware Velvet Loader target the hospitality sector, France and Europe, some of them through inferred relationships, so they score 100 and top the **Threats** tab. Operation Checkout Rush targets hotels in Spain: it matches the hospitality sector and Europe, not France, and scores 67. The malware Harbor Stealer targets the hospitality sector only and scores 33.
:::

## Read a PIR

Open the PIR from the list. Its tabs give four angles on the same threats:

| Tab | What you get |
| --- | --- |
| **Overview** | The state of the PIR and its trends, for a briefing |
| **Threats** | The flagged threats with their **Score** and **Last score evolution**, highest score first. Hover a score to see which criteria the threat matches. Fill the **Last score evolution** filter to see what changed this week |
| **Analyses** | The reports and other containers that mention a flagged threat or a location or industry of the PIR: the reading list of your analysts |
| **Activities** | Every event that matters to the PIR: a threat entering or leaving, a score going up or down, a relationship of a threat created or deleted, a threat added to or removed from a container |

::screenshot{id=opencti-pir-threats alt="The Threats tab of a PIR listing intrusion sets, threat actors, campaigns and malware with their score gauge, sorted from the highest score."}

The **Overview** holds what you show in a briefing:

- **Threat map**: each threat updated in the last week, placed from **One week ago** to **Today** and from **0 - Less relevant** to **Most relevant - 100**. The threats in the top right corner are the ones to talk about. Click a type in the legend to hide it.
- **Number of threats** by type, and **Number of threats over time**, to show whether the pressure grows.
- **News feed**: the latest entries, exits and score changes.
- **Top authors of threat entities** and **Top authors of relationships from threats**: the sources your answer rests on. A PIR fed by one source only deserves a second one.
- **PIR Details**: the settings, with **Filters** showing the minimum confidence, and the **Processing delay**. **ON TIME** means the PIR is up to date; a delay followed by **behind live stream** means new threats show up in the PIR later than in the knowledge base. Hover the icon for the **Last event processed**. Below it, the number of **messages in queue** counts the events still waiting to be processed for this PIR: when it keeps growing, so does the delay, and your administrator should look at the workers.

::screenshot{id=opencti-pir-threat-map alt="The Threat map of a PIR: threats placed by the date of their last update over the past week and by score, with campaigns, intrusion sets, threat actors and malware in their own colors."}

:::note
The **Analyses** tab looks up containers from the first 5,000 flagged threats only, so on a very broad PIR some containers can be missing. The **Activities** tab leaves out **indicates** relationships, which would flood it.
:::

### For risk managers and executives

The **Threats** tab sorted by score is the prioritized list of who threatens the scope you care about, and the **Overview** shows how it moves. Use the threats that score 100 as the threats to price in a risk analysis, or to check your defenses against in [Measure your defenses against a threat](/opencti/use/analyze/security-coverage/). If you cannot see it, ask the CTI team to share it with you: they do it with **Manage access restriction**.

## Act when a PIR changes

- **Run a playbook.** Start it with the **Listen PIR events** entry point and pick the events: **A new entity enters a selected PIR**, **An entity has left a selected PIR**, **An entity from a selected PIR has been updated** or **An entity is linked to an entity from a selected PIR**. The playbook can then notify the team that owns the PIR or open a case. See [Create a playbook](/opencti/use/automate/playbooks/create/).
- **Filter on the score.** The **PIR Score** filter works in playbooks, live streams and triggers, so you act only on the threats that matter, for example those scoring 100.

## What's next

- [Explore threats](/opencti/use/explore/threats/), to dig into a flagged intrusion set or malware.
- [Get alerted when knowledge changes](/opencti/use/automate/notifications/), to be told when the threats of your PIR change.
- [Handle incidents and requests in cases](/opencti/use/analyze/cases/), to turn a new high-score threat into work.
