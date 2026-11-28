---
title: How intelligence gets into OpenCTI
description: The ways intelligence gets into OpenCTI, how connectors and built-in feeds turn it into STIX bundles that workers write, and where you manage them.
sidebar_position: 2
doc_type: explanation
products:
  - opencti
personas: [cti-lead, cti-analyst, admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/import/getting-started/
---

OpenCTI fills its knowledge base in two ways: automatically, from connectors and feeds that fetch or receive intelligence on their own, and by hand, from files, forms and entities that analysts create. As a team lead, you choose which sources feed your team, which of them a person reviews first, and under which account each one writes. This page explains the automated ways: what each one is for, how its data reaches the knowledge base, and the screens and rights you need to run it. The pages under it give the steps for each method.

## Ways data gets in

| Method | Use it when | Page |
| --- | --- | --- |
| Connector | A vendor, an open source feed or a tool has a dedicated integration in the catalog | [Import with connectors](/opencti/use/ingest/connectors/) |
| OpenCTI stream | Another OpenCTI instance shares a live stream with you | [Import from another OpenCTI instance](/opencti/use/ingest/streams/) |
| TAXII feed | A Trusted Automated Exchange of Intelligence Information (TAXII) 2.1 server publishes a collection you can poll | [Import from a TAXII server](/opencti/use/ingest/taxii-feed/) |
| TAXII Push | A partner or a tool sends STIX 2.1 objects to you instead of you fetching them | [Let partners push intelligence to you](/opencti/use/ingest/taxii-push/) |
| RSS feed | A blog or a news site publishes articles you want as reports | [Turn RSS articles into reports](/opencti/use/ingest/rss-feed/) |
| CSV feed | A URL serves a CSV file, such as a list of indicators | [Import a CSV file from a URL](/opencti/use/ingest/csv-feed/) |
| JSON feed | A web API answers in JSON, with or without pagination | [Import from a JSON API](/opencti/use/ingest/json-feed/) |
| File | You have a document, a STIX bundle or a spreadsheet to import once | [Import from files](/opencti/use/ingest/files/) |
| Form | Analysts submit the same kind of knowledge often, in a guided way | [Create knowledge with forms](/opencti/use/ingest/forms/) |
| Manual creation | You record one entity or relationship | [Create knowledge manually](/opencti/use/ingest/manual/) |

Connectors are programs that run beside the platform. The other automated methods are built into OpenCTI: you create them from the interface, with no extra component to deploy.

## Decide what a person reviews first

Not every source deserves the same trust. Decide, for each one, whether its data reaches the knowledge base at once or waits for an analyst:

- **Files and forms can wait in a draft.** An analyst reviews what a connector found in a vendor report, or what a partner submitted, then approves it. See [Prepare changes in a draft](/opencti/use/ingest/drafts/).
- **Feeds write straight to the knowledge base.** For them, the safeguard is the account they write as: its confidence level decides what they may overwrite, its markings what they may create. Give a source you trust less a lower confidence level, so that it cannot change what your analysts wrote.
- **A playbook can triage what comes in**, for example label every new report on the hospitality sector so that the team reviews it. See [Automate with playbooks](/opencti/use/automate/playbooks/).

## How automated import works

Every automated method converts what it gets into a STIX 2.1 bundle, a package of Structured Threat Information Expression (STIX) objects, and puts it in a queue. Workers take the bundles from the queue and write them into the knowledge base.

::diagram{id=opencti-ingest-ways}

These rules explain most of what you see:

1. **Data appears after a delay.** A bundle waits in its queue until a worker is free. On a busy platform, data can take a while between the fetch and its display.
2. **Data is written as a user.** Each connector and feed writes as one account: its service account, or the **User responsible for data creation**. That account's confidence level, markings and capabilities decide what the import can create or update, and it appears as the creator of the data.
3. **Built-in feeds poll on a cycle.** The ingestion manager checks the running feeds every 30 seconds by default. A feed fetches when its **Schedule period** has passed, and RSS and CSV feeds wait at least 5 minutes between two fetches.
4. **A busy feed waits for its queue.** While its queue still holds bundles, a feed does not fetch new data. Its status shows that it is buffering.
5. **A feed remembers where it stopped.** Each feed keeps a state, such as the date of the last item or a pagination cursor, so the next run fetches only new data. **Reset state** starts again from the beginning.
6. **A new feed is stopped.** Nothing is fetched or accepted until you start it.

OpenCTI streams follow the same path, but through the sync manager, which listens to the remote stream continuously instead of polling. TAXII Push has no schedule: it accepts objects when a client sends them. The managers' settings, such as the polling interval, are in [Tune feeds and streams](/opencti/use/ingest/feed-options/).

## Where you manage imports

::::only{opencti=">=7.260811"}
Connectors and feeds share one screen, :menu[Integrations], with two tabs:

- **Deployed** lists what runs on your platform: the connectors that registered, the feeds you created and the forms, each with its status, its last run and the messages waiting in its queue.
- **Available** is the catalog: the connectors of the [XTM Hub integrations library](https://hub.filigran.io/cybersecurity-solutions/open-cti-integrations), and the **Built-in ingestion** cards (**OpenCTI Stream**, **TAXII Feed**, **TAXII Push**, **RSS Feed**, **CSV Feed**, **JSON Feed** and **Form intake**). A built-in card has a **Create** button that opens the creation form.

::screenshot{id=opencti-ingest-integrations-available alt="The Available tab of Integrations shows the Built-in ingestion cards, each with a Create button, above the connector cards of the catalog."}

Click a feed in the **Deployed** tab to open its page. It shows the feed's configuration and its activity: the bundles in queue, the last and next run, and the current state, then each import with its errors.

:::only{opencti=">=7.261030"}
The page has three tabs: **Overview**, **Works**, which lists each import with its errors, and **Logs**, which lists the recent fetches and their outcome. An OpenCTI stream has no **Works** tab.
:::
::::

::::only{opencti="<7.260811"}
Connectors and feeds are in :menu[Data > Ingestion], one tab per kind: **Connector catalog**, **Connectors**, **OpenCTI Streams**, **TAXII Feeds**, **TAXII push**, **RSS Feeds**, **CSV Feeds** and **JSON Feeds**. Each feed tab lists the feeds of that kind and has a button to create one.
::::

### Manage a feed

Open the actions menu (⋮) of a feed to act on it. From version 7.260811, use the menu on the feed's own page: the menu of a feed in the **Deployed** list only offers **Start** or **Stop**, **Update** and **Delete**. The actions depend on the kind of feed:

| Action | What it does |
| --- | --- |
| **Start**, **Stop** | Starts or pauses fetching. A stopped TAXII Push collection refuses pushes |
| **Update** | Changes the configuration. A change of URL is checked against the [URI deny list](/opencti/use/ingest/feed-options/#uri-deny-list) |
| **Reset state** | Forgets the state, so the next run imports again from the beginning, or from the import start date you set. RSS, TAXII, CSV and JSON feeds |
| **Duplicate** | Opens the creation form filled with this feed's configuration. CSV and JSON feeds only |
| **Export** | Downloads the configuration as a JSON file, without credentials. A CSV or JSON feed carries its mapper in the file |
| **Delete** | Deletes the feed. The data it imported stays |

:::only{opencti=">=7.261127"}
**Stop** also interrupts a run in progress of a CSV feed, or of a JSON feed with **Sub pagination** on, before its next page. A CSV feed sends no more bundles: those it already sent are still written. A JSON feed sends nothing from that run. In both cases the state stays where it was, so the next run after **Start** fetches the same data again.
:::

### Share feed configurations

A feed exported as JSON can be imported on another platform. Credentials are not exported, so you enter them again.

::::only{opencti=">=7.260811"}
To import a configuration file, use the import button of the built-in card in the **Available** tab and choose the file. The creation form opens, filled with the configuration: check it, then create the feed.
::::

::::only{opencti="<7.260811"}
To import a configuration file, use the import button next to the create button of the feed tab and choose the file. The creation form opens, filled with the configuration: check it, then create the feed.
::::

The XTM Hub shares ready-made configurations for OpenCTI streams, TAXII feeds, RSS feeds and CSV feeds. **Import from Hub** opens the XTM Hub library when your platform can reach it. If your platform is [registered with the XTM Hub](/opencti/configure/xtm-hub/), **Deploy in OpenCTI** on the XTM Hub opens the filled creation form directly on your platform.

## Rights you need

| To | Capability |
| --- | --- |
| See the feeds | **Access ingestion** |
| Create, change, start and delete feeds | **Manage ingestion** |
| See the connectors | **Access connectors** |
| Deploy a connector from the catalog | **Manage ingestion** and **Connectors API usage: register, ping, export push ...** |
| Reset a connector, clear its works, start, stop or delete it | **Manage connector state** |
| Create the CSV and JSON mappers that feeds use | **Manage data mappers** |

The [capability reference](/reference/opencti/capabilities/) lists these capabilities with the rest of each tree.

Deploying a connector from the catalog also needs an Enterprise Edition license and XTM Composer: see [Deploy integrations with XTM Composer](/deploy/integrations/composer/).

## One account and one author per source

Give each source its own account and its own author. You can then tell where each piece of data comes from, filter and build dashboards by source, and change the rights of one source without touching the others.

- **A service account per source.** Most creation forms create one for you with **Automatically create a service account**: it is named after the feed, with the prefix `[F]` for a feed or `[S]` for a stream, and you set its confidence level. This needs a default group: an administrator sets **Default group for Service accounts** in :menu[Settings > Security > Policies]. See [Create accounts for integrations](/opencti/administer/service-accounts/).
- **An organization per source.** Create an organization named after the source and set it as **Default author** where the form offers it.

::::only{opencti=">=7.260811"}
:::xtm-one[XTM One and imports]
For documents you import by hand, the Import Document AI connector can hand each file to an XTM One AI agent, which extracts its entities, observables and relationships. See [Import from files](/opencti/use/ingest/files/). To process what feeds bring in, a [playbook](/opencti/use/automate/playbooks/) can send new data to an AI agent.
:::
::::

## What's next

- [Import with connectors](/opencti/use/ingest/connectors/), from the catalog to monitoring.
- [Import a CSV file from a URL](/opencti/use/ingest/csv-feed/) and [Import from a JSON API](/opencti/use/ingest/json-feed/), with the [CSV mappers](/opencti/configure/mappers/csv/) and [JSON mappers](/opencti/configure/mappers/json/) they rely on.
- [Tune feeds and streams](/opencti/use/ingest/feed-options/), for the polling interval, the URI deny list and other platform settings.
