---
title: Send emails from OpenCTI
description: Configure the mail server OpenCTI sends emails through from the interface, with basic or OAuth2 authentication, and test it without restarting the platform.
sidebar_position: 27
doc_type: how-to
products:
  - opencti
personas: [admin, platform-engineer]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/administration/smtp-configuration/
---

OpenCTI sends notifications, digests, password resets and disseminated files by email. This page shows how to set the mail server from the interface, so a change applies at once, without editing the configuration file or restarting the platform.

:::only{opencti="<7.260811"}
In this version, the mail server is set only in the configuration of the platform: see the `smtp` keys in the [SMTP Service](/reference/opencti/configuration/#smtp-service) section of the configuration reference.
:::

:::only{opencti=">=7.260811"}
## Before you start

- Your role has the **Manage credentials** capability.
- You know the host name and port of your mail server, and the account OpenCTI sends with. Port 25 is not accepted here: use 465 (SSL/TLS) or 587 (STARTTLS).
- For OAuth2, you have registered OpenCTI as an application with your mail provider and hold a client ID, a client secret, a refresh token and the issuer URL. The refresh token must carry the scope that allows sending mail over SMTP: `https://outlook.office.com/SMTP.Send` with `offline_access` for Microsoft 365, `https://mail.google.com/` for Google Workspace. Without it, the mail server rejects the authentication of every email.

## How the interface and the configuration file share the job

The mail server can come from two places: the `smtp` keys of the [platform configuration](/reference/opencti/configuration/#smtp-service), or this screen. The screen wins when its **Use configuration in interface** setting is on. Open :menu[Settings > Security > SMTP configuration] to see which one applies:

- **A warning says you cannot configure a new SMTP integration here.** The `smtp:forced_sender_email` key is set. The table shows the values of the configuration file, read-only. To use the screen, remove that key and restart the platform.
- **The table is empty, with only an Update button.** Nothing is set on the screen yet: the configuration file applies.
- **A message says SMTP is configured through a backend configuration.** Settings are saved on the screen, but **Use configuration in interface** is off: the configuration file applies.
- **The table lists your settings, next to a Test button.** The screen applies.

In the table, **SMTP enabled** is the **Enable email sending** switch of the screen, and **Use configuration in interface** says whether the screen applies.

## Configure the mail server

:::steps

1. Open :menu[Settings > Security > SMTP configuration] and click **Update**. The **Update SMTP configuration** panel opens.
2. Turn on **Enable email sending**, and **Use this configuration instead of the platform config file**. With **Enable email sending** off, OpenCTI sends no email at all and reports no error: password resets, notifications and disseminated files never leave the platform.
3. Fill **Sender email address**, the `From:` address of every email.
4. Fill **Hostname** and **Port**. Turn on **Use SSL/TLS** for port 465, and **Reject unauthorized certificates** to refuse servers with an invalid certificate.
5. Choose the **Authentication type**:
   - **Basic**: fill **Username** and **Password**.
   - **OAuth2**: fill **OAuth user** (the mailbox address), **OAuth client ID**, **OAuth client secret**, **OAuth issuer** (the OpenID Connect issuer URL) and **OAuth refresh token**. Set **Refresh token expiration date** to the date your provider revokes the token: it only serves as a reminder.
6. Click **Save**. The table shows the new settings; the password and the client ID appear masked.

:::

:::example[Send through the corporate mail relay with STARTTLS]
Example CERT sends its notifications through the relay `smtp.example.com`, which accepts authenticated clients on port 587.

- **Enable email sending** and **Use this configuration instead of the platform config file**: on.
- **Sender email address**: `cti-platform@example.com`, an address the relay is allowed to send as.
- **Hostname**: `smtp.example.com`. **Port**: `587`.
- **Use SSL/TLS**: off. On port 587 the connection starts in clear text, then upgrades to TLS with STARTTLS.
- **Reject unauthorized certificates**: on, so that OpenCTI refuses a relay whose certificate it cannot verify.
- **Authentication type**: **Basic**. **Username**: `cti-platform@example.com`. **Password**: the password of that mailbox, from your vault.

After **Save**, click **Test** and send a test email to `soc@example.com`.
:::

OpenCTI encrypts the password, the client secret and the refresh token before it stores them. With OAuth2, it gets an access token from the issuer with the refresh token and renews it before it expires. When the provider issues a new refresh token, OpenCTI stores it in place of the old one.

A mail server set on the screen ignores the TLS versions and ciphers of the configuration file: `smtp:tls_min_version`, `smtp:tls_max_version` and `smtp:tls_ciphers` apply only to the server of the file.

:::only{opencti=">=7.261030"}
The connection timeouts of the file, `smtp:connection_timeout` and `smtp:greeting_timeout`, 10 seconds each by default, apply to both.
:::

::screenshot{id=opencti-smtp-configuration alt="The SMTP configuration table lists each setting with its value, under the menu, Test and Update buttons."}

## Check that it works

:::steps

1. Click **Test**.
2. Enter an **Email address** you can read and click **Send**.
3. Open that mailbox.

:::

The confirmation only means the mail server accepted the message. If nothing arrives, check the spam folder and the logs of the mail server.

With OAuth2, a banner warns users who can manage credentials when the refresh token is about to expire, then when it has expired. Get a new token from your provider and paste it in **OAuth refresh token** before that date.

## Go back to the configuration file

Open the menu (⋮) next to **Test** and choose **Remove configuration**. In the **Delete SMTP configuration** dialog, click **Delete**. The settings of the screen are deleted, and the `smtp` keys of the configuration file apply again at once, without a restart.

## Which sender address is used

The `From:` address of an email is, in this order:

1. the **Sender email address** of this screen, when the screen applies;
2. the `smtp:forced_sender_email` key;
3. the **Sender email address** of the [platform parameters](/opencti/configure/parameters/).

The display name is the **Platform title** of Parameters, unless the feature sending the email sets its own.
:::

## Set the mail server in the configuration file

The `smtp` keys of the [platform configuration](/reference/opencti/configuration/#smtp-service) set the mail server at startup, in every version, unless the SMTP configuration screen applies. Change them, then restart the platform.

`smtp:enabled` (`SMTP__ENABLED`), `true` by default, turns email sending on or off. With `false`, OpenCTI skips the mail server check at startup, logs `SMTP disabled by configuration`, and sends no email at all, without any error: password resets, notifications and disseminated files never leave the platform. Set it to `false` on a platform that must not send email, such as a test copy of production. When the SMTP configuration screen applies, its **Enable email sending** switch decides instead.

:::only{opencti=">=7.261109"}
`smtp:reject_unauthorized` (`SMTP__REJECT_UNAUTHORIZED`), when you leave it unset, follows `tls:reject_unauthorized` (`TLS__REJECT_UNAUTHORIZED`), the platform-wide certificate check, `false` by default. A later version turns the platform-wide check on by default: set this key yourself, to `true` once OpenCTI trusts the certificate of your relay, so that the change of default does not change how OpenCTI connects to it. See [Check certificates on every outgoing connection of OpenCTI](/deploy/configure/certificates/#check-certificates-on-every-outgoing-connection-of-opencti).
:::

:::only{opencti=">=7.260811"}
`smtp:auth_type` (`SMTP__AUTH_TYPE`) chooses how OpenCTI authenticates to the mail server:

- `basic`, the default: with `smtp:username` and `smtp:password`. When `smtp:username` is empty, OpenCTI sends without authentication.
- `oauth2`: with an access token that OpenCTI gets from your identity provider. It needs the five keys below, which have no default. If one is missing, every email fails with `SMTP OAuth2 configuration is incomplete`.

| Key | Variable | Value |
| --- | --- | --- |
| `smtp:oauth_user` | `SMTP__OAUTH_USER` | The address of the mailbox OpenCTI sends from |
| `smtp:oauth_client_id` | `SMTP__OAUTH_CLIENT_ID` | The client ID of the application registered with your provider |
| `smtp:oauth_client_secret` | `SMTP__OAUTH_CLIENT_SECRET` | Its client secret |
| `smtp:oauth_issuer` | `SMTP__OAUTH_ISSUER` | The OpenID Connect issuer URL of the provider. OpenCTI reads its discovery document to find the token endpoint |
| `smtp:oauth_refresh_token` | `SMTP__OAUTH_REFRESH_TOKEN` | A refresh token with the scope that allows sending mail over SMTP, as described in [Before you start](#before-you-start) |

OpenCTI exchanges the refresh token for an access token before it sends, keeps the access token in memory, and renews it five minutes before it expires. Unlike the screen, the configuration file is never rewritten: when your provider issues a new refresh token, OpenCTI does not keep it. Replace `SMTP__OAUTH_REFRESH_TOKEN` and restart the platform before the token you gave expires, or every email fails.
:::

:::example[Send through a mail relay over TLS]
Example CERT sends through `smtp.example.com`, which accepts authenticated clients on port 465 with TLS from the first byte. In the docker compose file, the `opencti` service gets:

```yaml
services:
  opencti:
    environment:
      SMTP__HOSTNAME: smtp.example.com
      SMTP__PORT: 465
      SMTP__USE_SSL: 'true'
      SMTP__REJECT_UNAUTHORIZED: 'true'
      SMTP__TLS_MIN_VERSION: TLSv1.2
      SMTP__USERNAME: cti-platform@example.com
      SMTP__PASSWORD: ${SMTP_PASSWORD}
```

`SMTP__USE_SSL` opens the connection over TLS, and `SMTP__REJECT_UNAUTHORIZED` refuses a relay whose certificate does not verify. If the relay uses a private certificate authority, give its root certificate to `NODE_EXTRA_CA_CERTS` rather than turning the check off. `SMTP_PASSWORD` comes from the `.env` file next to the compose file. The `From:` address is the **Sender email address** of the [platform parameters](/opencti/configure/parameters/).
:::

::::only{opencti=">=7.260811"}
:::example[Send through Microsoft 365 with OAuth2]
Example CERT's mailbox `cti-platform@example.com` is hosted on Microsoft 365, where the tenant has basic authentication turned off. The platform team registered OpenCTI in Microsoft Entra ID, with the `https://outlook.office.com/SMTP.Send` and `offline_access` scopes, and keeps the secrets in the `.env` file next to the compose file:

```yaml
services:
  opencti:
    environment:
      SMTP__HOSTNAME: smtp.office365.com
      SMTP__PORT: 587
      SMTP__USE_SSL: 'false'
      SMTP__REJECT_UNAUTHORIZED: 'true'
      SMTP__AUTH_TYPE: oauth2
      SMTP__OAUTH_USER: cti-platform@example.com
      SMTP__OAUTH_CLIENT_ID: ${SMTP_OAUTH_CLIENT_ID}
      SMTP__OAUTH_CLIENT_SECRET: ${SMTP_OAUTH_CLIENT_SECRET}
      SMTP__OAUTH_ISSUER: https://login.microsoftonline.com/${ENTRA_TENANT_ID}/v2.0
      SMTP__OAUTH_REFRESH_TOKEN: ${SMTP_OAUTH_REFRESH_TOKEN}
```

On port 587 the connection upgrades to TLS with STARTTLS, so `SMTP__USE_SSL` stays `false`. After the restart, OpenCTI gets an access token from Entra ID and sends as `cti-platform@example.com`; the platform team puts the refresh token's expiry date in its calendar.
:::
::::

## What's next

- [Notifiers](/opencti/configure/notifiers/) to send notifications by email.
- [Customize the emails OpenCTI sends](/opencti/configure/email-templates/) to shape the emails of the platform.
- The [SMTP Service keys](/reference/opencti/configuration/#smtp-service) of the configuration reference, for timeouts and TLS versions.
