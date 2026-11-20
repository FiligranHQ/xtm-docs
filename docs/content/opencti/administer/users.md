---
title: Manage users
description: Create, edit, disable, merge and delete OpenCTI users, read what each user can do and has done, manage their API tokens, and end their sessions.
sidebar_position: 2
doc_type: how-to
products:
  - opencti
personas: [admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/users/
---

Every person and every machine that uses OpenCTI has a user account, listed in :menu[Settings > Security > Users]. This page shows how to create, change, disable and delete users, how to read what a user can do and has done, and how to control their API tokens and sessions. A user's rights come from their groups: read [How access control works](/opencti/administer/access-control/) first if the model is new to you.

## Before you start

- Your role has the **Manage credentials** capability. An organization administrator manages only the members of the organizations they administer: see [Separate data between organizations](/opencti/administer/organizations/).
- The groups you want to give exist. See [Organize users in groups](/opencti/administer/groups/).

## Kinds of accounts

| Kind | How it logs in | How it is created |
| --- | --- | --- |
| User | Email and password, or single sign-on | By hand, or at the first single sign-on login |
| Service account | API token only, never the web interface | By hand, or by an integration. See [Create accounts for integrations](/opencti/administer/service-accounts/) |

The page of a service account shows **Account type**: **Service account**. For a user created by a single sign-on login, the **Password** tab is disabled: their password lives in the identity provider.

## Create a user

With single sign-on, you do not create users: OpenCTI creates each account at its first login. See [Configure single sign-on](/opencti/administer/sso/#how-a-login-provisions-the-account).

:::steps

1. Open :menu[Settings > Security > Users] and click **Create User**.
2. Enter a **Name** and an **Email address**. **Firstname**, **Lastname** and **Description** are optional.
3. Enter a **Password** and its **Confirmation**. The password must follow the password policy, set on the local strategy in [Manage login providers in the UI](/opencti/administer/authentications/#configure-the-built-in-strategies).
4. Choose the **Organizations** and the **Groups** of the user. The user also joins the groups granted by default at user creation, unless you turn on **Don't add the user to the default groups**. The tooltip next to it names those groups.
5. Optionally, change the **Account Status** (**Active** by default), set an **Account Expire Date**, or choose an **Email template** to send the user.
6. To give this user a maximum confidence level of their own, instead of the one their groups give, turn on **Enable user Max Confidence Level** and set the level.
7. Click **Create**.

:::

:::example[Give a new analyst access to the CTI team's work]
Leo Martin joins the threat intelligence team of Grand Meridian Hotels, which signs in with local accounts.

- **Name**: `Leo Martin`. **Email address**: `leo.martin@example.com`. **Firstname**: `Leo`. **Lastname**: `Martin`.
- **Password** and **Confirmation**: a temporary password that follows the policy, given to Leo by phone.
- **Organizations**: Grand Meridian Hotels. **Groups**: CTI analysts.
- **Don't add the user to the default groups**: on, so that Leo gets only the rights of CTI analysts.
- **Enable user Max Confidence Level**: off, so that Leo inherits 75 from CTI analysts.

The user page then shows the Analyst role, the CTI analysts group and a **Max Confidence Level** of 75: Leo can read and write knowledge up to TLP:AMBER.
:::

::screenshot{id=opencti-users-user-page alt="The page of the user Ana Duval: her email address, 2FA state, API tokens and account status under Basic information, and her Analyst role, CTI analysts group and Max Confidence Level of 75 under Permissions."}

:::warning
A user needs a maximum confidence level, from a group or set on the user, to create, update or delete data. Check the **Max Confidence Level** on the user page after you create the user.
:::

## Edit a user

Open the user and click **Update**. The panel has five tabs:

| Tab | What you change |
| --- | --- |
| **Overview** | **Name**, **Email address**, **Firstname**, **Lastname**, **Description**, **Language**, **Organizations**, **Account Status** and **Account Expire Date** |
| **Password** | A new **Password** and its **Confirmation**. Disabled for single sign-on users and for the platform administrator, whose password comes from `app:admin:password` |
| **Groups** | The groups of the user. Each group shows its maximum confidence level, or a warning when it has none |
| **Organizations admin** | The organizations this user administers, among their own organizations. Requires Enterprise Edition |
| **Confidences** | A **Max Confidence Level** for this user, and a level per entity type with **Add a specific max confidence level for an entity type** |

:::only{opencti=">=7.261109"}
A service account has no password: its panel has no **Password** tab.
:::

:::only{opencti="<7.261109"}
A service account has no password: its **Password** tab is disabled.
:::

A level set on the user replaces the levels of their groups, even when it is lower, and also replaces the per-type levels of the groups. Without one, the user gets the highest level among their groups.

To add or remove groups or organizations for many users at once, select them in the list, click **Update** in the toolbar, choose **Add** or **Remove**, then **Groups** or **Organizations**.

Click **Convert** on the user page to turn a user into a service account, or back, one account at a time. See [Create accounts for integrations](/opencti/administer/service-accounts/#convert-a-user-into-a-service-account).

:::only{opencti=">=7.260811"}
To make a user choose a new password at their next request, click **Force password change** in the **Password** tab. The user page then shows the date under **Password valid until**. Passwords can also expire on their own: see the password policy in [Manage login providers in the UI](/opencti/administer/authentications/#configure-the-built-in-strategies).
:::

::::only{opencti=">=7.261117"}
### Set a password the user has not used before

When the local strategy sets **Number of recent passwords that cannot be reused (0 equals disabled)**, the password you set in the **Password** tab must differ from the user's current password and their recent ones. See [Stop users from reusing a password](/opencti/administer/authentications/#stop-users-from-reusing-a-password) for the rule itself. The **Password security policies** box above the field says how many: **Must be different from the last 5 passwords** for a value of 5, **Must be different from the current password** for 1.

::screenshot{id=opencti-users-password-history alt="The Password tab of the Update User panel for Leo Martin: the Password security policies box reads Must be different from the last 5 passwords, above the Password and Confirmation fields and the Force password change and Update buttons."}

Only the platform knows the previous passwords, so this line gets no check mark while you type. A password the user already had is refused when you click **Update**, with **This password has already been used recently. Please choose a different one.**, and the account keeps its password.

- **Create User** does not show the rule: a new account has no previous password.
- While the rule is on, you get 5 password changes per user in 15 minutes by default, successful ones included, then **Too many password change attempts. Please try again in a few minutes.** Your attempts never block another administrator, nor the user, on the same account.
- Through the API, send the password alone in its `fieldPatch`: while the rule is on, a password sent with other fields is refused with `The password must be changed on its own`.

:::example[Reset the password of a compromised account]
Grand Meridian Hotels keeps the last 5 passwords: **Number of recent passwords that cannot be reused (0 equals disabled)** is `5`. Leo Martin reports that he typed his password on a fake booking page. You open his user page, click **Update**, then the **Password** tab, and the box reads **Must be different from the last 5 passwords**. The temporary password you gave Leo when he joined is refused: he used it before. You enter a new one in **Password** and **Confirmation** and click **Update**, then click **Force password change** so that Leo picks his own at his next request.
:::
::::

## Disable a user

Disable an account rather than delete it: the user can no longer log in, and everything they created, shared or own stays in place.

Open the user, click **Update**, and set **Account Status** to **Inactive** or **Locked**. OpenCTI ends the open sessions of the user at once. At the next login, the user sees the message of their status. An account also stops working when its **Account Expire Date** passes: OpenCTI refuses the login and sets the status to **Expired**.

To change many accounts at once, select them in the list, click **Update** in the toolbar, choose **Replace**, then **Account status** or **Account expiration date**.

:::example[Close the access of a contractor at the end of the engagement]
Sam Reyes, of the Harbor contractors group, works for the sharing community until 30 June 2026. When you create his account, set **Account Expire Date** to 30 June 2026, 6:00 PM. From that evening, OpenCTI refuses his logins and sets his status to **Expired**, without anyone having to think of it. If the engagement ends early, set **Account Status** to **Inactive** instead: his open sessions end at once, and the objects he created keep him as their creator.
:::

:::only{opencti=">=7.260811"}
The same toolbar action sets a **Force password change date** for all the selected users.
:::

### Statuses and their messages

Two keys of the platform configuration, in [Functional customization](/reference/opencti/configuration/#functional-customization), shape account statuses:

- `app:locked_account_statuses` defines the statuses that block login, besides **Expired**, with the message each one shows at login. By default, **Inactive** and **Locked**: `app:locked_account_statuses:Inactive` (`APP__LOCKED_ACCOUNT_STATUSES__INACTIVE`) and `app:locked_account_statuses:Locked` (`APP__LOCKED_ACCOUNT_STATUSES__LOCKED`).
- `app:account_statuses_default` (`APP__ACCOUNT_STATUSES_DEFAULT`), `Active` by default, is the status of an account created without one: at the first single sign-on login, by an integration, or through the API. The creation form always sends the status you choose. The value must be `Active`, `Expired` or a status of `app:locked_account_statuses`, or the platform refuses to start.

:::example[Hold new single sign-on accounts until an administrator approves them]
Grand Meridian Hotels signs its staff in with Microsoft Entra ID, but only people the CTI team lead approves may use OpenCTI. In the docker compose file, the `opencti` service gets:

```yaml
services:
  opencti:
    environment:
      APP__ACCOUNT_STATUSES_DEFAULT: Inactive
      APP__LOCKED_ACCOUNT_STATUSES__INACTIVE: Your account waits for approval. Write to cti-admin@example.com with your team and your manager's name.
      APP__LOCKED_ACCOUNT_STATUSES__LOCKED: Your account is locked for security reasons. Write to soc@example.com to unlock it.
```

After a restart, the first login of a newcomer creates their account as **Inactive** and shows them the approval message. The administrator finds the account in :menu[Settings > Security > Users], checks its groups, and sets **Account Status** to **Active**. Accounts that feeds create for themselves are **Inactive** too: activate them when you create the feed. The platform administrator always starts **Active**. Since **Inactive** now means waiting for approval, close the accounts of people who leave with **Locked**.
:::

:::note
When a platform organization is set in [Set platform security policies](/opencti/administer/policies/), a user who belongs to no organization cannot log in. Service accounts and users with **Bypass all capabilities** are not affected.
:::

## Delete a user

On the user page, open the actions menu (⋮), choose **Delete**, and confirm. The menu shows **Delete** only when your role also has **Delete knowledge**.

Deleting a user also:

- ends their open sessions;
- deletes their notifications, triggers and digests;
- deletes the investigations and dashboards where they are the only administrator;
- turns private the public live streams, TAXII collections and CSV feeds that share data with this user's rights. Give them another user to make them public again.

::::only{opencti=">=7.261109"}
## Merge two accounts of the same person

When one person ends up with two accounts, for example a local account and the one their first single sign-on login created, merge the old account, the **source**, into the one they keep, the **target**. OpenCTI moves to the target what the source created, owns and takes part in: authorship of knowledge, history, assignments and participations, authorized-member entries, triggers, feeds, streams, TAXII collections, playbooks, investigations and dashboards. It ends the source's sessions, sets it to **Expired** and marks it as merged into the target. The names, email addresses and passwords of both accounts stay as they are.

The merge has no screen: it is a GraphQL API call, which needs **Bypass all capabilities**. Run it from the [playground](/develop/apis/graphql/#use-the-playground), or with a token of an administrator.

:::steps

1. Read the IDs of both accounts in the URL of their user pages, and pass them as the `source` and `target` variables.
2. Run a dry run, the default: OpenCTI computes what it would change and changes nothing.

   ```graphql
   mutation($source: ID!, $target: ID!) {
     userMerge(sourceId: $source, targetId: $target) {
       status message
       report { merge_id total_updated
         handlers { handler changes { entity_type count } alerts { kind message blocking } } }
     }
   }
   ```

3. Read the `alerts`. A `blocking` alert stops the real run: the target lacks the right to manage the authorized members it would receive, or a public feed, TAXII collection or live stream would share more data under the target's rights.
4. Choose what happens to the source's groups and organizations with `rightsStrategy`: `STRICT`, the default, removes them from the source without giving them to the target; `UNION` also gives them to the target. Roles and capabilities are never copied.
5. Run the merge, with `options: {dryRun: false, rightsStrategy: STRICT}`, plus `acknowledgeExposureChange: true` once you have checked a blocking alert. OpenCTI computes the plan again and refuses with `Platform state changed between the dry pass and the real pass, nothing was written` if the platform changed in between.

:::

Every run is written to a journal, kept 30 days: `userMergeJournal(mergeId: $mergeId)`, with the `merge_id` of the report, gives each step, its status and the number of objects it updated, even if your connection was cut. With activity logging, the activity log also records `merges user <source email> into user <target email>`.

The source account cannot be deleted from the user page afterwards. When `userMergeSourceDeletionReadiness(sourceId: $source, targetId: $target)` returns `allowed: true`, delete it with `userMergeDeleteSource(sourceId: $source, targetId: $target)`. Otherwise its `blockers` say what remains. This cannot be undone.

:::example[Merge Leo Martin's local account into his single sign-on account]
Grand Meridian Hotels moved to Microsoft Entra ID. Leo Martin's first single sign-on login created a second account, `l.martin@example.com`, while his reports and his triggers stay on his local account, `leo.martin@example.com`. The administrator runs the dry run with the local account as source, reads 214 objects to move and no blocking alert, then runs it for real, keeping the default `STRICT`: Leo's single sign-on account already gets its groups from Entra ID.

```graphql
mutation {
  userMerge(
    sourceId: "3f9c2a71-5d0e-4b8a-9c61-2e7f0a4d8b15"
    targetId: "a81e6c04-7b2f-4f39-8d5a-6c0b9e3f1a72"
    options: {dryRun: false, rightsStrategy: STRICT}
  ) { status message report { merge_id total_updated } }
}
```

The answer reads `SUCCESS`. Leo's reports now name his single sign-on account as their creator, his triggers notify that account, and his local account shows **Expired**.
:::
::::

## Read the user page

The user page has three tabs: **Overview**, **Analytics** and **History**. Use them to check an account before you change it, or to review what someone did, for example before an account is closed.

### Overview

- **Basic information**. For a user: the **Email address**, the **2FA state**, the **API Tokens**, **Firstname**, **Lastname**, **Account status** and **Account expiration date**. For a service account: **Account type**, **Account status**, **Account expiration date**, **API Tokens**, and who created it and when.
  To reset the two-factor authentication of a user who lost their device, click the delete icon next to **2FA state**. The user can then set it up again.
- **Permissions**: the **Roles**, **Groups** and **Organizations** the user gets, their open **Sessions**, their **Hidden entity types** and their effective **Max Confidence Level**. A tooltip next to the level says where it comes from: a group, the user, or **Bypass all capabilities**. Per-type levels follow, each marked **from group** with the group's name, or **from user**.
- **Triggers and Digests**: the notifications sent to the user. Service accounts have none.
- **Operations**: the **Number of operations** of the user per month. Requires Enterprise Edition.
- **History**: the latest changes the user made, with links to every entity and relationship they created. It requires the **Access security activity** capability.

:::only{opencti=">=7.260811"}
For a user, **Basic information** also shows **Password valid until**, the date the password must be changed.
:::

### Analytics

**Analytics** charts the activity of the user, from the activity log. It requires Enterprise Edition.

| Chart | What it tells you |
| --- | --- |
| **Login to the platform** | When the user logged in |
| **Knowledge generation** | How many objects they created, updated and deleted |
| **Top global search keywords** | What they searched for |
| **Top events** | The actions they took most |
| **Top authors of read and exported entities**, **Top read or exported entities** and **Latest exports** | What they read and took out of the platform |

Searches, reads and exports are recorded only for the users, groups and organizations that extended activity listens to. For anyone else, these charts stay empty: see [Choose what activity is recorded](/opencti/configure/activity/configuration/).

### History

**History** lists every change the user made to the knowledge. With the **Access security activity** capability, it also lists their actions from the activity log, such as logins and exports. A change made with a commit message, when an entity type requires an external reference, shows an icon: click it, **View commit message**, to read the **Commit message**.

## API tokens

A token lets a person or a script call the API with the rights of the user. The user's role needs **Allow token usage**, without which every token of the user is refused.

Users generate their own tokens on their profile. As an administrator, you see, generate and revoke the tokens of any user on their page. The date a token was last used is updated at most once a minute, to spare Redis: `app:token_activity_period` (`APP__TOKEN_ACTIVITY_PERIOD`), in seconds, 60 by default.

:::steps

1. On the user page, click **Generate Token** next to **API Tokens**. The **Generate a new token** panel opens.
2. Enter a **Name** that says what the token is for.
3. Choose a **Duration**: **30 days**, **60 days**, **90 days**, **1 year** or **Unlimited**, the default. Prefer a limited duration: a leaked token stops working on its own.
4. Click **Generate**. The **Token details** panel shows the token.
5. Copy the token and give it to its owner. OpenCTI keeps only a fingerprint: once you close the panel, nobody can read the token again.

:::

The **API Tokens** list shows, for each token, when it was **Last Used**, or **Never used**, and when it **Expires At**. A token that expires within seven days shows **Expires soon**; after that date it shows **Expired** and is refused. To stop a token at once, click the **Revoke** icon on its line, then confirm with **Revoke** in the **Revoke API Token** dialog. Before you revoke a token, check that no connector or script still uses it: their next call is refused.

For a connector, a feed or a shared script, create a [service account](/opencti/administer/service-accounts/) rather than a token on a person's account: the token then survives the person leaving, and the data it writes is traced to the integration.

:::example[Let an analyst script the weekly hospitality report]
Ana Duval runs a script that pulls the reports on the hospitality sector every Monday. On her profile, she generates a token:

- **Name**: `Weekly hospitality report script`, so that the token is easy to find and revoke.
- **Duration**: **90 days**, to limit the damage if the token leaks.

The script calls the API with this token and sees exactly what Ana sees. On Ana's user page, you see the token, when the script last used it and when it expires. When Ana leaves the team, revoke it there.
:::

### Send an email

With an Enterprise Edition license, click **Send email** on the user page, choose an **Email template** and click **Send**, for example to resend the welcome email. See [Customize the emails OpenCTI sends](/opencti/configure/email-templates/).

## End sessions

A session starts at each login. :menu[Settings > Security > Sessions] lists the open sessions, grouped by user. Each one shows when it started, and how many minutes it has left out of its full length, as in `18 minutes left / 20`.

::screenshot{id=opencti-users-sessions alt="The Sessions screen lists the open sessions under the name of each user, each with the time it started and the minutes it has left out of 20."}

- To end one session, click its delete icon in :menu[Settings > Security > Sessions] and confirm.
- To end all the sessions of a user, open the user page and click the delete icon next to **Sessions**, with the tooltip **Kill all sessions**.

Disabling or deleting a user ends their sessions on its own. End them by hand when you suspect a session was stolen, or to make a user log in again.

How long a session lasts is set by `app:session_timeout`, 20 minutes without activity by default. See [Cookies](/opencti/administer/cookies/). To limit how many sessions a user can open at the same time, set **Max concurrent sessions** in [Manage login providers in the UI](/opencti/administer/authentications/#set-platform-wide-login-policies): beyond it, a new login ends the oldest sessions of the user.

## Who sees which users

Lists of users across OpenCTI, such as assignees, participants and filters, can be limited to the people of your own organizations.

:::only{opencti=">=7.260811"}
A user sees all other users when their role has **Manage credentials**, **Manage Playbooks** or **Manage customization**, or when no platform organization is set and **Allow users to view users of other organizations** is on in [Set platform security policies](/opencti/administer/policies/). With a platform organization set, that option has no effect.
:::

:::only{opencti="<7.260811"}
When picking users, for example in an assignee or participant field, a user sees all other users when their role has **Manage credentials**, **Manage Playbooks** or **Manage customization**, when no platform organization is set, or when **Allow users to view users of other organizations** is on in [Set platform security policies](/opencti/administer/policies/).

The users already shown on an object, such as its creators, assignees and participants, follow a stricter rule: a user sees all of them only with one of these capabilities, or when no platform organization is set and **Allow users to view users of other organizations** is on.
:::

Otherwise, a user sees only the users in no organization, the users of their own organizations, and service accounts. Only direct membership counts: a user who belongs to an organization only through the **Organization propagation via participation** rule does not see its other members, and they do not see that user.

## Check that it works

Log in as the user in a private window, or read the user page: **Roles**, **Groups** and **Max Confidence Level** show what the user can do. To check what they see, compare their **Organizations** with the sharing of the data, and their groups' allowed markings with the markings of the data.

## What's next

- [Organize users in groups](/opencti/administer/groups/), to give users their rights.
- [Create accounts for integrations](/opencti/administer/service-accounts/), for connectors and scripts.
- [Configure single sign-on](/opencti/administer/sso/), to create users from your identity provider.
