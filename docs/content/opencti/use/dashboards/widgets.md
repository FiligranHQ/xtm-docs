---
title: Create widgets
description: Build OpenCTI dashboard widgets step by step, choose the right visualization and perspective, filter the data and break a chart down by attribute.
sidebar_position: 2
doc_type: how-to
products:
  - opencti
personas: [cti-lead, cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/widgets/
  - opencti:/usage/tips-widget-creation/
---

A widget answers one question with one picture: how many reports came in, who targets the sector, where, since when. It counts or lists objects of the knowledge base, or events of the platform's activity, and draws them as a number, a chart, a list or a map. This page walks through the four steps of a widget, helps you choose the visualization that fits the question, explains how knowledge graph filters work, and gives complete settings for each example. Widgets live in [custom dashboards](/opencti/use/dashboards/) and [custom views](/opencti/use/dashboards/custom-views/).

## Before you start

- Your role has the **Create / Update dashboards** capability, and you have the **can edit** access level on the dashboard.
- You know which objects and relationships hold the answer. For example, intrusion sets are linked to attack patterns by **uses** relationships, and a report **contains** the entities it covers rather than being related to them. The [data model](/opencti/concepts/data-model/) and [containers](/opencti/concepts/containers/) pages describe these links.

## Build a widget

:::steps

1. Open the dashboard and click **Create Widget**. The panel walks you through **Visualization**, **Perspective**, **Filters** and **Parameters**.
2. **Visualization**: choose how to draw the data. See [Choose a visualization](#choose-a-visualization).

   ::screenshot{id=opencti-dashboards-widgets-visualizations alt="The Visualization step of the widget panel shows one card per visualization, from Text and Number to Bookmark and Word Cloud, above the four steps of the widget."}
3. **Perspective**: choose what the widget counts.
   - **Entities**: objects such as reports, malware or indicators.
   - **Knowledge graph**: relationships between objects, such as an intrusion set that uses an attack pattern.
   - **Activity & history**: what users and connectors did on the platform, such as creations, updates and logins.
4. **Filters**: define the data set, with the same filters as the lists of the platform, which the [filter key reference](/reference/opencti/filter-keys/) lists. Some visualizations accept several data sets, for example one line per data set on a **Line** chart: add one with the buttons under the filters and give each a **Label**, shown as the legend. Click **Validate filters** to go on to the parameters.

5. **Parameters**: give the widget a **Title** and set the options of its visualization, described in [Set the parameters](#set-the-parameters).
6. Click **Create**. The widget appears on the dashboard; move and resize it as you need.

:::

:::only{opencti=">=7.260811"}
### Use a saved filter

In the **Filters** step, the saved filters button switches between your own filters and a saved filter created from a list of the platform. Only compatible saved filters are offered: saved filters of entity lists for **Entities**, of relationship lists for **Knowledge graph**.

A widget keeps a reference to its saved filter, so editing the saved filter changes the widget. Duplicating the dashboard keeps the reference. Exporting the widget, the dashboard or the custom view, or creating a public dashboard, copies the filter content into the result instead, so later changes to the saved filter do not reach it.

If the saved filter is deleted, the widget shows **Saved filter unavailable**, and exporting or publishing the dashboard fails, until you choose another filter or recreate the saved filter.
:::

## Choose a visualization

Dashboards offer 17 visualizations. A data set is one group of filters; the perspectives column lists where each visualization is available.

| Visualization | Use it to | Data sets | Perspectives |
| --- | --- | --- | --- |
| **Number** | Count objects, with the change over the period | 1 | All |
| **List** | List the latest objects, with the columns you choose | 1 | All |
| **List (distribution)** | Rank the top values of an attribute | 1 | All |
| **Donut**, **Radar**, **Polar Area**, **Word Cloud** | Show the share of each value of an attribute | 1 | All |
| **Tree** | Compare volumes as nested blocks | 1 | All |
| **Horizontal Bar** | Rank the top values, optionally broken down by a second attribute | 2 | All |
| **Line**, **Area**, **Vertical Bar** | Follow volumes over time and compare data sets | 5 | All |
| **Heatmap** | Spot when activity peaks, for several data sets | 5 | All |
| **Timeline** | Show objects in date order | 1 | Entities, Knowledge graph |
| **Map** | Place the targets of relationships on a world map | 1 | Knowledge graph |
| **Bookmark** | Show the entities the viewer bookmarked, among those the filters match | 1 | Entities |
| **Text** | Write a heading or an explanation in Markdown | None | None |

As a rule of thumb:

- for one kind of object, use **Number**, **List**, **List (distribution)**, **Donut**, **Radar**, **Map**, **Bookmark** or **Tree**;
- to compare up to five data sets, use **Line**, **Area**, **Vertical Bar** or **Heatmap**. Every data set must count the same thing: in **Knowledge graph**, add relationship data sets, not entity data sets;
- to split one data set into parts, use **Horizontal Bar** with a second data set, as in [Break down a chart](#break-down-a-chart).

## Filter the knowledge graph

A **Knowledge graph** widget counts relationships, and runs its filters in two stages:

1. **Dynamic source filters** and **dynamic target filters** first look for the entities at each end of the relationship, for example intrusion sets that target the finance sector, and attack patterns of the initial access phase. Each returns up to 5,000 entities.
2. **Relationship filters** then select the relationships between those entities, for example the **uses** relationships. Without source or target filters, they apply to every relationship of the platform.

:::warning
When a source or target filter matches more than 5,000 entities, the widget counts only the relationships of the first 5,000, and the result is wrong. Make these filters specific. The limit is the platform's maximum page size, which administrators set with `elasticsearch:max_pagination_result` (see the [Elasticsearch settings](/reference/opencti/configuration/#elasticsearch)).
:::

:::example[Rank the techniques used against the hospitality sector]
The CTI team wants the attack techniques of the intrusion sets that target the hospitality sector, ranked by how many of these intrusion sets use each one. Click **Create Widget** and enter:

- **Visualization**: **List (distribution)**
- **Perspective**: **Knowledge graph**
- **Dynamic source filters**: **Entity type** = Intrusion Set, and **In regards of** with **Relationship type** = targets and **Entity** = Hospitality
- **Dynamic target filters**: **Entity type** = Attack Pattern
- Relationship filters: **Relationship type** = uses
- **Parameters**: **Title** `Techniques against the hospitality sector`, **Attribute** **Entity**, **Display the source** off, **Number of results** 10, **Date attribute** `created (Functional date)`

The source filters find Velvet Harbor and the other intrusion sets that target the sector, the target filters keep only attack patterns, and the widget lists the techniques they use, such as Phishing and Valid Accounts, the most used first. Turning on **Display the source** would rank the intrusion sets instead.
:::

## Set the parameters

The parameters depend on the visualization:

| Parameter | What it does |
| --- | --- |
| **Attribute** | The attribute to count by, such as the entity, the author, a label or the malware type |
| **Display the source** | In **Knowledge graph**, show the entities at the source of the relationships when on, at the target when off |
| **Date attribute** | The date the dashboard period filters on. `created_at (Technical date)`, the default, is when the object reached the platform; `created (Functional date)` is when it was true in the world, such as the publication date of a report. Use the functional date for intelligence imported late, or old reports would count as new. Relationships also offer `start_time` and `stop_time`, entities `first_seen` and `last_seen`. When a data set has a **Label**, the field shows that label instead of **Date attribute** |
| **Interval** | For charts over time: **Day**, **Week**, **Month**, **Quarter** or **Year** |
| **Number of results** | How many values or objects to show: up to 500 for **List**, up to 100 otherwise |
| **Sort by**, **Sort mode** | The order of a list |
| **Stacked**, **Display legend** | Stack the data sets of a bar or area chart, and show their labels |
| **Distributed** | Give each block of a **Tree** its own color |
| **Center latitude**, **Center longitude**, **Zoom** | The initial view of a **Map** |
| **Customize columns** | The columns of a **List** |

:::only{opencti=">=7.260811"}
In **Activity & history**, **Number**, **Line** and **Area** widgets can count the **Distinct** values of a field, for example the number of different users who logged in. Above 40,000 distinct values, the count is an estimate.
:::

## Break down a chart

This example ranks the countries most targeted by malware, and splits each country's bar by malware type.

:::steps

1. Click **Create Widget**, choose **Horizontal Bar**, then the **Knowledge graph** perspective.
2. In the first data set, filter on **Source type** = Malware, **Target type** = Country and **Relationship type** = targets. Name it `Targeted countries`.
3. Add an entity data set with the **Entities** button, and filter on **Entity type** = Malware and **In regards of** = targets. Name it `Malware`.
4. In **Parameters**, enter the **Title** `Countries targeted by malware`, set the first data set's **Attribute** to **Entity** and turn off **Display the source**, so the bars show the targeted countries.
5. Set the second data set's **Attribute** to **Malware_types**, the option that holds the malware type (the list shows the attribute names of the entity type as they are stored), then click **Create**.

:::

To rank malware instead, with their top targets in each bar, add a relationship data set with the **Relationships** button in step 3, keep the same relationship filters, and turn on **Display the source** for the first data set.

## What's next

- [Build dashboards](/opencti/use/dashboards/): arrange widgets and set the period.
- [Share dashboards](/opencti/use/dashboards/share/): export a widget or publish the dashboard.
- [Add a dashboard tab to entity pages](/opencti/use/dashboards/custom-views/): the same widgets on every entity of a type.
