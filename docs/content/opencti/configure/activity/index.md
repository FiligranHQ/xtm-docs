---
title: Audit who did what
description: What OpenCTI activity records about who did what, where and when, the difference between knowledge history, audit and extended activity, and where each one goes.
sidebar_position: 40
doc_type: explanation
products:
  - opencti
edition: ee
personas: [admin, auditor, executive]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/audit/overview/
---

Activity answers "who did what, where and when" in OpenCTI. It records the administration and security actions of every user, and, for the users you choose, what they read, download, export or search. Security, audit and compliance teams use it to detect misuse and to prove how the platform is run.

You find it in :menu[Settings > Activity]. It requires an Enterprise Edition license, and the **Access security activity** capability to see the events, choose who is listened to and set up alerts.

## What it is good for

| Goal | Example |
| --- | --- |
| Investigate an incident | List everything a user did in the hour before their account was disabled |
| Detect misuse | Be alerted when someone fails to log in, or tries an action they are not allowed to |
| Watch sensitive users | Record what contractors read and export, not only what they change |
| Feed your SIEM | Ship the audit log of the platform to your log pipeline |
| Prove compliance | Export the changes made to roles, groups and authentication over a quarter |

## Three kinds of records

| Kind | What it covers | Recorded for | Where you see it |
| --- | --- | --- | --- |
| Knowledge history | Creation, update and deletion of STIX knowledge: reports, indicators, relationships | Everyone, without Enterprise Edition | The **History** of each entity, and the events screen with **Include knowledge** |
| Audit | Administration and security actions: logins, users, groups, roles, settings, connectors, feeds, customization | Everyone, as soon as Enterprise Edition is on. It cannot be turned off | The events screen and the audit log |
| Extended activity | Reads of knowledge and dashboards, file uploads and downloads, imports, exports, enrichments, global searches | Only the users, groups and organizations you [listen to](/opencti/configure/activity/configuration/) | The events screen |

Extended activity is off by default because it records a lot: every read is an event. Turn it on for the people who need it.

## How it works

Every action a user takes through the interface or the API is handed to the activity listener. The listener drops extended actions of users nobody listens to, writes administration actions to the audit log, and passes everything else to the activity manager. The manager stores the events, shown in :menu[Settings > Activity > Events], and checks them against your activity triggers.

::diagram{id=opencti-configure-activity-flow}

Rules that surprise people:

- **Nothing is recorded before you listen.** Extended activity starts when you add a user, a group or an organization; past actions are not recovered.
- **Reads are counted once an hour.** A user who opens the same report ten times in an hour leaves one read event.
- **The audit log holds administration events only**, by default. The events screen holds both kinds. To ship extended events to the log as well, see [Choose what activity is recorded](/opencti/configure/activity/configuration/#send-activity-to-your-log-pipeline).
- **Demo mode hides names.** On a platform in demo mode, the names in the activity are redacted.

[Activity events](/opencti/configure/activity/events/) explains every event type and scope, and the [audit event reference](/reference/opencti/audit-events/) lists each event the platform publishes.

## Activity and the other tools

| You want to | Use |
| --- | --- |
| See what changed on one entity | Its **History** tab, available to every user who can see the entity |
| See what a user did across the platform | The events screen, filtered on that user, or the **History** tab of the user in :menu[Settings > Security > Users] |
| Be told about an administration or security event | An [activity trigger](/opencti/configure/activity/triggers/) |
| Be told about new or changed knowledge | A [knowledge trigger](/opencti/use/automate/notifications/) |
| Chart activity over time | A dashboard widget with the **Activity & history** perspective: see [widgets](/opencti/use/dashboards/widgets/) |

## What's next

- [Choose what activity is recorded](/opencti/configure/activity/configuration/) to choose who is listened to and where the audit log goes.
- [Activity events](/opencti/configure/activity/events/) to read and export the events.
- [Get alerted on security events](/opencti/configure/activity/triggers/) to be notified of the events that matter.
