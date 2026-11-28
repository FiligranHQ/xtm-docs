---
title: Connect the modules to each other
description: Register OpenCTI, OpenAEV and OpenCRQ with XTM One, link OpenCTI and OpenAEV, connect OpenCRQ to both, and give XTM Composer its tokens, with the public and internal URLs each link uses.
doc_type: how-to
products:
  - suite
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-10-31
---

This page connects the modules of the XTM stack: OpenCTI, OpenAEV and OpenCRQ to XTM One, OpenCTI and OpenAEV to each other, OpenCRQ to both, and XTM Composer to both. xtm-docker wires every link from `.env`, so on a new stack you mostly set one shared secret and check the result. Read it too when you publish the platforms on other URLs, or run a module outside xtm-docker.

::diagram{id=deploy-configure-connect-modules}

## Before you start

- The stack installed as [Install with Docker Compose](/deploy/install/docker-compose/) describes.
- After a change to `.env` or `docker-compose.override.yml`, run `docker compose up -d`: Docker Compose recreates the containers whose configuration changed. Leave `docker-compose.yml` as it is: see [Change the stack without editing docker-compose.yml](/deploy/install/docker-compose/#change-the-stack-without-editing-docker-compose-yml).

## The links and their settings

| Link | Where it is set in xtm-docker | Address | Credential |
| --- | --- | --- | --- |
| OpenCTI registers with XTM One | `opencti` service | `XTM__XTM_ONE_URL=http://xtm-one:4000` | `XTM__XTM_ONE_TOKEN`, from `PLATFORM_REGISTRATION_TOKEN` |
| OpenAEV registers with XTM One | `openaev` service | `OPENAEV_XTM_ONE_URL=http://xtm-one:4000` | `OPENAEV_XTM_ONE_TOKEN`, from `PLATFORM_REGISTRATION_TOKEN` |
| OpenCRQ registers with XTM One | `opencrq` service | `XTM_ONE__URL=http://xtm-one:4000`, and the public URL of OpenCRQ in `XTM_ONE__PLATFORM_URL` | `XTM_ONE__TOKEN`, from `PLATFORM_REGISTRATION_TOKEN` |
| XTM One accepts the registrations | `xtm-one` and `xtm-one-worker` services | | `PLATFORM_REGISTRATION_TOKEN` |
| XTM One calls OpenCTI, OpenAEV and OpenCRQ | `xtm-one` service | `OPENCTI_API_URL=http://opencti:8080`, `OPENAEV_API_URL=http://openaev:8080`, `OPENCRQ_API_URL=http://opencrq:3002` | A short-lived token signed for each user |
| OpenAEV answers OpenCTI | `openaev` service | `openaev.xtm.opencti.api-url=http://opencti:8080/graphql` | `OPENAEV_XTM_OPENCTI_TOKEN`, from `OPENCTI_ADMIN_TOKEN` |
| OpenCTI links to OpenAEV | `opencti` service | `XTM__OPENAEV_URL` (public) | None: a link in the interface |
| OpenCRQ reads OpenCTI, for one tenant | `opencrq` service | `CONNECTORS__OPENCTI__<tenant>__URL` | `CONNECTORS__OPENCTI__<tenant>__TOKEN` |
| OpenCRQ reads OpenAEV, for one tenant | `opencrq` service | `CONNECTORS__OPENAEV__<tenant>__URL` | `CONNECTORS__OPENAEV__<tenant>__TOKEN` |
| XTM Composer runs integrations | `xtm-composer` service | `http://opencti:8080`, `http://openaev:8080` | `OPENCTI_ADMIN_TOKEN`, `OPENAEV_ADMIN_TOKEN` |

:::only{opencti="<7.260811"}
This version of OpenCTI signs no short-lived token for XTM One. At each registration, it creates for every user with an email an API token named `XTM One`, which never expires, and sends it to XTM One, which calls OpenCTI with it. A revoked `XTM One` token is created again at the next registration.
:::

## Register the platforms with XTM One

Each platform registers itself with XTM One, then registers again every 5 minutes: the registration is also its heartbeat. Once a platform is registered, its users get the XTM One assistant in its interface, and XTM One can work with its data.

:::steps
1. Generate the registration token and put it in `.env`. OpenCTI, OpenAEV, OpenCRQ, `xtm-one` and `xtm-one-worker` all read this one variable, so the value is the same everywhere.

   ```bash
   TOKEN=$(openssl rand -hex 32)
   sed -i.bak "s/^PLATFORM_REGISTRATION_TOKEN=.*/PLATFORM_REGISTRATION_TOKEN=${TOKEN}/" .env
   ```

   :::warning
   A platform outside xtm-docker, or a second XTM One, must use the exact same value. XTM One refuses every registration while the token is not set, and the published default `xtm-default-registration-token` cannot register a new platform. After 10 failed attempts in a minute, XTM One refuses a client with HTTP 429 until the minute ends.
   :::

2. Give XTM One the admin email of the platforms. XTM One matches users by email between the modules, so `XTM_ONE_ADMIN_EMAIL` must be the admin email of OpenCTI and OpenAEV.

   ```bash
   grep -E '^(OPENCTI|OPENAEV|XTM_ONE)_ADMIN_EMAIL=' .env
   ```

   The three lines show the same address.

3. Apply the change.

   ```bash
   docker compose up -d
   ```

   OpenCTI and OpenAEV register 30 seconds after they start, OpenCRQ as soon as it starts.

   :::verify
   :::only{opencti=">=7.260811"}
   Ask OpenCTI which version of XTM One it is registered with:

   ```bash
   set -a; . ./.env; set +a
   curl -s "http://localhost:$OPENCTI_PORT/graphql" \
     -H "Authorization: Bearer $OPENCTI_ADMIN_TOKEN" -H 'Content-Type: application/json' \
     -d '{"query":"{ about { dependencies { name version } } }"}'
   ```
   The `XTM-One` entry shows a version number, not `Not connected`.
   :::

   :::only{opencti="<7.260811"}
   This version of OpenCTI does not report XTM One among its dependencies. Check that it logs no registration failure:

   ```bash
   docker compose logs opencti | grep "\[XTM One\] Registration failed"
   ```
   The command prints nothing.
   :::

   For OpenCRQ, `docker compose logs opencrq | grep "Registered with XTM One"` prints a line: see [Finish the setup of OpenCRQ](/deploy/configure/opencrq/#finish-the-setup-after-the-install).

   In XTM One, **My Profile** > **MCP Endpoint** shows each registered platform as `Connected`.
   :::
:::

## Public and internal URLs

Each module has two URLs, and the signed calls between them use both.

- The **public URL** is the one users open in their browser. It is also the module's identity: every token a module signs names it as the issuer, and every token sent to a module must be addressed to it. xtm-docker builds it from `*_EXTERNAL_SCHEME`, `*_HOST` and `*_PORT` in `.env`, and passes it as `APP__BASE_URL` to OpenCTI, `OPENAEV_BASE-URL` to OpenAEV, `XTM_ONE__PLATFORM_URL` to OpenCRQ, and `BASE_URL` to both XTM One containers.
- The **internal URL** is only where the other containers connect: `http://opencti:8080`, `http://openaev:8080`, `http://opencrq:3002` and `http://xtm-one:4000`. The public host names therefore do not need to resolve inside Docker. OpenCRQ opens OpenCTI objects at the internal URL of its connection, as [Connect the first tenant](/deploy/configure/opencrq/#connect-the-first-tenant) describes.

When you publish the platforms on other URLs, change the three variables of each platform together, with the scheme set to `https` over TLS, as described in [Publish the platforms over HTTPS](/deploy/install/reverse-proxy/). Keep these rules:

- `BASE_URL` must be the same on `xtm-one` and `xtm-one-worker`: the worker signs the tokens of background work, and the platforms refuse a token from another issuer. xtm-docker sets both from the same variables.
- Keep `XTM_AUTH_ACCEPTED_AUDIENCES=http://xtm-one:4000` on `xtm-one`. It lets XTM One accept tokens addressed to the URL the platforms reach it on.

  ::::only{opencti=">=7.260811 <7.261030"}
  :::note
  This version of OpenCTI addresses its tokens to the URL it reaches XTM One on, `XTM__XTM_ONE_URL`. XTM One accepts them only because that URL is listed in `XTM_AUTH_ACCEPTED_AUDIENCES`.
  :::
  ::::

  ::::only{opencti=">=7.261030"}
  :::note
  OpenCTI reads XTM One's public identity from `/xtm/auth/metadata` on `XTM__XTM_ONE_URL`, and addresses its tokens to that identity.
  :::
  ::::

- Keep the clocks of the hosts synchronized with NTP. Each module signs its tokens for 10 minutes, and XTM One refuses a token from a clock more than 60 seconds ahead.

  :::only{opencti="<7.260811"}
  This version of OpenCTI signs the tokens of its calls to the XTM One assistant for one hour, with no issuer nor audience, and has no setting for that lifetime.
  :::

  :::only{opencti=">=7.260811"}
  On OpenCTI, `xtm:auth:token_ttl` (`XTM__AUTH__TOKEN_TTL`) sets that lifetime, in seconds, for the tokens it signs to call XTM One for a user. The default, `600`, is also the maximum: a higher value counts as `600`. Lower it to shorten the window in which a stolen token works.
  :::

:::verify
XTM One publishes the identity it signs with:

```bash
curl -s "http://localhost:$XTM_ONE_PORT/xtm/auth/metadata"
```
The `issuer` is the public URL of XTM One, such as `https://xtm-one.example.com`, whatever URL you reach it on.
:::

If the assistant answers but every action on a platform's data fails with `401`, look for the reason in the XTM One logs: the message names the issuer, the audience or the key URL that did not match.

```bash
docker compose logs xtm-one | grep "XTM JWT rejected"
```

## Link OpenCTI and OpenAEV

xtm-docker links the two platforms in both directions:

::::tabs
:::tab[OpenAEV to OpenCTI]
OpenAEV registers in OpenCTI as an enrichment connector named `OpenAEV Coverage - <tenant>`, which answers the security coverage requests of OpenCTI.

| Setting on `openaev` | Value in xtm-docker | Role |
| --- | --- | --- |
| `OPENAEV_XTM_OPENCTI_ENABLE` | `true` | Turns the link on |
| `OPENAEV_XTM_OPENCTI_URL` | The public URL of OpenCTI | Links to OpenCTI |
| `openaev.xtm.opencti.api-url`, passed in `SPRING_APPLICATION_JSON` | `http://opencti:8080/graphql` | The API OpenAEV calls. Without it, OpenAEV calls the public URL followed by `/graphql` |
| `OPENAEV_XTM_OPENCTI_TOKEN` | `${OPENCTI_ADMIN_TOKEN}` | The OpenCTI token OpenAEV registers with |

The `api-url` key holds a hyphen, which Spring does not read from an environment variable of that form, so xtm-docker passes it as JSON. These keys configure the default tenant of OpenAEV. With several tenants, each one has its own keys, `openaev.xtm.opencti.<tenant id>.enable`, `.url`, `.api-url` and `.token`: see [OpenAEV configuration](/reference/openaev/configuration/).
:::
:::tab[OpenCTI to OpenAEV]
OpenCTI reads one setting: `XTM__OPENAEV_URL`, the public URL of OpenAEV. The OpenAEV entry of the product switcher in the OpenCTI interface opens it, and shows **Platform connected** while it is set.

xtm-docker also sets `XTM__OPENAEV_API_URL=http://openaev:8080` and `XTM__OPENAEV_TOKEN=${OPENAEV_ADMIN_TOKEN}` on `opencti`. OpenCTI reads neither: it makes no call to OpenAEV, so no certificate setting applies to this link. The coverage data comes from OpenAEV, through the connector of the other tab.

xtm-docker also sets `APP__GRAPHQL__PLAYGROUND__FORCE_DISABLED_INTROSPECTION=false` on `opencti`. OpenAEV sends a GraphQL `__type` query before it registers its coverage connector. OpenCTI 7.261109.0 to 7.261119.0 refuse that query by default: without this setting, the connector of the other tab does not register. From 7.261127.0, OpenCTI answers it by default. With `APP__GRAPHQL__SECURE_BY_DEFAULT=true`, OpenCTI refuses it again, and OpenAEV then needs both `APP__GRAPHQL__PLAYGROUND__ENABLED=true` and this setting. See [Query OpenCTI with GraphQL](/develop/apis/graphql/).
:::
::::

:::verify
List the connectors OpenCTI knows:

```bash
curl -s "http://localhost:$OPENCTI_PORT/graphql" \
  -H "Authorization: Bearer $OPENCTI_ADMIN_TOKEN" -H 'Content-Type: application/json' \
  -d '{"query":"{ connectors { name active connector_scope } }"}'
```
The list holds `OpenAEV Coverage - 2cffad3a-0001-4078-b0e2-ef74274022c3`, the default tenant of OpenAEV, with the scope `security-coverage`.
:::

## Link OpenCRQ to OpenCTI and OpenAEV

OpenCRQ reads the threats of OpenCTI and the simulation results of OpenAEV, for each of its tenants separately. The connection of a tenant is two variables per platform on the `opencrq` service, keyed by the tenant's identifier, which its **Settings** > **Parameters** page in OpenCRQ shows under **Identifier**: see [Connect the first tenant](/deploy/configure/opencrq/#connect-the-first-tenant), and every connector in [Connectors of each tenant](/deploy/configure/opencrq/#connectors-of-each-tenant).

A connection set this way comes up enabled. At each start, for every tenant it holds with these variables, OpenCRQ does the following:

- **OpenAEV.** It creates the service account `OpenAEV (deployment)` in the tenant's **Connectors** group, and binds the OpenAEV connector to it.
- **OpenCTI.** It creates the Live Stream `OpenCRQ threat feed` in OpenCTI, with the token of the connection, or brings its filter back to what OpenCRQ needs: the Intrusion Sets, Threat Actors, Campaigns and Reports, and the `targets`, `uses` and `attributed-to` relationships between them and what they target or use. It then binds that stream with the service account `OpenCTI (deployment)`.

xtm-docker also prepares the other direction, from OpenAEV to OpenCRQ, for an OpenAEV release that reads it: on `openaev`, `OPENAEV_XTM_OPENCRQ_URL` (the public URL of OpenCRQ), `OPENAEV_XTM_OPENCRQ_TOKEN` (`OPENCRQ_ADMIN_TOKEN`) and `openaev.xtm.opencrq.api-url` (`http://opencrq:3002/trpc`, in `SPRING_APPLICATION_JSON`). OpenAEV 3.261118.0 reads none of them. The same `SPRING_APPLICATION_JSON` turns on `BULK_SNAPSHOT_EXPORT`, the preview feature of OpenAEV that the OpenAEV connector of OpenCRQ reads the simulation results from: see [Read the snapshot export of OpenAEV](/opencrq/use/sources/openaev/#read-the-snapshot-export-of-openaev).

These steps never replace a choice a tenant administrator made in **Integrations**: an account already bound, a stream already chosen or a connector turned off stay as they are. A variable that names a tenant the platform does not hold is skipped, and a platform that does not answer leaves its connector for the next start. xtm-docker sets these variables for the tenant of `OPENCRQ_TENANT_ID`, by default the tenant OpenCRQ creates at its first start, which is therefore connected from that start. For a tenant created later, in the Enterprise Edition, add its variables and run `docker compose up -d` so that OpenCRQ starts again and connects it.

To create or update its stream, the user of the OpenCTI token needs **Manage data sharing** or **Bypass all capabilities**: the admin token of OpenCTI holds the second. With another token, OpenCRQ changes nothing in OpenCTI, and a connector nobody configured stays idle, its page saying which capability the token lacks. Grant the capability in OpenCTI and restart OpenCRQ, or let a tenant administrator choose a stream with **Configure**: see [Connect OpenCTI with a Live Stream](/opencrq/use/sources/opencti/). OpenCRQ finds its stream again by a marker in the stream's description, so a stream renamed in OpenCTI still counts; an edit to its filter is replaced at the next start.

:::verify
Ask OpenCTI for its streams:

```bash
curl -s "http://localhost:$OPENCTI_PORT/graphql" \
  -H "Authorization: Bearer $OPENCTI_ADMIN_TOKEN" -H 'Content-Type: application/json' \
  -d '{"query":"{ streamCollections { edges { node { name stream_live } } } }"}'
```
The list holds `OpenCRQ threat feed`, with `stream_live` set to `true`. In OpenCRQ, the tenant's **Settings** > **Security** > **Service accounts** lists `OpenCTI (deployment)` and `OpenAEV (deployment)`.
:::

:::only{opencti=">=7.261127"}
In the other direction, OpenCTI reads one setting: `xtm:opencrq_url` (`XTM__OPENCRQ_URL`), the public URL of OpenCRQ. The OpenCRQ entry of the product switcher in the OpenCTI interface opens it, and shows **Platform connected** while it is set. Unset, the entry shows **Get OpenCRQ now** and opens the OpenCRQ page of the Filigran website. OpenCTI makes no call to OpenCRQ: the link is in the interface only.

When `XTM__OPENCRQ_URL` is empty, OpenCTI reads `xtm:opengrc_url` (`XTM__OPENGRC_URL`) instead: the same URL under the former name of OpenCRQ, OpenGRC, now deprecated. xtm-docker sets both to the public URL of OpenCRQ, for the OpenCTI releases up to 7.261119.0, which read only the former name.
:::

:::only{opencti="<7.261127"}
In the other direction, OpenCTI reads `xtm:opengrc_url` (`XTM__OPENGRC_URL`), the URL of OpenCRQ under its former name, OpenGRC. OpenCTI passes it to the interface, but no screen of this version uses it, and it makes no call to OpenCRQ. xtm-docker sets it to the public URL of OpenCRQ, and also sets `XTM__OPENCRQ_URL`, which this version does not read.
:::

## Give XTM Composer its tokens

XTM Composer deploys and runs the integrations that users choose in the interface of OpenCTI and OpenAEV. xtm-docker configures it from `.env`:

| Setting on `xtm-composer` | Value in xtm-docker | Role |
| --- | --- | --- |
| `MANAGER__ID` | `${XTM_COMPOSER_ID}` | The identifier of this XTM Composer, a UUID |
| `OPENCTI__ENABLE`, `OPENAEV__ENABLE` | `true`, `true` | It manages both platforms |
| `OPENCTI__URL`, `OPENAEV__URL` | `http://opencti:8080`, `http://openaev:8080` | The platforms, inside the stack |
| `OPENCTI__TOKEN`, `OPENAEV__TOKEN` | `${OPENCTI_ADMIN_TOKEN}`, `${OPENAEV_ADMIN_TOKEN}` | The token it uses on each platform |
| `OPENCTI__DAEMON__DOCKER__NETWORK_MODE`, `OPENAEV__DAEMON__DOCKER__NETWORK_MODE` | `${COMPOSE_PROJECT_NAME}_default` | The Docker network of the integrations it starts, so that they reach the platforms by service name |
| `MANAGER__CREDENTIALS_KEY_FILEPATH` | `/keys/private_key.pem` | The RSA key that the `rsa-key-generator` service creates at the first start, in the `rsakeys` volume |

Every other setting of XTM Composer is in [Configure XTM Composer](/deploy/integrations/configure-composer/).

## What's next

- [Configure XTM Composer](/deploy/integrations/configure-composer/), for its other settings.
- [Network and ports](/deploy/plan/network/), for every connection of the stack.
- [XTM Suite settings of OpenCTI](/reference/opencti/configuration/#xtm-suite), every key of its links to the other modules.
- [Compatibility matrix](/deploy/upgrade/compatibility/), for the versions that work together.
