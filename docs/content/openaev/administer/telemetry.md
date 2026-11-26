---
title: Check or stop the usage telemetry sent to Filigran
description: What anonymous usage data OpenAEV sends to Filigran, how and how often, what it never contains, how to see whether it is on, and how to stop it.
doc_type: how-to
products:
  - openaev
personas: [admin, platform-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
no_screenshot: "Usage telemetry has no screen in OpenAEV: the platform sends it in the background, and the diagram shows how."
migrated_from:
  - openaev:/reference/deployment/telemetry/
---

OpenAEV sends anonymous usage telemetry to Filigran: counts and on/off flags that show how the platform is used, so that Filigran can decide what to improve. No content of your scenarios, findings or payloads leaves the platform. This page explains what is sent, how and how often, how to see whether it is on, and how to stop it. It is not about observability metrics, the OpenTelemetry metrics you collect to watch your own deployment: see [Observability metrics](/deploy/operate/metrics/) for those.

## Before you start

- To read whether telemetry is on, your role has **Access tenant settings**.
- To stop it, you control the outbound network of the platform and can restart it.

## How it is sent

::diagram{id=openaev-telemetry-flow}

| | |
| --- | --- |
| Destination | `https://telemetry.oaev.filigran.io/v1/metrics`, over HTTPS |
| Format | OpenTelemetry metrics (OTLP over HTTP, protobuf) |
| Frequency | Counts collected every hour, sent every 6 hours. The first sending comes about 5 hours after the platform starts |
| Start-up check | When the platform starts, it sends one test request to the destination. If the destination does not answer, telemetry stays off until the next restart |

Telemetry is sent for the whole platform, once, with the counts of every tenant together.

## What is sent

Counts and flags, by area:

- **The platform**: its version, unique identifier and creation date, the IP address of the host it runs on, and the deployment tags of `telemetry.tags`, when set.
- **Use**: whether the Enterprise Edition is on, the numbers of users, logins, active tenants, assets, deployed integrations, scenarios, simulations and injects played.
- **AI and the XTM platform**: how often the AI features are used, never what they wrote; whether AI and XTM One are configured; whether a tenant is connected to XTM Hub; which login strategies are on.
- **Content**: the numbers of payloads, teams, assets, findings, vulnerabilities, dashboards and other objects.
- **Results and automation**: expectation traces, security coverage processing, workflow runs, payloads created and emails sent.

A few labels go with the counts. They are the names of the security platforms that AI detection rules were generated for, as your users entered them, and the identifiers of the XTM One agents called, cut to 64 characters. The values of the safety policies of attack chaining go too.

## What is never sent

- No content: no scenario, inject, finding, vulnerability, payload or document, only their numbers.
- No personal data: no user names, email addresses, nor the IP addresses of users.
- No secrets and no configuration values, beyond the on and off flags above.

Filigran uses the data, aggregated, to improve OpenAEV and its user experience, and for internal and external statistics.

## See whether it is on

Open :menu[Settings > Parameters]. In the **Tools** card, the **Telemetry manager** reads **Enable** when the start-up check reached the destination, and **Disabled** when it did not.

## Stop it

No setting turns usage telemetry off. To stop it, block the platform's outbound access to `telemetry.oaev.filigran.io`, at your firewall or by DNS, then restart the platform: its start-up check fails, and it sends nothing until the next restart. Blocking without a restart only makes the sendings fail. A platform without internet access never sends telemetry.

:::example[Tag Example CERT's platform, then stop its telemetry]
Example CERT runs one OpenAEV instance for its member hotel chains. The platform engineer first tags it, so that Filigran can tell it apart from a test platform:

```yaml
services:
  openaev:
    environment:
      TELEMETRY_TAGS: "production,eu-west"
```

The security policy of a member chain later forbids any outbound flow to a vendor. The platform engineer adds `telemetry.oaev.filigran.io` to the deny list of the outbound proxy and runs `docker compose up -d --force-recreate openaev`. After the restart, the **Telemetry manager** of :menu[Settings > Parameters] reads **Disabled**.
:::

## What's next

- [Observability metrics](/deploy/operate/metrics/), to watch your own deployment.
- The [telemetry block](/reference/openaev/configuration/#telemetry) of the configuration reference.
- [Install offline or with FIPS 140-3](/deploy/install/air-gapped/), for a platform without internet access.
