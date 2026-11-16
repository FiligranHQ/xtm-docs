---
title: Get alerted on security events
description: Get notified of administration and security events in OpenCTI, such as failed logins or unauthorized actions, as soon as they happen or in a regular digest.
sidebar_position: 3
doc_type: how-to
products:
  - opencti
edition: ee
personas: [admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/audit/triggers/
---

An activity trigger watches the [activity events](/opencti/configure/activity/events/) and notifies chosen recipients when one matches its filters, so you do not have to watch the events screen. This page shows how to create a live trigger, which notifies at once, and a digest, which groups the events of live triggers over a period. To be notified about knowledge rather than activity, use [knowledge triggers](/opencti/use/automate/notifications/).

## Before you start

- Your platform has an Enterprise Edition license.
- Your role has the **Access security activity** capability.
- The events you want to watch are recorded: extended events, such as reads and downloads, need the users to be [listened to](/opencti/configure/activity/configuration/).
- To notify by email, the platform can send emails: see [Send emails from OpenCTI](/opencti/administer/smtp/).

## Create a live trigger

:::steps

1. Open :menu[Settings > Activity > Alerting] and click **Create Live trigger**. The **Create a live activity trigger** panel opens.
2. Enter a **Name** and a **Description** that say what the trigger watches.
3. In **Notifiers**, choose how recipients are told: **User interface** for the notification center, **Default mailer** for an email, or a [notifier](/opencti/configure/notifiers/) you created. Leave it empty for a trigger that only feeds a [digest](#create-a-digest): it then tells nobody on its own.
4. In **Recipients**, add the users, groups or organizations to notify. At least one is required.
5. Add filters to keep only the events that matter. Combine them as in any OpenCTI filter.

   | Filter | Matches | Example |
   | --- | --- | --- |
   | **Event type** | The type of event | `authentication` |
   | **Event scope** | The scope within the type | `login` |
   | **User** | The user who acted | A contractor account |
   | **Group** | A group of the user who acted | `Administrators` |
   | **Organization** | An organization of the user who acted | A partner organization |

6. Click **Create**.

   ::screenshot{id=opencti-activity-trigger-live-create alt="The Create a live activity trigger panel with the Name and Description fields, the notifiers and recipients fields, and the filter bar limited to activity filters."}

:::

For example, to hear about login failures, filter on **Event type** `authentication` and **Event scope** `login`; failed logins carry the status error. To catch attempts to go beyond one's rights, filter on **Event scope** `unauthorized`.

:::example[Alert administrators when a contractor oversteps]
The contractors of Harbor Security work on the Example CERT platform with limited rights. The administrators want to know at once when one of them tries an action their role does not allow.

- **Name**: `Contractor unauthorized actions`
- **Description**: `Harbor contractors trying actions outside their role`
- **Notifiers**: **User interface** and **Default mailer**
- **Recipients**: the **Administrators** group
- Filters: **Event scope** `unauthorized`, and **Group** `Harbor contractors`

The two filters combine with AND, so analysts of Example CERT who hit a missing capability do not raise the alert. When Sam Reyes, a member of Harbor contractors, tries to delete a report, every administrator gets a notification and an email naming Sam Reyes and the refused action.
:::

## Create a digest

A digest sends, at a set time, the events that one or more live triggers caught over the last period: the last day for a daily digest, the last week for a weekly one. When none of its triggers caught anything, no digest is sent. Use it for events worth a weekly review rather than an immediate alert, and create the live triggers first.

:::steps

1. In :menu[Settings > Activity > Alerting], click **Create Regular digest**. The **Create a regular activity digest** panel opens.
2. Enter a **Name** and a **Description**.
3. In **Triggers**, choose the live triggers to group. At least one is required.
4. Choose the **Period**: **hour**, **day** (the default), **week** or **month**. For a week, choose the **Week day**; for a month, the **Month day**. Except for an hourly digest, set the **Time** of sending, in your browser's time zone. It starts at 12:00 AM.
5. Choose at least one notifier in **Notifiers** and at least one recipient in **Recipients**, then click **Create**. A recipient of the digest only gets the events of the live triggers they are also a recipient of: add them to those triggers too.

   ::screenshot{id=opencti-activity-digest-create alt="The Create a regular activity digest panel with the Name, Description and Triggers fields, the Period set to day, the Time field, and the Notifiers and Recipients fields."}

:::

## Check that it works

Make the trigger fire: for a login trigger, log out and log in again, or enter a wrong password. The recipients get a notification, in their notification center or by email, that names the user and the event. If nothing arrives, check the filters against the event in :menu[Settings > Activity > Events].

## Change or delete a trigger

Open the row menu of the trigger in :menu[Settings > Activity > Alerting] and choose **Update** or **Delete**. **Update** opens **Update an activity live trigger** or **Update an activity digest trigger**, with the fields of the creation panel; each change is saved when you leave the field. The list shows the **Type** of each trigger, live or digest, and its **Notification** channels.

## What's next

- [Activity events](/opencti/configure/activity/events/) for every type and scope you can filter on.
- The [audit event reference](/reference/opencti/audit-events/) for the message of each event a trigger can catch.
- [Notifiers](/opencti/configure/notifiers/) to send notifications to a webhook or another channel.
