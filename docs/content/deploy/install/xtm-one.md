---
title: Install XTM One
description: Take the XTM One of an XTM stack from running to usable, behind a reverse proxy it trusts, with a language model and its edition, or run it on its own host with its own Docker Compose file.
doc_type: how-to
products:
  - xtm-one
personas: [platform-engineer]
owner: docs-xtm-one
last_reviewed: 2026-11-08
migrated_from:
  - xtm-one:/deployment/installation/
---

XTM One is the AI layer at the center of the XTM suite, and every XTM stack runs it: [Install with Docker Compose](/deploy/install/docker-compose/) starts it with OpenCTI, OpenAEV, XTM Composer and the shared services. This page takes that XTM One from running to usable: what it needs, what xtm-docker sets for it, the reverse proxy it must trust, the first sign-in, a language model and its edition. Its last part runs XTM One on its own host, with the Docker Compose file of the product. At the end, people sign in at `https://xtm-one.example.com` and AI agents answer.

::diagram{id=xtm-one-deploy-install-components}

## What XTM One needs

| Component | What XTM One needs | In xtm-docker |
| --- | --- | --- |
| Images | `xtmone/platform`, the interface, the API and the chat on port 4000, and `xtmone/worker`, which runs the background work: assignments, schedules, knowledge base indexing. Both at the same version. They are published for `linux/amd64` only | The `xtm-one` and `xtm-one-worker` services, always started |
| PostgreSQL | PostgreSQL 17 with the `vector` extension (pgvector) in its database. The extension needs a superuser to create it | The shared `pgsql` service: the `xtm-one-db-init` job creates the `xtm_one` database, its role `XTM_ONE_POSTGRES_USER` and the extension at each start |
| Redis | The queue of background jobs, the live chat, locks, rate limits and quota counters. It holds working state, not your data | The shared `redis` service |
| S3 storage | A bucket for uploaded documents, chat attachments, generated files and large trace payloads. XTM One creates it when it does not exist | The shared `minio` service, bucket `XTM_ONE_S3_BUCKET` (`xtm-one-files`) |
| A language model | At least one provider XTM One reaches, such as OpenAI, Anthropic, Google Gemini, Mistral AI, Azure OpenAI or AWS Bedrock, or a model on your network through Ollama, LiteLLM or an OpenAI-compatible endpoint | None: you add it after the first sign-in |
| An edition | AI agents and the chat run with the Enterprise Edition only: a Filigran license in XTM One, or a registered OpenCTI or OpenAEV that has one | `XTM_ONE_ENTERPRISE_LICENSE`, empty in `.env.sample` |

With the Community Edition, XTM One starts and its screens open, but the chat and the agents stay off. The MCP servers of the registered platforms work in both editions. See [What each edition of XTM One brings](/xtm-one/administer/editions/).

## Before you start

- The stack installed and running as [Install with Docker Compose](/deploy/install/docker-compose/) describes, with the public URL of XTM One set (`XTM_ONE_EXTERNAL_SCHEME`, `XTM_ONE_HOST`, `XTM_ONE_PORT`) and XTM One published on `127.0.0.1:8090`. Run the commands of this page from its folder.
- A reverse proxy that serves `xtm-one.example.com` over HTTPS, as [Publish the platforms over HTTPS](/deploy/install/reverse-proxy/) describes.
- An API key of a language model provider, kept in your secret store, or the address of a model served on your network for a host without internet access.
- A Filigran license certificate for the Enterprise Edition, unless a registered OpenCTI or OpenAEV already has one.

## Check what xtm-docker sets

xtm-docker builds the whole configuration of XTM One from `.env`. Three values decide what you cannot change later without trouble:

| Variable of `.env` | What it is |
| --- | --- |
| `XTM_ONE_SECRET_KEY` | Signs the sessions and the tokens XTM One sends to the other platforms, and encrypts the credentials it stores, such as the API keys of the language models. Never change it once XTM One has started: every credential it encrypted becomes unreadable |
| `XTM_ONE_ADMIN_EMAIL` | The administrator account, created at the first start. It must be the admin email of OpenCTI and OpenAEV: XTM One finds a person in each platform by email |
| `XTM_ONE_ADMIN_PASSWORD` | The password of that account, 72 bytes at most. XTM One applies it again at every start: change it in `.env`, then restart, rather than in the interface, where a change lasts until the next restart |

:::steps
1. Check that `.env` holds your own values, not those of `.env.sample`.

   ```bash
   grep -E '^(XTM_ONE_(HOST|PORT|EXTERNAL_SCHEME|ADMIN_EMAIL|SECRET_KEY)|PLATFORM_REGISTRATION_TOKEN)=' .env
   ```

   :::verify
   `XTM_ONE_EXTERNAL_SCHEME=https`, `XTM_ONE_HOST=xtm-one.example.com`, `XTM_ONE_PORT=443`, your administrator's email, and long random values for the secret and the token, not `ChangeMeWithGeneratedRandomString`.
   :::

2. Let the reverse proxy keep long requests open and stream the chat. An agent's answer can take minutes, and Nginx closes a request after 60 seconds without data by default. Add these lines to the `location /` block of `xtm-one.example.com` in your proxy configuration, then reload it:

   ```nginx
   proxy_read_timeout 300s;
   proxy_send_timeout 300s;
   ```

   Keep `client_max_body_size` at 50 MB or more: knowledge base uploads reach that size. The chat streams its answers as server-sent events, which need three things from the proxy:

   - HTTP/2 between the browsers and the proxy (`http2 on;` in the `server` block). Every open tab of XTM One holds its streams, and over HTTP/1.1 a few tabs use up the connections a browser opens to one host: XTM One then stops loading.
   - Response buffering off for `/api/`, or each answer shows only once it is complete. Nginx follows the `X-Accel-Buffering: no` header XTM One sends with its streams.
   - A read timeout above 15 seconds: a stream with nothing to send writes a keep-alive every 15 seconds. The 300 seconds above cover it.

   The server block of [Publish the platforms over HTTPS](/deploy/install/reverse-proxy/) already serves HTTP/2 and turns buffering off. It also forwards WebSockets, which the XTM One Desktop app, the runners and the relays use; the chat does not.

3. Tell XTM One which address its reverse proxy reaches it from. Behind a proxy, every request comes from the proxy: until XTM One trusts it, the audit log records the proxy's address for every person, the sign-in rate limit counts everyone as one client, and the agent-to-agent endpoints refuse every request with `403 https required`. A proxy on the Docker host reaches the container from the gateway of the stack's network:

   ```bash
   docker network inspect xtm_default --format '{{(index .IPAM.Config 0).Gateway}}'
   ```

   `xtm` is the `COMPOSE_PROJECT_NAME` of `.env`. Write that address as `FORWARDED_ALLOW_IPS` under the `xtm-one` service of `docker-compose.override.yml`, above the `# Pinned versions` line:

   ```yaml
   services:
     xtm-one:
       environment:
         - FORWARDED_ALLOW_IPS=172.18.0.1
   ```

   `FORWARDED_ALLOW_IPS` takes an address, a network in CIDR form or a comma-separated list, and no host name. For a proxy elsewhere, name the address XTM One sees it from; behind several proxies, list each of them. See [Configure XTM One](/deploy/configure/xtm-one/#public-url-and-reverse-proxy).

4. Apply the changes, then wait until XTM One is ready.

   ```bash
   docker compose up -d
   docker compose exec xtm-one curl -sS http://localhost:4000/api/health/ready
   ```

   :::verify
   It prints `{"database":"ok","redis":"ok","s3":"ok","status":"ok"}`. The first start creates the tables and the built-in agents: until it ends, the answer reads `"status": "starting"`, with HTTP status 503. A dependency that does not answer turns its own field into an error, such as `"s3":"error: bucket not found"`. The healthcheck of xtm-docker asks `/api/health`, which answers as soon as the server listens, so `healthy` in `docker compose ps` comes before ready.
   :::

5. Open `https://xtm-one.example.com` and sign in once, then read the last request in the access log of XTM One.

   ```bash
   docker compose logs --tail 200 xtm-one | grep '"http_request"' | tail -n 1
   ```

   :::verify
   `client_ip` is the address of your workstation. The gateway address there means that `FORWARDED_ALLOW_IPS` does not name the proxy: correct it and restart `xtm-one`.
   :::
:::

## Connect a language model

:::steps
1. Sign in at `https://xtm-one.example.com` with `XTM_ONE_ADMIN_EMAIL` and `XTM_ONE_ADMIN_PASSWORD`.
2. Open **Settings** > **AI and models** > **Models**, click **Add Provider**, choose the provider, enter its API key and the models to offer. See [Set up the platform in Settings](/xtm-one/administer/settings/).
3. Add the license: **Settings** > **Platform** > **Enterprise License**, or the certificate in `XTM_ONE_ENTERPRISE_LICENSE`, which then locks the field. Skip this step when a registered OpenCTI or OpenAEV has the Enterprise Edition.
:::

To keep the provider in the configuration of the deployment instead, set its key as a variable of the `xtm-one` service: `LLM_PROVIDER_<NAME>_API_KEY`, where `<NAME>` is `OPENAI`, `ANTHROPIC`, `GEMINI`, `MISTRAL`, `AZURE_OPENAI`, `BEDROCK`, `OPENROUTER`, `OLLAMA`, `LITELLM` or `CUSTOM_OPENAI`. XTM One applies it at each start, and **Models** shows the provider with an **Env** badge. [Configure XTM One](/deploy/configure/xtm-one/#language-models) lists the variables that go with it.

::screenshot{id=xtm-one-settings-ai-models alt="The Models section of Settings, tab AI and models: the provider set by the deployment carries its Env badge and is active, its model is the default of the agents, and the routing, fallback, embedding and decision models each have their own section."}

:::verify
Open **Chat** and ask "What can you do?". The **CTEM Assistant**, the default agent of every person, answers in a few seconds. An error that names the provider or the model means the key or the model is wrong.
:::

:::example[Bring XTM One into service at Grand Meridian Hotels]
Marc Dupuis, of IT operations at Grand Meridian Hotels, installed the stack on `xtm-one.grand-meridian.example` and its sister host names, behind Nginx on the Docker host. The stack's network gateway is `172.18.0.1`. He keeps the Anthropic key in the team's secret store and adds it to `.env` as a named variable, `XTM_ONE_ANTHROPIC_API_KEY`, then writes `docker-compose.override.yml`:

```yaml
services:
  xtm-one:
    environment:
      - FORWARDED_ALLOW_IPS=172.18.0.1
      - LLM_PROVIDER_ANTHROPIC_API_KEY=${XTM_ONE_ANTHROPIC_API_KEY}
      - LLM_PROVIDER_ANTHROPIC_MODELS=claude-sonnet-5,claude-haiku-4-5-20251001
      - DEFAULT_MODEL=claude-sonnet-5
```

After `docker compose up -d`, `/api/health/ready` answers `"status":"ok"`, and Hélène Marchetti, the CISO, signs in with `XTM_ONE_ADMIN_EMAIL`, `helene.marchetti@grand-meridian.example`, the admin email of OpenCTI and OpenAEV too. **Models** shows Anthropic with its **Env** badge, and the license OpenCTI holds gives XTM One its Enterprise Edition once OpenCTI registers.
:::

## Run XTM One on its own host

Use the Docker Compose file of the product for an XTM One outside xtm-docker: on a host of its own, with the other platforms elsewhere. It brings its own PostgreSQL (`postgres`), Redis (`redis`) and S3 storage (`minio`), and starts the platform as `platform` on port 4000, the worker as `worker`, and `worker-preview`, two more replicas of the worker that share the background work.

:::steps
1. Download the files of the version, here 1.261112.0. The PostgreSQL service mounts `backend/init-pgvector.sql`, which creates the pgvector extension: without the file, Docker mounts an empty folder in its place and PostgreSQL fails to initialize.

   ```bash
   VERSION=1.261112.0
   mkdir -p xtm-one/backend && cd xtm-one
   BASE="https://raw.githubusercontent.com/XTM-One-Platform/xtm-one/${VERSION}"
   curl -fsSLO "${BASE}/docker-compose.yml"
   curl -fsSLO "${BASE}/.env.sample"
   curl -fsSL -o backend/init-pgvector.sql "${BASE}/backend/init-pgvector.sql"
   ```

2. Run the published images instead of building them. The file builds `platform`, `worker` and `worker-preview` from the sources; this override gives each one its image, and publishes port 4000 on the host itself for the reverse proxy.

   ```bash
   cat > docker-compose.override.yml <<EOF
   services:
     platform:
       image: xtmone/platform:${VERSION}
       build: !reset null
       ports: !override
         - "127.0.0.1:4000:4000"
     worker:
       image: xtmone/worker:${VERSION}
       build: !reset null
     worker-preview:
       image: xtmone/worker:${VERSION}
       build: !reset null
   EOF
   ```

3. Create `.env` with your values. `.env.sample` documents every variable; set at least these, with fresh secrets:

   ```bash
   cp .env.sample .env && chmod 600 .env
   cat >> .env <<EOF
   SECRET_KEY=$(openssl rand -base64 48 | tr -d '\n')
   ADMIN_EMAIL=admin@example.com
   ADMIN_PASSWORD=$(openssl rand -hex 24)
   BASE_URL=https://xtm-one.example.com
   FRONTEND_URL=https://xtm-one.example.com
   DB_PASSWORD=$(openssl rand -hex 24)
   S3_ACCESS_KEY=xtmone
   S3_SECRET_KEY=$(openssl rand -hex 24)
   PLATFORM_REGISTRATION_TOKEN=$(openssl rand -hex 32)
   FORWARDED_ALLOW_IPS=172.18.0.1
   EOF
   ```

   Docker Compose keeps the last value of a variable that `.env` repeats, so these lines replace those of `.env.sample`. `PLATFORM_REGISTRATION_TOKEN` is the token OpenCTI and OpenAEV register with: give them the same value. Left empty, Docker Compose runs XTM One with a published default, which registers no new platform. Name in `FORWARDED_ALLOW_IPS` the address the platform container sees your proxy from.

   `.env.sample` also sets `CUSTOM_TOOL_SANDBOX=opensandbox`, which this file passes to the containers. The code interpreter and custom tools, off by default in XTM One, then run their code in the `opensandbox` service. That service starts only with its profile. Without it, both fail once turned on, and the **Security audit** reports `Custom tool sandbox requires OpenSandbox but server is unreachable`. Choose one of the two:

   - Run OpenSandbox, which isolates each run in a container of its own. Add its profile and a fresh API key to `.env`, then download its configuration. The service mounts the Docker socket of the host to start those containers.

     ```bash
     cat >> .env <<EOF
     COMPOSE_PROFILES=opensandbox
     OPENSANDBOX_API_KEY=$(openssl rand -hex 24)
     EOF
     mkdir -p backend/sandbox
     curl -fsSL -o backend/sandbox/sandbox.docker.toml "${BASE}/backend/sandbox/sandbox.docker.toml"
     ```

   - Or add `CUSTOM_TOOL_SANDBOX=auto` to `.env`. With no Docker in its containers, XTM One then runs that code in a plain subprocess, which the **Security audit** reports as a warning once custom tools are on.

4. Start XTM One and wait until it is ready.

   ```bash
   docker compose pull
   docker compose up -d
   curl -sS http://127.0.0.1:4000/api/health/ready
   ```

   :::verify
   `docker compose ps` shows `platform`, `worker`, `worker-preview`, `postgres`, `redis` and `minio` running, with `opensandbox` too under its profile, and the readiness answer ends with `"status":"ok"`. Here, the healthcheck of `platform` asks `/api/health/ready`: `healthy` means ready.
   :::
:::

Then connect a language model as above, and register the platforms as [Connect the modules to each other](/deploy/configure/connect-modules/#register-the-platforms-with-xtm-one) describes, with `http://<host of XTM One>:4000` or its public URL as the address of XTM One. This file passes to the containers only the variables it lists. They include the sandbox variables and the on/off switches of `.env.sample`, except `DEBUG`, `A2A_ALLOW_INSECURE_LOOPBACK` and `TRUST_PROXY_HEADERS`. Any other one, such as `LLM_PROVIDER_ANTHROPIC_API_KEY`, goes under `environment` in `docker-compose.override.yml`.

## Where the data is

| Data | Where, in xtm-docker |
| --- | --- |
| Agents, conversations, knowledge base chunks and their vectors, assignments, traces, settings, and the encrypted credentials | The `xtm_one` database of `pgsql`, in the `pgsqlshareddata` volume |
| Uploaded documents, chat attachments, generated reports and decks, large trace payloads | The `xtm-one-files` bucket of `minio`, in the `s3data` volume |
| Queued jobs, the state of running chats, rate limits | Redis, in the `redisdata` volume. Working state: XTM One rebuilds it |

Back them up with the rest of the stack, together with `.env`: without the same `XTM_ONE_SECRET_KEY`, a restored XTM One cannot read the credentials it stores. See [Back up and restore](/deploy/operate/backup/).

## What's next

- [Configure XTM One](/deploy/configure/xtm-one/), for the language models, mail, security headers and sizing.
- [Connect the modules to each other](/deploy/configure/connect-modules/), to register OpenCTI and OpenAEV with XTM One.
- [Monitor and troubleshoot XTM One](/deploy/operate/xtm-one/), for its health, metrics and logs.
- [Set up the platform in Settings](/xtm-one/administer/settings/), for the administrator who takes over.
