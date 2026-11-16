---
title: Restrict one object to chosen people
description: How OpenCTI restricts one object, such as a dashboard, a report or an organization, to chosen users, groups or organizations, and when to use it rather than markings.
sidebar_position: 23
doc_type: explanation
products:
  - opencti
personas: [admin, cti-analyst, cti-lead]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/authorized-members/
---

Roles say what a user can do everywhere; sometimes you need to decide access for one object instead. OpenCTI does it with **authorized members**: a list of users, groups or organizations that restricts the object to them, each with an access level. This page explains how they work, where they apply, and when to use them rather than markings or organizations.

## What it is good for

| Goal | Example |
| --- | --- |
| Share a workspace with your team | A dashboard visible to the threat intelligence group only, editable by its lead |
| Keep a sensitive investigation to a few people | An incident response case only its responders can open, inside an organization that sees everything else |
| Hide organizations from one another | Each customer of a managed security provider sees only its own organization entity |

## How it works

By default, everyone whose markings and organization allow it can access an object. Once you restrict it, **only** the authorized members can access it, each with an access level:

| Level | Allows |
| --- | --- |
| **can view** | Read the object |
| **can edit** | Read and change it |
| **can manage** | Read, change and delete it, and change its authorized members |

::diagram{id=opencti-administer-authorized-members}

Authorized members add a check; they do not replace the others. A member still needs the capabilities for the action and the markings of the object. Every restricted object must keep at least one member with **can manage**, so nobody creates an object no one can manage. Users with **Bypass all capabilities** always have full access.

### Where they apply

| Objects | Who has access without a restriction |
| --- | --- |
| Custom dashboards and investigations | Only the creator, who gets **can manage** |
| Priority intelligence requirements (PIR) | Everyone can view; the creator manages |
| Reports, groupings, incident responses, requests for information, requests for takedown and feedbacks | Everyone the markings and organizations allow |
| Organizations | Everyone |
| Drafts | Everyone who can access drafts |

Sharing dashboards and investigations works in Community Edition. Restricting knowledge objects and organizations needs an Enterprise Edition license: without it, the padlock is disabled.

### Restrict an object

For a knowledge object, such as a report or an organization, your role needs the **Manage authorized members** capability. On any object, you need **can manage** once it is restricted.

1. Open the object and click the padlock, **Manage access restriction**, in its header. When the header has no room for it, and for a dashboard or an investigation, choose **Manage access restriction** in the actions menu (⋮).
2. For a knowledge object, turn on **Activate access restriction**.
3. Under **Add new specific access**, select a user, group or organization, choose its **Access right** and click **Add**. For an organization, **Advanced options** > **Groups restriction** narrows the access to the members of some groups.
4. Click **Save**. The padlock shows the object is restricted.

When you turn on the restriction, OpenCTI lists you with **can manage**, so that you keep control of the object. For reports, groupings, incident responses, requests for information and requests for takedown, you can also set authorized members in the creation form.

:::example[Keep an incident case to its responders]
The incident response "Booking system intrusion at a hotel chain" holds exports of guest records from a member hotel. Until the investigation closes, only the SOC analysts who work it may change it, and the CTI analysts may read it to link it to Velvet Harbor.

- **Activate access restriction**: on. You stay listed with **can manage**.
- **Add new specific access**: SOC analysts, **Access right** **can edit**, then **Add**.
- **Add new specific access**: CTI analysts, **Access right** **can view**, then **Add**.

After **Save**, other users no longer find the case in lists or searches, even members of the platform organization, unless they have **Bypass all capabilities**. A SOC analyst can change it only if their role also has **Create / Update knowledge**. The IP address, the domain and the Velvet Harbor intrusion set in the case stay visible to everyone their markings allow: restrict them too if they must stay hidden.
:::

### Rules that surprise people

- **Contents keep their own access.** Restricting a report does not restrict the entities it contains.
- **Restriction replaces organization sharing.** Members of a restricted object see it even when it is not shared with their organization. On a restricted container, **Share with an organization** is disabled until you remove the restriction.
- **No "everyone" with a platform organization.** When a platform main organization is set, you cannot grant access to all users on knowledge; name the groups or organizations instead.
- **Not in drafts.** You cannot change the authorized members of an object from inside a draft.
- **Service accounts are not held back.** A [service account](/opencti/administer/service-accounts/) reads and changes every restricted object, even when it is not among its authorized members, so that integrations keep working on restricted reports and cases. It still needs the markings of the object. Give connectors a service account, never a person's account, or their writes on restricted objects fail.
- **Organizations.** A restricted organization disappears, for users who are not its authorized members, from lists, author fields and sharing targets. Use it to hide organizations from one another when you segregate by organization.

:::only{opencti=">=7.260811"}
A service account listed with **can manage** also manages the object and its authorized members; otherwise it gets **can edit**.
:::

:::only{opencti="<7.260811"}
A service account always gets **can edit**, even when it is listed with **can manage**: it cannot delete the object or change its authorized members.
:::

### Find and lift restrictions

Users with **Bypass all capabilities** see every restricted object in :menu[Data > Restriction > Restricted entities], with the date the restriction was set, and can remove a restriction there. The list needs an Enterprise Edition license. **Restricted drafts** does the same for drafts.

A playbook can also set or remove restrictions in bulk, with **Manage access restrictions** and **Remove access restrictions**: see [Playbook components](/reference/opencti/playbook-components/).

## Authorized members and the alternatives

| You want to | Use |
| --- | --- |
| Decide what a user can do everywhere | [Grant rights with roles](/opencti/administer/roles/) |
| Hide classified data from users who are not cleared | [Markings](/opencti/administer/markings/) |
| Separate the data of the organizations you serve | [Separate data between organizations](/opencti/administer/organizations/) |
| Restrict one object to a few people | Authorized members |
| Let users ask for an object they cannot see | [Let users request access](/opencti/administer/request-access/) |

## What's next

- [How OpenCTI decides who sees what](/opencti/administer/segregation/), for how authorized members combine with markings and organizations.
- [Let users request access](/opencti/administer/request-access/), the workflow that shares restricted knowledge on request.
- [Access control](/opencti/administer/access-control/), the model behind capabilities.
