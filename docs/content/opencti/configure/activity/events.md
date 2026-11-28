---
title: Activity events
description: Every activity event OpenCTI records, by type and scope, which ones are audit and which are extended, and how to read, filter and export them.
sidebar_position: 2
doc_type: reference
products:
  - opencti
edition: ee
personas: [admin, auditor, soc-manager]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/audit/events/
---

This page explains which events the OpenCTI activity records and when, and the screen where you read them. The [audit event reference](/reference/opencti/audit-events/) lists every event the platform publishes, with the message it writes and whether it is audit or extended. Each event has a type, a scope that refines it, an access (audit or extended) and a status (success or error). Use the types and scopes to filter the events screen and to write [activity triggers](/opencti/configure/activity/triggers/). For the difference between audit and extended activity, read [Audit who did what](/opencti/configure/activity/).

## The events screen

Open :menu[Settings > Activity > Events]. You need an Enterprise Edition license and the **Access security activity** capability.

:::only{opencti=">=7.261127"}
The former address of this list, `/dashboard/audits`, now opens :menu[Settings > Activity > Events], with the filters, search and sorting written in the address, so that bookmarks keep working.
:::

:::only{opencti="<7.261127"}
The same list also answers at `/dashboard/audits`.
:::

::screenshot{id=opencti-activity-events alt="The Events screen of Settings, Activity: a list of events with their timestamp, user, event type, event scope and message, under the filter bar and the Include knowledge checkbox."}

- Each line gives the **Timestamp**, the **User**, the **Event type**, the **Event scope** and a **Message** such as ``creates user `jdoe@example.com` ``.
- Click a line to open **Activity raw detail**: the message of the event, the fields it changed with their old and new values, and the **Raw data**. When the event concerns an element that still has a page, **Instance context** gives a **View the element** link to it.
- Search the messages, and add filters, for example on the event type, the event scope or the user.
- Tick **Include knowledge** to add the [knowledge history](/opencti/configure/activity/#three-kinds-of-records) to the list. The checkbox shows when you also have the **Access knowledge** capability.
- Export the list as CSV with the export button. The file holds the 5,000 most recent events that match your filters, with one column per field: identifiers, type, scope, status, timestamp, user, element concerned, message and changes.

## Event types and scopes

| Type | Scopes | Access | Recorded when |
| --- | --- | --- | --- |
| `authentication` | `login`, `logout`, `forgot` | Audit | A user logs in, fails to log in (status error), is refused because of their IP address, logs out, or goes through a password reset |
| `read` | `read` | Extended | A user opens an entity, a relationship, a dashboard or an investigation. Counted once an hour per user and element |
| `read` | `unauthorized` | Audit | A user tries to read something their capabilities do not allow |
| `mutation` | `create`, `update`, `delete` | Audit, or extended for personal objects | A user creates, changes or deletes a platform object: [Audit events](#audit-events) and [Extended events](#extended-events) say which |
| `mutation` | `unauthorized` | Audit | A user tries a change their capabilities do not allow |
| `file` | `read`, `download` | Extended | A user opens or downloads a file |
| `file` | `create`, `delete` | Extended | A user uploads a file, a new version of a file, or an analyst workbench, or deletes one |
| `file` | `disseminate` | Audit | A user sends files to a dissemination list |
| `command` | `search` | Extended | A user runs a global search |
| `command` | `enrich`, `analyze` | Extended | A user asks a connector to enrich or analyze an entity |
| `command` | `import`, `export` | Extended | A user asks for a file import or an export |
| `command` | `send` | Audit | An administrator sends an [email template](/opencti/configure/email-templates/) to a user |

:::only{opencti=">=7.261127"}
Two more events are audit events, recorded for everyone:

- `authentication` · `logout` with the message `logout (maximum session lifetime reached)`: a session reached the [maximum session lifetime](/opencti/administer/cookies/#end-sessions-after-a-fixed-time) set by `app:session_max_lifetime`, and the platform ended it. An ended session gives one event, even when several of its requests reach the limit at the same time.
- `ingestion_health`, with the scopes `critical`, `degraded` and `recovered`: the [health of a connector, a feed or a synchronizer](/opencti/use/ingest/connectors/) changed and stayed so. The message names the source and what fails, and the status is error, except for `recovered`.
:::

## Audit events

Audit events are recorded for every user as soon as Enterprise Edition is on, and written to the audit log. They cover the creation, update and deletion of the objects that shape the platform: users, groups, roles and authentication settings; platform parameters; customization such as entity settings, custom fields, rules, retention and decay rules; connectors and ingestion feeds; data sharing feeds and streams; and collaboration objects shared with others. For example, creating a user writes ``creates user `jdoe@example.com` `` in the audit log.

The [audit event reference](/reference/opencti/audit-events/) lists each of these events by area, with its type, scope and message, so that you can pick the right ones for a filter or a trigger.

:::only{opencti=">=7.261127"}
Each run of a [retention policy](/opencti/configure/retention/) that deletes something, or fails to, writes one audit event of type `mutation` and scope `delete`, by the retention manager, such as ``Retention rule `Old activity` deleted `120` `activity` entries, `2` failed``. Its data gives the counts: per entity type for knowledge, with the identifiers of the deleted entries for history and activity.
:::

:::only{opencti=">=7.260811 <7.261127"}
Each run of a [retention policy](/opencti/configure/retention/) that deletes history or activity entries writes one audit event of type `mutation` and scope `delete`, by the retention manager, such as ``Retention rule `Old activity` deleted `120` `activity` entries``, with the identifiers of the deleted entries.
:::

## Extended events

Extended events are recorded only for the users, groups and organizations you [listen to](/opencti/configure/activity/configuration/). Besides the reads, files and commands of the table above, they cover part of the user's own work: the background tasks they launch or delete, the drafts they create or delete, the triggers they create for themselves, the public dashboards they publish or remove, and the dashboards, investigations, playbooks and JSON mappers they duplicate or import. The [audit event reference](/reference/opencti/audit-events/) marks each of them as extended.

Other personal actions are audit events, recorded for everyone: creating, changing or deleting a dashboard or an investigation, a trigger made for someone else, and the changes users make to their own profile. You do not need to listen to a user to see who deleted a dashboard.

## Audit log format

Each audit event is also written as one JSON line to the audit log file and to the console:

```json
{
  "category": "AUDIT",
  "level": "info",
  "message": "creates user `jdoe@example.com`",
  "auth": { "email": "admin@example.com", "ip": "10.0.0.12", "user_id": "…", "socket": "query" },
  "resource": {
    "version": "…",
    "type": "mutation",
    "event_scope": "create",
    "event_access": "administration",
    "data": { "id": "…", "entity_type": "User", "input": {} }
  },
  "timestamp": "2026-01-15T10:00:00.000Z",
  "version": "7.261030.1",
  "source": "backend"
}
```

`level` is `error` for failed actions. `auth` carries the user and the origin of the request, including the HTTP headers listed in `app:audit_logs:trace_request_headers`. Values of the fields listed in `app:app_logs:logs_redacted_inputs`, such as passwords, are replaced by `*** Redacted ***`.

:::only{opencti=">=7.261127"}
A text value of `resource.data` longer than 8,192 characters, such as a long description, is cut there and ends with `[truncated]`, so that the event can still be stored. The **Raw data** of the events screen shows the same cut value. The message itself is not cut.
:::

To choose where the log goes, see [Choose what activity is recorded](/opencti/configure/activity/configuration/#send-activity-to-your-log-pipeline).

## Watch the platform from your SIEM

Once the audit log reaches your SIEM, the platform is one more system the SOC monitors. Match on these fields of the JSON line:

| To detect | Match |
| --- | --- |
| Failed logins, such as password guessing on one account | `resource.type` is `authentication`, `resource.event_scope` is `login` and `level` is `error`. The message names the account, as in ``detects `login failure` for `jdoe@example.com` `` |
| A user trying to go beyond their role | `resource.event_scope` is `unauthorized` |
| Changes to who can do what | `resource.type` is `mutation` and `resource.data.entity_type` is `User`, `Group` or `Role` |
| Where a request came from | `auth.ip`, and the headers listed in `app:audit_logs:trace_request_headers` |

The log carries extended events, such as reads and downloads, only when `app:audit_logs:logs_in_transports` includes `extended`. Inside OpenCTI, [activity triggers](/opencti/configure/activity/triggers/) catch the same events without a SIEM.

## What's next

- [Get alerted on security events](/opencti/configure/activity/triggers/) to be notified of some of these events.
- [Choose what activity is recorded](/opencti/configure/activity/configuration/) to listen to more users.
- The [audit event reference](/reference/opencti/audit-events/) for the full list of events.
