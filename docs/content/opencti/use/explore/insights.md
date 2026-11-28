---
title: Read the home dashboard and AI summaries
description: Read the OpenCTI home dashboard, choose which dashboard opens at login, and get AI summaries of the activity of a threat or of a report.
sidebar_position: 3
doc_type: how-to
products:
  - opencti
  - xtm-one
personas: [cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/insights/
---

Two features tell you what changed without opening every object. The home dashboard shows, every morning, the threats, victims, malware and vulnerabilities with the most new knowledge over the last three months, and the latest reports. **AI Insights** writes a short summary of one threat, victim or report, ready for a briefing: whether its activity is increasing, a digest of the reports about it, a forecast and how the knowledge about it changed. This page shows how to read both, and how to make the home page open on the dashboard your team needs.

## Before you start

- The home dashboard needs the **Access knowledge** capability. Without it, the home page shows the message an administrator sets in the [platform parameters](/opencti/configure/parameters/).
- Choosing another dashboard as your home page needs the **Access dashboards** capability.
- **AI Insights** needs an Enterprise Edition license and generative AI enabled on the platform, through the platform AI settings or through XTM One. Without a license, the button shows an Enterprise Edition badge.

## Read the home dashboard

Click **Home** in the left navigation. Unless you chose another dashboard, you see the platform dashboard:

| Widget | What it shows |
| --- | --- |
| Four counters | The number of intrusion sets, malware, reports and indicators |
| **Most active threats (Last 3 months)** | The threat actors, intrusion sets and campaigns with the most new relationships |
| **Most targeted victims (Last 3 months)** | The organizations, sectors and locations most often targeted |
| **Relationships created** | How many relationships were created, month by month |
| **Most active malware (Last 3 months)** | The malware with the most new relationships |
| **Most active vulnerabilities (Last 3 months)** | The vulnerabilities with the most new relationships |
| **Targeted countries (Last 3 months)** | A map of targeting by country |
| **Latest reports** | The last reports, each one a link to its page |
| **Most active labels (Last 3 months)** | The labels used the most |

::screenshot{id=opencti-insights-home-dashboard alt="The platform dashboard: four counters at the top, then the most active threats, the most targeted victims, the relationships created over time, a map of targeted countries and the latest reports."}

Every widget only counts what you can access, so two people can see different numbers.

:::only{opencti=">=7.261127"}
**Targeted countries (Last 3 months)** shades each country on a logarithmic scale of its number of `targets` relationships: a country targeted a hundred times no longer flattens all the others into one shade.
:::

## Choose your home dashboard

::::steps

1. Open **Profile** in the top bar, then **Profile** in its menu.
2. In **Dashboard settings**, pick a **Custom dashboard**:
   - **Default dashboard** opens the first dashboard set for your groups or organization, or the platform dashboard when none is set;
   - **Platform dashboard** always opens the built-in dashboard above;
   - any dashboard listed under **Dashboards from your groups & organizations** or **Other custom dashboards** opens that one.

   :::only{opencti=">=7.261119"}
   When you pick a custom dashboard, **Default preset** shows under it. Choose one of the presets saved on that dashboard: your home page then opens on its variable values and time range. Left on **No preset (default values of the dashboard)**, it opens on the values the dashboard itself sets. Choosing another dashboard clears the preset.

   ::screenshot{id=opencti-insights-default-preset alt="The Dashboard settings of the profile: Technical date as the Date reference, Member hotel overview as the Custom dashboard and, under it, Last month in Default preset, whose helper text says the dashboard opens on the values and time range of this preset."}
   :::

3. Pick a **Date reference** for the counters: **Technical date** counts objects by when they were created in the platform, **Functional date** by the dates the knowledge itself carries.
4. Click **Home** to check the result.

::::

:::example[Count threats by when they were active, not when they were imported]
After a connector imported two years of history in one night, the home counters jump for the current month. In **Dashboard settings**, keep **Custom dashboard** on **Platform dashboard** and set **Date reference** to **Functional date**. The widgets now count knowledge by the dates it carries, such as the publication date of a report, so the reports on Velvet Harbor spread from October 2025 to January 2026 instead of piling up on the day of the import.
:::

::::only{opencti=">=7.261119"}
:::example[Open the member dashboard on last month's activity]
Every Monday you brief the member hotels from **Member hotel overview**. Its time range is the last three months, and its editor saved a **Last month** preset. In **Dashboard settings**, set **Custom dashboard** to **Member hotel overview** and **Default preset** to **Last month**. Click **Home**: the dashboard opens on the last month, ready for the briefing, and the dashboard itself keeps its three months.
:::
::::

Administrators set the default dashboards of a group or an organization in their settings. See [Organize users in groups](/opencti/administer/groups/). To build a dashboard, see [Dashboards](/opencti/use/dashboards/).

:::only{opencti=">=7.261119"}
They can also set the preset that dashboard opens on: with **Default dashboard**, your home page opens on the preset your organization, or else your group, set with it. Your own choice of a dashboard, with or without a preset, comes first. See [Separate data between organizations](/opencti/administer/organizations/).
:::

## Get an AI summary of an entity or a report

**AI Insights** sits next to the tabs of an entity page, on **Overview**. It is available on:

| Objects | Tabs of the summary |
| --- | --- |
| Threat actors (group and individual), intrusion sets, campaigns, malware, incidents, sectors, regions, countries | **Activity**, **Containers digest**, **Forecast**, **Internal history** |
| Reports, groupings, incident responses, requests for information, requests for takedown | **Container summary** |

:::steps

1. Open the entity or the report, then click **AI Insights**. A panel opens on the first tab.
2. Read the tab you need:
   - **Activity** rates the recent activity of the threat as **Increasing**, **Stable** or **Decreasing**, from the evolution of its indicators, victims and other relationships, and explains why;
   - **Containers digest** summarizes the reports and incident responses that contain the entity. Narrow it with **Relative time** and **Limit of elements**;
   - **Forecast** suggests how the activity may evolve;
   - **Internal history** summarizes how the knowledge about the entity changed in the platform;
   - **Container summary** summarizes the description, content and files of the report or case.
3. Use **Copy to clipboard** to reuse the text, for example in the **Content** tab.

:::

:::example[Brief management on Velvet Harbor in two minutes]
Your manager wants to know what happened with Velvet Harbor last month. Open :menu[Threats > Intrusion sets], then **Velvet Harbor**, and click **AI Insights**. On **Containers digest**, set **Relative time** to **Last month** and **Limit of elements** to **10**.

The digest summarizes the reports and incident responses of the last month that contain Velvet Harbor, such as **Velvet Harbor targets hotel booking systems** and the booking system intrusion at a hotel chain. **Activity** then says whether the threat is increasing, stable or decreasing. Copy both texts with **Copy to clipboard**, and check each claim against the reports before you send them.
:::

A summary is generated by AI and can contain mistakes: check every claim against the reports before you share it. To spare the model, the platform reuses a summary for 60 minutes by default; **Retry** generates a new one now. Your administrator can change that duration with `ai:insights_refresh_timeout` (see the [configuration reference](/reference/opencti/configuration/#ai-service)).

::::only{opencti=">=7.260811"}
:::xtm-one[AI Insights with XTM One]
When XTM One is connected, each tab of **AI Insights** is written by an XTM One agent. **Select agent** lets you pick which agent writes it, and the panel says when the answer was generated. An agent's answer is shared by every user who opens the entity and is reused for 24 hours by default (`ai:agents_refresh_timeout`); **Retry** asks again. If no agent handles the task, ask your administrator to configure one. See [XTM One in OpenCTI](/xtm-one/use/in-opencti/).
:::
::::

:::xtm-one[Ask questions in a chat]
**Ask Ariane**, in the top bar, answers questions about the knowledge of your platform in a chat, for example about a threat, a sector or the latest reports. It shows when an administrator has enabled the AI chat. See [XTM One in OpenCTI](/xtm-one/use/in-opencti/).
:::

## What's next

- [Refine content with AI](/opencti/use/enrich/refine-content/) to write and reword descriptions and reports with AI.
- [Dashboards](/opencti/use/dashboards/) to build the dashboard your team opens every day.
- [Explore threats](/opencti/use/explore/threats/) to read the knowledge behind a summary.
