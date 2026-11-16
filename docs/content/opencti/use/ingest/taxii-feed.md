---
title: Import from a TAXII server
description: Poll a TAXII 2.1 collection, from an information sharing community or another OpenCTI platform, and import its STIX objects on a schedule.
sidebar_position: 3
doc_type: how-to
products:
  - opencti
personas: [cti-lead, cti-analyst, admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/import/taxii-feed/
---

A TAXII feed polls a collection on a Trusted Automated Exchange of Intelligence Information (TAXII) 2.1 server and imports the STIX objects published since its last run. Use it for the collections of an Information Sharing and Analysis Center (ISAC), a vendor or another OpenCTI platform. For how imported data reaches the knowledge base, read [How intelligence gets into OpenCTI](/opencti/use/ingest/).

## Before you start

- Your role has the **Manage ingestion** capability.
- You have the API root URL of the TAXII server, the ID of the collection, and the credentials it requires.
- To let the form create a service account, an administrator has set **Default group for Service accounts** in :menu[Settings > Security > Policies].

### Find the API root URL

OpenCTI needs the API root, not the discovery URL or the collections URL. Many providers give you one of those: remove the last part of the path.

| You were given | Enter |
| --- | --- |
| `https://tip.example/tipapi/tip21/collections` | `https://tip.example/tipapi/tip21` |
| A collection of another OpenCTI platform | `https://<platform>/taxii2/root`, with the collection ID shown in its **TAXII collections** list |

## Create the feed

Open the creation form:

::::only{opencti=">=7.260811"}
Open :menu[Integrations], select the **Available** tab, and click **Create** on the **TAXII Feed** card.
::::

::::only{opencti="<7.260811"}
Open :menu[Data > Ingestion > TAXII Feeds] and click the create button.
::::

Then fill it in:

:::steps

1. Enter a **Name** and a **Description** that say where the data comes from.
2. Choose a **Schedule period**. **Platform default** fetches about every 30 seconds; a longer period suits a collection that changes a few times a day.
3. Enter the **TAXII server URL**: the API root.
4. Keep **TAXII version** on **TAXII 2.1**, the only version supported.
5. Enter the collection ID in **TAXII Collection**.
6. Choose the **Authentication type** and enter the credentials: **Basic user / password**, **Bearer token** or **Client certificate**, or **None** for a public collection. For a collection of another OpenCTI platform, use **Bearer token** with the token of a user who can read the collection.
7. Keep **Automatically create a service account** on and set its confidence level, or turn it off and choose an existing account.
8. Set the import start date to the oldest object to fetch. Leave it empty to import the whole collection.
9. Turn on **Copy confidence level to OpenCTI scores for indicators** to set the score of each imported indicator from its STIX confidence.
10. Keep **Verify SSL certificate** on, unless the server uses a certificate your platform cannot check.
11. Click **Create**, then **Start** the feed.

:::

::screenshot{id=opencti-ingest-taxii-feed-create alt="The Create a TAXII ingester form lists the schedule period, the TAXII server URL, the TAXII version, the collection and the authentication type, followed by the service account options."}

Credentials are stored encrypted, and the update form does not show them again.

:::example[Poll the hotel indicators of a sharing community]
Your organization joined a fictional hospitality sharing community. Its welcome email gives the discovery URL `https://taxii.example.com/taxii2/`, the API root `https://taxii.example.com/hospitality/`, the collection "Hotel indicators" with the ID `8f3c2a71-5d4e-4b9a-9c61-2e7f0d1b4a53`, and a token for your member account.

- **Name**: `Hospitality ISAC - Hotel indicators`
- **Description**: `Indicators shared by the members of the hospitality sharing community`
- **Schedule period**: **1 hour**, as members share a few times a day
- **TAXII server URL**: `https://taxii.example.com/hospitality/`, the API root and not the discovery URL
- **TAXII version**: **TAXII 2.1**
- **TAXII Collection**: `8f3c2a71-5d4e-4b9a-9c61-2e7f0d1b4a53`
- **Authentication type**: **Bearer token**, with the member token in **Token**
- **Automatically create a service account**: on, with a confidence level of 60, below that of your own analysts, so the feed never overwrites their corrections
- **Import from date (empty = all TAXII collection possible items)**: `2026-01-01 12:00 am`, to skip the years of history the collection holds
- **Copy confidence level to OpenCTI scores for indicators**: on, as members set a confidence on each indicator

After the first run, the indicators shared since January 2026, such as `booking-velvetharbor.example`, appear in :menu[Observations > Indicators], created by the service account of the feed.

The community's server times out when asked for a whole month of indicators at once. Your platform engineer limits each poll to 500 objects in the environment of the platform, for every TAXII feed, and the feed then pages through the collection:

```yaml
services:
  opencti:
    environment:
      - INGESTION_MANAGER__TAXII_FEED__LIMIT_PER_REQUEST=500
```
:::

## Check that it works

::::only{opencti=">=7.261030"}
Open the feed from the **Deployed** tab of :menu[Integrations]. After the first run, **Overview** shows the last run and a state cursor, **Works** shows the bundles imported, and **Logs** shows each fetch with the number of objects received.
::::

::::only{opencti=">=7.260811 <7.261030"}
Open the feed from the **Deployed** tab of :menu[Integrations]. After the first run, **Overview** shows the last run and a state cursor, and **Works** shows the bundles imported.
::::

::::only{opencti="<7.260811"}
In :menu[Data > Ingestion > TAXII Feeds], the feed shows as running with its last run date. Follow its imports in :menu[Data > Ingestion > Connectors].
::::

A run that receives nothing is normal when the collection has no new object. If every fetch fails, check the API root URL first: a discovery or collections URL returns an error.

To limit the number of objects requested per poll on a large collection, an administrator sets `ingestion_manager:taxii_feed:limit_per_request`: see [Tune feeds and streams](/opencti/use/ingest/feed-options/).

## What's next

- [Manage a feed](/opencti/use/ingest/#manage-a-feed): stop, reset, export or share the configuration on the XTM Hub.
- [Let partners push intelligence to you](/opencti/use/ingest/taxii-push/), to let a partner send objects instead of polling them.
- [Share with feeds and streams](/opencti/use/share/feeds/), to publish your own TAXII collections.
