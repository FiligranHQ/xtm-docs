---
title: Run several tenants in one platform
description: Create a tenant for each organization OpenCRQ prices risk for, switch between them, connect each to its sources, rename one, and delete one you no longer serve.
sidebar_position: 2
doc_type: how-to
products:
  - opencrq
edition: ee
personas: [admin, platform-engineer]
owner: docs-opencrq
last_reviewed: 2026-11-20
---

A tenant holds everything OpenCRQ knows about one organization: its risk scopes, scenarios, assets, controls, plans and budget reviews, its members, groups and roles, its service accounts, its integrations and its jobs. Nothing crosses from one tenant to another. With several tenants, one OpenCRQ platform serves several organizations, such as a CERT pricing the risk of each of its members, or a group whose subsidiaries keep their figures apart. This page creates tenants, connects each to its sources, renames one, and deletes one.

::diagram{id=opencrq-tenants}

## Before you start

- More than one tenant requires the Enterprise Edition. OpenCRQ is in Enterprise Edition while XTM One grants it an XTM license that covers this instance, when OpenCRQ registers with XTM One (the `xtm_one` keys of the [configuration](/reference/opencrq/configuration/#xtm-one)). Each answer of XTM One grants it for 7 days, and a restart starts in the Community Edition until the next registration succeeds: see [Configure OpenCRQ](/deploy/configure/opencrq/#xtm-one).
- Your platform role has **Create Tenants**, and **Platform Administration Access** to open the **Platform** pages of :menu[Settings > Security]. XTM One's administrator, who signs in to OpenCRQ as its super administrator, holds both, and so does the **Platform administrator** role, which OpenCRQ creates at its first boot with the **Platform administrators** group. See [Give people access with users, groups and roles](/opencrq/administer/users-and-roles/).

A Community Edition platform serves one tenant. OpenCRQ creates it at its first boot, when the platform holds no tenant: its name is **Default** and its identifier `01a043da-ff47-72c2-aaa4-7da56d85c83e`, unless the deployment sets `INITIAL_TENANT__NAME` and `INITIAL_TENANT__ID` before that boot (the `initial_tenant` keys of the [configuration](/reference/opencrq/configuration/#initial-tenant)). Setting the identifier first lets the platform engineer write the tenant's connection keys in the same deployment. See [Install OpenCRQ](/deploy/install/opencrq/). Creating a second tenant is refused with "Multiple tenants require an Enterprise Edition license". A platform that returns to the Community Edition keeps every tenant and its data, but each person then reaches only one: of the tenants they reach, the one whose **Identifier** comes first in alphabetical order. For the tenants OpenCRQ created, that is the oldest; an identifier set with `INITIAL_TENANT__ID` takes its place by its own value.

## Create a tenant

:::steps

1. Open :menu[Settings > Security], choose **Platform** at the top of the menu, and open **Tenants**. When you reach no tenant yet, OpenCRQ opens on this page, at `/platform/security/tenants`.
2. Click **Create**. In the **Create a new tenant** panel, enter the **Name** of the organization and click **Submit**.
3. Open the new tenant from its card, and add its people from :menu[Settings > Security > Users] with **Add members**. See [Give people access with users, groups and roles](/opencrq/administer/users-and-roles/).

:::

A new tenant starts with its two groups, **Admin** and **Connectors**, its own copy of the reference catalogues (countries, sectors, attack outcomes and loss drivers), and **Automatic** as its **Default language**. It also gets a first risk scope, **Enterprise**, and a **Get started** wizard, first in its menu until it is finished, whose steps are **Company**, **Assets**, **Threats**, **Defenses**, **Finances**, **Tolerance**, **Protect** and **Scenarios**. It has no integration yet: even the **OpenCRQ Default Threats** set starts switched off, and loading it is a choice of the wizard's **Threats** step. The tenant OpenCRQ creates at its first boot starts the same way. At each start, OpenCRQ then connects the OpenCTI and OpenAEV integrations the deployment sets for a tenant: see [Connect a tenant to its sources](#connect-a-tenant-to-its-sources). While the wizard is unfinished, a person who opens the tenant's home for the first time in a browser lands on **Get started**.

Its only member is the person who created it, in its **Admin** group, unless that person is the super administrator: they reach every tenant without being a member of any, so a tenant they create starts with no member. The tenant OpenCRQ creates at its first boot starts with no member either.

The **Tenants** page lists the tenants you belong to, with their number of members; the super administrator sees every tenant. The lines view adds their **Identifier**. The card of the tenant you are in is marked **Current tenant**. The actions menu of a tenant offers **Rename** and **Delete** only when your platform role has **Create Tenants**.

## Rename a tenant

Renaming works in both editions: on a Community Edition platform, it gives the **Default** tenant the name of your organization.

:::steps

1. Open the **Tenants** page, open the actions menu of the tenant, on its card or its line, and choose **Rename**.
2. In the **Rename the tenant** panel, change the **Name** and click **Save**.

   ::screenshot{id=opencrq-tenants-rename alt="The Rename the tenant panel opened from the Tenants page, its Name field holding Grand Meridian Hotels, with the Save button."}

:::

OpenCRQ confirms with "Tenant renamed successfully.", and the new name shows on the **Tenants** page, in the tenant switcher of the sidebar and in the tenant's :menu[Settings > Parameters]. The tenant keeps its **Identifier**, so its connection keys in the deployment configuration stay valid. You need **Create Tenants** and must reach the tenant: be its member, or be the super administrator.

:::example[Name the tenant of a new platform]
Example CERT installs OpenCRQ in Community Edition to price the risk of Grand Meridian Hotels. At its first boot, OpenCRQ creates the tenant **Default**. XTM One's administrator signs in, opens the **Tenants** page, chooses **Rename** on the **Default** card, enters **Name** `Grand Meridian Hotels` and clicks **Save**. The tenant's :menu[Settings > Parameters] now reads Grand Meridian Hotels, and its identifier, `01a043da-ff47-72c2-aaa4-7da56d85c83e`, is unchanged.
:::

## Switch between tenants

When you reach more than one tenant, the first row of the sidebar is the tenant switcher: it shows the name of the tenant you are in, and opens the list of your tenants. Choose one to open its home. With a single tenant, the sidebar shows no switcher and no tenant name. A person is added to each tenant on its own, by an administrator of that tenant, and can give themselves a **Full name** and a **Title** of their own in each, in the **Tenant profile** of their profile page.

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
- In the new tenant, they click **Add members** in :menu[Settings > Security > Users], find the risk manager of Seaside Resorts, `nina.hart@seaside-resorts.example`, in XTM One's directory under **Find a person**, and choose `Admin` under **Assign to groups**, so that she manages her own members and integrations. XTM One emails her to sign in.
- :menu[Settings > Parameters] of the new tenant shows its **Identifier**, `019a3c5e-8f21-7c4d-9b6a-2e7f1d0c4a83`. The platform engineer gives the tenant its own connection to Example CERT's OpenCTI, in the `environment` of the `opencrq` service:

```yaml
CONNECTORS__OPENCTI__019a3c5e-8f21-7c4d-9b6a-2e7f1d0c4a83__URL: https://cti.example.com
CONNECTORS__OPENCTI__019a3c5e-8f21-7c4d-9b6a-2e7f1d0c4a83__TOKEN: ${SEASIDE_OPENCTI_TOKEN}
```

After a restart, OpenCRQ connects the OpenCTI integration of Seaside Resorts by itself: it creates the service account `OpenCTI (deployment)` and binds the connector to the Live Stream `OpenCRQ threat feed`, which it creates in Example CERT's OpenCTI when it is not there yet. The user of the token needs **Manage data sharing** in OpenCTI for that. Nina can switch to the stream Example CERT shares with Seaside Resorts in the connector's **Configure integration** panel. Grand Meridian Hotels keeps its own stream and its own figures. Nina belongs to Seaside Resorts alone: her sidebar shows no tenant switcher.
:::

## Delete a tenant

Open the **Tenants** page, open the tenant's actions menu, choose **Delete**, and confirm with **Delete**. You need **Create Tenants** and must reach the tenant: be its member, or be the super administrator. OpenCRQ deletes the tenant and everything in it: its risk scopes, scenarios, plans and reviews, its groups, roles and service accounts, and its configuration. This cannot be undone. Its members keep their accounts and their other tenants. The tenant's connection keys stay in the deployment configuration until you remove them. A link to the deleted tenant, like a link to a tenant someone does not reach, shows "This tenant isn't available." and sends the person where they land when they sign in, their first tenant for most.

## Check that it works

Open the new tenant from the tenant switcher of the sidebar, or from its card on the **Tenants** page: its :menu[Settings > Parameters] shows its **Name** and **Identifier**. Its **Users** list in :menu[Settings > Security] shows you in the **Admin** group, or, when you are the super administrator, tagged **Super Administrator**, and then the people you add.

## What's next

- [Give people access with users, groups and roles](/opencrq/administer/users-and-roles/), to bring each organization's people in.
- [Connect OpenCTI with a Live Stream](/opencrq/use/sources/opencti/), to stream the threats that target each tenant.
- [Configure OpenCRQ](/deploy/configure/opencrq/), for the deployment settings of each tenant's connections.
