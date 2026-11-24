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
- **The markings of each asset**, when OpenAEV enforces markings, that is when its `MARKING` preview feature is on. OpenCRQ copies the marking definitions the OpenAEV user is cleared for, and a member reads an asset only when cleared for all its markings: see [Limit what people see with markings](/opencrq/administer/users-and-roles/#limit-what-people-see-with-markings). A marking OpenCRQ cannot match hides the asset from everyone but the super administrator. With markings off in OpenAEV, its default, every asset comes unmarked, and every member of the tenant reads it, as every user of OpenAEV does.

Two things are not imported:

- **The posture score** of an asset. OpenCRQ computes the **OpenAEV Posture** of an asset's page from what the sync stored: how many of its OpenAEV findings held at their latest test, and how many found it exposed. It never asks OpenAEV live, and the tile stays empty until an OpenAEV test reports on the asset.
- **Credentials of AI targets**: their token and configuration are never read into OpenCRQ.

A machine that OpenAEV and Microsoft Defender both report becomes one asset in OpenCRQ: the OpenAEV agent of a Defender executor carries the Defender device id. See [Import assets from Microsoft Defender](/opencrq/use/sources/microsoft-defender/).

## How often it runs

The connector is a job of the tenant, run **Every hour** by default. You can set another schedule, down to every hour, or **Manual only**, and **Sync now** runs it at any time. With OpenAEV's default settings, each run:

1. reads the whole inventory of OpenAEV again,
2. reads the CVE findings updated since the last completed run, never more than 7 days back,
3. reads the simulations finished since the last completed run, the oldest first, never more than 7 days back. The first run therefore reads the CVE findings and the simulations of the last 7 days, not older ones,
4. retires what a newer simulation of the same scenario no longer reports on an asset it played: the finding is dismissed as **No longer reported by OpenAEV**, and comes back by itself if OpenAEV reports it again,
5. fetches, within a budget of 500 calls, the description of the assets OpenAEV serves one by one, such as web applications and identities. A large estate is described over several runs.

A run stops after 30 minutes and the next one resumes where it stopped. A run that fails is tried again, up to 3 attempts. A result on an asset the inventory did not return is dropped, not imported on an asset created on the spot.

When OpenAEV serves its snapshot export, the connector reads it instead, at the same schedule: see [Read the snapshot export of OpenAEV](#read-the-snapshot-export-of-openaev).

## Before you start

- In OpenCRQ, your role has the **Manage Security** capability, to change the connector's account or schedule.
- In OpenAEV, a user for OpenCRQ whose role has these capabilities:
  - **Access assets**, to read the inventory. Without it, the run fails.
  - **Access findings**, to read the CVEs. Without it, the run brings no CVE, and the next run reads the same window again.
  - **Access assessment**, so that it sees every scenario and simulation of the OpenAEV tenant. Without it, the user sees only the simulations it was granted, and no result arrives for the others, with no error to say why.

  When OpenAEV serves its snapshot export, the role needs another capability: see [Read the snapshot export of OpenAEV](#read-the-snapshot-export-of-openaev).
- The deployment of OpenCRQ carries the OpenAEV URL and that user's token for your tenant:

  | Variable | What it holds |
  | --- | --- |
  | `CONNECTORS__OPENAEV__<tenant-uuid>__URL` | The base URL of the OpenAEV platform |
  | `CONNECTORS__OPENAEV__<tenant-uuid>__TOKEN` | The API token of the OpenAEV user. Pass it through a secret store |

  The tenant's id is the first segment of the address of OpenCRQ once you are signed in to that tenant. Set both together, with a valid URL: with one alone, OpenCRQ does not start. Until the deployment carries them, the connector page shows **Unavailable** and says **Add this connector to your deployment configuration to use it.** See the [configuration reference](/reference/opencrq/configuration/#connectors).

## Connect OpenAEV to your tenant

Once the variables are set, OpenCRQ binds the connector at its next start, under a service account it creates, `OpenAEV (deployment)`, in the **Connectors** group. The **OpenAEV** card, under **Simulation** on the **Deployed** tab of :menu[Integrations], then reads **Never synced**, and the connector runs **Every hour**. An account you bound yourself is never replaced at start.

To change the account or the schedule:

:::steps
1. Open :menu[Integrations], stay on the **Deployed** tab, and click the **OpenAEV** card, under **Simulation**.
2. Click **Configure**.
3. Under **Service account**, keep the account the connector writes the findings as, such as `OpenAEV (deployment)`, or choose another: **Use an existing service account** picks one of the tenant's accounts, and **Create a new service account** creates one with the integration, in the **Connectors** group, `OpenAEV connector` by default.
4. Under **Schedule**, keep **Every hour** in **Frequency**, or choose another. **Next run** shows when the schedule runs next.

   ::screenshot{id=opencrq-openaev-configure alt="The Configure integration panel of the OpenAEV connector: the Service account to choose, and the Schedule with Every hour as the frequency and the next run on Apr 12, 2026, 11:00 AM (UTC)."}

5. Click **Save**. The state reads **Never synced**.
6. Click **Sync now** to run it without waiting for the schedule. The state reads **Syncing**, then **Healthy**.
:::

:::example[Price the front desks on the latest phishing simulation]
Grand Meridian's validation team simulates Velvet Harbor's phishing and credential theft on the front-desk workstations of its French hotels, in an OpenAEV platform at `https://openaev.example.com`. The risk manager wants the data breach on guest records priced on what those results show, not on declared strengths alone.

In OpenAEV, the administrator gives a user `OpenCRQ reader` a role with **Access assets**, **Access findings** and **Access assessment**, and generates its token. The platform engineer sets the entry of Grand Meridian's tenant, `01a043da-ff47-72c2-aaa4-7da56d85c83e`:

```yaml
services:
  opencrq:
    environment:
      - CONNECTORS__OPENAEV__01a043da-ff47-72c2-aaa4-7da56d85c83e__URL=https://openaev.example.com
      - CONNECTORS__OPENAEV__01a043da-ff47-72c2-aaa4-7da56d85c83e__TOKEN=${GRAND_MERIDIAN_OPENAEV_TOKEN}
```

After the restart, the **OpenAEV** card reads **Never synced**, bound to `OpenAEV (deployment)`. The administrator opens it and clicks **Sync now**.

The first run imports the workstations OpenAEV knows, the CVEs it found or updated on them in the last 7 days, and the simulations finished in the last 7 days. Each technique the EDR did not prevent, or did not detect, becomes a finding on the workstation it reached. On those techniques, the EDR's **Engine efficacy** falls below its declaration, by as much as the tests fell short and in proportion to the share of the scope's hosts they ran on. The scenarios that reach the front desks are priced again on that efficacy. Every hour after that, a new simulation of the same scenario updates the findings.
:::

## Check that it works

1. On the connector page, the state reads **Healthy** and the **Overview** tab shows the **Last run**, its **Status**, its **Duration** and the number **Processed**.
2. The **Works** tab lists each run, and the **Logs** tab what it did.
3. :menu[Assets] lists the assets of OpenAEV, and :menu[Findings] the results and the CVEs.

If no result arrives while the inventory does, check that the user's role has **Access assessment**: without it, OpenAEV returns no simulation and no error.

## Start over

**Reset state**, in the **More actions** menu of the connector page, makes the next run read again everything the connector can see, instead of what changed since the last run: the CVE findings and the simulations of the last 7 days, or the whole snapshot export when OpenAEV serves it. Nothing already imported is removed. It requires the **Manage Security** capability.

## Read the snapshot export of OpenAEV

When OpenAEV's `BULK_SNAPSHOT_EXPORT` preview feature is on (`openaev.enabled-dev-features=BULK_SNAPSHOT_EXPORT` in OpenAEV's configuration), OpenAEV serves a snapshot export of its results and CVEs. The connector then reads that export instead of the simulations and the findings. Nothing is set in OpenCRQ: each run asks OpenAEV for the export first, so turning the feature on or off in OpenAEV takes effect at the next run.

The role of the OpenAEV user then needs:

- **Access observation snapshots**, to read the export. OpenAEV shows this capability in its role screen only while the feature is on. Without it, the run fails at once, without a new attempt, and says `OpenAEV refused the snapshot request: the account lacks the ACCESS_SNAPSHOT_OBSERVATION capability`.
- **Access assets** and **Access assessment**, as in [Before you start](#before-you-start). **Access findings** is not used.

Each run then:

1. rebuilds the inventory once a day, not at every run. With it, the connector imports the asset groups of OpenAEV with their members, and the attack paths of its simulations: see [Find the assets of a group](/opencrq/use/attack-surface/assets/#find-the-assets-of-a-group) and [Read access relations and events](/opencrq/use/attack-surface/assets/#read-access-relations-and-events),
2. reads the results and the CVEs that changed since the previous run, from where it stopped, with no 7-day window,
3. reads the whole export again every 7 days, and retires then the assets, scenarios and techniques OpenAEV deleted.

The first run, and the first run after **Reset state**, read the whole export: results and CVEs older than 7 days arrive too. On a large estate, this takes several runs.

## What's next

- [Set how strong your controls are and read how they held](/opencrq/use/defenses/controls/)
- [Review the findings on your assets](/opencrq/use/attack-surface/findings/)
- [Build the attack surface from your assets](/opencrq/use/attack-surface/assets/)
- [OpenCRQ connectors](/reference/opencrq/connectors/) and the [configuration reference](/reference/opencrq/configuration/#connectors)
