---
title: Set up your profile
description: Set your OpenCTI profile the way you work - language, theme, date format, notifications - secure it with two-factor authentication, and manage your API tokens.
sidebar_position: 95
doc_type: how-to
products:
  - opencti
personas: [cti-analyst, soc-analyst, admin]
owner: docs-opencti
last_reviewed: 2026-11-01
---

Your profile holds what you decide for yourself in OpenCTI: how the platform speaks and shows dates, how you hear about the work assigned to you, how your account is protected, and the API tokens your scripts and tools use to work as you. Nothing you set here changes what other users see. This page walks through each card of the profile, from your identity to the XTM One MCP server, and links to the pages that explain more.

## Before you start

- Every user can open their profile and change their identity, preferences, password and two-factor authentication: no capability is needed.
- API tokens and the **Playground** button need the **Allow token usage** capability in one of your roles. Without it, the **API access** card says **You do not have the right to use API tokens.**
- The **Dashboard settings** card shows only with the **Access knowledge** capability.
- The **XTM One MCP server** card shows only when an administrator has connected the platform to XTM One.

## Open your profile

In the top bar, click the **Profile** button, then **Profile** in its menu. The page shows one card per topic. Fields save as soon as you change or leave them: only the password has an **Update** button.

## Check your identity

The **Profile** card holds who you are on the platform:

- **Name**: how other users see you, for example in the assignees and participants of a report, or in the history of an object.
- **Email address**: where the **Default mailer** notifier sends your alerts. It is also the login of a local account.
- **Organizations**: the organizations you belong to. Only an administrator changes them, in your user account.
- **Firstname**, **Lastname** and **Description**: free information about you.

If you log in through single sign-on, the heading of the card says **external**. Your name and email address come from your identity provider: you cannot change them here, and the profile has no password to change.

## Set how the platform looks and reads

The **User experience** card changes the interface for you only.

::screenshot{id=opencti-profile-user-experience alt="The User experience card of the profile, with the Theme, Language and Unit system fields, the date and time formats of tables, the two navigation switches and the Personal notifiers field."}

| Field | Choices | Effect |
| --- | --- | --- |
| **Theme** | **Default**, or a theme of the platform | **Default** follows the default theme an administrator set. The other entries are the themes of the platform, listed in the [platform parameters](/opencti/configure/parameters/#brand-the-interface-with-themes) |
| **Language** | **Automatic**, or one of nine languages: Deutsch, English, Español, Français, Italiano, 日本語, 한국어, 简化字, Русский | **Automatic** follows the language of the platform, or the language of your browser when the platform is set to **Automatic** too. A browser language OpenCTI does not offer falls back to English |
| **Unit system** | **Automatic**, **Imperial**, **Metric** | How heights and weights of individual threat actors are shown and entered. **Automatic** uses **Imperial** in English and **Metric** in the other languages |
| **Show left navigation submenu icons** | On or off | Shows an icon next to each entry of the submenus of the left navigation |
| **Auto collapse submenus in left navigation** | On or off | Keeps only one submenu of the left navigation open at a time |

The language changes the interface, not the knowledge: a report written in English stays in English.

:::only{opencti=">=7.261030"}
Two more fields set how dates appear in lists and tables:

- **Date format in tables**: **Platform default** keeps the format an administrator chose for the platform. **ISO 8601 (YYYY-MM-DD)** writes every date as `2026-01-14`, whatever your language.
- **Time in tables**: **Platform default**, **Date only**, **Hours and minutes (HH:mm)** or **Hours, minutes and seconds (HH:mm:ss)**. A time you choose shows on 24 hours, whatever your language.

With **ISO 8601 (YYYY-MM-DD)** and a time, the date carries the offset of your browser's time zone, such as `2026-01-14 09:30+01:00`. When you and the platform both keep the default, each column decides whether it shows the time.
:::

:::only{opencti=">=7.261127"}
### Choose the key that opens the search

**Search shortcut**, at the bottom of the **User experience** card, is the key that puts the cursor in the search bar of the top bar, from any page. It is `/` by default, and the search bar then says **Type / to search**. [Search for knowledge](/opencti/use/explore/search/) explains what the search finds.

To change it, click the **Search shortcut** field and press the keys you want: the field records them, and the new shortcut works at once.

- A symbol, such as `/` or `?`, works alone.
- A letter, a digit, Space or an arrow needs Ctrl, or Command on macOS, such as Ctrl+K.
- Alt, Option on macOS, is never accepted.

A combination that cannot be used shows **This key combination cannot be used. Try a symbol such as /, or add Ctrl / Command.** under the field. **Reset** brings back `/`, and **Disable** turns the shortcut off: the field then shows **Disabled**.

The shortcut does nothing while a dialog, a panel or a menu is open. A key without Ctrl or Command does nothing while you type in a field, so `/` stays a character in your text. Unlike the other fields of the profile, the shortcut is kept in this browser only: another browser starts with `/`, and so does this one after **Clear local storage**.

::screenshot{id=opencti-profile-search-shortcut alt="The Search shortcut field at the bottom of the User experience card shows the default key /, with the Disable button and the hint Press the keys you want to use to focus the search bar."}
:::

The **Dashboard settings** card chooses the dashboard your home page opens on and the dates its counters use. [Choose your home dashboard](/opencti/use/explore/insights/#choose-your-home-dashboard) explains both fields.

## Choose how you are notified

OpenCTI alerts you when you become an assignee or a participant of an object, such as a report or a case. **Personal notifiers**, in the **User experience** card, are the channels these alerts use: **User interface** puts them in your notification list, **Default mailer** sends them to your email address, and an administrator may have added others.

:::only{opencti=">=7.260811"}
An administrator can turn these alerts off for the whole platform. The card then says **Automatic notifications for assignees and participants have been disabled by your platform administrator**, and the field is hidden.
:::

To be alerted about anything else, such as every new report on the hospitality sector, create a trigger or a digest. They are not on the profile:

:::only{opencti=">=7.261030"}
click the bell icon in the top bar to open the **Notification Center**, then its **Triggers** tab.
:::

:::only{opencti="<7.261030"}
click the **Triggers** icon in the top bar.
:::

[Get alerted when knowledge changes](/opencti/use/automate/notifications/) shows how to create live triggers, subscribe to an entity and group alerts in a digest.

:::only{opencti=">=7.260811 <7.261030"}
When the platform is connected to the XTM Hub, the **XTM Hub News Feed** card of the profile chooses which news of the XTM Hub library you receive. See [Follow what XTM Hub publishes](/opencti/configure/xtm-hub/news-feed/).
:::

:::only{opencti=">=7.261030"}
The news of the XTM Hub library has its own page: see [Follow what XTM Hub publishes](/opencti/configure/xtm-hub/news-feed/).
:::

## Change your password

:::steps
1. In the **Authentication** card, enter your **Current password**.
2. Enter the **New password**. The rules of the platform's password policy show above the field.
3. Enter it again in **Confirmation**, then click **Update**. The message **The password has been updated** confirms the change.
:::

:::only{opencti=">=7.261117"}
Your administrator may keep you from reusing your recent passwords. The rules above the field then include **Must be different from your current password**, or **Must be different from your last {count} passwords** with the number they chose. A password you used recently is refused with **This password has already been used recently. Please choose a different one.** under **New password**: the new password and its confirmation are cleared, your current password stays, and you type another one.

While this rule is on, you have a few attempts in a short time, the successful ones included: after that, OpenCTI says **Too many password change attempts. Please try again in a few minutes.** Wait, then try again. [Stop users from reusing a password](/opencti/administer/authentications/#stop-users-from-reusing-a-password) explains the rule and its limits.
:::

Accounts that log in through single sign-on have no password section: change your password with your identity provider.

## Turn on two-factor authentication

Two-factor authentication asks for a code from an application on your phone after your password, so a stolen password is not enough to log in as you.

:::steps
1. In the **Authentication** card, click **Enable two-factor authentication**. A dialog shows a QR code.
2. Scan the QR code with an authenticator application on your phone, such as the one your organization recommends.
3. Type the code the application shows. When it is right, the dialog closes and the button becomes **Disable two-factor authentication**. If the dialog says **The code is not correct.**, type the next code the application shows.

   ::screenshot{id=opencti-profile-enable-2fa alt="The Enable two-factor authentication dialog with the QR code to scan, the message Type the code generated in your application, and the boxes for the code."}
:::

From then on, each login asks for a code from the application. Calls to the API with a token are not asked for a code.

When an administrator turns on **Enforce two-factor authentication**, users who have not set it up do it at their next login, and **Disable two-factor authentication** is greyed out. See [Set platform-wide login policies](/opencti/administer/authentications/#set-platform-wide-login-policies).

If you lose your phone, ask an administrator to reset your two-factor authentication from your user account, as [Read the user page](/opencti/administer/users/#overview) describes. You then set it up again.

## Create and manage API tokens

An API token lets a script or a tool call the OpenCTI API as you: it reads what you can read and changes what you can change. Give each script its own token, so that you can stop one without touching the others.

:::steps
1. In the **API access** card, click **Generate Token**. The **Generate a new token** panel opens.
2. Enter a **Name** that says what the token is for, such as the script that uses it.
3. Choose a **Duration**: **30 days**, **60 days**, **90 days**, **1 year** or **Unlimited**, the default. Prefer a limited duration: a token that leaks stops working on its own.
4. Click **Generate**. The **Token details** panel shows the token.
5. Copy the token and store it where your script reads it, such as a password manager or a secret store. OpenCTI keeps only a fingerprint of it: once you close the panel, nobody can read it again.

   ::screenshot{id=opencti-profile-token-generate alt="The Generate a new token panel with the Name field, the Duration list and the Generate button."}
:::

A token starts with `flgrn_octi_tkn_`. The list shows only its last four characters, to tell your tokens apart.

::screenshot{id=opencti-profile-api-access alt="The API access card with the OpenCTI version, the Playground and Generate Token buttons, and a token list giving the name, the last four characters, the last use and the expiry date of each token."}

The list of the **API access** card follows the life of each token:

- **Last Used** gives the last time a script used the token, or **Never used**. The date is updated about once a minute, not on every call.
- **Expires At** gives its expiry date, or **Unlimited**. Seven days before that date, it shows **Expires soon**; after it, **Expired**, and OpenCTI refuses the token.
- **Revoke**, the icon at the end of the line, stops the token at once. Confirm with **Revoke** in the **Revoke API Token** dialog. A script that still uses it is refused at its next call.

To renew a token before it expires, generate a new one, put it in the script, check that **Last Used** moves on the new token, then revoke the old one. Your tokens also stop working when your account is disabled, or when your roles lose **Allow token usage**. With an Enterprise Edition license, each token generated or revoked is recorded in the [activity log](/opencti/configure/activity/).

Administrators see, generate and revoke the tokens of every user from their user account: see [API tokens](/opencti/administer/users/#api-tokens). A token for a connector, a feed or a script that the whole team relies on belongs to a [service account](/opencti/administer/service-accounts/), not to a person: it keeps working when that person leaves.

:::only{opencti="<7.260811"}
When the platform is registered with XTM One, the list also holds a token named `XTM One`, which OpenCTI created for you with no expiry. Revoking it cuts XTM One's access to OpenCTI as you. See [XTM One in OpenCTI](/xtm-one/use/in-opencti/).
:::

## Explore the API in the playground

**Playground**, in the **API access** card, opens the GraphQL playground of your platform in a new tab, to try queries on the API. The button shows with the **Allow token usage** capability, when an administrator has kept the playground on (see [GraphQL playground](/deploy/operate/harden/#graphql-playground)).

By default, the platform refuses the queries that list the whole schema, so the playground cannot show the documentation of the API. The [GraphQL schema reference](/reference/opencti/graphql-schema/) documents every query and type instead, and [Integrate and extend](/develop/) shows how to script the API.

:::::only{opencti=">=7.260811"}
::::xtm-one[Work with OpenCTI from your AI client]
When the platform is connected to XTM One, the **XTM One MCP server** card gives the address an AI client, such as Cursor or Claude Desktop, uses to search, read and create knowledge in OpenCTI with your own rights. Copy the **MCP endpoint URL** with **Copy MCP endpoint URL**: it is the address of XTM One followed by `/mcp/opencti`.

The client authenticates with a personal API key of XTM One, not with an OpenCTI token. **Manage in XTM One** opens your XTM One profile, with the connection status and a client configuration to copy. [Use OpenCTI from your own AI client](/xtm-one/use/in-opencti/#use-opencti-from-your-own-ai-client) lists what the client can do and gives a complete configuration.

:::only{opencti=">=7.261030"}
::screenshot{id=opencti-xtm-one-mcp-card alt="The XTM One MCP server card of the profile, with the MCP endpoint URL ending in /mcp/opencti, its copy button and Manage in XTM One."}
:::
::::
:::::

## Clear your local settings

Your browser keeps the last filters, searches and sorts you used in most views. When a list fails to load because of one of them, clear them:

:::steps
1. In the **Local settings** card, click **Clear local storage**.
2. Confirm with **Validate**.
:::

Every view opens again with its default filters, in this browser only. The preferences of your profile are kept: they are stored on the platform, not in the browser.

:::only{opencti=">=7.261127"}
The **Search shortcut** is the exception: it is kept in the browser, so it goes back to `/`.
:::

::::example[Set up Ana Duval's account at Example CERT]
Ana Duval joins the CTI team of Example CERT. She compares the timeline of the Velvet Harbor campaign with partner CERTs in other countries, and a script of hers pulls the new reports on the hospitality sector every Monday. On her profile, she sets:

- **Language**: **Français**, her working language.
- **Date format in tables**: **ISO 8601 (YYYY-MM-DD)**, and **Time in tables**: **Hours and minutes (HH:mm)**, so that a date reads `2026-01-14 09:30+01:00` in every list, the way partners write it.
- **Personal notifiers**: **User interface** only, without **Default mailer**: reports assigned to her show in the platform, not in her mailbox.
- **Enable two-factor authentication**, scanned with the authenticator application of her phone.
- In **API access**, **Generate Token** with the **Name** `Weekly hospitality report script` and the **Duration** **90 days**.

:::only{opencti=">=7.261127"}
She also clicks the **Search shortcut** field and presses Ctrl+K, the shortcut of the other tools she uses: from any page, Ctrl+K puts the cursor in the search bar, ready for the next IP address a partner asks about.
:::

The interface turns to French while the reports stay as written, every list gives dates the partners read without converting, and each login asks for a code from her phone. Her script calls the API with the token and sees exactly what Ana sees. When **Expires soon** shows next to the token in April 2026, she generates the next one, updates the script and revokes the old token.
::::

## Check that it works

- Reload the page: the interface shows in the language and theme you chose, and lists show dates in your format.
- Log out and log in again: after two-factor authentication is on, the login asks for a code.
- Run your script with its new token: **Last Used** shows a date instead of **Never used**.

## What's next

- [Get alerted when knowledge changes](/opencti/use/automate/notifications/): create triggers and digests.
- [Choose your home dashboard](/opencti/use/explore/insights/#choose-your-home-dashboard): the **Dashboard settings** card.
- [XTM One in OpenCTI](/xtm-one/use/in-opencti/): what the AI does with your rights.
- [Manage users](/opencti/administer/users/): what an administrator controls on your account.
