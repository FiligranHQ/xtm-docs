---
title: Create accounts for integrations
description: Create OpenCTI service accounts for connectors, feeds and scripts, give them an API token and the right groups, and convert users to service accounts.
sidebar_position: 5
doc_type: how-to
products:
  - opencti
personas: [admin, platform-engineer]
owner: docs-opencti
last_reviewed: 2026-11-01
---

A service account is a user for a machine: a connector, an ingestion feed or a script that calls the API. It authenticates with an API token, never with a password, so it cannot log in to the web interface. This page shows how to create one, how it differs from a user, and how to convert an account. For the general model, read [How access control works](/opencti/administer/access-control/).

## Before you start

- Your role has the **Manage credentials** capability.
- A group gives the rights the account needs, and its role includes **Allow token usage**, without which the token is refused. For connectors, the built-in Connectors group, with the Connector role, is the usual choice. See [Organize users in groups](/opencti/administer/groups/).

## Create a service account

::::steps

1. Open :menu[Settings > Security > Users] and click **Create User**.
2. Turn on **This user is a service account**.
3. Enter a **Name**. The **Email address** is optional: without one, OpenCTI generates a random address.

   :::only{opencti=">=7.261109"}
   The form hides the password fields while the switch is on, and clears what you typed in them: a service account has no password.
   :::

   :::only{opencti="<7.261109"}
   Leave the password fields empty: a service account has no password.
   :::

4. Choose the **Groups** of the account, for example Connectors. To keep it out of the groups granted by default, such as Default, turn on **Don't add the user to the default groups**.
5. Click **Create**, then open the account.
6. Click **Generate Token**. In the **Generate a new token** panel, give the token a **Name** and a **Duration**: **30 days**, **60 days**, **90 days**, **1 year** or **Unlimited**, the default. Click **Generate**.
7. Copy the token from the **Token details** panel: OpenCTI keeps only a fingerprint, so nobody can read it again once the panel is closed. Put it in the configuration of the connector or script, as `OPENCTI_TOKEN` for a connector.

::::

::screenshot{id=opencti-service-accounts-create alt="The Create a user panel with the This user is a service account switch turned on at the top, above the Name and Email address fields."}

:::example[Give the IP reputation connector an account of its own]
Example CERT runs an IP reputation enrichment connector and wants its enrichments traced to it, not to an administrator.

- **This user is a service account**: on.
- **Name**: `IP reputation connector`. **Email address**: empty, OpenCTI generates one.
- **Groups**: Connectors. **Don't add the user to the default groups**: on.
- Token **Name**: `ip-reputation-2026`. **Duration**: **1 year**, with a reminder to rotate it before January 2027.

In the docker compose file of the connector:

```yaml
services:
  connector-ip-reputation:
    environment:
      OPENCTI_URL: https://opencti.example.com
      OPENCTI_TOKEN: ${IP_REPUTATION_CONNECTOR_TOKEN}
```

Every enrichment the connector writes then shows IP reputation connector as its creator, and revoking its token stops this connector without touching the others.
:::

## How a service account differs from a user

- **No password.** It cannot log in to the web interface, only call the API with its token, and has no password to change.
- **No notifications.** It receives no notifications or digests, and its page has no **Triggers and Digests**.
- **Always in the platform organization.** When a platform organization is set, a service account counts as a member of it, in addition to its own organizations, so organization segregation never blocks its ingestion. It can also log in without any organization.
- **Visible to all.** Every user who can pick users in a list sees service accounts, whatever their organizations.
- **Not held back by authorized members.** It reads and changes objects restricted to named users, groups or organizations, even when it is not one of them, as long as it has their markings. A connector can therefore keep updating a restricted report or case. See [Restrict one object to chosen people](/opencti/administer/sharing/#rules-that-surprise-people).

The **API Tokens** section of the account lists its tokens with when each was **Last Used**, or **Never used**, and when it **Expires At**. A token that expires within seven days shows **Expires soon**: generate the next one, update the connector, then revoke the old one. **Revoke** a token when the connector that used it is removed, or when it leaked. An expired token is refused, and the connector stops writing.

## Accounts created for you

Some integrations create their account themselves, for example a TAXII, RSS or CSV feed set to create its user on the fly. These accounts join only the group set as **Default group for Service accounts** in [Set platform security policies](/opencti/administer/policies/), the Connectors group on a new platform, not the groups granted by default. If no group is set there, OpenCTI refuses to create them.

## Convert a user into a service account

Open the user and click **Convert**, then **Convert** in the dialog.

- **User to service account**: the password is deleted, so the account can no longer log in to the interface. It joins the platform organization and stops receiving notifications. Its ID and tokens stay, so what it created keeps its author and integrations keep working. You can convert it back.
- **Service account to user**: OpenCTI generates a random password, and the user sets their own with the forgotten password flow. If the account was created as a service account, its address is random: give it a real email address first, or the user cannot receive the reset email. The account leaves the platform organization unless it is a member of it.

## Check that it works

Start the connector or script. In the **API Tokens** section of the account, **Never used** gives way to the date the token was **Last Used**, and a connector appears in the list of connectors once it has registered. If the connector logs `You are not allowed to use API Access Tokens`, the roles of the account lack **Allow token usage**.

## What's next

- [Organize users in groups](/opencti/administer/groups/), to give the account only the rights it needs.
- [Grant rights with roles](/opencti/administer/roles/#built-in-roles), for the capabilities of the Connector role.
- [Manage login providers in the UI](/opencti/administer/authentications/#set-platform-wide-login-policies), to exclude service accounts from the IP allow list.
