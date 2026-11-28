---
title: Configure single sign-on
description: Let people log in to OpenCTI with your identity provider, map its groups to OpenCTI groups, and keep an emergency local access.
sidebar_position: 6
doc_type: how-to
products:
  - opencti
edition: ee
personas: [admin, platform-engineer]
owner: docs-opencti
last_reviewed: 2026-11-01
migrated_from:
  - opencti:/deployment/authentication/
---

Single sign-on (SSO) lets people log in to OpenCTI with their account in your identity provider (IdP), instead of a password stored in OpenCTI. This page helps you choose a strategy and explains how a login creates the user and sets its groups. It also shows how to keep an emergency access. You then create the providers in [Manage login providers in the UI](/opencti/administer/authentications/).

## Before you start

- Your platform has an Enterprise Edition license. Without it, you can still configure providers, but only local login (email and password) works.
- Your role has the **Manage authentication** capability.
- You have administrator access to your IdP, to register OpenCTI as a client application.
- `app:base_url` is the public URL users reach OpenCTI at (see [Basic parameters](/reference/opencti/configuration/#basic-parameters)). Callback URLs are built from it.
- You know the password of a local administrator account, in case the IdP is unreachable. See [Keep an emergency access](#keep-an-emergency-access).

## Choose a strategy

A strategy is either a **form** or a **button** on the login page. With a form, the user types a login and a password, checked by OpenCTI or by LDAP. With a button, the IdP or the request identifies the user, who comes back logged in.

| Strategy | How the user logs in | How many |
| --- | --- | --- |
| Local | Email and password form | One, on by default |
| OpenID Connect (OIDC) | Button, then the IdP login page | Several providers |
| SAML 2.0 | Button, then the IdP login page | Several providers |
| LDAP | Login form, checked against your directory | Several providers |
| Client certificate | Button, then a certificate chosen in the browser. OpenCTI must serve HTTPS itself | One |
| HTTP headers | Button. OpenCTI reads the identity from headers set by your reverse proxy | One |

All the form strategies share one login form. OpenCTI checks the email and password against each enabled LDAP provider, then against the local accounts, and logs the user in with the first that accepts them. A user unknown to the directory can therefore still log in with a local password while **Local** is on.

Prefer OIDC or SAML in production, and keep local login for emergency access. When it migrates a configuration file, OpenCTI turns the older Google, GitHub, Facebook and Auth0 strategies into OIDC providers. GitHub and Facebook are not OIDC providers: their migrated issuer is a placeholder that you must replace. Set new providers up as OIDC.

## How a login provisions the account

OpenCTI trusts the IdP: it does not ask for a password, and it creates the account the first time someone logs in. Every login then refreshes the account from what the IdP returns.

::diagram{id=admin-identity-sso-provisioning}

Five rules explain most of what you will see:

1. **The email identifies the user.** OpenCTI looks up the account by the email the IdP returns, and refuses the login when there is none. A new account gets the groups marked **Granted by default at user creation**, unless the provider has **Prevent platform default groups association** on.
2. **Groups come from the mapping.** You map each value the IdP returns (a group, a role, a claim) to the name of an OpenCTI group. Values without a mapping are ignored. The **Default groups** of the provider are added to the mapped ones.
3. **A missing group blocks the login.** When a mapping or a default group names a missing group, OpenCTI refuses the login, unless **Auto create groups** is on. A group created this way has no role: give it one, or its members can do nothing.
4. **Groups are replaced at every login.** When the mapping gives at least one group, the user's groups become exactly those groups. Groups added by hand, and the default groups granted at creation, disappear. When nothing maps, groups stay as they are.
5. **Organizations work the same way.** The organization mapping replaces the user's organizations at every login. A missing organization blocks the login, unless **Auto create organizations** is on.

:::only{opencti=">=7.260811"}
To keep groups added by hand, turn on **Extend platform groups**. At login, OpenCTI then adds or removes only the groups that appear in the mapping.
:::

:::example[Follow the first Entra ID login of an analyst]
Ana Duval logs in for the first time with Microsoft Entra ID. The ID token returns `ana.duval@example.com` and the object ID of the Entra group mapped to CTI analysts. No account has this email, so OpenCTI creates one with the groups granted by default at user creation, then replaces them with CTI analysts, the only mapped group. Ana gets the Analyst role, a maximum confidence level of 75 and the TLP:AMBER clearance of that group. If an administrator later adds her to SOC analysts by hand, her next login removes it again, as rule 4 says.
:::

:::warning
A user needs a maximum confidence level, usually inherited from a group, to create or change data. Check that the groups you map to have one. See [Organize users in groups](/opencti/administer/groups/).
:::

## Configure the providers

OpenCTI stores its providers in the database. Create and edit them in :menu[Settings > Security > Authentications]: [Manage login providers in the UI](/opencti/administer/authentications/) describes each field.

::screenshot{id=opencti-sso-providers alt="The SSO providers list of the Authentications screen, with the Corporate IdP OpenID Connect provider, its status and its Update and Logs actions, next to the Create Authentication button."}

**Providers in the configuration file.** At startup, OpenCTI migrates the `providers` it finds in `config/production.json` or in `PROVIDERS__*` environment variables into the database. It migrates OIDC, SAML and LDAP providers, the older OAuth strategies as OIDC, and the client certificate and HTTP headers settings. A provider whose identifier already exists in the database is skipped. Once logins work, remove the providers from the file. A provider you delete in the interface comes back at the next start while it is still in the file.

:::warning
After the migration, open each SAML provider and check **Want assertion signed** and **Want authn response signed**: turn on the one your identity provider signs, as you would for a new provider.
:::

The [7.260224 migration notes](/deploy/upgrade/breaking-changes/opencti/7-260224-sso-migration/) cover the details.

**Keep using the file.** Set `app:authentication:force_env` to `true` (see [Functional customization](/reference/opencti/configuration/#functional-customization)) and restart. OpenCTI then deletes the providers stored in the database and loads them from the file only. The interface can no longer change them, except the password policy and two-factor authentication. This mode is a deprecated safeguard: plan to move back to the interface.

In the file, each provider is an entry of `providers` with a `strategy`, an optional `identifier` and a `config`.

- **Local login needs no entry.** The built-in `local` provider is declared by default, as `providers:local:strategy` (`PROVIDERS__LOCAL__STRATEGY`), and its value is not read (see [Network and security](/reference/opencti/configuration/#network-and-security)). You turn local login on or off in the **Local** settings of the interface. The file decides it only when OpenCTI first creates these settings, on a new platform or at the upgrade that moved providers to the database, and at every start with `force_env`. Local login is then on, unless `providers:local:config:disabled` is `true`.
- **Keep secrets out of the file and the database.** A provider created in the interface can read its client secret, private key or bind password from a named secret of the platform configuration, under `secrets` (for example `SECRETS__ENTRA_CLIENT_SECRET__VALUE`), or from CyberArk. See [Keep secrets out of the database](/opencti/administer/authentications/#keep-secrets-out-of-the-database).

The identifier sets the callback URL, `/auth/<identifier>/callback`. Without one, an OIDC provider gets `oic`, a SAML provider `saml` and an LDAP provider `ldapauth`. Each entry of a groups mapping is `<IdP value>:<OpenCTI group name>`, so a group name cannot contain a colon. The examples below are the `environment` of the `opencti` service in a docker compose file, with secrets read from its `.env` file.

:::example[Log in with Microsoft Entra ID through OpenID Connect]
Grand Meridian Hotels signs its staff in with Microsoft Entra ID. In the Entra admin center, register an application with the web redirect URI `https://opencti.example.com/auth/entra/callback`. Create a client secret, and under **Token configuration** add the `email` optional claim and a groups claim emitting group IDs to the ID token.

```yaml
services:
  opencti:
    environment:
      PROVIDERS__ENTRA__STRATEGY: OpenIDConnectStrategy
      PROVIDERS__ENTRA__IDENTIFIER: entra
      PROVIDERS__ENTRA__CONFIG__LABEL: Log in with Microsoft Entra ID
      PROVIDERS__ENTRA__CONFIG__ISSUER: https://login.microsoftonline.com/3f6c2a1e-8b4d-4e2a-9c71-5d0e8f2b7a14/v2.0
      PROVIDERS__ENTRA__CONFIG__CLIENT_ID: 0b7e5c9d-2f41-4a8e-b6d3-91c4e7a20f55
      PROVIDERS__ENTRA__CONFIG__CLIENT_SECRET: ${ENTRA_CLIENT_SECRET}
      PROVIDERS__ENTRA__CONFIG__REDIRECT_URIS: '["https://opencti.example.com/auth/entra/callback"]'
      PROVIDERS__ENTRA__CONFIG__GET_USER_ATTRIBUTES_FROM_ID_TOKEN: 'true'
      PROVIDERS__ENTRA__CONFIG__GROUPS_MANAGEMENT__TOKEN_REFERENCE: id_token
      PROVIDERS__ENTRA__CONFIG__GROUPS_MANAGEMENT__GROUPS_PATH: '["groups"]'
      PROVIDERS__ENTRA__CONFIG__GROUPS_MANAGEMENT__GROUPS_MAPPING: '["8d2f4c1a-6b3e-4f7d-9a21-0c5e7b9d3f48:CTI analysts","c41e9a07-3d5b-4b62-8f1e-2a7d6c90b3e5:SOC analysts"]'
```

The issuer holds the tenant ID, and the mapping turns the object IDs of two Entra groups into the OpenCTI groups CTI analysts and SOC analysts. The email, the name and the groups are read from the ID token. Staff who belong to neither Entra group log in with the groups granted by default at user creation. The login page shows **Log in with Microsoft Entra ID** next to the local form, which stays on for emergency access.

When a user belongs to more than 200 groups, Entra ID leaves the groups claim out of the token. Emit only the groups assigned to the application to avoid it.
:::

:::example[Log in through a SAML identity provider]
The sharing community logs its members in through a SAML identity provider at `idp.example.com`, where OpenCTI is registered as the client `opencti` and the assertion carries the `email`, `name` and `groups` attributes.

```yaml
services:
  opencti:
    environment:
      PROVIDERS__SAML__STRATEGY: SamlStrategy
      PROVIDERS__SAML__CONFIG__LABEL: Log in with Harbor community SSO
      PROVIDERS__SAML__CONFIG__ISSUER: opencti
      PROVIDERS__SAML__CONFIG__ENTRY_POINT: https://idp.example.com/realms/cti/protocol/saml
      PROVIDERS__SAML__CONFIG__SAML_CALLBACK_URL: https://opencti.example.com/auth/saml/callback
      PROVIDERS__SAML__CONFIG__CERT: ${SAML_IDP_CERT}
      PROVIDERS__SAML__CONFIG__WANT_ASSERTIONS_SIGNED: 'true'
      PROVIDERS__SAML__CONFIG__MAIL_ATTRIBUTE: email
      PROVIDERS__SAML__CONFIG__ACCOUNT_ATTRIBUTE: name
      PROVIDERS__SAML__CONFIG__GROUPS_MANAGEMENT__GROUP_ATTRIBUTES: '["groups"]'
      PROVIDERS__SAML__CONFIG__GROUPS_MANAGEMENT__GROUPS_MAPPING: '["harbor-analysts:Harbor analysts","harbor-contractors:Harbor contractors"]'
```

`SAML_IDP_CERT` holds the signing certificate of the IdP, on one line without its `BEGIN` and `END` lines. OpenCTI rejects assertions that are not signed with it. Without `MAIL_ATTRIBUTE` and `ACCOUNT_ATTRIBUTE`, OpenCTI reads the email and the name from the `nameID` of the assertion.
:::

:::example[Check logins against the corporate directory with LDAP]
The hotel group keeps its accounts in Active Directory. People type their email address and their directory password in the login form, and OpenCTI searches the directory with a read-only service account.

```yaml
services:
  opencti:
    environment:
      PROVIDERS__LDAP__STRATEGY: LdapStrategy
      PROVIDERS__LDAP__CONFIG__URL: ldaps://dc.example.com:636
      PROVIDERS__LDAP__CONFIG__BIND_DN: CN=svc-opencti,OU=Service Accounts,DC=example,DC=com
      PROVIDERS__LDAP__CONFIG__BIND_CREDENTIALS: ${LDAP_BIND_PASSWORD}
      PROVIDERS__LDAP__CONFIG__SEARCH_BASE: OU=Staff,DC=example,DC=com
      PROVIDERS__LDAP__CONFIG__SEARCH_FILTER: '(mail={{username}})'
      PROVIDERS__LDAP__CONFIG__MAIL_ATTRIBUTE: mail
      PROVIDERS__LDAP__CONFIG__ACCOUNT_ATTRIBUTE: displayName
      PROVIDERS__LDAP__CONFIG__GROUP_SEARCH_BASE: OU=Groups,DC=example,DC=com
      PROVIDERS__LDAP__CONFIG__GROUP_SEARCH_FILTER: '(member={{dn}})'
      PROVIDERS__LDAP__CONFIG__GROUPS_MANAGEMENT__GROUP_ATTRIBUTE: cn
      PROVIDERS__LDAP__CONFIG__GROUPS_MANAGEMENT__GROUPS_MAPPING: '["CTI-Analysts:CTI analysts","SOC-Analysts:SOC analysts"]'
```

The group search returns the directory groups the user is a member of, and their `cn` is matched against the mapping: members of `CTI-Analysts` land in CTI analysts. Without `ACCOUNT_ATTRIBUTE`, the name is read from `givenName`.
:::

**Identity provider with a private certificate authority.** If your own certificate authority issued the IdP certificate, OpenCTI fails with `unable to get local issuer certificate`. Mount the root certificate, with its full chain, into the container and point `NODE_EXTRA_CA_CERTS` to it, then restart. For example: `NODE_EXTRA_CA_CERTS=/cert_volume/ca.crt`. Do not turn certificate validation off. See [Certificates](/deploy/configure/certificates/).

## Keep an emergency access

If the IdP is down or a mapping is wrong, nobody can log in through SSO, administrators included. Keep a local administrator account, with a strong password stored in your vault, and use it only in that case.

The platform administrator is the account defined by `app:admin:email` and `app:admin:password` (see [Basic parameters](/reference/opencti/configuration/#basic-parameters)). OpenCTI resets its password to `app:admin:password` at every start. Keep local login on until you have logged in through SSO with the right groups and organizations.

:::only{opencti=">=7.261127"}
At every start, OpenCTI also checks `app:admin:password` against the password policy of the **Local** settings, unless `app:admin:externally_managed` is `true`. A password that fails it logs a warning; with `app:admin:secure_by_default` (`APP__ADMIN__SECURE_BY_DEFAULT`) set to `true`, the platform refuses to start instead, and refuses a new policy that the password does not meet. Choose an emergency password that meets your policy. See [Basic parameters](/reference/opencti/configuration/#basic-parameters).
:::

Three safety nets protect local login:

- The **Local** settings do not let you turn local login off while no other provider is enabled.
- At startup, OpenCTI turns local login back on when no other provider can work.
- With **Local** off, the login form, when an LDAP provider shows it, still accepts the email and password of the platform administrator.

If you are locked out anyway, [If you are locked out](/opencti/administer/authentications/#if-you-are-locked-out) gives the recovery steps with `app:authentication:force_local`.

## Check that it works

1. Open the login page in a private browser window. Each SSO provider shows its button, with its label.
2. Log in with an IdP account that belongs to a mapped group.
3. As an administrator, open the user in :menu[Settings > Security > Users] and check its groups and organizations.
4. If the login fails, click **Logs** on the row of the provider in :menu[Settings > Security > Authentications]. The logs show what the IdP returned and why OpenCTI refused the login.

## What's next

- [Manage login providers in the UI](/opencti/administer/authentications/), every field of the providers.
- [Organize users in groups](/opencti/administer/groups/) and [Roles](/opencti/administer/roles/), to prepare the groups you map to.
- [Organizations](/opencti/administer/organizations/), to prepare the organizations you map to.
- [OpenCTI configuration reference](/reference/opencti/configuration/), for `app:base_url` and the authentication keys.
