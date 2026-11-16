---
title: Set the platform parameters
description: Set the OpenCTI platform title, appearance, announcements, map files, themes and custom labels, and check its version, license, dependencies and managers.
sidebar_position: 0
doc_type: how-to
products:
  - opencti
personas: [admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/parameters/
---

The **Parameters** page holds the settings that apply to the whole OpenCTI platform: its name, its look, the banners every user sees and the themes they can pick. It also tells you which version and edition run, which services the platform depends on and which managers are on. This page walks through it card by card, then covers the interface customizations that live in the configuration instead: custom labels, a script snippet and the demo mode.

## Before you start

- Your role has the **Manage parameters** capability, under **Access to admin functionalities**.
- **Third-party analytics** and **Remove Filigran logos** need an Enterprise Edition license. See [Editions and licensing](/start/editions/).
- Settings fixed at deployment, such as the base URL or the mail server, are configuration keys, not parameters. They are listed in the [OpenCTI configuration reference](/reference/opencti/configuration/); the ones that customize the interface are in [Customize beyond the parameters](#customize-beyond-the-parameters).

## Open the parameters

Open :menu[Settings > Parameters]. The fields of the cards save as soon as you leave or change them: there is no save button. Announcements and themes open their own form.

## Read the platform summary

:::only{opencti=">=7.261030"}
The **OpenCTI platform** card at the top gives the **Version**, the **Edition** (**Community** or **Enterprise**), the **Architecture mode** (**Standalone** or **Cluster**), the **Number of node(s)**, how many **Managers** are enabled and the **Platform identifier**, with a **Copy** button. Support and the XTM Hub ask for this identifier. When XTM One is not configured, the card also shows **AI Powered**: the AI provider in use, **Disabled**, or **Missing token**.
:::

:::only{opencti="<7.261030"}
The **OpenCTI platform** card gives the **Platform identifier**, with a copy button, the **Version**, the **Edition** (**Community** or **Enterprise**), the **Architecture mode** (**Standalone** or **Cluster**) and the **Number of node(s)**. It also shows **AI Powered**: the AI provider in use, **Disabled**, or **Missing token**. Support and the XTM Hub ask for the platform identifier.
:::

**AI Powered** reads **Disabled** for one of two reasons, given in its tooltip: the Enterprise Edition is off, or AI is turned off on the platform (**AI is not enabled**). AI is turned on in the **Enterprise Edition** card of :menu[Settings > Filigran Experience]; the provider and its token are set in the [AI configuration](/reference/opencti/configuration/#ai-service).

On a Community platform, the **OpenCTI platform** card has the **Manage your Enterprise Edition license** button, which starts the [Enterprise Edition](/start/editions/). Once the Enterprise Edition is on, two more cards show the license:

- **Enterprise Edition**: the **Organization** the license was issued to, its **Creator** and its **Scope**. **Global** means the license works on every platform of your organization; **Current instance** means it is tied to this platform's **Platform identifier** and does not validate elsewhere.
- **License**: the **Start date**, the **Expiration date** and the **License type**. A warning appears when the license expires in less than three months.

To turn the Enterprise Edition off, click **Disable Enterprise Edition** in the **Enterprise Edition** card, then **Validate**. Your data stays intact. The button is missing when the license comes from the `app:enterprise_edition_license` configuration key. When this toggle is a [protected sensitive configuration](/opencti/administer/sensitive-configuration/), only users allowed to change sensitive configurations can use it.

## Set the name and the sender

The **Configuration** card holds:

- **Platform title**: the name in the browser tab, and the sender name of the emails the platform sends.
- **Platform favicon URL**: the icon of the browser tab.
- **Sender email address**: the address notifications come from. When a sender is forced, in the [SMTP configuration](/opencti/administer/smtp/) or with the `smtp:forced_sender_email` key, the field is greyed out and the forced address is used.

:::only{opencti=">=7.261030"}
- **Google Analytics (v4)**, under **Third-party analytics** (Enterprise Edition): a measurement ID such as `G-XXXXXXXXXX`. To show a consent message at login, set it in [Set platform security policies](/opencti/administer/policies/).
:::

:::only{opencti="<7.261030"}
- **Default theme**, **Language** and **Hidden entity types**, described in the next section.

The **Third-party analytics** card (Enterprise Edition) takes a **Google Analytics (v4)** measurement ID such as `G-XXXXXXXXXX`. To show a consent message at login, set it in [Set platform security policies](/opencti/administer/policies/).
:::

## Change the appearance

:::only{opencti=">=7.261030"}
The **Appearance** card sets defaults for every user. Users can override the theme, the language and the date formats in their profile.

- **Default theme**: one of the themes of the **Themes** card.
- **Language**: **Automatic** follows the browser.
- **Date format in tables** and **Time in tables**: how dates appear in lists.
- **Hidden entity types**: see below.
- **Remove Filigran logos** (Enterprise Edition): hides the Filigran logos of the interface.
:::

:::only{opencti="<7.261030"}
The **Default theme**, **Language** and **Hidden entity types** fields of the **Configuration** card set defaults for every user. Users can override the theme and the language in their profile. **Language** set to **Automatic** follows the browser.

**Remove Filigran logos** (Enterprise Edition), in the **OpenCTI platform** card, hides the Filigran logos of the interface.
:::

**Hidden entity types** removes entity types from menus and creation forms. It only declutters the interface. It is not access control: users can still read and write these types through the API. Groups and organizations can hide more types for their members; the list of types says so next to a type they already hide, for example (Hidden in Groups : Contractors).

## Post an announcement

An announcement is a banner at the top of the interface, for example before a planned downtime.

:::steps

1. In the **Platform announcement** card, click the **+** button (**Add**).
2. In the **Create a message** panel, write the **Message** and pick a **Color**.

   ::screenshot{id=opencti-parameters-announcement-create alt="The Create a message panel of a platform announcement, with its Message and Color fields, the Recipients field and the Dismissible and Activated switches above the Create button."}

3. To target some people only, fill **Recipients** with users, groups or organizations. Leave it empty to reach everyone.
4. Turn on **Dismissible** to let each user close the banner. Leave it off for a banner that stays.
5. Turn on **Activated**, then click **Create**.

:::

The card lists each announcement with its color, message, status, whether it is dismissible and its recipients. Its menu (⋮) updates or deletes it.

Only one banner shows at a time, and an active dismissible announcement takes precedence over a permanent one. When a user closes a dismissible banner, the next active announcement shows. The browser remembers the closing, so the banner shows again in another browser. If you update an announcement, it shows again to users who had closed it.

:::example[Warn every user before an upgrade]
Example CERT upgrades its platform on Saturday 17 January 2026, from 08:00 to 10:00 UTC. Analysts, members and contractors must all know a few days ahead, and must not be able to miss it.

- **Message**: the text below.
- **Color**: `#ff9800`, an orange that reads as a warning in both themes
- **Recipients**: empty, to reach everyone
- **Dismissible**: off, so the banner stays until the upgrade
- **Activated**: on

```text
OpenCTI is upgraded on Saturday 17 January 2026, 08:00 to 10:00 UTC. The platform is unavailable during the upgrade: save your work before 08:00.
```

Every user sees the orange banner at the top of each page from now on. After the upgrade, open the menu (⋮) of the announcement and choose **Delete**, or update it with a short "Upgrade done" message and turn on **Dismissible**.
:::

::::only{opencti=">=7.260811"}
## Replace the map files

The **Map configuration** card holds the files that map widgets are drawn from. File formats and how to build them are in [Map tiles](/deploy/configure/maps/).

:::only{opencti=">=7.261030"}
It has two rows, **Custom map** (a `.pmtiles` file) and **Custom country boundaries** (a `.json`, `.geojson` or `.gz` file). Each row reads **Default**, with **Using the bundled map** or **Using the bundled boundaries**, while the file shipped with OpenCTI is in use, and **Custom** once you upload one. Its menu (⋮) has **Upload**, or **Download**, **Replace** and **Delete** for a custom file. Changes can take up to five minutes to reach every browser.
:::

:::only{opencti="<7.261030"}
Its **Custom map** row takes a `.pmtiles` file: click **Upload** to replace the bundled map. Once a custom map is in place, **Download** saves it and **Delete** goes back to the bundled map.
:::
::::

## Brand the interface with themes

The **Themes** card lists the themes users can pick.

:::only{opencti=">=7.261030"}
The built-in **Filigran Dark** and **Filigran Light** themes can be viewed and exported, not changed or deleted.
:::

:::only{opencti="<7.261030"}
The built-in **Dark** and **Light** themes cannot be renamed or deleted.
:::

:::steps

1. Click the **+** button (**Create a custom theme**), or **Import a theme** to load a JSON file exported from another platform.
2. Give the theme a **Name**, at least two characters and different from the other themes.
3. Set the colors as six-digit hexadecimal values, such as `#122e52`, and the logos as image URLs.
4. Under **Login Page Customisation**, open **Select a background type** under **Right panel customization** and pick what fills the right panel of the login page: **Add background color** (one **Background color**), **Add background gradient** (a **First color** and a **Second color**) or **Image URL** (an image address in **Add image URL**). Once chosen, the image type reads **Add background image**. Changing the type clears the values of the previous one.
5. Click **Create**.
6. To apply the theme to everyone, select it as the **Default theme**. Users who chose another theme in their profile keep theirs.

:::

| Property | Use |
| --- | --- |
| **Background color** | Page background |
| **Paper color** | Cards and panels |
| **Navigation color** | Navigation |
| **Primary color** | Buttons, links and highlights |
| **Secondary color** | Secondary highlights |
| **Accent color** | Selected items |
| **Text color** | All text. Keep enough contrast with the paper and navigation colors |
| **Logo URL** | Expanded left navigation |
| **Logo URL (collapsed)** | Collapsed left navigation |
| **Logo URL (login)** | Login page |

A logo field takes the web address of an image, a path on the platform that starts with `/`, or the image itself as a base64 data URL, such as `data:image/png;base64,...`. A data URL keeps the theme working when the platform cannot reach the image host. Use a wide image for **Logo URL** and **Logo URL (login)**, and a square one for **Logo URL (collapsed)**: each logo is scaled down to fit its place. An empty field keeps the OpenCTI logo. While a user works in a draft, the navigation shows the orange OpenCTI logos instead of yours.

To copy a theme to another platform, open its menu (⋮) and choose **Export**: you get a JSON file. **Delete** removes a custom theme. The default theme cannot be deleted: select another **Default theme** first.

### The theme file

**Import a theme** takes one JSON file holding one theme, with the keys of an export. You can also write it by hand, for example from your brand guide:

| Key | Field |
| --- | --- |
| `name` | **Name**. Required |
| `theme_background`, `theme_paper`, `theme_nav`, `theme_primary`, `theme_secondary`, `theme_accent`, `theme_text_color` | The seven colors, in that order of the table above, as hexadecimal values. Required |
| `theme_logo`, `theme_logo_collapsed`, `theme_logo_login` | The three logos. Optional |
| `theme_login_aside_color`, `theme_login_aside_gradient_start`, `theme_login_aside_gradient_end`, `theme_login_aside_image` | The right panel of the login page: one color, two gradient colors, or an image. Optional |

The import fails with **Failed to import theme** when a theme of the same name exists: change the `name` in the file, or delete the other theme first.

:::only{opencti=">=7.261030"}
The import also refuses a file with a key that is not in the table, a color that is not hexadecimal, or a logo or image that is not an `http` or `https` address, a path starting with `/`, or a PNG, JPEG, GIF, WebP or SVG data URL.
:::

:::example[Brand the platform for Example CERT]
Example CERT wants its own colors on the platform it shares with member hotel chains, and its logo on the login page. The communications team sends the colors; the logos are on the CERT website. The administrator writes `example-cert.json` and imports it with **Import a theme**:

```json
{
  "name": "Example CERT",
  "theme_background": "#0b1d33",
  "theme_paper": "#12294a",
  "theme_nav": "#081526",
  "theme_primary": "#2fa4d9",
  "theme_secondary": "#f2a900",
  "theme_accent": "#7fd1f5",
  "theme_text_color": "#e8eef5",
  "theme_logo": "https://www.example.com/brand/example-cert-wide.png",
  "theme_logo_collapsed": "https://www.example.com/brand/example-cert-square.png",
  "theme_logo_login": "https://www.example.com/brand/example-cert-wide.png",
  "theme_login_aside_gradient_start": "#0b1d33",
  "theme_login_aside_gradient_end": "#2fa4d9"
}
```

The theme appears in the **Themes** card. Once it is selected as the **Default theme**, users who kept the default see the Example CERT colors and logo when they reload the page, and the login page shows a blue gradient beside the logo.
:::

## Check dependencies and managers

::::only{opencti=">=7.261030"}
The **Dependencies** card gives the version of each service the platform relies on: the **Search engine** (Elasticsearch or OpenSearch), **RabbitMQ**, **Redis** and, once registered, **XTM One**.

The **Managers** card lists the background managers, grouped by domain, such as **Core platform**, **Knowledge**, **Ingestion and connectors**, **Notifications**, **Enterprise Edition** and **Telemetry and Filigran ecosystem**. Filter them with **All**, **Enabled** and **Disabled**, plus **Enterprise Edition** on a Community platform, or search them by name. The [manager reference](/reference/opencti/managers/) says what each one does and which configuration keys control it.

A manager reads **Enabled** when the configuration turns it on and **Disabled** when it turns it off. The status describes the configuration, not whether the manager is working right now. In a cluster, a manager reads **Enabled** when at least one node turns it on. On a Community platform, managers that need the Enterprise Edition read **Enterprise Edition**.

::screenshot{id=opencti-parameters-managers alt="The Managers card with the All, Enabled and Disabled filters above managers grouped under Core platform, Knowledge, Ingestion and connectors, Notifications, Enterprise Edition and Telemetry and Filigran ecosystem."}
::::

:::only{opencti="<7.261030"}
The **Tools** card lists each background manager as **Enabled** or **Disabled**, as the configuration sets it, then the version of each service the platform relies on: the **Search engine** (Elasticsearch or OpenSearch), **RabbitMQ** and **Redis**.
:::

To turn a manager on or off, set its `enabled` key in the [Engines, Schedules and Managers](/reference/opencti/configuration/#engines-schedules-and-managers) section of the configuration and restart the platform. [How OpenCTI works](/opencti/concepts/architecture/) explains what each manager does.

## Customize beyond the parameters

Three customizations have no field on this page. They are configuration keys, read when the platform starts, so a change needs a restart.

| Key | Environment variable | Default | Effect |
| --- | --- | --- | --- |
| `app:translations` | `APP__TRANSLATIONS` | `{}` | Replaces labels of the interface, per language. A JSON object maps a language code, such as `en-us` or `fr-fr`, to the labels to replace and their new text. A label you do not list keeps its built-in text |
| `app:script_snippet` | `APP__SCRIPT_SNIPPET` | empty | HTML inserted in every page of the interface, such as an inline `<script>` for a web analytics tool. It runs in the browser of every user, inside their session: only paste code you trust. The content security policy blocks scripts loaded from another host |
| `demo_mode` | `DEMO_MODE` | `false` | Replaces user names and email addresses with a redacted value in the interface, the history and the activity logs, and hides the buttons that connect the platform to the XTM Hub. Use it for public demonstrations, never on a production platform, where it hides who did what |

The labels to replace are the English texts of the interface, as the [translation files](https://github.com/OpenCTI-Platform/opencti/tree/master/opencti-platform/opencti-front/lang/front) write them. For Google Analytics, prefer the **Google Analytics (v4)** field of the **Configuration** card to a snippet.

:::example[Call reports "advisories" for the member hotel chains]
Example CERT publishes its reports to member hotel chains as "advisories", and wants the interface to use their word, in English and in French. The platform engineer sets, in the docker compose file:

```yaml
services:
  opencti:
    environment:
      APP__TRANSLATIONS: '{"en-us": {"Reports": "Advisories"}, "fr-fr": {"Reports": "Avis"}}'
```

After a restart, every place of the interface that showed **Reports** shows **Advisories** to users in English and **Avis** to users in French, menus included. Data, filters and the API keep the Report type: only the labels change.
:::

## What's next

- [Send emails from OpenCTI](/opencti/administer/smtp/) to send emails from the platform.
- [Set platform security policies](/opencti/administer/policies/) for login messages, banners and the main organization.
- [Manage labels, vocabularies and templates](/opencti/configure/taxonomies/) to manage labels, vocabularies and templates.
- [OpenCTI configuration reference](/reference/opencti/configuration/) for the settings fixed at deployment.
