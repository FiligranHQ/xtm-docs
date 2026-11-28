---
title: Configure single sign-on
description: Sign people in to XTM One through OpenID Connect or SAML, decide who gets an account and which role, map SAML groups, and keep local sign-in, two-factor and sessions safe.
doc_type: how-to
products:
  - xtm-one
personas: [admin, platform-engineer]
owner: docs-xtm-one
last_reviewed: 2026-11-10
migrated_from:
  - xtm-one:/deployment/authentication/
---

XTM One signs people in with a local password, through OpenID Connect (OIDC) with the Authorization Code flow, or through SAML 2.0 as a service provider. Everything is set in :menu[Administration > Settings > Access and security > Authentication], stored in the database, and applied without a restart. This page connects an identity provider, decides who gets an account and with which role, and keeps the sign-in safe: local fallback, two-factor authentication and session lifetime. How roles and groups decide what people reach is in [How access works in XTM One](/xtm-one/concepts/access/).

::diagram{id=xtm-one-administer-sso-flow}

## Before you start

- Your account is an administrator.
- `BASE_URL` is the address people open XTM One at, `https://xtm-one.grand-meridian.example` for example: the addresses you register in the identity provider are built from it.
- For OIDC, an application registered in your identity provider, with its client ID and client secret. For SAML, the metadata URL of your identity provider.
- The client secret can be read from a vault instead of being stored: see [Keep secrets in your vault](/xtm-one/administer/secret-vaults/).

## Connect OpenID Connect

:::steps

1. Open :menu[Administration > Settings > Access and security > Authentication] and click **Configure**.
2. In your identity provider, set the redirect URI shown under **Set the OIDC Redirect URI in your identity provider to:**, `{BASE_URL}/api/auth/oidc/callback`.
3. Turn on **OpenID Connect (OIDC)** and fill in the fields:

   | Field | What to enter |
   | --- | --- |
   | **Display Name** | The label of the sign-in button, **Microsoft** by default. Required |
   | **Discovery URL** | The provider's `/.well-known/openid-configuration` URL, not the issuer alone. For Keycloak, `https://<host>/realms/<realm>/.well-known/openid-configuration`. Required unless you fill the three manual endpoints |
   | **Client ID**, **Client Secret** | From the application of your identity provider, stored encrypted and never shown again. Required the first time; once one is stored, leave the field empty to keep it |
   | **Scopes** | `openid`, `email` and `profile` by default. Press Enter after each scope, and keep `openid` and `email` |
   | **Advanced: Manual endpoint URLs (if not using discovery)** | The **Authorization URL**, **Token URL** and **UserInfo URL**, all three, only for a provider without discovery |

4. Click **Test connection**. It fetches the discovery document and says what is missing; saving does not depend on it.
5. Decide **On-the-fly user creation** and **Automatically admin users** (below), then click **Save** in the bar at the bottom, which shows **Unsaved changes** while the form differs from what is stored.

:::

A method switched on is saved only with its required fields, marked with an asterisk: **Save** then moves to the first empty one, which reads "This field is required."

::screenshot{id=xtm-one-administer-sso-authentication alt="The OpenID Connect section in edit mode, filled for Grand Meridian: the Display Name, the Entra ID discovery URL with Test connection, the client fields, the redirect URI to register, and On-the-fly user creation on with Automatically admin users off."}

The identity provider must return an email claim: a sign-in without one is refused. The name and job title are updated from the provider's claims at each sign-in.

## Connect SAML 2.0

:::steps

1. Open :menu[Administration > Settings > Access and security > Authentication] and click **Configure**.
2. In your identity provider, create a SAML application with the values the screen shows:
   - **Identifier (Entity ID): Required**: `{BASE_URL}/api/auth/saml/metadata`, unless you set your own **SP Entity ID**.
   - **Reply URL (Assertion Consumer Service URL): Required**: `{BASE_URL}/api/auth/saml/acs`.
   - The email claim, used to match and create accounts, and optionally the display name, the job title and the group memberships.
3. Turn on **SAML 2.0**, give a **Display Name** (**Corporate SSO** by default) and the **IdP Metadata URL**, both required. Leave **SP Entity ID** empty unless your identity provider already knows XTM One under another identifier.
4. Leave **Requested Authentication Context** off to accept every authentication method of the provider (certificates, MFA, FIDO2). On, the request asks for password-based authentication.
5. Decide the account rules and the group mapping (below), then click **Save**.

:::

## Decide who gets an account

XTM One matches a single sign-on to an account in this order:

1. the account already linked to that identity (its SSO subject);
2. otherwise the account with the same email, local or not, which is then linked;
3. otherwise a new account, with the **User** role.

| Setting | Default | Effect |
| --- | --- | --- |
| **On-the-fly user creation** (OIDC and SAML, each its own) | On | Off, only existing accounts can sign in through that method: everyone else is refused until an administrator or an invitation creates their account. |
| **Allowed email domains** (**New accounts**) | Empty: any domain | New accounts, from local registration or single sign-on, only for an address at one of these domains. Exact match: `grand-meridian.example` does not admit `eu.grand-meridian.example`. Existing accounts keep signing in. |
| **Automatically admin users** (OIDC and SAML, each its own) | Off | On, everyone who signs in through that method is made an administrator at each sign-in, existing accounts included. Turning it off demotes nobody. |

:::warning[Keep Automatically admin users off]
On, it makes every person your identity provider lets in an administrator of XTM One. Promote administrators by hand on their page instead, and check the **Users** tab of :menu[Administration > Users and groups], filtered on the **Admin** role, after turning it off.
:::

The domain of a single sign-on comes from the email the identity provider asserts. With a multi-tenant provider, make sure it asserts verified addresses only.

A disabled account is refused before anything changes on it: no profile update, no new link, no group synchronization. The refusal is recorded in :menu[Administration > Audit logs] as `login_refused`.

### Map SAML groups

**Automatic Groups Mapping** (on by default; the summary of the section then reads **Auto-sync groups from IdP**) synchronizes the person's groups at each SAML sign-in: they join the XTM One groups named in the assertion and leave those it no longer names. **Auto-create Groups** (on by default, available while the mapping is on) creates a group of the assertion that does not exist yet; off, the groups that do not exist in XTM One are ignored. XTM One reads the group claims of Microsoft Entra ID, Okta, ADFS and generic SAML providers. OIDC has no group mapping.

A group that keeps people off a model provider (an exclusion on its **Access**) follows your directory this way. See [Set up the platform in Settings](/xtm-one/administer/settings/#ai-and-models).

### Let the XTM Hub decide

When people reach XTM One through the XTM Hub, the XTM Hub can decide who may sign in through OIDC and who is an administrator. Set `XTM_HUB_ADMIN_GROUP` to the role that means administrator in the XTM Hub claim, and set `PLATFORM_ID`. It then wins over **Automatically admin users**: the XTM Hub also demotes. The bootstrap account of `ADMIN_EMAIL` (`admin@filigran.io` when unset) is never demoted, and local sign-in stays available if the gate is wrong. The gate does not depend on the registration of the platform on the XTM Hub, a separate step: see [Connect XTM One to the XTM Hub](/xtm-one/administer/xtm-hub/). See [XTM Hub sign-in gate](/reference/xtm-one/configuration/#xtm-hub-sign-in-gate).

## Keep local sign-in as a fallback

| Setting | Default | Effect |
| --- | --- | --- |
| **Local Authentication** (**Allow email & password login**) | On | Off, everyone signs in through single sign-on, administrators included. The very first account can always be created. |
| **Local Registration** (**Allow local registration**) | Off | On, anyone can create an account from the sign-in page, within the allowed domains. |

Before turning **Local Authentication** off, make sure an administrator signs in through single sign-on: an identity provider outage then locks everyone out. Accounts also come from an administrator or an invitation: see [How access works in XTM One](/xtm-one/concepts/access/#create-an-account).

## When a sign-in fails

The person returns to the sign-in page with a short message and a marker in the address. The details go to the server log, never to the page.

| `?sso_error=` | When |
| --- | --- |
| `cancelled` | They declined the consent screen of the identity provider. |
| `not_provisioned` | No account matches and none is created: on-the-fly creation is off, or the domain is not allowed. |
| `account_deactivated` | Their account is disabled. |
| `not_authorised` | The XTM Hub did not entitle them to this instance. |
| `misconfigured` | The discovery URL or the endpoints gave no usable authorization or token endpoint. |
| `failed` | Any other error, or the provider was unreachable, slow or rate-limiting. |

## Two-factor authentication

Each person can add a six-digit code from an authenticator app, in their profile. XTM One asks for it after the first factor whatever the method, local, OIDC or SAML, and the consent page of an MCP client too. It stacks with the identity provider's own multi-factor authentication. Enabling it gives 10 single-use recovery codes. API keys, channel bots and the tokens of OpenCTI, OpenAEV and OpenCRQ are not affected.

When a person loses their device and their codes, **Reset two-factor** on their **Profile** tab clears the enrollment. Changing `SECRET_KEY` makes every enrollment unreadable: recovery codes still work, and administrators reset the others.

## Bound the sessions

A sign-in opens a web session. Two settings of :menu[Administration > Settings > Access and security > Security options] bound it, applied on every server within 5 seconds:

| Field | Variable | Default | Effect |
| --- | --- | --- | --- |
| **Absolute session lifetime (minutes)** | `JWT_EXPIRE_MINUTES` | `10080` (7 days) | The session ends this long after the sign-in, even while it is used. Lowering it ends the older sessions on their next request. |
| **Idle session timeout (minutes)** | `SESSION_IDLE_TIMEOUT_MINUTES` | `0` (off) | The session ends after this long without a click, a key or a scroll; 5 minutes at least. The web app warns a minute before. It needs Redis: while Redis is unreachable, sessions are not ended for inactivity. |

The variable wins over the field. A day or less for the lifetime and 5 to 30 minutes of idle time are what the **Security audit** counts as enforced. After a session ends, people using single sign-on choose their provider again on the sign-in page: XTM One never opens it on its own.

## Terms of Use

Where AI is licensed, everyone accepts Filigran's Terms of Use after their first sign-in, and again when `TERMS_OF_USE_VERSION` changes. `TERMS_OF_USE_ENABLED=false` removes the screen, for a company that collects that acceptance elsewhere.

:::example[Sign the Grand Meridian staff in with Microsoft Entra ID]
Hélène Marchetti connects Microsoft Entra ID to `https://xtm-one.grand-meridian.example`. In Entra ID, she registers an application "XTM One" with the redirect URI `https://xtm-one.grand-meridian.example/api/auth/oidc/callback`, and puts its client secret in the company vault.

In :menu[Administration > Settings > Access and security > Authentication], **Configure**:

| Field | Value |
| --- | --- |
| **OpenID Connect (OIDC)** | On |
| **Display Name** | `Grand Meridian` |
| **Discovery URL** | `https://login.microsoftonline.com/grand-meridian.example/v2.0/.well-known/openid-configuration` |
| **Client ID** | The application ID of "XTM One" |
| **Client Secret** | **From a vault**: `vault://corp/secret/xtm-one/entra#client_secret` |
| **On-the-fly user creation** | On |
| **Automatically admin users** | Off |
| **Allowed email domains** | `grand-meridian.example` |
| **Local Authentication** | On, for the bootstrap account |

**Test connection** answers **The Discovery URL is usable.** Camille Roux signs in with **Grand Meridian** the next morning: her account is created with the **User** role, and Hélène adds her to **CTI team**. A contractor with an address outside `grand-meridian.example` is sent back with `?sso_error=not_provisioned`.

Marc Dupuis sets the session bounds in the deployment:

```bash
JWT_EXPIRE_MINUTES=1440
SESSION_IDLE_TIMEOUT_MINUTES=30
```
:::

## What's next

- [How access works in XTM One](/xtm-one/concepts/access/)
- [Set up the platform in Settings](/xtm-one/administer/settings/)
- [Security switches](/reference/xtm-one/configuration/#security-switches)
