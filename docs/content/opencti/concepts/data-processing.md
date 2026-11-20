---
title: Data processing
description: What OpenCTI does to each object on its way in, from the STIX bundle to the knowledge base, why it explains what analysts see, and what administrators set to control it.
sidebar_position: 5
doc_type: explanation
products:
  - opencti
personas: [cti-analyst, admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/reference/data-processing/
---

OpenCTI does not store data as it arrives. Each object from a connector, a feed, a file or another platform is checked, matched against what you already have, and written within the trust of its source. Once it is written, background processes go on with it: they infer relationships, age indicators and run your automations. This page follows an object through these steps. It explains what you see as an analyst, such as a report that fills in over a few minutes or a feed that does not overwrite your assessment. It also lists what administrators set to control each step.

## The path of an object

::diagram{id=opencti-data-processing-bundle}

Most data arrives as a Structured Threat Information Expression (STIX) 2.1 bundle. Take Example CERT's advisory "Velvet Harbor targets hotel booking systems", with its intrusion set, malware, indicators and relationships:

1. **The bundle is split.** It becomes one message per object, ordered so that an object comes after the objects it refers to: the label `hotel-sector` and the author Example CERT before the report that points to them.
2. **Workers write each object.** Several workers take the messages from the queue in parallel and send each object to the platform, as the user of the connector, the feed or the person who imported the file.
3. **References are resolved.** The platform finds each object the incoming one refers to. A relationship whose source or target is missing is rejected and tried again later. Another missing reference, such as a label or an object of a report, can hold the object back for a few attempts; then the platform writes it without that reference.
4. **Access is checked.** The user must hold every marking of the object, or the object is rejected. An Indicator must also have a valid pattern that no [exclusion list](/opencti/configure/exclusion-lists/) excludes.
5. **Duplicates are resolved.** The platform looks for an object it already has. It creates the object, or updates the existing one within the limits of confidence.
6. **The change is published.** Each creation and update becomes an event in the live stream, which the background processes, the streams and the notifications read.

How the platform, its workers and its queues carry these steps is in [How OpenCTI works](/opencti/concepts/architecture/).

When the data goes to a [draft](/opencti/use/ingest/drafts/) instead, as an import from a file usually does, steps 2 to 5 happen inside the draft and nothing reaches the live stream. Approving the draft sends its content through the same path, as the user who approved it.

## What it explains for your analysis

| What you see | Why | Where it is set |
| --- | --- | --- |
| A report appears, then its content fills in over a few minutes | Objects are written one by one, by several workers at once | Not configurable |
| A report lacks an object its source listed | The object was rejected (a marking its source may not write, an excluded or invalid indicator), or arrived too late | The connector user's groups, [Keep benign values out of indicators](/opencti/configure/exclusion-lists/) |
| A feed did not overwrite the description or the confidence you set | The feed's confidence, capped at its max confidence level, is lower than the object's confidence | [Reliability and confidence](/opencti/concepts/reliability-confidence/) |
| Two sources about Velvet Harbor point to the same Intrusion Set | The name of one is an alias of the other | [Deduplication](/opencti/concepts/deduplication/) |
| Velvet Harbour and Velvet Harbor Group stay separate from Velvet Harbor | Their names are not aliases of each other yet | Merge them, as [Merging](#merging) explains |
| A relationship appears that nobody created | An inference rule derived it from existing facts | [Turn inference rules on or off](/opencti/configure/rules/) |
| An indicator's score drops, then the indicator is revoked | Its decay rule lowers the score over time | [Age indicators with decay rules](/opencti/configure/decay/) |
| A partner does not see a report you see | Markings or organizations hide it from them | [How OpenCTI decides who sees what](/opencti/administer/segregation/) |

## Deduplication

The platform identifies each object by a few properties, such as the name and aliases of an Intrusion Set or the value of an IP address. Data that matches an existing object enriches it instead of creating a copy: the report keeps its own link to it, and the object gains the new source's details.

Relationships match on their type, their source and their target. When they have dates, the start and stop times must also be within 30 days of the existing ones. So "Velvet Harbor uses Spearphishing Attachment from March 2025" and the same link starting in October 2025 stay two relationships, which is how you follow an actor's techniques over time. The properties of each type are in [Deduplication](/opencti/concepts/deduplication/).

## Confidence levels

Every user, and so every connector, feed and synchronizer, has a max confidence level, set on its groups or on itself. It caps the confidence of what the user creates. When an object already exists, an update whose confidence, capped at its source's max confidence level, is equal to or higher than the object's confidence replaces its values. An update below it can only fill fields that are still empty and add labels and external references. A community feed at 40 cannot overwrite the Intrusion Set your team rated 80. See [Reliability and confidence](/opencti/concepts/reliability-confidence/).

## Markings and organizations

Markings, such as the Traffic Light Protocol (TLP) levels, classify data. A user only sees, and can only write, data whose markings their groups allow: a connector whose user may not write TLP:RED loses every TLP:RED object of its source. With the Enterprise Edition, organizations add a second barrier: an object shared with some organizations is only visible to their members, and a report or a case shared with an organization shares its content too. See [How OpenCTI decides who sees what](/opencti/administer/segregation/).

## Merging

Some duplicates cannot be recognized: Velvet Harbour and Velvet Harbor Group are the same actor as Velvet Harbor for you, but nothing in their names says so. A user with the **Merge knowledge** capability merges them. Select two to four entities of the same type in a list and choose **Merge**. You keep one entity, and the others are merged into it: their names become aliases, and their relationships and containers move to the kept entity. See [Merge duplicates](/opencti/use/enrich/merge/).

## Inference rules

The rules engine derives knowledge from what you have. For example, when an Indicator indicates Velvet Harbor and is sighted at Grand Meridian Hotels, a rule infers that Velvet Harbor targets Grand Meridian Hotels. Another rule raises an Incident when a valid Indicator is sighted. Every rule is off until an administrator turns it on:

- Turning a rule on applies it to all existing data in the background, then to every change.
- Turning a rule off deletes everything it inferred.

Inferred knowledge is marked as such, with the rule and the facts it comes from. See [How OpenCTI infers new relationships](/opencti/use/enrich/inferences/), and [Turn inference rules on or off](/opencti/configure/rules/) to turn rules on.

## Indicator decay

An Indicator loses value with time. Its decay rule, chosen by the type of observable it detects, lowers its score over time and revokes it when the score reaches the revoke score. A source that sends the indicator again with a new score restarts the decay. See [How indicators age and expire](/opencti/use/analyze/indicators-lifecycle/) and [Age indicators with decay rules](/opencti/configure/decay/).

## What administrators control

Most of these steps follow settings that administrators own. Each one changes what analysts get:

| Setting | Effect | Where |
| --- | --- | --- |
| Max confidence level of each connector's user | Which sources can overwrite which knowledge | [Create accounts for integrations](/opencti/administer/service-accounts/), [Organize users in groups](/opencti/administer/groups/) |
| Markings allowed to each connector's groups | Which classified objects a source can bring in | [Organize users in groups](/opencti/administer/groups/), [Restrict access with markings](/opencti/administer/markings/) |
| Exclusion lists | Indicators that are never created, such as your own IP ranges | [Keep benign values out of indicators](/opencti/configure/exclusion-lists/) |
| Inference rules | The relationships the platform derives | [Turn inference rules on or off](/opencti/configure/rules/) |
| Decay rules | How fast indicators lose their score, and when they are revoked | [Age indicators with decay rules](/opencti/configure/decay/) |
| Relationship date window: `relations_deduplication:past_days` and `relations_deduplication:next_days` (`RELATIONS_DEDUPLICATION__PAST_DAYS`, `RELATIONS_DEDUPLICATION__NEXT_DAYS`), 30 days each | When two dated relationships count as one | [Functional customization](/reference/opencti/configuration/#functional-customization) |
| Playbooks | Your own processing steps on incoming data | [Automate with playbooks](/opencti/use/automate/playbooks/) |
| Retention rules | Which data is deleted after a while | [Retention](/opencti/configure/retention/) |

## What's next

- [How OpenCTI works](/opencti/concepts/architecture/): the platform, workers and queues behind these steps.
- [Deduplication](/opencti/concepts/deduplication/): the properties that identify each type.
- [Reliability and confidence](/opencti/concepts/reliability-confidence/): who can write what.
- [How intelligence gets into OpenCTI](/opencti/use/ingest/): the ways data gets into OpenCTI.
- [Automate with playbooks](/opencti/use/automate/playbooks/): add your own processing steps to incoming data.
