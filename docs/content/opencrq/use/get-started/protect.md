---
title: Choose what you protect and link the assets behind it
description: Add your primary assets from six templates in the Protect step of Get started, then link the supporting assets they run on, so OpenCRQ knows what an attack can reach and what it would cost.
sidebar_position: 8
doc_type: how-to
products: [opencrq]
personas: [risk-manager, admin]
owner: docs-opencrq
last_reviewed: 2026-11-20
---

**Protect** is the seventh step of :menu[Get started], and its card reads **What you protect**. Your primary assets say what an attack can reach and what it would cost you: your customer data, your payments, your operations. The **Scenarios** step builds its scenarios on them. This step adds them from templates to the risk scope the steps fill, **Enterprise**, then links the supporting assets they run on.

A primary asset is what the business values, whatever runs it. Its business resource, such as personal data, sets which attacks can reach it. See [Primary assets](/opencrq/concepts/risk-scopes-and-assets/#primary-assets).

## Before you start

- You are a member of the tenant.
- To link supporting assets, the tenant has some, imported or created in the **Assets** step. See [Add your assets and declare the size of your estate](/opencrq/use/get-started/assets/).

## The templates

Under **1. Choose what you protect**, six template cards each add one primary asset, with one business resource. Beside them, the count of the primary assets of the scope and **See all primary assets**.

| Template | Business resource of the asset |
| --- | --- |
| **Customer data** | **Personal data** |
| **Payments and treasury** | **Own funds and payment authority** |
| **Production and operations** | **Operations and services** |
| **HR data** | **Personal data** |
| **Intellectual property** | **Proprietary information** |
| **Email and collaboration** | **Operations and services** |

The last card, **Add your own**, reads "Name it and pick its category": it creates a primary asset no template fits. For every business resource, see [Business resources](/reference/opencrq/business-resources/).

The step is done once the scope has one primary asset.

## Add your primary assets

:::steps
1. Under **1. Choose what you protect**, select a template card, such as **Customer data**. It reads **Adding…**, then **Added**: the scope has a primary asset of that name, with that business resource, and no owner or description yet.
2. Select the other templates that apply to your company.
3. For an asset no template fits, select **Add your own**. The **Create a new primary asset** panel opens: name the asset, pick its business resources, and save it.

   ::screenshot{id=opencrq-get-started-protect alt="The Protect step of Get started for Grand Meridian Hotels: Customer data, Payments and treasury, and Production and operations read Added, with Remove and Link assets, and the graph of the three primary assets below."}
:::

A template adds one asset per scope: its card stays **Added** even after you rename the asset. On an added card, **Remove** opens **Delete primary asset**, which warns: "This will permanently delete this primary asset and everything attached to it. This action cannot be undone." Removing the asset frees the template.

## Link the assets that support it

Once the scope has a primary asset, **2. Link the assets that support it** draws a graph of your primary assets and the supporting assets they run on. The supporting assets not linked yet show too.

:::steps
1. Under **2. Link the assets that support it**, choose the primary asset to link, when the scope has more than one.
2. Select **Link assets**. The **Manage dependencies of primary asset** panel opens on every supporting asset, with its **Roles**, **Category**, **Source**, **Exposure** and **Criticality**. A search and filters narrow the list.
3. For each asset that supports the primary asset, pick its role, such as the asset that stores the data or the one that performs the service. Removing its last role unlinks it.
4. Select the save button, which counts your changes.
:::

**Link assets** on an added template card opens the same panel for its asset. Above the graph, a button per primary asset shows or hides it, and the depth sets how many levels of dependencies the graph draws. When you hide every primary asset, the graph reads "Every primary asset is hidden: show one above." For what each role means to a simulated attack, see [Links and their roles](/opencrq/concepts/risk-scopes-and-assets/#links-and-their-roles).

:::example[Name what Grand Meridian protects]
Claire Morel lists what an attack on Grand Meridian Hotels would cost the most. In the **Protect** step, she selects three templates:

- **Customer data**: the guest records, names, contact details and stays of 2.4 million guests.
- **Payments and treasury**: the accounts the head office pays suppliers and refunds from.
- **Production and operations**: the booking platform that takes the room bookings of every hotel.

She leaves **HR data**, **Intellectual property** and **Email and collaboration** for later. She then selects **Link assets** on **Customer data** and links the booking database server, which stores the guest records, and the booking web application, which processes them. She saves: the graph draws **Customer data** with its two supporting assets, and the stepper shows its check on **Protect**.
:::

## Check that it works

- Each template you chose reads **Added**, and the count gives the number of primary assets in the scope.
- **See all primary assets** lists them.
- The graph draws each primary asset with the supporting assets linked to it.
- The stepper shows a check on **Protect**.

## What's next

- Select **Next** to go on to the **Scenarios** step: [Create and price your first risk scenarios](/opencrq/use/get-started/scenarios/).
- [Risk scopes, primary assets and supporting assets](/opencrq/concepts/risk-scopes-and-assets/), to understand how primary and supporting assets work together.
- [Build the attack surface from your assets](/opencrq/use/attack-surface/assets/#link-the-asset-to-what-it-supports), to link an asset from its own page.
