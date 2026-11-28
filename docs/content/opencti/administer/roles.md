---
title: Grant rights with roles
description: How OpenCTI capabilities nest and add up across roles, how to build a role that grants only what a team needs, the built-in roles, and the capabilities that apply in drafts.
sidebar_position: 4
doc_type: how-to
products:
  - opencti
personas: [admin]
owner: docs-opencti
last_reviewed: 2026-11-01
---

A role decides what its users can do in OpenCTI: create knowledge, delete it, manage users, run playbooks. It is a named set of capabilities, each allowing one kind of action. This page explains how capabilities nest and add up, shows how to build a role that grants only what a team needs, and lists the built-in roles. The [capability reference](/reference/opencti/capabilities/) lists every capability. For the overall model, read [How access control works](/opencti/administer/access-control/).

## How roles give rights

A role does nothing on its own. You give it to a group, and every member of the group gets its capabilities. Three rules follow:

- **Rights add up.** A user's capabilities are the union of the roles of all their groups. Adding a user to a group can only give rights, never take them away. To restrict a user, remove them from the group that gives the right.
- **A role says what, not which data.** Capabilities say which actions a user can take. Which objects they can see comes from markings, organizations and authorized members: see [How OpenCTI decides who sees what](/opencti/administer/segregation/).
- **Changes apply at once.** A capability you add to or remove from a role applies to every member of its groups, without a new login.

A user also needs a maximum confidence level, set on a group or on the user, to create or change knowledge, whatever their capabilities. See [Organize users in groups](/opencti/administer/groups/).

## How capabilities nest

The capabilities form a tree, in the order of the **Capabilities** tab. A capability **includes its parents, not its children**:

- A role with **Delete knowledge** also has **Create / Update knowledge** and **Access knowledge**, the capabilities above it. When you check it, they show as checked and greyed out.
- A role with **Create / Update knowledge** does not get **Delete knowledge**, **Merge knowledge** or any other capability under it. You check each one you want.

So you build a role from the bottom: check the most specific capabilities the team needs, and their parents follow.

**Access to admin functionalities** has no checkbox of its own: a role gets it with any of the capabilities below it, such as **Manage credentials**.

**Bypass all capabilities** is different: it gives every capability, all data whatever its markings and organizations, and a maximum confidence level of 100. Keep it to the Administrators group.

:::only{opencti="<7.260811"}
**Share filters** does not exist in this version. On a platform upgraded from an older version, **Manage parameters** shows as **Access administration parameters**, and **Access to support data** as **Access to support**.
:::

:::only{opencti=">=7.260811"}
The upgrade to this version gives **Share filters** to every role that has **Access knowledge**.
:::

## Create a role

You need the **Manage credentials** capability.

:::steps
1. Open :menu[Settings > Security > Roles] and click **Create Role**.
2. Enter a **Name** and a **Description**, then click **Create**.
3. Open the role, click **Update**, and check the capabilities in the **Capabilities** tab. Each checkbox is saved at once.
4. To give the role to users, open a group in :menu[Settings > Security > Groups], click **Update**, and check the role in the **Roles** tab.
:::

::screenshot{id=opencti-roles-capabilities alt="The Capabilities tab of the Update a role panel for the Analyst role, with the capability tree and the capabilities the role grants checked."}

:::example[Give analysts the right to create but not to delete]
At Grand Meridian Hotels, the CTI analysts model the Velvet Harbor campaign: they create reports, indicators and relationships, import files and enrich observables. Deleting or merging entities stays with the team lead, so that a mistake cannot erase shared knowledge.

- Role **Name**: `Analyst`. **Description**: `Reads and writes knowledge, imports files and asks for enrichment.`
- In the **Capabilities** tab, check **Create / Update knowledge**, **Upload knowledge files**, **Import knowledge** and **Ask for knowledge enrichment**. **Access knowledge** shows as checked, as their parent.
- Leave **Delete knowledge**, **Merge knowledge** and every capability under **Access to admin functionalities** unchecked.
- In :menu[Settings > Security > Groups], give the Analyst role to the CTI analysts group.

Members of CTI analysts can now create and change knowledge, but cannot delete or merge entities. They can create drafts, but not approve them: approving a draft needs **Delete knowledge**. The team lead gets a second role with **Delete knowledge** and **Merge knowledge**, through a group of their own.
:::

## Read a role

Click a role in :menu[Settings > Security > Roles] to open its page. The list shows, for each role, the number of **Groups with this role**.

::screenshot{id=opencti-roles-role-page alt="The page of the Analyst role: its description and the CTI analysts group under Groups using this role, then the capabilities it grants and its capabilities in drafts, side by side."}

- **Basic information** gives the **Description** and the **Groups using this role**. Before you change or delete a role, check this list: every member of these groups is affected.
- **Capabilities** lists what the role grants, parents included. **Allow modification of sensitive configuration** shows first when the role has it.
- **Capabilities in Draft** lists what it grants in drafts only (Enterprise Edition).

To delete a role, open the actions menu next to **Update** and choose **Delete**. The menu shows **Delete** only when your role also has **Delete knowledge**. The members of the groups that used it lose its capabilities at once.

## Sensitive configuration

**Allow modification of sensitive configuration** is not a capability but a setting of the role. It shows at the top of the **Capabilities** tab, with a **Danger Zone** chip, while sensitive configuration protection is on. **Bypass all capabilities** does not include it. See [Protect sensitive configuration](/opencti/administer/sensitive-configuration/).

## Built-in roles

| Role | Capabilities on a new platform | Used by |
| --- | --- | --- |
| Default | **Access knowledge** | The Default group |
| Administrator | **Bypass all capabilities** | The Administrators group |
| Connector | What connectors need: **Delete knowledge** and **Merge knowledge** (with **Create / Update knowledge**), **Bypass enforced reference**, **Bypass mandatory fields**, **Upload knowledge files**, **Import knowledge**, **Generate knowledge export**, **Ask for knowledge enrichment**, **Connectors API usage: register, ping, export push ...**, **Allow token usage**, **Manage connector state**, **Access data sharing**, **Manage marking definitions**, **Manage labels**, **Manage vocabularies** and **Manage kill chain phases** | The Connectors group |

:::only{opencti="<7.260811"}
In this version, the Connector role also has **Access ingestion**.
:::

:::only{opencti=">=7.261109"}
The Connector role also has **Create knowledge via Form Intake**.
:::

:::only{opencti=">=7.261109"}
## Capabilities added by an upgrade

An upgrade to 7.261109 adds two capabilities, and grants each to the roles that already had the rights it splits off, so that nobody loses access:

| Capability | What it allows | Granted on upgrade to the roles with |
| --- | --- | --- |
| **Create knowledge via Form Intake**, under **Access knowledge** | Create knowledge by submitting a [form](/opencti/use/ingest/forms/), without **Create / Update knowledge**. It only creates: updating an object that already exists still needs **Create / Update knowledge**, and creating a label still needs **Manage labels** | **Create / Update knowledge**, outside and in drafts |
| **Manage custom fields**, under **Access to admin functionalities** | Create, change and delete the [custom fields](/opencti/configure/taxonomies/#custom-fields) of the platform (Enterprise Edition). Adding them to entity types stays with **Manage customization** | **Manage customization** |

After the upgrade, uncheck them in the roles that should not keep them: for example, remove **Manage custom fields** from a role that customizes entity types but must not change the data model.
:::

The three roles are protected as sensitive configuration by default: only users allowed to change sensitive configuration can edit them. Create your own roles rather than widening Default: every new user joins the Default group unless you prevent it.

## Capabilities in drafts

With an Enterprise Edition license, a role can give more rights inside drafts than outside, so analysts can prepare changes without touching the main knowledge base. Set them in the **Capabilities in Draft** tab of the role. A capability the role already has outside drafts shows as checked there: a role never has fewer rights in a draft.

These capabilities can be added in drafts: **Access knowledge**, **Access to collaborative creation**, **Create / Update knowledge**, **Delete knowledge**, **Merge knowledge**, **Bypass enforced reference**, **Bypass mandatory fields**, **Upload knowledge files**, **Import knowledge**, **Ask for knowledge enrichment** and **Manage labels**.

:::only{opencti=">=7.261109"}
**Create knowledge via Form Intake** can also be added in drafts: users then submit forms into the draft they work in.
:::

:::only{opencti=">=7.260811"}
**Restrict organization access** can also be added in drafts. Creating a draft requires **Create / Update knowledge**, outside or in drafts. Validating or deleting a draft requires **Delete knowledge** outside drafts.
:::

:::only{opencti="<7.260811"}
Creating or deleting a draft requires **Access knowledge**. Validating a draft requires **Delete knowledge** outside drafts.
:::

For example, give the Analyst role **Delete knowledge** in drafts only. Analysts can then remove wrong entities from a draft they prepare, while the draft still needs the team lead, who has **Delete knowledge** outside drafts, to reach the main knowledge base. See [Approve drafts step by step](/opencti/use/automate/draft-workflow/).

## What's next

- [Organize users in groups](/opencti/administer/groups/), to give roles to users.
- [Protect sensitive configuration](/opencti/administer/sensitive-configuration/), to control who changes the built-in roles.
- [How access control works](/opencti/administer/access-control/), the model behind roles.
