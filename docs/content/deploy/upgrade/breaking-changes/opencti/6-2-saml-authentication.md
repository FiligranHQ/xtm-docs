---
title: SAML authentication
description: In OpenCTI 6.2, SAML requires signed responses and assertions by default. Set two parameters to false if sign-in fails after the upgrade.
doc_type: how-to
products:
  - opencti
personas: [platform-engineer, admin]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - opencti:/deployment/breaking-changes/6.2-saml-authentication/
---

OpenCTI 6.2 upgraded the `passport-saml` library, which changed two SAML defaults: responses and assertions must now be signed. If SAML sign-in fails after you upgrade from an earlier version, this page says what to change in your identity provider, and which parameter to set when it cannot sign.

::diagram{id=breaking-opencti-6-2-saml-authentication}

| | Version |
| --- | --- |
| Changed in | OpenCTI 6.2 |

## What changed

| SAML parameter | Default before 6.2 | Default from 6.2 |
| --- | --- | --- |
| `want_assertions_signed` | `false` | `true` |
| `want_authn_response_signed` | `false` | `true` |

The new defaults apply when your configuration does not set the parameter.

## What to do

If users cannot sign in with SAML after the upgrade, your identity provider does not sign its responses, its assertions, or both. Turn signing on for both in the identity provider, then try again: the signatures are what lets OpenCTI trust who signs in.

Set a parameter to `false` only for what your identity provider cannot sign, for example `want_authn_response_signed` when it signs only its assertions: OpenCTI then accepts that part unsigned.

From OpenCTI 7.260224.0, SAML is configured in the interface, where the two parameters are the switches **Want assertion signed** and **Want authn response signed** of the provider: see [SSO migration from environment variables to UI](/deploy/upgrade/breaking-changes/opencti/7-260224-sso-migration/) and [Add a SAML provider](/opencti/administer/authentications/#add-a-saml-provider).

:::warning[Check the two switches after the SSO migration]
After the SSO migration of OpenCTI 7.260224.0, open the provider in :menu[Settings > Security > Authentications] and check **Want assertion signed** and **Want authn response signed**: turn them on if your identity provider signs its responses and assertions.
:::

## What's next

- [Configure single sign-on](/opencti/administer/sso/), the SAML parameters.
- [Breaking changes](/deploy/upgrade/breaking-changes/), the other changes per version.
