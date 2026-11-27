---
title: Check or stop the usage telemetry sent to Filigran
description: What anonymous usage counts OpenCRQ sends to Filigran, how and how often, what it never contains, the log lines that say whether it is on, how to tag it and how to stop it.
sidebar_position: 5
doc_type: how-to
products:
  - opencrq
personas: [admin, platform-engineer]
owner: docs-opencrq
last_reviewed: 2026-11-26
no_screenshot: "Usage telemetry has no screen in OpenCRQ: the instance sends it in the background and says in its log whether it is on, and the diagram shows how it is sent."
---

OpenCRQ sends anonymous usage telemetry to Filigran: counts over the whole instance and one on/off flag, which show how OpenCRQ is used, so that Filigran can decide what to improve. No name, no content of your tenants, assets or risk scenarios, and no figure of any one tenant leaves the instance. This page lists what is sent, how and how often, the log lines that say whether it is on, how to tag your instance, and how to stop it. It is not about observability metrics, the OpenTelemetry metrics you collect to watch your own deployment: see [Observability metrics](/deploy/operate/metrics/) for those.

## Before you start

- OpenCRQ shows nothing about usage telemetry in its interface: you read the log of the `opencrq` container to see its state.
- To tag or stop it, you control the environment of the `opencrq` service and can recreate it, or you control the outbound network of the instance.

## How it is sent

::diagram{id=opencrq-telemetry-flow}

| | |
| --- | --- |
| Destination | `https://telemetry.ocrq.filigran.io/v1/metrics`, over HTTPS |
| Format | OpenTelemetry metrics (OTLP over HTTP, JSON encoding), every value a gauge, with delta temporality |
| Frequency | Counts read from the database at start, then every hour; sent every 6 hours, so the first sending comes about 6 hours after the start, and once more when the instance stops cleanly |
| Start-up check | Once the instance has started, without delaying it, OpenCRQ posts an empty payload to the destination and waits up to 10 seconds for an HTTP 200 answer. Without it, usage telemetry stays off until the next start, with no retry |

One sending covers the whole instance, every tenant together. The image of OpenCRQ sets `NODE_ENV=production`, which sends to the destination above. An instance run without it, from the source code for example, sends to Filigran's test collector, `telemetry.ocrq.staging.filigran.io`, instead.

OpenCRQ runs its work in processes, `http` and `worker` in the default profile: each process starts its own usage telemetry, under the same instance identifier, and writes its own log lines. The counts of events below come from the `http` process, which serves the users.

## What is sent

Every sending carries the identity of the instance, as OpenTelemetry resource attributes:

- `service.name`: always `opencrq-telemetry`.
- `service.version`: the version of OpenCRQ, such as `0.261123.0`.
- `service.instance.id`: a random identifier OpenCRQ creates at its first start and keeps in its database, across restarts and upgrades. It is not linked to your organization.
- `service.instance.creation`: the time of that first start.
- `filigran.telemetry.tags`: the tags of `TELEMETRY__USAGE__TAGS`, only when you set them (see [Tag your instance](#tag-your-instance)).

Then the values, each a total over every tenant of the instance:

- **Edition**: `is_enterprise_edition`, `1` when the instance runs the Enterprise Edition, `0` otherwise.
- **Platform**: `total_users_count`, `service_accounts_count`, `total_tenants_count`, `organizations_count`, `groups_count`, `roles_count` (tenant roles) and `platform_roles_count`.
- **Controls and findings**: `control_capabilities_count`, `control_capability_assessments_count` and `findings_count`.
- **Risk**: `risk_scenarios_count`, `crq_indicator_runs_count`, `loss_drivers_count`, `attack_outcomes_count` and `scenario_loss_estimates_count`.
- **Assets**: `primary_assets_count`, `supporting_assets_count`, `attack_surfaces_count`, and the dependencies between assets, `support_to_primary_links_count` and `support_to_support_links_count`.
- **Integrations**: `integration_settings_count`, the integrations configured, and `integration_<source>_count`, one value per source: `integration_first_count`, `integration_nist_count`, `integration_cisa_count`, `integration_microsoft_graph_count`, `integration_openaev_count` and `integration_drata_count`.

The counts of events since the previous sending. OpenCRQ keeps them in memory: a restart loses the events not sent yet.

- `user_login_count`: the sessions opened through XTM One, the hourly renewals of a session included. No user is named.
- `risk_scenario_created_count`: the risk scenarios created.
- `crq_simulation_launched_count`: the simulations launched on a risk scenario from the interface, the automatic first run of a scenario included. The recomputations OpenCRQ books by itself do not count.
- `connector_sync_launched_count`: the connector runs asked with **Sync now**. The scheduled runs do not count.

A count that fails to read keeps its previous value, and the log shows `Telemetry count refresh failed, keeping previous value`.

## What is never sent

- No names and no content: no user, email address, tenant, asset, risk scenario, control or document, only their numbers.
- No figure of a tenant: no breakdown per tenant, and no identifier of a tenant.
- No user identity behind a login, and no host name or IP address among the attributes.
- No secrets and no configuration values, beyond the edition, the integrations configured and the tags you set.

## See whether it is on

At every start, each process of OpenCRQ writes one of these lines in its log:

| Line | What it means |
| --- | --- |
| `Anonymous usage telemetry started` | Usage telemetry is on. The line gives the destination, the instance identifier, the tags and the interval between sendings, in milliseconds (`21600000`, 6 hours) |
| `Anonymous usage telemetry is disabled by TELEMETRY__USAGE__ENABLED=false, nothing is sent` | You turned it off: OpenCRQ made no connection to Filigran |
| `Telemetry endpoint is not reachable, anonymous usage telemetry is disabled` | The start-up check failed, as on an instance without internet access: nothing is sent until the next start |
| `Failed to start anonymous usage telemetry` | An error stopped it at start, given under the line: nothing is sent until the next start |

To find them in the reference deployment:

```bash
docker compose logs opencrq | grep -i "usage telemetry"
```

## Tag your instance

`TELEMETRY__USAGE__TAGS` gives the instance tags, separated by commas, sent with every value, so that Filigran can tell a production instance from a test one, or one region from another. It is empty by default. OpenCRQ trims the tags, puts them in lowercase, removes the duplicates and sorts them: `Production, EU-West` is sent as `eu-west,production`. Set it in the environment of the `opencrq` service, then recreate the container.

Earlier builds gave this variable another name, which now stops the instance at start: see the [Telemetry](/deploy/configure/opencrq/#telemetry) settings of OpenCRQ for the names to change.

## Stop it

Set `TELEMETRY__USAGE__ENABLED` to `false` in the environment of the `opencrq` service, then recreate the container with `docker compose up -d`. OpenCRQ then makes no connection to Filigran at all, not even the start-up check, and logs `Anonymous usage telemetry is disabled by TELEMETRY__USAGE__ENABLED=false, nothing is sent`. It defaults to `true`, and xtm-docker does not set it. To turn usage telemetry back on, remove the variable and recreate the container.

Blocking the outbound access of the instance to `telemetry.ocrq.filigran.io`, at your firewall or by DNS, then restarting, stops it too: the start-up check fails. Blocking without a restart only makes the sendings fail. An instance without internet access never sends usage telemetry.

:::example[Tag Example CERT's instance, then stop its telemetry]
Example CERT runs OpenCRQ with xtm-docker, for Grand Meridian Hotels and its other member hotel chains. Its platform engineer first tags the instance, so that Filigran can tell it apart from the CERT's test instance. In `docker-compose.override.yml`, under the `opencrq` service:

```yaml
    environment:
      - TELEMETRY__USAGE__TAGS=Production,EU-West
```

After `docker compose up -d`, `docker compose logs opencrq | grep -i "usage telemetry"` shows, for each of the `http` and `worker` processes, a line `Anonymous usage telemetry started` with the endpoint `https://telemetry.ocrq.filigran.io/v1/metrics`, the tags `eu-west` and `production`, and the interval `21600000`.

The security policy of Grand Meridian Hotels later forbids any outbound flow to a vendor from the instance that holds its figures. The platform engineer adds a line under the same `environment`:

```yaml
      - TELEMETRY__USAGE__ENABLED=false
```

After `docker compose up -d`, each process logs `Anonymous usage telemetry is disabled by TELEMETRY__USAGE__ENABLED=false, nothing is sent`, and the instance no longer contacts `telemetry.ocrq.filigran.io`.
:::

## What's next

- [Observability metrics](/deploy/operate/metrics/), to watch your own deployment.
- The [Telemetry](/deploy/configure/opencrq/#telemetry) settings of OpenCRQ, and the [telemetry block](/reference/opencrq/configuration/#telemetry) of the configuration reference.
- [Run several tenants in one platform](/opencrq/administer/tenants/), whose counts the usage telemetry adds up.
