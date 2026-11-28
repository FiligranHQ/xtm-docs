---
title: Restrict access with markings
description: Create marking definitions in OpenCTI, give groups access to them, apply default markings, and limit the markings that can leave the platform.
sidebar_position: 22
doc_type: how-to
products:
  - opencti
personas: [admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/segregation/
---

Markings classify each object by sensitivity, such as TLP:AMBER or PAP:RED, and OpenCTI hides an object from anyone whose groups are not cleared for its markings. This page shows how to create markings, grant them to groups, apply them by default and limit what leaves the platform. For how markings combine with organizations, read [How OpenCTI decides who sees what](/opencti/administer/segregation/) first.

## Before you start

- Your role has the **Manage marking definitions** capability to create markings, and **Manage credentials** to edit groups.
- You have decided your classification scheme. OpenCTI ships the Traffic Light Protocol (TLP:CLEAR, TLP:GREEN, TLP:AMBER, TLP:AMBER+STRICT, TLP:RED) and the Permissible Actions Protocol (PAP:CLEAR, PAP:GREEN, PAP:AMBER, PAP:RED) as markings.

## How markings grant access

A marking has a **Type**, such as `TLP`, and an **Order** inside that type. To see an object, a user needs access to every marking on it, through at least one of their groups. Access to a marking also opens the markings of the same type with a lower or equal order: a group allowed TLP:AMBER sees TLP:GREEN and TLP:CLEAR.

For each type, an object keeps only the markings of the highest order. When markings of the same type but different orders meet, OpenCTI drops the lower ones: at creation, when an update or a connector adds one, and when entities are merged. Markings of the same type and the same order are all kept, and so are markings of other types. A report created with PAP:AMBER, PAP:RED, TLP:AMBER+STRICT, TLP:CLEAR and the statement `Partner NDA` ends up with PAP:RED, TLP:AMBER+STRICT and `Partner NDA`. A connector therefore cannot downgrade a marking already on an object. When you pick a marking of another order of the same type by hand, OpenCTI asks you to confirm.

The Structured Threat Information Expression (STIX) 2.1 standard does not allow versioning marking definitions, so a change of definition never changes the markings of existing objects indirectly.

:::only{opencti=">=7.261127"}
## Recognize the TLP markings of other STIX producers

Many STIX producers mark their objects with the TLP 2.0 marking definitions that OASIS publishes, whose ids differ from those of the built-in TLP markings. OpenCTI keeps each OASIS id as another STIX id of the matching built-in marking, so a reference to it resolves to that marking, and the groups allowed it see the object:

| Built-in marking | OASIS TLP 2.0 id |
| --- | --- |
| TLP:CLEAR | `marking-definition--94868c89-83c2-464b-929b-a1a8aa3c8487` |
| TLP:GREEN | `marking-definition--bab4a63c-aed9-4cf5-a766-dfca5abac2bb` |
| TLP:AMBER | `marking-definition--55d920b0-5e8b-4f79-9ee9-91f868d9b421` |
| TLP:AMBER+STRICT | `marking-definition--939a9414-2ddd-4d32-a0cd-375ea402b003` |
| TLP:RED | `marking-definition--e828b379-4e03-4974-9ac4-e53a884c97c1` |

A new platform gives these ids to its TLP markings when it creates them; an upgraded platform adds them at its first start on this version. When another marking definition of the platform already holds one of these ids, for example a copy imported earlier from such a bundle, it keeps that id, and the migration logs a warning that names it.
:::

## Create a marking

:::steps

1. Open :menu[Settings > Security > Marking definitions] and click **Create Marking definition**.
2. Fill **Type** with the group of markings it belongs to, for example `TLP` or `Statement`.
3. Fill **Definition** with its name, for example `TLP:AMBER`.
4. Pick a **Color** and set the **Order**. A higher order is more restrictive within the type.
5. Click **Create**.

   ::screenshot{id=opencti-markings-create alt="The Create a marking definition panel with the Type, Definition, Color and Order fields, and the Cancel and Create buttons."}

:::

Nobody sees data with the new marking until a group is allowed it, except users who bypass all capabilities.

:::example[Keep partner reports to the CTI team]
Grand Meridian Hotels receives reports from its partners under a non-disclosure agreement, and only its CTI team may read them.

- **Type**: `Statement`, a type of its own, so that the marking does not interfere with TLP.
- **Definition**: `Partner NDA`.
- **Color**: `#d84315`, to stand out next to the TLP markings.
- **Order**: `10`. It is the only marking of its type for now; a later, more restrictive statement takes a higher order.

Then, in the **Markings** tab of the CTI analysts group, check `Partner NDA` under **Allowed marking definitions** and set Statement to **Not shareable** under **Maximum shareable marking definitions**. A report marked TLP:GREEN and `Partner NDA` is visible to Ana Duval and Leo Martin only, and never leaves in an export.
:::

## Give a group access to markings

:::steps

1. Open :menu[Settings > Security > Groups], open the group and click **Update**.
2. In the **Markings** tab, under **Allowed marking definitions**, check the markings the group may see.
3. To grant every future marking to this group automatically, turn on **Automatically authorize this group to new marking definition** in the **Overview** tab. New markings also become shareable by the group.

:::

The group page lists the result under **Allowed markings** and **Auto new markings**. See [Organize users in groups](/opencti/administer/groups/) for the other group settings.

## Apply markings by default

Default markings are added to the objects that members of a group create. It takes two settings:

:::steps

1. In the **Markings** tab of the group, under **Default marking definitions**, select the **Default markings** among the allowed ones. When you pick several of the same type, only the highest is kept.
2. Open :menu[Settings > Customization > Entity types] and open the entity type. In its **Attributes**, edit the **Markings** attribute and turn on **Activate/Deactivate default values**. Do this for each type that should get default markings.

:::

Default values only apply to objects created afterwards.

:::example[Mark every new report of the CTI team TLP:AMBER]
The CTI team of Grand Meridian Hotels shares nothing by mistake: its reports start as TLP:AMBER until an analyst lowers the marking.

- In the CTI analysts group, **Default marking definitions**: TLP:AMBER.
- In :menu[Settings > Customization > Entity types], **Report**, the **Markings** attribute: **Activate/Deactivate default values** on.

When Ana Duval creates a report on Velvet Harbor, the **Markings** field is already filled with TLP:AMBER. Other entity types, such as indicators, keep an empty field until you turn their default values on too.
:::

## Limit what can leave the platform

**Maximum shareable marking definitions**, in the **Markings** tab of a group, set for each type the highest marking its members may share outside the platform: in public dashboards and file exports. Data above that level is not shared; choose **Not shareable** to block a whole type. When you set nothing for a type, every marking of that type the group is allowed is shareable. Users who bypass all capabilities can share everything.

For example, with TLP set to TLP:GREEN and Statement set to **Not shareable**, members export TLP:CLEAR and TLP:GREEN data, and nothing that carries a statement marking.

## Restrict objects to named users

Markings restrict by clearance. To restrict one object to named users, groups or organizations, use [authorized members](/opencti/administer/sharing/).

The **Restricted entities** list, in :menu[Data > Restriction > Restricted entities], shows every entity restricted this way, with its **Restriction Date**, so that you can review them. Select entities and click **Remove access restriction** in the toolbar: they become visible again to everyone their markings, and their organizations, allow. Only users with **Bypass all capabilities** see :menu[Data > Restriction].

The **Restricted entities** list needs an Enterprise Edition license: without it, :menu[Data > Restriction] shows only **Restricted drafts**. Markings themselves work without the license. See [Editions and licensing](/start/editions/).

::screenshot{id=opencti-markings-restricted-entities alt="The Restricted entities list of the Data Restriction screen, with the incident response case Front-desk account compromise at Grand Meridian Hotels, its author, its markings and its Restriction Date."}

## Check that it works

Sign in as a member of the group, or ask one, and open a list filtered on the marking. Objects with a marking the group is not allowed must be missing. A member creating an object of a type with default values on sees the default markings filled in.

## What's next

- [Separate data between organizations](/opencti/administer/organizations/) to restrict data by organization as well.
- [Organize users in groups](/opencti/administer/groups/) for the rest of the group settings.
- [Configure entity types](/opencti/configure/entities/) for default values on other attributes.
