---
title: Import with connectors
description: Choose a connector in the OpenCTI catalog, deploy it, check what it imports in its works and errors, and reset, stop or remove it.
sidebar_position: 1
doc_type: how-to
products:
  - opencti
personas: [cti-lead, admin, cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/import/external-connectors/
---

Connectors bring intelligence into OpenCTI from vendors, open source projects and security tools, each through its own integration. As a team lead, you decide which sources feed your team, check that each one delivers, and act when one stops. This page follows a connector from the catalog to its daily follow-up: what its screens tell you, what its state and its works are, and what each action changes. An administrator usually deploys the connectors and owns their accounts. For how imported data reaches the knowledge base, read [How intelligence gets into OpenCTI](/opencti/use/ingest/) first.

## Before you start

| To | You need |
| --- | --- |
| See the connectors, their state and their works | The **Access connectors** capability |
| Deploy a connector from the catalog | The **Manage ingestion** and **Connectors API usage: register, ping, export push ...** capabilities, an Enterprise Edition license, and XTM Composer running beside the platform. See [Deploy integrations with XTM Composer](/deploy/integrations/composer/) |
| Start, stop, reset or delete a connector, clear or delete its works | The **Manage connector state** capability |
| Update the options of a connector deployed with XTM Composer | The **Manage connector state** and **Connectors API usage: register, ping, export push ...** capabilities |
| See the [forms](/opencti/use/ingest/forms/) listed with the integrations, and open one to fill it in | The **Create / Update knowledge** capability |
| See the roles, groups and organizations of the account a connector writes as | The **Manage credentials** capability |

:::only{opencti=">=7.261109"}
**Create knowledge via Form Intake** also shows the forms and lets you fill them in, without **Create / Update knowledge**.
:::

With an Enterprise Edition license but no XTM Composer registered, a banner, which you can dismiss, says that deploying requires the **Integration Manager**, and the deployment form is read-only. Without an Enterprise Edition license, **Deploy** carries the Enterprise Edition badge.

## What each kind of connector does

The **Type** of a connector says when it runs and what it does to your knowledge.

| Type | What it does for your team |
| --- | --- |
| **External import** | Fetches a source on its own schedule, such as a vendor feed or the MITRE ATT&CK matrices, and imports what is new |
| **Internal enrichment** | Adds context to an entity, on request or automatically. See [Enrich with connectors](/opencti/use/enrich/connectors/) |
| **Internal import file** | Reads the files you upload, such as a PDF report or a STIX bundle. See [Import from files](/opencti/use/ingest/files/) |
| **Internal export file** | Writes the exports you generate, such as a PDF or a STIX file |
| **Internal analysis** | Reads the content of a container or a file and suggests the entities it mentions |
| **Stream** | Listens to the platform's changes and sends them to another tool, such as a SIEM |
| **Internal ingestion** | Built into the platform, such as **[DRAFT] Draft validation**, which writes approved drafts |

This page is mostly about external import connectors, but every connector has the same screens.

Each connector writes as one account, usually a service account. That account's confidence level, markings and organizations apply to everything the connector creates: a connector cannot create what its account may not, and cannot overwrite knowledge of a higher confidence. Its objects show that account as **Creator**. See [Reliability and confidence](/opencti/concepts/reliability-confidence/).

## Choose a connector in the catalog

::::only{opencti=">=7.260811"}
:::steps

1. Open :menu[Integrations] and select the **Available** tab.
2. Search by name or description, or narrow the list with the filters on the left: **Kind**, **Connector type**, **Use cases**, **Solution categories**, **License type** and **Status**.
3. Check who maintains it in **Status**. **Supported by Filigran** marks the built-in feeds and the connectors Filigran verifies. **Supported by Community** marks connectors that the community contributes and maintains.
4. Click the connector to open its page. **Basic information** gives the link under **Integration documentation and code**, a **Vendor contact**, the **Latest Compatible Version** and the date it was **Last verified**.

:::

::screenshot{id=opencti-ingest-connectors-catalog alt="The Available tab lists connector cards with their type, their support origin and a Deploy button, and the filters Kind, Connector type, Use cases and Status on the left."}

:::only{opencti=">=7.261030"}
The **Deployment** filter separates the connectors XTM Composer can deploy, **Managed**, from those an administrator runs by hand, **Manual**. Only managed connectors show **Deploy**.
:::

**Sort by** orders the catalog by **Name (A-Z)**, **Most deployed** or **Verified first**. The view buttons switch between cards and lines. A count next to **Deploy** shows how many instances XTM Composer already runs; connectors started by hand are not counted.
::::

::::only{opencti="<7.260811"}
:::steps

1. Open :menu[Data > Ingestion > Connector catalog].
2. Search by name or description, or filter by connector type or use case.
3. Click **Details** on a card to read what the connector does and how to configure it.

:::

A badge on **Deploy** shows that the connector is already deployed.
::::

::::only{opencti=">=7.260811"}
The **Built-in ingestion** cards at the top of the **Available** tab are feeds, not connectors: the platform runs them itself, with nothing to deploy. Click **Create** on a card to open its form; for a CSV or JSON feed, **Verify** then **Test** show what the feed would import before you create it. Each form is described on its own page: [Import a CSV file from a URL](/opencti/use/ingest/csv-feed/), [Import from a JSON API](/opencti/use/ingest/json-feed/), [Import from a TAXII server](/opencti/use/ingest/taxii-feed/), [Turn RSS articles into reports](/opencti/use/ingest/rss-feed/) and [OpenCTI streams](/opencti/use/ingest/streams/).
::::

A connector that XTM Composer cannot deploy can still run: an administrator starts it beside the platform, as [Run an integration manually](/deploy/integrations/manual/) describes. It appears with the others once it registers.

## Deploy a connector

:::steps

1. On the connector's card or page, click **Deploy**. For a community connector, clicking **Create** at the end of the form opens **Deploy a Community Connector**, which asks you to acknowledge that Filigran does not support it.
2. In **Deploy a new connector**, enter a **Display name** that says what the connector brings, such as the source and its scope. The **Instance name** follows from it.
3. Choose the account the connector writes as. Keep **Automatically create a service account** to get a new account named after the connector, with the prefix `[C]`. Its confidence level is the one the catalog recommends for this connector; turn on **Edit confidence level** to change it. Otherwise, choose an existing account in **Service account responsible for data creation**.
4. Fill in the required options under **Configuration**, such as an API key. Open **Advanced options** for the others, such as how often the connector runs or the date from which it imports.
5. Click **Create**. The connector is created stopped.
6. Open it from the **Deployed** tab and click **Start**.

:::

Automatic service accounts need a default group. If the form says that no default group has been defined, ask an administrator to set **Default group for Service accounts**. See [Create accounts for integrations](/opencti/administer/service-accounts/).

A connector that is not compatible with your platform version cannot be deployed: the tooltip of **Deploy** gives the version it needs. The full deployment path, including XTM Composer, is in [Deploy a first integration with XTM Composer](/deploy/integrations/quick-start/).

## Follow what connectors import

::::only{opencti=">=7.260811"}
Open :menu[Integrations] and stay on the **Deployed** tab. The strip at the top sums up the platform: **Deployed integrations**, **Queued bundles** waiting for a worker, **Connected workers**, **Bundles processed** per second, **Operations** and **Total documents**. Below, each connector and feed shows its status, its last run or last heartbeat, and the messages waiting in its queue.

::screenshot{id=opencti-ingest-connectors-deployed alt="The Deployed tab of Integrations shows the platform figures at the top, then the deployed connectors and feeds grouped by type, each with its queued messages, its last activity and its status."}

- Filter with **Kind** (connectors or built-in feeds), **Type** and **Status**.
- Sort by **Name (A-Z)**, **Status**, **Last run** or **Queued messages**, to find the connectors with the largest backlog.

:::only{opencti=">=7.261030"}
The lines view adds **Throughput**, the bundles processed per second, and **Last activity**.
:::

:::only{opencti=">=7.261117"}
A connector is **Started** unless someone stopped it: only a connector deployed with XTM Composer can show **Stopped**. A connector that stops reporting stays **Started**, and its health says that it no longer answers: see [Spot an integration that stops working](#spot-an-integration-that-stops-working). **Processing** means that a start or a stop is under way.
:::

:::only{opencti="<7.261030"}
A connector is **Active** when it has reported in the last 5 minutes, **Inactive** otherwise. **Processing** means that a start or a stop is under way.
:::
::::

::::only{opencti="<7.260811"}
Open :menu[Data > Ingestion > Connectors]. Each connector shows its status, when it was last seen and the messages waiting in its queue.
::::

A large number of queued messages means the workers cannot keep up. The connector itself is fine: the data will arrive.

::::only{opencti=">=7.261117"}
## Spot an integration that stops working

Every minute, the platform checks the health of each connector, feed and synchronizer that runs: the account it writes as, the heartbeat and the errors of a connector, and the last run of a feed or a synchronizer. You see at a glance which sources stopped delivering, and why, without opening each one. A stopped integration is not checked.

A problem makes the integration **Critical** when it no longer imports, or **Degraded** when something needs fixing.

| Message | Health | What it means |
| --- | --- | --- |
| "No heartbeat received" | **Critical** | A connector that runs continuously has not reported for 5 minutes: its process is down. A connector that only runs on a schedule and exits is not checked |
| "Connector reported a critical error" | **Critical** | The connector reported an error that blocks it, followed by the error's code and message |
| "Connector reported an error" | **Degraded** | The connector reported a lesser error: it may still import |
| "Feed execution failed" or "Sync execution failed" | **Critical** | The last run of a feed or a synchronizer failed, for example because the source was unreachable. The message adds the HTTP status or the error. Nothing is imported until a run succeeds |
| "Feed or sync source denied access" | **Critical** | The source refused the credentials of the feed or the synchronizer, with HTTP 401, 402 or 403: update them |
| "User is missing" | **Degraded** | The account the integration writes as no longer exists |
| "User lacks required capability" | **Degraded** | The account of a connector lacks **Connectors API usage: register, ping, export push ...** |
| "User account is not active" | **Degraded** | The account is expired, locked or disabled |
| "User is not a service account" | **Degraded** | The integration writes as a person's account. A service account is recommended: it does not leave with a person |

The account problems show as **Degraded** even when they stop the integration: a connector whose account is missing, inactive or lacks the capability cannot register with the platform. Fix them first.

The health shows in three places:

- At the top of the **Deployed** tab, one alert for the integrations that are not working and one for those that are degraded, such as "1 integration is not working", with the names of up to three of them. **Review** opens the integration when there is only one; **See all** narrows the list to them.
- On each integration: a strip at the top of its card gives the first problem and leads to **Review**. In the lines view, the **Health status** column shows **Critical** or **Degraded**; hover over it to see since when, and every problem. The **Health status** filter on the left keeps the integrations in either state.
- On the page of the connector, the feed or the synchronizer: a banner, such as "This connector is not working since" and the date, lists its problems. When there are several, the first one shows and a button unfolds the others.

::screenshot{id=opencti-ingest-connectors-health alt="On the page of the Hospitality threat feed connector, a red banner says the connector is not working, with No heartbeat received, and an orange one says it is degraded, with User is not a service account."}

When a health stays critical, degraded or back to normal for the notification delay, 5 minutes by default and editable in the ingestion health settings, the platform records an `ingestion_health` event in the activity log. An administrator can receive these events by email through an [activity trigger](/opencti/configure/activity/triggers/) that uses the **Ingestion health default mailer** notifier. The [ingestion health manager](/opencti/concepts/architecture/#the-ingestion-health-manager) explains how the checks run.

:::example[Find why the hospitality feed went quiet]
On Monday morning, your analysts notice that no new report on the hospitality sector arrived over the weekend.

1. In :menu[Integrations], tab **Deployed**, an alert reads "1 integration is not working": **Hospitality threat feed**, with "No heartbeat received".
2. Click **Review**. The connector page shows "This connector is not working since" Saturday's date, and a second banner says that it is degraded: "User is not a service account".
3. The connector's process is down: ask the administrator who runs it to restart it. Ask them also to give the connector a service account. See [Create accounts for integrations](/opencti/administer/service-accounts/).
4. Once the connector reports again, the red banner disappears at the next check, within a minute. Open **Works**: the next run starts from the connector's state, the date of its last import.

The team knows that the feed works again, and the next outage shows on the **Deployed** tab within minutes, not after a quiet weekend.
:::
::::

## Read a connector's page

Click a connector to open its page. It answers three questions: who it writes as, where it stands, and what each import did.

### Who it writes as and what it handles

**Basic information** describes the connector:

- **Type**, and **Last seen**, the time of its last heartbeat.
- **Scope**: the types of data it handles, such as `IPv4-Addr` for an enrichment connector or `text/csv` for a file import connector.
- **Automatic trigger** and **Only contextual**, for enrichment and file import connectors: whether it runs on its own, and whether it only processes data related to the entity it runs on. Other types show **Not applicable**.
- **Trigger filters**, for enrichment connectors: the entities it enriches automatically. See [Enrich with connectors](/opencti/use/enrich/connectors/).
- **Associated user**, the account the connector writes as, with its **Max confidence level**, **User's roles**, **User's groups** and **User's organizations**. You need **Manage credentials** to see them.

A connector deployed with XTM Composer also shows **About this connector**: its use cases, and links to **View in catalog**, **Source code** and **Vendor contact**.

### Where it stands: its state

A connector keeps a **State**: a small record it saves after each run, such as the date of the last object it imported. At its next run, it reads the state and fetches only what is newer. **Details** shows the state and what follows from it:

- **Last run**, read from the state when the connector saves it there, as **Last run (from State)**.
- **Next run**, or **External schedule** when another tool starts the connector.
- **Server capacity**: the size of the messages waiting in the connector's queue, against the limit it accepts. Above the limit, the connector shows **Buffering** and waits for the workers before it fetches more.
- For a connector deployed with XTM Composer: its **Instance name** and, while it runs, its **Uptime**.

A value the connector does not report shows **Not provided**.

:::only{opencti=">=7.261030"}
**Version** gives the version of the connector that runs.
:::

::screenshot{id=opencti-ingest-connectors-overview alt="The Overview of an external import connector shows its type, scope and associated user, then its state, last run, next run and server capacity."}

### What each import did: its works

Each run, or each file processed, is a work.

:::only{opencti=">=7.260811"}
The **Works** tab lists the **In progress works** and the **Completed works**.
:::

:::only{opencti="<7.260811"}
Below the details, the page lists the **In progress works** and the **Completed works**.
:::

Each work shows its **Status**, its **Work start time** and **Work end time**, the **Operations completed** out of the **Total number of operations**, and its **Progress**. The button with the number of errors opens them, sorted in tabs:

- **Critical**: the object was not written, for example because of a database error or two objects claiming the same identifier.
- **Warning**: the object was refused for a reason you can act on, for example an indicator with a wrong pattern, a missing reference, or a confidence level too low to change an existing object.
- **All**: every error, including those the platform does not classify.

Each line gives the **Timestamp**, the **Code**, the **Message** and the **Source** object; **Details** shows the object and, for a relationship, its **From** and **To**. Only the first 100 errors of a work are listed. The [error reference](/reference/opencti/errors/) explains the messages.

::screenshot{id=opencti-ingest-connectors-works alt="The Works tab of a connector lists a completed work with its start and end times, its operations completed out of the total, a full progress bar, and a button counting its errors."}

A work that never ends usually means the connector stopped in the middle of a run. Many warnings about confidence mean the connector's account has a lower confidence level than the knowledge it tries to update: ask an administrator whether to raise it.

### Logs

A connector deployed with XTM Composer also has a **Logs** tab with its recent output: the last 100 lines of its container. Other connectors write their logs where they run: ask the administrator who runs them.

XTM Composer adds its own lines, which start with `[XTM Composer]`, so that a crash stays visible after the connector restarts:

- When the container restarted, or ended with an error, a line at the end says how the previous run ended, such as `[XTM Composer] The previous container ended: reason OOMKilled, exit code 137, at 2026-01-12T03:14:08Z`. `OOMKilled` means the connector ran out of memory: ask the administrator to give it more. On Docker, Podman and Portainer, the line shows only while the container is not running; on Docker Swarm, never.
- On Kubernetes, when the pod was replaced, for example after you update its options, the tab first shows up to 50 lines of the previous pod, under `[XTM Composer] Last lines of the previous pod:`, then those of the current one, under `[XTM Composer] Logs of the current pod:`. XTM Composer keeps these lines in memory: after it restarts, the tab shows the current pod only. When the container restarted inside its pod, the lines come from its previous run, under `container`. On Docker and Podman, a container XTM Composer recreates starts without them; Docker Swarm and Portainer never show them.

## Act on a connector

Open the connector page. A connector deployed with XTM Composer has a **Start** or **Stop** button; the other actions are in the actions menu (⋮).

| Action | What it does | Available for |
| --- | --- | --- |
| **Start**, **Stop** | Starts or stops the connector | Connectors deployed with XTM Composer |
| **Update** | Opens **Update a connector**, to change its **Display name** and its options. The change takes a few minutes to apply | Connectors deployed with XTM Composer |
| **Reset the connector state** | Forgets the state, so the connector imports again from its start date, and purges its queue. The confirmation shows the number of messages that will be lost | All connectors |
| **Clear all works** | Removes the history of imports from the works. The imported data stays | All connectors |
| **Delete** | Removes the connector from the platform. A connector deployed with XTM Composer must be stopped first; one run by hand, once it no longer reports | All but built-in connectors |

**Delete** on a single work removes that work from the list, not its data.

Reset a connector when you want to replay its source, for example after you change its start date, or when its queue holds outdated messages you do not want to process. Purging the queue avoids importing data twice. When your platform protects sensitive changes, the action shows as **Reset** with a danger zone mark, and only the people allowed can use it. See [Sensitive configuration](/opencti/administer/sensitive-configuration/).

When the options of a connector change in a new version, its page warns that deprecated configuration fields are still set. Click **Edit configuration** to review and remove them.

:::example[Keep MITRE ATT&CK current for the Velvet Harbor analysis]
Your analysts map the techniques of Velvet Harbor, such as T1566 Phishing, to MITRE ATT&CK, so the platform needs the ATT&CK matrices and their updates. Your platform runs XTM Composer.

1. In :menu[Integrations], tab **Available**, search `MITRE` and open **MITRE ATT&CK**. It is an **External import** connector, **Supported by Filigran**.
2. Click **Deploy**. Set **Display name** to `MITRE ATT&CK` and keep **Automatically create a service account** with the confidence level the form proposes, 80. The connector will write as `[C] MITRE ATT&CK`.
3. Open **Advanced options**. Keep the polling interval of 7 days, and turn on the option that removes the statement markings MITRE puts on its data, so that they do not show on every technique. Click **Create**.
4. In the **Deployed** tab, open **MITRE ATT&CK** and click **Start**. Within a few minutes, it shows **Started** and a recent **Last seen**.
5. Open **Works**: the first work imports the matrices with thousands of operations, and the techniques appear in :menu[Techniques > Attack patterns]. In **Overview**, **Next run** shows the next check, seven days later.
6. A week later, an analyst reports that a technique is missing. **Works** shows that the first run never completed: the platform restarted during the import. Choose **Reset the connector state** in the actions menu: the connector forgets its last run and imports the matrices again at its next check.

The team now finds every ATT&CK technique to link to Velvet Harbor, and the matrices stay current without anyone asking.
:::

## Check that it works

A connector that runs shows a recent **Last seen** and new lines in its works, with few errors.

:::only{opencti=">=7.261117"}
Its card shows no health strip, and its page no health banner.
:::

The objects it imports carry its account as **Creator**: filter any list by **Creator** to see them.

## Settings an administrator controls

Two platform settings shape how connectors run. An administrator sets them in the platform configuration.

| Setting | Effect | Default |
| --- | --- | --- |
| `app:composer_token_expiration` (`APP__COMPOSER_TOKEN_EXPIRATION`) | How long, in seconds, the API token of a connector deployed with XTM Composer stays valid between two reads of the connector's configuration by XTM Composer. Each read renews it. At the first read after it lapsed, the platform revokes the token, named **Composer managed token** on the connector's account, and issues a new one | `604800` (7 days) |
| `app:task_scheduler:max_queues_breakdown` (`APP__TASK_SCHEDULER__MAX_QUEUES_BREAKDOWN`) | Number of queues that background tasks, such as bulk edits from a list, are spread over for the workers | `4` |

The other settings of the queues and workers are in the [engines, schedules and managers settings](/reference/opencti/configuration/#engines-schedules-and-managers), and those of XTM Composer in the [XTM Suite settings](/reference/opencti/configuration/#xtm-suite).

## What's next

- [Enrich with connectors](/opencti/use/enrich/connectors/), for connectors that add context to existing entities.
- [Troubleshoot integrations](/deploy/integrations/troubleshoot/), when a connector stops reporting.
- [How integrations run](/deploy/integrations/how-they-run/), for the platform side of connectors.
- [How intelligence gets into OpenCTI](/opencti/use/ingest/), for the built-in feeds that need no connector.
