---
title: Choose what activity is recorded
description: Choose the users, groups and organizations whose reads, files and exports OpenCTI records, and send the audit log to files, the console or your log pipeline.
sidebar_position: 1
doc_type: how-to
products:
  - opencti
edition: ee
personas: [admin, platform-engineer]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/audit/configuration/
---

OpenCTI records the administration and security actions of every user without any setting. This page shows how to add extended activity for chosen users, groups or organizations, and how to route the audit log to your log pipeline. Read [Audit who did what](/opencti/configure/activity/) first for what each kind of record covers.

## Before you start

- Your platform has an Enterprise Edition license.
- Your role has the **Access security activity** capability.
- To change where the audit log goes, you can edit the platform configuration and restart the platform.

## Listen to users, groups or organizations

Extended activity records what people read, upload, download, import, export and search. It costs storage and indexing time, so listen to the people who need it rather than to everyone.

:::steps

1. Open :menu[Settings > Activity > Configuration].
2. In **Extended activity logging**, choose a user in **Add a user**, a group in **Add a group**, or an organization in **Add an organization**. It is added to the list below at once.
3. Repeat for everyone to listen to. A group or an organization covers all its members.

   ::screenshot{id=opencti-activity-configuration alt="The Extended activity logging card with the Add a user, Add a group and Add an organization fields, above a list holding one group and one user."}

:::

Recording starts right away; actions taken before are not recovered. To stop listening, click the delete icon on the line of the user, group or organization.

## Check that it works

Log in as a listened user, open a report and download one of its files. In :menu[Settings > Activity > Events], filter on that user: a `read` event and a `file` event with the scope `download` appear. See [Activity events](/opencti/configure/activity/events/).

## Send activity to your log pipeline

Besides the events screen, OpenCTI writes audit events as JSON lines to a daily `audit.log` file and to the console, where a container log collector picks them up. Set these keys in the platform configuration, or as environment variables, then restart the platform:

| Key (environment variable) | Default | Use it to |
| --- | --- | --- |
| `app:audit_logs:logs_files` (`APP__AUDIT_LOGS__LOGS_FILES`) | `true` | Write the `audit.log` file, one per day |
| `app:audit_logs:logs_directory` (`APP__AUDIT_LOGS__LOGS_DIRECTORY`) | `./logs` | Choose the folder of the file. In a container, mount a volume there, or the files are lost when the container is replaced |
| `app:audit_logs:logs_max_files` (`APP__AUDIT_LOGS__LOGS_MAX_FILES`) | `7` | Choose how many daily files are kept. Older files are deleted |
| `app:audit_logs:logs_console` (`APP__AUDIT_LOGS__LOGS_CONSOLE`) | `true` | Write the events to the console |
| `app:audit_logs:logs_in_transports` (`APP__AUDIT_LOGS__LOGS_IN_TRANSPORTS`) | `["administration"]` | Add `extended` to also write extended events to the file and the console |
| `app:audit_logs:trace_request_headers` (`APP__AUDIT_LOGS__TRACE_REQUEST_HEADERS`) | `["user-agent", "x-forwarded-for"]` | Choose the HTTP headers recorded with each event |

Turning off both `logs_files` and `logs_console` stops the audit log, not the activity: the events screen and the triggers keep working.

:::example[Log the work of external contractors in detail]
Example CERT lets the contractors of its partner Harbor Security work on its platform, and its security team must keep a trace of everything they read and download, in its log pipeline as well as in OpenCTI.

1. In :menu[Settings > Activity > Configuration], choose **Harbor contractors** in **Add a group**. Every current and future member of the group is listened to, such as Sam Reyes.
2. Add the analysts of the team who handle sensitive cases one by one in **Add a user**, rather than their whole group.
3. Write the extended events to the console and to the `audit.log` file too, keep 30 daily files on a mounted volume, and record the client address the reverse proxy sets, then restart the platform. With Docker Compose:

```yaml
services:
  opencti:
    environment:
      APP__AUDIT_LOGS__LOGS_IN_TRANSPORTS: '["administration","extended"]'
      APP__AUDIT_LOGS__LOGS_DIRECTORY: /opt/opencti/audit
      APP__AUDIT_LOGS__LOGS_MAX_FILES: 30
      APP__AUDIT_LOGS__TRACE_REQUEST_HEADERS: '["user-agent","x-forwarded-for","x-real-ip"]'
    volumes:
      - opencti-audit:/opt/opencti/audit
volumes:
  opencti-audit:
```

When Sam Reyes downloads the PDF of the report "Night-time exports from a hotel booking platform", the events screen shows a `file` event with the scope `download`, and the same event reaches the log collector as a JSON line. Analysts outside the listened group and users leave only their administration and security events, which keeps the volume low.
:::

The [configuration reference](/reference/opencti/configuration/) lists these keys with the rest. The format of a log line is in [Activity events](/opencti/configure/activity/events/#audit-log-format).

## What's next

- [Get alerted on security events](/opencti/configure/activity/triggers/) to be notified of the events that matter.
- [Activity events](/opencti/configure/activity/events/) for every event type and scope.
- The [audit event reference](/reference/opencti/audit-events/), which marks each event as audit or extended.
