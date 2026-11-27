---
title: Set the name, look and home page of a tenant
description: Name a tenant, set its default theme, language and dashboards, apply your colors and logos, and read the version, edition and services of the platform.
doc_type: how-to
products:
  - openaev
personas: [admin]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/administration/parameters/
---

:menu[Settings > Parameters] sets how a tenant looks to its users. It holds the name in the browser tab, the theme and language new users get, the dashboards that open by default, and the colors and logos of the two themes. It also shows the version, edition and services of the platform, which support asks for first. Each tenant has its own parameters, and changes apply at once, without a restart.

## Before you start

- Your role has **Access tenant settings** to open the screen, and **Manage tenant settings** to change it. The **Admin** group of a tenant has both.
- **Remove Filigran logos** needs an Enterprise Edition license and **Manage platform settings**: it applies to the whole platform.
- You work in the tenant to set up: its name shows in the left menu when you belong to several.

## How a value is chosen

::diagram{id=openaev-parameters-fallback}

A parameter left empty in a tenant takes the value of the platform, then the default of OpenAEV. A user's own theme, language or home dashboard, chosen in their profile, comes before the tenant's.

## Set the configuration

:::steps
1. Open :menu[Settings > Parameters].
2. In the **Configuration** card, set the fields:

   | Field | What it sets | Default |
   | --- | --- | --- |
   | **Platform name** | The name in the browser tab and the navigation. Required | `OpenAEV - Open Adversarial Exposure Validation Platform` |
   | **Default theme** | The theme of users who chose none: **Default**, **Dark** or **Light** | Dark |
   | **Default language** | The language of users who chose none: **Automatic** follows the browser, or one of the nine languages | **Automatic** |
   | **Home dashboard** | The custom dashboard of the home page | **Platform default**, the home page of OpenAEV |
   | **Default scenario dashboard** | The custom dashboard of the **Statistics** tab of every scenario | **Technical scenario**, from the starter pack |
   | **Default simulation dashboard** | The custom dashboard of the **Statistics** tab of every simulation | **Technical simulation**, from the starter pack |

3. Click **Update**.
:::

::screenshot{id=openaev-parameters-overview alt="The Configuration card of the default tenant, with its platform name, Dark theme, Automatic language and the Technical scenario and Technical simulation dashboards, next to the OpenAEV platform card with the tenant identifier, the version 3.261029.0 and the Enterprise Edition."}

The dashboards must exist in the tenant: build them first in **Dashboards**. The starter pack, which OpenAEV loads into each new tenant unless `openaev.starterpack.enabled` is `false` (see the [openaev.starterpack block](/reference/openaev/configuration/#openaev-starterpack)), brings the **Technical scenario** and **Technical simulation** dashboards and sets them as the defaults.

## Brand the themes

The **Dark theme** and **Light theme** cards customize each theme separately. Leave a field empty to keep the default of OpenAEV.

:::steps
1. In the card of the theme, enter the colors as `#RRGGBB`, such as `#4CAF50`: **Background color**, **Paper color** (cards and dialogs), **Navigation color**, **Primary color** (buttons and links), **Secondary color**, **Accent color** and **Text color**.
2. Enter the addresses of your logos: **Logo URL** for the open menu, **Logo URL (collapsed)** for the folded menu, **Logo URL (login)** for the login page.
3. Click **Update**. **Reset** empties the card again, until you click **Update**.
:::

::screenshot{id=openaev-parameters-theme alt="The Light theme card with the primary color #8C1D40 and the two logo addresses of Grand Meridian Hotels filled in, the other colors left on Default."}

When the **Primary color** is too close to the background or paper color, or the **Text color** to either of them, the card warns **Contrast below the accessibility floor**: the ratio is under 4.5 to 1. It saves anyway: choose another color so that every user can read the screens.

Logo and image addresses must use `https` and point to a public address: OpenAEV refuses the others when you save. On a closed network with no public place for your logos, the platform engineer turns on `openaev.theme.asset-allow-internal-targets`, off by default (see the [openaev.theme block](/reference/openaev/configuration/#openaev-theme)). It also lets reports load images from internal addresses.

The login page comes before any tenant: it keeps the platform's own theme. The **Login aside color**, **Login aside gradient start color**, **Login aside gradient end color** and **Login aside image URL** fields of a tenant's theme do not change it.

## Read the platform information

The **OpenAEV platform** card describes the deployment:

| Field | What it shows |
| --- | --- |
| **Tenant identifier** | The identifier of this tenant, to copy into an integration or a single sign-on provider |
| **Platform identifier** | The identifier of the platform |
| **Version** | The version of the platform. When the image was built with a commit (`OPENAEV_COMMIT`), hover it to see the **Commit hash**, and click it to copy both |
| **Edition** | **Enterprise** or **Community** |
| **AI Powered** | The AI provider of the platform, or **Disabled**. The row is hidden when OpenAEV is registered with XTM One |
| **Remove Filigran logos** | Hides the Filigran logos of the interface, for the whole platform |

The **Tools** card lists the **JAVA Virtual Machine**, **PostgreSQL**, **RabbitMQ** and search engine versions, and whether the **Telemetry manager**, **SMTP** and **IMAP** are on. Only administrators see the versions: the bootstrap admin and the users single sign-on maps to administrators. **SMTP** reads **Disabled** when the platform cannot connect to its outgoing mail server (the [spring.mail block](/reference/openaev/configuration/#spring-mail)), and **IMAP** when incoming mail is off or unreachable (the [openaev.mail block](/reference/openaev/configuration/#openaev-mail)).

:::example[Brand the tenant of Grand Meridian Hotels]
Grand Meridian Hotels wants its people to recognize their own workspace on Example CERT's platform. In the **Grand Meridian Hotels** tenant, its administrator sets:

| Field | Value |
| --- | --- |
| **Platform name** | `Grand Meridian validation` |
| **Default theme** | **Light** |
| **Default language** | **French** |
| **Home dashboard** | `Velvet Harbor replay`, the dashboard of the last simulation of Velvet Harbor against the front-desk workstations |
| **Primary color** (**Light theme**) | `#8C1D40` |
| **Logo URL** (**Light theme**) | `https://static.grand-meridian.example/brand/logo-wide.svg` |
| **Logo URL (collapsed)** (**Light theme**) | `https://static.grand-meridian.example/brand/logo-square.svg` |

New users of the tenant open OpenAEV in French, in the light theme, on the Velvet Harbor dashboard, with Grand Meridian's logo and color. The other tenants of the platform keep their own look.
:::

:::xtm-one[The name XTM One shows]
When OpenAEV is registered with XTM One, XTM One names it with the **Platform name** of the default tenant. Rename the default tenant's platform to tell your OpenAEV instances apart in XTM One.
:::

## What's next

- [Set the login message and the consent box](/openaev/administer/policies/).
- [Give each organization its own tenant](/openaev/administer/multi-tenancy/).
- [Troubleshoot OpenAEV](/openaev/administer/troubleshoot/), where the version and the services of this screen help.
