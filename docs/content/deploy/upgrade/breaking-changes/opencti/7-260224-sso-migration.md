---
title: SSO migration from environment variables to UI
description: What OpenCTI 7.260224.0 does with the SSO configured in environment variables, and how to keep external secrets and sign-in access through the migration.
doc_type: how-to
products:
  - opencti
personas: [platform-engineer, admin]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - opencti:/deployment/breaking-changes/7.260224.0-SSO-authentication-migration/
---

From OpenCTI 7.260224.0, single sign-on (SSO) is configured in the interface and is an Enterprise Edition feature. On its first start, the new version copies the authentications of the configuration file into the database. This page explains what is migrated, and how to keep secrets outside the database and keep a way to sign in.

::diagram{id=breaking-opencti-7-260224-sso-migration}

| | Version |
| --- | --- |
| Changed in | OpenCTI 7.260224.0 |

## What the migration does

When the platform starts, it reads the authentications of the configuration file, or of the environment variables, and stores them in the database. They stay in the configuration file. The platform then loads the authentications of the database, and users sign in with them as before.

| Configured before | Managed in the UI as |
| --- | --- |
| LDAP | LDAP |
| SAML | SAML |
| OpenID Connect | OpenID Connect |
| Auth0, Google, Facebook, GitHub | OpenID Connect |
| Client certificate | Client certificate |
| HTTP headers | HTTP headers |

Secret fields, such as an OpenID Connect client secret, a SAML private key or LDAP bind credentials, are read at migration time and stored in the database, encrypted.

A SAML configuration that did not set `want_assertions_signed` or `want_authn_response_signed` is migrated with both switches off: see [SAML authentication](/deploy/upgrade/breaking-changes/opencti/6-2-saml-authentication/) to turn them back on.

:::note[No platform is locked out]
If no other authentication is enabled before the migration, the migration enables local authentication. A Community Edition platform that had turned local authentication off still has a way to sign in.
:::

## Before you start

- Plan to sign in with local authentication right after the upgrade: keep it enabled, with an administrator account whose password you know.
- On a Community Edition platform, SSO stops working: sign in with local authentication, or request a trial license from [XTM Hub](https://hub.filigran.io/cybersecurity-solutions/opencti-free-trial). See [Editions and licensing](/start/editions/).

## Keep secrets outside the database

Follow these steps if a secret must keep coming from an environment variable or a credentials provider such as CyberArk, instead of the database.

:::steps
1. Before the upgrade, declare each secret in the secrets registry of the configuration.

   - From an environment variable: `SECRETS__<NAME>__VALUE`.
   - From CyberArk: `SECRETS__<NAME>__CREDENTIALS_PROVIDER__SELECTOR=cyberark` and the CyberArk variables of that secret.

   See [Manage login providers in the UI](/opencti/administer/authentications/) for the variables of each provider.

2. Optional: to never store the secret in the database, remove it from the provider configuration before the upgrade, for example `PROVIDERS__OIC__CONFIG__CLIENT_SECRET`. The migration then creates the provider with a placeholder.

3. Upgrade and start the platform. The migration runs.

4. Sign in with local authentication and go to :menu[Settings > Security > Authentications].

5. Open each migrated provider that should use an external secret. For each secret field, choose **Use external secret**, select the name you declared in step 1, and save. The platform now reads the value from the environment or from CyberArk when it loads the provider.

6. Sign in through each provider you changed.

   :::verify
   The SSO sign-in succeeds, and the provider shows no error in :menu[Settings > Security > Authentications].
   :::

7. Turn local authentication off in :menu[Settings > Security > Authentications] if you no longer need it.
:::

## Remove the old configuration

After the upgrade, the configuration file still holds the authentications. If you delete a migrated authentication in the UI, the next start migrates it again and it reappears.

Once every user signs in as before, remove the authentications from the configuration file or the environment variables. Delete them in the UI only after that.

## If the migration fails

If SSO worked before the upgrade and fails since, set `app:authentication:force_env` to `true` (`APP__AUTHENTICATION__FORCE_ENV=true`) and restart: OpenCTI signs users in with the providers of the configuration file again, deletes the providers stored in the database, and migrates them again at the next start without the setting. See [If you are locked out](/opencti/administer/authentications/#if-you-are-locked-out) for this and the other ways back in.

## What's next

- [Manage login providers in the UI](/opencti/administer/authentications/), to manage the migrated authentications.
- [Breaking changes](/deploy/upgrade/breaking-changes/#opencti-7-260224-0), the other changes of OpenCTI 7.260224.0.
- [Upgrade](/deploy/upgrade/), the upgrade procedure.
