---
title: Give people access with users, groups and roles
description: Invite people to an OpenCRQ tenant, give them rights with groups and roles, check what each one can do, and manage the platform administrators.
sidebar_position: 1
doc_type: how-to
products:
  - opencrq
personas: [admin]
owner: docs-opencrq
last_reviewed: 2026-11-04
---

In OpenCRQ, a person works in a tenant: the risk scopes, scenarios, assets, plans and reviews of one organization. You bring people into a tenant with an invitation, and give the few of them who run it the right to manage its security through a group and a role. Platform administrators get their own groups and roles, which create tenants and name the other administrators. This page does both, and shows how to check what each person can do.

## How access works

::diagram{id=opencrq-users-and-roles}

- **Every member of a tenant works on all its data.** Membership alone lets a person read and change the risk scopes, scenarios, assets, controls, treatment plans and budget reviews of the tenant. There is no read-only member and no right per risk scope in this version.
- **A capability adds what only some should do.** A tenant has one capability, **Manage Security**: it opens :menu[Settings > Security] (members, groups, roles, invitations, service accounts), **Integrations**, :menu[Settings > Jobs], and the choice of the tenant's home dashboard. A person gets it from a role, through a group.
- **The platform has its own capabilities**, granted by platform roles through platform groups: **Create Tenants**, **Manage Users** and **Platform Administration Access**. They never give access to the data of a tenant. See [OpenCRQ capabilities](/reference/opencrq/capabilities/).
- **Accounts are local.** A person signs in with an email address and a password of at least eight characters, and the session lasts 24 hours. Accounts are created by an invitation, or by the first administrator's registration when OpenCRQ starts for the first time (see [Install OpenCRQ](/deploy/install/opencrq/)).

Each tenant starts with two groups. **Admin** holds the **Admin** role, which has every tenant capability, and contains the person who created the tenant. **Connectors** has no role: it is the group a new service account joins by default (see [Let integrations sign in with service accounts](/opencrq/administer/service-accounts/)).

## Before you start

- In the tenant, your role has **Manage Security**: you are in its **Admin** group, or in a group whose role has that capability.
- For the platform's groups and roles, your platform role has **Manage Users** and **Platform Administration Access**. The first administrator has both.

## Invite a person to the tenant

:::steps

1. Open :menu[Settings > Security > Invitations] and click **Invite member**.
2. In the **Invite a new member** panel, enter the **Email address**. Under **Assign to groups**, choose the groups the person joins, or none for a member without security rights.

   ::screenshot{id=opencrq-users-and-roles-invite alt="The Invite a new member panel: a note that the invited user receives a link to join, an Email address field, and an optional Assign to groups selector explained as the groups that determine which roles and permissions the user will have."}

3. Click **Send invitation**. The panel shows the **Invitation link**: click **Copy** and send it to the person yourself. OpenCRQ sends no email; a person who already has an account also finds the invitation in their notifications.
4. The person opens the link. Without an account, they choose a password and click **Create Account & Join**. With an account, they sign in with the invited email address and click **Accept Invitation**.

:::

A link works for 48 hours, once, and only for the email address it was sent to. The **Invitations** list shows each one with its **Status**: **Pending**, **Accepted**, **Expired** or **Cancelled**. Its actions menu offers **Copy link** and **Cancel invitation** on a pending invitation, and **Resend invitation** on an expired one, which creates a new link for the same address and groups. OpenCRQ refuses a second invitation to an address that has one pending, or that is already a member.

## Give rights with groups and roles

A role lists capabilities; a group gives its roles to its members and service accounts.

:::steps

1. To create a role, open :menu[Settings > Security > Roles], click **Create**, enter a **Name** and click **Submit**. Then open its actions menu, choose **Edit capabilities**, check the capabilities under **Capabilities**, and click **Save**. The panel lists the groups that use the role under **Used by groups**.
2. To create a group, open :menu[Settings > Security > Groups], click **Create**, enter a **Name** and click **Submit**.
3. Open the group's actions menu and choose **Edit**. In the **Update group** panel, choose its **Roles**, its **Members** and its **Service accounts**, then click **Save**.
4. To change the groups of one person instead, open :menu[Settings > Security > Users], open the person, click **Manage**, choose their **Groups**, and click **Save**.

:::

A change applies at once: a person who loses **Manage Security** while a tenant's security page is open is taken out of it.

:::warning
OpenCRQ does not stop you from removing the last person who has **Manage Security** in a tenant, yourself included. Nobody could then invite members, configure integrations or follow its jobs. Before you leave the **Admin** group, or change its role, check that another member keeps the capability.
:::

## Read what a person can do

Open :menu[Settings > Security > Users] and click a person. Their page shows:

- **Profile**: the **Name** and **Title** the person holds in this tenant, and their **Email**.
- **Groups & Roles**: the groups they belong to here, and the roles those groups give.
- **Effective Permissions**: how many of the tenant's capabilities they hold, through how many groups and roles, and for each capability the role that grants it.

::screenshot{id=opencrq-users-and-roles-user alt="The page of Claire Morel, cyber risk manager and administrator of the Grand Meridian Hotels tenant: the Admin group and the Admin role under Groups and Roles, and under Effective Permissions 1 of 1 capabilities, Manage Security, granted via the Admin group and its Admin role."}

## Remove a person from the tenant

Open the person, click **Manage**, click **Remove from tenant**, and confirm with **Delete**. The person loses the tenant and every group they had in it. Their account stays on the platform: to give them access again, invite them again. A person who belongs to no tenant and has no platform role can no longer use OpenCRQ.

## Manage the platform administrators

The **Security** menu shows **This tenant** and **Platform** at the top when you have both **Manage Security** in the tenant and **Platform Administration Access**. In **Platform**, the menu lists **Roles**, **Groups**, **Users** and **Tenants**; without **Manage Users**, it lists **Tenants** only. On a platform that has no tenant yet, the same pages open at `/platform/security`.

| Platform capability | What it allows |
| --- | --- |
| **Platform Administration Access** | Open the **Platform** pages of :menu[Settings > Security], and :menu[Settings > Jobs] in every tenant you belong to |
| **Manage Users** | Create and change platform roles and platform groups, and choose the platform groups of each user |
| **Create Tenants** | Create and delete tenants. See [Run several tenants in one platform](/opencrq/administer/tenants/) |

- To create a platform role, open **Roles**, click **Create**, then **Edit capabilities** and check its **Platform Capabilities**.
- To create a platform group, open **Groups**, click **Create**, then **Edit** to choose its **Platform Roles** and its **Members**.
- To make someone a platform administrator, open **Users**, click the person, click **Update**, and add the **Platform Groups** they join. The list shows every account of the platform; nobody is created here.

The first administrator's platform group and role are both named **Admin** and hold the three capabilities. OpenCRQ refuses any change that would leave no user with **Manage Users** or with **Platform Administration Access**, since nobody could then reach these pages to repair it.

:::example[Bring the risk team of Grand Meridian Hotels in]
Claire Morel, cyber risk manager of Grand Meridian Hotels, is so far the only member of the tenant, in its **Admin** group. After the incident of 9 January 2026, she opens it to two people: a finance controller who reads and prices the scenarios but should not manage access, and the head of IT security, who takes over the integrations.

- **Invitation 1**: **Email address** `lucas.ferrand@grand-meridian.example`, **Assign to groups** left empty. Lucas becomes a member: he works on the scenarios of Grand Meridian France and Spain, and does not see :menu[Settings > Security], **Integrations** or **Jobs**.
- **Invitation 2**: **Email address** `marc.delon@grand-meridian.example`, **Assign to groups** `Admin`. Marc gets **Manage Security** through the **Admin** role, and configures the OpenCTI connector.
- Both links go out by the hotel group's own messaging, and stay valid for 48 hours.

Once they accept, Marc's page shows **Manage Security** granted via **Admin**, like Claire's; Lucas's shows none of the tenant's capabilities, and he still works on every scenario. With Marc holding **Manage Security** too, the tenant keeps a security manager whenever Claire is away.
:::

:::xtm-one[XTM One agents act as the person who asks]
An XTM One agent that works in OpenCRQ calls it as the person talking to it: XTM One signs each call with that person's email address, and OpenCRQ matches it to the account with the same address. The agent sees and changes what that person can, in the tenant they work in, and nothing more. A person without an OpenCRQ account gets no access through an agent.
:::

## Check that it works

Open the person's page: **Effective Permissions** says what they can do in this tenant, and which role gives it. To see what a member sees, sign in as them in a private window: a member without **Manage Security** has no **Integrations** entry and no **Security** or **Jobs** under **Settings**.

## What's next

- [Let integrations sign in with service accounts](/opencrq/administer/service-accounts/), for the accounts connectors and scripts use.
- [Run several tenants in one platform](/opencrq/administer/tenants/), to give another organization its own tenant.
- [OpenCRQ capabilities](/reference/opencrq/capabilities/), every capability a role can grant.
