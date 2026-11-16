---
title: Get alerted when knowledge changes
description: Get alerted when knowledge changes in OpenCTI with live triggers, instance subscriptions and digests, and read your alerts in one place.
sidebar_position: 2
doc_type: how-to
products:
  - opencti
personas: [cti-lead, cti-analyst, admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/notifications/
---

Triggers make sure the right people hear about what matters as it happens: the duty analyst learns that Velvet Harbor has a new indicator, the team gets one email each morning with the new hospitality reports. A trigger watches the knowledge base for the events you choose and sends an alert through the notifiers you choose: in the platform, by email or to a webhook. This page shows how to create a live trigger, subscribe to a specific entity, group alerts in a digest, alert a whole team, and manage the alerts you receive.

## How alerting works

A **live trigger** listens to creations, modifications and deletions that match its filters. A **regular digest** collects what one or more live triggers caught and sends it as one message on a schedule. Each trigger or digest hands its alerts to **notifiers**, the channels that deliver them.

::diagram{id=opencti-notifications-flow}

Some alerts need no setup:

- You are notified when you become an assignee or a participant of an object, and then of every change to it and of its deletion, through the **Personal notifiers** set in your **Profile**.

  :::only{opencti=">=7.260811"}
  An administrator can turn this off for the whole platform with **Automatically notify users when they are assigned as participant or assignee**, in :menu[Settings > Customization > Notifiers].
  :::

- When someone asks you for access to an object through a request access case, you are notified if you can approve it.

Triggers only alert: they never change data. To act on events, use a [playbook](/opencti/use/automate/playbooks/). To alert on user activity rather than knowledge, see [activity triggers](/opencti/configure/activity/triggers/).

## Before you start

- Any user can create triggers for themselves.
- To create triggers for other users, groups or organizations, your role needs the **Manage credentials** capability, or you must be an administrator of the organization.
- To deliver by email or webhook, a notifier must exist. Two are built in: **User interface** and **Default mailer**. Administrators add others in :menu[Settings > Customization > Notifiers]; see [Notifiers](/opencti/configure/notifiers/).

## Open your triggers and alerts

:::only{opencti=">=7.261030"}
Click the bell icon in the top bar to open the **Notification Center**. It has two tabs: **Alerts** lists the alerts you received, **Triggers** lists your triggers and digests.
:::

:::only{opencti="<7.261030"}
The top bar has two icons: the bell, **Notifications**, opens the alerts you received, and **Triggers** opens your triggers and digests.
:::

::screenshot{id=opencti-notifications-triggers-list alt="The Triggers tab of the Notification Center lists live triggers and digests, with the Create Regular digest and Create Live trigger buttons above the list."}

## Create a live trigger

:::steps

1. Open your triggers and click **Create Live trigger**. The **Create a live trigger** panel opens.
2. Enter a **Name** and, optionally, a **Description** that says what the trigger watches.
3. In **Triggering on**, choose the events to watch: **Creation**, **Modification**, **Deletion**, or several.
4. In **Notifiers**, choose how to receive the alerts, for example **User interface** and **Default mailer**.
5. Add filters to narrow what the trigger watches: entity types, labels, markings, authors, assignees and any other filter of the platform. Without filters, the trigger fires on every event. The [filter key reference](/reference/opencti/filter-keys/) lists every key and operator, including the ones that only stream, trigger and playbook filters accept.
6. Click **Create**.

   ::screenshot{id=opencti-notifications-live-trigger alt="The Create a live trigger panel with the Name and Description fields, Creation, Modification and Deletion selected in Triggering on, the Notifiers field, the Subscription to specific object(s) switch and the filters."}

:::

:::example[Alert the CTI team about new hospitality reports]
The CTI team wants to know as soon as a report on the hospitality sector arrives or changes. In the **Create a live trigger** panel:

- **Name**: `New hotel-sector reports`
- **Description**: `Reports labelled hotel-sector, when created or modified.`
- **Triggering on**: **Creation** and **Modification**
- **Notifiers**: **User interface** and **Default mailer**
- Filters: **Entity type** = Report, and **Label** = `hotel-sector`

Every report that gets the `hotel-sector` label, or is changed while it has it, appears in your alerts and in your mailbox. Reports on other sectors, and other entity types with the label, do not trigger it.
:::

A deletion alert also fires when an object leaves your view: a change of marking or sharing can make an object invisible to you, and the trigger reports it as deleted.

## Subscribe to specific entities

An instance trigger watches one or more entities you name, rather than a filter. It alerts you when:

- the entity is modified or deleted;
- a relationship to or from the entity is created or deleted;
- an object that refers to the entity is created or deleted, such as a report that contains it or an entity it authored;
- the entity is added to or removed from another object, such as a report or the author field of an entity.

You can subscribe in two ways.

**From the entity.** Open the entity and click the bell button in its header, **Subscribe to updates (modifications and new relations)**. The **Manage subscription** panel opens with the entity's name, **Modification** and **Deletion** in **Triggering on**, and the **User interface** and **Default mailer** notifiers: change them if needed and click **Subscribe**. Click the bell again to **Update** these settings or to **Unsubscribe**.

**From the trigger form.** Create a live trigger, turn on **Subscription to specific object(s)** and choose the entities. **Triggering on** then offers **Modification** and **Deletion** only.

## Group alerts in a digest

A digest sends what your live triggers caught as one message, so a busy trigger does not flood your inbox.

:::steps

1. Open your triggers and click **Create Regular digest**.
2. Enter a **Name** and, optionally, a **Description**.
3. In **Triggers**, choose the live triggers to include.
4. In **Period**, choose how often to send the digest: **hour**, **day**, **week** or **month**.
5. Set when it goes out: **Week day** for a weekly digest, **Month day** for a monthly one, and **Time** for any period longer than an hour.
6. In **Notifiers**, choose how to receive it, then click **Create**.

:::

:::example[Send the CTI team a daily hospitality digest]
The live trigger above can fire dozens of times a day during a campaign. To read it once each morning instead, create a regular digest:

- **Name**: `Daily CTI digest`
- **Description**: `Everything of the day at 8 AM.`
- **Triggers**: `New hotel-sector reports`
- **Period**: **day**
- **Time**: 08:00
- **Notifiers**: **Default mailer**

Every morning at 8:00, one email lists the hospitality reports created or modified over the last day. Keep only **User interface** in the notifiers of the live trigger, so you see each report in the platform as it arrives without receiving each one by email as well.
:::

In the alerts list, a digest shows as one line. Click it to open the alerts it groups.

## Alert a whole team

A trigger created on a group or an organization alerts every member, including people who join later and no longer those who leave. Aim the alerts of a rota at a group, such as the analysts on duty this week, and change the group's members when the duty changes: the triggers follow. This takes the **Manage credentials** capability, or being an administrator of the organization; a team lead without it asks an administrator.

A trigger you create for someone else alerts them, but they can only read it: the trigger list shows **This trigger/digest has been shared with you and you are not able to modify or delete it**.

:::steps

1. Open :menu[Settings > Security > Users], :menu[Settings > Security > Groups] or :menu[Settings > Security > Organizations], then the user, group or organization to alert.
2. In the **Triggers and Digests** card, create a live trigger or a digest as above.

:::

:::example[Notify the duty analyst when Velvet Harbor gets a new indicator]
At Example CERT, one analyst a week is on duty and must react as soon as anything new is linked to Velvet Harbor, starting with new indicators. Barbara, the team lead, keeps a group `Duty analysts` with the analyst of the week as its only member. With an administrator, she opens :menu[Settings > Security > Groups], then `Duty analysts`, and in the **Triggers and Digests** card creates a live trigger:

- **Name**: `Velvet Harbor watch`
- **Description**: `Anything new linked to Velvet Harbor, for the analyst on duty.`
- **Subscription to specific object(s)**: on, with the intrusion set `Velvet Harbor`
- **Triggering on**: **Modification** and **Deletion**
- **Notifiers**: **User interface** and **Default mailer**

When a feed adds an indicator that indicates Velvet Harbor, the relationship between them fires the trigger: the analyst on duty gets an alert in the platform and an email about the new relationship. New reports that contain Velvet Harbor and new techniques it uses alert them too. On Monday, Barbara replaces the member of `Duty analysts`, and the alerts go to the new analyst without touching the trigger.
:::

## Manage your alerts

In the alerts list you can search, filter and sort alerts, and click an alert to open the object it points to. The first column gives the event: **Creation**, **Modification** or **Deletion**. A line marked **Multiple**, with the message **Digest with multiple notifiers**, groups several events, from a digest or from several changes to the same object within a minute: click it to list them in **Digests Details**.

- To mark one alert as read or unread, use the button on its line. To remove it, click **Delete this notification**.
- To act on many alerts at once, select them and use the toolbar: **Mark as read**, **Mark as unread** or **Delete**. OpenCTI runs this as a [background task](/opencti/use/automate/background-tasks/).

## Check that it works

Create a trigger with your own user as **Creator** in its filters, then create an object that matches. An alert appears in your alerts list within about a minute, and the bell shows a counter. The delay is on purpose: the platform waits 60 seconds after the first event on an object, so that a burst of changes to the same object arrives as one alert rather than ten, on every notifier. A digest goes out at the time you set, checked every minute.

:::note[For administrators]
The grouping window is `publisher_manager:buffering_seconds` (`PUBLISHER_MANAGER__BUFFERING_SECONDS`, 60 by default). Set `publisher_manager:enable_buffering` (`PUBLISHER_MANAGER__ENABLE_BUFFERING`) to `false` to send each event as it comes, at the cost of one alert per change. Live events are followed as they happen and digests are checked every minute, both fixed in the code: `notification_manager:interval` (`NOTIFICATION_MANAGER__INTERVAL`) and `publisher_manager:interval` (`PUBLISHER_MANAGER__INTERVAL`) are declared in the default configuration but have no effect. See [Engines, Schedules and Managers](/reference/opencti/configuration/#engines-schedules-and-managers) in the configuration reference.
:::

:::example[Alert the duty analyst within seconds]
The analysts on duty want alerts on Velvet Harbor faster than a minute, and accept a few more of them. The platform runs with Docker Compose: the administrator shortens the grouping window on the `opencti` service and restarts it.

```yaml
services:
  opencti:
    environment:
      - PUBLISHER_MANAGER__BUFFERING_SECONDS=15
```

Alerts now arrive about 15 seconds after the first change to an object, and the changes made within those 15 seconds still arrive as one alert.
:::

## What's next

- [Notifiers](/opencti/configure/notifiers/): create email and webhook notifiers, and restrict who can use them.
- [Automate with playbooks](/opencti/use/automate/playbooks/): act on events, not only alert.
- [Get alerted on security events](/opencti/configure/activity/triggers/): alert administrators about what users do.
