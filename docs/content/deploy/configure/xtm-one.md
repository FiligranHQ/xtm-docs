---
title: Configure XTM One
description: Pass environment variables to XTM One in the XTM stack, and set its secrets, public URL and proxy trust, language models, edition, links to the platform, mail, security headers and sizing.
doc_type: how-to
products:
  - xtm-one
personas: [platform-engineer]
owner: docs-xtm-one
last_reviewed: 2026-11-08
migrated_from:
  - xtm-one:/deployment/configuration/
  - xtm-one:/deployment/xtm-suite-integration/
  - xtm-one:/deployment/security-headers/
---

This page sets the configuration a deployment of XTM One decides on: its secrets, its public URL and the proxy it trusts, the language models, its edition, its links to the other platforms, mail, the security headers of its pages and the size of its processes. Every setting here is an environment variable of the `xtm-one` and `xtm-one-worker` containers, read when they start. [Install XTM One](/deploy/install/xtm-one/) sets the minimum; come back here to change a setting, and read the [XTM One configuration](/reference/xtm-one/configuration/) reference for every variable.

::diagram{id=xtm-one-deploy-configure-sources}

## How XTM One reads its settings

- **Two containers.** `xtm-one` serves the interface, the API and the chat; `xtm-one-worker` runs the background work. Both read the public URL, the secret, the database, the storage and the license, and must have the same values. A variable this page gives for one container only is read by that one.
- **Read at start.** After a change, run `docker compose up -d`: Docker Compose recreates the containers whose configuration changed. A value XTM One refuses, such as a malformed list or a negative limit, stops it at start with a message that names the variable.
- **Variable or screen.** Agents, language models, single sign-on and most security options are set in **Settings** and stored in the database. Where a screen field has a variable, the variable wins when it is set: the field shows it read-only, and a change from the screen is refused.

xtm-docker passes XTM One a fixed list of variables, built from `.env`:

| In `.env` of xtm-docker | XTM One variable | Containers |
| --- | --- | --- |
| `XTM_ONE_EXTERNAL_SCHEME`, `XTM_ONE_HOST`, `XTM_ONE_PORT` | `BASE_URL`, `FRONTEND_URL` | Both |
| `XTM_ONE_ADMIN_EMAIL`, `XTM_ONE_ADMIN_PASSWORD` | `ADMIN_EMAIL`, `ADMIN_PASSWORD` | Both |
| `XTM_ONE_SECRET_KEY` | `SECRET_KEY` | Both |
| `XTM_ONE_POSTGRES_USER`, `XTM_ONE_POSTGRES_PASSWORD` | `DATABASE_URL`, on the `xtm_one` database of `pgsql` | Both |
| `MINIO_ROOT_USER`, `MINIO_ROOT_PASSWORD`, `XTM_ONE_S3_BUCKET` | `S3_ACCESS_KEY`, `S3_SECRET_KEY`, `S3_BUCKET` | Both |
| `PLATFORM_REGISTRATION_TOKEN` | `PLATFORM_REGISTRATION_TOKEN` | Both |
| `XTM_ONE_ENTERPRISE_LICENSE` | `ENTERPRISE_LICENSE` | Both |
| `XTM_ONE_LOG_LEVEL`, `XTM_ONE_LOG_FORMAT` | `LOG_LEVEL`, `LOG_FORMAT` | Both |
| Fixed | `PLATFORM_MODE=xtm_one`, `REDIS_URL=redis://redis:6379`, `S3_ENDPOINT=minio:9000`, `S3_USE_SSL=false` | Both |
| Fixed | `OPENCTI_API_URL=http://opencti:8080`, `OPENAEV_API_URL=http://openaev:8080`, `XTM_AUTH_ACCEPTED_AUDIENCES=http://xtm-one:4000` | `xtm-one` |

Any other variable goes in `docker-compose.override.yml`, under the `environment` of the service, above the `# Pinned versions` line: see [Change the stack without editing docker-compose.yml](/deploy/install/docker-compose/#change-the-stack-without-editing-docker-compose-yml). The Docker Compose file of the product passes its own list in the same way: what it does not list never reaches the containers. Its list holds every on/off switch of `.env.sample`, except `DEBUG`, `A2A_ALLOW_INSECURE_LOOPBACK` and `TRUST_PROXY_HEADERS`. It also holds the sandbox of the custom tools and the code interpreter, `CUSTOM_TOOL_SANDBOX` and the `OPENSANDBOX_*` variables: see [Run XTM One on its own host](/deploy/install/xtm-one/#run-xtm-one-on-its-own-host) for the `CUSTOM_TOOL_SANDBOX` line of `.env.sample`. A switch that is also a security option of **Settings** reaches the containers blank until `.env` sets it: the screen decides while it is blank.

## Secrets and the administrator

| Variable | Default | What to know |
| --- | --- | --- |
| `SECRET_KEY` | Required | Signs the sessions and the tokens XTM One sends, and encrypts the credentials it stores: provider keys, integration secrets, OAuth tokens. Never change it once XTM One has started. Changed or lost, every stored credential becomes unreadable and every session ends, with no way back but the old value |
| `ADMIN_EMAIL` | `admin@filigran.io` in the product | The administrator account. At every start, XTM One creates it or makes it an active administrator again |
| `ADMIN_PASSWORD` | Required | The password of that account, applied again at every start. At most 72 bytes: a longer value stops the platform |
| `ADMIN_TOKEN` | None | An API key for that account, for automation: it starts with `fcp-` and has 32 characters or more |

Because the administrator's password follows `.env` at every start, `.env` gives control of XTM One: keep it readable by its owner only (`chmod 600 .env`), out of version control, with a copy in your secret store. Change the password in `.env`, then restart.

## Public URL and reverse proxy

| Variable | Default | What to know |
| --- | --- | --- |
| `BASE_URL` | `http://localhost:4000` in the product | The public URL people open. It builds the links of emails and of single sign-on, and it is the issuer of every token XTM One signs for OpenCTI, OpenAEV and OpenCRQ: set the same value on both containers, as xtm-docker does |
| `FRONTEND_URL` | Same as `BASE_URL` | Where the browser comes back after a single sign-on. Mail clients load the images of XTM One's emails from it, under `/email-assets/` |
| `FORWARDED_ALLOW_IPS` | `127.0.0.1` | The proxies XTM One trusts, on `xtm-one`: an address, a CIDR network or a comma-separated list. It reads the client address from `X-Forwarded-For` and the scheme from `X-Forwarded-Proto` only on requests from them |
| `TRUST_PROXY_HEADERS` | `false` | Trusts the forwarded headers from any peer, and takes the leftmost `X-Forwarded-For` value, which a client can write. Leave it off and use `FORWARDED_ALLOW_IPS` |
| `BASE_PATH`, `BASE_PATH_MODE` | None, `auto` | Serve XTM One under a path, such as `/xtm-one`. `BASE_URL` and `FRONTEND_URL` then end with the same path |

Without `FORWARDED_ALLOW_IPS` naming the proxy, XTM One sees every request as plain HTTP from the proxy's address: the audit log records that address for every person, the per-address rate limits of the sign-in and password reset count everyone as one client, and the agent-to-agent endpoints refuse every request with `403 https required`. XTM One reads `FORWARDED_ALLOW_IPS` when it starts: it is not a field of **Settings** > **Access and security**. It resolves no host name, and a network must have its host bits at zero (`10.42.0.0/16`, not `10.42.0.1/16`); it logs any other entry at start, and that entry trusts nobody. Behind several proxies, such as a load balancer then an ingress, list each one. `*` trusts every peer: use it only when nothing but the proxy reaches the container.

Give XTM One its own host name, like the other platforms. Under `BASE_PATH`, the health endpoints and `/metrics` stay at the root of the container's port, and the prefix must not start with a path XTM One serves itself, such as `/api`, `/mcp` or `/xtm`: the platform refuses it at start. See the [URLs](/reference/xtm-one/configuration/#urls) of the reference.

## Language models

An administrator can add providers in **Settings** > **AI and models** > **Models**. To keep them in the configuration of the deployment instead, set them on `xtm-one`: the platform applies them at each start, and the worker reads them from the database.

| Variable | Default | What it sets |
| --- | --- | --- |
| `LLM_PROVIDER_<NAME>_API_KEY` | None | The key of a provider. `<NAME>` is `OPENAI`, `ANTHROPIC`, `GEMINI`, `MISTRAL`, `AZURE_OPENAI`, `BEDROCK`, `OPENROUTER`, `OLLAMA`, `LITELLM` or `CUSTOM_OPENAI`. The provider shows an **Env** badge and cannot be deleted from the screen: remove the variable and restart to remove it |
| `LLM_PROVIDER_<NAME>_MODELS` | Every model of the provider | The models offered, comma-separated |
| `LLM_PROVIDER_MODELS_EDITABLE` | `false` | `false`: the list of the variable is the catalog, rewritten at each start. `true`: it is a baseline that administrators can curate in **Models** |
| `MODELS_ENV_PROVIDING_ONLY` | `false` | `true` refuses every change to providers from the screen and the API |
| `DEFAULT_MODEL` | The first model configured | The default model of the agents, applied again at each start while a provider offers it |
| `BYOLLM_ACTIVATED` | `false` | Hands the language models to the administrators: `DEFAULT_*` values apply once, to an install that has no configuration of its own yet |

For `CUSTOM_OPENAI`, an endpoint that speaks the OpenAI API, the value is the key, two vertical bars, then the base URL of the endpoint. On a host without internet access, use a model served on your network, through `OLLAMA`, `LITELLM` or `CUSTOM_OPENAI`. The rest of the AI settings, such as image and speech models, are in [AI providers](/reference/xtm-one/configuration/#ai-providers).

## Edition and AI

| Variable | Default | What it sets |
| --- | --- | --- |
| `ENTERPRISE_LICENSE` | None | The Filigran license certificate, in PEM form, on both containers. It can also be pasted in **Settings** > **Platform** > **Enterprise License**; the variable then locks the field |
| `PLATFORM_MODE` | `xtm_one` | `ai_disabled` starts XTM One with AI switched off: its screens and data stay available, and no agent runs. See [Run XTM One with AI switched off](/xtm-one/administer/running-without-ai/) |

XTM One is in the Enterprise Edition with its own license, or when a registered OpenCTI or OpenAEV has one. A license certificate spans several lines: pasting it in **Enterprise License** is simpler than a variable.

## Links to the other platforms

[Connect the modules to each other](/deploy/configure/connect-modules/) registers OpenCTI and OpenAEV with XTM One and explains the public and internal URLs. On the side of XTM One:

| Variable | Default | What it sets |
| --- | --- | --- |
| `PLATFORM_REGISTRATION_TOKEN` | None | The secret every platform registers with. Unset, XTM One refuses every registration. The published default of the product's Compose file, `xtm-default-registration-token`, keeps registered platforms connected but registers no new one, and XTM One then sends the platforms no license certificate. After 10 failed attempts in a minute, a client is refused with HTTP 429 until the minute ends |
| `OPENCTI_API_URL`, `OPENAEV_API_URL`, `OPENCRQ_API_URL` | The URL each platform registers with | Where XTM One calls each platform, fetches its signing keys and, for OpenAEV, sends its callbacks. Every link people get stays on the registered public URL. A change applies at the platform's next registration, within 5 minutes |
| `XTM_AUTH_ACCEPTED_AUDIENCES` | None | The URLs the platforms reach XTM One on when they differ from `BASE_URL`, comma-separated. A platform version that addresses its tokens to that URL is refused unless it is listed |

`OPENGRC_API_URL`, the former name of `OPENCRQ_API_URL`, is read until 31 December 2026, with a warning in the logs. To add OpenCRQ, see [Install OpenCRQ](/deploy/install/opencrq/).

## Mail

XTM One sends invitations, password resets and announcements of model changes by email. Neither xtm-docker nor the product's Compose file passes it a mail server: until you set `SMTP_HOSTNAME`, XTM One sends nothing, and writes the link of a password reset in its log instead.

| Variable | Default | What it sets |
| --- | --- | --- |
| `SMTP_HOSTNAME`, `SMTP_PORT` | None, `587` | The mail relay |
| `SMTP_USERNAME`, `SMTP_PASSWORD` | None | The account, when the relay asks for one |
| `SMTP_FROM` | `SMTP_USERNAME` | The sender address |
| `SMTP_STARTTLS` | `true` | Upgrades the connection with STARTTLS, as port 587 expects. For a relay with TLS from the start, on port 465, set `SMTP_USE_TLS=true` |

Set them on both containers, and set `SMTP_FROM_NAME`, the name people see as the sender, to the name of your platform. The example at the end of this page reuses the mail settings of the stack.

## Security headers

XTM One sends a Content Security Policy on every page it serves, which tells the browser which scripts, styles and connections the page may use. Set these on `xtm-one`:

| Variable | Default | What it sets |
| --- | --- | --- |
| `CONTENT_SECURITY_POLICY_MODE` | `report-only` | `report-only` blocks nothing: browsers report what the policy would block. `enforce` blocks it. `off` sends no policy. Any other value stops the platform |
| `CONTENT_SECURITY_POLICY_FRAME_ANCESTORS` | None | The other sites allowed to show XTM One in a frame once the policy is enforced, such as an intranet portal: `https://portal.example.com`, comma-separated |

XTM One never sends `X-Frame-Options`, and your proxy must not add it: it would break the Microsoft Teams tab, the Office add-in and the sites you allow to frame XTM One. Until the policy is enforced, any site can frame XTM One. Once it is, XTM One itself, the sites you list and the registered platforms may frame it; the registered platforms only while `PLATFORM_REGISTRATION_TOKEN` is a value of your own, and at most 20 of them.

To move to `enforce`:

:::steps
1. Run in `report-only` for a few days of normal use: the chat, reports and decks, settings, and every site that embeds XTM One.
2. Read **Settings** > **Access and security** > **Security audit**: it lists the violations browsers reported in the last 7 days, by directive. The log of `xtm-one` names each new one once, in a line that starts with `CSP violation first seen`. Violations that browser extensions cause are counted apart.
3. Set `CONTENT_SECURITY_POLICY_MODE=enforce` and run `docker compose up -d`.
:::

::screenshot{id=xtm-one-settings-security-audit alt="The Security audit, which reads the live settings at each run: each finding ends on a recommendation that names the variable to set on xtm-one, such as AGENT_TOOLS_STRICT_ADMIN_ACCESS or CUSTOM_TOOL_SANDBOX, and the setting of Settings it locks."}

:::verify
```bash
curl -sI https://xtm-one.example.com/ | grep -i '^content-security-policy'
```
The header reads `content-security-policy:`, without `-report-only`. To go back, set `report-only` again: nothing else is stored.
:::

## Size the platform and the worker

| Variable | Default | What it sets |
| --- | --- | --- |
| `WEB_WORKERS` | `2` | The processes of `xtm-one` that serve the interface, the API and the chat. Raise it when many people chat at the same time: about twice the CPU cores plus one |
| `WORKER_CONCURRENCY` | `20` | The background jobs each worker container runs at once. Add worker replicas when jobs queue |
| `MAX_CONCURRENT_RUNS` | `10` | The assignment runs in parallel |
| `DB_POOL_SIZE`, `DB_MAX_OVERFLOW` | `15`, `10` | The PostgreSQL connections of each process |

Every process opens its own pools: `DB_POOL_SIZE` plus `DB_MAX_OVERFLOW` connections, and up to 10 more for the traces of agent runs, so 35 with the defaults. Count every web process and every worker replica against `max_connections` of PostgreSQL, which xtm-docker sets to `200` for XTM One and OpenAEV together, OpenAEV taking up to 20. With the defaults, XTM One takes up to 105: 35 for each of its 2 web processes and its worker.

To add worker replicas, set `deploy.replicas` on `xtm-one-worker` in `docker-compose.override.yml`, and count each one in the connections.

:::example[Send mail and serve the Grand Meridian intranet portal]
Grand Meridian Hotels shows XTM One in its intranet portal, `https://portal.grand-meridian.example`, and wants invitations to reach new members of the SOC. Marc Dupuis, of IT operations, reuses the mail relay that `.env` already gives OpenAEV, gives the four-core host one more web process per core, and adds to `docker-compose.override.yml`:

```yaml
services:
  xtm-one:
    environment:
      - SMTP_HOSTNAME=${SMTP_HOST}
      - SMTP_PORT=${SMTP_PORT}
      - SMTP_USERNAME=${SMTP_USERNAME}
      - SMTP_PASSWORD=${SMTP_PASSWORD}
      - SMTP_FROM=xtm-one@grand-meridian.example
      - SMTP_FROM_NAME=XTM One
      - CONTENT_SECURITY_POLICY_MODE=enforce
      - CONTENT_SECURITY_POLICY_FRAME_ANCESTORS=https://portal.grand-meridian.example
      - WEB_WORKERS=4
  xtm-one-worker:
    environment:
      - SMTP_HOSTNAME=${SMTP_HOST}
      - SMTP_PORT=${SMTP_PORT}
      - SMTP_USERNAME=${SMTP_USERNAME}
      - SMTP_PASSWORD=${SMTP_PASSWORD}
      - SMTP_FROM=xtm-one@grand-meridian.example
      - SMTP_FROM_NAME=XTM One
```

With 4 web processes and one worker, XTM One takes up to 175 connections, and OpenAEV 20: 195, under the `200` of `pgsql`. After `docker compose up -d`, Hélène Marchetti invites Paul Garnier from **Administration** > **Users and groups**, and his invitation arrives from `XTM One <xtm-one@grand-meridian.example>`.
:::

## What's next

- [XTM One configuration](/reference/xtm-one/configuration/), every variable with its default.
- [Monitor and troubleshoot XTM One](/deploy/operate/xtm-one/), for the metrics and the logs.
- [Set up the platform in Settings](/xtm-one/administer/settings/), for what administrators set on screen.
- [Connect the modules to each other](/deploy/configure/connect-modules/).
