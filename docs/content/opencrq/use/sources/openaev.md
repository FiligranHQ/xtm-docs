---
title: Import OpenAEV results
description: Bring the assets OpenAEV knows and the outcome of its simulations into OpenCRQ, so that your controls are priced on how they held when tested.
sidebar_position: 2
doc_type: how-to
products: [opencrq, openaev]
personas: [admin, validation-engineer, risk-manager]
owner: docs-opencrq
last_reviewed: 2026-11-04
---

A control declared strong is an assumption until an attack tests it. The OpenAEV connector imports the outcome of the simulations OpenAEV ran on your assets: which technique was prevented or detected, by which kind of security platform, and which CVEs OpenAEV found. OpenCRQ lands them as findings on your assets and audits your controls with them, technique by technique: where a control held less often than you declared, the price of a scenario reads it weaker. This page connects an OpenAEV platform to a tenant and says what comes in. For the model behind the price, read [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/).

## What the connector imports

- **The asset inventory of OpenAEV**: every asset it knows, whatever its category and whether or not a simulation targeted it, except its security platforms, which detect rather than get attacked. Each comes with its name, category, criticality, internet exposure and tags; an endpoint with its hostname, addresses, platform and architecture, and the cloud provider, type and region when it has them; an AI target with its provider, endpoint, model and modality.
- **The results of finished simulations**, per asset, scenario, technique and expectation (prevention or detection), one per type of security platform that reported on it, such as an EDR or a SIEM. A technique a platform type did not stop counts as exposed. Each result keeps the link to its simulation and its inject. A simulation that is scheduled, running, paused or canceled brings nothing yet.
- **The CVEs OpenAEV found on each asset**, from its findings.

Three things are not imported:

- **The markings** of an asset. It comes unmarked, so every member of the tenant reads it, as every user of OpenAEV does: see [Limit what people see with markings](/opencrq/administer/users-and-roles/#limit-what-people-see-with-markings).
- **The posture score** of an asset. OpenCRQ reads it from OpenAEV each time the asset's page opens, and shows nothing when OpenAEV does not answer within 10 seconds. The sync never depends on it.
- **Credentials of AI targets**: their token and configuration are never read into OpenCRQ.

A machine that OpenAEV and Microsoft Defender both report becomes one asset in OpenCRQ: the OpenAEV agent of a Defender executor carries the Defender device id. See [Import assets from Microsoft Defender](/opencrq/use/sources/microsoft-defender/).

## How often it runs

The connector is a job of the tenant, run **Every hour** by default. You can set another schedule, down to every hour, or **Manual only**, and **Sync now** runs it at any time. Each run:

1. reads the whole inventory of OpenAEV again,
2. reads the CVE findings,
3. reads the simulations finished since the last completed run, the oldest first, never more than 7 days back. The first run therefore reads the simulations of the last 7 days, not older ones,
4. retires what a newer simulation of the same scenario no longer reports on an asset it played: the finding is dismissed as **No longer reported by OpenAEV**, and comes back by itself if OpenAEV reports it again,
5. fetches, within a budget of 500 calls, the description of the assets OpenAEV serves one by one, such as web applications and identities. A large estate is described over several runs.

A run stops after 30 minutes and the next one resumes where it stopped. A run that fails is tried again, up to 3 attempts. A result on an asset the inventory did not return is dropped, not imported on an asset created on the spot.

## Before you start

- In OpenCRQ, your role has the **Manage Security** capability, to configure the connector, and a service account exists for it: see [Let integrations sign in with service accounts](/opencrq/administer/service-accounts/).
- In OpenAEV, a user for OpenCRQ whose role has the **Access findings** capability, to read the CVEs, and the **Access assessment** capability, so that it sees every scenario and simulation of the OpenAEV tenant. Without **Access assessment**, it sees only those it was granted, and no result arrives for the others, with no error to say why.
- The deployment of OpenCRQ carries the OpenAEV URL and that user's token for your tenant:

  | Variable | What it holds |
  | --- | --- |
  | `CONNECTORS__OPENAEV__<tenant-uuid>__URL` | The base URL of the OpenAEV platform |
  | `CONNECTORS__OPENAEV__<tenant-uuid>__TOKEN` | The API token of the OpenAEV user. Pass it through a secret store |

  The tenant's id is the first segment of the address of OpenCRQ once you are signed in to that tenant. Until both are set, the connector page shows **Unavailable** and says **Add this connector to your deployment configuration to use it.** See the [configuration reference](/reference/opencrq/configuration/#connectors).

## Connect OpenAEV to your tenant

:::steps
1. Open :menu[Integrations], stay on the **Deployed** tab, and click the **OpenAEV** card, under **Simulation**. Its state reads **Not configured**.
2. Click **Configure**.
3. Choose the account in **Service account**. The connector writes the findings as this account.
4. Under **Schedule**, keep **Every hour** in **Frequency**, or choose another. **Next run** shows when the schedule runs next.

   ::screenshot{id=opencrq-openaev-configure alt="The Configure integration panel of the OpenAEV connector: the Service account to choose, and the Schedule with Every hour as the frequency and the next run on Apr 12, 2026, 11:00 AM (UTC)."}

5. Click **Save**. The state reads **Never synced**.
6. Click **Sync now** to run it without waiting for the schedule. The state reads **Syncing**, then **Healthy**.
:::

:::example[Price the front desks on the latest phishing simulation]
Grand Meridian's validation team simulates Velvet Harbor's phishing and credential theft on the front-desk workstations of its French hotels, in an OpenAEV platform at `https://openaev.example.com`. The risk manager wants the data breach on guest records priced on what those results show, not on declared strengths alone.

In OpenAEV, the administrator gives a user `OpenCRQ reader` a role with **Access findings** and **Access assessment**, and generates its token. The platform engineer sets the entry of Grand Meridian's tenant, `abd0465d-51cd-5234-85b8-cb1cc781124c`:

```yaml
services:
  opencrq:
    environment:
      - CONNECTORS__OPENAEV__abd0465d-51cd-5234-85b8-cb1cc781124c__URL=https://openaev.example.com
      - CONNECTORS__OPENAEV__abd0465d-51cd-5234-85b8-cb1cc781124c__TOKEN=${GRAND_MERIDIAN_OPENAEV_TOKEN}
```

In OpenCRQ, the administrator creates the service account `OpenAEV connector`, then in **Configure integration** chooses it in **Service account**, keeps **Every hour**, saves, and clicks **Sync now**.

The first run imports the workstations OpenAEV knows and the simulations finished in the last 7 days. Each technique the EDR neither prevented nor detected becomes a finding on the workstation it reached. On those techniques, the EDR's **Engine efficacy** falls below its declaration, by as much as the tests fell short and in proportion to the share of the scope's hosts they ran on. The scenarios that reach the front desks are priced again on that efficacy. Every hour after that, a new simulation of the same scenario updates the findings.
:::

## Check that it works

1. On the connector page, the state reads **Healthy** and the **Overview** tab shows the **Last run**, its **Status**, its **Duration** and the number **Processed**.
2. The **Works** tab lists each run, and the **Logs** tab what it did.
3. :menu[Assets] lists the assets of OpenAEV, and :menu[Findings] the results and the CVEs.

If no result arrives while the inventory does, check that the user's role has **Access assessment**: without it, OpenAEV returns no simulation and no error.

## Start over

**Reset state**, in the **More actions** menu of the connector page, makes the next run read again everything the connector can see, instead of what changed since the last run: the simulations of the last 7 days. Nothing already imported is removed. It requires the **Manage Security** capability.

## What's next

- [Set how strong your controls are and read how they held](/opencrq/use/defenses/controls/)
- [Review the findings on your assets](/opencrq/use/attack-surface/findings/)
- [Build the attack surface from your assets](/opencrq/use/attack-surface/assets/)
- [OpenCRQ connectors](/reference/opencrq/connectors/) and the [configuration reference](/reference/opencrq/configuration/#connectors)
