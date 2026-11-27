---
title: Refine content with AI
description: Use Ask AI in OpenCTI to fix, shorten, lengthen, retone, summarize or explain text, generate a report from a container, and search in natural language.
sidebar_position: 4
doc_type: how-to
products:
  - opencti
  - xtm-one
edition: ee
personas: [cti-analyst, soc-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/refine-content/
---

**Ask AI** helps you write and read intelligence faster: it rewrites the description of a report for another audience, drafts a brief from the knowledge of a container, and turns a question into search filters. Every answer comes to you for review first, so nothing in the knowledge base changes until you accept it, and you stay the author of what you share. This page shows each use, and what changes when XTM One answers.

## Before you start

- Your platform has an Enterprise Edition license.
- An administrator has enabled AI on the platform and connected a model: XTM One, or a language model from Mistral AI, OpenAI or Azure OpenAI, hosted by Filigran or by your organization. The administrator sets it in the [configuration](/reference/opencti/configuration/#ai-service).
- Your role has the **Create / Update knowledge** capability to rewrite text and generate reports. **Explain** and search in natural language need **Access knowledge** only.

The **Ask AI** button only appears where AI can help, and stays greyed out until the field holds enough text.

## Improve the text of a field

:::steps

1. Open the edition form of an object, or its **Content** tab, and click **Ask AI** next to a description or content field.
2. Choose what to do:
   - **Fix spelling & grammar**
   - **Make it shorter** or **Make it longer**
   - **Change tone**, then, in **Select options**, pick the **Tone**: **Strategic** for decision makers, **Tactical** for team leaders, **Operational** for technical analysts, and click **Generate**
   - **Summarize**, as bullet points
   - **Explain**, to understand the context of the text. The explanation is for you to read: it cannot replace the text of the field
3. Read the proposal. Edit it if needed, then accept it to replace the text of the field, or click **Retry** to get another one.

:::

::screenshot{id=opencti-refine-content-ask-ai-menu alt="The Update a report panel with the Ask AI menu open next to the description, offering Fix spelling & grammar, Make it shorter, Make it longer, Change tone, Summarize and Explain."}

:::example[Rewrite a threat report for the executive committee]
The description of the report "Velvet Harbor targets hotel booking systems" is written for analysts. The head of security wants to quote it in the monthly briefing to the executive committee of a hotel group.

1. Open the report in :menu[Analyses > Reports] and click **Update**.
2. Click **Ask AI** next to the **Description**, choose **Change tone**, pick **Strategic** in **Tone**, and click **Generate**.
3. Read the proposal: it should keep the facts, front-desk phishing, stolen accounts and exported payment data, and drop the technical detail. Check that the intrusion set is still named Velvet Harbor, then accept it.

To keep the analyst wording in the report, copy the proposal into the briefing instead of accepting it.
:::

## Generate a report from a container

Ask AI can write a text report from the entities and relationships of a report, a grouping or a case.

:::steps

1. Open the container and click the export button in its header. This needs the **Generate knowledge export** capability.
2. In **Generate an export**, choose **Ask AI (multiple formats supported)**.
3. Set the **Format**, the **Number of paragraphs**, the **Tone** and the **Language**, then click **Generate**.
4. Review the result, then choose where it goes under **Select destination**: **Main content**, to put it in the embedded content of the container, or **New file**, to create a file with a **File name**. **New file** needs the **Upload knowledge files** capability.

:::

:::example[Draft a hospitality sector brief from a report]
The CTI team shares a short brief with the security leads of member hotels each time a report on the sector comes in. On the report "Velvet Harbor targets hotel booking systems", click the export button, choose **Ask AI (multiple formats supported)**, and set:

- **Format**: **HTML**, which opens in a browser and in the **Content** tab
- **Number of paragraphs**: `4`, enough for the attack, its targets, the indicators and what to do
- **Tone**: **Operational**, for the security leads who will act on it
- **Language**: English

Click **Generate**. Under **Select destination**, choose **New file** and enter the **File name** `velvet-harbor-hotel-brief`. The file appears in the **Content** tab of the report, where you review and edit it before sending it.
:::

## Search in natural language

Click **Ask AI** in the search bar at the top of the screen and type a question, such as "malware targeting the energy sector". A language model turns it into search filters and OpenCTI lists the matching entities. When the question names an entity, OpenCTI looks it up by name, value or alias; if it finds none, it leaves that part out.

The search runs even when the model only understood part of the question, and a message tells you what it left out:

- **Some entities you mentioned have not been found in the platform**, followed by the names: the results ignore these entities. Check the spelling, or search for an alias.
- **The NLQ model didn't find filters corresponding to your question**: the model found nothing it could filter on, often because the question is outside threat intelligence. Rephrase it with entity types, sectors, locations or names the platform knows.

This search has limits: it finds entities only, not relationships; it does not understand dates; it cannot filter on the properties of related entities; and it combines filters at one level only. See [Search for knowledge](/opencti/use/explore/search/).

:::warning
Natural language search sends long prompts to the model. With your own model provider, it can raise your costs noticeably. On Filigran SaaS platforms, queries go to the Filigran-hosted model at no extra cost.
:::

## Check every answer

The quality of the text depends on the model. Read every proposal before you accept it, especially facts, names and dates. Without XTM One, the answer panel recalls it under each proposal: OpenCTI treats generative AI as a beta feature. When you use the Filigran-hosted model or a model you host, your data is not used to train it.

:::::only{opencti=">=7.260811"}
::::xtm-one[Ask AI with XTM One]
When XTM One is connected to your platform, **Ask AI** hands each request to the XTM One AI agent set up for that task, instead of calling the language model directly. You review and accept the answer as before, with a few differences:

- **Change tone** goes straight to the agent: there is no **Tone** to pick.
- **Explain** answers can be accepted like the others, and there is no **Retry**: ask again from the menu.
- A field needs only two characters, instead of ten, before **Ask AI** becomes available.

:::only{opencti=">=7.261030"}
**Generate an export** with **Ask AI (multiple formats supported)** also goes through XTM One, and offers the **HTML** format only.
:::

Until an administrator accepts the Filigran AI terms, which the first click of an administrator offers, the button tells other users that Ask Ariane is not activated yet. Administrators choose which agent answers each task in XTM One. See [XTM One in OpenCTI](/xtm-one/use/in-opencti/).
::::
:::::

## What's next

- [XTM One in OpenCTI](/xtm-one/use/in-opencti/), for AI insights, the Ask Ariane assistant and AI agents in playbooks.
- [AI-assisted intelligence](/workflows/ai-assisted-intelligence/), a full workflow with AI steps.
- [Search for knowledge](/opencti/use/explore/search/), for the other ways to find knowledge.
