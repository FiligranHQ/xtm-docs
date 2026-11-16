---
title: Import compliance readiness from Drata
description: Mirror your ISO 27001 and SOC 2 readiness from Drata into OpenCRQ, so that the controls you price carry the evidence your compliance program already collects.
sidebar_position: 4
doc_type: how-to
products: [opencrq]
personas: [admin, risk-manager]
owner: docs-opencrq
last_reviewed: 2026-11-04
---

Your compliance program already tracks, in Drata, which security controls are in place and backed by evidence. The Drata connector mirrors that readiness into OpenCRQ every day: your ISO 27001:2022 and SOC 2 frameworks with their requirements, and the Drata controls with their state. OpenCRQ then derives the strength and coverage of its own controls from the Drata controls mapped to each, so the price of your scenarios starts from evidence rather than from a declaration. This page connects a Drata account to a tenant. For how controls weigh in the price, read [How OpenCRQ quantifies risk](/opencrq/concepts/how-risk-is-quantified/).

## What the connector imports

- **Two frameworks**: ISO 27001:2022 and SOC 2. Other frameworks of the account, ISO 27001:2013 included, are not imported. Each comes with its readiness as Drata reports it, and its requirements, each with its code, category, whether it is in scope and whether it is ready. An archived requirement is left out.
- **The Drata controls of the account**, each with its code and name, whether it is ready, monitored, backed by evidence and by a policy, and the requirements it serves.

Each run replaces the whole mirror with what Drata says at that moment. Nothing else is written: the connector creates no asset and no finding.

## Where it shows, and what it changes

- :menu[Compliance] lists each framework beside your own assessments, with **Drata** as its **Source** and, as **Progress**, the share of its in-scope requirements that are ready, or **Nothing in scope**. Its page shows the **Requirements** and the **Controls** as Drata reports them, with **Open in Drata**. See [Measure your compliance maturity against a framework](/opencrq/use/defenses/compliance/).
- Each OpenCRQ control that Drata controls are mapped to shows a **Drata evidence** section on its page, titled with how many of them are ready. It lists the mapped controls with their **Code**, the **Attested levels** of strength each one vouches for (**L1** to **L3**; hover a level to read it), their **Reach**, **Org-wide** or **Subset**, and their **State**, **Ready** or **Not ready**. **Not ready** narrows the list to the controls that fail.
- The readiness of the mapped controls sets the **strength** and the **coverage** of the OpenCRQ control, on the axes left automatic. An axis you set by hand is never changed. A strength level holds when one of the Drata controls that attest it is ready, at least three in four of them are, and every level below holds too; a Drata control that fails on a subset only lowers coverage, not strength. When none of the mapped controls attests the base level, L1, the section says so and Drata gives the control no level. For strength, Drata's reading weighs twice as much as the answers of a published assessment that reach the same control, except when Drata's evidence cannot prove a higher level than it reads: answers that rate the control higher then count in Drata's place. For coverage, a reading Drata measured comes first. When a Drata control leaves the mirror, the axis it fed keeps its last value. The scenarios the control protects are then priced again. See [Set how strong your controls are and read how they held](/opencrq/use/defenses/controls/).

## Before you start

- In OpenCRQ, your role has the **Manage Security** capability, and a service account exists for the connector: see [Let integrations sign in with service accounts](/opencrq/administer/service-accounts/).
- In Drata, an API key with read scopes on frameworks and controls. If the key restricts source IP addresses, allow the address OpenCRQ's deployment calls out from.
- The deployment of OpenCRQ carries that key for your tenant, keyed by the tenant's id, the first segment of the address of OpenCRQ once you are signed in to that tenant:

  | Variable | Required | What it holds |
  | --- | --- | --- |
  | `CONNECTORS__DRATA__<tenant-uuid>__API_KEY` | Yes | The Drata API key. Pass it through a secret store |
  | `CONNECTORS__DRATA__<tenant-uuid>__URL` | No | The API of your Drata region: `https://public-api.drata.com` by default, `https://public-api.eu.drata.com` for an account hosted in the EU, `https://public-api.apac.drata.com` in APAC. A key used on the wrong region is rejected |
  | `CONNECTORS__DRATA__<tenant-uuid>__WORKSPACE_ID` | No | The Drata workspace to read, `1` by default. An account without several workspaces always uses `1` |

  Until the key is set, the **Drata** card stays on the **Available** tab of :menu[Integrations] and its page reads **Unavailable**. See the [configuration reference](/reference/opencrq/configuration/#connectors).

## Turn the connector on

:::steps
1. Open :menu[Integrations], stay on the **Deployed** tab, and click the **Drata** card, under **Compliance**. Its state reads **Not configured**.
2. Click **Configure**.
3. Choose the account in **Service account**.
4. Under **Schedule**, keep **Every day** in **Frequency**, at `06:00` in the `UTC` **Time zone**, or change them. **Every 6 hours** is the most often it runs; **Every week** asks for a **Day**, and **Manual only** runs it only when you ask. **Next run** shows when the schedule runs next.

   ::screenshot{id=opencrq-drata-configure alt="The Configure integration panel of the Drata connector: the Service account, and a schedule of every day at 06:00 in the UTC time zone, with the next run on Apr 13, 2026, 06:00 AM (UTC)."}

5. Click **Save**, then **Sync now** to run it without waiting for the schedule.
:::

A run stops after 10 minutes, and a failed run is tried again, up to 3 attempts.

:::example[Bring Grand Meridian's ISO 27001 readiness into its controls]
Grand Meridian Hotels runs its ISO 27001:2022 program in Drata, on an account hosted in the EU. Before the budget review "Security budget 2026", the risk manager wants the controls of Grand Meridian to start from what Drata shows as ready.

The Drata administrator creates an API key with read access to frameworks and controls. The platform engineer sets the entry of Grand Meridian's tenant, `abd0465d-51cd-5234-85b8-cb1cc781124c`, with the EU API:

```yaml
services:
  opencrq:
    environment:
      - CONNECTORS__DRATA__abd0465d-51cd-5234-85b8-cb1cc781124c__API_KEY=${GRAND_MERIDIAN_DRATA_API_KEY}
      - CONNECTORS__DRATA__abd0465d-51cd-5234-85b8-cb1cc781124c__URL=https://public-api.eu.drata.com
```

The workspace stays `1`, the default. In OpenCRQ, the administrator creates the service account `Drata connector`, then in **Configure integration** chooses it, keeps **Every day** at `06:00` `UTC`, saves and clicks **Sync now**.

ISO 27001:2022 appears in :menu[Compliance] with **Drata** as its source. On the page of each mapped control, **Drata evidence** lists the Drata controls behind it, and the controls left automatic take the strength and coverage that evidence gives. Each morning at 06:00 UTC, a Drata control that turned ready raises the control it supports, and the scenarios it protects are priced again.
:::

## Check that it works

1. On the connector page, the state reads **Healthy**, and **Last run**, **Status**, **Duration** and **Processed**, counted in frameworks, describe the run. The **Logs** tab shows what it did.
2. :menu[Compliance] lists the frameworks with **Drata** as their source, and each framework's page gives its **Last synced** date.

A run that fails at once with a rejected key often means the wrong region: check `CONNECTORS__DRATA__<tenant-uuid>__URL`.

## What's next

- [Measure your compliance maturity against a framework](/opencrq/use/defenses/compliance/)
- [Set how strong your controls are and read how they held](/opencrq/use/defenses/controls/)
- [OpenCRQ connectors](/reference/opencrq/connectors/) and the [configuration reference](/reference/opencrq/configuration/#connectors)
