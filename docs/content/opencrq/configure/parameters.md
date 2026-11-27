---
title: Set the platform's parameters and file storage
description: Read a tenant's name, identifier and OpenCRQ version, choose the home dashboard and default language of its members, and set up the object store that keeps the files they upload.
sidebar_position: 1
doc_type: how-to
products:
  - opencrq
personas: [admin, platform-engineer]
owner: docs-opencrq
last_reviewed: 2026-11-04
---

:menu[Settings > Parameters] shows what identifies a tenant and the OpenCRQ it runs on, and sets the dashboard its members see first and the language they see it in. :menu[Settings > File Storage] lists the files stored in the tenant, which the object store of the deployment keeps. This page sets the home dashboard, reads the identifier that the deployment settings need, and configures the file storage.

## Before you start

- Every member of the tenant can read both pages. Changing the home dashboard or the default language needs **Manage Security** in the tenant. See [Give people access with users, groups and roles](/opencrq/administer/users-and-roles/).
- The file storage is configured in the deployment of OpenCRQ, by whoever runs it.

## Read the tenant and platform parameters

Open :menu[Settings > Parameters]. It has two cards:

- **Tenant**: its **Name**, its **Identifier**, its **Home dashboard** and its **Default language**.
- **OpenCRQ platform**: the **Version** of OpenCRQ that serves the page. Give it with any support request. A platform administrator with **Manage Users** also sees the **Instance identifier**, the identifier this OpenCRQ registers with XTM One under.

The copy button next to the **Identifier**, the **Version** and the **Instance identifier** puts the value in your clipboard.

::screenshot{id=opencrq-parameters alt="The Parameters page of the Grand Meridian Hotels tenant: the Tenant card with its name, its identifier, the CISO home dashboard and the Automatic default language, and the OpenCRQ platform card with its version and its instance identifier."}

The **Identifier** is what the deployment settings of the tenant use: the connections to OpenCTI, OpenAEV, Microsoft Defender and Drata, and the NVD API key, are keyed by it (the `connectors` keys of the [configuration](/reference/opencrq/configuration/#connectors)). It never changes. The tenant's name is set when it is created, and a platform administrator with **Create Tenants** changes it with **Rename** on the platform's **Tenants** page; see [Run several tenants in one platform](/opencrq/administer/tenants/).

## The platform pages, before any tenant

Everything in OpenCRQ happens in a tenant. Whenever OpenCRQ starts and holds no tenant, it creates one, named **Default**, with a first risk scope **Enterprise** and its **Get started** wizard: a platform whose tenants were all deleted gets it back at its next restart. Its name and identifier come from the `initial_tenant` keys of the [configuration](/reference/opencrq/configuration/#initial-tenant), and it has no member yet. XTM One's administrator, signed in as the **Super Administrator**, reaches every tenant without being a member, and lands in it.

A person who signs in through XTM One lands in the first of the tenants they belong to. A platform administrator who belongs to none lands on the platform's **Tenants** page, at `/platform/security/tenants`, where one with **Create Tenants** creates more. The platform's pages also hold its **Roles**, **Groups** and **Users**.

Once they belong to a tenant, a platform administrator reaches the same pages inside it: an address under `/platform/security` opens the page under the tenant's :menu[Settings > Security], with `?scope=platform`. A person who belongs to no tenant and has no **Platform Administration Access** reads **You don't have access to OpenCRQ yet** and "Ask your administrator for access.", with a **Sign out** button. Without that access, an address under `/platform` leads back to the home page. See [Give people access with users, groups and roles](/opencrq/administer/users-and-roles/#manage-the-platform-administrators) and [Run several tenants in one platform](/opencrq/administer/tenants/).

## Choose the home dashboard of the tenant

:::steps

1. Open :menu[Settings > Parameters].
2. In the **Tenant** card, choose a **Home dashboard**: **CISO** (the default), **Risk treatment**, **Risk (Sankey)**, **Findings & Assets** or **Remediation pace**.

:::

The change is saved at once, and **Home** shows that dashboard to every member who has not chosen their own. A member who chose one in their profile keeps it; see [Set your profile and follow your notifications](/opencrq/use/profile/). Members without **Manage Security** see the tenant's choice, and cannot change it. What each dashboard shows is in [Track cyber risk in money with dashboards](/opencrq/use/decide/dashboards/).

:::example[Open on the treatment view for the steering committee]
After the MFA rollout of March 2026, the steering committee of Grand Meridian Hotels opens OpenCRQ to see which controls to fund next, not the overall exposure. Claire Morel, the tenant's cyber risk manager, sets its **Home dashboard** to **Risk treatment**. Every member who has not chosen their own home now lands on each scenario's likelihood, annual loss exposure and the controls to improve first. A member who chose **CISO** in their profile still lands on it.
:::

## Choose the default language of the tenant

In the **Tenant** card, **Default language** sets the language of the interface for every member whose own **Language**, in their profile, is **Automatic**: **Automatic** (the default) leaves it to each member's browser, **English** or **French** sets it. The change is saved at once, and the page reloads when the language shown to you changes. How OpenCRQ chooses a member's language is in [Run several tenants in one platform](/opencrq/administer/tenants/#set-the-default-language-of-a-tenant).

## Find the files of the tenant

Open :menu[Settings > File Storage]. The list shows each file stored in the tenant, with its **Name**, **Type**, **Size**, who uploaded it (**Uploaded By**: a person's email address or a service account's name) and when (**Uploaded At**). A download link appears on each row when the file storage is configured.

The files are the STIX bundles uploaded with **Analyze new bundle** on the threats page, which OpenCRQ analyzes into threats. See [Follow the threats that target your scopes](/opencrq/use/threats/threat-center/). A file cannot be deleted from this page.

## Configure the file storage

OpenCRQ keeps files in an S3-compatible object store, such as RustFS, MinIO or Amazon S3, in one bucket for all tenants. Without it, OpenCRQ starts normally, **Analyze new bundle** is turned off with "File storage is not configured", and the file list has no download link. In the `environment` of the `opencrq` service:

```yaml
FILE_STORAGE__ENDPOINT: https://s3.example.com
FILE_STORAGE__ACCESS_KEY: ${OPENCRQ_S3_ACCESS_KEY}
FILE_STORAGE__SECRET_KEY: ${OPENCRQ_S3_SECRET_KEY}
FILE_STORAGE__BUCKET: opencrq
FILE_STORAGE__REGION: eu-west-3
```

| Key | Default | Effect |
| --- | --- | --- |
| `FILE_STORAGE__ENDPOINT` | | The S3 API endpoint. Setting it turns the file storage on |
| `FILE_STORAGE__ACCESS_KEY` | | The access key. Required once the endpoint is set |
| `FILE_STORAGE__SECRET_KEY` | | The secret key. Required once the endpoint is set. Pass it from a secret store |
| `FILE_STORAGE__BUCKET` | `opencrq` | The bucket that holds the files of every tenant |
| `FILE_STORAGE__REGION` | `eu-west-3` | The region the S3 client sends. Any value for RustFS or MinIO |

Once any `FILE_STORAGE__` key is set, the endpoint and both keys are required: a missing one stops OpenCRQ at start with a configuration error. Restart OpenCRQ after a change. See the `file_storage` keys of the [configuration](/reference/opencrq/configuration/#file-storage).

## Check that it works

Open the threats page: **Analyze new bundle** is enabled. Upload a bundle, then open :menu[Settings > File Storage]: the file is listed with your email under **Uploaded By**, and its download link returns it.

## What's next

- [Configure OpenCRQ](/deploy/configure/opencrq/), for every other deployment setting.
- [Set how OpenCRQ weighs threats, controls and assets](/opencrq/configure/quantification-engine/), the **Engine** settings of the tenant.
- [Read the risk catalogues and manage asset tags](/opencrq/configure/risk-taxonomies/), the **Taxonomies** of the tenant.
