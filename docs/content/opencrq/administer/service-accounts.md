---
title: Let integrations sign in with service accounts
description: Create the service accounts OpenCRQ's connectors and your scripts write as, give them tokens with an expiry, and rotate, revoke or disable them.
sidebar_position: 3
doc_type: how-to
products:
  - opencrq
personas: [admin, developer]
owner: docs-opencrq
last_reviewed: 2026-11-20
---

A service account is the identity an integration uses in a tenant of OpenCRQ: the threats the OpenCTI connector streams in, the results it imports from OpenAEV, the requests a script sends to the API. It belongs to no person, so the work it does survives the departure of whoever set it up, and you can see, limit and cut off what each integration does. This page creates one for a connector, gives one a token for a script, and shows how to rotate, revoke and disable them.

## How a service account works

- **It belongs to one tenant.** It reads and writes that tenant's data like a member, and never sees another tenant. A token carries its tenant: a request made with it needs no `x-tenant-id` header.
- **Its groups add capabilities.** Like a person, it gets the capabilities of the roles of its groups. The creation panel puts a new account in the tenant's **Connectors** group, which has no role: it can work on data, and cannot manage the tenant's security.
- **OpenCRQ's own connectors write as it, without a token.** The OpenCTI, OpenAEV, Microsoft Defender, Drata and CVE connectors run inside OpenCRQ: their **Configure integration** panel, and the panel of the same source in the steps of **Get started**, ask for a **Service account**, and what they import is recorded as that account's work. A connector without an account does not run its scheduled syncs. **OpenCRQ Default Threats** is the exception: it reads a public file, asks for no account, and writes as the platform.
- **A script signs in with a token.** It sends `Authorization: Bearer` and the token to the [OpenCRQ API](/reference/opencrq/api/). Tokens start with `flgrn_ocrq_tkn_`.
- **Its work is traced.** A run of a background job it started shows **Service account** under **Triggered by** in :menu[Settings > Jobs], and a file it stored shows its name under **Uploaded By** in :menu[Settings > File Storage]. The tags and groups it sets on an asset through the API count as a push from XTM One: such a tag shows **XTM One** as its **Source** in :menu[Settings > Taxonomies > Tags], and a push replaces only what an earlier push set, never the tags and groups people set by hand. See [Manage the tags of your assets](/opencrq/configure/risk-taxonomies/#manage-the-tags-of-your-assets).

## Before you start

- Your role has **Manage Security** in the tenant. XTM One's administrator, who signs in to OpenCRQ as its super administrator, holds it in every tenant. See [Give people access with users, groups and roles](/opencrq/administer/users-and-roles/).

## Create a service account

:::steps

1. Open :menu[Settings > Security > Service accounts] and click **Create**.
2. In the **Create a new service account** panel, enter a **Name** that says which integration uses it, and a **Description**.
3. Under **Groups**, keep **Connectors**, which is chosen for you. Add a group whose role grants **Manage Security** only if the integration must manage members, groups or integrations, or read or change the settings of :menu[Settings > Engine]: OpenCRQ refuses these calls to an account without it.
4. Click **Save**. The account appears in the list as **Active**.

   ::screenshot{id=opencrq-service-accounts-list alt="The Service accounts list of the Grand Meridian Hotels tenant: the OpenCTI connector account, Active, in the Connectors group."}

5. To let a connector write as it, open **Integrations**, open the connector, click **Configure**, choose the account under **Service account** in the **Configure integration** panel, and click **Save**. See [Connect OpenCTI with a Live Stream](/opencrq/use/sources/opencti/) for the OpenCTI connector.

:::

:::example[Give the OpenCTI connector its own account]
Grand Meridian Hotels streams the threats Example CERT publishes in OpenCTI into its tenant. Claire Morel, who administers the tenant, creates the account the connector writes as:

- **Name**: `OpenCTI connector`.
- **Description**: `Writes the threats of the OpenCTI Live Stream`.
- **Groups**: **Connectors** only. The connector writes threats; it never manages access.

In **Integrations**, the OpenCTI connector is configured with its **Live Stream** and this **Service account**. The threats of Velvet Harbor and Ember Tide then reach the tenant as the work of **OpenCTI connector**, and the account's page shows no capability under **Effective Permissions**: no token is needed, since the connector runs inside OpenCRQ.
:::

## Give a script a token

:::steps

1. Open the service account from the list, and click **Generate token** under **API Tokens**.
2. In the **Generate API token** panel, enter a **Token name** that says where the token is used, and choose its **Expiry**: **1 day**, **7 days**, **30 days**, **90 days** (the default), **1 year** or **No expiry**.

   ::screenshot{id=opencrq-service-accounts-token alt="The Generate API token panel: a Token name field and an Expiry selector set to 90 days by default."}

3. Click **Generate**. The **Your new API token** dialog shows the token once: click **Copy & close**, and store it in your secret store right away. OpenCRQ keeps only a hash of it.

:::

The **API Tokens** table lists each token by its name and the start of its value, with its **Status** (**Active**, **Revoked** or **Expired**), when it was **Last used**, when it **Expires**, and when it was **Created**. A token stops working at its expiry, when it is revoked, or while its account is disabled.

A token generated before 0.261120.0 starts with `ogrc_`, and OpenCRQ refuses it, although the table still lists it as **Active** with the new prefix. After the upgrade, generate a new token for each script, update the script, then revoke the old token.

:::example[Export the price of every scenario each week]
Grand Meridian's finance team loads the annualized loss of each risk scenario into its budget workbook every Monday. Claire Morel creates a service account **Name** `Weekly risk export` in the **Connectors** group, and generates a token **Token name** `Finance workbook`, **Expiry** **90 days**. The finance script keeps it in the variable `OPENCRQ_EXPORT_TOKEN` of its scheduler and calls:

```bash
curl -s https://crq.example.com/trpc/riskScenarios.getRiskScenariosOverview \
  -H "Authorization: Bearer ${OPENCRQ_EXPORT_TOKEN}"
```

The answer lists the scenarios of Grand Meridian France and Spain with their latest run. **Last used** shows the Monday of each export, and three months later the token expires: Claire generates the next one before it does.
:::

## Rotate, revoke or disable

- **Rotate** a token from its actions menu: OpenCRQ revokes it and shows a new token with the same name, once. In this version, the new token expires 24 hours after the rotation when the old one had an expiry date, whatever its length. To keep a longer expiry, generate a new token with the expiry you want, update the integration, then revoke the old one.
- **Revoke** a token from its actions menu and confirm: every integration that uses it loses access at once. A revoked token cannot be restored.
- **Disable** an account from its actions menu in the list, or with the **Active** switch of its **Manage** panel: its tokens stop working, and **Enable** brings them back. To stop a connector that writes as the account, also turn the connector off or change its configuration on its own page. A service account cannot be deleted.

Keep each integration on its own account and its own token, give each token the shortest expiry the integration tolerates, and revoke the tokens whose **Last used** date shows they are no longer used.

:::xtm-one[XTM One agents work as people, not as service accounts]
An XTM One agent that works in OpenCRQ acts as the person who asks it: XTM One signs each call with that person's XTM One identity, and OpenCRQ matches it to the account that person signs in with, never to an email address. No service account or token is needed for XTM One, and what an agent does is recorded as that person's work.
:::

## Check that it works

Open the service account: **Groups & Roles** and **Effective Permissions** show what it may do, and **API Tokens** shows when each token was **Last used**. For a connector, open its page under **Integrations**: it names the **Service account** it writes as and the groups of that account.

## What's next

- [Follow the background jobs and run one again](/opencrq/administer/background-jobs/), to follow the syncs a service account runs.
- [OpenCRQ API](/reference/opencrq/api/), every procedure a script can call.
- [Connect OpenCTI with a Live Stream](/opencrq/use/sources/opencti/), to bind the OpenCTI connector to its account.
