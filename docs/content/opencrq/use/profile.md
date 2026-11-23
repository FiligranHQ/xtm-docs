---
title: Set your profile and follow your notifications
description: Read the name XTM One gives you, set how OpenCRQ names you in each tenant, its language, theme, colours and home dashboard, and follow the notifications that tell you where you were added.
sidebar_position: 7
doc_type: how-to
products:
  - opencrq
personas: [risk-manager, admin, executive]
owner: docs-opencrq
last_reviewed: 2026-11-20
---

Your profile holds what you decide for yourself in OpenCRQ: the name and job title you go by in a tenant, the language, theme and colours of the interface, and the dashboard Home opens on. Your global name, title and email come from your XTM One account, which you sign in with. Nothing you set here changes what other users see, except your own name and title. Your notifications tell you when someone adds you to a tenant or to the platform administration.

## Before you start

Every user has a profile and notifications: no capability is needed.

## Open your profile

In the top bar, click **Account**, then **Profile**. The menu also shows the email address you are signed in with, and **Sign out**.

::screenshot{id=opencrq-profile alt="The profile page of Claire Morel, Cyber risk manager, with the Global profile card marked Managed in XTM One and its Edit in XTM One link, the Tenant profile card whose full name and title are marked Inherited, and the Preferences card with Language, Theme, the Softer tones switch of Pastel colours, and Home dashboard."}

The page holds four cards:

| Card | What it holds |
| --- | --- |
| **Global profile** | Your **Full name**, **Title** and **Email**, read from your XTM One account and the same in every tenant. Marked **Managed in XTM One**: you cannot change them in OpenCRQ |
| **Tenant profile** | The **Full name** and **Title** you go by in this tenant only |
| **Preferences** | **Language**, **Theme**, **Pastel colours** and **Home dashboard** |
| **XTM One MCP server** | Shown only when the platform is connected to XTM One |

## Set your name and title

Your name is what other members see where OpenCRQ names a person: as the owner of a risk scenario, a treatment plan or a budget review, for example. Without a name, OpenCRQ shows your email address instead.

Your global name and title belong to your XTM One account: OpenCRQ reads your **Full name**, **Title** and **Email** from XTM One each time you sign in. To change them, click **Edit in XTM One** in the **Global profile** card, change them in your XTM One profile, and sign in to OpenCRQ again.

To go by another name or title in one tenant only:

:::steps
1. Click **Edit profile**.
2. In **Tenant profile**, enter the **Full name** and the **Title**, your job title, for this tenant. The empty fields show your global values in grey.
3. Click **Save**. **Cancel** asks before it discards unsaved changes, and so does leaving the page.
:::

In the **Tenant profile** card, each value says where it comes from: **Inherited** when it is your global value, **Overridden** when you set one for this tenant. **Reset to global** removes both overrides.

The super administrator, the XTM One administrator who reaches every tenant without being a member of any, has no **Edit profile**: their **Tenant profile** always shows the global values.

## Choose your language, theme, colours and home dashboard

The **Preferences** card changes the interface for you only, and each field saves as soon as you change it.

| Field | Choices | Effect |
| --- | --- | --- |
| **Language** | **Automatic**, **English**, **French** | The language of the interface. **Automatic** follows the tenant's **Default language**, set by an administrator, then your browser's language when it is English or French, and English otherwise. A dialog asks to **Reload** the application to apply it |
| **Theme** | **Light**, **Dark**, **System** | **System** follows the theme of your operating system |
| **Pastel colours** | The **Softer tones** switch, off by default | On, severity, status and marking colours take softer tones: the chips of a finding's score history, for example, show a soft plain fill instead of an outline. Off, OpenCRQ draws the standard colours, the same in every Filigran product. The switch follows you in every tenant |
| **Home dashboard** | **Tenant home**, or one of the five dashboards | The dashboard **Home** opens on. **Tenant home**, with the tenant's choice in parentheses, follows the dashboard an administrator chose for the tenant |

The language changes the interface, not your data: a scenario named in English stays in English. [Track cyber risk in money with dashboards](/opencrq/use/decide/dashboards/) describes the five dashboards and how an administrator sets the tenant's home.

:::xtm-one[Connect your AI client through XTM One]
When an administrator has connected OpenCRQ to XTM One, the **XTM One MCP server** card shows the address of the Model Context Protocol (MCP) server XTM One exposes for OpenCRQ, with a button to copy it. AI clients such as Cursor or Claude Desktop connect to it with a personal XTM One API key, and work on OpenCRQ's frameworks, controls, assessments and risk scenarios with your own permissions. **Manage in XTM One** opens your XTM One profile, where you create the key and copy a ready-made client configuration. The MCP server is part of the Community Edition.
:::

## Follow your notifications

The bell in the top bar, **Notifications**, shows how many notifications you have not read. Click it to see the latest ones, **Mark all as read**, or **View all notifications** to open the **Notifications** page.

OpenCRQ notifies you when an administrator adds you:

| Notification, for example | What it means |
| --- | --- |
| "Claire Morel added you to Grand Meridian Hotels", then "You can now open Grand Meridian Hotels." | You are a member of that tenant, in the groups the administrator chose. In the bell's panel, click the notification to open the tenant |
| "Claire Morel added you to the platform administration (Admin)", then "You can now open the platform administration." | You are in those platform groups, and reach the platform administration in :menu[Settings > Security], under **Platform** |

There is nothing to accept: the access is yours as soon as the administrator adds you. XTM One usually tells you by email as well. How administrators add people is in [Give people access with users, groups and roles](/opencrq/administer/users-and-roles/).

On the **Notifications** page, the **All** and **Unread** tabs filter the list. Click a notification to mark it as read. Select notifications to **Mark as read**, **Mark as unread** or **Delete** them; deleted notifications cannot be brought back. **Load more** shows older ones.

:::example[Set up the profile of Grand Meridian's risk manager]
Claire Morel is the cyber risk manager of Grand Meridian Hotels. She signs in to OpenCRQ through XTM One with docs-admin@example.com, and owns the scenarios, the plan and the budget review of the chain.

- In her XTM One profile, her name is Claire Morel and her title Cyber risk manager. After she signs in, the **Global profile** card shows **Full name** Claire Morel and **Title** Cyber risk manager.
- **Tenant profile**: she sets nothing, so both values stay **Inherited**.
- **Preferences**: she keeps **Theme** on **System**, turns **Softer tones** on under **Pastel colours** for the long review sessions, and sets **Home dashboard** to **Risk treatment**.

Home now opens on each scenario's likelihood, annual loss exposure and tolerance, while the rest of the tenant keeps the **CISO** dashboard. Her name shows as the owner of the treatment plan MFA for the front-desk accounts and of the budget review Security budget 2026.
:::

## What's next

- [Track cyber risk in money with dashboards](/opencrq/use/decide/dashboards/), to read the dashboard you chose.
- [Give people access with users, groups and roles](/opencrq/administer/users-and-roles/), for administrators who add people.
