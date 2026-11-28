---
title: Connect a tenant to XTM Hub
description: Connect a tenant of OpenAEV to your organization in XTM Hub to deploy ready-made scenarios and threat arsenal actions in one click, and keep the connection alive.
doc_type: how-to
products:
  - openaev
  - xtm-hub
personas: [admin]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/administration/hub/
---

XTM Hub is Filigran's place for ready-made content, trials and resources of the XTM platform. Once a tenant of OpenAEV is connected to your organization in XTM Hub, you deploy its scenarios and threat arsenal actions into that tenant in one click. This page connects a tenant, checks the connection and disconnects it. Each tenant connects on its own, to the XTM Hub organization you choose.

## Before you start

- Your role has **Manage tenant settings** in the tenant. The **Admin** group of a tenant has it.
- You have an account in XTM Hub, in the organization to connect to.
- Both your browser and the platform reach XTM Hub, `https://hub.filigran.io` unless `openaev.xtm.hub.url` says otherwise. The integration is on by default, `openaev.xtm.hub.enable`. See the [openaev.xtm block](/reference/openaev/configuration/#openaev-xtm) of the configuration reference.

## Connect the tenant

::diagram{id=openaev-xtm-hub-connection}

:::steps
1. In the tenant to connect, open :menu[Settings > Filigran Experience]. The **XTM Hub** card reads **Not connected**.
2. Click **Connect to XTM Hub**. A dialog explains the steps; click **Continue**. XTM Hub opens in a new browser tab.
3. In XTM Hub, log in and choose the organization to connect the tenant to.
4. Confirm. Back in OpenAEV, a message says **Your OpenAEV platform is successfully connected**.
:::

::screenshot{id=openaev-xtm-hub-card alt="The XTM Hub card of Filigran Experience, Not connected, with its four tiles, XTM Platform free trial, Pre-built content, XTM Platform Roadmap and Academy, and the Explore XTM Hub and Connect to XTM Hub buttons."}

Keep the dialog open while you work in XTM Hub: closing it stops the connection. The card now reads **Connected**, with the **Connection status**, the **Connection date** and who it was **Connected by**, and offers **Go to the Hub**.

**Connect to XTM Hub** stays greyed out while your browser cannot reach XTM Hub, when the platform could not reach it at its last start or `openaev.xtm.hub.enable` is `false`, and for users without **Manage tenant settings**. The platform tests XTM Hub only when it starts: once XTM Hub is reachable from it, restart the platform.

## Keep the connection alive

OpenAEV checks the connection when an administrator of the tenant opens :menu[Settings > Filigran Experience]. Each check also sends the edition of the platform, Community or Enterprise, so XTM Hub shows the right one after a license change, without connecting again.

To check every connection in the background, the platform engineer turns on `openaev.xtm.hub.collector.enable`, off by default. OpenAEV then checks every connected tenant every hour, `openaev.xtm.hub.collector.connectivity-check-interval` in seconds. A failed check turns the card to **Connectivity lost**. When the connection stays lost for more than 24 hours, OpenAEV emails the administrators of the tenant once, unless `openaev.xtm.hub.connectivity-email-enable` is `false`. If the connection does not come back by itself, disconnect the tenant and connect it again.

## See the plan and usage of a cloud platform

When Filigran runs your platform in its cloud (SaaS), each connection check tells OpenAEV so. The **XTM Hub** card of a connected tenant then adds a block, **Your OpenAEV, by the numbers**, that points to the figures XTM Hub keeps for the platform:

| Item | What XTM Hub shows |
| --- | --- |
| **Your plan** | What you subscribed to |
| **Contract countdown** | The days left before renewal |
| **Storage & uptime** | The database space your data takes, and the uptime your service level agreement (SLA) guarantees |

**See my OpenAEV metrics** opens, in a new browser tab, the page of this tenant's registration on XTM Hub, or the home page of XTM Hub when it cannot find the registration. The figures stay in XTM Hub: OpenAEV shows none of them. A self-hosted platform does not show the block. When XTM Hub cannot be reached, the card keeps what the last check said.

## Disconnect the tenant

In :menu[Settings > Filigran Experience], click **Disconnect XTM Hub** and confirm in XTM Hub. The content you deployed stays in the tenant. You can connect again at any time.

:::example[Connect Grand Meridian's tenant to its XTM Hub organization]
Grand Meridian Hotels wants the ready-made phishing and ransomware scenarios of XTM Hub in its own tenant of Example CERT's platform. The administrator of the tenant, working in the Grand Meridian Hotels tenant, opens :menu[Settings > Filigran Experience] and clicks **Connect to XTM Hub**, then **Continue**. In XTM Hub, she logs in with her Grand Meridian account and chooses the organization `Grand Meridian Hotels`. The card reads **Connected**. A scenario deployed from XTM Hub now lands in the Grand Meridian Hotels tenant only: Example CERT's own tenant, connected to Example CERT's organization, does not see it.

The platform engineer of Example CERT adds, so that a lost connection does not go unnoticed:

```yaml
services:
  openaev:
    environment:
      OPENAEV_XTM_HUB_COLLECTOR_ENABLE: "true"
```
:::

## What's next

- [Build a scenario](/openaev/use/scenarios/build/), from scratch or from XTM Hub.
- [Give each organization its own tenant](/openaev/administer/multi-tenancy/).
- [Editions and licensing](/start/editions/).
