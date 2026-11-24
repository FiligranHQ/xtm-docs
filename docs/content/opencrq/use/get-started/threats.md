---
title: Import the threats that target your company
description: Load OpenCRQ's default threats, connect an OpenCTI Live Stream or analyze a STIX bundle in the Threats step of Get started, so your scenarios have attackers to simulate.
sidebar_position: 4
doc_type: how-to
products: [opencrq]
personas: [risk-manager, admin]
owner: docs-opencrq
last_reviewed: 2026-11-20
---

**Threats** is the third step of :menu[Get started]. The attacker groups that target your sector and your country, and their techniques, drive the simulated attacks: without a threat, no scenario can be computed. The step brings threats in from three sources, which add up: OpenCRQ's default threats, an OpenCTI Live Stream, and the Structured Threat Information Expression (STIX) bundles you analyze.

## Before you start

- You are a member of the tenant.
- To switch the default threats on or off, or to bind OpenCTI to the tenant, you have **Manage Security**.
- To connect OpenCTI, a platform administrator has set its URL and its token for the tenant. See [Give OpenCRQ the URL and the token](/opencrq/use/sources/opencti/#give-opencrq-the-url-and-the-token).
- To analyze a bundle, the platform has file storage. See [Set the platform's parameters and file storage](/opencrq/configure/parameters/).

## The threat sources

Under **Threat sources**, the step reads "Use as many as you need, they add up." Beside it, the count of the tenant's threats you can see, and **See all threats**, which opens the [Threat Center](/opencrq/use/threats/threat-center/).

::screenshot{id=opencrq-get-started-threats alt="The Threats step of Get started for Grand Meridian Hotels: under Threat sources, OpenCRQ Default Threats Never synced with Enable, OpenCTI with the threats its stream brought, and STIX bundle with Analyze new bundle."}

| Tile | What it brings | What it shows |
| --- | --- | --- |
| **OpenCRQ Default Threats** | Threat groups and campaigns derived from MITRE ATT&CK, so that scenarios have attackers before any feed is connected | The state of the connector and the number of threats it loaded. A new tenant starts with it switched off: it reads **Never synced**, and its hint says "Switched off." **Enable** or **Disable** switches it |
| **OpenCTI** | The threats of an OpenCTI Live Stream, kept current as analysts change them in OpenCTI | The state of the connector: **Unavailable**, **Not configured**, **Never synced**, **Syncing**, **Healthy** or **Failing**. Then the number of threats it brought, and the progress of its replay in percent. **Configure** opens it |
| **STIX bundle** | The threats of the bundle files you analyze | The number of threats imported from bundles, and **Active** once there is one. **Analyze new bundle** picks a file |

The step is done as soon as you can see one threat of the tenant, whatever its source. While OpenCTI replays its stream or the default threats load, the **Threats** step turns in the stepper.

## Load the default threats

On the **OpenCRQ Default Threats** tile, select **Enable**. The tile reads **Syncing** while the threats load, then **Healthy**, with their count. What the tile offers depends on where the connector stands:

- Without **Manage Security**, a switched-off tile reads "An administrator enables it."
- Once on, **Disable** asks for a confirmation that says how many of its threats leave the tenant, and how many from your other sources stay. Your scenarios are then simulated again without them. See [Retire the default threats](/opencrq/use/sources/opencti/#retire-the-default-threats).
- When the first load fails, the tile reads **Failing**, and **Follow in Integrations** opens the connector in :menu[Integrations].
- When the platform turned the default threats off for every tenant, the tile offers no action. See the `CONNECTORS__DEFAULT_THREATS__DATASET_URL` variable in the [configuration reference](/reference/opencrq/configuration/#connectors).

## Connect OpenCTI

Select **Configure** on the **OpenCTI** tile. The **OpenCTI** panel opens:

- When OpenCTI is not set up for the tenant, the panel lists the two variables a platform administrator sets in the platform's environment before restarting it, `CONNECTORS__OPENCTI__<tenant id>__URL` and `CONNECTORS__OPENCTI__<tenant id>__TOKEN`, with the tenant's id filled in. **Copy** copies them.
- Without **Manage Security**, the panel reads "An administrator configures this connector."
- When OpenCTI does not answer, the panel reads "Unable to load Live Streams from OpenCTI right now. It may be temporarily unavailable, so try again shortly."

Once the variables are set, OpenCRQ binds a stream itself at its next start. It creates the Live Stream `OpenCRQ threat feed` in OpenCTI, and binds it to the tenant with the service account "OpenCTI (deployment)". The tile then reads **Syncing** with nothing to configure. This needs the **Manage data sharing** capability, or **Bypass all capabilities**, for the user of the token in OpenCTI. Without it, OpenCRQ binds nothing, and the connector's page in :menu[Integrations] says which capability the token lacks. A stream you bound yourself is never replaced.

To bind another stream, or when OpenCRQ could not create its own, bind the stream yourself with **Manage Security**:

:::steps
1. On the **OpenCTI** tile, select **Configure**.
2. Choose the stream in **Live Stream**. OpenCRQ judges its filter before you save, as in [Bind the stream to your tenant](/opencrq/use/sources/opencti/#bind-the-stream-to-your-tenant).
3. Under **Service account**, choose the account the connector writes as. **Create a new service account**, the default, creates it with the integration, in the Connectors group: its **Service account name** is filled in with "OpenCTI connector", which you can change. **Use an existing service account** lets you pick one the tenant already has, such as "OpenCTI (deployment)". A tenant with no account to reuse is offered the new one only.
4. Select **Save**. The panel closes, and the connector reads the stream from its beginning.
:::

The tile reads **Syncing**, with the percent of the stream's history replayed, then **Healthy** once the stream reaches its live tail. Once the connector is configured, the panel also shows its **Enabled** switch. **Follow in Integrations**, at the bottom of the panel, opens the connector's page.

## Analyze a STIX bundle

:::steps
1. On the **STIX bundle** tile, select **Analyze new bundle**.
2. Choose a `.json` file. The button reads **Analyzing…** until OpenCRQ has read it.
:::

"Bundle analyzed successfully." confirms the import, and the tile counts the threats of the bundle. "Bundle analysis failed. Please check the file and try again." means the file is not a bundle OpenCRQ can read. Without file storage, the button is not available, and the tile reads "File storage must be configured on the platform to import a bundle."

:::example[Give Grand Meridian the threats Example CERT tracks]
Grand Meridian Hotels already follows Velvet Harbor and Ember Tide in its OpenCTI platform, at `https://opencti.example.com`. The platform engineer has set the tenant's two variables. Grand Meridian keeps the management of streams to its CTI team, so the token's user lacks **Manage data sharing**, and OpenCRQ bound no stream at its start.

In the **Threats** step, Claire Morel selects **Configure** on the **OpenCTI** tile:

- **Live Stream**: `Threats for risk quantification`, judged **Delivers threat actors**.
- **Service account**: **Create a new service account**, with **Service account name** left on `OpenCTI connector`.

She selects **Save**. The tile reads **Syncing** while the stream replays its history, then **Healthy**: Velvet Harbor and Ember Tide have arrived, with their campaigns and the techniques they use. She leaves **OpenCRQ Default Threats** switched off, since the stream covers the actors that target hotel chains in France and Spain. The stepper shows its check on **Threats**.
:::

## Check that it works

- The tile of each source you used counts its threats, and the total counts the tenant's threats you can see.
- **See all threats** lists them in the Threat Center, where you can filter them by source.
- The stepper shows a check on **Threats**.

## What's next

- Select **Next** to go on to the **Defenses** step: [Set how strong your controls are in the Defenses step](/opencrq/use/get-started/defenses/).
- [Connect OpenCTI with a Live Stream](/opencrq/use/sources/opencti/), to choose the stream and follow what it brings.
- [Follow the threats that target your scopes](/opencrq/use/threats/threat-center/), to see which threats each scenario counts.
