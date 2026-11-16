---
title: Breaking changes
description: The changes of OpenCTI, OpenAEV, XTM One and the XTM Docker deployment that need an action when you upgrade, per version, with the guide for each.
doc_type: reference
products:
  - suite
  - opencti
  - openaev
  - xtm-one
personas: [platform-engineer, developer]
owner: docs-platform
last_reviewed: 2026-11-12
migrated_from:
  - opencti:/deployment/breaking-changes/
  - openaev:/deployment/breaking-changes/
---

This page lists, per product and version, the changes that need an action when you upgrade: a setting to add, a script to update, a module to upgrade at the same time. Read the entries between your current version and the one you upgrade to, newest first, before you follow [Upgrade](/deploy/upgrade/).

::diagram{id=deploy-upgrade-breaking-changes}

A change is **deprecated** in the version that announces it, while the old behavior still works, and **changed** in the version where the old behavior stops.

## XTM Docker deployment

| Change | What to do |
| --- | --- |
| `docker-compose.yml` includes one file per shared service, in `infra/`, and one per module, in `modules/`. The services, their settings and the volume names do not change, and the data is kept. A `git pull` over a changed `docker-compose.yml` refuses to merge | Move your changes to `docker-compose.override.yml` before you pull: [Stay on a version or an LTS line](/deploy/upgrade/#stay-on-a-version-or-an-lts-line) |
| The whole stack always starts, and XTM Composer always manages both OpenCTI and OpenAEV. A `COMPOSE_PROFILES` line, `XTM_COMPOSER_OPENCTI_ENABLE` or `XTM_COMPOSER_OPENAEV_ENABLE` left in `.env` does nothing | Remove these lines from `.env`: step 2 of [Upgrade the stack](/deploy/upgrade/#upgrade-the-stack) |
| OpenAEV and XTM One share one PostgreSQL instance, on a new volume. The new version starts with empty OpenAEV and XTM One databases unless you move the data | Dump both databases before you pull the new version: [Move to the shared PostgreSQL instance](/deploy/upgrade/#move-to-the-shared-postgresql-instance) |

## OpenCTI

| Change | Deprecated in | Changed in |
| --- | --- | --- |
| [Removal of the filter script](#opencti-7-260401-0) | | 7.260401.0 |
| [SSO in the Enterprise Edition, configured in the UI](/deploy/upgrade/breaking-changes/opencti/7-260224-sso-migration/) | | 7.260224.0 |
| [Encryption key, tokens, XTM Composer and OpenAEV versions](#opencti-7-260224-0) | | 7.260224.0 |
| [Metrics renamed in client-python](/deploy/upgrade/breaking-changes/opencti/6-8-client-python-metrics/) | | 6.8 |
| [Scenario generation placeholders](/deploy/upgrade/breaking-changes/opencti/6-5-scenario-generation/) | 6.5 | 6.8 |
| [Promote to indicator](/deploy/upgrade/breaking-changes/opencti/6-2-promote-to-indicator/) | 6.2 | 6.8 |
| [Stream connectors data import](/deploy/upgrade/breaking-changes/opencti/6-3-stream-connectors/) | 6.3 | 6.6 |
| [Functions allowed in webhook templates](#opencti-6-4-11) | | 6.4.11 |
| [SAML authentication](/deploy/upgrade/breaking-changes/opencti/6-2-saml-authentication/) | | 6.2 |
| [New filter API](/deploy/upgrade/breaking-changes/opencti/5-12-filters/) | | 5.12 |

### OpenCTI 7.260401.0

**Removal of the filter script.** The GraphQL API refuses the filter script option by default, for security. A script that uses it gets the error `Filter script is not allowed`. To allow it again, at your own risk, set `elasticsearch:unsecure_script_filter_enabled` to `true`: `ELASTICSEARCH__UNSECURE_SCRIPT_FILTER_ENABLED=true` as an environment variable.

### OpenCTI 7.260224.0

**SSO needs the Enterprise Edition and moves to the UI.** A Community Edition platform can no longer sign in with an SSO configured in the configuration file. Local authentication, with password policies and two-factor authentication, stays available. OpenCTI converts the authentications of the configuration file into authentications managed in the UI. See [SSO migration from environment variables to UI](/deploy/upgrade/breaking-changes/opencti/7-260224-sso-migration/).

**OpenAEV and XTM Composer versions.** OpenCTI 7.260224.0 needs OpenAEV 2.2 or later and XTM Composer 2 or later. XTM Composer 2 supports the new token management and the JSON Web Token (JWT) authentication of OpenCTI 7. See the [compatibility matrix](/deploy/upgrade/compatibility/).

**An encryption key is mandatory.** Set `app:encryption_key` to a random key of at least 32 bytes, encoded in base64, and keep it secret. Generate one with `openssl rand -base64 32` and set it as `APP__ENCRYPTION_KEY`. xtm-docker sets it from `OPENCTI_ENCRYPTION_KEY` in `.env`.

**Tokens.** Each user can have several API tokens, stored hashed (HMAC), with expiration and usage tracking. Connectors authenticate to the platform with JSON Web Tokens signed by keys the platform derives (Ed25519). OpenCTI migrates the existing tokens; after the migration, a user can no longer read their token back. Update the scripts that read or renew tokens:

| Before | Now |
| --- | --- |
| GraphQL `User.api_token` (`String!`) | `User.api_tokens` (`[ApiToken!]!`) |
| GraphQL `MeUser.api_token` | `MeUser.api_tokens` |
| GraphQL `meTokenRenew` and `UserEditMutations.tokenRenew` | Removed |
| Python client `MeUser.api_token` | `MeUser.api_tokens` |
| Python client `token_renew()` | `create_token()` and `remove_token()` |

**Python 3.9.** Python 3.9 is no longer supported. Review the scripts that still run with it.

### OpenCTI 6.4.11

**Functions allowed in webhook templates.** A webhook template can only use the functions of an allowed list. You cannot create a template that uses another function, or update an existing template with one. In the versions this documentation describes, templates run in a sandbox whose list is fixed: the standard JavaScript objects and functions, such as `Date`, `JSON`, `Math`, `String`, `Array`, `parseInt` and `encodeURIComponent`. The `APP__NOTIFIER_AUTHORIZED_FUNCTIONS` setting of 6.4.11 no longer exists: remove it from your configuration. A template that uses another name fails with `Forbidden variable access`. See [Write a template](/opencti/configure/notifiers/#write-a-template).

## OpenAEV

| Change | Deprecated in | Changed in |
| --- | --- | --- |
| [CSRF token enforcement](/deploy/upgrade/breaking-changes/openaev/2-3-4-csrf-token-enforcement/) | | 2.3.4 |
| [Scenario generation from OpenCTI security coverage](/deploy/upgrade/breaking-changes/openaev/2-2-0-security-coverage/) | | 2.2.0, fixed in 2.2.1 |
| [Password encryption](/deploy/upgrade/breaking-changes/openaev/2-1-0-password-encryption/) | | 2.1.0 |
| [OpenAEV renaming](/deploy/upgrade/breaking-changes/openaev/2-0-0-openaev-renaming/) | OpenBAS 1.18.20 | 2.0.0 |

- **2.3.4**: API calls from the OpenAEV interface carry a Cross-Site Request Forgery (CSRF) token. Upgrade the collectors, injectors, OpenAEV agents and custom integrations together with the platform.
- **2.2.0**: OpenAEV 2.2.0 works only with OpenCTI 7, and OpenCTI 7 only with OpenAEV 2.2.0 or later. OpenAEV 2.2.1 works with older OpenCTI versions again.
- **2.1.0**: two settings become mandatory, `openaev.admin.encryption_key` and `openaev.admin.encryption_salt`, to encrypt the secrets of the integrations stored in the database.
- **2.0.0**: OpenBAS becomes OpenAEV. Images, configuration variables, the default bucket, queues and indices are renamed.

## XTM One

| Change | Deprecated in | Changed in |
| --- | --- | --- |
| [Chat over HTTP: HTTP/2 at the reverse proxy](#xtm-one-1-261112-0) | | 1.261112.0 |
| [Chat WebSocket removed from the API](#xtm-one-1-261112-0) | | 1.261112.0 |
| [GitHub Copilot and Cursor removed from AI adoption](#xtm-one-1-261112-0) | | 1.261112.0 |
| [On/off switches of `.env` reach the containers](#xtm-one-1-261110-2) | | 1.261110.2 |
| [Custom tool sandbox of `.env.sample` takes effect](#xtm-one-1-261110-2) | | 1.261110.2 |
| [New OpenSandbox images](#xtm-one-1-261110-2) | | 1.261110.2 |
| [Sign-in through platform registration only](#xtm-one-1-261110-2) | | 1.261110.2 |
| [Slack senders identified by their own member ID](#xtm-one-1-261110-2) | | 1.261110.2 |

### XTM One 1.261112.0

**Chat over HTTP: HTTP/2 at the reverse proxy.** No migration. The web chat, the rails of the deck and video studios, voice mode and the Office add-in no longer use a WebSocket: they stream over HTTP, as server-sent events, under `/api/v1/web/chat`. Every open tab holds its own streams, and a browser opens 6 HTTP/1.1 connections per host at most. Behind a proxy that speaks only HTTP/1.1 to browsers, a few open tabs use them all up and the app stops loading. Before you upgrade, check three settings of the proxy in front of XTM One:

- HTTP/2 between browsers and the proxy. The proxy may still reach XTM One over HTTP/1.1.
- No response buffering for `/api/`, or an answer shows only once it is complete. Nginx honours the `X-Accel-Buffering: no` header XTM One sends on the chat streams; another proxy needs the setting.
- A read timeout above 15 seconds: an idle stream writes a keep-alive every 15 seconds. An agent's answer can take minutes, so keep 300 seconds.

In Nginx 1.25.1 and later, the server block of XTM One in [Publish the platforms over HTTPS](/deploy/install/reverse-proxy/) becomes:

```nginx
server {
    listen 443 ssl;
    http2 on;
    server_name xtm-one.example.com;
    # certificates, client_max_body_size and location / as before, with:
    #   proxy_buffering    off;
    #   proxy_read_timeout 300s;
}
```

Before Nginx 1.25.1, write `listen 443 ssl http2;` instead of the `http2 on;` line. The proxy keeps forwarding the WebSocket upgrade: XTM One Desktop, runners and relays still connect over WebSocket, and fall back to slower HTTP polling without it.

**Chat WebSocket removed from the API.** A script that opened `WS /api/v1/chat/conversations/{id}/ws` must move to the HTTP routes:

| Before | Now |
| --- | --- |
| Open the WebSocket with the session token in `?token=` | Send the session token as `Authorization: Bearer`. These routes accept a web session only, and refuse an API key or a platform token with `403` |
| Send a message frame on the socket | `POST /api/v1/web/chat/messages`, with `content` and `conversation_id`. The response streams the same frames as server-sent events |
| Keep the socket open to follow the conversation | `GET /api/v1/web/chat/conversations/{id}/events` |
| Stop, Continue now, steering and approvals as frames | A `POST` each, under `/api/v1/web/chat/messages/`: `cancel`, `skip-wait`, `steer` and `approve` |
| `ping` and `pong` frames | None. An idle stream writes a keep-alive comment every 15 seconds |
| Close code `4003` | HTTP `403`, with `code` `chat_unavailable` or `terms_required` |
| An `error` frame for a send refused before it starts | An HTTP error with `detail` |
| The session ends while the socket is open | A `session_expired` frame, then the end of the stream |

A script that authenticates with an API key keeps `POST /api/v1/chat/conversations/{id}/messages`, which is unchanged. See the [XTM One chat API](/develop/apis/chat/).

**GitHub Copilot and Cursor removed from AI adoption.** **AI Adoption** now follows the agentic AI of XTM One alone: it keeps its **Overview** and **Adoption Gaps** tabs, and its configuration card leaves **Settings** > **Usage and quotas** > **Observability**. The migration `x11380devadopt1` deletes the GitHub Copilot and Cursor keys, the mapping of people to their GitHub logins and Cursor emails, and the metrics collected from both. The routes under `/api/v1/ai-external-keys` are removed. A rollback with the downgrade does not bring the data back: only the database backup taken before the upgrade does. The same release removes focus mode: the focus mode choices of people and groups are no longer used, and `x11380focdrop1` drops them. No action is needed for either.

### XTM One 1.261110.2

**On/off switches of `.env` reach the containers.** This concerns the Docker Compose file of the product, which [Run XTM One on its own host](/deploy/install/xtm-one/#run-xtm-one-on-its-own-host) uses. xtm-docker and Kubernetes deployments are unchanged. Before, that file passed a fixed list of variables, so most on/off switches of `.env.sample` did nothing in `.env`. It now passes them all to `platform`, `worker` and `worker-preview`: `DISABLE_CUSTOM_TOOLS`, `MCP_DCR_ENABLED`, `PROMETHEUS_ENABLED`, `ASSIGNMENT_QUEUE_ENABLED` and the others. Only `DEBUG`, `A2A_ALLOW_INSECURE_LOOPBACK` and `TRUST_PROXY_HEADERS` stay out.

A switch your `.env` sets applies at the next `docker compose up -d`. A switch that is also in **Administration** > **Settings** > **Access and security** of XTM One then shows read-only there: the variable wins until you remove it. A blank value counts as unset, and no longer stops the startup. Before you upgrade, list the switches your `.env` sets, and remove those you do not want applied:

```bash
grep -E '^[A-Z_]+=(true|false)' .env
```

**The custom tool sandbox of `.env.sample` takes effect.** The same file now also passes `CUSTOM_TOOL_SANDBOX` and the `OPENSANDBOX_*` variables. `.env.sample` sets `CUSTOM_TOOL_SANDBOX=opensandbox`, which had no effect in 1.261030.0. Once you turn them on, the code interpreter and custom tools fail unless the `opensandbox` service runs: `Custom tool sandbox is set to 'opensandbox' but the OpenSandbox server is not reachable`. Run that service with its profile, or set `CUSTOM_TOOL_SANDBOX=auto`: step 3 of [Run XTM One on its own host](/deploy/install/xtm-one/#run-xtm-one-on-its-own-host) gives both. A `docker-compose.override.yml` that passed these variables to `platform`, `worker` and `worker-preview` can drop them.

**New OpenSandbox images.** By default, XTM One now asks the OpenSandbox server for `sandbox-python:0.5` and `sandbox-nodejs:0.4`, instead of `0.3` for both. The server must pull or find them. They are published as `xtmone/sandbox-python` and `xtmone/sandbox-nodejs`, or built with `backend/sandbox/build-image.sh`. Otherwise, sandbox creation fails. To keep the previous images, set `OPENSANDBOX_IMAGES=python:sandbox-python:0.3,nodejs:sandbox-nodejs:0.3` on the platform and the worker: `0.3` stays published. On Kubernetes, a pool's image is set on the pool, not by `OPENSANDBOX_IMAGES`.

**Sign-in through platform registration only.** In 1.261030.0, an application registered at `/mcp/oauth/register` with the `openid` scope got ID tokens, and could sign people in with their XTM One account. It no longer gets them: only the sign-in client of a registered platform, `xtm-platform-<identifier>`, gets OpenID Connect. Register such an application as a platform instead, with `POST /api/v1/platform/register`:

- Authenticate with `PLATFORM_REGISTRATION_TOKEN`. The published default token gets no sign-in client.
- Send its callbacks in `sign_in_redirect_uris`, on the origin of its own `platform_url`. The answer gives its client ID in `sign_in_client_id`.
- Sending `[]` removes the client, and so does unregistering or purging the platform. A platform whose registration is not active signs nobody in.

Such a sign-in shows no consent page, and the audit log records it with `"consent": false`. Its token answer has no `refresh_token`. Its `access_token` is bound to the client, and no Model Context Protocol (MCP) endpoint accepts it: rely on the ID token only. An administrator who impersonates someone is never signed in to a product as them. MCP clients have nothing to change.

**Slack senders identified by their own member ID.** No action is needed. Before, the Slack bot identified a sender by the channel they wrote in, and could answer a member as another account. The migration `s5811slackid01` removes the identities that name a channel (`slack:C…`, `slack:G…`, `slack:D…`) from the users' linked identities. The member IDs an administrator linked (`slack:U…`, `slack:W…`) stay. XTM One links a member again on their next message whose email matches an account. A member whose email matches none is unidentified, or anonymous where the bot allows it. See [How the bot knows who is talking](/xtm-one/administer/channels/#how-the-bot-knows-who-is-talking).

## What's next

- [Upgrade](/deploy/upgrade/), the procedure itself.
- [Compatibility matrix](/deploy/upgrade/compatibility/), the versions that must move together.
- [Versioning and LTS](/deploy/upgrade/lifecycle/), how versions are numbered.
