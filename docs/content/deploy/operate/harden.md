---
title: Harden the deployment
description: Close the shared services of the XTM stack to the network, protect the health keys, limit who gets an OpenCRQ account, and tighten the OpenCTI settings, from rate limiting to sessions and logs.
doc_type: how-to
products:
  - opencti
  - opencrq
  - suite
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - opencti:/deployment/advanced/rate-limiting/
  - opencti:/deployment/advanced/security-configuration/
---

The XTM stack starts with settings that make a first deployment easy, not with the strictest ones. This page lists what to tighten before production: the shared services the Docker deployment publishes, the health keys, who can create an account in XTM One, and the security settings of OpenCTI, each with its key, its default and the value to choose.

::diagram{id=deploy-operate-harden}

## Before you start

- A deployment that works: see [Install with Docker Compose](/deploy/install/docker-compose/).
- Every password, token and key of `.env` replaced with a random value, as in the [Quick start](/deploy/quick-start/).
- The platforms published over TLS, behind a reverse proxy: see [Publish the platforms over HTTPS](/deploy/install/reverse-proxy/).

In xtm-docker, the platforms take their settings from `docker-compose.yml`. Put your changes in `docker-compose.override.yml`, as [Change the stack without editing docker-compose.yml](/deploy/install/docker-compose/#change-the-stack-without-editing-docker-compose-yml) describes, then apply them with `docker compose up -d`.

## Close Elasticsearch and Redis to the network

xtm-docker publishes the ports of the shared services on every interface of the host. Two of them have no authentication: Elasticsearch, on port 9200, runs with `xpack.security.enabled=false`, and Redis, on port 6379, has no password. Anyone who reaches these ports reads and changes all the data. The platforms reach them on the Docker network, so the published ports are not needed.

:::steps
1. Remove the published ports in `docker-compose.override.yml`. If the file already has `ports` for these services, replace them. The `!reset` tag needs Docker Compose 2.24 or later.

   ```yaml
   services:
     elasticsearch:
       ports: !reset []
     redis:
       ports: !reset []
   ```

2. Recreate the two services.

   ```bash
   docker compose up -d elasticsearch redis
   ```

   :::verify
   ```bash
   docker compose ps --format '{{.Service}} {{.Ports}}' | grep -E '^(elasticsearch|redis) '
   ```
   No line shows a `0.0.0.0:` mapping.
   :::
:::

RabbitMQ (5672, and 15672 for its management interface) and S3 storage (9000) are protected by the user and password of `.env`. Integrations that run outside the host connect to RabbitMQ: if you have none, restrict these ports with the firewall of the host.

## Protect the health keys

The health endpoints of OpenCTI and OpenAEV answer only with their key; those of XTM One and OpenCRQ need none. See [Check that the platforms are healthy](/deploy/operate/health-probes/).

- OpenCTI refuses every call while its key is empty or `ChangeMe`. Keep a random `OPENCTI_HEALTHCHECK_ACCESS_KEY`, and do not set it to `public` on a network you do not trust.
- OpenAEV accepts its default key, `ChangeMe`, and xtm-docker falls back to it when `OPENAEV_HEALTHCHECK_KEY` is missing from `.env`. Set a random value.

## Limit who gets an OpenCRQ account

OpenCRQ signs its users in through XTM One, and gives an account to an XTM One user at their first sign-in, unless another account already holds their email, even one who is a member of no tenant: they then read **You don't have access to OpenCRQ yet** until an administrator adds them. Who can sign in to OpenCRQ is therefore who can get an XTM One account.

In XTM One, keep **Local Registration** off, or limit new accounts to your domains with **Allowed email domains**, and do the same for the **On-the-fly user creation** of each single sign-on method. See [Decide who gets an account](/xtm-one/administer/sso/#decide-who-gets-an-account) in XTM One.

## Tighten OpenCTI

The settings below are keys of the OpenCTI configuration. Each one has an environment variable: the key in capitals, with `__` between its levels, such as `APP__SESSION_TIMEOUT` for `app:session_timeout`. Give a list as a JSON array.

```yaml
services:
  opencti:
    environment:
      - APP__SESSION_TIMEOUT=900000
      - 'APP__RATE_PROTECTION__IP_SKIP_RANGES=["10.0.0.0/8"]'
```

### Rate limiting

OpenCTI limits the number of HTTP requests each client can send in a time window, GraphQL included, before any session handling. A client over the limit gets `429 Too Many Requests` until the window ends. Use it when nothing in front of the platform limits the rate already.

:::only{opencti=">=7.260811"}
A client is an IP address with a `User-Agent` header: two user agents behind the same IP address have their own limits. An IPv6 address counts by its `/56` network.
:::

:::only{opencti="<7.260811"}
A client is an IP address.
:::

| Key | Default | What it does |
| --- | --- | --- |
| `app:rate_protection:max_requests` | `10000` | Requests a client may send in the window. `0` refuses every request: it does not turn the limiter off. To exempt clients, use the skip lists |
| `app:rate_protection:time_window` | `1` | Length of the window, in seconds. Below `1`, one second |
| `app:rate_protection:ip_skip_list` | `[]` | IP addresses never limited, as exact addresses |

:::only{opencti=">=7.260811"}
| Key | Default | What it does |
| --- | --- | --- |
| `app:rate_protection:ip_skip_ranges` | `[]` | Ranges never limited, in CIDR notation, IPv4 or IPv6, such as `10.0.0.0/8` or `fd00::/8`. A single address works too |
| `app:rate_protection:user_agent_skip_prefixes` | `[]` | User agents never limited: a request whose `User-Agent` starts with one of these values, ignoring case |

A request is not limited when it matches any of the three lists. For example, 5,000 requests per 10 seconds for each client, except the private networks and the browsers, whose user agent starts with `Mozilla`:

```yaml
- APP__RATE_PROTECTION__MAX_REQUESTS=5000
- APP__RATE_PROTECTION__TIME_WINDOW=10
- 'APP__RATE_PROTECTION__IP_SKIP_RANGES=["10.0.0.0/8","192.168.0.0/16"]'
- 'APP__RATE_PROTECTION__USER_AGENT_SKIP_PREFIXES=["Mozilla"]'
```
:::

:::only{opencti=">=7.261119"}
Unset, `app:rate_protection:max_requests` and `app:rate_protection:time_window` follow `app:http:secure_by_default`: 30,000 requests per 60 seconds when it is `true`. See [Secure HTTP defaults](#secure-http-defaults).
:::

Behind a reverse proxy, OpenCTI limits each client by the address in the proxy's `X-Forwarded-For` header only when it trusts the proxy. Loopback, link-local and private addresses are trusted.

:::only{opencti=">=7.260811"}
Add the address of a proxy outside these ranges to `trust_proxy_addresses`.
:::

### GraphQL query limits

The rate limiter counts HTTP requests, but one GraphQL request can hold many operations. OpenCTI checks each GraphQL document before it runs it, with two protections.

The batching protection is always on. It limits how many times one request calls the same query or mutation through aliases, a way to guess passwords or overload the platform in a single HTTP request. The login mutation is always limited to one call.

| Key | Default | What it does |
| --- | --- | --- |
| `app:graphql:batching_protection:query_default` | `2` | Calls of the same query in one request |
| `app:graphql:batching_protection:query_subtypes` | `4` | Calls of the `subTypes` query, which the interface sends several times |
| `app:graphql:batching_protection:mutation_default` | `1` | Calls of the same mutation in one request |

The armor protection is off by default: `app:graphql:armor_protection:disabled` is `true`. Once on, it refuses a document that goes over one of its limits, and stops suggesting field names in the error of a wrong query: these suggestions help an attacker map the API.

| Key | Default | What it does |
| --- | --- | --- |
| `app:graphql:armor_protection:disabled` | `true` | Set to `false` to turn the armor protection on |
| `app:graphql:armor_protection:max_depth` | `20` | Nesting depth of a document |
| `app:graphql:armor_protection:max_directives` | `20` | Directives in a document |
| `app:graphql:armor_protection:max_tokens` | `100000` | GraphQL tokens in a document |
| `app:graphql:armor_protection:cost_limit` | `50000000` | Computed cost of a document |
| `app:graphql:armor_protection:block_field_suggestion` | `true` | Hides the field suggestions |

:::only{opencti="<7.260811"}
In this version, `app:graphql:armor_protection:cost_limit` defaults to `3000000`.
:::

Turn it on in a staging platform first, and check that the screens and the integrations you use still work: a limit set too low refuses their queries.

```yaml
- APP__GRAPHQL__ARMOR_PROTECTION__DISABLED=false
- APP__GRAPHQL__ARMOR_PROTECTION__MAX_DEPTH=20
- APP__GRAPHQL__ARMOR_PROTECTION__BLOCK_FIELD_SUGGESTION=true
```

:::only{opencti=">=7.261109"}
### Secure GraphQL defaults

`app:graphql:secure_by_default` (`APP__GRAPHQL__SECURE_BY_DEFAULT`) turns on, in one setting, the GraphQL protections you have not set yourself:

| When `app:graphql:secure_by_default` is `true` | Overridden by |
| --- | --- |
| The armor protection is on | `app:graphql:armor_protection:disabled` |
| The [GraphQL playground](#graphql-playground) is off, and introspection with it | `app:graphql:playground:enabled` |
| The WebSocket endpoint accepts subscriptions only, the operations the interface sends there; a query or a mutation sent over WebSocket gets `Only subscriptions are allowed over WebSocket` | |

It is `false` by default, and OpenCTI logs a warning at each start while it is: `GraphQL secure defaults are disabled for the protections that do not set their own option`. A later version turns it on by default. Turn it on in a staging platform first, then in production, and set the other keys only where you need an exception:

```yaml
- APP__GRAPHQL__SECURE_BY_DEFAULT=true
- APP__GRAPHQL__PLAYGROUND__ENABLED=true
```

Here the armor protection is on and WebSocket carries only subscriptions, while the playground stays available to your analysts.
:::

:::only{opencti=">=7.261119"}
### Secure HTTP defaults

`app:http:secure_by_default` (`APP__HTTP__SECURE_BY_DEFAULT`) tightens, in one setting, the HTTP protections you have not set yourself:

| Protection | Unset key, by default | Unset key, when `app:http:secure_by_default` is `true` | Overridden by |
| --- | --- | --- | --- |
| [Rate limiting](#rate-limiting) of each client | 10,000 requests per second | 30,000 requests per 60 seconds | `app:rate_protection:max_requests`, `app:rate_protection:time_window` |
| Largest JSON request body, on the GraphQL, TAXII and REST APIs | `50mb` | `10mb` | `app:max_payload_body_size` |
| [Resources over plain HTTP](#insecure-http-resources) in the pages of the platform | Allowed | Refused | `app:allow_unsecure_http_resources` |
| Redirects followed by the requests OpenCTI sends, for feeds, synchronizers and notifiers | `21` | `5` | `app:http:max_redirects` |

It is `false` by default, and OpenCTI logs a warning at each start while it is: `HTTP secure defaults are disabled for the protections that do not set their own option`. A later version turns it on by default.

- `app:max_payload_body_size` (`APP__MAX_PAYLOAD_BODY_SIZE`) takes a number of bytes, or a value with `kb`, `mb` or `gb`. A larger body is refused. The TAXII API root advertises this limit, in bytes, as its `max_content_length`.
- `app:http:max_redirects` (`APP__HTTP__MAX_REDIRECTS`) takes a whole number, `0` to follow no redirect. A request that needs more redirects fails with `Maximum number of redirects exceeded`. Any other value falls back to the default, and OpenCTI logs `Invalid value for app:http:max_redirects, it must be a non-negative integer: using the default instead` at start.

Turn it on in a staging platform first. Check that your connectors and scripts stay under the rate limit and send bodies under 10 MB, then set the other keys only where you need an exception:

```yaml
- APP__HTTP__SECURE_BY_DEFAULT=true
- APP__MAX_PAYLOAD_BODY_SIZE=50mb
```

Here the rate limit, the plain HTTP resources and the redirects take their secure values, while the API still accepts the 50 MB bundles of connectors that send them over HTTP.
:::

### Requests and uploads

`app:request_timeout` closes a connection that stays idle this many milliseconds. The default, `1200000`, is 20 minutes. Keep the timeout of your reverse proxy consistent with it, or long requests fail at the proxy first.

:::only{opencti=">=7.261030"}
| Key | Default | What it does |
| --- | --- | --- |
| `app:max_upload_file_size` | `1gb` | Largest file a request may upload: a number of bytes, or a value with `kb`, `mb` or `gb`. A larger file is refused |
| `app:max_upload_files` | `20` | Files one request may upload |

An invalid value falls back to the default. Lower the size if the platform never ingests large artifacts, such as malware samples:

```yaml
- APP__MAX_UPLOAD_FILE_SIZE=200mb
- APP__MAX_UPLOAD_FILES=10
```
:::

:::only{opencti="<7.261030"}
This version does not limit the size or the number of uploaded files: limit the size of request bodies on your reverse proxy.
:::

### Encryption key

OpenCTI derives its encryption keys from `app:encryption_key` (`APP__ENCRYPTION_KEY`): 32 bytes or more, in base64, generated with `openssl rand -base64 32`. They protect the secrets it stores in its database, such as those of the authentication providers and of the managed connectors. OpenCTI does not start without it, and cannot read these secrets with another key: keep it in your secret store with the backups.

`app:crypto:master_seed` (`APP__CRYPTO__MASTER_SEED`) is the former name of this key, read only when `app:encryption_key` is not set. If your configuration still uses it, move its value to `app:encryption_key`.

### HTTPS served by OpenCTI

Behind a reverse proxy, OpenCTI does not need certificates. When it serves HTTPS itself, as [Serve HTTPS from a platform](/deploy/configure/certificates/#serve-https-from-a-platform) describes, three more keys apply. `app:https_cert:passphrase` decrypts an encrypted private key. With certificate authentication turned on, OpenCTI asks clients for a certificate, checks it against the authorities listed in `app:https_cert:ca`, paths or PEM contents, and refuses a client whose certificate does not match while `app:https_cert:reject_unauthorized` is `true`, the default.

### Sessions

`app:session_timeout` ends a session that stays idle this many milliseconds: each request starts the count again. The default, `1200000`, is 20 minutes: set it to what your security policy requires. See [Cookies](/opencti/administer/cookies/).

### Public dashboards

By default, no other website can embed a public dashboard. To allow some, list them in `app:public_dashboard_authorized_domains`, as the sources of a Content Security Policy `frame-ancestors` directive, separated by spaces, such as `https://intranet.example.com`. Leave it empty to keep embedding forbidden.

::::only{opencti=">=7.260811"}
### Insecure HTTP resources

:::only{opencti=">=7.261119"}
`app:allow_unsecure_http_resources` decides whether the pages of the platform may load images, files and connections from `http://` addresses, through its Content Security Policy. Unset, it is the opposite of `app:http:secure_by_default`: allowed by default, refused with the [secure HTTP defaults](#secure-http-defaults).

Set it to `false` to also make browsers load every resource of the platform over HTTPS: OpenCTI then sends the `upgrade-insecure-requests` directive. The secure HTTP defaults alone do not send it, so a platform served over plain HTTP keeps working with them.
:::

:::only{opencti="<7.261119"}
`app:allow_unsecure_http_resources` is `true` by default. Set it to `false` to make browsers load every resource of the platform over HTTPS: OpenCTI then sends the `upgrade-insecure-requests` directive.
:::

### Ingestion URL filtering

To keep feeds from reaching internal addresses, a server-side request forgery (SSRF) risk, list the URIs that no ingestion feed may call in `app:uri_deny_list`. See [Tune feeds and streams](/opencti/use/ingest/feed-options/).
::::

### Logs

| Key | Default | What it does |
| --- | --- | --- |
| `app:app_logs:logs_redacted_inputs` | `password`, `secret`, `token`, `api_tokens`, `authentication_value`, `enterprise_license` | GraphQL input fields whose value is hidden in the logs. The default ones are always hidden: add the fields of your own sensitive data |
| `app:audit_logs:trace_request_headers` | `user-agent`, `x-forwarded-for` | HTTP headers written to the audit logs with each event |
| `app:app_logs:extended_error_message` | `false` | Adds the input data to error logs, and the GraphQL query and its variables to the logs of API calls. Keep it `false`: the logs would then hold sensitive data |
| `app:performance_logger` | `false` | Logs every API call with its duration and size. Keep it `false` outside a performance analysis: it writes a lot of logs |

See [Troubleshoot the XTM stack](/deploy/operate/troubleshoot/) for the other log settings.

### GraphQL playground

`app:graphql:playground:enabled` is `true` by default and serves the GraphQL playground of the API.

:::only{opencti=">=7.261109"}
Unset, it follows `app:graphql:secure_by_default`: the playground is off when that key is `true`.
:::

Set it to `false` if your users do not need it. Without the playground, the [GraphQL schema reference](/reference/opencti/graphql-schema/) still documents every query and mutation.

`app:graphql:playground:force_disabled_introspection` is `true` by default, but xtm-docker sets it to `false`, for OpenAEV's Coverage connector: signed-in users can then introspect the API, which returns the whole schema. Set it to `true` to refuse introspection even with the playground on, once your OpenAEV no longer needs it. Without the playground, or with the interface turned off, introspection stays refused.

### Artifact archives

OpenCTI protects potentially malicious files, the artifacts, with a password: it opens with it the ZIP archives you import, and puts a file you download as an artifact in an encrypted ZIP archive. The password is `app:artifact_zip_password`, `infected` by default, the usual value for malware samples. Change it if your policy requires another one.

## What's next

- [Network and security settings of OpenCTI](/reference/opencti/configuration/#network-and-security), every security key with its default, and the [whole OpenCTI configuration](/reference/opencti/configuration/).
- [Set platform security policies](/opencti/administer/policies/), the security policies set in the interface.
- [Back up and restore](/deploy/operate/backup/), to keep the secrets safe with the data.
