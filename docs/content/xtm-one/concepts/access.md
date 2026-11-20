---
title: How access works in XTM One
description: The two roles of XTM One, how accounts are created and disabled, what groups and group managers do, and how sharing decides who reaches each agent and resource.
doc_type: explanation
products:
  - xtm-one
personas: [admin]
owner: docs-xtm-one
last_reviewed: 2026-11-16
migrated_from:
  - xtm-one:/administration/introduction/
---

XTM One has two roles, **Administrator** and **User**, and no finer permission set: what a user reaches is decided by who owns a resource and with whom it is shared. As an administrator, you create the accounts, organize them in groups, choose who follows a team's AI usage, and decide what the whole company may use. This page explains that model, the **Users and groups** screen of the **Administration** menu, and what to check after a change.

::diagram{id=xtm-one-administer-access}

## Two roles

| Role | What it gives |
| --- | --- |
| **User** | Chats with the agents they may reach, builds their own agents, knowledge bases, prompts and tools, and shares them. Sees no administration menu. |
| **Administrator** | Everything a user does, plus the **Administration** menu (**Observability**, with **Model usage**, **Outcomes**, **Traces** and **Sessions**, then **AI Adoption**, **Platform Health**, **Users and groups**, **Objects**, **Audit logs** and **Settings**) and, on a platform in its XTM One mode, **Intents** in the **Orchestration** menu. Only an administrator edits a company-managed resource. |

The role is the only switch. **Promote** and **Demote**, on a user's page, change it, and every change is recorded in **Audit logs**.

Three accounts get the administrator role without an administrator choosing it:

- **The bootstrap account.** The account of `ADMIN_EMAIL` (`admin@filigran.io` when unset) is created at the first start and reset at every start: its password goes back to `ADMIN_PASSWORD`, its role to administrator, and the account is enabled again. It shows a **Platform Admin** badge, and nobody can demote, disable or delete it. Change its password in the deployment, never on the screen.
- **The first account.** Whichever way the first account of the platform is created, it is an administrator.
- **Single sign-on, when you ask for it.** With **Automatically admin users** on, everyone who signs in through that method becomes an administrator at each sign-in. See [Configure single sign-on](/xtm-one/administer/sso/).

:::warning[Keep an administrator who can always sign in]
The bootstrap account keeps working when the identity provider is down, as long as **Local Authentication** stays on. Store its password where your emergency procedures are.
:::

## Users

:menu[Administration > Users and groups] opens on the **Users** tab, a table of every account with its email, its role (**Admin** or **User**, and **Platform Admin** for the bootstrap account) and its creation date. **Columns** brings in the others: **Status** (**Active** or **Disabled**), **Sign-in method**, and how many integrations, MCP servers, agents and knowledge bases the person owns. Search by name or email, filter by **Role**, **Status**, **Created** (the last 24 hours, 7, 30 or 90 days) and **Groups**, and download the list with **Export CSV**. With a filter on, the total shows how many accounts match out of all of them. Clicking a row opens the person's page.

::screenshot{id=xtm-one-administer-users alt="The Users tab of Users and groups: the accounts in a table with their email, role and creation date, Hélène Marchetti with the admin shield and the Admin badge."}

### Create an account

Four ways bring a person in:

- **New User** creates the account now. Choose **Local Account** to set an email and a password yourself, or **SSO Pre-provision** to create it without a password: the person's first single sign-on links to it by email, with the role you set. **SSO Pre-provision** is offered once OpenID Connect or SAML is on. Turn on **Administrator** to create an administrator.
- **Invite user** emails a single-use link bound to one address and one role. The invitee sets their own password, and you never see it. The link lasts 7 days by default (`INVITATION_EXPIRE_HOURS`, at most 30 days). An invitation works even while **Local Registration** is off or the email domain is not allowed. **Pending invitations**, in the same dialog, lists the links not used yet, and revokes one.
- **Single sign-on** creates the account at the first sign-in, unless you turned on-the-fly creation off. See [Configure single sign-on](/xtm-one/administer/sso/).
- **SCIM provisioning** lets an identity provider such as Microsoft Entra create, update and deactivate accounts through System for Cross-domain Identity Management (SCIM), from a token you generate in :menu[Settings > Access and security > Provisioning]. It creates a **User** without a password, and links an account that already has the same email instead of creating a second one. See [Set up the platform in Settings](/xtm-one/administer/settings/#access-and-security).

A local password has at least 8 characters by default (`MIN_PASSWORD_LENGTH`), with a letter and a digit. Invitations and password reset emails need an SMTP server: when it refuses the message, the dialog says so and the link stays pending.

### A user's page

| Tab | What it holds |
| --- | --- |
| **Profile** | Name, email, role, sign-in method, the password (**Set a password directly** or **Email a reset link**) and **Two-factor authentication** |
| **Groups** | The groups the person belongs to: **Add to Group**, **Remove from group** |
| **Objects** | What the person owns: integrations, MCP servers, agents, knowledge bases, custom tools |
| **API Keys** | Their personal API keys, with **Revoke** |
| **Quotas** | Their own agentic limit, which wins over their groups' and the platform default. Requires the Enterprise Edition |
| **Activity** | Their entries of the audit log, in the table of [Audit logs](/xtm-one/administer/audit-logs/): search them and filter them by **Action**, **Entity** and **Time range** |

The secret of an authenticator app and its recovery codes are never readable, not even by an administrator. When a person loses their device, **Reset two-factor** clears the enrollment: they sign in with their password alone until they enroll a new one.

### Disable, impersonate, delete

- **Disable** stops the person from signing in and stops everything that runs as them. It first opens **Disable {name}?**, which lists under **What happens** what the account will lose, then **Disable account** does it. See [What disabling an account does](#what-disabling-an-account-does).
- **Enable** gives the account back its sign-in, its private agents, its paused assignments and the objects moved away from it. See [Enable an account again](#enable-an-account-again).
- **Impersonate** signs you in as the person, to see what they see. A banner shows it the whole time, every action is recorded in **Audit logs** as yours, and the impersonation ends with your own session. A disabled account cannot be impersonated.
- **Delete** removes the account and everything it owns: agents, assignments, runs, conversations, integrations, knowledge bases, MCP servers, custom tools, API keys and audit entries. It cannot be undone. Transfer what the team still needs first.

You cannot disable or delete your own account. Disable rather than delete when someone leaves: the account, its history and its shares stay, and nothing runs as the person any more.

### What disabling an account does

Every way of disabling an account runs the same steps: **Disable** on the person's page, the identity provider through SCIM (an account set inactive, or removed: XTM One then deactivates it and purges nothing), and an AI agent with the **User Administration** tools. The dialog counts each step before you confirm:

| Step | What happens |
| --- | --- |
| Sign-in and keys | The person can no longer sign in, and their personal API keys are revoked. |
| Live work | Their running and queued runs stop, their armed timers and their scheduled actions are cancelled. |
| Private agents | The agents they own that are neither shared nor company-managed are disabled, with their assignments. |
| Shared objects | Their shared and company-managed agents, fleets, workspaces and integrations move to the platform administrator: the `ADMIN_EMAIL` account when it is active, else the oldest active administrator. |
| Assignments run as them | On a moved agent, the assignments that run as the person are paused; the others keep running as the new owner. |
| Integrations | Their private integrations are disconnected, and their personal sign-ins to shared integrations are removed. |

Their group memberships and the shares that name them are kept, for audit and for a later **Enable**. Their private fleets and workspaces stay with them: nothing in them runs. When no other active administrator exists, the shared objects stay with the account, and the dialog warns about it.

Every active administrator gets a notification and an email. Each object the steps moved, paused, disconnected or had to keep with the account, and another person's agent that used one of the disconnected integrations, waits in the **Deactivation review** card of :menu[Settings > Access and security > Provisioning]. **Mark as kept**, **Mark as reassigned** or **Mark as deleted** records what you decided, once you have done it: marking an object changes nothing else.

On your own page, the dialog only says "Cannot deactivate yourself", and **Disable account** stays off. The bootstrap account, marked **Platform Admin**, has no **Disable** at all.

::screenshot{id=xtm-one-administer-user-disable alt="The Disable Sophie Laurent? dialog: under What happens, the count of shared agents that move to the platform administrator, then the Disable account button."}

### Enable an account again

**Enable** runs no dialog. It gives back the sign-in, turns the private agents on again, resumes the assignments paused because of the person, and returns the moved objects to them. An object comes back only if the successor still owns it and no administrator marked it in **Deactivation review**. Revoked API keys, cancelled timers and scheduled actions, stopped runs and disconnected integrations stay ended: the person creates new keys and connects their integrations again.

## Groups

A group gathers people to share resources with them at once, to give them a quota, to let a manager follow their AI usage, or to switch AI off for them. The **Groups** tab of :menu[Administration > Users and groups] lists the groups with their description, number of members and creation date, and their **Kind** on a wide screen. Search by name or description, filter by **Kind** and **Members** (**Has members** or **Empty**), and download the list with **Export CSV**. **New Group** asks for a **Name** and a **Description**.

A group's page has four tabs:

- **Overview**: its name, description, **Kind** and AI access. **Kind** gives no access: it sorts the group in **Model usage** and in the **Kind** filter of the **Groups** tab. It is **Org unit**, **Access list**, **Distribution list** or **Not classified**.
- **Members**: a table of the members with their email, role, a **Manager** badge for the group's managers, and the date they were **Added**. Search it and filter it by **Role** and **Manager** (**Managers** or **Other members**); a row opens the person's page. **Add a member...**, above the table, picks a person who is not in the group yet. **Remove from group**, on a row, takes one out. Before you add or remove someone, XTM One lists what the group's shares lend them or take from them.
- **Managers**: the people who follow the group's AI usage.
- **Quotas**: an agentic limit for each member, which a member's own limit overrides. Requires the Enterprise Edition.

With SAML, groups can follow your directory: the groups of the assertion are synchronized at each sign-in, and created when missing. See [Configure single sign-on](/xtm-one/administer/sso/#map-saml-groups).

### Group managers

A group manager follows the AI usage of the group's current members in **Model usage**: the agents they use, the costs, the errors, exported as CSV. A manager does not have to be a member, and gets no other right: the **Administration** menu shows them **Model usage** only, narrowed to the groups they manage. Add one with **Add a manager...** on the **Managers** tab.

### Switch AI off for a group

The **AI access** card of a group's **Overview** turns AI off for its members alone, where the platform switch turns it off for everyone (see [Run XTM One with AI switched off](/xtm-one/administer/running-without-ai/)). Turn on **AI disabled for this group**: the change applies at once, without **Save changes**, and **Audit logs** records it as "Switched AI off for group" with the group's name.

::screenshot{id=xtm-one-administer-group-ai-access alt="The AI access card of the Outside auditors group, with AI disabled for this group turned on."}

- **What members lose.** No chat, no agents, and nothing runs a model on their behalf: the work done for them in the background, such as an assignment run acting as them or a task they delegated, is held as on a platform with AI off. This holds whatever their other groups say, and administrators who are members are held to it too.
- **What keeps working.** Everything that needs no model: their agents, knowledge bases, flows, runs and traces stay readable, flows without an AI step keep running, and an administrator held by it still reaches every setting.
- **What members see.** Their home reads **AI is switched off for your group**, and the AI actions are disabled with "AI is disabled for your group. Ask an administrator if you need it.".

AI comes back for a person when you turn the switch off, remove them from the group, or delete the group.

## Sharing decides what a user reaches

Every agent, knowledge base, prompt, skill, integration, MCP server, custom tool and variable has an owner, and is private to them until it is shared. Each list of the platform shows three tiers, administrators included: your own items, the items shared with you, and the items the company publishes.

- **Shared with people and groups**: the owner picks each person or group and their access, **Read only** (view and use), **Can configure** (also edit the configuration) or **Full access (incl. chats)** (also the other users' chat history and the agent's traces). A person reached both directly and through a group gets the stronger access.
- **Company-managed**: visible to every user, and edited by administrators only. **Restrict access** keeps a company-managed resource to the people and groups you list.
- **Built-in**: the agents XTM One ships are company-managed and read-only, even for administrators.

Sharing a resource can lend what it carries, such as an integration's credential. XTM One says so before the change: see [Share agents and resources](/xtm-one/build/sharing/).

Administrators find any item, whoever owns it, fleets included, in :menu[Administration > Objects]. Search its table and filter it by **Type**, **Owner**, **Status** and **Scope** (**Company-managed** or **Personal**); a row opens the item. Opening another person's agent shows it whole, with what it is bound to. Reading the delegations of another user's agent, or the command history of another person's machine, is recorded in **Audit logs** as a `break_glass` entry.

## Check after a change

- **Audit logs** records every role change, account toggle, impersonation, group change, the AI access of a group included, and sharing change, with who did it.
- A user's **Objects** tab shows what they own before you disable or delete them, and **Disable {name}?** counts what disabling will stop and move.
- The **Deactivation review** card of :menu[Settings > Access and security > Provisioning] lists what each deactivation moved, disabled or kept, until you mark it.
- The **Security audit** of Settings lists the active accounts nobody signed in to for 30 days that still own an automation. See [Set up the platform in Settings](/xtm-one/administer/settings/#access-and-security).

:::example[Organize the security teams of Grand Meridian Hotels]
Hélène Marchetti, the CISO, administers XTM One at `https://xtm-one.grand-meridian.example`, which signs people in through Microsoft Entra ID.

1. On the **Groups** tab of :menu[Administration > Users and groups], she creates **CTI team**, **SOC** and **Crisis cell**, and sets the **Kind** of each to **Org unit**.
2. On the **Users** tab, she adds Camille Roux to **CTI team**, Paul Garnier to **SOC**, and Julien Bernard to **SOC** and **Crisis cell**. Sophie Laurent, the AI engineer, gets no group: she builds the agents.
3. On the **Managers** tab of **SOC**, she adds Julien Bernard, the SOC manager: he follows the night shift's AI costs in **Model usage**, and nothing more.
4. Sophie shares her agent "Night shift triage" with **SOC**, **Read only**, and "Velvet Harbor watcher" with **CTI team**, **Read only**. Paul and Camille chat with them; neither can change their instructions.
5. Hélène makes "Crisis briefer" company-managed, with **Restrict access** and **Crisis cell** as its only audience.
6. For the firm that reviews the hotels' access controls each quarter, she creates **Outside auditors** and turns on **AI disabled for this group** in its **AI access** card: the auditors read what is shared with them, and no model runs on their behalf.

When Paul leaves the night shift, Hélène removes him from **SOC**: XTM One first lists what the group's shares stop lending him, "Night shift triage" included.

When Sophie leaves Grand Meridian Hotels, Hélène opens her page and clicks **Disable**. Under **What happens**, **Disable Sophie Laurent?** counts the shared agents that move to the platform administrator. Hélène clicks **Disable account**, then finds "Night shift triage" and "Velvet Harbor watcher" in **Deactivation review**, marked **Moved**, and clicks **Mark as kept** on each: the SOC and the CTI team keep chatting with them.
:::

## What's next

- [Configure single sign-on](/xtm-one/administer/sso/)
- [Set up the platform in Settings](/xtm-one/administer/settings/)
- [Share agents and resources](/xtm-one/build/sharing/)
- [Route requests to the right agent with intents](/xtm-one/administer/intents/)
