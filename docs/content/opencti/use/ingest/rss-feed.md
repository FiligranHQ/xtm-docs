---
title: Turn RSS articles into reports
description: Turn the articles of an RSS or Atom feed into OpenCTI reports, with the author, markings and report types you choose, fetched on a schedule.
sidebar_position: 5
doc_type: how-to
products:
  - opencti
personas: [cti-lead, cti-analyst]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/import/rss-feed/
---

An RSS feed watches an RSS 2.0 or Atom feed, such as a security blog or a vendor's advisories, and creates one report for each new article. Analysts then read the reports, extract the knowledge they contain and link it. For how imported data reaches the knowledge base, read [How intelligence gets into OpenCTI](/opencti/use/ingest/).

## Before you start

- Your role has the **Manage ingestion** capability.
- You have the URL of the feed.
- An organization exists for the source, to use as the author of the reports. See [One account and one author per source](/opencti/use/ingest/#one-account-and-one-author-per-source).

## What a report contains

Each article becomes a report:

| Report | Comes from |
| --- | --- |
| Name | The title of the article, or its link when it has no title |
| Description | The summary of the article, converted to Markdown |
| Publication date | The date of the article |
| Labels | The categories of the article (RSS 2.0 only) |
| External reference | The link to the article |
| Author, markings, report types | The defaults of the feed |

Only articles published after the last run are imported, oldest first. An article without a valid date is skipped.

## Create the feed

Open the creation form:

::::only{opencti=">=7.260811"}
Open :menu[Integrations], select the **Available** tab, and click **Create** on the **RSS Feed** card.
::::

::::only{opencti="<7.260811"}
Open :menu[Data > Ingestion > RSS Feeds] and click the create button.
::::

Then fill it in:

:::steps

1. Enter a **Name** and a **Description**.
2. Choose a **Schedule period**. Whatever you choose, an RSS feed is fetched at most every 5 minutes.
3. Enter the **RSS Feed URL**.
4. Keep **Automatically create a service account** on and set its confidence level, or turn it off and choose an existing account.
5. Set the import start date to the oldest article to import. Leave it empty to import every article the feed holds.
6. Choose the **Default report types**, such as `threat-report`.
7. Choose the organization of the source as **Default author**.
8. Choose the **Default marking definitions**, such as `TLP:CLEAR` for a public blog.
9. Keep **Verify SSL certificate** on, unless the site uses a certificate your platform cannot check.
10. Click **Create**, then **Start** the feed.

:::

:::example[Follow the advisories of a security blog]
Harbor Security, a fictional vendor that tracks threats to hotels, publishes its advisories on a public blog with an RSS 2.0 feed. Your team wants each new advisory as a report, ready to be read and linked.

- **Name**: `Harbor Security Blog`
- **Description**: `Advisories of a fictional security blog`
- **Schedule period**: **1 hour**
- **RSS Feed URL**: `https://blog.harbor-security.example/feed.xml`
- **Automatically create a service account**: on
- **Import from date (empty = all RSS feed possible items)**: `2026-01-15 08:00 am`, the day you subscribe, so the archive of the blog is left out
- **Default report types**: `threat-report`
- **Default author**: `Harbor Security`, the organization of the vendor
- **Default marking definitions**: `TLP:CLEAR`, as the blog is public
- **Verify SSL certificate**: on

An article titled "Velvet Harbor targets hotel booking systems", published on 16 January 2026, becomes a report of that name, of type `threat-report`, authored by Harbor Security, marked `TLP:CLEAR`, with the link to the article as external reference.
:::

## Check that it works

::::only{opencti=">=7.260811"}
Open the feed from the **Deployed** tab of :menu[Integrations]. After the first run, its page shows the last run and the date of the latest article imported.

:::only{opencti=">=7.261030"}
The page has an **Overview**, a **Works** tab with each import, and a **Logs** tab with each fetch.
:::

::screenshot{id=opencti-ingest-rss-feed-overview alt="The page of an RSS feed: its configuration with the feed URL, the report types and the service account, and its activity with the last run, the next run and the current state."}
::::

::::only{opencti="<7.260811"}
In :menu[Data > Ingestion > RSS Feeds], the feed shows as running with its last run date.
::::

The reports appear in :menu[Analyses > Reports], with the feed's author.

Some sites reject requests that do not come from a browser. If every fetch fails while the URL opens in your browser, an administrator can change the user agent the platform sends: see [Tune feeds and streams](/opencti/use/ingest/feed-options/#rss-feeds).

## What's next

- [Manage a feed](/opencti/use/ingest/#manage-a-feed): reset, export or share the feed on the XTM Hub.
- [Analyses](/opencti/use/explore/analysis/), to work on the imported reports.
- [Automate with playbooks](/opencti/use/automate/playbooks/), to process each new report automatically.
