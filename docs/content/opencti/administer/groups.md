---
title: Organize users in groups
description: Create an OpenCTI group, give it roles, markings and a confidence level, choose its members, and set the defaults its users get.
sidebar_position: 3
doc_type: how-to
products:
  - opencti
personas: [admin]
owner: docs-opencti
last_reviewed: 2026-11-01
---

A group is how users get their rights in OpenCTI: its roles say what members can do, its markings say which data they can see and share, and its confidence level says how much they can change. This page shows how to create a group, set each of its options and read the result. For the model behind groups, read [How access control works](/opencti/administer/access-control/).

## Before you start

- Your role has the **Manage credentials** capability.
- The roles you want to give exist. See [Grant rights with roles](/opencti/administer/roles/).
- To change the built-in groups (Administrators, Connectors, Default), you also need the right to change sensitive configuration. See [Protect sensitive configuration](/opencti/administer/sensitive-configuration/).

## Create a group

:::steps

1. Open :menu[Settings > Security > Groups] and click **Create Group**.
2. In the **Create a group** panel, enter a **Name**, a **Description** and the **Max Confidence Level** of members, from 0 to 100 (100 by default), then click **Create**.
3. Open the group and click **Update**. The **Update a group** panel has five tabs: **Overview**, **Roles**, **Markings**, **Members** and **Confidences**. Every change is saved as you make it.
4. In **Roles**, check the roles to give to members.
5. In **Markings**, check the **Allowed marking definitions** members can see, and set the other marking options described below.

   ::screenshot{id=opencti-groups-edit-markings alt="The Markings tab of the Update a group panel for CTI analysts: TLP:CLEAR, TLP:GREEN and TLP:AMBER checked under Allowed marking definitions, the Default marking definitions selector, and TLP:GREEN chosen for TLP under Maximum shareable marking definitions."}

6. In **Members**, add the users. With single sign-on, prefer a group mapping, which keeps membership in sync with your directory: see [Manage login providers in the UI](/opencti/administer/authentications/#map-groups-and-organizations).

:::

:::example[Set up the CTI team of a hotel group]
The threat intelligence team of Grand Meridian Hotels reads and writes knowledge up to TLP:AMBER, may only share TLP:GREEN outside the platform, and must not delete what others produced.

- **Name**: `CTI analysts`. **Description**: `Threat intelligence team of Grand Meridian Hotels`. **Max Confidence Level**: `75`: members can create and change data up to a confidence of 75, and cannot change data rated above it.
- **Roles**: Analyst, which reads and writes knowledge, imports files and asks for enrichment. It has no **Delete knowledge**, so members create and update but never delete.
- **Allowed marking definitions**: TLP:CLEAR, TLP:GREEN and TLP:AMBER.
- **Maximum shareable marking definitions**: TLP:GREEN for TLP, so that TLP:AMBER data never leaves in an export or a public dashboard.
- **Members**: Ana Duval and Leo Martin.
- **Granted by default at user creation**: off, so that new users of the platform do not join the team.

Ana and Leo now see Velvet Harbor reports marked TLP:AMBER, and their user page shows a **Max Confidence Level** of 75. When Leo exports a report, he can include its content up to TLP:GREEN; asking for its TLP:AMBER content is refused.
:::

## Group options

| Option | Tab | Effect |
| --- | --- | --- |
| **Default dashboard** | Overview | The home dashboard of members who have not chosen one. The group must also be allowed on the dashboard, or a warning shows next to it on the group page |
| **Hidden entity types** | Overview | Entity types hidden from members in the menus and creation forms. This declutters the interface, it does not restrict access |
| **Granted by default at user creation** | Overview | New users join the group, unless their creator turns on **Don't add the user to the default groups** or their single sign-on provider prevents it. The list shows it as **Default membership** |
| **Automatically authorize this group to new marking definition** | Overview | Each new marking definition is added to the allowed markings of the group, and members can share it. The list shows it as **Auto new markings** |
| **Do not accumulate creators for the users of this group** | Overview | When members update an object someone else created, they are not added to its creators. Applies to a user only when all their groups have it |
| **Do not allow these users to delete content created by other users** | Overview | Members can delete only what they created. Applies to a user only when all their groups have it, and never to a user who has **Bypass all capabilities** |
| **Allowed marking definitions** | Markings | The markings whose data members can see. A marking also opens the lower markings of its type. See [Restrict access with markings](/opencti/administer/markings/) |
| **Default marking definitions** | Markings | The markings proposed when members create data, for the entity types whose customization turns default markings on. Only the highest marking of each type is kept |
| **Maximum shareable marking definitions** | Markings | For each marking type, the highest marking of the data members can put in file exports and public dashboards. Choose **Not shareable** to block a whole type. A type left on **No restrictions** is shareable up to what the group is allowed |
| **Max Confidence Level** and overrides | Confidences | The highest confidence members can set or change, with exceptions per entity type |

## Read the group page

The page of a group shows its effective settings in three cards, then its **Triggers and Digests** and its members.

::screenshot{id=opencti-groups-overview alt="The page of the CTI analysts group: the Basic information card with Auto new markings and Default membership set to No, the Permissions card with the Analyst role and a Max Confidence Level of 75, and the Markings card with three allowed markings and TLP:GREEN as maximum shareable marking for TLP."}

- **Basic information**: the description, the hidden entity types, **Auto new markings** and **Default membership**.
- **Permissions**: the **Roles**, the **Default dashboard**, the **Max Confidence Level** with its overrides, **No creators accumulation** and **Restrict delete to created entities**.
- **Markings**: the **Default markings**, the **Allowed markings**, and the **Maximum shareable markings** of each marking type: a marking, **Not shareable**, or **No restrictions**.

A warning icon next to a maximum shareable marking means the group is not allowed that marking. Members can then share only what they are allowed, whatever the maximum says: allow the marking, or choose a lower one.

## Confidence levels

A user can only create, update or delete data up to their maximum confidence level. With several groups, a user gets the **highest** maximum confidence level of their groups, unless a level is set on the user. A user with **Bypass all capabilities** always has 100.

:::warning
A user who gets no maximum confidence level, from a group or set on the user, cannot create or change data. The **Groups** tab of a user shows a warning next to each group without a level.
:::

A group created without a level gets `app:group_confidence_level:max_confidence_default` (`APP__GROUP_CONFIDENCE_LEVEL__MAX_CONFIDENCE_DEFAULT`), 100 by default. This covers the groups that single sign-on creates with **Auto create groups**, and the built-in groups of a new platform. The **Create a group** panel always sends a level, so the setting does not change it. See [Functional customization](/reference/opencti/configuration/#functional-customization).

:::example[Give the groups that single sign-on creates a cautious level]
Example CERT lets its identity provider create the groups of new member teams, and reviews each new group before it trusts its members' changes. In the `environment` of the `opencti` service:

```yaml
APP__GROUP_CONFIDENCE_LEVEL__MAX_CONFIDENCE_DEFAULT: 30
```

A group such as `Seaside Resorts CTI`, created at the first login of its members, then gets a **Max Confidence Level** of 30 instead of 100. Its members cannot overwrite the knowledge Example CERT rated higher until an administrator raises the level.
:::

## Built-in groups

| Group | Role | Purpose |
| --- | --- | --- |
| Default | Default (**Access knowledge**) | Granted by default at user creation. It allows no marking out of the box, so a user in this group only sees data without markings |
| Administrators | Administrator (**Bypass all capabilities**) | Platform administrators. Automatically authorized to new markings |
| Connectors | Connector | The group of the service accounts that integrations create. Automatically authorized to new markings |

The Connectors group is the **Default group for Service accounts** of a new platform: the group every service account created by an ingestion or a connector deployment joins. Change it in [Set platform security policies](/opencti/administer/policies/).

## Delete a group

Open the group, open the actions menu (⋮), choose **Delete** and confirm. The menu shows **Delete** only when your role also has **Delete knowledge**. Members lose the roles and markings the group gave them. A user left without a group with a role can do nothing; a user left without a confidence level can no longer write.

## Check that it works

Open a member in :menu[Settings > Security > Users]: the user page lists the groups, the roles and the effective **Max Confidence Level** of the user. To check what a member sees, log in as them in a private window.

## What's next

- [Configure single sign-on](/opencti/administer/sso/), to fill groups from your identity provider.
- [Restrict access with markings](/opencti/administer/markings/), to decide which data each group sees.
- [Create accounts for integrations](/opencti/administer/service-accounts/), for the groups of connectors.
