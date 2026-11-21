---
title: How OpenCTI decides who sees what
description: How OpenCTI decides who sees each object of one shared knowledge base, with markings, organization sharing and authorized members, and which one to use.
sidebar_position: 20
doc_type: explanation
products:
  - opencti
personas: [admin, cti-lead, evaluator]
owner: docs-opencti
last_reviewed: 2026-11-01
---

OpenCTI keeps one knowledge base and decides, object by object, who may see it. Three mechanisms do it: markings, organization sharing and authorized members. This page explains how they combine, so you pick the right one before you create users and share data. You set them in :menu[Settings > Security] and on the objects themselves.

## What each mechanism is good for

| Goal | Mechanism | Example |
| --- | --- | --- |
| Hide data above a user's clearance | Markings | Analysts cleared up to TLP:AMBER never see TLP:RED reports |
| Serve member organizations from one intelligence base | Organization sharing (Enterprise Edition) | An ISAC shares selected reports with each member company |
| Keep one sensitive object to a few people | Authorized members | An incident response case visible to three named responders |

## How OpenCTI decides who sees an object

Every object is stored once. When a user opens a list or an object, OpenCTI runs up to three checks and shows the object only if they let the user through. Users with the **Bypass all capabilities** capability skip every check.

::diagram{id=admin-segregation-access}

1. **Markings.** The user needs access to every marking on the object, not only one. Access comes from the groups of the user. A marking also opens the markings of the same type with a lower order: a group allowed TLP:AMBER sees TLP:GREEN and TLP:CLEAR. This check is always on. See [Restrict access with markings](/opencti/administer/markings/).
2. **Authorized members.** When an object is restricted to a list of users, groups and organizations, only those members get through. See [Restrict one object to chosen people](/opencti/administer/sharing/).
3. **Organization sharing** (Enterprise Edition). It starts when you set a platform main organization in [Set platform security policies](/opencti/administer/policies/). Members of the main organization see all the data their markings allow. Members of other organizations see only what is shared with one of their organizations. An object restricted with authorized members skips this check: its members see it even when it is not shared with their organization. See [Separate data between organizations](/opencti/administer/organizations/).

What surprises people:

- **A shared object is still one object.** Every organization it is shared with sees the same version, and an edit is visible to all of them. Put organization-specific comments in a note, and share the note.
- **Sharing an entity does not share its relationships or neighbors.** A partner may see a system and an IP address but not the relationship between them, and their dashboards stay empty. Sharing a container, such as a report or a case, with **Share with an organization** also shares its contents.
- **Visibility adds up.** A user in several organizations and groups sees the union of what each one opens.
- **Some types are never segregated by organization.** Organizations, sectors and locations, and objects such as labels and markings, are visible to every user who can access knowledge, unless authorized members restrict an organization.
- **Hiding entity types is not a restriction.** [Hidden entity types](/opencti/configure/parameters/) only declutter the interface; the API still serves them.

:::example[Let a hotel chain see only what you release to it]
Example CERT runs one OpenCTI platform for the hotel chains it serves. The CTI team of Grand Meridian Hotels must read the intelligence the CERT releases to it, and nothing about the other chains.

- In [Set platform security policies](/opencti/administer/policies/), the **Platform organization** of the **Platform main organization** card is Example CERT. The CERT's own analysts belong to it: they see everything their markings allow.
- The Grand Meridian analysts get **Organizations**: Grand Meridian Hotels, and **Groups**: CTI analysts, a group allowed markings up to TLP:AMBER.
- On the report on Velvet Harbor's attacks against hotel booking systems, an analyst of the CERT uses **Share with an organization** and picks Grand Meridian Hotels. The entities and relationships the report contains are shared with it too.
- The organization entity of each other chain is restricted with authorized members to Example CERT, so that Grand Meridian does not see who else the CERT serves.

The Grand Meridian analysts now see that report and its contents, as long as they are marked TLP:AMBER or lower. Reports the CERT shares with another chain, or with no one, stay invisible to them, and so does a TLP:RED report even when it is shared with them.
:::

## Markings, organizations and authorized members

| You need | Use |
| --- | --- |
| Users who must not see data above their clearance | Markings |
| Partners who read the same intelligence, each seeing only what you release to them | Organization sharing |
| Customers who must not know each other exist | Organization sharing, plus authorized members on each organization entity |
| One sensitive object hidden from everyone else, even inside the main organization | Authorized members |
| Data that must never sit in the same database | Separate platforms: no mechanism replaces that |

## What's next

- [Restrict access with markings](/opencti/administer/markings/) to set clearances.
- [Separate data between organizations](/opencti/administer/organizations/) to set up organizations and share with them.
- [Restrict one object to chosen people](/opencti/administer/sharing/) to restrict one object.
- [Access control](/opencti/administer/access-control/) for users, groups and roles.
