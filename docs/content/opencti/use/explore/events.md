---
title: Explore events
description: Find out which organizations you follow were hit, by what and by whom, from the incidents, sightings and observed data of the OpenCTI Events menu.
sidebar_position: 6
doc_type: how-to
products:
  - opencti
personas: [cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/exploring-events/
---

The **Events** menu holds what actually happened on the systems you protect or follow: incidents, sightings of indicators by security tools, and raw observations from logs. It is where your threat knowledge meets reality: which hotel chain was hit, when, by which infrastructure, and whether it matches the campaign you are tracking. This page shows how to find an event, read its page and tie it to a threat. The tabs every page shares are described in [Explore knowledge](/opencti/use/explore/#read-an-entity-page).

## Before you start

- Your role has the **Access knowledge** capability.
- Attributing an incident, recording or qualifying a sighting needs **Create / Update knowledge**. Deleting an event needs **Delete knowledge**.

## What the Events menu holds

| Entry | What it records | Typical source |
| --- | --- | --- |
| **Incidents** | A negative event on an information system, from a qualified alert to an intrusion, or an information operation in a disinformation context | A victim's report, a SIEM or EDR alert qualified by an analyst, an inference rule |
| **Sightings** | The fact that something, usually an indicator, was seen somewhere, how many times and when | A SIEM or an EDR, through a connector |
| **Observed datas** | Raw observations from a log, such as a firewall line, that hold observables without saying what they mean | A log collector or a sharing partner |

An incident in **Events** is the event itself. The work your team does to respond to it is an incident response, in **Cases**: see [Explore cases](/opencti/use/explore/cases/).

## Find out what hit an organization

:::steps
1. Open :menu[Events > Incidents].
2. Filter on **Incident type**, **Severity** or a date to find the incidents that matter.
3. Click an incident to open it on **Overview**.
:::

Each line of the list also gives the **Assignees**, **Labels**, **Processing status** and **Marking** of the incident. Sort on **Severity** to put the worst first, or on **Original creation date**, the date the source recorded the incident, to put the latest first.

::screenshot{id=opencti-events-incident-overview alt="The Overview of the incident Unauthorized access to booking database: incident type Compromise and severity high, first seen on January 9, 2026 at 2:10 AM, reported by Grand Meridian Hotels, and an Entities distribution of the attack patterns, places, people, systems and threats linked to it."}

The **Details** of an incident tell you how serious it is and what it was about:

| Field | What it tells you |
| --- | --- |
| **Incident type** | The kind of event, such as `compromise`, `phishing` or `ransomware` |
| **Severity** | How bad it is for the victim, from `low` to `critical` |
| **First seen**, **Last seen** | When the activity started and stopped, not when it was recorded |
| **Source** | Who or what detected it |
| **Objective** | What the attacker was after, when known |
| **Entities distribution**, **Observables distribution** | The kinds of objects and values linked to the incident |

:::only{opencti=">=7.261030"}
An incident also carries a **Score**, from 0 to 100, set by its source or by your team to rank incidents across sources. The incidents list shows it in its **Score** column, which you can sort on.
:::

**Latest containers about the object** lists the reports and cases that hold the incident, often the quickest way to its story. **Knowledge** opens on the diamond model, a timeline and the global kill chain, as for threats. Its side menu splits what is linked to the incident: **Attribution** for the threats it is attributed to, [**Victimology**](/opencti/use/explore/#victimology) for who and where it hit, then the malware, attack patterns, indicators and observables involved.

**AI Insights** summarizes the incident: see [Read the home dashboard and AI summaries](/opencti/use/explore/insights/). **Add Security coverage** measures with OpenAEV whether your defenses would stop the same attack: see [Measure your defenses against a threat](/opencti/use/analyze/security-coverage/).

:::example[Find recent compromises at member hotels]
In :menu[Events > Incidents], add three filters:

- **Incident type**, operator **Equals**, value `compromise`.
- **Severity**, operator **Equals**, values `high` and `critical`.
- **Platform creation date**, operator **Within**, **From** `now-30d` **To** `now`.

The list shows **Unauthorized access to booking database**, reported by Grand Meridian Hotels: a front-desk account used at night to export guest records. **Latest containers about the object** lists the report **Night-time exports from a hotel booking platform** and the incident response **Booking system intrusion at a hotel chain**, which hold it together with Velvet Harbor, and the hotel chain's own report, **Night-time export of guest records at Grand Meridian Hotels**.
:::

## Attribute an incident to a threat

When the evidence points to an actor, link the incident to it: anyone who opens the threat then finds the incident in its knowledge, and the incident feeds the timeline of the threat.

:::steps
1. Open the incident, then **Knowledge** > **Attribution** to see whether it is already attributed.
2. Click **Create Relationship** in the header and pick the intrusion set, threat actor or campaign.
3. Choose the relationship type **attributed to**, set the **Confidence level** your evidence supports, and click **Create**.
:::

:::example[Attribute the booking database intrusion to Velvet Harbor]
The incident **Unauthorized access to booking database** shows the same pattern as the Velvet Harbor reports: stolen front-desk accounts, then night-time exports from the booking platform. Open the incident, click **Create Relationship**, pick the intrusion set **Velvet Harbor**, choose **attributed to** and set a **Confidence level** of 60, since the link rests on the method only.

The incident now shows under **Attribution**, and under **Incidents** in the **Knowledge** tab of Velvet Harbor. Raise the confidence level when an indicator of Velvet Harbor is sighted on the hotel's systems.
:::

Some incidents are created by the platform itself. When the inference rule **Raise incident based on sighting** is on, every sighting of a valid indicator at an organization, a sector or another identity creates an incident named *Sighting on valid indicator* followed by the indicator name, related to the indicator and targeting the place it was seen. See [Inferences](/opencti/use/enrich/inferences/).

## Sightings

A sighting is a relationship, not an object: it links what was seen to where it was seen, such as an organization, a sector, a system, an individual or a location. It answers the question "did this indicator hit the people we protect?", with how many times and when.

:::steps
1. Open :menu[Events > Sightings]. The list opens with an empty **False positive** filter: set it to keep or hide false positives.
2. Filter on the indicator, the place it was seen, or the dates.
3. Read each line: **Qualification** says whether it is a true or a false positive, **Nb.** how many times it was seen, **Entity** where, and **First obs.** and **Last obs.** over which period.
4. Click a sighting to open it.
:::

::screenshot{id=opencti-events-sightings-list alt="The sightings list: each line gives what was seen, such as the indicators booking-velvetharbor.example and 203.0.113.10, how many times, where, such as Grand Meridian Hotels or the hospitality sector, and its first and last observation."}

A sighting page shows what was seen, **sighted in/at** where, its **First seen** and **Last seen** dates, whether it is a **True positive** or a **False positive**, and its **Count**. Like any relationship, it also shows its author, markings, confidence level, external references and notes. When an inference rule created it, **Inferred knowledge** and **Inference explanation** say which rule and which facts.

:::example[Check what the security tools saw at Grand Meridian Hotels]
In :menu[Events > Sightings], add **Target entity**, operator **Equals**, value **Grand Meridian Hotels**. Three sightings remain: the indicator `booking-velvetharbor.example`, with a **Count** of 14 between January 8 and 9, 2026, the indicator `203.0.113.10`, with a **Count** of 3 in December 2025, and the event European Hospitality Summit 2026, seen 4 times from January 5 to 8 in the booking requests that named it. Both indicators point to Velvet Harbor, so the hotel chain has been in contact with its infrastructure twice, three weeks apart. To follow the intrusion inside the chain, filter on its systems and people too, such as the booking database server, where `203.0.113.10` was seen on the night of January 9.
:::

When the hotel shows that the traffic was legitimate, open the sighting, click **Update**, and turn on **False positive** in **Update a sighting**. The sighting stays, so the history is kept, and its **Qualification** reads **False positive** in every list.

You also find sightings from the other side, in the **Sightings** tab of an indicator, an observable, an organization or a location. To record one yourself, see [Record that your tools saw it](/opencti/use/explore/observations/#record-that-your-tools-saw-it).

## Observed data

Observed data keep the raw observation that STIX 2.1 defines: a set of observables seen together over a time window, without an intelligence assertion. Few sources use them; when they do, they give the volume behind an activity.

The page of an observed data has **Overview**, **Entities**, **Observables**, **Data** and **History**. Its **Entity details** show the **First observed** and **Last observed** dates, the **Number observed**, and an **Observables distribution** chart that stays empty: the **Observables** tab lists what was observed.

::::only{opencti=">=7.261030"}
Two more counters tell a single large burst from a widespread activity:

| Counter | What it counts | When it changes |
| --- | --- | --- |
| **Number observed** | How many times the observables were seen | Added up when a new ingestion extends the observation window |
| **Number seen** | How many observation events were ingested | Added up at every ingestion, by 1 or by the value the source provides; 1 at creation |
| **Max distinct count** | The highest number of distinct sources, such as users or endpoints, that saw the data at the same time | Keeps the highest value received; empty until a source provides it |

For example, one host sending 60,000 requests in a batch gives `Number observed = 60000`, `Number seen = 1` and `Max distinct count = 1`. A beaconing candidate deployed widely gives 60,000 for all three.

An ingestion with a lower confidence level than the existing observed data does not change these counters; it only fills them when they are empty. See [Reliability and confidence](/opencti/concepts/reliability-confidence/).

::screenshot{id=opencti-events-observed-data-counters alt="The details of an observed data: its first and last observed dates, with Number observed, Number seen and Max distinct count side by side."}
::::

## What's next

- [Explore observations](/opencti/use/explore/observations/) for the indicators and observables that events refer to.
- [Explore threats](/opencti/use/explore/threats/) to read the victimology and timeline an attributed incident feeds.
- [How indicators age and expire](/opencti/use/analyze/indicators-lifecycle/) to see how sightings affect an indicator.
- [Handle incidents and requests in cases](/opencti/use/analyze/cases/) to respond to an incident.
- The [data model reference](/reference/opencti/data-model/#events) for the attributes and relationships of each type of the Events menu.
