---
title: Create the widgets of a dashboard
description: Add a widget to an OpenAEV dashboard in three steps, its visualization, the data it counts and how it breaks them down, and make it follow the dashboard's period and parameters.
doc_type: how-to
products:
  - openaev
personas: [validation-engineer, soc-manager]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/dashboards/widgets/widgets/
---

A widget answers one question on a dashboard: how many detection expectations failed this week, which techniques of the threat get through, which security platform misses the most. You build it in three steps: the **Visualization**, which decides how it looks; the **Dimensions**, which decide what it counts; and the **Parameters**, which decide how it breaks the data down and over which period. This page goes through the three steps and the choices of each. Creating the dashboard itself is in [Build dashboards of your validation results](/openaev/use/dashboards/).

::diagram{id=openaev-widgets-steps}

## Before you start

- Your role has **Manage dashboards**.
- The dashboard exists. If the widget must follow a scenario or a simulation, the dashboard has a **Scenario** or **Simulation** parameter.

## Add a widget

:::steps
1. Open the dashboard and click **Create** in its header. The widget dialog opens on **Visualization**.
2. Click the visualization that answers your question (see the table below). The dialog moves on to **Dimensions**.
3. Choose the **Entity type** the widget counts, give the series a **Label (entities)**, and add the filters that narrow it. A visualization with several series, such as a bar chart comparing detected and not detected, takes one series per **Add**. Click **Validate**.
4. On **Parameters**, enter the **Title**, then the breakdown, the period and the options of the visualization (see below).
5. Click **Create**. The widget lands at the end of the dashboard, where you can move and resize it.
:::

::screenshot{id=openaev-widgets-visualization alt="The first step of the widget dialog, Visualization, with its thirteen choices from Command Center to Number, before the Dimensions and Parameters steps."}

To change or remove a widget, open its menu and choose **Update** or **Delete**.

## Choose the visualization

| Visualization | What it shows | Series |
| --- | --- | --- |
| **Command Center** | The adversarial exposure score, with the validations your controls stopped and those that got through, around your security platforms | Up to 2 |
| **Exposure Score** | The adversarial exposure score alone, from 0 to 100: the share of validations not stopped. A higher score is worse: **Low exposure** up to 24, **Moderate exposure** up to 49, **High exposure** up to 74, then **Critical exposure** | Up to 2 |
| **Resilience Gauge** | The share of successful expectations of one series, such as every prevention expectation | 1 |
| **Posture Radar** | Successes and failures across your security platforms | Up to 5 |
| **Security Domains** | The success rate of each expectation type in each security domain. See [How security domains group your results](/openaev/concepts/security-domains/) | 1 |
| **Security Coverage** | The MITRE ATT&CK matrix, each technique colored by the expectations it passed and failed | Up to 2 |
| **Attack Paths** | The techniques of the attacks played, linked in the order they ran | Up to 2 |
| **Vertical Bar**, **Horizontal Bar** | Counts broken down by a property, or by period for a vertical bar | Up to 5, up to 2 |
| **Line** | Counts over time | Up to 5 |
| **Donut** | The share of each value of a property | 1 |
| **List** | The entities themselves, in columns you choose | 1 |
| **Number** | One count, with its change over the period | 1 |

## Choose what it counts

On **Dimensions**, the **Entity type** decides what the widget counts and the properties it can break down by:

| **Entity type** | Counts | Breaks down by |
| --- | --- | --- |
| **Inject expectations** | The results of injects: the most common choice to measure your defenses | Status, type, scenario, simulation, asset, asset group, security platform |
| **Finding** | What injects discovered: CVEs, open ports, credentials | Type, endpoint, scenario, simulation |
| **asset** | Your assets | Category, platform, architecture, hostname, IP addresses |
| **Vulnerable endpoint** | The endpoints where a vulnerability was found | Platform, architecture, agent status and privileges |
| **Inject** | The injects played | Status, tags, assets, asset groups, teams, platforms, scenario, simulation |
| **Simulation**, **Scenario** | Your runs and your scenarios | Status, tags, assets, asset groups, teams, platforms |

The filters of a series take fixed values, such as the type **Detection** and the status **Failed**. A filter on the scenario or the simulation can also take a dashboard parameter, listed under **Parameters**: the widget then follows the selector at the top of the dashboard, and the **Statistics** tab of each scenario or simulation.

Widgets show sensitive finding values masked: a credential found by an inject shows its user name and the first characters of its secret only, such as `night.audit:Me******`. A filter on a finding value matches that masked form, never the hidden part.

## Set the parameters

**Parameters** adapts to the visualization:

- **Title**: what the dashboard shows above the widget. Left empty, seven visualizations show a title of their own: **Performance by security domain** for **Security Domains**, **Exposure command center** for **Command Center**, **Resilience** for **Resilience Gauge**, **Exposure score**, **Posture radar**, **Security Coverage** and **Attack Path** for the others of the table's first seven. The charts, lists and numbers show none.
- **Mode**, for the charts that offer both: **structural** breaks the data down by a property, chosen in **Breakdown by**; **temporal** spreads it over time, by the **Date attribute** you choose and an **Interval** of a **Day**, a **Week**, a **Month**, a **Quarter** or a **Year**.
- **Number of results**: how many bars, slices or values to show, from 1 to 100. A list shows up to 1,000 lines.
- **Time range**: **Dashboard time range** by default, so the widget follows the period chosen at the top of the dashboard. Choose a fixed period, such as **Last 6 months**, for a widget that must keep its own, or **Custom range** with a **Start date** and an **End date**.
- For a **List**: **Customize columns** to pick and order its columns, then the **Sort field** and its **Direction**.

:::example[Follow what Meridian EDR misses, week after week]
On the dashboard `Velvet Harbor replay`, which has the **Simulation** parameter `Weekly run`, the SOC manager of Grand Meridian Hotels adds two widgets:

| Step | **Security Coverage** widget | **Line** widget |
| --- | --- | --- |
| **Entity type** | **Inject expectations** | **Inject expectations** |
| Series | `Detected`: type **Detection**, status **Success**, simulation `Weekly run`; `Not detected`: type **Detection**, status **Failed**, simulation `Weekly run` | `Not detected`: type **Detection**, status **Failed**, scenario `Velvet Harbor replay` |
| **Title** | `Velvet Harbor techniques, this run` | `Missed detections per week` |
| **Mode** | | **temporal**, **Interval** **Week** |
| **Time range** | **Dashboard time range** | **Last 6 months** |

The matrix follows the run chosen in `Weekly run`, and shows Data from Local System (T1005) in red on 21 January 2026. The line keeps six months whatever the run, and shows the misses dropping after the SOC's new rules of February.
:::

## What's next

- [Build dashboards of your validation results](/openaev/use/dashboards/): where the dashboard opens, and its parameters.
- [How an attack becomes injects and expectations](/openaev/concepts/injects-expectations/): what a successful or failed expectation means.
- [How security domains group your results](/openaev/concepts/security-domains/): read the **Security Domains** widget.
