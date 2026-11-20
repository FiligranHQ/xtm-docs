---
title: Give people access with users, groups and roles
description: Add people from XTM One to an OpenCRQ tenant, give them rights with groups and roles, check what each one can do, and manage the platform administrators.
sidebar_position: 1
doc_type: how-to
products:
  - opencrq
personas: [admin]
owner: docs-opencrq
last_reviewed: 2026-11-04
---

In OpenCRQ, a person works in a tenant: the risk scopes, scenarios, assets, plans and reviews of one organization. You add people to a tenant from XTM One's directory, and give the few of them who run it the right to manage its security through a group and a role. Platform administrators get their own groups and roles, which create tenants and name the other administrators. This page does both, and shows how to check what each person can do.

## How access works

::diagram{id=opencrq-users-and-roles}

- **Every member of a tenant works on all its data.** Membership alone lets a person read and change the risk scopes, scenarios, assets, controls, treatment plans and budget reviews of the tenant. There is no read-only member and no right per risk scope in this version.
- **A capability adds what only some should do.** A tenant has one capability, **Manage Security**: it opens :menu[Settings > Security] (users, groups, roles, service accounts), **Integrations**, :menu[Settings > Engine], :menu[Settings > Jobs], the changes to a CWE's techniques, and the choice of the tenant's home dashboard. A person gets it from a role, through a group.
- **The platform has its own capabilities**, granted by platform roles through platform groups: **Create Tenants**, **Manage Users** and **Platform Administration Access**. They never give access to the data of a tenant. See [OpenCRQ capabilities](/reference/opencrq/capabilities/).
- **People sign in through XTM One.** OpenCRQ has no password of its own: a person clicks **Sign in with XTM One**, and their OpenCRQ account is tied to their XTM One account. A session lasts about an hour, and XTM One renews it while the person works: an account disabled in XTM One loses OpenCRQ within the hour.
- **XTM One's administrator is the super administrator.** Tagged **Super Administrator**, they hold every capability of the platform and of every tenant, from XTM One at each sign-in, without being a member of any tenant or group. They create the first tenant (see [Install OpenCRQ](/deploy/install/opencrq/)).

Each tenant starts with two groups. **Admin** holds the **Admin** role, which has every tenant capability, and contains the person who created the tenant, unless the super administrator created it. **Connectors** has no role: it is the group a new service account joins by default (see [Let integrations sign in with service accounts](/opencrq/administer/service-accounts/)).

## Before you start

- In the tenant, your role has **Manage Security**: you are in its **Admin** group, or in a group whose role has that capability. The super administrator has it in every tenant.
- For the platform's groups and roles, your platform role has **Manage Users** and **Platform Administration Access**. The super administrator has both.
- OpenCRQ reaches XTM One: adding people searches XTM One's directory, and XTM One sends the emails.

## Add people to the tenant

:::steps

1. Open :menu[Settings > Security > Users] and click **Add members**.
2. Under **Find a person**, type at least two characters of a name or an email: OpenCRQ searches XTM One's directory. Pick each person to add, up to 50 at once. A person who is already a member shows **Already a member** and cannot be picked. For someone who has no XTM One account yet, type their whole email address and pick it: it reads "No XTM One account yet: XTM One will invite them".
3. Under **Assign to groups**, choose the groups they all join, or none for members without security rights.

   ::screenshot{id=opencrq-users-and-roles-add-members alt="The Add members panel: a Find a person field that searches XTM One's directory by name or email, an optional Assign to groups selector, and the Cancel and Add buttons."}

4. Click **Add**: the button counts the people picked. The panel says how many members were added to the tenant, and that each of them will get an email to sign in. Click **Close**, or **Add more members**.

:::

XTM One sends each person an email that leads to the tenant. A person who has no XTM One account gets an invitation from XTM One instead: they create their account with the same email address, and then land in the tenant. In OpenCRQ, each person also gets a notification that you added them. The **Users** list shows their **Status**: **Never signed in** until their first sign-in, then **Active**.

When XTM One could not send an email, the panel gives you the link to send yourself: the **Access link**, for the people who already have an XTM One account (**Copy the access link**), or the **Invitation link** of each person XTM One invited (**Copy the invitation link**). A person who could not be added stays picked, and the panel says that their email address was not added, with the reason:

| Reason shown | What to do |
| --- | --- |
| "This email cannot be added here. Check it, or, if they have no XTM One account yet, ask an XTM One administrator to invite them." | XTM One refused to invite this email for you, or the email belongs to another account. Check it, or ask an XTM One administrator |
| "An XTM One invitation is already pending for this email. Add them here once they have created their XTM One account." | Wait until they accept the invitation XTM One already sent, then add them |
| "XTM One could not be reached. Try again in a few minutes." | Click **Add** again later |
| "They are an XTM One super administrator, who already reaches every tenant and the platform administration." | Nothing: the super administrator holds every right already |

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
OpenCRQ does not stop you from removing the last person who has **Manage Security** in a tenant, yourself included. Only the super administrator could then add members, configure integrations or follow its jobs. Before you leave the **Admin** group, or change its role, check that another member keeps the capability.
:::

## Read what a person can do

Open :menu[Settings > Security > Users] and click a person. Their page shows:

- **Profile**: the **Name** and **Title** the person holds in this tenant, and their **Email**.
- **Groups & Roles**: the groups they belong to here, and the roles those groups give.
- **Effective Permissions**: how many of the tenant's capabilities they hold, through how many groups and roles, and for each capability the role that grants it.

The **Users** list shows the super administrator in every tenant, tagged **Super Administrator** beside their groups. Their page has no **Manage** button, and each capability reads "via Super administrator (XTM One)".

::screenshot{id=opencrq-users-and-roles-user alt="The page of Claire Morel, cyber risk manager and super administrator: the Super Administrator tag under Groups and Roles, and under Effective Permissions Manage Security, granted via Super administrator (XTM One)."}

## Remove a person from the tenant

Open the person, click **Manage**, click **Remove from tenant**, and confirm with **Delete**. The person loses the tenant and every group they had in it. Their account stays on the platform: to give them access again, add them again with **Add members**. A person who belongs to no tenant and has no platform role can no longer use OpenCRQ.

## Manage the platform administrators

The **Security** menu shows **This tenant** and **Platform** at the top when you have both **Manage Security** in the tenant and **Platform Administration Access**. In **Platform**, the menu lists **Roles**, **Groups**, **Users** and **Tenants**; without **Manage Users**, it lists **Tenants** only. A platform administrator who belongs to no tenant finds the same pages at `/platform/security`, under **Administration** in the menu.

| Platform capability | What it allows |
| --- | --- |
| **Platform Administration Access** | Open the **Platform** pages of :menu[Settings > Security], and :menu[Settings > Jobs] in every tenant you belong to |
| **Manage Users** | Create and change platform roles and platform groups, choose the platform groups of each user, and add members to the platform |
| **Create Tenants** | Create and delete tenants. See [Run several tenants in one platform](/opencrq/administer/tenants/) |

- To create a platform role, open **Roles**, click **Create**, then **Edit capabilities** and check its **Platform Capabilities**.
- To create a platform group, open **Groups**, click **Create**, then **Edit** to choose its **Platform Roles** and its **Members**.
- To make someone a platform administrator, open **Users**, click the person, click **Update**, and add the **Platform Groups** they join. The list shows every account of the platform, with its **Platform groups**, **Roles** and **Tenants**; the super administrator's reads **All tenants**.
- To make a platform administrator of someone who has no OpenCRQ account yet, click **Add members** on the **Users** page, find them under **Find a person**, choose at least one group under **Assign to platform groups**, and click **Add**. XTM One emails or invites them as for a tenant; once signed in, they land on the platform's **Tenants** page.

A new platform has no platform group and no platform role: the super administrator holds the three capabilities from XTM One, and creates the groups and roles the other administrators need. OpenCRQ refuses to remove a role from a platform group, a person from a platform group, or a whole platform group or role, when no user would then hold **Manage Users** or **Platform Administration Access** through a platform group. The super administrator does not count.

:::example[Bring the risk team of Grand Meridian Hotels in]
Claire Morel, cyber risk manager of Grand Meridian Hotels, administers XTM One, and reaches the tenant as OpenCRQ's super administrator. After the incident of 9 January 2026, she opens it to two people who already have an XTM One account: an analyst who reads and prices the scenarios but should not manage access, and the head of IT security, who takes over the integrations.

- **First add**: under **Find a person**, she types `Camille` and picks Camille Roux, `camille.roux@grand-meridian.example`, and leaves **Assign to groups** empty. Camille becomes a member: she works on the scenarios of Grand Meridian France and Spain, and does not see :menu[Settings > Security], **Integrations** or **Jobs**.
- **Second add**: she picks Marc Dupuis, `marc.dupuis@grand-meridian.example`, with `Admin` under **Assign to groups**. Marc gets **Manage Security** through the **Admin** role, and configures the OpenCTI connector.
- XTM One emails each of them a link to the tenant. The **Users** list shows them both as **Never signed in** until they sign in.

::screenshot{id=opencrq-users-and-roles-users-list alt="The Users list of Grand Meridian Hotels: Claire Morel tagged Super Administrator and Active, Camille Roux with no group, and Marc Dupuis in the Admin group with the Admin role, both Never signed in."}

Marc's page shows **Manage Security** granted via **Admin**; Camille's shows none of the tenant's capabilities, and she still works on every scenario. With Marc holding **Manage Security** in the tenant, its security no longer depends on Claire alone.
:::

:::xtm-one[XTM One agents act as the person who asks]
An XTM One agent that works in OpenCRQ calls it as the person talking to it: XTM One signs each call for that person, and OpenCRQ matches it to the account tied to their XTM One account, not to an email address. The agent sees and changes what that person can, in the tenant they work in, and nothing more. A person whose XTM One account is tied to no OpenCRQ account gets no access through an agent.
:::

## Check that it works

Open the person's page: **Effective Permissions** says what they can do in this tenant, and which role gives it. To see what a member sees, sign in as them in a private window: a member without **Manage Security** has no **Integrations** entry and no **Security**, **Engine** or **Jobs** under **Settings**.

## What's next

- [Let integrations sign in with service accounts](/opencrq/administer/service-accounts/), for the accounts connectors and scripts use.
- [Run several tenants in one platform](/opencrq/administer/tenants/), to give another organization its own tenant.
- [OpenCRQ capabilities](/reference/opencrq/capabilities/), every capability a role can grant.
