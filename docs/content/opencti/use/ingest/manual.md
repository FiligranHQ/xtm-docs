---
title: Create knowledge manually
description: Create OpenCTI entities one at a time or in bulk, link them with relationships, and add objects to reports, groupings and cases from the interface.
sidebar_position: 11
doc_type: how-to
products:
  - opencti
personas: [cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/manual-creation/
---

A briefing, a phone call with a partner CERT or your own analysis tells you something new about a threat: you record it by hand, so that it joins what the team already knows and others can find it. This page shows how: create an entity, create several of the same type at once, link entities with relationships, and fill a container. For knowledge your team submits often in the same shape, a [form](/opencti/use/ingest/forms/) is faster.

## How OpenCTI holds what you know

Before you click, decide which object carries each piece of what you learned:

| You learned | Record it as |
| --- | --- |
| A group of attackers, such as Velvet Harbor | An entity: an Intrusion Set, or a Threat Actor when you know who is behind it |
| A tool, a technique, a targeted sector | An entity: a Malware or a Tool, an Attack Pattern, a Sector |
| A fact that links two of them, such as "Velvet Harbor targets the hospitality sector" | A relationship, with its own dates, confidence and markings |
| A document that gathers your analysis | A container: a report, a grouping or a case, which contains the entities and relationships it talks about |
| An address, a domain or a file hash | An observable, and an Indicator when it serves detection |

OpenCTI does not store the same object twice. If you create an entity that already exists, under its name or one of its aliases, the platform updates the existing one instead. See [Deduplication](/opencti/concepts/deduplication/).

## Before you start

- Your role has the **Create / Update knowledge** capability.
- You know the [containers](/opencti/concepts/containers/) concept: reports, groupings and cases contain objects, they are not related to them.
- What you create gets at most your own confidence level. See [Reliability and confidence](/opencti/concepts/reliability-confidence/).

## Create an entity

:::steps

1. Open the list of the type you want, for example :menu[Analyses > Reports].
2. Click the create button at the top right, such as **Create Report**. A form opens on the right.
3. Fill in the fields. The required ones are marked: some come with the type, others are set by your administrator in [entity type customization](/opencti/configure/entities/), along with default values.
4. Click **Create**. The entity appears in the list.

:::

## Create several entities at once

For types that support it, such as Malware, you can create many entities that share the same fields in one go. Reports and other containers do not support it.

:::steps

1. Open the list of the type and click its create button.
2. Click **Create multiple entities** at the top right of the form.

   ::screenshot{id=opencti-ingest-manual-bulk-button alt="The Create a malware panel, with the Create multiple entities button at its top right, next to Close."}

3. Type one name per line, then confirm. The names appear in the **Name** field.

4. Fill in the other fields. Every entity gets the same values: a marking set to TLP:GREEN applies to all of them.
5. Click **Create**. A progress bar counts the entities created; close it when it ends.

:::

## Create a relationship

A relationship goes from one entity to another, and the type of relationship depends on both types: a malware **targets** a sector, an intrusion set **uses** a malware. The entity you start from is the source of the relationship, so start from the right one.

:::steps

1. Open the entity that is the source of the relationship and go to its **Knowledge** tab.
2. In the menu on the right, choose the category of the target, for example **Victimology** to link a malware to a sector.
3. Click the create button. A panel opens to find the targets.
4. Search for the entities to link and select them. You can select several. If a target does not exist yet, create it from this panel.
5. Click **Continue**.
6. Choose the relationship type and fill in its fields, such as its dates, confidence and markings: a relationship is an object, with fields like an entity.
7. Click **Create**.

:::

**Create Relationship**, at the top of an entity, does the same from any screen of the entity, with every compatible type of target.

## Add objects to a container

:::steps

1. Open the report, grouping or case and go to its **Entities** tab, or its **Observables** tab for observables.
2. Click the add button. The **Add entities** panel opens.
3. Search for the objects and click each one. It is added to the container at once.

:::

:::example[Record the new tools of Velvet Harbor from a briefing]
A partner briefing tells your team that Velvet Harbor now also uses two malware families, Velvet Courier and Harbor Wiper, against the hospitality sector. You record it by hand.

1. Open :menu[Arsenal > Malwares], click the create button, then **Create multiple entities**, and type `Velvet Courier` and `Harbor Wiper`, one per line.
2. Fill in the shared fields: **Is family?** on, **Author** `Example CERT`, **Markings** `TLP:GREEN`. Click **Create**: the two malware are created with the same fields.
3. Open the intrusion set Velvet Harbor, tab **Knowledge**, and choose **Malwares** in the menu on the right. Click the create button, select Velvet Courier and Harbor Wiper, click **Continue**, choose the relationship type **uses**, set its **Confidence level** to 60, as the briefing has not been confirmed yet, and click **Create**.
4. Open the report "Velvet Harbor targets hotel booking systems", tab **Entities**, click the add button and click Velvet Courier and Harbor Wiper.

The **Knowledge** tab of Velvet Harbor now shows that it uses the two new malware, and the report contains them.
:::

## Other places to create knowledge

- **The knowledge graph** of a container and **investigations** let you create entities and relationships on the canvas. See [Pivot and investigate](/opencti/use/analyze/pivot/).
- **An analyst workbench** lets you add entities, observables and relationships before you import them. See [Review what a file import found](/opencti/use/ingest/workbench/).
- **A draft** holds anything you create until someone approves it. See [Prepare changes in a draft](/opencti/use/ingest/drafts/).

:::xtm-one[Write descriptions with AI]
When AI is available on your platform, text fields such as **Description** offer **Ask AI**: it can **Fix spelling & grammar**, **Make it shorter**, **Make it longer**, **Change tone** or **Summarize** what you wrote, and you keep or discard the result. See [XTM One in OpenCTI](/xtm-one/use/in-opencti/).
:::

## Check that it works

Open the new entity: its **Overview** shows the fields you set, and its **Knowledge** tab shows its relationships. In a container, the **Entities** tab lists what you added.

## What's next

- [Create knowledge with forms](/opencti/use/ingest/forms/), for knowledge you submit often.
- [Import from files](/opencti/use/ingest/files/), for knowledge you already have in a document.
- [Customize entity types](/opencti/configure/entities/), to set required fields and default values.
