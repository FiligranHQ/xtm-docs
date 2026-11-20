---
title: Meaning of dates
description: Which date of an OpenCTI object answers which question, which dates come from the source and which from your platform, and how imports move them.
sidebar_position: 8
doc_type: explanation
products:
  - opencti
personas: [cti-analyst, soc-analyst, developer]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/dates/
---

When you follow Velvet Harbor, "new this week" can mean three things: what Example CERT published this week, what reached your platform this week, or what the actors did this week. OpenCTI keeps a date for each, and a filter, a sort or a dashboard widget only answers your question if it uses the right one. This page tells you which date answers which question, which dates the source sets and which your platform sets, and how a new import moves them.

## Which date answers your question

| Your question | Date to use |
| --- | --- |
| What did my sources publish this week? | **Publication date** of reports, otherwise **Original creation date** |
| What reached my platform this week? | **Platform creation date** |
| What changed in my platform recently? | **Modification date** |
| When was this actor or campaign active? | **First seen** and **Last seen** |
| When did Velvet Harbor use this technique? | **Start time** and **Stop time** of the relationship |
| When was 203.0.113.10 seen in our network? | **First seen** and **Last seen** of the sighting |
| Is this indicator still worth detecting? | **Valid from** and **Valid until** |

A report that a connector imports today about a campaign of last spring has a **Platform creation date** of today, a **Publication date** of the day the source published it, and objects whose **First seen** is last spring. A dashboard of "reports this week" built on **Platform creation date** therefore includes old reports that a new feed brought in. Build it on **Publication date** to follow what sources say now.

## How dates reach an object

::diagram{id=opencti-dates-sources}

Every entity and relationship carries four dates about the data itself. The overview of an object shows them in its **Basic information** card.

| Label | Set by | Meaning |
| --- | --- | --- |
| **Original creation date** | The source | When the source created the data. When neither the source nor its connector gives it, the date of the import |
| **Modification date**, on the overview | The source, then your platform | When the source last changed the data, as received at the import. After that, the date of each change in your platform |
| **Platform creation date** | Your platform | When the object was created in this platform. It never changes |
| **Modification date**, in lists, filters and dashboards | Your platform | When the object last changed in this platform |

The first two travel with the data when it is shared in Structured Threat Information Expression (STIX) 2.1: a partner CERT that receives your report sees the same **Original creation date**. The last two belong to each platform: the partner's platform sets its own **Platform creation date** when it imports your report.

::screenshot{id=opencti-dates-overview alt="The Basic information card of a report shows its Original creation date and Modification date near the top, and its Platform creation date further down."}

:::note[Two "Modification date" fields]
On the overview, **Modification date** is the STIX modification date, which first holds the source's date. In lists, filters and dashboards, **Modification date** is the platform's own date of last change. They differ for an object that has not changed since its import.
:::

## Dates of the threat

Some types carry dates that describe what happened, not when the data was written:

| Type | Dates | Meaning |
| --- | --- | --- |
| Intrusion Set, Campaign, Malware, Infrastructure, Threat Actor, Incident | **First seen**, **Last seen** | When the threat or its activity was first and last observed |
| Relationship | **Start time**, **Stop time** | When the relationship was true, such as the period an intrusion set used a technique |
| Sighting | **First seen**, **Last seen**, **Count** | When and how often the sighted object was seen at that place |
| Report | **Publication date** | When the report was published |
| Indicator | **Valid from**, **Valid until** | When the indicator is worth detecting. See [Indicator lifecycle](/opencti/use/analyze/indicators-lifecycle/) |
| Observed Data | **First observed**, **Last observed**, **Number observed** | The period and number of the observations |

Dated relationships are how you follow an actor over time: Velvet Harbor uses Spearphishing Attachment from March to September 2025, then Valid Accounts from October 2025. Each period is the **Start time** and **Stop time** of a "uses" relationship.

## How a new import moves the dates

When an object is updated, by hand or because a source sends it again, each date follows its own rule:

- **Platform creation date** never changes.
- **Modification date**, both fields, moves to the date of the change, whenever something actually changes. An import that brings nothing new leaves it as it is.
- **Original creation date** only moves back: when a source sends an older date, the platform keeps the older one.
- **Start time** of a relationship only moves back and **Stop time** only moves forward. **First seen** and **Last seen** of a sighting, and **First observed** and **Last observed** of observed data, do the same. The period widens to cover what every source reported. When it widens, a sighting adds the incoming **Count** to its own, and observed data adds the incoming **Number observed**.
- **First seen** and **Last seen** of an entity, such as an Intrusion Set or a Campaign, do not widen: a source allowed to change the entity replaces them with its own values, even later ones. An Infrastructure keeps its values and only takes the source's when it has none.

:::only{opencti=">=7.261030"}
Observed data also has a **Number seen**, which counts the times the observation was received: each import adds to it, whether the period widens or not.
:::

A source is "allowed to change" an object when its confidence is at least the object's. A source with a lower confidence only fills the dates that are empty. See [What an incoming duplicate can change](/opencti/concepts/deduplication/#what-an-incoming-duplicate-can-change).

:::example[Two sources report when Velvet Harbor used spearphishing]
Your platform holds the relationship "Velvet Harbor uses Spearphishing Attachment", with **Start time** 4 March 2025, **Stop time** 30 September 2025 and a confidence of 75. A partner feed sends the same relationship at confidence 80, with **Start time** 20 February 2025 and **Stop time** 15 October 2025.

Both dates are within 30 days of yours, so the platform treats it as the same relationship. The feed's confidence is higher, so the period widens to 20 February – 15 October 2025: the relationship now covers what both sources saw.

If the same feed sent a **First seen** of June 2025 for the Intrusion Set itself, Velvet Harbor's **First seen** would move from March to June 2025. Check the **History** tab of the entity before you quote its first activity.
:::

## Dates that identify objects

Some dates take part in recognizing an object, so a different date can mean a different object:

- Two reports with the same name but different **Publication date** values are two reports.
- Incidents, groupings, notes, opinions and cases are identified with their **Original creation date**.
- Two relationships between the same objects are the same relationship only when their start times and stop times are within 30 days of each other, by default.

See [Deduplication](/opencti/concepts/deduplication/) for the full rules.

## In the API and in STIX bundles

For developers who write connectors or query the GraphQL API, the dates above map to these fields:

| Label | API field | In an exported STIX 2.1 object |
| --- | --- | --- |
| **Original creation date** | `created` | `created` |
| **Modification date**, on the overview | `modified` | `modified` |
| **Platform creation date** | `created_at` | `created_at`, in the OpenCTI extension |
| **Modification date**, in lists | `updated_at` | `updated_at`, in the OpenCTI extension |
| **Publication date** | `published` | `published` |

The OpenCTI extension is the `extension-definition--ea279b3e-5c71-4632-ac08-831c66a786ba` entry of the object's `extensions`. The API also has `refreshed_at`, which moves each time the object or one of its relationships changes, and which you can filter on.

A connector should send `created`, `modified` and `published` as the source gives them, never the time of its run. These dates identify reports, incidents, notes and cases: a run date creates a new object at each run. The [filter keys reference](/reference/opencti/filter-keys/) lists every date you can filter on.

## What's next

- [Deduplication](/opencti/concepts/deduplication/): how dates help recognize duplicates, and what an incoming duplicate can change.
- [Search for knowledge](/opencti/use/explore/search/): filter on dates.
- [Indicator lifecycle](/opencti/use/analyze/indicators-lifecycle/): how **Valid until** and the score of an indicator change over time.
- [Data model](/opencti/concepts/data-model/): the families of objects and their attributes.
