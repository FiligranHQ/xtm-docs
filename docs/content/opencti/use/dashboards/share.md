---
title: Share dashboards
description: Choose how to share an OpenCTI dashboard, publish it to people without an account under a marking cap, and move dashboards between platforms.
sidebar_position: 3
doc_type: how-to
products:
  - opencti
personas: [cti-lead, cti-analyst, admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/dashboards-share/
---

A dashboard can reach people beyond the users who already see it on your platform. Choose the way by who must see it:

| Who must see it | Share it with | What they get |
| --- | --- | --- |
| Colleagues and executives with an account | **Manage access restriction** on the dashboard, see [Build dashboards](/opencti/use/dashboards/#share-it-with-your-team-and-leadership) | The live dashboard, with the data their own access allows |
| People without an account: partner hotels, a board, a screen in the operations room | A [public dashboard](#public-dashboards) | A read-only link, with the data you choose to expose |
| Another OpenCTI instance, or a backup | A [JSON export](#export-and-import) | The widgets and their settings, not the data |
| Another team, from a ready-made design | The [XTM Hub](#dashboards-from-the-xtm-hub) | A dashboard built by Filigran or the community |

## Public dashboards

A public dashboard is a read-only copy of a custom dashboard, reachable at a link without signing in. Use it to show a situation to management, partners or a screen in the operations room.

Keep three rules in mind:

- **Its widgets are frozen, its data is live.** The public dashboard copies the widgets and the period as they were when you created it. Later changes to the custom dashboard do not reach it: to change a widget, delete the public dashboard and create it again with the same name. The data, though, is computed each time someone opens the link, and a relative period such as **Last 3 months** keeps moving.
- **It shows your data, capped by markings.** Widgets show what you, its creator, can see, limited to the **Max level markings** you choose. Filters on **@me** use your user too.
- **Anyone with the link can open it** while it is enabled. Disable or delete it when it is no longer needed. Deleting the custom dashboard deletes its public dashboards too.

:::only{opencti=">=7.261119"}
The period and the values of the dashboard variables are frozen as well: you choose them under **Published values** when you create the public dashboard, and visitors cannot change them. See [Choose the published values](#choose-the-published-values).
:::

Check the widgets before you share, because a few of them do not work on the public page:

- a **Bookmark** widget, or any widget of the **Activity & history** perspective, shows a message saying it is not supported;
- a **Horizontal Bar** of the **Entities** perspective broken down by a second data set shows **Not implemented yet**. The same chart on the **Knowledge graph** perspective works.

:::only{opencti=">=7.261117"}
A bar, line, area or heatmap widget split with **Break down by** keeps one series per value on the public page, with the **Maximum number of series**, the order and **Include Other** you chose. Over a long period, the public page may count it in weeks or months instead of days so that the chart stays drawable.
:::

Give each widget a **Title** and each data set a **Label** as well. Without them, the public page shows generic names, such as **Distribution of entities** or **Relationships distribution** as a title and **Number of entities** in a legend.

:::only{opencti=">=7.260811"}
A number of relationships without a title reads **Relationships number**.
:::

### Before you start

- Your role has the **Manage public dashboards** capability.
- You have the **can manage** access level on the custom dashboard, and it has at least one widget.
- Administrators cap the markings each group may share in public dashboards with **Maximum shareable marking definitions**. See [Organize users in groups](/opencti/administer/groups/).

### Create a public dashboard

::::steps

1. Open the custom dashboard, open the actions menu (⋮) in its header and choose **Create a public dashboard**. You can also start from :menu[Dashboards > Public dashboards] with **Create Public Dashboard**, then choose the **Custom dashboard**.
2. Enter a **Name**. It is shown on the public page and generates the **Public dashboard URI KEY**, the end of the link: lowercase, with dashes for spaces and no special characters. The key must be unique on the platform, so choose a name no other public dashboard uses.
3. Leave **Enabled** on to make the link work right away.
4. In **Max level markings**, choose the highest marking of each kind the widgets may show, for example `TLP:AMBER`. Data marked higher is left out. You only see the markings your groups allow you to share.

   :::only{opencti=">=7.261119"}
   Below, check the period and the variable values under **Published values**, as [Choose the published values](#choose-the-published-values) explains.
   :::

5. Click **Create**.

   ::screenshot{id=opencti-dashboards-share-public-form alt="The Create a public dashboard panel with the custom dashboard to share, the Name, the Public dashboard URI KEY that the name fills in, the Enabled switch and Max level markings."}

::::

::::only{opencti=">=7.261119"}
### Choose the published values

Once a custom dashboard is chosen, the **Published values** section of the panel sets what the public page computes its widgets on. The values start from what you see on the dashboard: the period and variable values of its address when you start from the dashboard itself, else the last ones you used in this browser, else your default preset, else the defaults of the dashboard. Change them to publish another view, without changing the dashboard.

- **Load a preset** shows when the dashboard has presets. Choosing one fills the period and the variables with its values. The public dashboard keeps a copy of them: changing the preset later does not change it.
- **Relative time**, or **Start date** and **End date**, set the period. A date in the future, or an end before the start, blocks **Create** until you fix it.
- One field per variable that a widget uses, named after the variable. Variables that no widget uses are not offered. A text or number field left empty publishes the default value of the variable, as the line under it says.

::screenshot{id=opencti-dashboards-share-published-values alt="The Published values section of the Create a public dashboard panel for Partner briefing: Load a preset, the Relative time, Start date and End date of the period, and the Report type variable."}

The public page shows the period, greyed out, and computes every widget with the values you published. Visitors cannot change them. To publish other values, create another public dashboard.

:::example[Publish the member advisories of the last six months]
The `Partner briefing` dashboard has a **Report type** variable, used by its `Reports of the selected type` widget, and a `Member advisories` preset. The CTI team lead wants partner hotels to see the advisories written for them, while the team keeps working on threat reports in the same dashboard. In the dashboard's actions menu, choose **Create a public dashboard**, enter the **Name** `Member advisories for partners` and the **Max level markings** `TLP:GREEN`, then, under **Published values**:

- **Load a preset**: `Member advisories`. **Relative time** becomes **Last 6 months**, and **Report type** becomes `member-advisory`.

Click **Create**. The public page lists the member advisories of the last six months. The team can switch the dashboard back to `threat-report` for its own work: the public page keeps showing the member advisories.
:::
::::

:::example[Show the hotel campaign to partner hotels]
The CTI team wants the security managers of partner hotel chains, who have no account on the platform, to follow the campaign. From the `Hotel sector campaign` dashboard, choose **Create a public dashboard** and enter:

- **Name**: `Hotel sector campaign for partners`. The **Public dashboard URI KEY** fills in as `hotel-sector-campaign-for-partners`.
- **Enabled**: on
- **Max level markings**: `TLP:GREEN`

The partners open `https://opencti.example.com/public/dashboard/hotel-sector-campaign-for-partners` without signing in. They see the widgets of the dashboard as they were when you created the public dashboard, computed on data marked up to `TLP:GREEN`: objects marked `TLP:AMBER` or higher are left out of every widget. When the campaign is over, choose **Disable public link** in :menu[Dashboards > Public dashboards].
:::

The link has the form `https://<your platform>/public/dashboard/<URI key>`. Copy it with **Copy public link** and send it to your audience.

### Manage public dashboards

:menu[Dashboards > Public dashboards] lists the public dashboards you can see, with their URI key, their source dashboard, whether they are enabled, who shared them and their allowed markings. The menu at the end of each line offers:

- **Go to Original dashboard**;
- **Copy public link**;
- **Disable public link**, or **Enable public dashboard** to turn it back on;
- **Delete**.

To delete several public dashboards at once, select them with their checkboxes and delete them from the toolbar. From a custom dashboard, **View associated public dashboards** in its actions menu opens this list filtered on it.

## Export and import

Export a dashboard or a widget to a JSON file to copy it to another platform, keep a backup, or attach it to a support request.

**Export a dashboard** from the menu of its line in :menu[Dashboards > Custom dashboards] with **Export**, or from the export button in the dashboard header. **Export a widget** with **Export** in the widget's menu. The file is named after the date, the type and the name, for example `20261101_octi_dashboard_Hotel sector.json`.

**Import a dashboard** with **Import dashboard** in :menu[Dashboards > Custom dashboards]. **Import a widget** into an open dashboard with **Import Widget** in its header.

Before you import, check that:

- the file was exported from OpenCTI 5.12.16 or later, and is of the right type: a dashboard file for **Import dashboard**, a widget file for **Import Widget**;
- the objects the filters refer to exist on the target platform. Filters keep their values as exported; a value missing on the target shows as deleted, and you remove it by hand.

Whoever imports a dashboard is its only member at first: share it again with **Manage access restriction**. To copy a dashboard on the same platform, use **Duplicate the dashboard** instead, as [Build dashboards](/opencti/use/dashboards/#duplicate-export-or-delete) explains.

:::only{opencti=">=7.260811"}
A widget built on a saved filter exports the content of that filter, not a reference to it, so the file works on a platform without the saved filter. Export fails if the saved filter no longer exists.
:::

## Dashboards from the XTM Hub

The XTM Hub hosts ready-made dashboards built by Filigran and the community.

- Click **Import from Hub** in :menu[Dashboards > Custom dashboards] to browse them and download a JSON file to import. The button only shows when your platform can reach the XTM Hub.
- If your platform is [registered with the XTM Hub](/opencti/configure/xtm-hub/), click **Deploy in OpenCTI** on a dashboard in the Hub, choose the platform if you registered several, and OpenCTI creates the dashboard and opens it after a few seconds.

## What's next

- [Build dashboards](/opencti/use/dashboards/): build and arrange dashboards.
- [Create widgets](/opencti/use/dashboards/widgets/): the widgets a dashboard holds.
- [XTM Hub](/opencti/configure/xtm-hub/): register your platform to deploy from the Hub in one click.
