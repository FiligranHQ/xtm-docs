---
title: Check or stop the usage telemetry sent to Filigran
description: What anonymous usage data XTM One sends to Filigran, where and how often, and how to send it to your own collector or turn it off.
doc_type: how-to
products:
  - xtm-one
personas: [admin, platform-engineer]
owner: docs-xtm-one
last_reviewed: 2026-11-08
no_screenshot: Usage telemetry is set in the deployment's environment and has no XTM One screen.
migrated_from:
  - xtm-one:/deployment/telemetry/
---

Like OpenCTI and OpenAEV, XTM One sends anonymous usage telemetry to Filigran: counts, averages, on and off flags, and a few labels. Filigran uses it to improve the platform and to produce aggregated statistics. This page lists what is sent, where and how often, and how to redirect it or turn it off. It is set in the environment of the platform and the worker, with no screen.

::diagram{id=xtm-one-administer-telemetry-flow}

## What is sent

No personal data, no user or agent names, no credentials, and no content of conversations, documents, knowledge bases, prompts or traces. The labels are:

- the platform version, its identifier (a random UUID created at installation) and its creation date;
- the deployment tags you set in `TELEMETRY_TAGS`;
- the edition, and where the Enterprise Edition comes from (an XTM One license or a connected platform);
- the catalog identifiers of the configured integrations, channel bots and model provider types (`github`, `slack_bot`, `openai`), never the names you gave them;
- the format of agent exports and imports;
- the day each daily value belongs to.

Every metric starts with `xtm_one_`:

| Family | Metrics |
| --- | --- |
| Platform and edition | Whether AI is licensed and switched on, whether the Terms of Use are accepted, the number of registered OpenCTI, OpenAEV and OpenCRQ platforms |
| Inventory | Users, active users, groups, agents, company-managed agents, assignments, integrations, channel bots, knowledge bases, MCP servers, custom tools, prompts, skills and model providers, some per type |
| Activity | Assignment runs, conversations, messages, agentic executions and sign-ins, in total and per day over the last 30 days; agent exports and imports |
| Model usage | Cost, tokens, average latency and average feedback score, in total and per day, never per user, agent or conversation |

The metrics carry no IP address; the collector, like any HTTPS server, sees the address the request comes from.

## Where and how often

- **Destination**: `https://telemetry.one.filigran.io/v1/metrics`, with OpenTelemetry (OTLP) over HTTPS. `TELEMETRY_ENDPOINT` sends it to another collector, your own for example.
- **Frequency**: one export every 6 hours per installation, whatever the number of replicas. The first one goes a few minutes after the start, unless one went in the last 6 hours.
- **Startup check**: at start, the platform posts an empty JSON object to the destination. If it cannot be reached or does not answer HTTP 200, telemetry stays off until the next restart.
- Inside a CI/CD pipeline, nothing is sent. A platform without access to the internet never sends telemetry.

## Before you start

- You deploy the platform, and can change its environment and restart it.

## Turn telemetry off

:::steps

1. Set `TELEMETRY_ENABLED=false` in the environment of the platform and the worker.
2. Restart the platform and the worker.
3. Check the platform's startup log: it says `[TELEMETRY] Disabled by TELEMETRY_ENABLED=false - nothing is sent`.

:::

The platform then makes no connection to the collector at all, not even the startup check. Blocking outbound access to `telemetry.one.filigran.io` and restarting has the same effect.

:::example[Tag the Grand Meridian platform, then keep telemetry in-house]
Marc Dupuis deploys XTM One for Grand Meridian Hotels in the Paris datacenter. He tags its telemetry so that Filigran support can tell production from the staging copy:

```bash
TELEMETRY_TAGS=production,eu-west
```

Six months later, Hélène Marchetti asks that no usage data leaves the network. Marc points telemetry at the company's own OpenTelemetry collector, which the monitoring team reads:

```bash
TELEMETRY_ENABLED=true
TELEMETRY_ENDPOINT=https://otel.grand-meridian.example/v1/metrics
```

After the restart, the counts reach `otel.grand-meridian.example` every 6 hours, and nothing reaches `telemetry.one.filigran.io`.
:::

## What's next

- [Usage telemetry](/reference/xtm-one/configuration/#usage-telemetry)
- [Monitor and troubleshoot XTM One](/deploy/operate/xtm-one/)
