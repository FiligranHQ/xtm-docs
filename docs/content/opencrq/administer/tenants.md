---
title: Run several tenants in one platform
description: Create a tenant for each organization OpenCRQ prices risk for, switch between them, connect each to its sources, and delete one you no longer serve.
sidebar_position: 2
doc_type: how-to
products:
  - opencrq
edition: ee
personas: [admin, platform-engineer]
owner: docs-opencrq
last_reviewed: 2026-11-04
no_screenshot: "More than one tenant requires the Enterprise Edition, and the screens of this documentation are captured on a Community Edition platform: the diagram shows how tenants are kept apart."
---

A tenant holds everything OpenCRQ knows about one organization: its risk scopes, scenarios, assets, controls, plans and budget reviews, its members, groups and roles, its service accounts, its integrations and its jobs. Nothing crosses from one tenant to another. With several tenants, one OpenCRQ platform serves several organizations, such as a CERT pricing the risk of each of its members, or a group whose subsidiaries keep their figures apart. This page creates tenants, connects each to its sources, and deletes one.

::diagram{id=opencrq-tenants}

## Before you start

- More than one tenant requires the Enterprise Edition. OpenCRQ is in Enterprise Edition while XTM One grants it an XTM license that covers this instance, when OpenCRQ registers with XTM One (the `xtm_one` keys of the [configuration](/reference/opencrq/configuration/#xtm-one)).
- Your platform role has **Create Tenants**, and **Platform Administration Access** to open the **Platform** pages of :menu[Settings > Security]. See [Give people access with users, groups and roles](/opencrq/administer/users-and-roles/).

A Community Edition platform serves one tenant. The first administrator gets it when OpenCRQ starts for the first time: it is named **Default**, and has the same identifier on every installation. Creating a second one is refused with "Multiple tenants require an Enterprise Edition license". A platform that returns to the Community Edition keeps every tenant and its data, but each person then reaches only the first tenant they were given.

## Create a tenant

:::steps

1. Open :menu[Settings > Security], choose **Platform** at the top of the menu, and open **Tenants**. On a platform without any tenant, the page is at `/platform/security/tenants`.
2. Click **Create**. In the **Create a new tenant** panel, enter the **Name** of the organization and click **Submit**.
3. Open the new tenant from its card. You are its first member, in its **Admin** group: invite its people from :menu[Settings > Security > Invitations].

:::

A new tenant starts with its two groups, **Admin** and **Connectors**, its own copy of the reference catalogues (countries, sectors, attack outcomes and loss drivers), and **Automatic** as its **Default language**. It has no risk scope, no member but you, and no integration. Its name cannot be changed afterwards in this version.

The **Tenants** page lists the tenants you belong to, with their number of members; the lines view adds their **Identifier**. The card of the tenant you are in is marked **Current tenant**.

## Switch between tenants

When you belong to more than one tenant, the header shows a **Tenant** selector: choose a tenant to open its home. With a single tenant, the header shows its name. A person belongs to each tenant through its own invitation, and has a **Name** and a **Title** of their own in each.

## Set the default language of a tenant

Each tenant has a **Default language**: the language of the interface for its members whose own **Language**, in their profile, is **Automatic**. It is in the tenant's :menu[Settings > Parameters], in the **Tenant** card, with three options: **Automatic** (the default), **English** and **French**.

OpenCRQ chooses the language of each person in this order:

1. Their own **Language**, when they chose **English** or **French** in their profile. See [Set your profile and follow your notifications](/opencrq/use/profile/).
2. The tenant's **Default language**, when it is **English** or **French**.
3. Their browser's language, when it is English or French.
4. English otherwise.

Changing it needs **Manage Security** in the tenant, from a signed-in session: a service account's token cannot. Members without it read the tenant's choice and cannot change it. OpenCRQ confirms with "Tenant default language updated.", and reloads the page when the language shown to you changes. The default language applies inside the tenant only: the platform's own pages at `/platform/security` do not read it.

## Connect a tenant to its sources

Each tenant has its own integrations, configured from its **Integrations** page by a member with **Manage Security**. The connections to OpenCTI, OpenAEV, Microsoft Defender and Drata, and the NVD API key, are deployment settings keyed by the tenant's identifier: `CONNECTORS__OPENCTI__` followed by the identifier and `__URL`, for example. Read the identifier in the tenant's :menu[Settings > Parameters], under **Identifier**, and see the `connectors` keys of the [configuration](/reference/opencrq/configuration/#connectors).

:::example[Serve a second member of Example CERT]
Example CERT already prices the risk of Grand Meridian Hotels in OpenCRQ, and takes on Seaside Resorts, another member hotel chain, whose figures Grand Meridian must not see.

- A platform administrator of Example CERT creates the tenant: **Name** `Seaside Resorts`.
- In the new tenant, they invite the risk manager of Seaside Resorts, `nina.hart@seaside-resorts.example`, into the **Admin** group, so that she manages her own members and integrations.
- :menu[Settings > Parameters] of the new tenant shows its **Identifier**, `019a3c5e-8f21-7c4d-9b6a-2e7f1d0c4a83`. The platform engineer gives the tenant its own connection to Example CERT's OpenCTI, in the `environment` of the `opencrq` service:

```yaml
CONNECTORS__OPENCTI__019a3c5e-8f21-7c4d-9b6a-2e7f1d0c4a83__URL: https://cti.example.com
CONNECTORS__OPENCTI__019a3c5e-8f21-7c4d-9b6a-2e7f1d0c4a83__TOKEN: ${SEASIDE_OPENCTI_TOKEN}
```

After a restart, Nina configures the OpenCTI integration of her tenant with the Live Stream Example CERT shares with Seaside Resorts. Grand Meridian Hotels keeps its own stream and its own figures, and neither chain appears in the other's **Tenant** selector.
:::

## Delete a tenant

Open the **Tenants** page, open the tenant's actions menu, choose **Delete**, and confirm with **Delete**. You need **Create Tenants** and must be a member of the tenant. OpenCRQ deletes the tenant and everything in it: its risk scopes, scenarios, plans and reviews, its groups, roles and service accounts, and its configuration. This cannot be undone. Its members keep their accounts and their other tenants. The tenant's connection keys stay in the deployment configuration until you remove them.

## Check that it works

Open the new tenant from the **Tenant** selector: its :menu[Settings > Parameters] shows its **Name** and **Identifier**, and its **Users** list shows you as the only member, in the **Admin** group.

## What's next

- [Give people access with users, groups and roles](/opencrq/administer/users-and-roles/), to bring each organization's people in.
- [Connect OpenCTI with a Live Stream](/opencrq/use/sources/opencti/), to stream the threats that target each tenant.
- [Configure OpenCRQ](/deploy/configure/opencrq/), for the deployment settings of each tenant's connections.
