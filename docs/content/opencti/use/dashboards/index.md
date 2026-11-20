---
title: Build dashboards
description: Build an OpenCTI dashboard that answers one standing question, such as the threats targeting your sector this quarter, and share it with your team and leadership.
sidebar_position: 1
doc_type: how-to
products:
  - opencti
personas: [cti-lead, executive, cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/dashboards/
---

A custom dashboard answers a standing question on one screen, for example which threats target the hospitality sector this quarter, how much intelligence came in about them, and whether the activity is growing. Every widget is computed from the knowledge base each time the dashboard opens, so it stays current without anyone updating it. Each reader sees only the data their markings and organizations allow. A team lead builds the dashboard and shares it; an executive opens it with the **can view** access level, or through a [public link](/opencti/use/dashboards/share/) without an account.

## Start from the question

Write down what the reader must learn in ten seconds, then give each answer its own widget. For a CISO who asks "what threatens our sector this quarter?":

| The reader wants to know | Widget |
| --- | --- |
| How much intelligence came in on the sector | A **Number** of reports with the sector's label |
| Who targets the sector | A **List (distribution)** of the sources of **targets** relationships to the sector |
| Where these threats strike | A **Map** of the countries they target |
| Whether the activity is growing | A **Line** of new indicators per week |
| What to read first | A **List** of the latest reports |

Five to eight widgets stay readable on a laptop and on a wall screen. Put the most important answer at the top left, where the eye starts.

:::only{opencti=">=7.261117"}
When the question is about each value of a field, such as "how many reports of each type do we publish every month?", one chart answers it: break it down by that field, as in [Follow each value of a field over time](/opencti/use/dashboards/widgets/#follow-each-value-of-a-field-over-time). A new value gets its own series without any change to the dashboard.
:::

## Before you start

- Your role has **Access dashboards** to open dashboards, **Create / Update dashboards** to create and change them, and **Delete dashboards** to delete them. **Manage public dashboards** lets you publish one to people without an account.
- The knowledge holds the answer. A dashboard only counts what analysts have modeled: if reports on the sector carry no label and threats have no **targets** relationship to the sector, the widgets stay empty.

## Create the dashboard

:::steps

1. Open :menu[Dashboards > Custom dashboards]. The list shows the dashboards you created and those shared with you; **Publicly shared** says which ones have an enabled public link.
2. Click **Create Dashboard**, enter a **Name** of at least two characters and, optionally, a **Description**, then click **Create**. The empty dashboard opens.
3. Set the period in **Relative time**, as described in [Set the period](#set-the-period).
4. Click **Create Widget** and build the first widget. [Create widgets](/opencti/use/dashboards/widgets/) describes each step and setting.
5. Arrange the widgets: drag a widget to move it, and drag its bottom-right corner to resize it. The layout saves as you go.

   ::screenshot{id=opencti-dashboards-overview alt="The Hotel sector campaign dashboard with Relative time set to Last 6 months: a count of hospitality reports, the threats that target the sector, a line of new indicators per week, a map of the targeted countries and the latest reports."}

:::

::::example[Threats targeting the hospitality sector over six months]
The head of Example CERT wants the CISOs of member hotel chains to see, on one screen, what threatens the sector over the last six months. Create the dashboard with the **Name** `Hotel sector campaign` and the **Description** `Follow the campaign against the hospitality sector.`, set **Relative time** to **Last 6 months**, then add five widgets:

| Title | Visualization | Perspective | Filters | Parameters |
| --- | --- | --- | --- | --- |
| `Hotel-sector reports` | **Number** | **Entities** | **Entity type** = Report, **Label** = `hotel-sector` | **Date attribute**: `created (Functional date)`, the publication date of a report |
| `Threats targeting the sector` | **List (distribution)** | **Knowledge graph** | **Relationship type** = targets, **Target entity** = Hospitality | **Attribute**: **Entity**, **Display the source**: on, **Number of results**: 10, **Date attribute**: `created (Functional date)` |
| `Countries they target` | **Map** | **Knowledge graph** | **Dynamic source filters**: **In regards of** with **Relationship type** = targets and **Entity** = Hospitality. Relationship filters: **Relationship type** = targets, **Target type** = Country | **Date attribute**: `created (Functional date)`, default center and zoom |
| `New indicators per week` | **Line** | **Entities** | **Entity type** = Indicator | **Interval**: **Week**, **Date attribute**: `created (Functional date)` |
| `Latest reports` | **List** | **Entities** | **Entity type** = Report, **Label** = `hotel-sector` | **Number of results**: 10, **Sort by**: **Created**, **Sort mode**: **Desc**, **Date attribute**: `created (Functional date)` |

Put the counter and the threat ranking in a narrow column on the left, the line and the map on the right, and the list across the bottom. Then choose **Manage access restriction** and give the group of member CISOs **can view**.

:::only{opencti=">=7.261119"}
Also choose **Save** > **Set as dashboard defaults** in the bar above the widgets, so that the CISOs open the dashboard on the last six months, not only you.
:::

The CISOs see how many hospitality reports were published in the last six months; Velvet Harbor, Ember Tide, Copper Lantern, Velvet Loader and Operation Low Season among the threats that target the sector; France, Spain and Italy on the map; and whether new indicators keep coming. Next month, the same dashboard covers the six months before it without any change.
::::

To start from a dashboard someone else built, import its JSON file with **Import dashboard**, or browse ready-made dashboards with **Import from Hub**, which only shows when your platform can reach the XTM Hub. See [Share dashboards](/opencti/use/dashboards/share/).

## Set the period

The period applies to every widget, each on the date chosen in its own **Date attribute**:

- **Relative time** keeps a moving window: **Last 24 hours**, **Last 7 days**, **Last month**, **Last 3 months**, **Last 6 months** or **Last year**. Use it for a dashboard people come back to.
- **Start date** and **End date** fix the period, for a quarterly review or an incident. They are greyed out while a relative time is set: choose **None** in **Relative time** first.

**Last 3 months** is a moving window, not a calendar quarter. For "the third quarter of 2026", set **Start date** and **End date**.

:::only{opencti="<7.261119"}
The period is saved with the dashboard, and only users who can edit it see these controls. A reader with **can view** always sees the period the editor chose.
:::

:::only{opencti=">=7.261119"}
A dashboard opens on the period saved with it. Changing the period changes your own view only: the other readers keep theirs. To save the period for everyone, an editor then chooses **Save** > **Set as dashboard defaults**, as described in [Let each reader choose the values](#let-each-reader-choose-the-values). Readers with **can view** also get these controls, unless an editor turns off **Allow viewers to edit time filters**: they then always see the period saved with the dashboard.
:::

:::only{opencti=">=7.260811"}
For a wall screen, choose a refresh interval in **Refresh interval**, next to **Refresh**, from **1m** to **1d**, or **Off**. The interval is saved with the dashboard, so every reader's screen reloads its widgets at that pace. Click **Refresh** to reload them now.
:::

::::only{opencti=">=7.261119"}
## Let each reader choose the values

When the same question comes back for several values, such as one briefing per report type, a variable saves you a copy of the dashboard per value. The widgets filter on the variable, and each reader picks its value in a chip of the bar above the widgets. A preset saves variable values and a period under a name, to switch to that view in one click.

:::steps

1. In the bar above the widgets, click **Variable**. In **Create variable**, enter a **Name** and choose a **Type**, such as **Entity**, **Label**, **Marking**, **Status**, **Vocabulary**, **Entity type**, **Text**, **Numeric** or **Date**. Some types ask for one more setting: the **Entity types** of an entity, the **Vocabulary category** of a vocabulary, the **Kill chain** of a kill chain phase.
2. Choose which values readers can pick. **No restriction** offers every value of the type. **Manual selection** offers only the **Values** you list. **By filter**, for an entity, offers the entities that match the filters you set.
3. Choose the **Default value**, then click **Create**. The variable shows as a chip in the bar. A dashboard holds up to 50 variables.
4. In the filters of each widget that must follow the variable, set the value of the filter to **Variable** instead of **Fixed value**, and pick the variable. A marker on the widget then lists, in its tooltip, the variables it uses.
5. Pick values in the chips and a period, then click **Save** and choose **Save as new preset…**. Enter the **Preset name** and click **Save**.

   ::screenshot{id=opencti-dashboards-variables alt="The Partner briefing dashboard with, above its widgets, the bar of variables: the Preset list, the Report type chip set to threat-report, the Variable button and the Save menu."}

:::

The **Save** menu says for whom each choice writes:

- **Update** followed by the name of the active preset saves the values on screen in that preset, for everyone using it.
- **Save as new preset…** saves them under a new name, for anyone to pick.
- **Set as dashboard defaults** makes them the values and the period the dashboard opens on, for everyone without a preset.

The **Preset** list, at the start of the bar, switches from one preset to another, and **No preset** goes back to the values of the dashboard. For editors, its menu also offers **Rename preset** and **Delete preset**. The users and groups that open the dashboard on a deleted preset see its default values instead. A preset can be the one a dashboard opens on for a user, a group or an organization: see [Read the home dashboard and AI summaries](/opencti/use/explore/insights/) and [Separate data between organizations](/opencti/administer/organizations/).

A value, a period or a preset that a reader picks changes their own view only. **Modified**, and a dot on each changed chip, show that the view departs from the preset or from the defaults; **Reset to defaults**, or **Reset to** the reader's default preset, goes back. To change or remove a variable, use the menu of its chip. In **Edit variable**, the **Type** and the setting that comes with it can no longer change. **Delete variable** lists the widgets that use the variable: they show no data until you change their filters. The value of the variable also leaves the presets that hold it.

Readers with **can view** can change the period, the variable values and the preset for themselves. To fix what they see, click **Update** in the header of the dashboard and turn off **Allow viewers to edit time filters**, **Allow viewers to change variable values** or **Allow viewers to switch presets**. None of these switches lets a reader change the dashboard.

:::example[One partner briefing for every report type]
The CTI team lead briefs partner hotels on threat reports, and sometimes on the advisories written for members or on malware analyses. Rather than three dashboards, the `Partner briefing` dashboard gets a variable. Click **Variable** and set:

- **Name**: `Report type`, **Type**: **Vocabulary**, **Vocabulary category**: report types.
- **Manual selection**, with the **Values** `threat-report`, `member-advisory` and `malware`.
- **Default value**: `threat-report`.

In the `Reports of the selected type` widget, set the report type filter to **Variable** and pick `Report type`. Then choose `member-advisory` in the **Report type** chip and **Last 6 months** in **Relative time**, click **Save** > **Save as new preset…** and enter the **Preset name** `Member advisories`.

Readers open the dashboard on threat reports. Choosing **Member advisories** in the **Preset** list shows six months of member advisories, without changing what the rest of the team sees.
:::
::::

## Share it with your team and leadership

A new dashboard is visible to its creator only.

:::steps

1. Open the dashboard, click the actions menu (⋮) next to the export buttons, and choose **Manage access restriction**. The dialog lists who has access today, starting with you as the creator.

   ::screenshot{id=opencti-dashboards-manage-access alt="The Manage access restriction dialog of the Hotel sector campaign dashboard, with the field to add users, groups or organizations and the current accesses, the creator holding can manage."}

2. Add users, groups or organizations, and give each one an access level:
   - **can view**: open the dashboard. Give it to executives and to anyone who only reads.
   - **can edit**: also change the widgets, the layout and the period.
   - **can manage**: also delete it, publish it as a public dashboard and change who can access it.
3. Save. At least one member keeps **can manage**.

:::

Access levels add to capabilities, they do not replace them: a user with **can edit** still needs **Create / Update dashboards** to change the dashboard, and a user with **can manage** needs **Delete dashboards** to delete it.

Sharing a dashboard does not share its data. Two readers of the same dashboard can see different numbers: a CISO whose group cannot see `TLP:RED` gets counts without the `TLP:RED` reports. To show the dashboard to people who have no account, such as partner hotels, create a [public dashboard](/opencti/use/dashboards/share/).

## Duplicate, export or delete

The dashboard header, its actions menu (⋮), and the menu at the end of its line in :menu[Dashboards > Custom dashboards] hold the other actions:

- **Update**, a button in the header, changes the **Name** and **Description** of the dashboard. It needs the **can edit** access level.
- **Duplicate the dashboard** (**Duplicate** in the list) copies it with all its widgets. Enter the **New name**, which starts as the name followed by `- copy`. Use it for a variant, for example the same dashboard for another sector. The confirmation message links to the copy, which also appears in the list. You are the only member of the copy.
- **Export**, in the list menu or as the export button of the header, downloads the dashboard as a JSON file to import on another platform. The header also offers **Export to image** and **Export to PDF**, for a briefing or a slide.
- **Add tag**, next to the name, tags the dashboard so it is easier to find in the list.
- **View associated public dashboards** opens :menu[Dashboards > Public dashboards] filtered on the public dashboards made from this one: check them before you change or delete it, since they show its widgets to people without an account.
- **Delete** removes the dashboard and every public dashboard made from it. A dashboard does not go to the trash: this cannot be undone. To delete several dashboards, select them with their checkboxes and delete them from the toolbar.

The menu of each widget offers **Update**, **Duplicate**, **Export** and **Delete** for that widget.

:::only{opencti=">=7.261119"}
An exported widget does not carry the variables it uses. Imported into a dashboard that lacks them, it shows no data until you fix its filters, and a message says how many variables are missing.
:::

:::note
**Investigations**, the graph workspaces in the menu of the same name, use the same access levels with their own capabilities, **Create / Update investigations** and **Delete investigations**. See [Pivot and investigate](/opencti/use/analyze/pivot/).
:::

## What's next

- [Create widgets](/opencti/use/dashboards/widgets/): every visualization and how to filter it.
- [Share dashboards](/opencti/use/dashboards/share/): public links, export and import, the XTM Hub.
- [Add a dashboard tab to entity pages](/opencti/use/dashboards/custom-views/): the same widgets as a tab on every entity of a type.
