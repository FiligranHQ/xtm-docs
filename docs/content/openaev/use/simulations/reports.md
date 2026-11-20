---
title: Share simulation results in a report
description: Build a branded PDF or HTML report on a scenario, a simulation or the whole platform, generate it on demand, and email it on a schedule to people without access.
doc_type: how-to
products:
  - openaev
personas: [soc-manager, validation-engineer, exercise-planner]
owner: docs-openaev
last_reviewed: 2026-11-10
migrated_from:
  - openaev:/usage/simulation-reports/
---

A report turns the results of OpenAEV into a document for people who never open it: a CISO, an auditor, a board. You choose what it is about, a scenario, a simulation, an asset group or the whole platform, and the sections it holds; OpenAEV renders it in your colors as a PDF or an HTML page. Each generation is a snapshot kept with its date, and a schedule can generate it and email it every week or month. Dashboards stay the place to explore live data: see [Custom dashboards](/openaev/use/dashboards/).

::diagram{id=openaev-simulations-reports-flow}

## Before you start

- Your role has **Access reportings** to read and download reports, **Manage reportings** to create, generate and schedule them, and **Delete reportings** to delete one.
- To email a report, the platform can send email.

## Create a report

:::steps

1. Open :menu[Reporting] and click **Create report**. The **Create a report** panel opens on its first step, **Report subject**.
2. Give a **Name** and a **Description**, then choose the **Subject type** and the subject itself:

   | **Subject type** | What the report covers |
   | --- | --- |
   | **Platform** | Every result of the tenant. The subject reads the tenant's **Platform name**, set in [Parameters](/openaev/administer/parameters/), or the platform's when the tenant sets none |
   | **Scenario** | Every simulation of the scenario |
   | **Simulation** | One simulation |
   | **Atomic testing** | One atomic test |
   | **Endpoint**, **Asset group** | What ran on these assets |
   | **Person**, **Team** | The results of these people |

3. Choose the **Time range**, from **Last 7 days** to **All time**, **Last 30 days** by default, and the **Default format**, **PDF** or **HTML**. Click **Next**.
4. On **Modules**, select the sections and drag them in order. Each takes a **Custom title (optional)**. Every module but **Attack paths** and **Custom content** is selected by default:

   | Module | What it shows |
   | --- | --- |
   | **Cover** | The report name, the subject and the generation date |
   | **Executive summary** | The posture in a few key figures |
   | **Subject details** | The scenario, simulation or asset the report is about |
   | **Kill chain coverage** | The techniques exercised, on the kill chains you choose in **Kill chains**, all by default |
   | **Results breakdown** | Successes and failures per expectation type |
   | **Performance by security domain** | The success rate per security domain, such as endpoint or email |
   | **Score trends** | The scores over the time range |
   | **Failed expectations** | The 15 latest failed expectations |
   | **Findings** | The findings per type, then the 15 latest. Empty for an asset group, a person or a team |
   | **Attack paths** | The kill chain phases the injects went through |
   | **Custom content** | Your own text, in Markdown |

   ::screenshot{id=openaev-reports-modules alt="The Modules step of Create a report: every module checked but Attack paths and Custom content, each with its description, and below them the Report structure with a Custom title (optional) for each section."}

5. Click **Next**. On **Branding**, choose the **Theme**, the six colors and the **Logo (defaults to the platform logo)**. **Reset to platform defaults** goes back to the platform's theme.
6. Click **Next**. On **Schedule**, turn on **Generate this report on a schedule** to add a first schedule now, or leave it off.
7. Click **Create**.

:::

The report opens on its **Preview** tab: the report as it will print, page by page, with the data of today. It follows every change of the report.

::screenshot{id=openaev-reports-preview alt="The Preview tab of the report Monthly validation posture, on the Platform, in PDF over the last 30 days: its cover in the colors of Grand Meridian, then its table of contents, Executive summary, Kill chain coverage, Results breakdown, Failed expectations and Findings."}

## Generate it

- **Generate now**, in the header, generates it in its default format; the arrow next to it, **Choose format**, offers the other one. The generation runs in the background, then **Download latest generation** downloads it.
- The **Generations** tab lists every generation with its status, format, **Trigger**, **Manual** or scheduled, and dates. Download any of them: each one keeps the data of the moment it was generated.
::screenshot{id=openaev-reports-generations alt="The Generations tab of the report: one generation, Success, in PDF, triggered Manual, created and completed on April 12, 2026, with the download button in the header."}

- On a scenario, a simulation, an atomic test, an endpoint, an asset group, a person or a team, the **Reports** button of the header lists the reports on it. **Generate report** creates one at once, named after the entity, in PDF over the last 30 days, when none exists yet.

## Send it on a schedule

On the **Schedules** tab, click **Add schedule**:

| Field | Effect |
| --- | --- |
| **Period** | **Every hour**, **Every day**, **Every week** (the default) or **Every month** |
| **Day of week**, **Day of month** | When a weekly or monthly schedule runs. A day beyond the end of a short month runs on its last day |
| **Time of day** | When it runs, in UTC, 08:00 by default. An hourly schedule runs at the start of each hour |
| **Format** | **PDF** or **HTML** |
| **Recipient users**, **Recipient emails** | Users of the platform and other addresses that receive the file by email |
| **Enabled** | Turns the schedule off without deleting it |

::screenshot{id=openaev-reports-schedules alt="The Schedules tab of the report: the schedule Monthly, Every month at 1 07:00, in PDF to 1 email, enabled, with the Add schedule button."}

Each run is a generation of its own, listed on **Generations**, and the recipients receive the file attached to an email titled `Scheduled report:` followed by the report's name. Recipients outside the platform read everything the report shows: check the subject and the modules before you add them.

:::example[Send the CISO the Velvet Harbor posture every month]
Hélène Marchetti, CISO of Grand Meridian Hotels, wants each month the posture of the chain against Velvet Harbor, without opening OpenAEV. The validation engineer creates:

| Step | Value |
| --- | --- |
| **Name**, **Description** | `Monthly validation posture`, `The posture of Grand Meridian Hotels against Velvet Harbor, for the CISO.` |
| **Subject type** | **Platform** |
| **Time range**, **Default format** | **Last 30 days**, **PDF** |
| **Modules** | **Cover**, **Executive summary**, **Kill chain coverage**, **Results breakdown**, **Failed expectations**, **Findings**, and **Custom content** titled `What we are fixing`: the upgrade of Apache on `booking-web-01`, the new password of the `booking_app` account, and the prevention policy of Meridian EDR extended to the web front |
| **Branding** | **Theme** light, the primary color of Grand Meridian, `#8C1D40`, and its secondary color, `#1E88E5` |
| Schedule | **Every month**, **Day of month** 1, **Time of day** 07:00, **PDF**, **Recipient emails** `helene.marchetti@grand-meridian.example` |

On 12 April 2026, the engineer clicks **Generate now** to check the PDF before the first send. From then on, Hélène receives it on the first of each month: the booking database export that Meridian EDR blocked, the discovery on the web front that it only detected, the CVE and the database account that discovery found, then what the teams are fixing.
:::

:::xtm-one[Ask XTM One for a readout]
With XTM One connected and the Enterprise Edition, **Ask Ariane** can also write a readout, a slide deck or a video from your OpenAEV data, in conversation, for a review meeting rather than a recurring report. You download it as PDF, PowerPoint or MP4 from its card. See [XTM One in OpenAEV](/xtm-one/use/in-openaev/).
:::

## What's next

- [Run a simulation and read its results](/openaev/use/simulations/).
- [Custom dashboards](/openaev/use/dashboards/), to explore the results live.
- [Findings](/openaev/use/findings/), for every finding the report summarizes.
