---
title: Set the name, look and home page of a tenant
description: Name a tenant, set its defaults, colors and logos, restore the starter pack dashboards, tune how injects and agents run, and read the version and services of the platform.
doc_type: how-to
products:
  - openaev
personas: [admin]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/administration/parameters/
---

:menu[Settings > Parameters] sets how a tenant looks to its users. It holds the name in the browser tab, the theme and language new users get, the dashboards that open by default, and the colors and logos of the two themes. It also restores the starter pack dashboards and tunes how injects and agents run. It shows the version, edition and services of the platform, which support asks for first. Each tenant has its own parameters, except **Execution and agents** and **Remove Filigran logos**, which apply to the whole platform. Changes apply without a restart, except the time limit of the OpenAEV implant, which follows **Inject execution threshold** after the next restart (see [Tune inject execution and agents](#tune-inject-execution-and-agents)).

## Before you start

- Your role has **Access tenant settings** to open the screen, and **Manage tenant settings** to change it. The **Admin** group of a tenant has both.
- **Execution and agents** and **Remove Filigran logos** need **Manage platform settings**: they apply to the whole platform. Without it, you only read them.
- **Remove Filigran logos** and **Agent auto-update** need an Enterprise Edition license.
- **Restore starter pack dashboards** needs **Manage dashboards** as well.
- You work in the tenant to set up: its name shows in the left menu when you belong to several.

## How a value is chosen

::diagram{id=openaev-parameters-fallback}

**Platform name**, **Default theme** and **Default language**, when a tenant does not set them, take the value of the platform, then the default of OpenAEV. The dashboards and the colors and logos of the themes take no value from the platform: left empty, they keep the default of OpenAEV. A user's own theme, language or home dashboard, chosen in their profile, comes before the tenant's.

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

::screenshot{id=openaev-parameters-overview alt="The Configuration card of the default tenant, with its platform name, Dark theme, Automatic language and the Technical scenario and Technical simulation dashboards, above the Starter pack dashboards card. Beside them, the OpenAEV platform card shows the tenant identifier, the version 3.261126.1 and the Enterprise Edition."}

The dashboards must exist in the tenant: build them first in **Dashboards**. The starter pack, which OpenAEV loads into each new tenant unless `openaev.starterpack.enabled` is `false` (see the [openaev.starterpack block](/reference/openaev/configuration/#openaev-starterpack)), brings the **Technical scenario** and **Technical simulation** dashboards and sets them as the defaults.

## Restore the starter pack dashboards

When someone deleted dashboards of the starter pack, the **Starter pack dashboards** card, under **Configuration**, brings them back.

:::steps
1. In the **Starter pack dashboards** card, click **Restore starter pack dashboards**.
2. Click **Restore** to confirm.
:::

OpenAEV creates again, under their original names, the starter pack dashboards the tenant no longer has. A dashboard whose name still exists stays as it is, even when someone changed it. An empty **Default scenario dashboard** or **Default simulation dashboard** is set back to the dashboard of the starter pack, and a default you chose is kept. A message gives the number of dashboards restored, or reads **All starter pack dashboards already exist**. The restore works even on a platform where `openaev.starterpack.enabled` is `false`.

## Tune inject execution and agents

The **Execution and agents** card sets how long injects wait for their result and how agents take their jobs. Its values apply to every tenant of the platform. A change applies the next time the platform uses the value, without a restart.

:::steps
1. In the **Execution and agents** card, set the fields:

   | Field | What it sets | Values | Default |
   | --- | --- | --- | --- |
   | **Inject execution threshold** | The minutes an inject waits for the result of its agents before they time out | 1 to 1440 | 10 |
   | **Jobs per agent poll** | The most jobs an agent receives each time it asks the platform for work | 1 to 100 | 5 |
   | **Agent queue threshold** | The pending jobs from which an agent counts as overloaded: new injects skip it. `0` turns the check off | 0 to 10000 | 0 |
   | **Agent auto-update** | Whether agents upgrade when a new version of the platform is deployed | On or off | On |

2. Click **Update**.
:::

::screenshot{id=openaev-parameters-runtime alt="The Execution and agents card on its defaults: Inject execution threshold 10 minutes, Jobs per agent poll 5 jobs and Agent queue threshold 0 jobs, each marked Default value, and Agent auto-update on."}

Raise **Inject execution threshold** when injects need more time, such as an Nmap scan of a large network: past it, they end in **Timeout** (see [Agents that do not report](/openaev/use/injects/status/#agents-that-do-not-report)). The injects already waiting follow the new value. The OpenAEV implant, which agents run for each inject, gets its own time limit when the platform starts: in the existing tenants, it follows a change after the next restart.

Lower **Jobs per agent poll** to spare busy endpoints, and raise it to run large simulations faster. An agent that **Agent queue threshold** skips gets the **AGENT_OVERLOADED** status (see [Inject status](/openaev/use/injects/status/)).

**Agent auto-update** needs an Enterprise Edition license. With it, turn the switch off to keep agents on their version: the platform stops sending them upgrade jobs. Without it, the switch shows off and cannot change, yet agents always upgrade with the platform.

**Default value** under a field means that nobody set it: it keeps the default of the platform configuration. A value set by an environment variable wins over the screen: its field shows an `env` marker and cannot change. Hover the marker to read the name of the variable. Setting the key in `application.properties` changes the default only, which the screen can still override.

| Field | Variable | Key |
| --- | --- | --- |
| **Inject execution threshold** | `INJECT_EXECUTION_THRESHOLD_MINUTES` | `inject.execution.threshold.minutes`, in the [inject block](/reference/openaev/configuration/#inject) |
| **Jobs per agent poll** | `EXECUTOR_OPENAEV_AGENT_MAX_SIMULTANEOUS_JOBS` | `executor.openaev.agent.max-simultaneous-jobs`, in the [executor.openaev block](/reference/openaev/configuration/#executor-openaev) |
| **Agent queue threshold** | `OPENAEV_AGENT_QUEUE_THRESHOLD` | `openaev.agent.queue-threshold`, in the [openaev.agent block](/reference/openaev/configuration/#openaev-agent) |
| **Agent auto-update** | `EXECUTOR_OPENAEV_AGENT_AUTO_UPDATE_ENABLED` | `executor.openaev.agent.auto-update-enabled`, in the [executor.openaev block](/reference/openaev/configuration/#executor-openaev) |

:::example[Give the network scans of Velvet Harbor an hour]
On Example CERT's platform, the Velvet Harbor scenario runs an Nmap scan of the 300 hosts of Grand Meridian's front-desk network. The scan ends in **Timeout** after 10 minutes, before its results come. The platform administrator sets **Inject execution threshold** to `60` and clicks **Update**: the next scans have an hour. To make the value part of the deployment, the platform engineer sets `INJECT_EXECUTION_THRESHOLD_MINUTES=60` in the environment of the platform instead. The field then shows the `env` marker, and nobody changes it from the screen.
:::

## Brand the themes

The **Themes** card customizes the dark and the light theme separately, each in its own tab. Leave a field empty to keep the default of OpenAEV.

:::steps
1. In the **Themes** card, choose the **Dark theme** or **Light theme** tab. An edit you did not save stays when you switch tabs, but each tab saves with its own **Update**.
2. Under **Colors**, enter the colors as `#RRGGBB`, such as `#4CAF50`: **Background color**, **Paper color** (cards and dialogs), **Navigation color**, **Primary color** (buttons and links), **Secondary color**, **Accent color** and **Text color**.
3. Under **Logos**, enter the addresses of your logos: **Logo URL** for the open menu, **Logo URL (collapsed)** for the folded menu, **Logo URL (login)** for the login page.
4. Click **Update**. **Reset** empties the tab again, until you click **Update**.
:::

::screenshot{id=openaev-parameters-theme alt="The Light theme tab of the Themes card, with the primary color #8C1D40 and the two logo addresses of Grand Meridian Hotels filled in, the other colors left on Default."}

When the **Primary color** is too close to the background or paper color, or the **Text color** to either of them, the tab warns **Contrast below the accessibility floor**: the ratio is under 4.5 to 1. It saves anyway: choose another color so that every user can read the screens.

Logo and image addresses must use `https` and point to a public address: OpenAEV refuses the others when you save. On a closed network with no public place for your logos, the platform engineer turns on `openaev.theme.asset-allow-internal-targets`, off by default (see the [openaev.theme block](/reference/openaev/configuration/#openaev-theme)). It also lets reports load images from internal addresses.

The login page comes before any tenant: it keeps the platform's own theme. In a tenant's theme, the fields under **Login page** do not change it: **Login aside color**, **Login aside gradient start color**, **Login aside gradient end color** and **Login aside image URL**.

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
- [Keep agents running well](/openaev/use/agents/#keep-agents-running-well).
- [Troubleshoot OpenAEV](/openaev/administer/troubleshoot/), where the version and the services of this screen help.
