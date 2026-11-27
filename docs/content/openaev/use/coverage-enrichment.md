---
title: Connect OpenAEV to OpenCTI to test security coverages
description: Register an OpenAEV tenant in OpenCTI as the OpenAEV Coverage connector, so that security coverages become scenarios and their results come back to OpenCTI.
doc_type: how-to
products:
  - openaev
  - opencti
personas: [admin, platform-engineer, validation-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/xtm-suite-connector/
---

Once OpenAEV is connected to OpenCTI, a security coverage created in OpenCTI with **Automated using enrichment** becomes an OpenAEV scenario, and the results of its simulations come back to the coverage: which techniques of the threat your defenses prevented and detected. This page sets up the link for one OpenAEV tenant and checks it. What the analysts and the validation engineers then do is in [Measure your defenses against a threat](/opencti/use/analyze/security-coverage/) and [Turn an OpenCTI security coverage into a scenario](/openaev/use/scenarios/from-opencti/).

## How the link works

::diagram{id=openaev-coverage-enrichment-connector}

OpenAEV makes every call. When its configuration holds the address and a token of OpenCTI, each tenant of OpenAEV registers in OpenCTI as an enrichment connector named **OpenAEV Coverage -** followed by the tenant's identifier, with the scope `security-coverage`, and pings it every 40 seconds. The connector enriches automatically: OpenCTI sends it every automated coverage as it is created or refreshed.

| Direction | What travels | Through |
| --- | --- | --- |
| OpenAEV to OpenCTI | The registration and pings of the connector; the link to the scenario; the results of each finished simulation | The OpenCTI API, with the token you set |
| OpenCTI to OpenAEV | The coverage, the threat and its entities, as a STIX bundle | The address `openaev.base-url` followed by `/api/tenants/<tenant id>/stix/process-bundle`, which OpenAEV gives when it registers |
| OpenAEV from OpenCTI | The files of the coverage, to drop them on endpoints | The OpenCTI address, with the same token |

Each tenant has its own connection, so two tenants can test the same threat for two companies, each on its own assets.

## Before you start

- In OpenCTI, a service account that OpenAEV signs in as, in the **Connectors** group: its **Connector** role has **Connectors API usage: register, ping, export push ...** and the rights to write the results. See [Create accounts for integrations](/opencti/administer/service-accounts/). Copy its API token.
- In OpenAEV, the **Tenant identifier** of the tenant to connect, in :menu[Settings > Parameters]. The default tenant is `2cffad3a-0001-4078-b0e2-ef74274022c3`.
- You can change the configuration of OpenAEV and restart it.
- OpenCTI reaches OpenAEV at its `openaev.base-url`, and OpenAEV reaches OpenCTI.

## Connect a tenant

:::steps

1. In the configuration of OpenAEV, set three keys for the tenant, under `openaev.xtm.opencti.<tenant id>`:

   | Key | Value |
   | --- | --- |
   | `enable` | `true` |
   | `url` | The address of OpenCTI, such as `https://opencti.example.com`. OpenAEV builds the links to OpenCTI from it |
   | `token` | The API token of the OpenCTI service account. Pass it through a secret |

   OpenAEV calls the API at `url` followed by `/graphql`. When OpenAEV must call OpenCTI on another address, such as an internal one, set `api-url` to the full GraphQL address, in a properties file or `SPRING_APPLICATION_JSON`: Spring cannot read that key from an environment variable. Every key is in the [configuration reference](/reference/openaev/configuration/#openaev-xtm).

2. Restart OpenAEV. It registers the connector within a minute. If OpenCTI does not answer, it tries again every 5 minutes and logs a warning.

:::

The keys `openaev.xtm.opencti.enable`, `.url` and `.token`, without a tenant identifier, still work: they configure the default tenant. Use the form with the identifier for any other tenant.

:::example[Connect the OpenAEV of Grand Meridian Hotels to the OpenCTI of Example CERT]
Grand Meridian validates its defenses in the default tenant of its OpenAEV, at `https://openaev.example.com`. Example CERT, which serves the hotel chains, runs OpenCTI at `https://opencti.example.com`. In OpenCTI, the administrator creates the service account `OpenAEV Grand Meridian` in the **Connectors** group and keeps its token in the secret `GRAND_MERIDIAN_OPENCTI_TOKEN`. The platform engineer adds to the `openaev` service:

```yaml
services:
  openaev:
    environment:
      - OPENAEV_BASE-URL=https://openaev.example.com
      - OPENAEV_XTM_OPENCTI_2CFFAD3A-0001-4078-B0E2-EF74274022C3_ENABLE=true
      - OPENAEV_XTM_OPENCTI_2CFFAD3A-0001-4078-B0E2-EF74274022C3_URL=https://opencti.example.com
      - OPENAEV_XTM_OPENCTI_2CFFAD3A-0001-4078-B0E2-EF74274022C3_TOKEN=${GRAND_MERIDIAN_OPENCTI_TOKEN}
```

After a restart, OpenCTI lists the connector `OpenAEV Coverage - 2cffad3a-0001-4078-b0e2-ef74274022c3`. On the report "Velvet Harbor targets hotel booking systems", **Automated using enrichment** is now offered, and the coverage the SOC creates becomes a scenario in OpenAEV within seconds.
:::

## Check the connection

1. In OpenCTI, the connector **OpenAEV Coverage -** followed by the tenant identifier is listed among the connectors, and active. See [Follow what connectors import](/opencti/use/ingest/connectors/#follow-what-connectors-import).

   ::screenshot{id=opencti-openaev-coverage-connector alt="The page of the connector OpenAEV Coverage - 2cffad3a-0001-4078-b0e2-ef74274022c3 in OpenCTI, Started: an internal enrichment with the scope Security-Coverage and the automatic trigger, last seen on April 12, 2026."}

2. On a threat in OpenCTI, **Add Security coverage** offers **Automated using enrichment**: it stays greyed while no enrichment connector for security coverage is available.
3. After the first coverage, the scenario appears in :menu[Scenarios] in OpenAEV, and the **Enrichment** panel of the coverage in OpenCTI shows the request as processed.

If the connector does not show, read the logs of OpenAEV: a failed registration logs `OpenCTI connector registration or ping failed`, with the tenant and the reason. A wrong token or an unreachable address are the usual causes. While a tenant has no registered connector, the results of its simulations wait, and leave once it registers.

To stop the link for a tenant, set its `enable` key to `false` and restart OpenAEV.

## What's next

- [Turn an OpenCTI security coverage into a scenario](/openaev/use/scenarios/from-opencti/), to aim the scenarios at the right assets.
- [Connect the modules to each other](/deploy/configure/connect-modules/), for the links between OpenAEV, OpenCTI and XTM One in xtm-docker.
- [Check your defenses against the threats you track](/workflows/threat-informed-validation/), the whole loop across the XTM platform.
