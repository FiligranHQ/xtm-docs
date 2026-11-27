---
title: Add a dashboard tab to entity pages
description: Add a tab of widgets to every entity of a type in OpenCTI, scoped to the entity being viewed, and share custom views between platforms.
sidebar_position: 4
doc_type: how-to
products:
  - opencti
personas: [cti-lead, admin, cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/custom-views/
---

A custom view is a dashboard attached to an entity type: it appears as a tab on the page of every entity of that type, and its widgets show data about the entity being viewed. For example, a custom view on **Vulnerability** can show the campaigns and malware that exploit each vulnerability and a timeline of related events, right on the vulnerability's page. This page shows how to create a custom view, scope its widgets, make it the landing tab, and move views between platforms.

:::only{opencti="<7.260811"}
Custom views arrived in OpenCTI 7.260811. In this version, build a [custom dashboard](/opencti/use/dashboards/) with filters on the entity instead.
:::

::::only{opencti=">=7.260811"}

## How custom views appear

- When an entity type has one enabled custom view, its entities show a tab named after the view.
- When it has several, they are grouped under one **Custom view** tab.
- A **Default** view replaces the overview as the first tab, the one users land on when they open an entity of that type. Each type has at most one default view.

Each viewer sees the data they are allowed to see, as in any dashboard.

## Before you start

- Your role has the **Manage customization** capability.
- Custom views exist for most domain object types, such as reports, threat actors, malware, vulnerabilities and incidents.

## Create a custom view

:::steps

1. Open :menu[Settings > Customization > Entity types], choose the entity type, then the **Custom Views** tab. The list shows each view's name, description and status, with the default view marked.
2. Click **Create a new custom view**.
3. Enter a **Name** of at least two characters, which becomes the tab label, and optionally a **Description** for other administrators.
4. Turn on **Set as default custom view** if the view should replace the overview as the landing tab. Then click **Create**.

   ::screenshot{id=opencti-custom-views-create alt="The Create custom view panel for the Vulnerability type, with the Name and Description fields and the Set as default custom view switch."}

5. The view opens in edition mode. Add widgets with **Create Widget**, as in a dashboard. See [Create widgets](/opencti/use/dashboards/widgets/).

:::

Custom views offer the same visualizations as dashboards, plus **Attributes**, which shows chosen attributes of the entity itself.

## Scope widgets to the current entity

A new widget in a custom view comes with a filter on the entity being viewed, so it is relevant on every page:

| Entity type | Perspective | Filter added |
| --- | --- | --- |
| Containers, such as reports and cases | Any | **Contains** the current entity |
| Other types | **Entities** | **In regards of** the current entity |
| Other types | **Knowledge graph** | The current entity is the source or the target of the relationship |
| Other types | **Activity & history** | **Related entity** is the current entity |

Add more filters to narrow each widget, or replace the filter with another one on the current entity, for example **Source entity** to count only relationships that start from it.

To check the result while you edit, choose an entity in the selector at the top of the editor. Widgets then render with that entity's data, and those that depend on the current entity show **Preview data**.

:::only{opencti=">=7.261117"}
A **Vertical Bar**, **Line**, **Area** or **Heatmap** widget of a custom view can also draw one series per value of a field, as in [Follow each value of a field over time](/opencti/use/dashboards/widgets/#follow-each-value-of-a-field-over-time). With the filter on the current entity, the series count only that entity's data: on an intrusion set, the reports about it, one series per label.
:::

:::example[Show the targets and tools of each intrusion set]
Analysts want, on the page of every intrusion set, where it strikes and what it uses. In :menu[Settings > Customization > Entity types], choose **Intrusion Set**, then **Custom Views**, and create a view with the **Name** `Targeting`, the **Description** `Countries, malware and indicators of the intrusion set.` and **Set as default custom view** off. Add three widgets, each keeping the filter on the current entity that OpenCTI adds:

| Title | Visualization | Perspective | Filters you add |
| --- | --- | --- | --- |
| `Targeted countries` | **Map** | **Knowledge graph** | **Relationship type** = targets, **Target type** = Country |
| `Malware used` | **Number** | **Knowledge graph** | **Relationship type** = uses, **Target type** = Malware |
| `Indicators` | **List** | **Entities** | **Entity type** = Indicator |

Choose Velvet Harbor in the preview selector: the map shows France and Spain, the number counts its three malware, and the list shows the indicators that point to it, such as `booking-velvetharbor.example`. Enable the view: every intrusion set now has a **Targeting** tab with its own data.
:::

## Enable, set as default, duplicate or delete

A view is visible to users only while it is enabled. Use the enable button in the editor header, or the menu on its line in the list.

- **Enable** or **Disable** a view to show or hide it without deleting it.
- **Set as default** makes the view the landing tab. If another view is already the default, OpenCTI asks you to replace it. **Remove default** brings the overview back first.
- **Duplicate the custom view**, in the editor's menu, copies it with its widgets under a **New name**, which starts as the name followed by `- copy`. The copy is disabled, so you can change it before users see it.
- **Delete** removes the view and its widgets. This cannot be undone.

## Export and import

Move a custom view to another platform, or keep a copy, with a JSON file.

- **Export** a view from the export button in its editor. The file holds its name and all its widgets. To export one widget, use **Export** in the widget's menu.
- **Import a custom view** from the **Custom Views** tab of the entity type, and choose the file. The view arrives disabled and not default: review it, then enable it.
- **Import Widget** in the editor adds a widget exported from a view or a dashboard.

:::only{opencti=">=7.261030"}
**Import from Hub** in the **Custom Views** tab browses the ready-made views of the XTM Hub. The button only shows when your platform can reach the XTM Hub.
:::

::::

## What's next

- [Create widgets](/opencti/use/dashboards/widgets/): visualizations, perspectives and filters.
- [Build dashboards](/opencti/use/dashboards/): dashboards that are not tied to an entity.
- [Customize entity types](/opencti/configure/entities/): the other settings of an entity type.
