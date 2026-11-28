---
title: Add your assets and declare the size of your estate
description: Declare the size of your estate, import assets from Microsoft Defender or create them by hand in the Assets step of Get started, so OpenCRQ knows where an attacker can get in.
sidebar_position: 3
doc_type: how-to
products: [opencrq]
personas: [risk-manager, admin]
owner: docs-opencrq
last_reviewed: 2026-11-20
---

**Assets** is the second step of :menu[Get started]. Your assets and their vulnerabilities tell OpenCRQ where an attacker can get in. The step brings them in from three sources, which add up: the estate you declare, Microsoft Defender, and the assets you create by hand.

## Before you start

- You are a member of the tenant. Declaring the estate and creating assets need no capability.
- To set up Microsoft Defender, you have **Manage Security**, and a platform administrator has given OpenCRQ its credentials. See [Import assets from Microsoft Defender](/opencrq/use/sources/microsoft-defender/#give-opencrq-the-credentials).

## The asset sources

Under **Asset sources**, the step reads "Use as many as you need, they add up." Beside it, the count of the assets you can see in the tenant and **See all assets**, which opens :menu[Assets].

::screenshot{id=opencrq-get-started-assets alt="The Assets step of Get started for Grand Meridian Hotels: under Asset sources, the Declared estate tile with no kind declared, the Microsoft Defender tile Unavailable, and the Manual creation tile Active with 6 assets."}

| Tile | What it brings | What it shows |
| --- | --- | --- |
| **Declared estate** | How many assets of each kind you have. It "Gives every scenario the size of your estate, even before your assets are imported." | How many of the 12 kinds you declared, and **Active** once you declared one. **Declare**, then **Edit**, opens the declaration |
| **Microsoft Defender** | The assets the Microsoft Defender connector imports | The state of the connector: **Unavailable**, **Not configured**, **Never synced**, **Syncing**, **Healthy** or **Failing**. Then the number of assets it imported, and the progress of a run in percent. **Configure** opens it |
| **Manual creation** | The assets you create one by one | The number of assets created by hand, and **Active** once there is one. **Create** opens the **Create a new supporting asset** panel, as in [Declare a supporting asset](/opencrq/use/attack-surface/assets/#declare-a-supporting-asset) |

The sources work together. The attack run walks the assets imported or created, then completes them up to what you declared.

The step is started once you answered one kind, or once you can see an asset of the tenant: the stepper then counts the kinds answered out of 12. It is done once you answered the 12 kinds, **None** included. Assets alone never complete it.

## Declare your estate

The declaration belongs to the risk scope the steps fill, **Enterprise**: it is the same as the **Declared estate** tab of that scope. See [The declared estate](/opencrq/concepts/risk-scopes-and-assets/#the-declared-estate).

:::steps
1. On the **Declared estate** tile, select **Declare**, or **Edit** once you declared a kind. The **Declared estate** panel opens on the 12 kinds of asset, such as **Host › Workstations and laptops**, **Identity › User accounts**, **Web application** or **AI target**. Each kind reads how many assets are measured, or **None measured**.
2. For each kind you know, choose a band: **None**, **1 to 10**, **11 to 50**, **51 to 250**, **251 to 1,000**, **1,001 to 5,000** or **More than 5,000**. **More than 5,000** reads as 5,001 to 25,000. Selecting the chosen band again clears it.
3. To give a figure rather than a band, select **Specify**, then **Exact number**, or **Range** between a minimum and a maximum. Specify a figure beyond 25,000 the same way.
4. For servers, containers, cloud resources, web applications, AI targets and network devices, answer **At least one reachable from the Internet?** with **Yes** or **No**.

   ::screenshot{id=opencrq-get-started-declared-estate alt="The Declared estate panel: Host › Workstations and laptops on 1,001 to 5,000 and Host › Servers and virtual machines on 251 to 1,000, each with what is measured, and the other kinds still unanswered."}

5. Select **Save**. It is available once you changed an answer and every figure is valid. The panel closes.
:::

A figure that is not valid says why, such as **Enter a number.** or **The minimum is above the maximum.**

## Import from Microsoft Defender

Select **Configure** on the Microsoft Defender tile. The Microsoft Defender panel opens:

- When Microsoft Defender is not set up for the tenant, the panel lists the three variables a platform administrator sets in the platform's environment before restarting it. **Copy** copies them, and **See the documentation** opens the platform's configuration on GitHub. The steps are in [Give OpenCRQ the credentials](/opencrq/use/sources/microsoft-defender/#give-opencrq-the-credentials).
- When it is set up and you have **Manage Security**, choose the account the connector runs as under **Service account**, and the **Schedule**, then select **Save**:
  - **Create a new service account**, the default, creates the account with the integration, in the Connectors group. Its **Service account name** is filled in with "Microsoft Defender connector", which you can change.
  - **Use an existing service account** runs the connector as an account the tenant already has, chosen in the **Service account** list.
  - A tenant with no account to reuse is offered the new one only. A connector already bound to an account opens on that account.
- Below the form, **Sync now** starts a run at once, once a service account is bound and no run is going. **Follow in Integrations** opens the connector in :menu[Integrations].
- Without **Manage Security**, the panel reads "An administrator configures this connector."

While a run goes, the tile shows its progress, and the **Assets** step turns in the stepper.

:::example[Size Grand Meridian's estate]
Grand Meridian Hotels has no Microsoft Defender yet. Claire Morel selects **Declare** on the **Declared estate** tile:

- **Host › Workstations and laptops** on **1,001 to 5,000**: 3,800 workstations in France and 1,100 in Spain, 4,900 in all.
- **Host › Servers and virtual machines** on **251 to 1,000** for 480 servers, with **Yes**: one of them is reachable from the Internet.
- **Identity › User accounts**: **Specify**, **Exact number**, 5800, one account per employee.
- **Web application** on **1 to 10**, with **Yes**: the booking site faces the Internet.
- **AI target** on **None**.

She selects **Save**: the tile reads 5 of 12 kinds declared, **Active**, and its button reads **Edit**. The attack run now walks at least 1,001 workstations, each simulated world drawing its own count between 1,001 and 5,000 and keeping it for every year it plays. It counts an internet-facing server even before any device is imported. The step stays started until she answers the 7 other kinds.
:::

## Check that it works

- The **Declared estate** tile counts the kinds you declared and reads **Active**.
- The **Declared estate** tab of the **Enterprise** risk scope shows the same answers.
- The count of assets in the tenant includes the assets imported and created.
- The stepper shows a check on **Assets** once the 12 kinds are answered.

## What's next

- Select **Next** to go on to the **Threats** step.
- [Build the attack surface from your assets](/opencrq/use/attack-surface/assets/), to set the exposure and the criticality of each one.
- [Import assets from Microsoft Defender](/opencrq/use/sources/microsoft-defender/), to keep them in sync.
- [Risk scopes, primary assets and supporting assets](/opencrq/concepts/risk-scopes-and-assets/), to understand how the declaration counts.
