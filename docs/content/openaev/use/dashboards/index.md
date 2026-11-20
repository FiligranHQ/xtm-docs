---
title: Build dashboards of your validation results
description: Create a custom dashboard of your OpenAEV results, scope it to a scenario or a simulation, filter it by period, and make it the dashboard of Home or of the Statistics tab.
doc_type: how-to
products:
  - openaev
personas: [validation-engineer, soc-manager]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/dashboards/custom-dashboards/custom-dashboards/
---

The home dashboard shows your whole platform. A custom dashboard shows what matters to one audience: the detection rate of the SOC against one threat, the trend of a scenario run every week, a summary for the CISO. You build it from widgets, give it parameters so that it can follow any scenario or simulation, and choose where it opens: on :menu[Home], or in the **Statistics** tab of every scenario and simulation. This page creates the dashboard and decides where it shows; the widgets themselves are in [Create the widgets of a dashboard](/openaev/use/dashboards/widgets/).

::diagram{id=openaev-dashboards-defaults}

## Before you start

- Your role has **Access dashboards** to see :menu[Dashboards], **Manage dashboards** to create dashboards, change them and add widgets, and **Delete dashboards** to delete them.
- To choose the dashboards every user gets by default, your role also needs **Manage tenant settings**.

## Create a dashboard

:::steps
1. Open :menu[Dashboards] and click **Create**. The **Create a custom dashboard** panel opens on its **General** tab.
2. Enter the **Name** and a **Description** that says who the dashboard is for.
3. To make the dashboard follow a scenario or a simulation, open the **Parameters** tab, click **Add**, and choose the **Parameter Type**: **scenario** or **simulation**. Give it a **Parameter Name**, the label the selector will show above the widgets.
4. Click **Create**. The dashboard opens, empty.
5. Click **Create** in the dashboard's header to add its first widget: see [Create the widgets of a dashboard](/openaev/use/dashboards/widgets/).
:::

::screenshot{id=openaev-dashboards-create-parameters alt="The Parameters tab of the Create a custom dashboard panel, with one parameter named Weekly run of the type simulation, and the Cancel and Create buttons."}

Drag a widget to move it, and drag its bottom right corner to resize it. The layout is saved as you go, for every user of the dashboard. Only the dashboard's own page can be arranged: on :menu[Home] and in the **Statistics** tab, it is read-only.

## Filter by period and by scenario

The top of a dashboard holds its filters:

- **Time range** applies to every widget that follows the dashboard: **Last 3 months** by default, from **Last 24 hours** to **All time**. **Custom range** adds a **Start date** and an **End date**.
- Each parameter adds a selector, named after it, with the scenarios or simulations to choose from. The widgets that use the parameter are computed for the one you pick.

A widget can also keep its own period instead of the dashboard's: see [Create the widgets of a dashboard](/openaev/use/dashboards/widgets/).

Click a bar, a slice, a cell or a number to open the list of what it counts, such as the expectations, findings or simulations behind it.

## Read the resilience gauges

A **Resilience Gauge** widget, such as the **Prevention**, **Detection**, **Vulnerability** and **Human Response** gauges of the **Platform default** home dashboard, sorts the expectations of its series by result. Its ring shows one arc per result: **Success** in green, **Failed** in red, **Pending** in grey, and **Partial** in an arc of its own. The legend under the ring lists **Partial** and **Pending** only when there are some.

- The number in the middle, **Resilience**, is the share of successes among every expectation of the gauge, pending ones included. It matches the green arc.
- Its color goes from red below 25 to green from 75. While no expectation has a result yet, every one still pending, the gauge stays grey instead of red.
- Click the ring to list every expectation behind it, or a result of the legend to list only those.

For example, after a weekly run of `Velvet Harbor replay`, a **Detection** gauge with 3 **Success**, 1 **Failed** and 2 **Pending** reads 50%. The **Results** gauges of a simulation compute the same number, so the home dashboard and the simulation agree: see [Read the results](/openaev/use/simulations/#read-the-results).

## Choose where a dashboard opens

| Where | Which dashboard it shows | Who chooses |
| --- | --- | --- |
| :menu[Home] | The user's own **Home dashboard**, else the tenant's **Home dashboard**, else the **Platform default** dashboard | Each user in their profile; an administrator for the tenant |
| **Statistics** tab of a scenario | The scenario's own dashboard, else the tenant's **Default scenario dashboard** | Anyone who can change the scenario; an administrator for the tenant |
| **Statistics** tab of a simulation | The simulation's own dashboard, else the tenant's **Default simulation dashboard** | Anyone who can change the simulation; an administrator for the tenant |

::screenshot{id=openaev-dashboards-home alt="The home dashboard of the tenant: the exposure command center with a score of 40 At risk, 1 prevention and 2 detections, the Prevention, Detection, Vulnerability and Human response gauges, and the performance by security domain, 75% for Endpoint."}

- The tenant's three defaults are in :menu[Settings > Parameters]: see [Set the name, look and home page of a tenant](/openaev/administer/parameters/).
- Your own home dashboard is in your profile: see [Set up your profile](/openaev/use/profile/).
- To give one scenario or simulation its own dashboard, open its **Statistics** tab, click **Change dashboard** (or **Select a dashboard** when none shows), choose the **Dashboard**, and click **Continue**.

A simulation created from a scenario starts with the tenant's **Default simulation dashboard**. In the **Statistics** tab, a **Scenario** or **Simulation** parameter takes the current scenario or simulation by itself: one dashboard serves every scenario.

## Share and manage dashboards

:menu[Dashboards] lists every dashboard of the tenant, as cards or as a list. The starter pack, which OpenAEV loads into each new tenant, brings six of them: **Technical home**, **Technical scenario** and **Technical simulation** for technical validation, and their **Tabletop Exercice** counterparts for table-top exercises.

::screenshot{id=openaev-dashboards-list alt="The Custom dashboards of the tenant as cards: Tabletop Exercice home, scenario and simulation, and Technical home, scenario and simulation, with the Import a custom dashboard and Create buttons."}

The menu of a dashboard holds:

- **Update**: its name, description and parameters.
- **Export**: a ZIP file of the dashboard and its widgets, to load on another platform with **Import a custom dashboard** at the top of the list.
- **Delete**: not offered for the dashboard the tenant uses as its **Home dashboard**. Choose another home dashboard first.

:::example[A dashboard of the Velvet Harbor replay for the SOC]
Grand Meridian Hotels runs `Velvet Harbor replay` every Wednesday. The SOC manager wants to follow what Meridian EDR misses, run after run. The validation engineer creates the dashboard:

| Field | Value |
| --- | --- |
| **Name** | `Velvet Harbor replay` |
| **Description** | Detection and prevention of each weekly run, for the SOC |
| Parameter | **Parameter Type** **simulation**, **Parameter Name** `Weekly run` |

Then three widgets: a **Security Coverage** matrix of the detection expectations, a **Posture Radar** by security platform, and a **Line** of failed expectations over time. In :menu[Settings > Parameters], the administrator makes it the **Default simulation dashboard**. From then on, the **Statistics** tab of each new simulation opens on it, scoped to that run: on 21 January 2026, the SOC sees that the collection of guest records on the booking database server, Data from Local System (T1005), is still not detected.
:::

:::xtm-one[Turn your results into a report]
When OpenAEV is registered with XTM One, **Ask Ariane** in the top bar can turn your results into a report or a deck for a review meeting, which you download as PDF or PowerPoint. See [XTM One in OpenAEV](/xtm-one/use/in-openaev/).
:::

## What's next

- [Create the widgets of a dashboard](/openaev/use/dashboards/widgets/): choose what each widget shows.
- [How a scenario becomes a simulation](/openaev/concepts/scenarios-simulations/): what the **Statistics** tab compares.
- [How you validate your defenses in OpenAEV](/openaev/concepts/validation/): the home dashboard and its widgets.
