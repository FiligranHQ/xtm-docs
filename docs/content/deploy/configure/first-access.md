---
title: Sign in the first time and keep an emergency account
description: Set the bootstrap admin of OpenCTI, OpenAEV and XTM One, and the admin token of OpenCRQ, sign in the first time, and keep a local account that still works when single sign-on fails.
doc_type: how-to
products:
  - opencti
  - openaev
  - opencrq
personas: [platform-engineer, admin]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - opencti:/deployment/authentication/
  - openaev:/deployment/authentication/
---

Each platform of the XTM stack creates its first administrator from `.env`: the bootstrap admin. This page explains how the three bootstrap admins behave, how to sign in the first time and change their passwords, and how to keep one of them as a break-glass account, a local account that still works when single sign-on (SSO) is down. OpenCRQ has no bootstrap admin: its users sign in through XTM One, and its optional admin token works on its API only. Configuring SSO itself is in [Configure single sign-on](/opencti/administer/sso/).

::diagram{id=deploy-configure-first-access}

## Before you start

- The stack installed as [Install with Docker Compose](/deploy/install/docker-compose/) describes.
- Shell access to the host, to read and change `.env`.

## The bootstrap admins

| Platform | Variables in `.env` | Passed to the platform as | Rules |
| --- | --- | --- | --- |
| OpenCTI | `OPENCTI_ADMIN_EMAIL`, `OPENCTI_ADMIN_PASSWORD`, `OPENCTI_ADMIN_TOKEN` | `APP__ADMIN__EMAIL`, `APP__ADMIN__PASSWORD`, `APP__ADMIN__TOKEN` | A valid email address and a UUID token. OpenCTI does not start while the password or the token is `ChangeMe` |
| OpenAEV | `OPENAEV_ADMIN_EMAIL`, `OPENAEV_ADMIN_PASSWORD`, `OPENAEV_ADMIN_TOKEN` | The same names | A valid email address and a UUID token |
| XTM One | `XTM_ONE_ADMIN_EMAIL`, `XTM_ONE_ADMIN_PASSWORD` | `ADMIN_EMAIL`, `ADMIN_PASSWORD` | The admin email of OpenCTI and OpenAEV: XTM One matches users by email. A password of 72 bytes at most |

Every platform applies these values again **at every start**: it creates the admin the first time, then resets its email and password, and for OpenCTI and OpenAEV its token, to the values of `.env`. Three consequences:

- A password changed in the interface lasts until the next restart. Change it in `.env`.
- Anyone who can read `.env` and restart the stack can take over the admin accounts. Keep `.env` readable by its owner only (`chmod 600 .env`) and out of version control.
- In xtm-docker, the admin tokens are also the credentials of the services that call the platforms: the OpenCTI worker and connectors, the OpenAEV collectors and injectors, XTM Composer, and the link between OpenCTI and OpenAEV. They read the admin token variables of `.env`, so they follow a change of token. The three export connectors use `CONNECTOR_EXPORT_FILE_STIX_TOKEN`, `CONNECTOR_EXPORT_FILE_CSV_TOKEN` and `CONNECTOR_EXPORT_FILE_TXT_TOKEN` instead when you set them; empty, as in `.env.sample`, they use `OPENCTI_ADMIN_TOKEN`.

:::only{opencti=">=7.261127"}
### Checks of the OpenCTI admin password

At every start, OpenCTI also checks the value of `APP__ADMIN__PASSWORD` twice, unless `APP__ADMIN__EXTERNALLY_MANAGED=true`:

| Check | Fails when | Warning in the logs | Setting that stops the start instead |
| --- | --- | --- | --- |
| Default value | The password is `changeme`, ignoring case, as in `.env.sample` | `Default credentials are configured, change them`, which names `app:admin:password` | `APP__CREDENTIALS__SECURE_BY_DEFAULT=true` |
| Password policy | The password fails the **Local password policies** of the **Local** strategy, in :menu[Settings > Security > Authentications] | `The admin password (app:admin:password) does not match the local password policy`, with the rules it fails | `APP__ADMIN__SECURE_BY_DEFAULT=true` |

Both settings are `false` by default: OpenCTI starts and logs the warning, never the password. xtm-docker sets `APP__APP_LOGS__LOGS_LEVEL=error`, which hides warnings. A later version turns both settings on by default. The default policy has no rule, so it accepts any password until an administrator sets one.

With `APP__ADMIN__SECURE_BY_DEFAULT=true`, OpenCTI also refuses to save a password policy that the admin password fails, with the message `The configured admin password (app:admin:password) does not match this password policy`. Change the admin password first, as [Change an admin password](#change-an-admin-password) describes, then the policy.

The default value check also covers the health check key, `OPENCTI_HEALTHCHECK_ACCESS_KEY`, except its exact value `ChangeMe`, which keeps `/health` closed, and the credentials of the shared services: see [Replace the default credentials of the services](/deploy/configure/shared-services/#replace-the-default-credentials-of-the-services).
:::

### OpenCRQ

OpenCRQ keeps no password and has no sign-in form: XTM One signs its users in, and the account of `XTM_ONE_ADMIN_EMAIL` is its super administrator. See [Sign in to OpenCRQ](/deploy/configure/opencrq/#sign-in-to-opencrq).

`APP__ADMIN__TOKEN` on the `opencrq` service is a token for the API, the counterpart of the admin tokens of OpenCTI and OpenAEV. It signs nobody in to the interface. xtm-docker sets it from `OPENCRQ_ADMIN_TOKEN` of `.env`, which `init-env.sh` draws as a UUID version 4.

| `APP__ADMIN__TOKEN` | What OpenCRQ does |
| --- | --- |
| Unset | Both uses below are off. `/api/xtm/peer-tokens` answers `404` |
| A UUID version 4 | A request with `Authorization: Bearer` and this token acts as the service account `Deployment admin token`, which OpenCRQ creates in the **Admin** group of the initial tenant, the tenant of `INITIAL_TENANT__ID`. The token also opens `/api/xtm/peer-tokens`, where a deployment manager gives another XTM module a service account and a token in that tenant, and `/api/xtm/peer-tokens/revoke` |
| Another value | OpenCRQ does not start, and names the key |

OpenCRQ applies it at every start, like the other platforms: it creates or updates the `Deployment admin token` account, and keeps no copy of the token, so a new value replaces the old one at the next start. The initial tenant is the one OpenCRQ creates at its first start, as [Configure OpenCRQ](/deploy/configure/opencrq/#connectors-of-each-tenant) describes. If it does not exist when OpenCRQ starts, OpenCRQ refuses the token on the API. After the initial tenant was deleted, the next start recreates it, with the same identifier, only when no tenant is left: the token then works again. While other tenants remain, it is not recreated, and the token stays refused.

To cut the token off the API at once, choose **Disable** for the `Deployment admin token` service account of the initial tenant: see [Let integrations sign in with service accounts](/opencrq/administer/service-accounts/). No restart is needed, and a restart does not enable it again. The account does not guard `/api/xtm/peer-tokens`: only unsetting `APP__ADMIN__TOKEN` and restarting OpenCRQ closes it.

## Sign in the first time

:::steps
1. Read the admin accounts in `.env`.

   ```bash
   grep -E '^(OPENCTI|OPENAEV|XTM_ONE)_ADMIN_(EMAIL|PASSWORD)=' .env
   ```

2. Open each platform at its public URL, `http://localhost:8080` for OpenCTI, `http://localhost:8081` for OpenAEV and `http://localhost:8090` for XTM One with the values of `.env.sample`, and sign in with its email and password.

   :::verify
   Each platform opens with the admin signed in.
   :::
:::

## Change an admin password

:::steps
1. Set the new password in `.env`. For OpenCTI:

   ```bash
   NEW=$(openssl rand -hex 24)
   sed -i.bak "s/^OPENCTI_ADMIN_PASSWORD=.*/OPENCTI_ADMIN_PASSWORD=${NEW}/" .env
   ```

   Use `OPENAEV_ADMIN_PASSWORD` or `XTM_ONE_ADMIN_PASSWORD` for the other platforms.

   :::only{opencti=">=7.261127"}
   The new password of OpenCTI must pass its **Local password policies**: see [Checks of the OpenCTI admin password](#checks-of-the-opencti-admin-password). `openssl rand -hex 24` draws lowercase letters and digits only. When the policy also asks for uppercase letters or symbols, add them, such as with `NEW="$(openssl rand -hex 24)-X"`. With `APP__ADMIN__SECURE_BY_DEFAULT=true`, a password the policy refuses stops OpenCTI at step 2, and `docker compose logs opencti` shows `The admin password (app:admin:password) does not match the local password policy`.
   :::

2. Recreate the container so that it reads the new value.

   ```bash
   docker compose up -d
   ```

   :::verify
   After the platform is healthy again (`docker compose ps`), the new password signs you in and the old one is refused.
   :::
:::

## Keep a break-glass account

When users sign in through SSO only, an outage, a misconfiguration or an expired certificate of the identity provider locks everyone out, administrators included. A break-glass account is a local administrator kept for that case, and used only then. The bootstrap admin is that account: its password is in `.env`, and the platforms apply it at every start.

To keep it usable, leave local authentication on next to SSO. The sign-in page then shows the email and password form with the SSO buttons. Regular users sign in through SSO; the bootstrap admin uses the form when SSO fails.

::diagram{id=deploy-configure-first-access-break-glass}

::::tabs
:::tab[OpenCTI]
Local authentication is the **Local** strategy of :menu[Settings > Security > Authentications]. Leave it enabled. OpenCTI refuses to disable it while no other strategy is enabled.

The sign-in form checks an email and a password against each enabled LDAP provider first, then against the local accounts. The bootstrap admin is the exception to the **Local** switch: OpenCTI accepts its email and password even while **Local** is disabled. The sign-in page shows the form only while **Local** or an LDAP provider is enabled. So with an LDAP provider in place, the bootstrap admin can still sign in through the form when the directory is down.

If local authentication was disabled and SSO no longer works:

1. Add this variable to the `opencti` service in `docker-compose.override.yml`, as [Change the stack without editing docker-compose.yml](/deploy/install/docker-compose/#change-the-stack-without-editing-docker-compose-yml) describes:

   ```yaml
   services:
     opencti:
       environment:
         - APP__AUTHENTICATION__FORCE_LOCAL=true
   ```

2. Run `docker compose up -d`. At start, OpenCTI turns local authentication back on and logs `[MIGRATION-SAFETY] No other provider available, forcing local_auth to enabled`, at the `warn` level: xtm-docker sets `APP__APP_LOGS__LOGS_LEVEL=error`, which hides it.
3. Sign in with `OPENCTI_ADMIN_EMAIL` and `OPENCTI_ADMIN_PASSWORD`, fix the SSO strategy, and check that the **Local** strategy is enabled.
4. Remove the variable from `docker-compose.override.yml`, then run `docker compose up -d` again.

OpenCTI also turns local authentication back on by itself at start when no other way to sign in is usable, for example after the Enterprise Edition license that SSO needs has expired.

:::warning
`APP__ADMIN__EXTERNALLY_MANAGED=true` locks the bootstrap admin at every start. Set it only if another administrator can always sign in: it removes the break-glass account.
:::
:::
:::tab[OpenAEV]
Local authentication is a setting of the platform, `OPENAEV_AUTH-LOCAL-ENABLE`. xtm-docker sets it to `true` in the `openaev` service. Keep it when you enable SSO with `OPENAEV_AUTH-OPENID-ENABLE=true` or `OPENAEV_AUTH-SAML2-ENABLE=true`: the three settings are independent, and the sign-in page shows the form above one button per SSO provider.

If local authentication was turned off, set `OPENAEV_AUTH-LOCAL-ENABLE=true` again in the `openaev` service of `docker-compose.override.yml`, run `docker compose up -d`, and sign in with `OPENAEV_ADMIN_EMAIL` and `OPENAEV_ADMIN_PASSWORD`.
:::
:::tab[XTM One]
Local authentication is the **Local Authentication** switch in **Settings** > **Access and security** > **Authentication** of XTM One, on by default. While it is off, nobody can sign in with a password: leave it on to keep the bootstrap admin as a break-glass account.

At every start, XTM One also gives the `XTM_ONE_ADMIN_EMAIL` account back the administrator role and re-enables it, so an account disabled or demoted in the interface works again after a restart. When XTM Hub decides who is an administrator of XTM One, it never demotes this account.

Its email cannot be changed in **My Profile**: whoever holds that address is the platform administrator, so XTM One refuses the change. To use another address, change `XTM_ONE_ADMIN_EMAIL` in `.env` and run `docker compose up -d`: XTM One then creates a new admin account with that address, and leaves the account of the previous address as it was.
:::
:::tab[OpenCRQ]
OpenCRQ has no local account to keep: **Sign in with XTM One** is its only way in, and the account of `XTM_ONE_ADMIN_EMAIL` is its super administrator. The break-glass account of XTM One is therefore the one of OpenCRQ too: once it has signed in to XTM One with its password, it opens OpenCRQ.

When XTM One does not answer, the sign-in page of OpenCRQ shows **XTM One is unavailable** and nobody can sign in to the interface until XTM One is back. The token of `APP__ADMIN__TOKEN` does not depend on XTM One and still works on the API: see [OpenCRQ](#opencrq).
:::
::::

:::tip
Store the passwords of the bootstrap admins in your secret store, with the procedure to use them, and review the audit logs after each use.
:::

## What's next

- [Configure single sign-on](/opencti/administer/sso/), for the identity provider of each platform.
- [SSO migration from environment variables to UI](/deploy/upgrade/breaking-changes/opencti/7-260224-sso-migration/), if OpenCTI still takes its SSO from the environment.
- [Harden the deployment](/deploy/operate/harden/), for sessions and the other security settings.
