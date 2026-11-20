---
title: Usage telemetry and privacy
description: The anonymous usage data OpenCTI sends to Filigran, how and how often it is sent, what is never collected, and how to stop it.
sidebar_position: 70
doc_type: reference
products:
  - opencti
personas: [admin, platform-engineer, evaluator]
owner: docs-opencti
no_screenshot: "Usage telemetry has no screen in OpenCTI: it is sent in the background by the telemetry manager."
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/reference/usage-telemetry/
---

OpenCTI sends anonymous usage telemetry to Filigran: counts and on/off flags that show how the platform is used, so Filigran can decide what to improve. No intelligence and no personal data leave the platform. This page explains what is sent, how, how to see it yourself, and how to stop it. It is not about observability metrics, the OpenTelemetry metrics you collect to monitor your own deployment: see [Metrics](/deploy/operate/metrics/) for those.

## What is never collected

- No threat intelligence: no names, descriptions, indicators, files or observables. Only counts.
- No personal data: no user names, email addresses or IP addresses.
- No configuration values: connectors are reported by name or catalog identity, never with their settings or the host of their registry.

Filigran uses the data, aggregated and anonymized, to improve the product and the user experience, and for internal and external statistics.

## How it is sent

::diagram{id=opencti-telemetry-flow}

| | |
| --- | --- |
| Destination | `https://telemetry.filigran.io`, over HTTPS |
| Format | OpenTelemetry metrics (OTLP over HTTP) |
| Frequency | Every 6 hours, with data points collected every hour |
| Sender | The **Telemetry manager** of the platform, on one node of a cluster at a time |
| Local copy | The same data, in the `telemetry` folder of the platform's working directory: three rotating files of up to 1 MB each. [Support packages](/deploy/operate/support-package/) include them |

The platform tests the destination once, when its telemetry manager starts. If it cannot reach it, nothing is sent until the next restart; the local copy is still written.

## See what is sent

Read the files of the `telemetry` folder. To follow the exports as they happen, set `telemetry_manager:console_debug` (`TELEMETRY_MANAGER__CONSOLE_DEBUG`) to `true` and restart: the platform also prints each export to its console log, every 6 hours.

## Turn it off

::::only{opencti=">=7.261030"}
Set `telemetry_manager:enabled` (`TELEMETRY_MANAGER__ENABLED`) to `false` and restart the platform. The telemetry manager no longer starts: nothing is sent and no local copy is written. With Docker Compose:

```yaml
services:
  opencti:
    environment:
      TELEMETRY_MANAGER__ENABLED: "false"
```

To check, open :menu[Settings > Parameters]: in the **Managers** card, under **Telemetry and Filigran ecosystem**, the **Telemetry manager** reads **Disabled**.

To keep the local copy but send nothing, leave the manager on and block outbound traffic from the platform to `telemetry.filigran.io` instead, then restart.
::::

::::only{opencti="<7.261030"}
No configuration key turns usage telemetry off in this version. To stop sending, block outbound traffic from the platform to `telemetry.filigran.io`, then restart the platform: it no longer reaches the destination when the telemetry manager starts, and only writes the local copy.
::::

## Data collected

The data falls into a few groups: the platform itself (version, unique identifier, number of nodes, Enterprise Edition on or off), users and access (numbers of users, groups, roles and organizations, which authentication strategies are on), integrations and ingestion (active connectors by catalog identity, ingesters, synchronizers), drafts, workbenches and forms, knowledge and features (counts of objects by type, playbooks, triggers, custom fields), and AI features (numbers of requests, which provider is configured).

:::only{opencti=">=7.260811"}
When the operator sets `telemetry_manager:tags`, such as `saas,eu-west`, the deployment tags are sent too, lowercased and sorted.
:::

:::only{opencti=">=7.261119"}
The number of investigations is sent too, as `opencti_investigations_count`: every investigation of the platform counts, whoever it is shared with. Only the number leaves the platform, never their names or content.
:::

The [Usage telemetry sent to Filigran](/reference/opencti/metrics/#usage-telemetry-sent-to-filigran) section of the metrics reference lists every value your version sends. Earlier versions send fewer.

## What's next

- [Metrics](/deploy/operate/metrics/) to monitor your own deployment.
- The [metrics reference](/reference/opencti/metrics/) for the observability metrics and the usage telemetry of your version.
- [Engines, Schedules and Managers](/reference/opencti/configuration/#engines-schedules-and-managers) in the configuration reference, for the `telemetry_manager` keys of your version.
