---
title: Import from a JSON API
description: Query a JSON web API on a schedule and map its answers to OpenCTI entities with a JSON mapper, including APIs that paginate with offsets, cursors or next links.
sidebar_position: 7
doc_type: how-to
products:
  - opencti
personas: [cti-lead, cti-analyst, admin]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/import/json-feed/
---

A JSON feed calls a web API that answers in JSON, maps the answer to entities and relationships with a JSON mapper, and repeats on a schedule. Use it when a partner or a vendor exposes its intelligence as an API rather than as a file or a TAXII collection. For an API that returns its data page by page, the feed keeps a state between runs, such as an offset or a cursor, and puts it in the next request. For how imported data reaches the knowledge base, read [How intelligence gets into OpenCTI](/opencti/use/ingest/).

## Before you start

- Your role has the **Manage ingestion** capability. To create a JSON mapper, it also needs **Manage data mappers**.
- A [JSON mapper](/opencti/configure/mappers/json/) exists for the answers of the API.
- You know how the API paginates, if it does: which parameter moves to the next page, and where the answer gives its value.

## Create a feed for a simple API

An API that returns all its data at once needs no pagination settings. Open the creation form:

::::only{opencti=">=7.260811"}
Open :menu[Integrations], select the **Available** tab, and click **Create** on the **JSON Feed** card.
::::

::::only{opencti="<7.260811"}
Open :menu[Data > Ingestion > JSON Feeds] and click the create button.
::::

Then fill it in:

:::steps

1. Enter a **Name** and a **Description**.
2. Choose a **Schedule period**. An API without pagination returns everything at each call, so prefer a long period, such as 24 hours.
3. Enter the **HTTP JSON URL**.
4. Choose the **HTTP VERB**. For `POST`, enter the request body in **HTTP BODY POST**.
5. Add the headers the API requires with **Add header**, each with a **Header name** and a **Header value**.
6. Choose the account that writes the data in **User responsible for data creation (empty = System)**. Leave it empty only if the data should be written with the full rights of the system.
7. Choose the mapper in **JSON Mappers**. If the mapper lets the user choose markings, choose them in **Marking definition levels**.
8. Choose the **Authentication type** and enter the credentials: **Basic user / password**, **Bearer token** or **Client certificate**, or **None**.
9. Keep **Verify SSL certificate** on, unless the server uses a certificate your platform cannot check.
10. Click **Verify**, then **Test** in the dialog. The test maps up to 50 objects and shows them, with the state the next run would use.
11. If the result is what you expect, close the dialog and click **Create**. **Create** stays disabled until a test finds at least one entity.
12. Open the actions menu of the new feed and choose **Start**.

:::

## Paginate with query attributes

A query attribute is a variable the feed computes from each answer and sends in the next request. You write it in the URL, the body or a header as `$name`, for example `$offset`.

:::only{opencti=">=7.261127"}
Keep variables out of the server part of the URL. When the URL, once its variables are replaced, points to another scheme, host or port than the URL you entered, the run fails with "The URI of the feed points to another server once its variables are replaced". The credentials and headers of the feed only go to its own server.
:::

Take an API that returns 20 vulnerabilities per call and moves forward with a `startIndex` parameter:

```text
https://api.example/rest/json/cves/2.0?resultsPerPage=20&startIndex=$offset
```

Click **Add query attribute** and set:

| Field | Value | Meaning |
| --- | --- | --- |
| **Resolve from** | **Data** | Read the value from the body of the answer. **Header** reads it from a response header |
| **Exposed attribute to** | **Query parameter** | Put it in the URL. **Body** and **Header** put it in the request body or a request header |
| **Resolve operation** | **Count** | Count the elements found at the path. **Data** takes the value found as it is |
| **State operation** | **Sum** | Add the new value to the previous one. **Replace** keeps the new value, or the previous one when the answer has none |
| **Get from path** | `$.vulnerabilities` | The JSONPath of the value in the answer, or the header name |
| **To attribute name** | `offset` | The variable to fill, written `$offset` in the request |
| **Default value** | `0` | The value of the first run |

With these settings, the first run asks for `startIndex=0`. It receives 20 vulnerabilities, so the next run asks for `startIndex=20`, and so on. A cursor-based API uses **Data** and **Replace** instead: the feed reads the cursor in the answer and sends it as it is.

The same variables work in a `POST` body, for example `{ "page": "$offset" }`, or in a query such as `SELECT * FROM observables WHERE created_at > TIMESTAMP '$created' ORDER BY created_at ASC LIMIT 10`.

::screenshot{id=opencti-ingest-json-feed-query-attribute alt="A query attribute reads the count of elements at $.vulnerabilities in the answer, sums it into the offset variable, and exposes it as a query parameter with a default value of 0."}

The **Test** dialog shows the state computed from the first answer: check that it holds the value you expect before you create the feed.

:::only{opencti=">=7.261127"}
When you export the feed, a query attribute whose **To attribute name** looks like a credential, such as `api_key`, `token` or `password`, loses its **Default value** in the file, as the authentication and the sensitive headers do. After an import, enter it again.
:::

:::example[Fetch only the threats a partner changed since the last run]
A partner of Example CERT exposes its threat tracking as a fictional JSON API. Each call returns the threats modified after a date, with the addresses they use, and tells in `meta.last_modified` the date of the most recent change it returned:

```json
{
  "meta": { "count": 2, "last_modified": "2026-01-14T17:42:00Z" },
  "threats": [
    {
      "name": "Velvet Harbor",
      "addresses": [{ "ip": "203.0.113.10" }, { "ip": "198.51.100.7" }]
    },
    {
      "name": "Quiet Concierge",
      "addresses": [{ "ip": "203.0.113.94" }]
    }
  ]
}
```

The JSON mapper `Threats and addresses` maps each element of `$.threats[*]` to an intrusion set named after `$.name`, each element of `$.addresses[*]` to an IPv4 address, and links each address to its intrusion set with `related-to`.

- **Name**: `Partner threat API`
- **Schedule period**: **1 hour**
- **HTTP JSON URL**: `https://api.threats.example/v2/threats?modified_after=$since`
- **HTTP VERB**: **GET**
- **Add header**: **Header name** `X-Client-Id`, **Header value** `example-cert`, the client name the partner asked for
- **User responsible for data creation (empty = System)**: `Partner threat API`, a service account created for this source, so its data is easy to filter and its confidence level limits what it can overwrite
- **Authentication type**: **Bearer token**, with the API key of the partner in **Token**
- **JSON Mappers**: `Threats and addresses`
- **Add query attribute**: **Resolve from** **Data**, **Exposed attribute to** **Query parameter**, **Resolve operation** **Data**, **State operation** **Replace**, **Get from path** `$.meta.last_modified`, **To attribute name** `since`, **Default value** `2026-01-01T00:00:00Z`

The first run asks for `modified_after=2026-01-01T00:00:00Z` and creates two intrusion sets, three addresses and three relationships. The state then holds `since` = `2026-01-14T17:42:00Z`, so the next run only asks for what changed after it. When nothing changed, the answer has no `last_modified`, and **Replace** keeps the previous date.
:::

### Sub pagination

Some APIs, such as Trino, answer with a link to the next page instead of the data. Turn on **Sub pagination**, choose the **Sub pagination verb**, and enter in **Attribute path to get next uri** the JSONPath of that link. Within one run, the feed then follows the links until the answer has no next link. Use it only for those APIs.

:::only{opencti=">=7.261127"}
A next link on another server, such as a content delivery network, is fetched without the authentication, the client certificate and the headers whose name holds `authorization`, `token`, `key`, `secret`, `password`, `cookie` or `credential`.

If you **Stop** the feed during such a run, it stops before the next page. Nothing from that run is imported and the state does not move, so when you start the feed again, its next run starts from the same first page. The **Logs** tab shows "Feed execution stopped: the feed is no longer running".
:::

## Check that it works

::::only{opencti=">=7.260811"}
Open the feed from the **Deployed** tab of :menu[Integrations]. After the first run, its page shows the last run and the current state, which holds the variables of the next request.

:::only{opencti=">=7.261030"}
The **Logs** tab shows each run with the number of items mapped.
:::
::::

::::only{opencti="<7.260811"}
In :menu[Data > Ingestion > JSON Feeds], the feed shows as running with its last run date.
::::

If the state stops moving, the path no longer matches the answer. Fix the query attribute, then **Reset state** to start again from the default values.

:::only{opencti=">=7.261127"}
An API on your internal network fails when your platform engineer blocks internal destinations: the **Logs** show an error that starts with `JSON feed:` and names the host. See [Internal destinations](/opencti/use/ingest/feed-options/#internal-destinations).
:::

## What's next

- [Map JSON APIs to knowledge](/opencti/configure/mappers/json/), to map the answer to entities and relationships.
- [Manage a feed](/opencti/use/ingest/#manage-a-feed): duplicate, reset or export the feed.
- [Tune feeds and streams](/opencti/use/ingest/feed-options/), for the request timeout and the URI deny list.
