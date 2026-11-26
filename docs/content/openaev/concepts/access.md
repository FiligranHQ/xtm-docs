---
title: How access works in OpenAEV
description: How OpenAEV decides who can do what, through tenants, users, groups, roles, capabilities and grants, and where an administrator sets each of them.
doc_type: explanation
products:
  - openaev
personas: [admin, platform-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/administration/introduction/
  - openaev:/administration/users-and-rbac/
---

OpenAEV decides, for every user, which tenants they can open and what they can do in each of them. It uses role-based access control (RBAC): a user belongs to groups, each group holds roles, and each role is a list of capabilities. Grants then open a single scenario or simulation to a group. You set all of it under :menu[Settings > Security], and the rest of the platform under :menu[Settings]. This page explains the model, the roles a new tenant starts with, and the choices that keep the platform safe. The pages of the **Administer** tab then cover each task.

## What administration covers

| Menu | What you set there | Who sees it |
| --- | --- | --- |
| :menu[Settings > Parameters] | The name, theme, language and default dashboards of the tenant, and the version and services of the platform | **Access tenant settings** |
| :menu[Settings > Security] | Users, groups, roles, organizations, sessions and the login messages of the tenant; with the **Platform** scope, the users, groups and roles of the whole platform and its tenants | Any capability of the security group, **Access tenant settings** or **Access tenants** |
| :menu[Settings > Customization] | Default asset rules, custom domains, notifiers, lessons learned templates and autonomous attack | **Access tenant settings**, or **Access lessons learned** for the templates |
| :menu[Settings > Taxonomies] | Tags, attack patterns, kill chain phases and vulnerabilities | **Access tenant settings**, or **Access tags** for the tags |
| :menu[Settings > Data ingestion] | The XLS mappers that import injects into scenarios | **Access tenant settings** |
| :menu[Settings > Filigran Experience] | The Enterprise Edition license and the connection to XTM Hub | **Access platform settings** for the license, **Access tenant settings** for XTM Hub |

Deploying the platform, its configuration file and its shared services are in [Deploy & operate](/deploy/).

## How it works

::diagram{id=openaev-administer-access-model}

| Element | What it is | Where you set it |
| --- | --- | --- |
| Tenant | An isolated workspace: its own scenarios, simulations, assets, teams, findings, integrations and settings. A platform without the Enterprise Edition has one, the default tenant | :menu[Settings > Security > Tenants], with the **Platform** scope. See [Give each organization its own tenant](/openaev/administer/multi-tenancy/) |
| User | A person, or the account of an integration. One account can belong to several tenants | :menu[Settings > Security > Users] |
| Group | A set of users that receives roles and grants. A tenant group counts in its tenant only; a platform group counts in every tenant its members belong to | :menu[Settings > Security > Groups] |
| Role | A named set of capabilities. A tenant role holds tenant capabilities, a platform role holds platform capabilities | :menu[Settings > Security > Roles] |
| Capability | The right to do one kind of action, such as **Launch assessment** or **Manage tenants** | On the **Capabilities** tab of a role. The [capabilities reference](/reference/openaev/capabilities/) lists them all |
| Grant | The right of a group on one scenario, simulation, atomic testing or threat arsenal action | **Manage grants** on a group |

Six rules explain most of what you will see:

1. **Membership comes first.** A user opens a tenant only when they are a member of it, whatever their roles, administrators included. Without it, OpenAEV shows **Tenant access denied**.
2. **Capabilities add up, and include the level below.** A user gets every capability of every role of the groups that count in the tenant they work in: the groups of that tenant and their platform groups. **Delete assessment** includes **Manage assessment**, which includes **Access assessment**. No capability removes another.
3. **The scope decides where a capability works.** A tenant role grants rights on the content of its tenant: scenarios, assets, the tenant's settings, its users and groups. A platform role grants the platform rights: **Manage tenants**, **Manage platform settings**, and the users, groups and roles of the platform. **Bypass (user has all rights)** in a tenant role gives every tenant right in that tenant only. In a platform role, it gives every right, in every tenant the user belongs to.
4. **A grant opens one object, a capability opens them all.** A group with **Access assessment** sees every scenario, simulation and atomic testing of the tenant, and a group with **Access threat arsenal** every threat arsenal action. A group without them sees only the objects it has a grant on. A grant on a single threat arsenal action is enough to show the **Threat Arsenal** menu to the group's members.
5. **You give only what you hold.** In a role, a capability you do not hold shows with a padlock: you can remove it from a role that has it, but never add it back. A role that holds such a capability is locked in the role picker of a group, and the members of a group whose roles you do not fully hold cannot be changed. The API enforces the same rule. A user with **Bypass (user has all rights)** in that scope holds everything, so nothing is locked for them.
6. **The administrator flag bypasses every check.** The bootstrap admin, set by `OPENAEV_ADMIN_EMAIL`, carries it. It cannot be set in the interface, and single sign-on does not set it: it puts the administrators of your identity provider in the **Admin** group of their tenant. To make someone a platform administrator, give them a platform role with **Bypass (user has all rights)**.

Some reference data is readable by every member of a tenant, whatever their roles: teams and persons, organizations, tags, attack patterns, kill chain phases, vulnerabilities, security domains, injectors, XLS mappers and notifiers. The forms of a scenario need them, to pick a team or a tag. Creating, changing or deleting them still needs the capability, such as **Manage teams & persons** or **Manage tags**. Do not count on roles to keep the names, email addresses or phone numbers of the persons of a tenant from its other members.

## What a new tenant starts with

Each tenant you create gets three roles, each with a group of the same name. You, its creator, become a member of the tenant and of its **Admin** group.

| Group and role | Capabilities | For |
| --- | --- | --- |
| **Admin** | **Bypass (user has all rights)**, in this tenant | The administrators of the tenant |
| **Manager** | Access, manage and delete scenarios, simulations, atomic testings, teams and persons, assets, threat arsenal, findings, dashboards, reportings, documents, channels, phishing, challenges, lessons learned, security platforms and tags; **Launch assessment** | The people who build and run simulations |
| **Observer** | Access only, to the same content except teams, persons and tags | The people who read the results |

Neither **Manager** nor **Observer** holds the tenant settings or the tenant's users, groups and roles: only **Admin** manages the tenant itself. The default tenant has the same three groups, one of each: the bootstrap admin is a member of its **Admin** group, with the **Admin** role. A platform **Administrators** group, with a platform **Admin** role, holds the bootstrap admin too. An upgrade renames and deletes no group: a platform installed with an earlier release keeps the **Administrators** group of its default tenant, beside **Admin**, and the bootstrap admin stays in it.

## Users, groups and roles in a tenant

The Security menu opens on **This tenant**. With the Enterprise Edition and a platform capability, a switch at its top moves to **Platform**, where the same **Users**, **Groups** and **Roles** entries list the platform's own.

- **Users.** **Create** asks for an **Email address**, a **Firstname** and a **Lastname**, and optionally the **Organization**, **Tags**, phone numbers and **PGP public key** used by email injects. A new address gets an account, and OpenAEV emails a reset code so the person chooses a password. An address that already has an account is attached to the tenant and keeps its password. **Delete**, in **This tenant**, only detaches the user from the tenant and removes them from its groups; the account stays. In **Platform**, **Delete** removes the account and ends its sessions, and the **Tenants** field of a user sets the tenants it belongs to.
- **Groups.** A group has a name, a description and **Auto assign**: on, every user who joins the tenant joins the group, including users created by single sign-on. Its menu holds **Manage users**, **Manage roles** and **Manage grants**.
- **Grants.** **Manage grants** opens a tab per kind of object: **Scenarios**, **Simulations**, **Atomic testings** and **Threat Arsenal**. Check **Access** to read the object, **Manage+Delete** to change and delete it, and **Launch** to run it. Each level includes the one before it; threat arsenal actions have no **Launch**.
- **Roles.** A role has an **Overview** tab, its name and description, and a **Capabilities** tab, a tree where checking a capability checks the ones it includes.
- **Organizations.** The organizations of the tenant, such as a member company or a department, that users and teams belong to. They need **Access tenant settings**.
- **Sessions.** The open sessions, grouped by user, with their **Last activity** and when they were **Created**. **Kill session** signs one session out; **Kill all sessions**, shown for a user with several, signs that user out everywhere. It needs **Manage tenant sessions**, or **Manage platform sessions** with the **Platform** scope.

Some actions need two capabilities. To write a media pressure article in a scenario, a user needs **Manage assessment** and **Access channels**; without the second, the form says which capability is missing.

:::example[Work out what Grand Meridian's people can do]
Example CERT runs one OpenAEV instance for its member hotel chains, with a tenant for each. In the **Grand Meridian Hotels** tenant:

- Hélène Marchetti, the CISO, is in **Observer**. She reads the results of every simulation, but cannot change or launch anything.
- The security engineer of Grand Meridian is in **Manager**. He builds the scenario that replays Velvet Harbor against the front-desk workstations, launches it and reads its expectations.
- A front-desk supervisor of the Paris hotel must run one simulation on the night shift without seeing the others. He joins a new group, **Paris night shift**, with no role. In **Manage grants**, **Simulations**, the group gets **Launch** on the simulation of the fake booking request for the European Hospitality Summit 2026. He sees that simulation, and nothing else.
- Example CERT's own analysts belong to the **Example CERT** tenant only. They see nothing of Grand Meridian until an administrator attaches them to that tenant and puts them in one of its groups.
:::

## Secure choices first

| Choice | Why |
| --- | --- |
| Keep **Bypass (user has all rights)** to the **Admin** groups, and the platform one to a few named people | In a platform role it opens every tenant the user belongs to, with every right |
| Keep the bootstrap admin for emergencies | Its password comes from `OPENAEV_ADMIN_PASSWORD` and is applied at every start. See [Sign in the first time and keep an emergency account](/deploy/configure/first-access/) |
| Turn **Auto assign** on only for a group with a narrow role, such as **Observer** | Every user who joins the tenant gets it, including every single sign-on login |
| Prefer grants to a broad capability for occasional users | A grant opens one simulation; **Launch assessment** opens them all |
| Map groups from your identity provider | Group memberships then follow the directory at each login. See [Configure single sign-on](/openaev/administer/sso/) |
| Give integrations their own account | Its token then carries the rights of its own groups, not those of a person. See [Run an integration without XTM Composer](/deploy/integrations/manual/) |

## The administration tasks

| To | Read |
| --- | --- |
| Separate the organizations you serve | [Give each organization its own tenant](/openaev/administer/multi-tenancy/) |
| Log users in with your identity provider | [Configure single sign-on](/openaev/administer/sso/) |
| Show a message and ask for consent on the login page | [Set the login message and the consent box](/openaev/administer/policies/) |
| Name and brand a tenant, choose its dashboards | [Set the name, look and home page of a tenant](/openaev/administer/parameters/) |
| Keep tags, attack patterns and vulnerabilities in order | [Manage the tags, attack patterns and vulnerabilities](/openaev/administer/taxonomies/) |
| Deploy content from XTM Hub | [Connect a tenant to XTM Hub](/openaev/administer/xtm-hub/) |
| Protect the links sent to players | [Protect the links in player emails](/openaev/administer/url-access-token/) |
| Start the platform without background work | [Start the platform in safe mode](/openaev/administer/run-modes/) |
| Record who logged in and who changed what | [Keep an audit log of who changed what](/openaev/administer/audit-log/) |
| Find what slows the platform down | [Turn on the debug mode to find slow requests](/openaev/administer/debug-mode/) |
| See what is sent to Filigran | [Check or stop the usage telemetry sent to Filigran](/openaev/administer/telemetry/) |
| Fix a problem | [Troubleshoot OpenAEV](/openaev/administer/troubleshoot/) |

## What's next

- [Give each organization its own tenant](/openaev/administer/multi-tenancy/), to set up the tenants first.
- [Configure single sign-on](/openaev/administer/sso/), so that accounts and groups follow your directory.
- The [capabilities reference](/reference/openaev/capabilities/), every capability a role can grant, by scope.
