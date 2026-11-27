---
title: Set up your profile
description: Set your name, theme, language and home dashboard in OpenAEV, the phone and PGP key injects use, change your password or email address, and renew your API token.
doc_type: how-to
products:
  - openaev
personas: [validation-engineer, exercise-planner]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/administration/profile/
---

Your profile holds what you decide for yourself in OpenAEV: your name, the theme and language of the interface, the dashboard **Home** opens on, the phone numbers and PGP key injects use when you are a target, your password and email address, and the token your scripts call the API with. Nothing you set here changes what other users see, except your name. Your notifications have their own page: see [Get notified of score drops and changes](/openaev/configure/notifications/).

## Before you start

Every user has a profile: no capability is needed. The default administrator account, whose address and password come from the platform configuration (`openaev.admin.email` and `openaev.admin.password`, see the [admin settings](/reference/openaev/configuration/#openaev-admin)), has no **Password** and **Email** sections: change them in the configuration.

## Open your profile

In the top bar, open the account menu and click **Profile**. The page holds these sections:

| Section | What it holds |
| --- | --- |
| **Profile** | **Firstname**, **Lastname**, **Organization**, **Country**, **Theme**, **Language** and **Home dashboard** |
| **Information** | **Phone number (mobile)**, **Phone number (landline)** and **PGP public key** |
| **Password** | The change of your password |
| **Email** | The change of your email address |
| **API access** | Your API token, under **Token key** |
| **XTM One MCP server** | Shown only when OpenAEV is connected to XTM One |

## Set your preferences

:::steps
1. In **Profile**, check your **Firstname** and **Lastname**: they name you wherever OpenAEV shows a person, such as the members of a team.
2. Choose the **Theme**: **Default** follows the one your administrators set, or force **Dark** or **Light**.
3. Choose the **Language**: **Automatic** follows the one your administrators set, or your browser's when they set none.
4. Choose the **Home dashboard**: the custom dashboard :menu[Home] opens on for you. **Platform default** keeps the one your administrators chose for everyone. See [Build dashboards of your validation results](/openaev/use/dashboards/).
5. Click **Update**.
:::

:::warning
In this version, the **Home dashboard** field shows **Platform default** each time the page opens, even when you chose another dashboard, and **Update** in the **Profile** section saves what the field shows. When you change your name, theme or language, choose your home dashboard again before you click **Update**.
:::

::screenshot{id=openaev-profile alt="The profile page of the signed-in user: the Profile section with the names, organization, country, theme, language and home dashboard, above the Information section."}

## Set how injects reach you

When a scenario targets a team you belong to, injects use the details of the **Information** section:

- **Phone number (mobile)** receives SMS injects. Start it with a plus sign and the country code.
- **Phone number (landline)** is kept with your details.
- **PGP public key** encrypts the email injects sent to you.

Click **Update** in that section to save them.

## Change your password

:::steps
1. In **Password**, enter your **Current password**.
2. Enter the **New password**, then again in **Confirmation**.
3. Click **Update**.
:::

Your other sessions of OpenAEV, in other browsers or on other devices, are signed out. The session you made the change in stays open.

## Change your email address

::diagram{id=openaev-profile-email-change}

:::steps
1. In **Email**, enter your **Current password** and the new **Email**.
2. Click **Update**. Your other sessions are signed out.
3. Open the email OpenAEV sends to your current address, and follow its link within 10 minutes.
:::

Until you follow the link, your address does not change. Once you follow it, the new address applies, your other sessions are signed out again, and OpenAEV sends a confirmation to the new address. An address that already belongs to another account is refused. If the link expired, start again from step 1.

## Use your API token

**API access** shows your **Token key**. A script or an integration that calls the OpenAEV API sends it in the `Authorization` header, as `Bearer` followed by the token, and acts with your rights. **API specifications** opens the description of every endpoint.

Click **RENEW** when the token may have leaked, or when someone who knew it leaves: OpenAEV deletes the token and creates a new one at once. Every script that used the old token stops working until you give it the new one.

:::example[Point a reporting script at OpenAEV]
The validation engineer of Grand Meridian Hotels sets **Language** to French, **Theme** to **Dark**, and **Home dashboard** to `Velvet Harbor replay`, then clicks **Update**. For the weekly report to Hélène Marchetti, the CISO, a script reads the simulations through the API, with the token stored in the variable `OPENAEV_TOKEN` of the script's vault:

```bash
curl -s https://openaev.example.com/api/exercises \
  -H "Authorization: Bearer $OPENAEV_TOKEN" \
  -H "Content-Type: application/json"
```

When the engineer leaves the team in March 2026, their successor clicks **RENEW** in their own profile and gives the script the new token: the script runs with the rights of the person who owns the token.
:::

:::xtm-one[Use OpenAEV from an AI client]
When OpenAEV is connected to XTM One, the **XTM One MCP server** section gives the **MCP endpoint URL** that AI clients, such as Claude Desktop or Cursor, connect to. They work with scenarios, simulations, payloads and findings with your own rights, authenticated with a personal API key of XTM One. **Manage in XTM One** opens your XTM One profile, where the key and the client configuration are.
:::

## What's next

- [Get notified of score drops and changes](/openaev/configure/notifications/): choose what OpenAEV tells you, and how.
- [Build dashboards of your validation results](/openaev/use/dashboards/): build the dashboard your **Home** opens on.
