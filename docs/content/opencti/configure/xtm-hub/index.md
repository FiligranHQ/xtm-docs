---
title: Connect to XTM Hub
description: Connect OpenCTI to the XTM Hub to deploy its content in one click, then follow, repair or remove the connection, and know who may deploy what.
sidebar_position: 60
doc_type: how-to
products:
  - opencti
personas: [admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/hub/
---

The XTM Hub is Filigran's portal for ready-made content, free trials, the roadmap and the Academy. Connecting OpenCTI to it lets your teams deploy its content, such as dashboards, playbooks and feeds, in one click. This page shows how to connect the platform, follow and repair the connection, disconnect it, and which rights each kind of content takes. It also describes the rest of the **Filigran Experience** page, where the connection lives.

## Before you start

- To connect or disconnect, your role has the **Manage XTM Hub** capability, under **Access to admin functionalities**. The **Filigran Experience** page opens with **Manage XTM Hub** or **Access to support data**.
- You have an XTM Hub account, in the XTM Hub organization the platform should belong to.
- Both your browser and the platform reach the XTM Hub (`https://hub.filigran.io` by default). The connect button only shows when both do. The keys below are in the [XTM Suite](/reference/opencti/configuration/#xtm-suite) section of the configuration.

| Key | Environment variable | Default | Set it when |
| --- | --- | --- | --- |
| `xtm:xtmhub_url` | `XTM__XTMHUB_URL` | `https://hub.filigran.io` | You use another XTM Hub address. An empty value removes the XTM Hub from the interface |
| `xtm:xtmhub_api_override_url` | `XTM__XTMHUB_API_OVERRIDE_URL` | empty | The platform reaches the XTM Hub at another address than the one users open in their browser, for example through an outbound proxy |

A platform in demo mode (`demo_mode`) shows no connect or disconnect button.

## How the connection works

::diagram{id=admin-xtm-hub-registration}

When you connect, your browser sends the XTM Hub the URL, title, identifier, edition and version of the platform, and brings back a token. Every hour, and each time you open the **Filigran Experience** page, the platform checks with the XTM Hub that its registration is still valid. If the XTM Hub no longer knows the platform, the platform resets its registration and reads as not connected.

:::only{opencti=">=7.260811"}
While connected, the platform also fetches the [news feed](/opencti/configure/xtm-hub/news-feed/) every hour.
:::

## Connect the platform

::::only{opencti=">=7.260811"}
:::steps

1. Open :menu[Settings > Filigran Experience].

   ::screenshot{id=opencti-xtm-hub-card alt="The XTM Hub card of the Filigran Experience page reads Not connected and lists what the XTM Hub offers: a free trial, pre-built content, the roadmap and the Academy."}

2. In the **XTM Hub** card, click **Connect to XTM Hub**. The **Connect your product to XTM Hub** dialog opens.
3. Click **Continue**. A new browser tab opens on the XTM Hub; your OpenCTI session stays open. If you lose the tab, **Continue to connect** in the dialog brings it back.
4. Log in to the XTM Hub and choose the organization to connect the platform to.
5. Go back to OpenCTI. A message confirms that your OpenCTI product is connected, and the card reads **Connected**.

:::

Closing the dialog before the end asks **Close connection process?**: **Yes, close** stops the connection, **Continue connection** goes back to it.

You can also start from the XTM Hub. When it sends you to the platform, a dialog titled **Authorize connection** asks to **Allow OpenCTI to connect with XTM Hub**: click **Continue** and finish in the XTM Hub tab.
::::

::::only{opencti="<7.260811"}
:::steps

1. Open :menu[Settings > Filigran Experience].
2. In the **XTM Hub** card, click **Register in XTM Hub**, then **Continue**. A new browser tab opens on the XTM Hub; your OpenCTI session stays open. If you lose the tab, **Continue to register** brings it back.
3. Log in to the XTM Hub and choose the organization to register the platform in.
4. Go back to OpenCTI. A message confirms that the platform is registered, and the card gives the **Registration date** and who registered it.

:::
::::

## Follow the connection

::::only{opencti=">=7.260811"}
The **Connection status** of the **XTM Hub** card shows one of three states:

| State | Meaning |
| --- | --- |
| **Connected** | The last check succeeded. The card gives the **Connection date** and who connected the platform (**Connected by**), and **Go to the Hub** opens the XTM Hub |
| **Connectivity lost** | The XTM Hub stopped confirming the registration. The card gives the **Last successful check** |
| **Not connected** | The platform is not registered. **Explore XTM Hub** opens a page about the XTM Hub |

To repair a lost connection, click **Connect to XTM Hub** in the card and go through the connection steps again.
::::

:::only{opencti=">=7.261117"}
### See the metrics of a platform in the Filigran cloud

At each connection check, the XTM Hub also tells the platform under which commercial model it is registered. When the platform runs in the Filigran cloud (SaaS) and the card reads **Connected**, the **XTM Hub** card adds the **Your OpenCTI, by the numbers** panel. It names what the XTM Hub shows about your subscription: **Your plan**, the **Contract countdown** before renewal, and **Storage & uptime**, the database space used and the guaranteed uptime.

Click **See my OpenCTI metrics**: the XTM Hub opens in a new tab, on the page of this platform's registration, or on its home page when it cannot find the platform. The panel goes away while the card reads **Connectivity lost**, and when you disconnect the platform. A platform registered under another commercial model never shows it.
:::

::::only{opencti="<7.260811"}
The **XTM Hub** card gives the **Registration status**: **Registered**, with the **Registration date** and who registered the platform (**Registered by**), or **Connectivity lost**, with the **Last successful check**. An unregistered platform shows what the XTM Hub offers and a **Discover the Hub** button.

To repair a lost connection, click **Register in XTM Hub** and go through the registration steps again.
::::

When connectivity has been lost for 24 hours since the last successful check, OpenCTI sends one email to the users who bypass all capabilities or hold **Manage XTM Hub**. It sends another only after the connection has come back and been lost again. The email needs a working [SMTP configuration](/opencti/administer/smtp/).

| Key | Environment variable | Default | Effect |
| --- | --- | --- | --- |
| `xtm:xtmhub_connectivity_email_enabled` | `XTM__XTMHUB_CONNECTIVITY_EMAIL_ENABLED` | `true` | `false` stops the email |
| `xtm:xtmhub_to_email` | `XTM__XTMHUB_TO_EMAIL` | `no-reply@filigran.io` | The **To** address of the email. The administrators receive it in blind copy |
| `smtp:email_max_cc_size` | `SMTP__EMAIL_MAX_CC_SIZE` | `500` | Maximum number of administrators in blind copy. Beyond it, only the first ones receive the email |

:::example[Connect a production and a staging platform]
Example CERT runs two OpenCTI instances: production at `https://opencti.example.com` and staging at `https://opencti-staging.example.com`. Both should deploy XTM Hub dashboards for testing and use, under the XTM Hub organization "Example CERT". Staging is shut down every night, so its connectivity emails would only be noise. Production emails should reach the shared mailbox of the platform team.

1. On each platform, open :menu[Settings > Filigran Experience], click the connect button of the **XTM Hub** card, then **Continue**.
2. In the XTM Hub tab, log in and choose the organization **Example CERT**.
3. Set the connectivity emails and restart each platform. With Docker Compose, on staging:

```yaml
services:
  opencti:
    environment:
      XTM__XTMHUB_CONNECTIVITY_EMAIL_ENABLED: "false"
```

And on production:

```yaml
services:
  opencti:
    environment:
      XTM__XTMHUB_TO_EMAIL: "cti-platform@example.com"
```

Both cards then read connected, with the date and the administrator who connected them, and both instances appear in the Example CERT organization on the XTM Hub. When staging is down for more than 24 hours, it goes to **Connectivity lost** without emailing anyone. When production loses the connection for 24 hours, the platform team's mailbox gets the email, and every administrator gets a blind copy.
:::

## Disconnect the platform

:::only{opencti=">=7.260811"}
In the **XTM Hub** card of a connected platform, click **Disconnect from XTM Hub**. In the **Disconnect your product from XTM Hub** dialog, click **Continue**, and confirm in the XTM Hub tab. As for connecting, **Continue to disconnect** brings back a lost tab, and **Continue disconnection** resumes a disconnection you started to close.
:::

:::only{opencti="<7.260811"}
In the **XTM Hub** card, click **Unregister from XTM Hub**, then **Continue**, and confirm in the XTM Hub tab.
:::

The platform can no longer deploy XTM Hub content. You can connect it again at any time. The button is missing on a platform that runs an Enterprise Edition trial license.

## Who can deploy content

A connected platform lets any user deploy XTM Hub content, but the import runs as that user: each kind of content takes the right to create it. Give these capabilities to the people who should bring content in, such as the CTI team lead for dashboards and playbooks.

:::only{opencti=">=7.260811"}
| Content | Capability |
| --- | --- |
| Custom dashboards | **Create / Update dashboards** |
| Playbooks | **Manage Playbooks** |
| Custom views | **Manage customization** |
| CSV, RSS and TAXII feeds, OpenCTI-to-OpenCTI synchronizers, connectors | **Manage ingestion** |
:::

:::only{opencti="<7.260811"}
| Content | Capability |
| --- | --- |
| Custom dashboards | **Create / Update dashboards** |
| CSV and TAXII feeds, OpenCTI-to-OpenCTI synchronizers, connectors | **Manage ingestion** |
:::

What each kind of content becomes, and what to check before using it, is in the [XTM Hub content library](/opencti/configure/xtm-hub/content-library/).

## The rest of the Filigran Experience page

Next to the **XTM Hub** card, the page holds two more parts.

**The Enterprise Edition card.** Its chip reads **Community edition**, **Activated** or **Expired**. On a Community platform, it lists what the Enterprise Edition adds; with **Manage parameters**, its button starts the Enterprise Edition (see [Editions and licensing](/start/editions/)). Once the Enterprise Edition is on, the card gives the license, as the [platform summary](/opencti/configure/parameters/#read-the-platform-summary) page does, and, with **Manage parameters**, **Disable Enterprise Edition**, unless the license comes from the configuration. With **Manage parameters**, the card also turns the AI features of the platform on or off:

- **Agentic AI (Ariane Assistant)**, marked **Preview**: the assistant users open from the top bar. The first time, click **Validate the Filigran AI Terms**, read them with **Read the Filigran AI Terms**, check the box, then click **I Agree to Filigran AI Terms**. **Decline** keeps the assistant off. Afterwards, a switch turns it on or off.
- **Generative AI (AI Insight, NLQ)**: the AI features built into the screens, such as the summaries of an entity and search in natural language. They use the AI provider set in the [AI configuration](/reference/opencti/configuration/#ai-service).

::::only{opencti=">=7.261109"}
The card also gives the **License source**:

- **OpenCTI license**: an OpenCTI license entered in OpenCTI, or set in its configuration.
- **Filigran license**: the Filigran license of your XTM deployment, the one installed on XTM One, entered in OpenCTI with **Update license** or set in its configuration. OpenCTI accepts it when Filigran signed it and its OpenCTI sub-license is global or names this platform.
- **Filigran license (through XTM One)**: OpenCTI holds no valid license of its own and takes the Filigran license that XTM One returns when the platform registers. This source has no **Disable Enterprise Edition** button; a license entered with **Update license** takes precedence over it.

::screenshot{id=opencti-experience-enterprise-edition-card alt="The Enterprise Edition card of the Filigran Experience page, with its Activated chip, the License source, Organization, Scope, dates and License type of the license, and the switches of the AI features."}
::::

:::::only{opencti=">=7.260811"}
::::xtm-one[With XTM One]
When the platform is registered with XTM One, the first AI row reads **XTM One (Agentic AI)** and its switch governs the XTM One agents in OpenCTI, under the same Filigran AI Terms; the **Generative AI** row goes away. See [XTM One in OpenCTI](/xtm-one/use/in-opencti/).
::::
:::::

**Support packages.** With **Access to support data**, the bottom of the page lists the [support packages](/deploy/operate/support-package/) and has **Generate Support Package**.

## What's next

- [XTM Hub content library](/opencti/configure/xtm-hub/content-library/) for what you can deploy.
- [Follow what XTM Hub publishes](/opencti/configure/xtm-hub/news-feed/) to follow new content.
- [Reuse content from XTM Hub](/workflows/reuse-from-hub/), the end-to-end workflow.
