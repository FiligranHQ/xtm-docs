---
title: What's new
description: What OpenCTI 7.261119.0 brings to analysts, team leads, administrators and platform engineers coming from 7.260811 LTS, each change linked to the page that documents it.
doc_type: reference
products:
  - opencti
  - openaev
  - xtm-one
  - opencrq
personas: [admin, cti-analyst, platform-engineer]
owner: docs-platform
last_reviewed: 2026-11-01
sidebar_position: 4
---

::::only{opencti=">=7.261119"}
OpenCTI 7.261119.0, released in November 2026, is the latest release that this documentation describes. This page lists what it changes if you come from the 7.260811 long-term support (LTS) line, the changes of 7.261030 and 7.261109 included. The changes are grouped by who uses them, each with the page that documents it. It leaves out bug fixes and internal changes: the [OpenCTI release notes](https://github.com/OpenCTI-Platform/opencti/releases) on GitHub list every release since 7.260811.0, one by one.

::screenshot{id=opencti-graphs-knowledge alt="The new knowledge graph of a report: badges above the nodes, the legend at the bottom left and the toolbar under the graph."}

## For analysts

| Change | What you get | Page |
| --- | --- | --- |
| A new graph experience | Badges for markings, low confidence and inferred entities, a legend, a toolbar under the graph, details that appear as you zoom, hover cards, keyboard shortcuts, and an export of the whole graph as a high-resolution image | [Work with graphs](/opencti/use/analyze/graphs/) |
| Relationships of a container as a list | A **Relationships** tab on reports, groupings and cases, beside **Entities** and **Observables** | [Explore analysis](/opencti/use/explore/analysis/) |
| A score on threats, malware and events | A **Score** from 0 to 100, set by your team or the source, on intrusion sets, threat actor groups, malware, incidents and events | [Explore threats](/opencti/use/explore/threats/), [Explore arsenal](/opencti/use/explore/arsenal/), [Explore events](/opencti/use/explore/events/) |
| SSVC on vulnerabilities | An **SSVC** tab with the Stakeholder-Specific Vulnerability Categorization decision points | [Explore arsenal](/opencti/use/explore/arsenal/) |
| Counters on observed data | **Number seen** and **Max distinct count**, to tell a single burst from a widespread activity | [Explore events](/opencti/use/explore/events/) |
| Fuzzy search | A `~` after a word matches close spellings: `lazarous~` finds `lazarus` | [Search for knowledge](/opencti/use/explore/search/) |
| Duplicate an investigation | **Duplicate** in the menu of an investigation, to try another line of inquiry | [Pivot and investigate](/opencti/use/analyze/pivot/) |
| Manual security coverage results | Pick the tested entities and enter prevention, detection and vulnerability scores for tests run outside OpenAEV | [Measure your defenses against a threat](/opencti/use/analyze/security-coverage/) |
| Finished intelligence in one step | A fintel PDF generated straight from the **Content** tab, with **Include cover page**, **Include back page** and **Remove empty sections** | [Brand your PDF reports](/opencti/configure/fintel-designs/), [Export knowledge to a file](/opencti/use/share/export/) |
| A Notification Center | The bell of the top bar opens **Alerts** and **Triggers** in one place | [Get alerted when knowledge changes](/opencti/use/automate/notifications/) |
| Steadier indicators | A source whose confidence is lower than the indicator's can no longer change its score or its dates | [How indicators age and expire](/opencti/use/analyze/indicators-lifecycle/) |
| Vulnerability propagation | Inference rules give systems and infrastructures the vulnerabilities of the software they use | [How OpenCTI infers new relationships](/opencti/use/enrich/inferences/) |
| Enrichment inside an investigation | **Enrichment** in the investigation toolbar runs connectors on the selected entities, and you choose which results join the graph | [Pivot and investigate](/opencti/use/analyze/pivot/) |
| Your team's own fields | The custom fields your administrator defines show in the forms and the **Details** card of the types they apply to, and work in filters, bulk updates and playbooks | [Manage labels, vocabularies and templates](/opencti/configure/taxonomies/#custom-fields) |
| Several linked entities at once | **Add relationship** on a relationship's **Linked entities** links several objects in one go, and creates the missing ones | [Explore knowledge](/opencti/use/explore/) |
| Variables in dashboards | A variables bar on custom dashboards: a widget filter takes a **Variable** instead of a **Fixed value**, so changing the variable updates every widget that uses it. **Save** > **Save as new preset…** keeps the values of the variables with the time range, to restore them in one click | [Build dashboards](/opencti/use/dashboards/) |
| A matrix in investigations | **Matrix view**, beside **Graph view**, places the attack patterns of an investigation on the tactics matrix, with a heat map of how much each technique is used, and **Export to image** saves it. On reports, groupings and cases, **Tactics matrix view** is now named **Matrix view** | [Pivot and investigate](/opencti/use/analyze/pivot/) |
| Images in the Content tab | PNG and JPEG files preview in the **Content** tab, a protected PDF asks for its password, and a file of the **Data** tab opens there, with **Open in a new tab** beside it | [Explore knowledge](/opencti/use/explore/) |
| A history read at a glance | Each event of the history fits on one line, with the full text in a tooltip and the details on a click | [Explore knowledge](/opencti/use/explore/) |
| Empty numeric fields | **Empty** and **Not empty** in numeric filters, such as the EPSS score of vulnerabilities | [Explore arsenal](/opencti/use/explore/arsenal/) |

:::xtm-one[XTM One in this release]
**Choose AI agent**, beside **Ask AI** in the search bar, picks the XTM One agent that translates your question into filters. Exports generated with **Ask AI (multiple formats supported)** go through XTM One too. See [Search for knowledge](/opencti/use/explore/search/) and [Refine content with AI](/opencti/use/enrich/refine-content/).
:::

## For CTI team leads and administrators

| Change | What you get | Page |
| --- | --- | --- |
| Ready-made custom views | **Import from Hub** in the **Custom Views** tab, to browse the views of the XTM Hub | [Add a dashboard tab to entity pages](/opencti/use/dashboards/custom-views/) |
| Statuses kept between platforms | **Entity status sync**, so entities received from another OpenCTI platform keep a status of the same name | [Track the progress of reports and cases](/opencti/use/automate/entity-workflows/), [Import from another OpenCTI platform](/opencti/use/ingest/streams/) |
| A new playbook editor | **Add an entry point** on an empty canvas, **Insert a component on this link**, a **Components** panel with **Search components**, **Undo** and **Redo**, **Edit mode** and **View only**, a **Horizontal layout** or a **Vertical layout**, and **Delete this step only** to keep the steps after it. A playbook has a single entry point, and a step whose required choice is empty cannot be saved | [Automate with playbooks](/opencti/use/automate/playbooks/) |
| A status per entity type in playbooks | **Manipulate knowledge** asks for the **Entity type** whose workflow holds the **Status** it sets, and leaves the elements of other types untouched | [Playbook components](/reference/opencti/playbook-components/) |
| A default dashboard preset | **Default preset**, under **Default dashboard** on a profile, a group or an organization, opens the dashboard on the variable values and time range of that preset | [Read the home dashboard and AI summaries](/opencti/use/explore/insights/), [Separate data between organizations](/opencti/administer/organizations/) |
| A license bound to its platform | A Filigran license of the latest model, entered in OpenCTI, must be issued for this platform or for every platform; any other is refused | [Editions and licensing](/start/editions/) |
| OpenID Connect with Basic authentication | `token_endpoint_auth_method` set to `client_secret_basic` in the **Extra configuration** of an OpenID Connect provider is now applied, as Okta and Authelia expect | [Manage login providers in the UI](/opencti/administer/authentications/) |
| Logs of every feed | A **Logs** tab on TAXII, CSV, RSS and JSON feeds and on OpenCTI streams, with each run | [How intelligence gets into OpenCTI](/opencti/use/ingest/) |
| Faster draft review | Users with **Bypass all capabilities** move a draft to any status from the toolbar | [Approve drafts step by step](/opencti/use/automate/draft-workflow/) |
| Clearer connector status | Filters on the **Deployment** (**Managed** or **Manual**), the state (**Started** or **Stopped**) and the **Version** of each connector | [Import with connectors](/opencti/use/ingest/connectors/) |
| A redesigned Parameters page | Default theme, language and date formats for every user, a grid of managers, the versions of the dependencies, and the built-in **Filigran Dark** and **Filigran Light** themes | [Set the platform parameters](/opencti/configure/parameters/) |
| Your own country boundaries | **Custom country boundaries**, beside the custom map, to color countries in map widgets | [Set the platform parameters](/opencti/configure/parameters/), [Maps](/deploy/configure/maps/) |
| The Filigran license | The Filigran license of your XTM deployment turns on the Enterprise Edition, entered in OpenCTI (**Filigran license**) or returned by XTM One (**Filigran license (through XTM One)**) | [Editions and licensing](/start/editions/) |
| Custom fields | New typed fields, such as a selection list of impacts, added to the entity types you choose (Enterprise Edition), with the new **Manage custom fields** capability | [Manage labels, vocabularies and templates](/opencti/configure/taxonomies/#custom-fields), [Customize entity types](/opencti/configure/entities/) |
| Forms without edit rights | **Create knowledge via Form Intake** lets a role fill in forms without **Create / Update knowledge** | [Create knowledge with forms](/opencti/use/ingest/forms/), [Grant rights with roles](/opencti/administer/roles/#capabilities-added-by-an-upgrade) |
| Merge user accounts | Move what one account created and owns to another account of the same person, through the API | [Manage users](/opencti/administer/users/#merge-two-accounts-of-the-same-person) |
| Blog posts in the news feed | A **Blogpost** tab with the latest posts of the XTM Hub | [Follow what XTM Hub publishes](/opencti/configure/xtm-hub/news-feed/) |
| Delegated administration across organizations | An administrator of several organizations gives new users the grantable groups of all of them | [Separate data between organizations](/opencti/administer/organizations/) |
| Markings in notifications | Each notified object carries its markings, to print its TLP in a custom notifier | [Send notifications by email or webhook](/opencti/configure/notifiers/) |
| Integrations in the news feed | XTM Hub news about new integrations, marked **Integration** | [Follow what XTM Hub publishes](/opencti/configure/xtm-hub/news-feed/) |
| Usage telemetry that can be stopped | `telemetry_manager:enabled` set to `false` stops what the platform sends to Filigran | [Usage telemetry and privacy](/opencti/configure/telemetry/) |

## For platform engineers

| Change | What to know | Page |
| --- | --- | --- |
| Node.js 22.18.0 | The minimum version for a manual installation | [Requirements](/deploy/plan/requirements/) |
| Session signing key | Derived from the encryption key and shared by every node; `app:session_secret` is gone, with nothing to set instead | [OpenCTI configuration](/reference/opencti/configuration/) |
| Health checks in the background | The health endpoint returns the last check of the dependencies, with `&details=true` for each of them | [Check that the platforms are healthy](/deploy/operate/health-probes/) |
| Keep-alive behind a load balancer | `app:keep_alive_timeout`, 65 seconds by default, longer than the idle timeout of most load balancers | [Check that the platforms are healthy](/deploy/operate/health-probes/) |
| More observability metrics | Lock contention, the state of each dependency and the storage used | [Metrics](/deploy/operate/metrics/), [Scale](/deploy/operate/scale/) |
| Redis | TLS server name and per-node checks in a cluster, and large events moved to S3 storage with `REDIS__MAX_EVENT_LENGTH` | [Shared services](/deploy/configure/shared-services/) |
| Elasticsearch | `ELASTICSEARCH__MAX_MAPPING_FIELDS` raises the field limit of the indices | [Shared services](/deploy/configure/shared-services/) |
| Upload limits | `app:max_upload_file_size` and `app:max_upload_files` | [Harden](/deploy/operate/harden/) |
| Remote catalogs | Catalog entries given as URLs, loaded by the catalog manager | [XTM Composer](/deploy/integrations/composer/) |
| Template limits | Configurable memory limits and worker pool for the renders of notifier and email templates | [Send notifications by email or webhook](/opencti/configure/notifiers/), [Customize the emails OpenCTI sends](/opencti/configure/email-templates/) |
| Archive limit | An archive whose content exceeds 100 MB is imported as is, set with `APP__ARTIFACT_ZIP_MAX_EXTRACTED_SIZE` | [Explore observations](/opencti/use/explore/observations/) |
| Connection to XTM One | OpenCTI reads the public identity of XTM One and addresses its tokens to it | [Connect the modules](/deploy/configure/connect-modules/) |
| Certificate checks in one setting | `TLS__REJECT_UNAUTHORIZED` checks the certificates of every outgoing connection without its own setting. Off by default, with a warning at start: a later version turns it on | [Trust your certificate authority and proxy](/deploy/configure/certificates/) |
| Secure GraphQL defaults | `APP__GRAPHQL__SECURE_BY_DEFAULT` turns on the armor protection, turns off the playground and keeps WebSocket for subscriptions. Off by default, with a warning at start: a later version turns it on | [Harden the deployment](/deploy/operate/harden/#secure-graphql-defaults) |
| Search engine client | `ELASTICSEARCH__ENGINE_CLIENT` replaces `ELASTICSEARCH__ENGINE_SELECTOR` and `ELASTICSEARCH__ENGINE_CHECK`, now ignored. Elasticsearch 9 gets its own client; versions before 9 are deprecated | [Shared services](/deploy/configure/shared-services/) |
| Prometheus address | `APP__TELEMETRY__METRICS__EXPORTER_PROMETHEUS_HOST`, `0.0.0.0` by default, sets where the metrics endpoint listens | [OpenCTI configuration](/reference/opencti/configuration/#telemetry) |
| JSONPath limits | JSONPath filters of JSON mappers and feeds run in worker threads, with a timeout and memory limits set by the `json_path` keys | [OpenCTI configuration](/reference/opencti/configuration/#technical-customization) |
| Secure HTTP defaults | `APP__HTTP__SECURE_BY_DEFAULT` lowers the default request body limit to 10 MB, the default rate protection to 30000 requests per 60 seconds and the redirects followed by outgoing calls to 5. A key you set keeps its value. Off by default, with a warning at start: a later version turns it on | [Harden the deployment](/deploy/operate/harden/) |
| Redirects of outgoing calls | `APP__HTTP__MAX_REDIRECTS`, 21 by default, caps the redirects that feeds, synchronizers and notifiers follow | [Harden the deployment](/deploy/operate/harden/) |
| TAXII payload size | The TAXII API root advertises `app:max_payload_body_size` as its `max_content_length`, instead of a fixed 100 MB | [OpenCTI configuration](/reference/opencti/configuration/) |

Before you upgrade, read [Breaking changes](/deploy/upgrade/breaking-changes/) and [Upgrade](/deploy/upgrade/).

## The other products

This documentation describes OpenAEV 3.261118.0, XTM One 1.261119.0 and OpenCRQ 0.261120.0 with OpenCTI 7.261119.0. OpenCRQ 0.261120.0 is the first public release of the cyber risk quantification product: it prices each risk of your register in money, from the threats you track in OpenCTI and the results of your OpenAEV simulations. Start with [OpenCRQ](/opencrq/), then [Install OpenCRQ](/deploy/install/opencrq/). Since 1.261112.0, XTM One streams its chat over HTTP instead of a WebSocket: the reverse proxy in front of it must serve HTTP/2 to browsers, without buffering `/api/` and with a read timeout above 15 seconds. See [XTM One 1.261112.0](/deploy/upgrade/breaking-changes/#xtm-one-1-261112-0). What each OpenAEV release changes is in the [OpenAEV release notes](https://github.com/OpenAEV-Platform/openaev/releases) on GitHub; which versions work together is in the [compatibility matrix](/deploy/upgrade/compatibility/).
::::

::::only{opencti="<7.261119"}
You are reading the documentation of an OpenCTI long-term support (LTS) line. What the latest release brings is on this page in the documentation of the latest version: switch to it with the version picker. The [OpenCTI release notes](https://github.com/OpenCTI-Platform/opencti/releases) on GitHub list every release, and [Release lifecycle](/deploy/upgrade/lifecycle/) explains how to choose between an LTS line and the latest releases.
::::
