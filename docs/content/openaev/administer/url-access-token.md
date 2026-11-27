---
title: Protect the links in player emails
description: How OpenAEV protects the links of media pressure, challenge and lessons learned emails with short-lived tokens, how long they last, and how to set their expiry and retention.
doc_type: how-to
products:
  - openaev
personas: [platform-engineer, admin, exercise-planner]
owner: docs-openaev
last_reviewed: 2026-11-07
no_screenshot: "URL access tokens work behind the links of player emails, with no screen of their own: the diagram shows the path of a link."
migrated_from:
  - openaev:/deployment/platform/url-access-token/
---

The emails OpenAEV sends to players during a simulation carry links: to an article of media pressure, to a challenge, to the lessons learned questionnaire. Each link holds a URL access token, an opaque value that opens that page for that player in that simulation only, and expires after the simulation. This page explains how the tokens work, how long they last, and the two settings that tune them.

## Before you start

- You can change the environment of the `openaev` service and restart it.
- `openaev.base-url` (`OPENAEV_BASE-URL`) is the public address players reach OpenAEV at: the links are built from it. See the [openaev block](/reference/openaev/configuration/#openaev).

## How a link works

::diagram{id=openaev-url-access-token-flow}

1. When an inject sends a media pressure, challenge or lessons learned email, OpenAEV creates a token for the player and the simulation, and puts a link `/url/access?token=…` in the email. It keeps only a SHA-256 hash of the token.
2. The player clicks. OpenAEV checks the token: known, not expired, not revoked.
3. OpenAEV sets the token in an `HttpOnly` cookie with `SameSite=Strict`, and redirects the player to the page, at an address without the token. The token never stays in the browser history, in the logs of a proxy, or in the `Referer` sent to another site.

A token that is expired or revoked is refused, and deleted at once.

## How long a link lasts

A token expires a number of days after the end of its simulation, 7 by default. A simulation that has not ended yet when the email is sent has no end date: the token then expires 7 days after it is sent. In a simulation that lasts more than a week, a player who opens an old email after that is refused, and needs a new email.

Revoked and expired tokens stay in the database for audit, then a job deletes them every Sunday at 2:00, once they are older than the retention period, 30 days by default.

| Key | Environment variable | Default | What it sets |
| --- | --- | --- | --- |
| `openaev.url.access.token.expiry-margin-days` | `OPENAEV_URL_ACCESS_TOKEN_EXPIRY-MARGIN-DAYS` | `7` | The days a token lasts after the end of its simulation |
| `openaev.url.access.token.retention-days` | `OPENAEV_URL_ACCESS_TOKEN_RETENTION-DAYS` | `30` | The days an expired or revoked token is kept before the purge |

Both are in the [openaev.url block](/reference/openaev/configuration/#openaev-url) of the configuration reference.

## Mark the cookie secure

The cookie carries the `Secure` flag, which keeps the browser from sending it over plain HTTP, only when `openaev.cookie-secure` (`OPENAEV_COOKIE-SECURE`) is `true`. It is `false` by default. Serve OpenAEV over HTTPS, as [Publish the platforms over HTTPS](/deploy/install/reverse-proxy/) describes, and turn it on: the session cookie of OpenAEV follows the same key.

:::example[Keep the links of a long crisis exercise working]
Example CERT runs a two-week crisis exercise for Grand Meridian Hotels: the fake booking request for the European Hospitality Summit 2026 reaches the night auditors, then media pressure articles follow for ten days. Links sent on day one would expire on day eight, while the exercise still runs. The platform engineer lengthens the margin, and keeps the tokens three months for the audit of the exercise:

```yaml
services:
  openaev:
    environment:
      OPENAEV_URL_ACCESS_TOKEN_EXPIRY-MARGIN-DAYS: "21"
      OPENAEV_URL_ACCESS_TOKEN_RETENTION-DAYS: "90"
      OPENAEV_COOKIE-SECURE: "true"
```

After a restart, the links of every new email of the running exercise last 21 days after they are sent. Links already sent keep the expiry they had.
:::

## Check that it works

Send a media pressure email to a test player and open it. The link has the form `https://openaev.example.com/url/access?token=…`. Opening it shows the article, at an address without the token. Opened again after the token has expired, the link is refused.

## What's next

- [Publish the platforms over HTTPS](/deploy/install/reverse-proxy/).
- [Harden the deployment](/deploy/operate/harden/).
- The [openaev block](/reference/openaev/configuration/#openaev), for `openaev.cookie-secure` and the session cookie.
