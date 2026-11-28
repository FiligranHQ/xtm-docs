---
title: Separate data between organizations
description: Turn on organization segregation in OpenCTI, delegate user management to organization administrators, and share knowledge with organizations by hand or with playbooks.
sidebar_position: 21
doc_type: how-to
products:
  - opencti
personas: [admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/organization-segregation/
  - "opencti:/administration/Part 1 - Organization-Based Segregation Concept, Installation & RBAC Setup/"
  - "opencti:/administration/Part 2 - Customizing the Platform as the Main Organization/"
  - "opencti:/administration/Part 3 - How to Share Data With Organizations/"
---

Organization segregation lets one OpenCTI instance serve several organizations, such as business units, ISAC members or customers, each seeing only the knowledge you share with it. This page takes you from planning to a running setup, then covers delegated user management and manual and automated sharing. Read [How OpenCTI decides who sees what](/opencti/administer/segregation/) first: segregation filters one shared knowledge base, it does not create separate copies.

## Before you start

- Your platform has an Enterprise Edition license for segregation itself: the platform main organization, **Share with an organization**, the sharing playbook step and organization administrators. Organizations as entities, and the default dashboard and hidden entity types of their members, work without it. See [Editions and licensing](/start/editions/).
- Your role has the **Manage credentials** capability. The platform main organization is a [protected sensitive configuration](/opencti/administer/sensitive-configuration/) by default: one of your roles also needs the setting **Allow modification of sensitive configuration** turned on, at the top of its **Capabilities** tab.
- Every user belongs to at least one organization. Once segregation is on, a user without an organization cannot log in, unless they have **Bypass all capabilities** or are a [service account](/opencti/administer/service-accounts/).

## Plan organizations, groups and roles

Organizations decide which objects a user sees; groups and roles decide what the user can do and up to which marking. A user can belong to several organizations and several groups: their visibility is the union of all of them.

:::steps

1. Create each organization as an entity, in :menu[Entities > Organizations]. An organization is both a piece of knowledge and a container of users, so it then appears in :menu[Settings > Security > Organizations] for administration.
2. Add each user to their organizations, in the **Organizations** field of the user, and to their groups, in :menu[Settings > Security > Users].
3. Give the right capabilities to the roles of these groups. Two matter here, both under **Create / Update knowledge**:
   - **Restrict organization access** shows the **Share with an organization** button. It decides who can release data to other organizations: keep it to a few trusted roles.
   - **Manage authorized members** lets users restrict an object to named users, groups and organizations.

:::

## Turn on segregation

This needs an Enterprise Edition license: without it, **Platform organization** stays disabled.

:::steps

1. Open :menu[Settings > Security > Policies].
2. In the **Platform main organization** card, select the organization that owns the platform in **Platform organization**.
3. Read the warning about the consequences and click **Validate**.

   ::screenshot{id=opencti-organizations-main-organization alt="The Platform main organization card of the Policies page, marked Danger Zone, with the notice that setting it turns on organization sharing and segregation, and the Platform organization selector."}

:::

From then on, members of the main organization see all the data their markings allow. Members of other organizations only see what is shared with one of their organizations. Service accounts count as members of the main organization without being listed among its users. The **Allow users to view users of other organizations** option of **Users visibility** turns off.

## Delegate user management to organization administrators

An organization administrator creates, edits and deletes the users of their organization, without access to the rest of the settings. The **Organizations admin** tab and **Grantable groups by Organization administrators** need an Enterprise Edition license.

:::steps

1. In :menu[Settings > Security > Users], open the user, click **Update** and open the **Organizations admin** tab. It lists the organizations the user belongs to, and stays disabled while they belong to none.
2. Check the organizations this user administers.
3. In :menu[Settings > Security > Organizations], open the organization, click **Update** and set **Grantable groups by Organization administrators**. Organization administrators can only put their users in these groups, so they cannot grant more than you intended.

:::

On the organization page, a warning icon next to a grantable group means its roles have **Manage credentials** or **Bypass all capabilities**. An organization administrator could then give a user rights over the whole platform: remove that group from the list.

:::only{opencti=">=7.261030"}
An administrator of several organizations who clicks **Create User** in :menu[Settings > Security > Users] can choose, in **Add a group**, among the grantable groups of all the organizations they administer.
:::

:::only{opencti="<7.261030"}
An administrator of several organizations who clicks **Create User** in :menu[Settings > Security > Users] can choose, in **Add a group**, only among the grantable groups of one of the organizations they administer.
:::

:::only{opencti=">=7.261127"}
Without other settings capabilities, an organization administrator finds **Users**, **Organizations** and **Email templates** under :menu[Settings > Security]. They can read the email templates, to send one to a user of their organization, but creating or changing a template needs **Manage credentials**.

Deleting a user also removes them from the administrators of every organization they administered, whoever deletes them, an organization administrator included.
:::

:::only{opencti="<7.261127"}
Without other settings capabilities, an organization administrator finds **Users** and **Organizations** under :menu[Settings > Security].
:::

:::example[Let a member hotel manage its own analysts]
Example CERT owns the platform and serves several hotel chains. Ana Duval leads the CTI team of Grand Meridian Hotels and should onboard the hotel's analysts herself.

- On Ana Duval, **Organizations admin**: Grand Meridian Hotels, one of her organizations.
- On the Grand Meridian Hotels organization, **Grantable groups by Organization administrators**: CTI analysts and SOC analysts.

Ana now sees :menu[Settings > Security > Users] with the members of Grand Meridian Hotels only. She can create an account for a new SOC analyst and put it in SOC analysts, but cannot make anyone a member of Administrators.
:::

## Tailor the platform for each organization

In :menu[Settings > Security > Organizations], open an organization. Its page shows two cards, then its **Triggers and Digests** and its **Users**:

- **Basic information**: the **Description**, the **Organization type** and the **Contact information** of the organization entity;
- **More information**: the **Hidden entity types**, the **Parent organizations** and **Child organizations**, which come from the **part-of** relationships between organization entities, the **Default dashboard**, and the **Grantable groups by organization administrators**.

::screenshot{id=opencti-organizations-settings-page alt="The page of the Grand Meridian Hotels organization in the security settings: the Basic information card with its type and contact, and the More information card with the CTI analysts and SOC analysts grantable groups."}

Click **Update** to open the **Update the organization** panel. **Name**, **Description**, **Organization type** (such as `csirt`, `vendor` or `constituent`) and **Contact information** describe the organization entity: analysts see them on the entity, and they change no access. Two fields act on its members:

- a **Default dashboard**. This does not open the dashboard to them: add the organization to the authorized members of the dashboard, or a warning shows next to it and members land on a dashboard they cannot read;
- **Hidden entity types**, on top of those hidden for the whole platform and for their groups. This only declutters the interface, it does not restrict access.

To hide organizations from one another, for example the customers of a managed security provider, restrict each organization entity with [authorized members](/opencti/administer/sharing/): your administrators and its own members only. Do it when you create the organization, before you share anything with it.

::::only{opencti=">=7.261119"}
### Open the default dashboard on a preset

Once you choose a **Default dashboard**, a **Default preset** field shows under it. It lists the presets saved on that dashboard: each one holds values for the dashboard's variables and a time range, saved with **Save** > **Save as new preset…** on the dashboard (see [Build a dashboard](/opencti/use/dashboards/)). Members of the organization then open the dashboard on these values instead of its defaults.

::screenshot{id=opencti-organizations-default-preset alt="The Update the organization panel of Grand Meridian Hotels: Member hotel overview as the Default dashboard and, under it, the Last month preset in Default preset, with the helper text saying the dashboard opens on the values and time range of this preset."}

- Left empty, the field reads **No preset (default values of the dashboard)**: the dashboard opens on its own values. Clear the field to come back to it.
- Choosing another default dashboard empties the preset, since presets belong to one dashboard.
- A preset deleted from the dashboard no longer applies: members open the dashboard on its own values.
- When a member's profile, their organizations and their groups point to the same dashboard, the first of them decides, in this order: the profile, the organizations, the groups. A member who chose this dashboard on their profile without a preset does not get the organization's preset.
- On their home page, members always see the default preset. When they open the dashboard itself, a preset they pick there is kept in their browser and replaces the default one.

:::example[Open the member dashboard on the last month]
Example CERT publishes the Member hotel overview dashboard to Grand Meridian Hotels, with the organization among its authorized members. It wants the hotel's analysts to see the last month when they sign in, not the three months the dashboard shows by default. A dashboard editor saves the Last month preset on the dashboard. In the **Update the organization** panel of Grand Meridian Hotels, set:

- **Default dashboard**: Member hotel overview;
- **Default preset**: Last month.

The hotel's analysts now land on Member hotel overview with the time range of the last month.
:::
::::

## Share knowledge with an organization

### By hand

:::steps

1. Open the entity, report or case and click **Share with an organization** in its header.
2. Select the **Organization** and click **Share**.

:::

What travels with the share depends on the object:

| You share | What the organization gets |
| --- | --- |
| A container, such as a report, a grouping or a case | The container, and every entity and relationship it contains. A background task shares the contents |
| Any other entity | The entity only. Its relationships and the entities at their other end stay hidden |

Partners who see two entities but not the relationship between them get empty widgets, because dashboards are built on relationships. When you share a bare entity, share its neighborhood too, with the playbook below.

On a container restricted with authorized members, the **Share with an organization** button is disabled. The authorized members list decides who sees it, even organizations the container was never shared with.

### At creation

Objects created by a user who is not in the main organization are shared with that user's organizations.

:::only{opencti=">=7.260811"}
The same goes for a service account that you add to other organizations. To send everything a connector ingests to an organization, add the connector's service account to that organization: no playbook needed. Everything the connector brings later is shared too, so review its scope when it changes.
:::

For selective sharing, use a playbook.

### With a playbook

A playbook shares recurring knowledge reliably. To share an entity with its neighborhood:

:::steps

1. Start the playbook on the entities to share, with **Listen knowledge events** or **Query knowledge on a regular basis** and a filter.
2. Add **Apply predefined rule** with the **Rule to apply** set to **Resolve neighbors relations and entities (add in bundle)**. The bundle now holds the entity, its relationships and the entities at their other end.
3. To release only part of it, add **Reduce knowledge** with a filter, for example observables only.
4. Add **Share with organizations** and select the **Target organizations**.

   :::only{opencti=">=7.260811"}
   Set **Apply to** to **All elements in the bundle**. Left on **Only main element in the bundle**, it shares the entity alone. **Apply with filters** narrows what is shared.
   :::

   :::only{opencti="<7.260811"}
   Turn on **Share all elements included in the bundle**. Left off, it shares the entity alone.
   :::

5. End with **Send for ingestion**.

:::

::::example[Share the threats against hotels with a member hotel]
Example CERT tracks the intrusion sets that target hotels, labeled `hotel-sector`, and wants Grand Meridian Hotels to see each of them with its neighborhood.

1. **Listen knowledge events**, with a filter on the entity type Intrusion set and the label `hotel-sector`.
2. **Apply predefined rule**, **Rule to apply**: **Resolve neighbors relations and entities (add in bundle)**.
3. **Share with organizations**, **Target organizations**: Grand Meridian Hotels.

   :::only{opencti=">=7.260811"}
   **Apply to**: **All elements in the bundle**.
   :::

   :::only{opencti="<7.260811"}
   **Share all elements included in the bundle**: on.
   :::

4. **Send for ingestion**.

When an analyst updates Velvet Harbor, analysts of the hotel see the intrusion set, the malware and attack patterns it uses, the indicators that point to it and the relationships between them, so their dashboards are not empty.
::::

**Share with organizations** reads the database before ingestion: it fails on an object created earlier in the same playbook. Create in one playbook and share in a second. The settings of each step are in the [playbook components reference](/reference/opencti/playbook-components/#playbook-sharing-component).

### Sharing patterns

- **Public data for everyone.** Create one organization, for example `All members`, and add every member's users to it. A single playbook shares TLP:CLEAR knowledge with it.
- **Data for one organization.** The target organizations are fixed in the playbook step, so build one playbook per audience. To keep an organization in sync, schedule a playbook that queries what is already shared with it, resolves the neighbors and shares again. This stays manageable up to about ten organizations.
- **Data for a group of organizations.** Create a cluster organization, such as `Energy`, and link each member organization to it with a **part-of** relationship. Turn on the rule **Organization propagation via participation** in :menu[Settings > Customization > Rules engine]. Users of every organization part of `Energy` then become members of `Energy` and see what you share with it. Keep this for data meant for the whole group.

## Let users ask for access

A user who tries to create something that exists but is hidden from them gets a "Restricted entity already exists" error. Set up [request access](/opencti/administer/request-access/) so they can ask the owning organization instead, which shares the knowledge when it accepts.

## Check that it works

Sign in as a member of a partner organization, or ask one. Open a report you shared with their organization and check that its entities and relationships show. Search for an entity you did not share: it must not appear in lists or search.

## Best practices

- Design your organizations before you turn segregation on; changing it with users and data in place is harder.
- Use authorized members for anything sensitive, even inside the main organization: segregation never hides data from it.
- Put organization-specific analysis in notes shared with that organization, never in the fields of a shared entity. **Access to collaborative creation** lets partner analysts write notes and opinions on what you share without the right to change the entity itself.
- Review the organizations list and :menu[Data > Restriction > Restricted entities] regularly.

## What's next

- [Restrict access with markings](/opencti/administer/markings/) to combine clearances with organizations.
- [Restrict one object to chosen people](/opencti/administer/sharing/) to restrict single objects.
- [Automate with playbooks](/opencti/use/automate/playbooks/) to automate sharing.
