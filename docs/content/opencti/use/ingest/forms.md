---
title: Create knowledge with forms
description: Design an OpenCTI form intake that creates a main entity, related entities and relationships in one submission, then fill it in, directly or into a draft for review.
sidebar_position: 12
doc_type: how-to
products:
  - opencti
personas: [cti-lead, cti-analyst, admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/form-intake/
---

A form intake turns the knowledge your team records again and again, such as an incident reported by a member or a phishing campaign seen by the SOC, into a few fields in your team's words. You design it once, usually as the team lead: one submission then creates a main entity, such as a report, with the entities, observables and relationships you decided. Analysts and partners fill it in without knowing the Structured Threat Information Expression (STIX) model, and every submission has the same shape. You also decide whether submissions reach the knowledge base at once or wait in a draft for review. This page shows how to design a form, fill it in, and share it with another platform.

## What a form decides for your team

| You decide | What it changes |
| --- | --- |
| The main entity and the related entities | What each submission creates, and how it is linked: the same relationships every time |
| The labels and the required fields | What people see and must provide. A field you do not add cannot be filled in |
| **Entity lookup (select existing entities)** | Whether people pick existing entities, such as your member organizations, instead of typing names that create new ones |
| **Create as draft by default** | Whether a reviewer checks each submission before it is shared |

## Before you start

- To design, change, export and delete forms, your role has the **Manage ingestion** capability.
- To fill in a form, your role has the **Create / Update knowledge** capability. A role that only grants **Import knowledge** inside drafts can fill in forms too, and the submission always goes to a draft.
- You know which attributes of each entity type your analysts must provide. Required attributes and default values set in [entity type customization](/opencti/configure/entities/) still apply.

:::only{opencti=">=7.261109"}
**Create knowledge via Form Intake** also lets a role fill in forms, without **Create / Update knowledge**: give it to the people who report incidents but should not edit the knowledge base, such as hotel staff. With it alone, a submission only creates: when an entity of the form already exists, changing or merging it still needs **Create / Update knowledge**, and a new label still needs **Manage labels**. Granted inside drafts only, it lets the role fill in forms in a draft. The upgrade to 7.261109 gives it to every role that has **Create / Update knowledge**: see [Grant rights with roles](/opencti/administer/roles/#capabilities-added-by-an-upgrade).
:::

## Design a form

:::steps

1. Open the form designer.

   ::::only{opencti=">=7.260811"}
   Open :menu[Integrations], go to the **Available** tab and, under **Built-in ingestion**, click **Create** on the **Form intake** card.
   ::::

   ::::only{opencti="<7.260811"}
   Open :menu[Data > Ingestion > Form intakes] and click the create button.
   ::::

2. Enter a **Name** and a **Description** that tell analysts when to use the form. Keep **Active** on: an inactive form is hidden from analysts.
3. Choose the **Main Entity Type**, **Report** by default. The form opens from the list of this type, and this entity is created first. The attributes OpenCTI needs to [deduplicate](/opencti/concepts/deduplication/) the type, and those your administrator made required in [entity type customization](/opencti/configure/entities/), are added as fields for you.

   ::::only{opencti=">=7.261127"}
   Each of these fields takes the default value set in entity type customization as its **Default value**: checked or unchecked for a checkbox, a number for a number. A date whose default is the time of creation is not prefilled. A required reference, such as the author or the markings, is not added: add its field with **Add field**.
   ::::
4. Add the fields of the main entity with **Add field**. For each one, choose the attribute in **Map to attribute**, write a **Field Label** in your team's words, choose the **Field Type**, set the **Field Width** (full, half or third) and turn on **Required** if analysts must fill it in.
5. Choose how many main entities a submission creates. By default, one. Turn on **Allow multiple instances of main entity** to accept several, then choose the **Multiple Mode**:
   - **Multiple fields**: analysts add one block of fields per entity;
   - **Parsed values**: analysts paste a list in one field, split by the **Parse Mode** (**Comma-separated** or **One per line**), and each value fills the attribute set in **Map parsed values to attribute**. The other fields apply to every entity created.
6. To make analysts pick an existing entity rather than create one, turn on **Entity lookup (select existing entities)**. Add **Disable on-the-fly entity creation** to forbid new ones.
7. Add the related entities with **Add additional entity**. Give each a **Label for entities**, such as "Attacker" for an Intrusion Set, and its fields. Turn on **Allow multiple instances** and set a **Minimum amount (0 for optional)**, or keep a single instance and set **Required**.
8. Add the relationships with **Add relationship**: choose the **Source Entity**, the **Target Entity** and the **Relationship Type**, which lists only the types valid between them. Under **Additional Fields**, add the fields analysts fill in for the relationship, such as its confidence level or its start time: they show in a **Relationships** section of the form.
9. Choose where submissions go. Turn on **Create as draft by default** to send them to a [draft](/opencti/use/ingest/drafts/), and **Allow users to uncheck draft mode** to let trusted analysts write straight to the knowledge base.
10. Click **Create**.

:::

### How relationships are created

Analysts do not choose the relationships. Each relationship you add is created on every submission, from every source entity to every target entity the submission holds: with three suspicious addresses and one affected hotel, a `related-to` relationship from the addresses to the hotel gives three relationships. When the submission has no source or no target for it, nothing is created. The **Required** switch of a relationship does not change this.

### Field types

The field types offered for an attribute depend on its type. Markings, labels, authors, external references and files always use their own field.

| Field type | Analysts enter |
| --- | --- |
| Text, Text Area | A line or a paragraph |
| Number | A number |
| Date & Time | A date and time, from a picker |
| Checkbox, Toggle | Yes or no. A toggle starts from the default of the attribute, such as **is_family** for a Malware |
| Select, Multi-Select | One or several values from options you list |
| Open Vocabulary | Values from the OpenCTI vocabulary of the attribute |
| Created By | The author |
| Object Marking | Marking definitions, such as TLP and PAP |
| Object Label | Labels |
| External References | External references |
| Files | Files attached to the entity |

### Containers, indicators and observables

- **A container as main entity.** With a report, a grouping or a case, **Include entities in container** adds the additional entities to it. When a submission creates several containers, they are created one after the other: each also contains the containers created before it.
- **Indicators and observables.** For an Indicator, **Automatically convert to STIX patterns** lets analysts type plain values, such as an IP address, and **Automatically create observables from indicators** adds the matching observables. For an observable, **Automatically create indicators from observables** does the reverse. Turn on one direction only. These switches show once the entity takes **Parsed values**, and the two **Automatically create** switches only for the main entity.

:::only{opencti="<7.261119"}
In this version, the two **Automatically create** switches are saved with the form but create nothing on submission.
:::

::::only{opencti=">=7.260811"}
### Draft settings

Open **Advanced Draft Settings** to prepare the draft each submission creates. For the **Draft Name**, **Draft Description**, **Draft Assignees** and **Draft Participants**, set a default value and choose whether it is **Editable by end user**. A field that is not editable is hidden from the analyst and its default applies; users with **Bypass all capabilities** still see and change it.

- **Draft Author**: choose the **Default author source**: **Main entity author (reuse the same author)**, a **Specific Author**, or **None (no author specified)**. A specific author the analyst cannot see shows as restricted on the draft.
- **Authorized Members**: turn on **Activate access restriction** to restrict who can open the draft. Besides users, groups and organizations, the **Dynamic from draft** members adapt to each submission: the draft creator, its assignees, its participants, and **Draft author (org)**, the organization set as author, which you can intersect with a group. For example, the analysts of the organization that submitted the draft can edit it, while its other members only view it.

Values set in the form override the default and required values of entity type customization for these drafts. An analyst granted **can manage** on a draft still needs the **Manage authorized members** capability to change its authorized members.
::::

## Fill in a form

:::steps

1. Open the form from one of these places:
   - **Import data** in the top bar, then **Import using a Form**;
   - the list of the main entity type, with the **Use a form to create** button next to the create button. It shows on reports, groupings, cases, incidents, campaigns, intrusion sets, threat actors, malware and indicators;
   - the form itself, in the list of forms.
2. Fill in the fields. In a field that takes a list of values, one per line or comma-separated, paste observables as they come: defanged values such as `hxxp://` or `example[.]com` are converted back. Other fields take the values as typed.

   ::screenshot{id=opencti-ingest-forms-fill alt="A form for hotel incident reports, with fields labeled in the team's words for the incident title, the affected hotel and the suspicious IP addresses, one per line, and the Create as draft checkbox."}
3. Check **Create as draft** if the form lets you choose, and set the draft fields it shows.
4. Click **Submit**.

:::

:::only{opencti=">=7.261109"}
A form you open while you work in a draft submits into that draft. Entities you create on the fly in a lookup field are created only when you submit, with their files. When the submission goes to a draft you cannot open, the form says **Your submission has been sent and is being processed.**, with **Submit again** to fill it in once more and **Close**.
:::

:::only{opencti=">=7.261119"}
An entity you pick in a lookup field is only referenced: the submission creates its relationships and adds it to the container when the form includes entities in it, but never changes the entity itself. This holds for every type, an existing Observed Data included.
:::

:::example[Let member hotels report an incident to the sector CERT]
Example CERT wants the hotels of its community to report incidents in their own words: what happened, which hotel, which addresses they saw. Each report must wait for an analyst before it is shared.

The team lead of Example CERT designs the form:

- **Name**: `Hotel incident intake`, **Description**: `How a member hotel reports an incident to the sector CERT.`, **Active** on
- **Main Entity Type**: **Report**, with three fields: `name` mapped as **Field Label** `Incident title`, **Field Type** Text, **Required** on; `description` as `What happened`, Text Area; `objectMarking` as `Markings`, Object Marking
- **Include entities in container**: on, so the report contains the hotel and the addresses
- **Add additional entity**: **Entity Type** Organization, **Label for entities** `Affected hotel`, **Entity lookup (select existing entities)** on, as member hotels already exist on the platform
- **Add additional entity**: **Entity Type** IPv4 address, **Label for entities** `Suspicious IP addresses`, **Allow multiple instances** on, **Multiple Mode** **Parsed values**, **Parse Mode** **One per line**, **Map parsed values to attribute** `value`
- **Add relationship**: **Source Entity** `Suspicious IP addresses`, **Target Entity** `Affected hotel`, **Relationship Type** **related to**
- **Create as draft by default** and **Allow users to uncheck draft mode**: on

A front desk manager of Grand Meridian Hotels then fills it in:

- **Incident title**: `Fake booking confirmations sent to the Lisbon front desk`
- **What happened**: `Three front desk staff received emails confirming bookings they never made, with a link to a login page. One of them entered her password.`
- **Markings**: `TLP:AMBER`
- **Affected hotel**: `Grand Meridian Hotels`
- **Suspicious IP addresses**, one per line: `203.0.113.21` and `203.0.113.22`
- **Create as draft**: kept checked

The submission creates a draft holding the report, the two addresses, each related to Grand Meridian Hotels, and the report containing them all. An analyst of Example CERT reviews it and approves it.
:::

OpenCTI checks the required fields and the syntax of observables, such as IP addresses, domains, URLs, emails and hashes, then builds a STIX bundle and imports it, into the knowledge base or into a new draft. A value with the wrong syntax stops the submission with an error on its field. An entity missing an attribute that your administrator made required is not created.

## Duplicate, export and import forms

Open the actions menu (⋮) of a form:

- **Duplicate** creates a copy to start a variant;
- **Export** downloads its definition as a JSON file: fields, entity types and relationships, without any submitted data;
- **Delete** removes the form. What its submissions created stays.

To bring a form from another platform, click **Import form intake** and choose the JSON file. Check the result when the two platforms run different versions.

::::only{opencti=">=7.260811"}
**Import form intake** is on the **Form intake** card of the **Available** tab of :menu[Integrations]. Your forms are listed in the **Deployed** tab.
::::

## Design tips

- Start with the few fields analysts always know, and add more from their feedback.
- Use **Parsed values** for lists of indicators: pasting one value per line is the fastest way to submit many.
- Mark a field as required only when the data is useless without it.
- Send submissions to a draft when many people submit, so that someone reviews them before they are shared.

## What's next

- [Prepare changes in a draft](/opencti/use/ingest/drafts/), to review and approve submissions.
- [Approve drafts step by step](/opencti/use/automate/draft-workflow/), to decide who approves them.
- [Create knowledge manually](/opencti/use/ingest/manual/), for one-off entries.
