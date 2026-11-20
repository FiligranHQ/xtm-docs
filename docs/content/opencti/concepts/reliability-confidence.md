---
title: Reliability and confidence
description: How OpenCTI rates the reliability of sources and the confidence of information, and how max confidence levels decide who can create and change knowledge.
sidebar_position: 7
doc_type: explanation
products:
  - opencti
personas: [cti-analyst, admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/reliability-confidence/
---

OpenCTI rates knowledge on two axes. **Reliability** says how much you trust a source, such as Example CERT or a commercial vendor. **Confidence** says how credible a piece of information is, whoever said it. Confidence also does a third job: every user, connector and feed has a **Max Confidence Level**, and the platform compares it with the confidence of an object to decide who may change it. This page explains how to read an assessment such as B2, what your confidence rating protects, and why a feed could not overwrite your analysis, or why you could not edit an object.

## What it is good for

| Goal | Example |
| --- | --- |
| Assess a piece of knowledge at a glance | A report from Example CERT, a usually reliable source (B), with information rated probably true (2): B2 |
| Work only with trusted sources | Filter a list of indicators on the reliability of their author, to leave out an unreliable scanner feed |
| Protect what your analysts validated | A feed with a max confidence level of 30 cannot overwrite an Indicator your team rated 75 |
| Rank connectors and feeds by trust | Example CERT's feed writes at 80, an unreliable scanner feed at 30 |

## Reliability of a source

Reliability measures the trust you have in a source, based on its history and its means. You usually set it once per organization or person, because it takes the whole history with that source to judge it.

You set reliability on the objects that can be authors: Organizations, Individuals and Systems. You can also set it on a Report, to rate the original author when the report reached you through a provider, and on an Indicator.

The overview of an object shows its **Reliability**. When the object has none, it shows the reliability of its author, marked "of author". To work only with reliable sources, filter lists on **Reliability (self or author)**, which takes the object's own reliability or else its author's.

Reliability is an open vocabulary, `reliability_ov`, that you change in :menu[Settings > Taxonomies > Vocabularies]. By default, it holds the reliability scale of the [Admiralty code](https://en.wikipedia.org/wiki/Admiralty_code), from "A - Completely reliable" to "F - Reliability cannot be judged", as the [vocabularies reference](/reference/opencti/vocabularies/#reliability-ov) lists them. See [Manage labels, vocabularies and templates](/opencti/configure/taxonomies/).

## Confidence of information

Even a reliable source can be wrong. Confidence rates the credibility of the information itself, from what is known about the subject and how well other sources corroborate it. Analysts who know the subject usually set it.

:::note[Why confidence, and not credibility]
Intelligence doctrine separates the credibility of information from the confidence of an assessment. Most teams use a single confidence level, so OpenCTI merges credibility into confidence. Teams that want both axes use confidence with reliability.
:::

Every entity and every relationship has a **Confidence level**: a number from 0 to 100. Observables, such as IP addresses and domain names, have none: they carry a score instead, and any user who can update knowledge can change them. A scale gives meaning to the numbers, with ticks and labels. You set the scale per entity type in :menu[Settings > Customization > Entity types], on the confidence attribute, and you can start from a template with **Selected scale template**:

| Template | Levels |
| --- | --- |
| Admiralty, the default | The credibility scale of the Admiralty code: "6 - Truth Cannot be judged" at 0, "5 - Improbable" from 1, "4 - Doubtful" from 20, "3 - Possibly True" from 40, "2 - Probably True" from 60, "1 - Confirmed by other sources" from 80 |
| Objective | How the information was obtained rather than how much you believe it: "Cannot be judged" at 0, "Told" from 1 (reported to the source, not checked), "Induced" from 25 (inferred from similar information), "Deduced" from 51 (a logical conclusion), "Witnessed" from 75 (seen by the source itself) |
| Standard | The historical levels of OpenCTI: "Low" from 1, "Med" from 30, "High" from 70 |

You can change any template into a custom scale; the number stored on the object does not change, only its label. With the Admiralty code on both reliability and confidence, the overview of an object reads like the NATO notation: A1, B2, C3.

::screenshot{id=opencti-reliability-confidence-overview alt="The Basic information card of a report shows the author Example CERT, its reliability B - Usually reliable marked of author, and a confidence level of 2 - Probably True."}

:::example[Read B2 on a Velvet Harbor report]
Your organization rates Example CERT "B - Usually reliable". Its report "Velvet Harbor targets hotel booking systems" reaches your team, and your analyst who follows the hospitality sector rates it 60, "2 - Probably True". A SOC analyst who opens the report reads B2 in **Basic information**: a usually reliable source with probably true information, so the indicators in it deserve to be blocked.

A national CSIRT that uses the Objective scale instead rates what it saw during an incident "Witnessed", what it concluded from its investigation "Deduced" or "Induced", and what it relays without checking "Told".
:::

## What your confidence level controls

Users do not all have the same expertise, and connectors, feeds and synchronizers each write as a user. The **Max Confidence Level** of a user says how far you trust what it writes; when an override exists for the entity type of an object, the override applies instead. It works as a ceiling:

- **On creation**, the confidence of a new object is the confidence it comes with, capped at the creator's max level. An object created without a confidence gets the creator's max level.
- **On update and deletion**, a user can change or delete an object only if its confidence is not above the user's max level. Otherwise the change fails with "User effective max confidence level is insufficient to update this element". Adding labels and external references is the exception: any user who can update knowledge can add them.
- **On a change of confidence**, a user cannot set a confidence above their own max level: the value is capped. An object that had no confidence gets the user's max level at its first change.
- **On a merge**, the user needs a max level at least equal to the confidence of every entity merged.
- **On an incoming duplicate**, the confidence decides which values are replaced and which are only filled when empty. See [What an incoming duplicate can change](/opencti/concepts/deduplication/#what-an-incoming-duplicate-can-change).

A user without any max confidence level cannot create or update knowledge: the platform answers "User has no effective max confidence level and cannot create this element".

For you as an analyst, this has two consequences. The confidence you give an object protects it: a feed whose level is below it can add labels and fill empty fields, but cannot rewrite your description or your aliases. And when you cannot edit an object, its confidence is above your level: ask a colleague with a higher level, or your administrator.

## How the effective level is computed

This section is for administrators, who set the levels.

::diagram{id=opencti-reliability-confidence-effective}

You set a **Max Confidence Level** on groups and, optionally, on users, in the **Confidences** tab of their edition panel. Each can also have overrides, with **Add a specific max confidence level for an entity type**. The user's effective level is computed this way:

1. The effective max level is the highest max level among the user's groups.
2. The overrides of all groups add up. When two groups override the same entity type, the higher value wins.
3. If the user has their own max level, turned on with **Enable user Max Confidence Level**, it replaces everything that comes from the groups, overrides included.
4. Each override set on the user replaces the value that comes from the groups for that entity type.
5. A user with the **Bypass all capabilities** capability always has an effective level of 100, without overrides.

A group's max level is required: a group without one shows a warning, because its members might not be able to create data. The details view of a user shows the effective **Max Confidence Level**, and its tooltip says which group it is inherited from. See [Organize users in groups](/opencti/administer/groups/#confidence-levels) and [Manage users](/opencti/administer/users/).

The level of a connector is the level of the account it writes as. Give each source a level that matches your trust in it: a feed set too high overwrites your analysts' work; a feed set too low cannot correct its own earlier data once an analyst has raised its confidence. See [Connectors](/opencti/use/ingest/connectors/).

:::example[Rank the sources of a hotel group's CTI team]
The analysts of Grand Meridian Hotels write at 75, through their group CTI analysts. The feed of Example CERT, rated "B - Usually reliable", writes as a service account at 80. The Example Scanner Feed, rated "E - Unreliable", writes at 30.

When the scanner feed sends the Indicator of 203.0.113.10, which your analysts rated 75, it cannot change their description, but it can add its label and fill a field they left empty. When Example CERT's feed sends it at 80, its description replaces theirs, and 80 becomes the confidence of the Indicator.
:::

## What's next

- [Deduplication](/opencti/concepts/deduplication/): how confidence decides what an incoming duplicate can change.
- [Organize users in groups](/opencti/administer/groups/#confidence-levels): set max confidence levels and overrides.
- [Entity types](/opencti/configure/entities/): customize the confidence scale of a type.
- [Manage labels, vocabularies and templates](/opencti/configure/taxonomies/): change the reliability vocabulary.
- [Connectors](/opencti/use/ingest/connectors/): the account and confidence level a connector writes with.
