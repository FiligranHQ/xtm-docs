---
title: How access control works
description: How OpenCTI decides what each user can do and see, through users, groups, roles, capabilities, organizations and markings, and where to set each of them.
sidebar_position: 1
doc_type: explanation
products:
  - opencti
personas: [admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/users/
  - opencti:/administration/introduction/
---

OpenCTI decides, for every user, what they can do and which data they can see. It uses role-based access control (RBAC): a user belongs to groups, each group carries roles, and each role is a list of capabilities. Markings and organizations then limit the data each user sees. You set all of it in :menu[Settings > Security], which this part of the documentation covers.

## What access control is good for

| Goal | Example |
| --- | --- |
| Give each team the actions it needs | Analysts create and update knowledge; only leads delete or merge |
| Keep sensitive data to the people cleared for it | Only the incident response group sees data marked TLP:RED |
| Separate the entities you serve | Each member organization sees only what is shared with it |
| Set people up at their first login | A single sign-on login puts users in the group that matches their directory group |
| Protect the platform from mistakes | Only a few administrators can change the built-in roles or the platform organization |

## How it works

::diagram{id=admin-access-control-model}

| Element | What it is | Where you set it |
| --- | --- | --- |
| User | A person, or a service account for a machine | :menu[Settings > Security > Users] |
| Group | A set of users that receives roles, allowed markings, a maximum confidence level and defaults | :menu[Settings > Security > Groups] |
| Role | A named set of capabilities | :menu[Settings > Security > Roles] |
| Capability | The right to do one kind of action, such as **Delete knowledge** or **Manage credentials** | In a role |
| Marking definition | A label on data, such as TLP:AMBER, that only the groups allowed to see it can see | :menu[Settings > Security > Marking definitions] |
| Organization | A segregation unit: users see the data shared with their organizations | :menu[Settings > Security > Organizations] |

Five rules explain most of what you will see:

1. **Capabilities add up.** A user gets every capability of every role of every group they belong to. No capability removes another: to give fewer rights, use a group with a narrower role.
2. **Groups carry everything.** Roles, allowed markings, the maximum confidence level, the default dashboard and the hidden entity types are set on groups. A user in no group with a role can do nothing.
3. **A capability says what, markings and organizations say which.** A user who can update knowledge still sees only the objects whose markings their groups allow and, with organization segregation, the objects shared with their organization.
4. **Confidence limits changes.** A user creates or changes data only up to their maximum confidence level: the highest of their groups, unless one is set on the user. Without one, they cannot write at all.
5. **Bypass all capabilities bypasses almost everything.** A user with it has every capability, sees all data and has a maximum confidence level of 100. Only the right to change [sensitive configuration](/opencti/administer/sensitive-configuration/) is set apart.

[How OpenCTI decides who sees what](/opencti/administer/segregation/) explains how markings, organizations and authorized members combine to decide who sees an object.

:::example[Work out what an analyst of a member hotel can do and see]
Example CERT runs the platform for the hotel sector and has set itself as the platform main organization. Ana Duval, of Grand Meridian Hotels, belongs to one organization and one group:

- **Organization**: Grand Meridian Hotels.
- **Group**: CTI analysts, with the Analyst role (**Access knowledge**, **Create / Update knowledge**, **Upload knowledge files**, **Import knowledge**, **Ask for knowledge enrichment**), the allowed markings TLP:CLEAR, TLP:GREEN and TLP:AMBER, TLP:GREEN as maximum shareable marking, and a maximum confidence level of 75.

Applying the five rules:

- **What she can do.** She creates reports and indicators and updates them, but cannot delete or merge anything: no group of hers gives **Delete knowledge** or **Merge knowledge**.
- **Which data she sees.** The objects shared with Grand Meridian Hotels, plus the types that are never segregated, such as sectors and locations, and among them only those marked up to TLP:AMBER. The Velvet Harbor report that Example CERT shared with the hotel shows; a TLP:RED incident case of Example CERT does not, even if someone shares it with the hotel.
- **How much she can change.** An indicator another analyst rated at a confidence of 80 stays read-only for her: 80 is above her 75.
- **What she can release.** Her exports and public dashboards carry data up to TLP:GREEN only.

To let her delete only what she created, do not add **Delete knowledge** to the Analyst role, which every analyst shares. Give her a second group whose role has it, and turn on **Do not allow these users to delete content created by other users** on both her groups: the option applies only when every group of a user has it. See [Organize users in groups](/opencti/administer/groups/#group-options).
:::

## Secure choices first

Each choice below closes a gap that administrators often find too late:

| Choice | Why |
| --- | --- |
| Keep **Bypass all capabilities** to a few named administrators | It opens all data, whatever its markings and organizations, and every setting. Protect the built-in groups and roles too, with [sensitive configuration](/opencti/administer/sensitive-configuration/) |
| Keep the Default group narrow | Every new user joins it unless told otherwise. Out of the box it gives **Access knowledge** and no marking: users see only unmarked data until another group opens more |
| Map groups from your identity provider, and turn on **Prevent platform default groups association** | Rights then come from the directory at each login, not from memberships added by hand and forgotten. Read [how a login provisions the account](/opencti/administer/sso/#how-a-login-provisions-the-account) for the exact rules |
| Set a **Max Confidence Level** below 100 on analyst groups | Data from your most reliable sources then cannot be overwritten by mistake. Groups created by single sign-on get `app:group_confidence_level:max_confidence_default`, 100 unless you lower it |
| Set **Maximum shareable marking definitions** on every group | Without it, members can export everything they can read |
| Give **Restrict organization access** to a few roles | With organization segregation, it decides who releases data to other organizations |

## Pages of this section

| To | Read |
| --- | --- |
| Create, disable or delete accounts, and end sessions | [Manage users](/opencti/administer/users/) |
| Give users their rights, markings and confidence level | [Organize users in groups](/opencti/administer/groups/) |
| Look up what each capability allows | [Grant rights with roles](/opencti/administer/roles/) and the [capability reference](/reference/opencti/capabilities/) |
| Give connectors and scripts an account | [Create accounts for integrations](/opencti/administer/service-accounts/) |
| Log users in with your identity provider | [Configure single sign-on](/opencti/administer/sso/) and [Manage login providers in the UI](/opencti/administer/authentications/) |
| Limit who changes the settings that can break the platform | [Protect sensitive configuration](/opencti/administer/sensitive-configuration/) |
| Decide who sees which object | [How OpenCTI decides who sees what](/opencti/administer/segregation/), [Separate data between organizations](/opencti/administer/organizations/), [Restrict access with markings](/opencti/administer/markings/) and [Restrict one object to chosen people](/opencti/administer/sharing/) |
| Let users ask for data they cannot see | [Let users request access](/opencti/administer/request-access/) |
| Set the platform organization, login messages and other rules | [Set platform security policies](/opencti/administer/policies/) |
| Set session cookies and the mail server | [Cookies](/opencti/administer/cookies/) and [Send emails from OpenCTI](/opencti/administer/smtp/) |

## Who sees the Security menu

:menu[Settings > Security] shows each page only to the users whose role has its capability:

| Capability | Pages |
| --- | --- |
| **Manage credentials** | **Users**, **Groups**, **Organizations**, **Roles**, **Sessions**, **Policies**, **Email templates** and **SMTP configuration** |
| **Manage authentication** | **Authentications** |
| **Manage marking definitions** | **Marking definitions** |
| **Manage dissemination lists** | **Dissemination list** |

:::only{opencti="<7.260811"}
**SMTP configuration** does not exist in this version.
:::

:::only{opencti=">=7.261127"}
An organization administrator without **Manage credentials** sees **Users** and **Organizations**, limited to the organizations they administer, and **Email templates**. They can read the templates; creating, changing, deleting or test-sending a template still requires **Manage credentials**.

A Security page opened by its URL follows the same rules as the menu: a user whose capabilities do not show the page is sent to the first **Settings** page they can open, or to the home page when there is none. The menu and the URL only decide what the screen shows: the API checks the capability of every request, whatever the screen.
:::

:::only{opencti="<7.261127"}
An organization administrator without **Manage credentials** sees only **Users** and **Organizations**, limited to the organizations they administer.
:::

## What's next

- [Organize users in groups](/opencti/administer/groups/), to set up the groups users belong to.
- [Manage users](/opencti/administer/users/), to create accounts.
- [Grant rights with roles](/opencti/administer/roles/), to create roles, and the [capability reference](/reference/opencti/capabilities/), everything a role can grant.
