---
title: Explore locations
description: See which threats target a region, a country or a city in the OpenCTI Locations menu, what was seen there, and why a threat can show on a place no source named.
sidebar_position: 12
doc_type: how-to
products:
  - opencti
personas: [cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/exploring-locations/
---

The **Locations** menu answers "where": where threats strike, where victims are, and where indicators were seen, from continents down to precise coordinates. This page shows how to see the threats around a place, how to tell what sources stated from what the platform inferred, and what each kind of location records. The tabs and panels every entity page shares are described in [Explore knowledge](/opencti/use/explore/#read-an-entity-page); this page covers what is specific to locations.

## Before you start

- Your role has the **Access knowledge** capability to read these pages.
- To link a place to a threat or to another place, your role also needs **Create / Update knowledge**; to delete, **Delete knowledge**. See [Create knowledge manually](/opencti/use/ingest/manual/).

## What the Locations menu holds

| Entry | What it is | Example |
| --- | --- | --- |
| **Regions** | A large territory, often a continent or a part of one | Europe, Western Europe |
| **Countries** | A country | France |
| **Administrative areas** | An area inside or across countries, often without precise borders | The Balkans, the Persian Gulf |
| **Cities** | A city or a town | Lyon |
| **Positions** | A precise point, with coordinates and an address | A hotel, a data center |

Places nest through **located at** relationships: a city is located in a country, a country in a region. That nesting is what lets the platform carry a targeting from a country up to its region.

## See which threats target a place

:::steps

1. Open :menu[Locations > Countries], or the entry you need, and search the name or an alias, such as `FR`.
2. Open the location. Its **Overview** shows a map that locates it: the map of a region highlights the countries located in it.
3. Open **Knowledge**. Its **Overview** gives the same charts as for any victim: **Distribution of reports** by author, **Top 10 threats targeting this entity** and **Sightings distribution**. See [Explore entities](/opencti/use/explore/entities/#see-which-threats-target-a-victim).
4. In the side menu, open **All threats**, or one entry such as **Intrusion sets** or **Campaigns**, to list what targets the place. The number beside an entry counts its relationships.
5. Open **Sightings** to see which indicators were seen in that place, as a **True positive** or a **False positive**, with their **Count**, **First obs.** and **Last obs.**

:::

::screenshot{id=opencti-locations-country-overview alt="The Overview of France: its aliases FR and FRA, its description, a mini map centered on it, and its basic information."}

Besides the threats, the side menu places the location among the others:

- **Regions**, **Countries**, **Areas** and **Cities** list the places it is located in and the places located in it: a region lists its countries, a country its cities and the regions it belongs to.
- **Organizations** lists the organizations located there. Use it to see which of your constituents sit in a targeted country.
- **Observables** lists the observables located in the place or related to it, such as the IP addresses a source geolocated there.

:::only{opencti=">=7.261127"}
The number beside **Observables** counts the observables of that list only, each one once, even when two relationships link it to the place.
:::

:::example[See which threats target France]
Your hotel group runs most of its hotels in France. Open :menu[Locations > Countries], search `France` and open it. **Overview** centers the map on France and lists, among its latest relationships, the intrusion set **Velvet Harbor** that targets it.

In **Knowledge**, the side menu lists **Velvet Harbor** under **Intrusion sets**, **Copper Lantern** and **Harbor Syndicate** under **Threat actors** and **Operation Night Desk** under **Campaigns**. **Organizations** lists **Grand Meridian Hotels**, located in France: one of your members sits in a targeted country.
:::

**AI Insights** on a region or a country summarizes the activity and the reports about it: see [Read the home dashboard and AI summaries](/opencti/use/explore/insights/).

## Why a threat shows on a place it never targeted

A source rarely states every level: a report says that Velvet Harbor targets France, not Europe. With the rule **Targeting propagation via location** turned on, the platform infers that a threat targeting a place also targets every place it is located in. A region then lists the threats against its countries, and a country the threats against its cities.

To tell what was stated from what was inferred:

:::steps

1. In the side menu of the location, open **All threats**.
2. Switch the list to **Relationships view**. An inferred line carries a wand icon, whose tooltip reads **Inferred knowledge based on the rule** and the rule's name. Its **Creators** column names the rule manager, and its **Start time** and **Stop time** come from the relationships it was inferred from: the period they share. The same threat can appear twice: once stated by a source, once inferred.
3. Click an inferred line. Its description reads **Inferred knowledge**, and **Inference explanation** names the rule and the relationships it started from.

:::

::screenshot{id=opencti-locations-region-inferred-threats alt="The threats targeting Europe in relationships view: intrusion sets, threat actors, campaigns, malware, tools, attack patterns and an incident, all inferred by the rule manager with a wand icon, and Operation Night Desk listed a second time, stated by a source."}

An inferred relationship lasts as long as one of the relationships it comes from: Velvet Harbor targets Europe through France and through Spain, so deleting the targeting of France leaves the inference, explained by Spain. If the rule is off, a region lists only the threats stated against it: an empty region does not mean that its countries are safe. An administrator turns rules on in [Turn inference rules on or off](/opencti/configure/rules/); see [How OpenCTI infers new relationships](/opencti/use/enrich/inferences/).

:::example[Brief on the threats to European hotels]
Your manager wants the threats to the group's European hotels. Open :menu[Locations > Regions], then **Europe**, then **Knowledge** > **All threats**, and switch to **Relationships view**.

Every line but one shows the rule manager as creator: no source names Europe, the platform inferred the threats from the places they target in it, France, Italy and Spain, and from what they target there, such as Grand Meridian Hotels and its booking database server in France. Operation Night Desk shows twice: the report **Operation Night Desk targets European hotels**, by Example CERT, states that it targets Europe, and the rule also infers it from France. In your briefing, cite the countries for the inferred threats, and the Example CERT report for Operation Night Desk.
:::

## Read the details of a location

- A region, a country, an area or a city has a **Description** and aliases, such as `FR` and `FRA` for France, which the search box matches.
- A position gives its **Latitude**, **Longitude**, **Street address** and **Postal code**, and the **City**, **Country** and **Region** it lies in. Use a position for a precise target, such as the head office of a hotel chain.

To draw targeting on a map for a dashboard, use the map widget: see [Dashboard widgets](/opencti/use/dashboards/widgets/).

## What's next

- [Explore entities](/opencti/use/explore/entities/) for the sectors and organizations in those places.
- [Explore threats](/opencti/use/explore/threats/) for the actors that target them.
- [Follow your priority threats with PIRs](/opencti/use/analyze/pir/) to follow threats against the places that matter to you.
- The [data model reference](/reference/opencti/data-model/#locations) for the attributes and relationships of each type of the Locations menu.
