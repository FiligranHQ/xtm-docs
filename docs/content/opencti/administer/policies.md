---
title: Set platform security policies
description: Set the platform-wide security policies of OpenCTI, such as the platform main organization, user visibility, the classification banner and the messages shown at login.
sidebar_position: 25
doc_type: how-to
products:
  - opencti
personas: [admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/policies/
---

The **Policies** screen holds the security settings that apply to the whole platform: the platform main organization, the group of service accounts, user visibility, a classification banner, and the login and error messages. This page shows how to set each of them. Login providers, password rules and two-factor authentication are elsewhere, in [Manage login providers in the UI](/opencti/administer/authentications/).

## Before you start

- Your role has the **Manage credentials** capability, to open the screen and change the platform main organization, the user policy and user visibility.
- To change the banner and the messages, your role also needs **Manage parameters**.
- To set or change the platform main organization, one of your roles also needs the setting **Allow modification of sensitive configuration** turned on, at the top of its **Capabilities** tab. See [Protect sensitive configuration](/opencti/administer/sensitive-configuration/).

Open :menu[Settings > Security > Policies]. Every field is saved as soon as you change it or leave it; the platform main organization asks for a confirmation.

::screenshot{id=opencti-policies-overview alt="The Policies screen with the Platform main organization, User policy, Users visibility and Platform Banner Configuration cards, and the Login messages card below them."}

## Set the platform main organization

The **Platform main organization** card turns organization segregation on. It needs an Enterprise Edition license. Users of the main organization see all the data their markings allow; users of other organizations only see what is shared with their organization.

:::warning
This choice affects the whole platform. Once it is set, a user who belongs to no organization cannot log in, unless they have **Bypass all capabilities** or are a service account. Put every user in an organization before you set it. Knowledge shared with no organization becomes visible to members of the main organization only.
:::

Select the organization in **Platform organization**, read the warning and click **Validate**. Service accounts automatically count as members of the main organization, without being listed among its users. Read [Separate data between organizations](/opencti/administer/organizations/) before you start.

## Choose the group of service accounts

In **User policy**, **Default group for Service accounts** is the group given to the service accounts that OpenCTI creates on the fly, for example for ingestion feeds or for connectors deployed by XTM Composer. It is the Connectors group out of the box. The new accounts get exactly the rights of this group: choose one whose role can create knowledge and whose allowed markings cover what your feeds bring, or what they ingest is refused. Accounts already created keep their group. See [Create accounts for integrations](/opencti/administer/service-accounts/#accounts-created-for-you).

## Choose who sees other users

In **Users visibility**, **Allow users to view users of other organizations** lets everyone see every user in assignee, participant and author lists. It is off by default, and turned off automatically when a platform main organization is set. See [who sees which users](/opencti/administer/users/#who-sees-which-users) for the full rule.

## Show a classification banner

In **Platform Banner Configuration**, choose a **Platform banner level** (**GREEN**, **RED** or **YELLOW**) and enter the **Platform banner text**. The banner shows at the top and bottom of every screen, for example to state the classification of the platform or its purpose. It shows only when both fields are set: clear the level or the text to hide it.

## Write the login messages

Three Markdown fields in the **Login messages** card customize the login page:

- **Platform login message** shows above the login form, for example to announce maintenance.
- **Platform consent message** hides the login form until the user checks a box to agree, for example to acceptable use terms.
- **Platform consent confirm text** is the text next to that box.

:::example[State the classification and the acceptable use terms]
The sharing platform of Example CERT holds data up to TLP:GREEN, and its members must accept the acceptable use policy before they log in.

- **Platform banner level**: **GREEN**. **Platform banner text**: `TLP:GREEN - Example CERT hospitality sharing platform`.
- **Platform login message**: `Authorized users only. Activity is logged.`
- **Platform consent message**: `This platform holds threat intelligence shared by the hospitality sector. Do not share it outside your organization.`
- **Platform consent confirm text**: `I accept the acceptable use policy`.

The login page shows the message and keeps the form hidden until the box is checked. Every screen then shows a green banner at the top and bottom, so a screenshot shared by mistake still carries its classification.
:::

:::only{opencti=">=7.260811"}
## Write the no access message

**No access to knowledge error message**, in the **Platform error messages** card, replaces the home page of users who cannot access knowledge, for example to tell them whom to ask for access. It takes Markdown.
:::

## Check that it works

Open the login page in a private browser window: the login message shows above the form and, if you set a consent message, the form stays hidden until you check the box. Once you log in, the banner shows on every screen.

## What's next

- [Manage login providers in the UI](/opencti/administer/authentications/), for the password policy, two-factor authentication and the IP allow list.
- [Separate data between organizations](/opencti/administer/organizations/), before you set a platform main organization.
- [Cookies](/opencti/administer/cookies/), for session duration and cookie security.
