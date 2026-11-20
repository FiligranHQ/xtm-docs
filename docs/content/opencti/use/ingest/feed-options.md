---
title: Tune feeds and streams
description: The platform settings that control OpenCTI feeds and streams, such as the polling interval, the URI deny list, request timeouts and per-feed limits.
sidebar_position: 8
doc_type: reference
products:
  - opencti
personas: [platform-engineer, admin, cti-lead]
owner: docs-opencti
no_screenshot: "These options are configuration keys of the platform, set outside the interface."
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/usage/import/advanced-feed-configuration/
---

This page lists the platform settings that change how the built-in feeds, TAXII push collections and OpenCTI streams run: which URLs they may call, how often they poll, how long they wait and how much they accept. They are configuration keys of the platform, not settings of a feed, so an administrator sets them for every feed at once. Each key has an environment variable, its name in upper case with `__` between levels: set it in Docker, or set the key in `config/production.json`. The [configuration reference](/reference/opencti/configuration/#engines-schedules-and-managers) lists every key of your version.

::diagram{id=opencti-feed-options-ingestion}

## URI deny list

::::only{opencti=">=7.260811"}
The deny list stops feeds from calling hosts you do not want the platform to reach, such as internal services. It applies to OpenCTI streams and to TAXII, RSS, CSV and JSON feeds. Creating a feed, changing its URL or testing it with a denied URL fails with `This URI is not allowed for ingestion.` The same list also restricts the URL of webhook notifiers.

| Key | Environment variable | Default |
| --- | --- | --- |
| `app:uri_deny_list` | `APP__URI_DENY_LIST` | `[]` |
| `ingestion_manager:uri_deny_list` | `INGESTION_MANAGER__URI_DENY_LIST` | none |

The platform combines both lists. Each holds host patterns, as a JSON array in the environment variable:

```bash
APP__URI_DENY_LIST='["internal-service.local","*.corp.internal","localhost:4200"]'
```

| Pattern | Denies |
| --- | --- |
| `mydomain.com` | Any URL on that host, whatever its port |
| `*.mydomain.com` | Every subdomain of `mydomain.com`, and `mydomain.com` itself |
| `localhost:4200` | That host on that port only |

Matching ignores case. A JSON feed also checks the URL after it puts its variables in, and each next link it follows, so a variable cannot hide a denied host.
::::

::::only{opencti="<7.260811"}
This version has no URI deny list: restrict the hosts the platform can reach with your network rules.
::::

## Ingestion manager

The ingestion manager polls the TAXII, RSS, CSV and JSON feeds.

| Key | Environment variable | Default | Effect |
| --- | --- | --- | --- |
| `ingestion_manager:enabled` | `INGESTION_MANAGER__ENABLED` | `true` | Turns the polling of every feed on or off |
| `ingestion_manager:interval` | `INGESTION_MANAGER__INTERVAL` | `30000` | Time between two checks of the running feeds, in milliseconds. A feed's **Schedule period** can only make it run less often |
| `ingestion_manager:lock_key` | `INGESTION_MANAGER__LOCK_KEY` | `ingestion_manager_lock` | Redis lock that lets only one platform node poll at a time |

::::only{opencti=">=7.260811"}
| Key | Environment variable | Default | Effect |
| --- | --- | --- | --- |
| `ingestion_manager:feed:request_timeout` | `INGESTION_MANAGER__FEED__REQUEST_TIMEOUT` | `300000` | Time a feed waits for the source to answer, in milliseconds |
::::

::::only{opencti=">=7.261119"}
## Redirects

The TAXII, RSS, CSV and JSON feeds, the OpenCTI streams and the notifiers follow the redirects a source answers with, up to a limit.

| Key | Environment variable | Default | Effect |
| --- | --- | --- | --- |
| `app:http:max_redirects` | `APP__HTTP__MAX_REDIRECTS` | `21`, or `5` when `app:http:secure_by_default` is `true` | Number of redirects a request follows. `0` follows none. A request that needs more fails with `Maximum number of redirects exceeded` |

A value that is not a whole number of zero or more falls back to the default, with a warning in the log at start. Lower the limit when you want a feed to fail rather than wander through a chain of redirects; raise it when a source sits behind several redirecting gateways. See [Secure HTTP defaults](/deploy/operate/harden/#secure-http-defaults).
::::

## TAXII feeds

| Key | Environment variable | Default | Effect |
| --- | --- | --- | --- |
| `ingestion_manager:taxii_feed:limit_per_request` | `INGESTION_MANAGER__TAXII_FEED__LIMIT_PER_REQUEST` | `0` | Maximum number of objects asked for in one poll. `0` sends no limit, so the server decides |

Set a limit when a server times out or sends answers too large to process. The feed then pages through the collection, one poll at a time.

## TAXII push

| Key | Environment variable | Default | Effect |
| --- | --- | --- | --- |
| `app:max_payload_body_size` | `APP__MAX_PAYLOAD_BODY_SIZE` | `50mb` | Largest request body the platform accepts, on its API and on the TAXII push collections. A larger push is refused |

Raise it when partners push large bundles in one request, and check that your reverse proxy accepts the same size.

:::only{opencti=">=7.261119"}
Unset, the limit falls to `10mb` when `app:http:secure_by_default` is `true`: see [Secure HTTP defaults](/deploy/operate/harden/#secure-http-defaults). The TAXII API root, `/taxii2/root/`, advertises the limit in bytes as `max_content_length`, so that a TAXII client knows how large a push the platform accepts.
:::

## RSS feeds

| Key | Environment variable | Default | Effect |
| --- | --- | --- | --- |
| `ingestion_manager:rss_feed:min_interval_minutes` | `INGESTION_MANAGER__RSS_FEED__MIN_INTERVAL_MINUTES` | `5` | Minimum time between two fetches of the same RSS feed, in minutes, also after a failure |
| `ingestion_manager:rss_feed:user_agent` | `INGESTION_MANAGER__RSS_FEED__USER_AGENT` | A Firefox user agent | `User-Agent` header sent to RSS sources |

The default user agent is a browser one, because some sites reject other clients. Change it if a site still refuses the platform's requests.

## CSV feeds

| Key | Environment variable | Default | Effect |
| --- | --- | --- | --- |
| `ingestion_manager:csv_feed:min_interval_minutes` | `INGESTION_MANAGER__CSV_FEED__MIN_INTERVAL_MINUTES` | `5` | Minimum time between two fetches of the same CSV feed, in minutes, also after a failure |
| `app:csv_ingestion:max_bundle_size` | `APP__CSV_INGESTION__MAX_BUNDLE_SIZE` | `1000` | Maximum number of objects in each bundle a CSV import sends to the workers |

## OpenCTI streams

The sync manager follows the remote streams of OpenCTI streams.

| Key | Environment variable | Default | Effect |
| --- | --- | --- | --- |
| `sync_manager:enabled` | `SYNC_MANAGER__ENABLED` | `true` | Turns every OpenCTI stream on or off |
| `sync_manager:interval` | `SYNC_MANAGER__INTERVAL` | `10000` | Time between two checks for streams to start or stop, in milliseconds |
| `sync_manager:lock_key` | `SYNC_MANAGER__LOCK_KEY` | `sync_manager_lock` | Redis lock that lets only one platform node run the streams |

::::only{opencti=">=7.260811"}
| Key | Environment variable | Default | Effect |
| --- | --- | --- | --- |
| `sync_manager:file_fetch_timeout` | `SYNC_MANAGER__FILE_FETCH_TIMEOUT` | `300000` | Time to download a file attached to a streamed object, in milliseconds |
::::

## What's next

- [How intelligence gets into OpenCTI](/opencti/use/ingest/), for how feeds use these settings.
- [OpenCTI configuration](/reference/opencti/configuration/), for every configuration key.
