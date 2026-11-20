---
title: Give each organization its own tenant
description: Create a tenant for each organization you serve on one OpenAEV platform, give people access to it, connect its integrations, and delete or reactivate it.
doc_type: how-to
products:
  - openaev
edition: ee
personas: [admin, platform-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/administration/multi-tenancy/
---

A tenant is an isolated workspace of OpenAEV: its own scenarios, simulations, atomic testings, assets, teams, findings, integrations, settings, users, groups and roles. One platform can hold many tenants, so a CERT or a managed security service provider (MSSP) runs a single deployment for all its customers, and a large company keeps its subsidiaries apart. This page creates a tenant, gives people access to it and connects its integrations. Read [How access works in OpenAEV](/openaev/concepts/access/) first if roles and groups are new to you.

## Before you start

- The platform has an Enterprise Edition license. Without it, the platform has a single tenant, the default one, and the **Platform** scope of the Security menu is not shown. See [Editions and licensing](/start/editions/).
- Your role is a platform role with **Manage tenants**, and **Delete tenants** to delete one. The bootstrap admin has them.
- To attach users to tenants from the platform, your role also has **Manage platform users, groups and roles**.

## What a tenant separates

::diagram{id=openaev-multi-tenancy-scopes}

| In each tenant | Shared by the whole platform |
| --- | --- |
| Scenarios, simulations, atomic testings, findings, dashboards and reportings | The deployment: configuration, database, search indices, queues and storage |
| Assets, asset groups, teams, persons and organizations | The login page and its [login messages](/openaev/administer/policies/) |
| Injectors, collectors and executors, each registered for the tenant | The Enterprise Edition license |
| Tags, attack patterns, kill chain phases and vulnerabilities | The platform users, groups and roles |
| Its users, groups, roles and grants, and its [parameters](/openaev/administer/parameters/) | The [usage telemetry](/openaev/administer/telemetry/) |
| Its connection to [XTM Hub](/openaev/administer/xtm-hub/) and to OpenCTI | |

A user of one tenant sees nothing of the others. One account can belong to several tenants: its rights are worked out in each tenant separately, from the groups of that tenant and from its platform groups.

## Rename the default tenant

Every platform starts with a default tenant named **First default tenant auto created to rename**. It holds what existed before multi-tenancy, it cannot be deleted, and the integrations that set no tenant work in it.

:::steps
1. Open :menu[Settings > Security], then choose **Platform** in the switch at the top of the right menu.
2. Click **Tenants**. The list shows each tenant's **Name** and **Status**; the default tenant carries a **Danger Zone** tag.
3. Open the menu of the default tenant, choose **Update**, enter your organization's name in **Name**, and click **Update**.
:::

::screenshot{id=openaev-multi-tenancy-tenants alt="The Tenants management page under the Platform scope: the tenant Grand Meridian Hotels, Active, with its Danger Zone and the Create button."}

## Create a tenant

:::steps
1. In :menu[Settings > Security > Tenants], with the **Platform** scope, click **Create**.
2. In **Create a new tenant**, enter a **Name**, unique on the platform, and a **Description**.
3. Click **Create**. OpenAEV shows **Creating tenant…** while it sets the tenant up.
:::

The tenant shows in the list as **Active**. It already holds its **Admin**, **Manager** and **Observer** groups and roles, described in [What a new tenant starts with](/openaev/concepts/access/#what-a-new-tenant-starts-with), a few sample vulnerabilities, and its built-in injectors and collectors. You are a member of the tenant and of its **Admin** group, so it appears in your tenant switcher.

## Give people access

A user works in a tenant once they are attached to it and belong to one of its groups. Two ways lead there:

- **From the tenant.** Switch to the tenant, open :menu[Settings > Security > Users] with **This tenant**, and click **Create**. An email address that already has an account attaches that account to the tenant; a new one creates the account and sends a reset code by email. This needs **Manage tenant users, groups and roles** in the tenant, not a platform role.
- **From the platform.** With the **Platform** scope, open **Users**, open a user's menu, choose **Update**, and add the tenant to **Tenants**. Removing a tenant from the list detaches the user and removes them from that tenant's groups.

Then open :menu[Settings > Security > Groups] in the tenant and use **Manage users** on the group that fits. Single sign-on can do both at each login: see [Configure single sign-on](/openaev/administer/sso/#send-users-to-the-right-tenant).

## Open a tenant

The left menu shows the current tenant above the navigation when you belong to more than one: click it and choose another. Each tenant has its own address, with the tenant identifier after the host, so you can open two tenants in two browser tabs. When the address names no tenant, OpenAEV opens the last tenant you used in this browser, or else the first one you belong to.

A user who opens the address of a tenant they do not belong to sees **Tenant access denied**: an administrator must attach them. A user attached to no tenant at all is told that their account is not attached to any tenant.

## Connect the integrations of a tenant

Each tenant has its own integrations, and its **Tenant identifier**, under :menu[Settings > Parameters], is what they need:

- **Built-in injectors and collectors** are registered for every new tenant: nothing to do.
- **Integrations deployed by XTM Composer** from the tenant's **Integrations** screen get the tenant identifier automatically.
- **Integrations you run yourself** need `OPENAEV_TENANT_ID` next to `OPENAEV_URL` and `OPENAEV_TOKEN`. Without it they work in the default tenant, like the collectors xtm-docker starts: a new tenant gets no MITRE ATT&CK attack patterns and no NVD vulnerabilities until a collector runs for it. See [Run an integration without XTM Composer](/deploy/integrations/manual/).
- **The link to OpenCTI** is configured per tenant, with the keys `openaev.xtm.opencti.<tenant identifier>.*`. See [Link OpenCTI and OpenAEV](/deploy/configure/connect-modules/#link-opencti-and-openaev).

## Delete or reactivate a tenant

**Delete**, in the menu of a tenant, is a soft delete. The tenant turns **Deleted**, its members lose access at once, and its data is kept for 30 days. During those 30 days, **Reactivate** in the same menu brings it back as it was. After them, a job that runs every night at 2:00 removes the tenant and all its data, documents and search entries included, for good.

The default tenant cannot be deleted: its **Delete** is greyed out.

:::warning
A platform started in [safe mode](/openaev/administer/run-modes/) runs no purge: tenants deleted more than 30 days ago stay until the platform runs in normal mode again, and cannot be reactivated any more.
:::

:::example[Open a tenant for Grand Meridian Hotels]
Example CERT runs one OpenAEV platform for the hotel chains it serves. Its administrator gives the default tenant to its first member chain, then creates a tenant for each other chain, and one, **Example CERT**, for its own analysts. The default tenant gets:

| Field | Value |
| --- | --- |
| **Name** | `Grand Meridian Hotels` |
| **Description** | `Member hotel chain of Example CERT: Paris head office, hotels in France and Spain. CISO: Hélène Marchetti.` |

In that tenant, under **This tenant**, he creates `helene.marchetti@grand-meridian.example` and the security engineer of Grand Meridian, then puts Hélène in **Observer** and the engineer in **Manager**. He reads the **Tenant identifier** in :menu[Settings > Parameters], `7c1e4b2a-5d3f-4e8b-9a61-2f0c8d4e6b13`, and gives it to the platform engineer, who sets it as `OPENAEV_TENANT_ID` on the collector of Meridian EDR. Grand Meridian's people now replay Velvet Harbor against their front-desk workstations without seeing the other chains, and Example CERT's analysts see Grand Meridian only once they are attached to it.
:::

## What's next

- [Configure single sign-on](/openaev/administer/sso/), to attach each chain's staff to its tenant at login.
- [Set the name, look and home page of a tenant](/openaev/administer/parameters/).
- [How access works in OpenAEV](/openaev/concepts/access/), for the groups and roles of each tenant.
