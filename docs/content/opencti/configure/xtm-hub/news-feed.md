---
title: Follow what XTM Hub publishes
description: Get notified in OpenCTI when the XTM Hub library publishes new dashboards, playbooks, custom views or integrations, and choose which kinds of news you receive.
sidebar_position: 2
doc_type: how-to
products:
  - opencti
personas: [cti-lead, admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/xtm-hub-news-feed/
---

The XTM Hub news feed tells OpenCTI users when the [XTM Hub library](/opencti/configure/xtm-hub/content-library/) publishes new content, such as a dashboard or a playbook. A CTI team lead spots a ready-made dashboard or playbook the team can reuse instead of building it. This page shows where to read the news and how each user chooses what they receive; administrators find how long the platform keeps the news.

:::only{opencti="<7.260811"}
The news feed is not available in this version.
:::

::::only{opencti=">=7.260811"}
## Before you start

- An administrator has [connected the platform to the XTM Hub](/opencti/configure/xtm-hub/).
- Your role has the **Access knowledge** capability, and you are not working in a draft: the top bar icon that leads to the news only shows then. Each user manages their own preferences: no other capability is needed.

:::only{opencti=">=7.261127"}
- The page follows the same rule when you open it by its address: without **Access knowledge**, or from a draft, you land on the dashboard.
:::

## How news reaches the platform

Every hour, a connected platform fetches the news published on the XTM Hub since the last check. Each item is a new library resource, such as a **Custom Dashboard**, a **Playbook** or a **Custom View**. New items pop up as notifications, with **Open in XTM Hub** on each and **Dismiss all** to close them. Beyond five new items, a link opens them all on the XTM Hub.

Items older than 180 days are deleted. To keep them for another period, an administrator sets `hub_registration_manager:news_feed_cleanup_interval_value`, a positive number, and `hub_registration_manager:news_feed_cleanup_interval_unit`, one of `seconds`, `minutes`, `hours`, `days`, `weeks`, `months` or `years`, in the [configuration](/reference/opencti/configuration/#engines-schedules-and-managers). Any other value stops the platform at startup.
::::

::::only{opencti=">=7.261030"}
Items about new integrations read **Integration**. An item of a kind this version of OpenCTI does not know yet reads **Unsupported type**: it still opens in the XTM Hub.

## Read the news

:::steps

1. Click the **News Feed** icon, the megaphone in the top bar. A dot on it shows unread news.
2. Read the items of the **News feed** tab: **Type**, **Title**, **Original creation date** and **Tags**.
3. Click **Open in XTM Hub** on an item to review it and deploy it.

:::

:::only{opencti=">=7.261127"}
The global search also leads there: type `news` or `hub` in the search bar, and the **News Feed** page shows under **Application pages**.
:::

::screenshot{id=opencti-xtm-hub-news-feed alt="The XTM Hub News Feed page with its News feed and Settings tabs, the XTM Hub is disconnected banner with its Connect product to XTM Hub button, and the news table with its Type, Title, Original creation date and Tags columns, empty because the platform is not connected to the XTM Hub."}

:::only{opencti=">=7.261109"}
When the XTM Hub publishes blog posts for your version of OpenCTI, a **Blogpost** tab shows between **News feed** and **Settings**, with its own count of unread posts. It shows the latest six posts as cards, each with its reading time, such as `5 min read`: click a card to read the post on the XTM Hub, or **See all** for the other posts. The **News feed** tab then lists the other items only, and each tab marks its own items as read when you open it.
:::

Items are marked as read when you open the page. When the platform is not connected, the page says **XTM Hub is disconnected**, with a **Connect product to XTM Hub** button. When the platform cannot reach the XTM Hub at all, for example behind a firewall that blocks it, the banner says **XTM Hub is unreachable and connection can't be established** and no news arrives.

## Choose what you receive

:::steps

1. On the **XTM Hub News Feed** page, open the **Settings** tab.
2. In the **XTM Hub News Feed settings** card, turn **Enable News Feed notifications** off to stop all news, or keep it on and turn off the kinds of content you do not want.

:::

The page stays available when all news is off, but you no longer get notifications.

:::example[Follow only new dashboards and playbooks]
Ana Duval builds the dashboards and playbooks of the Example CERT team. New custom views and integrations are another colleague's job, so she only wants news about the first two.

1. On the **XTM Hub News Feed** page, open the **Settings** tab.
2. Keep **Enable News Feed notifications** on, keep **Custom Dashboard** and **Playbook** on, and turn off **Custom View** and **Integration**.

From the next hourly fetch, Ana gets a notification only when the XTM Hub library publishes a dashboard or a playbook. The team reviews news once a quarter, so the administrator also keeps items for 90 days instead of 180, and restarts the platform:

```yaml
services:
  opencti:
    environment:
      HUB_REGISTRATION_MANAGER__NEWS_FEED_CLEANUP_INTERVAL_VALUE: 90
      HUB_REGISTRATION_MANAGER__NEWS_FEED_CLEANUP_INTERVAL_UNIT: days
```
:::
::::

::::only{opencti=">=7.260811 <7.261030"}
## Read the news

:::steps

1. Click the **Notifications** icon in the top bar.
2. Open the **XTM Hub News Feed** tab. It shows only when the platform is connected and your news is on.
3. Read the items: **Type**, **Title**, **Original creation date** and **Tags**. Click **Open in XTM Hub** on an item to review it and deploy it.

:::

Items are marked as read when you open the tab.

## Choose what you receive

:::steps

1. Open your profile.
2. In the **XTM Hub News Feed** card, turn **Enable News Feed notifications** off to stop all news, or keep it on and turn off the kinds of content you do not want.

:::

:::example[Follow only new dashboards and playbooks]
Ana Duval builds the dashboards and playbooks of the Example CERT team; new custom views are another colleague's job. In the **XTM Hub News Feed** card of her profile, she keeps **Enable News Feed notifications**, **Custom Dashboard** and **Playbook** on, and turns off **Custom View**. From the next hourly fetch, she gets a notification only when the XTM Hub library publishes a dashboard or a playbook. The team reviews news once a quarter, so the administrator keeps items for 90 days instead of 180, and restarts the platform:

```yaml
services:
  opencti:
    environment:
      HUB_REGISTRATION_MANAGER__NEWS_FEED_CLEANUP_INTERVAL_VALUE: 90
      HUB_REGISTRATION_MANAGER__NEWS_FEED_CLEANUP_INTERVAL_UNIT: days
```
:::
::::

## What's next

- [XTM Hub content library](/opencti/configure/xtm-hub/content-library/) for what you can deploy.
- [Notifications](/opencti/use/automate/notifications/) for the other notifications of OpenCTI.
