---
title: Cookies
description: The two browser cookies OpenCTI sets, what each one does, how long it lives, the configuration keys that control it, and when it is removed.
sidebar_position: 26
doc_type: reference
products:
  - opencti
personas: [admin, platform-engineer]
owner: docs-opencti
no_screenshot: "Cookies live in the browser and the server configuration; OpenCTI has no screen for them."
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/cookies/
---

OpenCTI uses cookies only for sessions in the browser. API clients, such as the Python client `pycti`, connectors and workers, send an API token in the `Authorization` header and get no cookie. This page lists the cookies, the settings that control them, and when they are removed.

## The cookies

| Cookie | Purpose | Lifetime |
| --- | --- | --- |
| `opencti_session` | Keeps the user logged in after any login: local, single sign-on, LDAP, HTTP headers or certificate | `app:session_timeout`, 20 minutes by default, renewed at every request |
| `opencti_flash` | Carries an error to the interface after a request that redirects the browser fails, such as a single sign-on, certificate or HTTP headers login | 10 seconds |

Both are needed for the platform to work; neither is used for anything but the session. The session cookie is `HttpOnly`: scripts in the page cannot read it.

## How the session cookie works

::diagram{id=opencti-cookies-session}

1. The user logs in. The server creates a session and sends the `opencti_session` cookie. With `app:session_manager` set to `shared`, the default, the session is stored in Redis; otherwise it stays in the memory of the platform node.
2. The browser sends the cookie with every request, and the server checks the session.
3. Every request pushes the expiry back by `app:session_timeout`: the session only ends after that long without activity.

With several platform nodes, keep the shared session manager so every node sees the same sessions: see [Scale](/deploy/operate/scale/).

## Settings

| Key | Default | Effect |
| --- | --- | --- |
| `app:session_timeout` | `1200000` (20 minutes) | How long, in milliseconds, a session lasts without activity |
| `app:session_idle_timeout` | `0` (off) | How long, in milliseconds, the interface waits without activity before it locks the screen |
| `app:session_cookie` | `false` | When `true`, the cookie has no expiry date and the browser removes it when it closes |
| `app:session_manager` | `shared` | Where sessions are stored: `shared` for Redis, any other value for the memory of the node |
| `app:https_cert:cookie_secure` | `false` | When `true`, the browser sends the cookies over HTTPS only |
| `app:https_cert:cookie_samesite` | `lax` | The `SameSite` attribute of the session cookie: `strict`, `lax` or `none`. Browsers accept `none` only with `cookie_secure` set to `true`, and `strict` can break the return from a single sign-on provider |

The four `app:session_*` keys are in [Basic parameters](/reference/opencti/configuration/#basic-parameters), the two `app:https_cert:*` keys in [Network and security](/reference/opencti/configuration/#network-and-security).

:::warning
Set `APP__HTTPS_CERT__COOKIE_SECURE=true` whenever users reach OpenCTI over HTTPS, including behind a reverse proxy that terminates TLS. Otherwise the session cookie can travel over an unencrypted connection.
:::

For example, a one-hour session over HTTPS:

```bash
APP__SESSION_TIMEOUT=3600000
APP__HTTPS_CERT__COOKIE_SECURE=true
APP__HTTPS_CERT__COOKIE_SAMESITE=lax
```

## When the cookies are removed

| Event | What happens |
| --- | --- |
| The user logs out | The server ends the session and tells the browser to delete the cookie |
| The session expires | The cookie no longer matches a session; the next request fails and the interface returns to the login page |
| An administrator kills the session | Same as expiry, from :menu[Settings > Security > Sessions] or the user page. See [End sessions](/opencti/administer/users/#end-sessions) |
| The browser closes, with `app:session_cookie` set to `true` | The browser deletes the cookie |
| 10 seconds pass after an error | The browser deletes `opencti_flash` |

## What's next

- [Configure single sign-on](/opencti/administer/sso/), for the ways users log in.
- [Manage login providers in the UI](/opencti/administer/authentications/#set-platform-wide-login-policies), to limit concurrent sessions.
- [OpenCTI configuration reference](/reference/opencti/configuration/), for every key.
