---
title: Set the login message and the consent box
description: Show a message on the OpenAEV login page and make users accept your terms before they log in, with the three login messages of the Policies screen.
doc_type: how-to
products:
  - openaev
personas: [admin]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/administration/policies/
---

The **Policies** screen holds the three login messages of OpenAEV: a message shown above the login form, and a consent text that keeps the form hidden until the user checks a box. Use them to announce maintenance, or to make every user accept your acceptable use terms before they log in. The login page comes before any tenant, so these messages are the same for every tenant of the platform.

## Before you start

- Your role has **Manage platform settings**: the messages belong to the whole platform. The bootstrap admin has it.
- The screen is in the **This tenant** scope of the Security menu, shown to users with **Access tenant settings**. A user who has the tenant settings capabilities but not **Manage platform settings** sees the fields, but OpenAEV refuses the update.

## Write the messages

::diagram{id=openaev-policies-login-page}

:::steps
1. Open :menu[Settings > Security > Policies]. If the right menu shows the **Platform** scope, switch to **This tenant** first: **Policies** is listed there.
2. In the **Login messages** card, fill in the fields you need. Each one takes Markdown.

   | Field | What it does | Empty |
   | --- | --- | --- |
   | **Platform login message** | Shows above the login form, for an announcement | Nothing is shown |
   | **Platform consent message** | Shows above the form and hides the form and the SSO buttons until the user checks the box below it | No consent is asked |
   | **Platform consent confirm text** | The text next to the box | **I have read and comply with the above statement** |

3. Click **Update**.
:::

::screenshot{id=openaev-policies-login-messages alt="The Login messages card of Policies, filled with the maintenance notice, the rules of the Example CERT validation platform and the text next to the consent box, above the Update button."}

The consent is asked again at every visit of the login page: OpenAEV keeps no record of who accepted it.

:::example[Ask the hotel chains to accept the platform rules]
Example CERT hosts the simulations of several hotel chains on one OpenAEV instance, and every user must accept its rules before logging in.

| Field | Value |
| --- | --- |
| **Platform login message** | `Example CERT validation platform. Maintenance on Sunday 18 January 2026, 06:00 to 08:00 CET.` |
| **Platform consent message** | `This platform runs attack simulations against the systems of Example CERT's members. Launch a simulation only on the assets your organization owns, and report any unexpected effect to Example CERT.` |
| **Platform consent confirm text** | `I accept the rules of the Example CERT validation platform` |

The login page shows the rules with a box to check, then the maintenance notice. The form, and the **Log in with Grand Meridian** button of the chain's single sign-on, show only once the box is checked.
:::

## Check that it works

Open the login page in a private browser window. The login message shows above the form; with a consent message, the form stays hidden until you check the box.

## What's next

- [Configure single sign-on](/openaev/administer/sso/), for the buttons shown under the form.
- [Set the name, look and home page of a tenant](/openaev/administer/parameters/).
- [How access works in OpenAEV](/openaev/concepts/access/).
