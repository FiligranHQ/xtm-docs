---
title: OpenCRQ API
description: Every procedure of the OpenCRQ API in this version, its kind and its input, generated
  from the source code.
sidebar_label: API
doc_type: reference
products:
  - opencrq
personas: [developer]
generated: true
generator: reference:opencrq:api
---

::diagram{id=reference-opencrq-api}

The interface of OpenCRQ and the agents of XTM One call the same procedures. A script authenticates as a service account, created by an administrator in its tenant, with the token it gets: it then acts in that tenant with the rights of the account's groups. Some procedures need a signed-in person and refuse a token: `getMe`, `signOut` and those of `edition`, `xtmOne`, `events` and `members`, the writes of `cwe` and `controlCapabilities`, `tenants.setTenantDefaultLocale`, `dashboard.setTenantHomeDashboard` and `organizations.setOrganizationPinned`. A service account never holds a platform capability either. `health`, `isLoggedIn` and `authStatus` answer without authentication.

A few HTTP routes live outside `/trpc` and are not listed here: the STIX bundle upload (`/api/threats/upload`), file downloads, the sign-in through XTM One, the chat with its agents (`/api/xtm-one/chat/...`) and the peer token provisioning (`/api/xtm/peer-tokens`, see `APP__ADMIN__TOKEN` in the [configuration](/reference/opencrq/configuration/)).
