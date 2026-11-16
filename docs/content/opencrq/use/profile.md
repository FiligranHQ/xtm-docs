---
title: Set your profile and follow your notifications
description: Set how OpenCRQ names you in each tenant, its language, theme and home dashboard, and answer the invitations that reach your notifications.
sidebar_position: 7
doc_type: how-to
products:
  - opencrq
personas: [risk-manager, admin, executive]
owner: docs-opencrq
last_reviewed: 2026-11-04
---

Your profile holds what you decide for yourself in OpenCRQ: the name and job title other members see, the language and theme of the interface, and the dashboard Home opens on. Nothing you set here changes what other users see, except your own name and title. Your notifications tell you when you are invited to join a tenant, and let you accept or decline.

## Before you start

Every user has a profile and notifications: no capability is needed.

## Open your profile

In the top bar, click **Account**, then **Profile**. The menu also shows the email address you are signed in with, and **Sign out**.

::screenshot{id=opencrq-profile alt="The profile page of Claire Morel, Cyber risk manager, with the Global Profile card, the Tenant Profile card whose name and title are marked Inherited, and the Preferences card with Language on Automatic, Theme on System and Home dashboard on Tenant home (CISO)."}

The page holds four cards:

| Card | What it holds |
| --- | --- |
| **Global Profile** | Your **Name**, **Title** and **Email**, the same in every tenant you belong to |
| **Tenant Profile** | The **Name** and **Title** you go by in this tenant only |
| **Preferences** | **Language**, **Theme** and **Home dashboard** |
| **XTM One MCP server** | Shown only when the platform is connected to XTM One |

## Set your name and title

Your name is what other members see where OpenCRQ names a person: as the owner of a risk scenario, a treatment plan or a budget review, for example. Without a name, OpenCRQ shows your email address on your profile and a dash where you own something.

:::steps
1. Click **Edit profile**.
2. In **Global Profile**, enter your **Name** and your **Title**, your job title. The **Email** cannot be changed here.
3. To go by another name or title in this tenant only, enter them in **Tenant Profile**. The empty fields show your global values in grey.
4. Click **Save**. **Cancel** asks before it discards unsaved changes, and so does leaving the page.
:::

In the **Tenant Profile** card, each value says where it comes from: **Inherited** when it is your global value, **Overridden** when you set one for this tenant. **Reset to global** removes both overrides.

## Choose your language, theme and home dashboard

The **Preferences** card changes the interface for you only, and each field saves as soon as you change it.

| Field | Choices | Effect |
| --- | --- | --- |
| **Language** | **Automatic**, **English**, **French** | The language of the interface. **Automatic** follows the tenant's **Default language**, set by an administrator, then your browser's language when it is English or French, and English otherwise. A dialog asks to **Reload** the application to apply it |
| **Theme** | **Light**, **Dark**, **System** | **System** follows the theme of your operating system |
| **Home dashboard** | **Tenant home**, or one of the five dashboards | The dashboard **Home** opens on. **Tenant home**, with the tenant's choice in parentheses, follows the dashboard an administrator chose for the tenant |

The language changes the interface, not your data: a scenario named in English stays in English. [Track cyber risk in money with dashboards](/opencrq/use/decide/dashboards/) describes the five dashboards and how an administrator sets the tenant's home.

:::xtm-one[Connect your AI client through XTM One]
When an administrator has connected OpenCRQ to XTM One, the **XTM One MCP server** card shows the address of the Model Context Protocol (MCP) server XTM One exposes for OpenCRQ, with a button to copy it. AI clients such as Cursor or Claude Desktop connect to it with a personal XTM One API key, and work on OpenCRQ's frameworks, controls, assessments and risk scenarios with your own permissions. **Manage in XTM One** opens your XTM One profile, where you create the key and copy a ready-made client configuration. The MCP server is part of the Community Edition.
:::

## Follow your notifications

The bell in the top bar, **Notifications**, shows how many notifications you have not read. Click it to see the latest ones, **Mark all as read**, or **View all notifications** to open the **Notifications** page.

In this version, OpenCRQ notifies you of one thing: an invitation to join a tenant, when an administrator invites the email address of an account you already have. Belonging to more than one tenant requires the Enterprise Edition. The notification names the tenant and shows how long it stays valid: an invitation expires 48 hours after it is sent.

:::steps
1. Open the notification, in the bell's panel or on the **Notifications** page.
2. Click **Accept** to join the tenant: OpenCRQ opens it, with the access the administrator's invitation grants. Click **Decline** to refuse it.
:::

An invitation you answered, or that the administrator cancelled or sent again, shows **Resolved**; one past its time shows **Expired**. Neither can be answered any more: ask the administrator to send it again. How administrators invite people is in [Give people access with users, groups and roles](/opencrq/administer/users-and-roles/).

On the **Notifications** page, the **All** and **Unread** tabs filter the list. Select notifications to **Mark as read**, **Mark as unread** or **Delete** them; deleted notifications cannot be brought back. **Load more** shows older ones.

:::example[Set up the profile of Grand Meridian's risk manager]
Claire Morel is the cyber risk manager of Grand Meridian Hotels. She signs in to OpenCRQ with docs-admin@example.com, and owns the scenarios, the plan and the budget review of the chain.

**Edit profile**, then:

- **Global Profile**: **Name** Claire Morel, **Title** Cyber risk manager.
- **Tenant Profile**: leave both fields empty, so they stay **Inherited**.
- **Save**.

In **Preferences**, she keeps **Theme** on **System** and sets **Home dashboard** to **Risk treatment**: Home now opens on each scenario's likelihood, annual loss exposure and tolerance, while the rest of the tenant keeps the **CISO** dashboard. Her name now shows as the owner of the treatment plan MFA for the front-desk accounts and of the budget review Security budget 2026.
:::

## What's next

- [Track cyber risk in money with dashboards](/opencrq/use/decide/dashboards/), to read the dashboard you chose.
- [Give people access with users, groups and roles](/opencrq/administer/users-and-roles/), for administrators who invite people.
