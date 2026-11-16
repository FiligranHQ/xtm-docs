---
title: Editions and licensing
description: What the Community and Enterprise Editions of the XTM products include, how a Filigran license turns the Enterprise Edition on, and what happens when it expires.
doc_type: explanation
products:
  - opencti
  - openaev
  - xtm-one
personas: [evaluator, admin, executive]
owner: docs-platform
last_reviewed: 2026-11-01
sidebar_position: 2
migrated_from:
  - opencti:/administration/enterprise/
  - openaev:/administration/enterprise/
---

OpenCTI and OpenAEV each come in two editions. The Community Edition is free and open source. The Enterprise Edition adds features for organizations with demanding needs: automation, data segregation, single sign-on, audit, AI. Both editions are the same software and the same container images: a license key from Filigran turns the Enterprise Edition on, and nothing needs to be reinstalled. When the license ends, the platform falls back to the Community Edition and keeps all its data. This page explains what each edition contains, how a license is applied, and what happens when it expires.

::diagram{id=start-editions-license}

## One code base, two licenses

The products follow the open core model. The source code of the Enterprise Edition features is public, in the same GitHub repository as the rest of the platform, but under another license:

- the Community Edition is under the Apache License, Version 2.0;
- the Enterprise Edition is under the Enterprise Edition License of the product, in the `LICENSE` file of its repository.

Each source file says in its header which license it falls under; a file without such a header belongs to the Community Edition.

## Who gets a license

Filigran is the only provider of Enterprise Edition licenses.

| Situation | License |
| --- | --- |
| On-premises production use | An annual subscription, under a Filigran Enterprise agreement |
| Filigran SaaS | Included, without anything to do |
| Trying the Enterprise Edition | A free 30-day trial license, from the [Filigran trial page](https://filigran.io/enterprise-editions-trial/) |
| Development, testing and research, such as building a connector or a partner integration | Free of charge, on request |
| Charity organizations with NGO status | Free of charge, on request |

To ask for any of them, [contact Filigran](https://filigran.io/contact/).

## What the Enterprise Edition adds to OpenCTI

The pages of these features carry the Enterprise Edition badge, or say in their first lines which part needs it.

| Area | Enterprise Edition features |
| --- | --- |
| Automation | [Automate with playbooks](/opencti/use/automate/playbooks/), [priority intelligence requirements](/opencti/use/analyze/pir/) |
| Access and segregation | [Separate data between organizations](/opencti/administer/organizations/), restricting an object to its [authorized members](/opencti/administer/sharing/), the restricted entities screen of [markings](/opencti/administer/markings/), [capabilities in drafts](/opencti/administer/roles/), [request access](/opencti/administer/request-access/) |
| Sign-in | [Single sign-on](/opencti/administer/sso/): every login provider other than local email and password, such as OpenID Connect, SAML or LDAP |
| Audit | [Audit who did what](/opencti/configure/activity/): the audit log of administration and security actions, its alerts, and user analytics |
| Finished intelligence | Fintel templates of [entity types](/opencti/configure/entities/), [fintel designs](/opencti/configure/fintel-designs/), [dissemination lists](/opencti/use/share/dissemination-lists/), [email templates](/opencti/configure/email-templates/) |
| Search and AI | [Search inside files](/opencti/configure/file-indexing/) to search inside documents, the AI features of [Refine content with AI](/opencti/use/enrich/refine-content/), [AI extraction](/deploy/configure/ai-extraction/) of entities from documents |
| Integrations | Deploying connectors from the catalog with [XTM Composer](/deploy/integrations/composer/): without the license, the catalog is read-only |
| Customization | Third-party analytics and **Remove Filigran logos**, in the [platform parameters](/opencti/configure/parameters/) |

Throughout the interface, an Enterprise Edition feature on a Community platform carries an EE chip. An administrator can start the Enterprise Edition from there; other users get **Create a feedback**, to ask for it.

::::only{opencti=">=7.260811"}
:::xtm-one[XTM One and the AI features]
The AI features of OpenCTI need the Enterprise Edition. When OpenCTI is registered with XTM One, the XTM One agents take over the agentic AI of OpenCTI, under the same Filigran AI Terms. An administrator turns them on in the **Enterprise Edition** card of :menu[Settings > Filigran Experience]. See [Connect to XTM Hub](/opencti/configure/xtm-hub/) for the card and [XTM One in OpenCTI](/xtm-one/use/in-opencti/) for the agents.
:::
::::

## What the Enterprise Edition adds to OpenAEV

The **Enterprise Edition** card of the OpenAEV settings lists what the license unlocks:

- AI-powered scenario generation and remediation;
- agentless execution through your endpoint detection and response (EDR) tool, with executors such as CrowdStrike Falcon, SentinelOne and Tanium;
- [single sign-on](/openaev/administer/sso/);
- [multi-tenancy](/openaev/administer/multi-tenancy/), to serve several tenants from one platform;
- autonomous scenarios with chaining;
- deploying collectors and injectors from the catalog with [XTM Composer](/deploy/integrations/composer/);
- dedicated technical support.

The [audit log](/openaev/administer/audit-log/) of logins and changes needs it too.

An administrator applies the license from that card: its button opens the **OpenAEV Enterprise Edition (EE) license agreement** dialog, where you paste the license and click **Enable**. A license can also be set in `openaev.application-license`, in the [openaev block](/reference/openaev/configuration/#openaev) of the configuration reference: it replaces the one entered in the card, and the card then offers neither to change it nor to disable it. Once a license applies, the card reads **Activated** and shows its **License source**, **Organisation**, **Scope**, **Start date**, **Expiration date** and **License type**. It reads **Expired** once the license has ended, and **Disable Enterprise Edition** turns the edition off without losing data.

**With the Filigran license of your deployment.** The license installed on XTM One, signed by Filigran for the XTM suite, also works as the license of OpenAEV when its OpenAEV sub-license is global or names this platform. The **Platform identifier** it names is in the **OpenAEV platform** card of :menu[Settings > Parameters]. Enter the license in the card or set it in the configuration, as above: the **License source** then reads **Filigran license**, instead of **OpenAEV license**.

**Through XTM One.** OpenAEV can also take its Enterprise Edition from XTM One, without a license of its own. When OpenAEV is registered with XTM One and the Filigran license installed there covers this platform, XTM One returns it each time OpenAEV registers, every 5 minutes, and OpenAEV verifies its signature itself. The **License source** then reads **Filigran license (through XTM One)**, and the card notes that the Filigran license of the connected XTM One grants the edition. It has no **Disable Enterprise Edition**: OpenAEV holds no license to disable. A valid license entered in OpenAEV or set in its configuration takes precedence. How OpenAEV registers with XTM One is in [Connect the modules](/deploy/configure/connect-modules/#register-the-platforms-with-xtm-one).

An XTM One that answers without its license grants nothing. OpenAEV then logs a warning that starts with `[XTM One] XTM One reports ee_enabled=true but returned no Filigran-signed license certificate`, and asks for an XTM One upgrade.

A Filigran license, entered in OpenAEV or received from XTM One, keeps the Enterprise Edition on for 90 days after its expiration date. A trial license stops at once.

::screenshot{id=openaev-experience-enterprise-edition-card alt="The Enterprise Edition card of the OpenAEV Filigran Experience page, with its Activated chip and the License source, Organisation, Scope, dates and License type of the license in force."}

## Apply a license to OpenCTI

A license is a certificate in PEM format, which you enter in the interface or set in the configuration.

**In the interface.** Open :menu[Settings > Parameters] and click **Manage your Enterprise Edition license**, in the **OpenCTI platform** card. You need the **Manage parameters** capability. Paste the license in the **OpenCTI Enterprise Edition (EE) license agreement** dialog and click **Enable**: by doing so, you accept the Enterprise Edition license terms for your organization. The **Enterprise Edition** card of :menu[Settings > Filigran Experience] has the same button, and **Update license** to replace a license. The [platform summary](/opencti/configure/parameters/#read-the-platform-summary) then shows the license: who it was issued to, its scope, its dates and its type.

**In the configuration.** Set the certificate in `app:enterprise_edition_license`, or `APP__ENTERPRISE_EDITION_LICENSE` in the environment of the platform. A license set this way replaces the one entered in the interface, and the interface no longer offers to disable the Enterprise Edition. Use it when the configuration of your deployment is managed as code. The key is in the [Basic parameters](/reference/opencti/configuration/#basic-parameters) of the configuration reference.

:::only{opencti=">=7.261109"}
**With the Filigran license of your deployment.** The license installed on XTM One, signed by Filigran for the XTM suite, also works as the license of OpenCTI when its OpenCTI sub-license is global or names this platform. Enter it in OpenCTI or set it in the configuration, as above: the **License source** of the **Enterprise Edition** card then reads **Filigran license**, instead of **OpenCTI license**.

**Through XTM One.** OpenCTI can also take its Enterprise Edition from XTM One, without a license of its own. When OpenCTI is registered with XTM One and the Filigran license installed there covers this platform, XTM One returns it at each registration, and OpenCTI verifies its signature itself. The **License source** then reads **Filigran license (through XTM One)**. A valid license entered in OpenCTI or set in its configuration takes precedence. How OpenCTI registers with XTM One is in [Connect the modules](/deploy/configure/connect-modules/).

This needs an XTM One release that sends its license back each time OpenCTI registers, such as the one the [compatibility matrix](/deploy/upgrade/compatibility/) documents: upgrade XTM One before OpenCTI. With an older XTM One, OpenCTI gets no Enterprise Edition from it, and its logs warn `[XTM One] Enterprise Edition not granted by the XTM license`, asking for the XTM One upgrade.

::screenshot{id=opencti-experience-enterprise-edition-card alt="The Enterprise Edition card of the Filigran Experience page, with its Activated chip, the License source, Organization, Scope, dates and License type of the license."}
:::

### What OpenCTI checks

A license turns the Enterprise Edition on only when all of these hold:

- Filigran signed it, for OpenCTI;
- its scope covers the platform: **Global** covers every platform of your organization, **Current instance** only the platform whose **Platform identifier** it names. Give this identifier, from :menu[Settings > Parameters], when you ask for a license tied to one platform;
- today falls between its start date and its expiration date, grace period included;
- on a long-term support (LTS) version, it is an LTS license.

### Long-term support versions

An LTS version of OpenCTI, such as 7.260811 LTS, needs an LTS license before anything else. Until one is applied, every user who signs in sees the **OpenCTI LTS license agreement** dialog instead of the platform, with the **Platform identifier** to give Filigran. A user with the **Manage parameters** capability pastes the license there and clicks **Enable**. A trial or standard license does not open an LTS version. See [Release lifecycle](/deploy/upgrade/lifecycle/) to choose between an LTS line and the latest releases.

## When the license expires

| License | Before it ends | When it ends |
| --- | --- | --- |
| Trial | A banner counts the days left | The Enterprise Edition stops at once |
| Standard or LTS | From three months before, the license cards warn **Your Enterprise Edition license will expire in less than 3 months.** | A red banner counts down a grace period of three months, then the Enterprise Edition stops |

:::only{opencti=">=7.261030"}
A license that comes from XTM One gets a grace period of 90 days instead of three months.
:::

When the Enterprise Edition stops, the platform runs as a Community Edition:

- Enterprise Edition screens show the EE chip again, and the background work behind them stops. Playbooks no longer run, files are no longer indexed, and the audit log records nothing new.
- Your data stays intact: playbooks, PIRs, templates and every piece of knowledge are kept, and work again as soon as a valid license is applied.
- Single sign-on stops working. When no other way to sign in is left, OpenCTI turns local authentication back on, so an administrator can still sign in: see [Sign in the first time and keep an emergency account](/deploy/configure/first-access/).
- On an LTS version, the whole platform is blocked again behind the LTS license dialog.

To turn the Enterprise Edition off on purpose, click **Disable Enterprise Edition** in the **Enterprise Edition** card of :menu[Settings > Filigran Experience], then **Validate**. The same rules apply: features stop, data stays.

## What's next

- [Set the platform parameters](/opencti/configure/parameters/), where the edition and the license show.
- [Release lifecycle](/deploy/upgrade/lifecycle/), for LTS versions and their licenses.
- [Quick start](/start/quick-start/), to try OpenCTI in Community Edition on your machine.
