---
title: CSRF token enforcement
description: From OpenAEV 2.3.4, API calls from the interface carry a CSRF token. Upgrade the integrations and agents together with the platform to avoid 401 and 403 errors.
doc_type: how-to
products:
  - openaev
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - openaev:/deployment/breaking-changes/2.3.4-csrf-token-enforcement/
---

From OpenAEV 2.3.4, every API call that the OpenAEV interface makes must carry a valid Cross-Site Request Forgery (CSRF) token. It protects signed-in sessions, and every component that calls the OpenAEV API through a signed-in web session must support it. This page says what to upgrade together and how to check the result.

::diagram{id=breaking-openaev-2-3-4-csrf-token-enforcement}

| | Version |
| --- | --- |
| Introduced in | OpenAEV 2.3.4 |

## What breaks with mixed versions

When the platform is on 2.3.4 or later and some components are older, API calls can fail at sign-in or when they change data:

- `401 Unauthorized` responses.
- `403 Forbidden` responses, for a missing or invalid CSRF token.
- Integrations that fail to create, update or trigger operations through the OpenAEV API.

## Upgrade together

In the same maintenance window, upgrade the platform and every component that calls it, to versions released for OpenAEV 2.3.4 or later:

- injectors
- collectors
- OpenAEV agents
- other integrations
- your own integrations that use the API

:::warning
Do not run mixed versions in production, for example OpenAEV 2.3.4 with older injectors or collectors.
:::

With xtm-docker, the default collectors and injectors (the `openaev-integrations` profile) use the `latest` tag: `docker compose pull` and `docker compose up -d` upgrade them with the platform. See [Upgrade](/deploy/upgrade/).

## Check that it works

:::steps
1. Sign in to OpenAEV and run a standard action, such as launching an inject.

   :::verify
   The action succeeds, with no authentication error.
   :::

2. Look for `401` and `403` responses in the logs of the platform and of the integrations.

   ```bash
   docker compose logs --since 1h | grep -wE '401|403'
   ```

   :::verify
   None of the lines it prints is an OpenAEV API call that failed, and no component reports a failed API connection.
   :::
:::

## What's next

- [Upgrade](/deploy/upgrade/), the upgrade procedure.
- [How integrations run](/deploy/integrations/how-they-run/), to upgrade integrations you deployed yourself.
- [Breaking changes](/deploy/upgrade/breaking-changes/), the other changes per version.
