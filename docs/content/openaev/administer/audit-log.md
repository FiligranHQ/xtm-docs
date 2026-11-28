---
title: Keep an audit log of who changed what
description: Turn on the audit log of OpenAEV to record every login, change and refused access with who did it and from where, to the console or to rotated files, and choose what happens when it cannot be written.
doc_type: how-to
products:
  - openaev
edition: ee
personas: [platform-engineer, admin]
owner: docs-openaev
last_reviewed: 2026-11-09
no_screenshot: "The audit log is set in the configuration and written to the log and to files on the host, with no screen: the diagram shows what it records and where."
---

The audit log of OpenAEV records who did what on the platform: every login and logout, every creation, change, deletion and launch, and every access a role refused. Each event says who acted, from which address, in which tenant, on which object, and what changed. Send it to your log collector or keep it in files, to answer an auditor or investigate a change. It is off by default. This page turns it on, says what it records, and chooses what happens when an event cannot be written.

## Before you start

- The platform has an Enterprise Edition license. Without it, a configured audit log stays inactive: the platform shows the banner **Audit logging is inactive: an Enterprise Edition license is required to enable this feature.**, and the platform logs the same warning once. It starts recording as soon as a valid license is applied, without a restart. See [Editions and licensing](/start/editions/).
- You can change the environment of the `openaev` service and restart it. For the `file` transport, you can mount a volume on it.

## What it records

::diagram{id=openaev-audit-log-flow}

| Event | `event_type` and `event_scope` | Recorded |
| --- | --- | --- |
| Logins | `authentication`: `login`, `logout`, `session_expired` | Each login, successful or not, with the login method (`local`, or the identifier of the single sign-on provider), each logout and each session that expires |
| Changes | `mutation`: `create`, `update`, `delete`, `duplicate`, `status_change` | Each change made through the interface or the API, such as a scenario updated or a simulation launched. Reads that succeed are not recorded |
| Refused access | `event_scope` `unauthorized`, `event_status` `error` | Each request a role refused, reads included |
| Execution | `execution`: `scheduled_launch`, `inject_queued`, `inject_status_transition`, `target_resolution`, `agent_trace_step`, `expectation_result`, `coverage_gap` | What the platform does on its own while a simulation runs: injects launched on schedule, sent to their targets, their status, and the expectation results |

Each event is one JSON document with its `timestamp`, `event_status` (`success` or `error`), the `user_id` and `tenant_id`, the `user_metadata` of the request (`user_email`, `ip`, `x_forwarded_for`, `user_agent`, `session_id`) and its `context_data`. For a change, `context_data` names the `entity_type` and the `resource_id`, with the input of the request, the result and the fields that changed.

Secrets never reach the log: a field whose name contains `password`, `secret` or `token`, and the cloud credentials of integrations, are hidden before the event is written. A very large change is cut to `openaev.audit.normalization.max-event-size-bytes`, 32 KB by default, its long texts ending with `...[TRUNCATED]`. Under heavy load, OpenAEV skips this size reduction but still hides the secrets. The keys are in the [openaev.audit block](/reference/openaev/configuration/#openaev-audit).

## Turn it on

`OPENAEV_AUDIT-LOGS_TRANSPORTS` turns the audit log on. It takes a list of transports, separated by commas:

| Transport | Where the events go |
| --- | --- |
| `console` | The output of the platform, as JSON lines next to the other logs, for a log collector that reads the containers' output |
| `file` | A file in `OPENAEV_AUDIT-LOGS_FILE_DIR`, `logs` by default, relative to the folder the platform runs in. Give an absolute path on a volume, so the files survive the container |

Empty, the default, records nothing. `engine` is accepted, but no version sends the audit log to the search engine yet.

The `file` transport writes `audit.log`, named after `OPENAEV_AUDIT-LOGS_FILE_FILENAME`, and rolls it every day, or sooner when it reaches `OPENAEV_AUDIT-LOGS_FILE_ROLLOVER-MAX-SIZE`, `100MB`. Rolled files are named after their date, such as `audit.2026-01-14.0.log`. OpenAEV deletes them after `OPENAEV_AUDIT-LOGS_FILE_RETENTION-DAYS`, 365 days, or earlier when all of them together exceed `OPENAEV_AUDIT-LOGS_FILE_ROLLOVER-TOTAL-SIZE-CAP`, `20GB`: the oldest go first. Set the retention your audit policy asks for, and a size cap large enough to hold it. `AUDIT_LOG_DIR` and `AUDIT_LOG_FILENAME`, read by older versions, still take precedence when they are set. All the keys are in the [openaev.audit-logs block](/reference/openaev/configuration/#openaev-audit-logs).

## Choose what happens when an event cannot be written

By default, a write that fails is logged as a warning and the platform goes on: the action is done, but missing from the audit log.

With `OPENAEV_AUDIT-LOGS_HALT-ON-FAILURE` set to `true`, nothing changes without its audit event. OpenAEV waits for each event to be written before it confirms the change. When the write fails, it cancels the change, signs the user out and stops the platform. Turn it on when your rules forbid any unrecorded change, and watch the disk of the audit folder: a full disk then stops the platform. Each change also takes a little longer, the time of its write.

:::example[Keep a year of audit for Example CERT's platform]
Example CERT hosts the simulations of several hotel chains, and its contracts require a record of every change and every login, kept for 400 days. Its SIEM already collects the output of the containers. The platform engineer turns on both transports, keeps the files on the host, and forbids unrecorded changes:

```yaml
services:
  openaev:
    environment:
      OPENAEV_AUDIT-LOGS_TRANSPORTS: "console,file"
      OPENAEV_AUDIT-LOGS_FILE_DIR: /var/log/openaev-audit
      OPENAEV_AUDIT-LOGS_FILE_RETENTION-DAYS: "400"
      OPENAEV_AUDIT-LOGS_FILE_ROLLOVER-TOTAL-SIZE-CAP: 50GB
      OPENAEV_AUDIT-LOGS_HALT-ON-FAILURE: "true"
    volumes:
      - ./openaev-audit:/var/log/openaev-audit
```

After `docker compose up -d openaev`, the SIEM receives an event when the security engineer of Grand Meridian Hotels updates the **Velvet Harbor replay** scenario, with his email address, his IP address, the tenant of Grand Meridian and the fields he changed. The same event is in `./openaev-audit/audit.log` on the host.
:::

## Check that it works

Log in to OpenAEV, then read the last events:

```bash
docker compose exec openaev tail -n 2 /var/log/openaev-audit/audit.log
```

The last line is your login, with `"event_type":"authentication"`, `"event_scope":"login"` and `"event_status":"success"`. Without any line, check that the platform has an Enterprise Edition license: the banner says so when it has none.

## What's next

- [How access works in OpenAEV](/openaev/concepts/access/), for the roles whose refusals the log records.
- [Configure single sign-on](/openaev/administer/sso/), whose logins the log records with the provider.
- [Troubleshoot the XTM stack](/deploy/operate/troubleshoot/), to read the other logs of the platform.
