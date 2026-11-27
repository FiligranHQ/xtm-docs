---
title: Configure single sign-on
description: Let people log in to OpenAEV with your identity provider through OpenID Connect or SAML, send them to their tenant, map their groups and keep an emergency access.
doc_type: how-to
products:
  - openaev
personas: [admin, platform-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
no_screenshot: "Single sign-on is set in the configuration of the platform, outside the interface: the diagram shows what a login does."
migrated_from:
  - openaev:/deployment/platform/authentication/
---

Single sign-on (SSO) lets people log in to OpenAEV with their account in your identity provider (IdP), through OpenID Connect (OIDC) or SAML 2.0, instead of a password stored in OpenAEV. OpenAEV creates the account at the first login, attaches it to a tenant and keeps its groups in line with the IdP at every login. Providers are set in the configuration of the platform, not in the interface. This page explains what a login does, then configures a provider, its tenant and its group mapping.

## Before you start

- You can change the configuration of the `openaev` service and restart it. With xtm-docker, see [Change the stack without editing docker-compose.yml](/deploy/install/docker-compose/#change-the-stack-without-editing-docker-compose-yml).
- `openaev.base-url` (`OPENAEV_BASE-URL`) is the public address of OpenAEV: the addresses your IdP calls back are built from it. See the [openaev block](/reference/openaev/configuration/#openaev) of the configuration reference.
- You have administrator access to your IdP, to register OpenAEV as an application.
- The groups that logins will map to exist, each with a name used by no other group of its tenant, or of the platform for a platform group. See [Map groups](#map-groups).
- Local login stays on for an emergency access. See [Keep an emergency access](#keep-an-emergency-access).

## Choose a strategy

| Strategy | Key and default | How the user logs in | How many |
| --- | --- | --- | --- |
| Local | `openaev.auth-local-enable`, `true` | Email and password form | One |
| OpenID Connect | `openaev.auth-openid-enable`, `false` | A button, then the login page of the IdP | Several providers |
| SAML 2.0 | `openaev.auth-saml2-enable`, `false` | A button, then the login page of the IdP | Several providers |

The three switches are independent. With local login and SSO both on, the login page shows the form, with one button per provider below it. Prefer OIDC or SAML in production, and keep local login for the emergency account.

## What a login does

::diagram{id=openaev-sso-login-provisioning}

OpenAEV trusts the IdP: it asks for no password, and creates the account the first time someone logs in. Five rules explain what you will see:

1. **The email identifies the user.** With OIDC, OpenAEV reads the `email` claim and refuses the login without one; it takes the names from `given_name` and `family_name`. With SAML, the email is the `NameID` of the assertion, and the names come from the attributes you name. A new account has no password. The names are refreshed at every login.
2. **The account joins one tenant.** When it logs in through a provider and is not a member of its tenant, the account is attached to the tenant of that provider, `tenant_id`, or to the default tenant without it. It then joins the groups of that tenant that have **Auto assign** on.
3. **Mapped groups follow the IdP.** At every login, OpenAEV reads the groups of the user from the claims or attributes in `groups_path`. A group of the mapping is added when its IdP value is there, and removed when it is not any more. Groups outside the mapping, added by hand or by **Auto assign**, are never touched.
4. **A missing group is skipped.** When the mapped group does not exist, the login goes on without it, unless the rule has `autoCreate`. A group created that way belongs to the tenant of the rule's `tenantId`, or to the platform without it, and has no role: it gives nothing until you give it one.
5. **Administrators join the Admin group of the tenant, only if you ask.** With `roles_admin`, a user whose roles, read from `roles_path`, contain one of its values is put in the **Admin** group of the provider's tenant, `tenant_id` or the default tenant. They leave it at the next login when their roles no longer do. With `all_admin` set to `true`, every user of the provider joins it. Without either, the **Admin** group is never touched. When the tenant has no group named **Admin**, the role is ignored, with a warning in the log.

Single sign-on never makes an account a platform administrator. An account that carries the administrator flag from an earlier version loses it at its first login without an administrator role, when `roles_admin` or `all_admin` is set; the bootstrap admin keeps it.

:::warning
A rule with a `tenantId` looks its group up by name in that tenant only. A rule without one looks it up among the platform groups only, never in a tenant. Every tenant has groups named **Admin**, **Manager** and **Observer**: a rule that names `Manager` without a `tenantId` looks for a platform group called `Manager`, and skips it, or creates it with `autoCreate`. Set `tenantId` on every rule that maps to a tenant group.
:::

## Configure a provider

Each provider has an identifier you choose, the `<id>` of its keys, in lower case. Set the keys as environment variables of the `openaev` service: the key in upper case, each dot replaced by an underscore, dashes kept. `openaev.provider.grandmeridian.tenant_id` becomes `OPENAEV_PROVIDER_GRANDMERIDIAN_TENANT_ID`.

::::tabs
:::tab[OpenID Connect]
Turn on `openaev.auth-openid-enable` (`OPENAEV_AUTH-OPENID-ENABLE`), then declare the provider with the Spring Security keys:

| Key | Value |
| --- | --- |
| `spring.security.oauth2.client.provider.<id>.issuer-uri` | The issuer of the IdP. OpenAEV reads its discovery document |
| `spring.security.oauth2.client.registration.<id>.client-id` | The client ID of the application registered in the IdP |
| `spring.security.oauth2.client.registration.<id>.client-secret` | Its secret |
| `spring.security.oauth2.client.registration.<id>.scope` | `openid,profile,email` |
| `spring.security.oauth2.client.registration.<id>.redirect-uri` | `<base URL>/login/oauth2/code/<id>`, the address to register in the IdP |
| `openaev.provider.<id>.audience` | Optional. Sent as the `audience` parameter, for an IdP such as Auth0 that needs it to issue an access token for an API |

The button sends the user to `/oauth2/authorization/<id>`.

Spring Security already knows the addresses of GitHub, Google, Facebook and Okta: give the provider the identifier `github`, `google`, `facebook` or `okta`, and set only its `client-id` and `client-secret`. GitHub speaks plain OAuth 2.0, not OpenID Connect: OpenAEV reads the email from the GitHub profile, so a user whose email is not public there cannot log in, with the message `You probably need a public email in your github account`.
:::
:::tab[SAML 2.0]
Turn on `openaev.auth-saml2-enable` (`OPENAEV_AUTH-SAML2-ENABLE`), then declare the provider:

| Key | Value |
| --- | --- |
| `spring.security.saml2.relyingparty.registration.<id>.entity-id` | The identifier of OpenAEV in the IdP |
| `spring.security.saml2.relyingparty.registration.<id>.assertingparty.metadata-uri` | The address of the metadata XML of the IdP |
| `openaev.provider.<id>.firstname_attribute_key` | The attribute that holds the first name |
| `openaev.provider.<id>.lastname_attribute_key` | The attribute that holds the last name |

In the IdP, set the reply address, where assertions go, to `<base URL>/login/saml2/sso/<id>`, and the `NameID` to the email address. OpenAEV publishes its own metadata at `<base URL>/saml2/service-provider-metadata/<id>`, which most IdPs import.
:::
::::

Both strategies then read these keys:

| Key | What it sets | Default |
| --- | --- | --- |
| `openaev.provider.<id>.login` | The text of the button | `Login with <id>` |
| `openaev.provider.<id>.tenant_id` | The tenant the account joins, and whose **Admin** group administrators join. It does not apply to the rules of `groups_management` | The default tenant |
| `openaev.provider.<id>.groups_path` | The claims or attributes that hold the groups, separated by commas | None: no group is read |
| `openaev.provider.<id>.groups_management` | The group mapping, a JSON array of rules | None |
| `openaev.provider.<id>.roles_path` | The claims or attributes that hold the roles | None |
| `openaev.provider.<id>.roles_admin` | The role values that put a user in the **Admin** group of the provider's tenant, separated by commas | None |
| `openaev.provider.<id>.all_admin` | Every user of the provider joins the **Admin** group of its tenant | `false` |

These keys carry the identifier of your provider, so the [configuration reference](/reference/openaev/configuration/) does not list them.

## Map groups

Each rule of `groups_management` has four fields:

| Field | Value |
| --- | --- |
| `idpGroup` | The group value as the IdP sends it: a name, or an object ID with Microsoft Entra ID. Case counts |
| `userGroup` | The name of the OpenAEV group |
| `autoCreate` | `true` to create the group when it does not exist, in the tenant of `tenantId` or on the platform. Keep `false` and create the group in its tenant first, with its role |
| `tenantId` | The **Tenant identifier** of the tenant that holds `userGroup`. When the rule applies, OpenAEV also attaches the user to this tenant. Without it, `userGroup` names a platform group. The provider's `tenant_id` does not stand in for it |

The rule removes the user from the same group it adds: the group of that name in the tenant of `tenantId`, or the platform group of that name without it. A `tenantId` that names no tenant skips the rule, with a warning in the log. Without `groups_path`, OpenAEV reads no group, so every mapped group is removed at the next login.

## Send users to the right tenant

With [one tenant per organization](/openaev/administer/multi-tenancy/), each login must reach its tenant. Two ways:

- **One provider per organization.** Each organization logs in through its own IdP, or its own application in a shared IdP. Set `tenant_id` on each provider: the button of the provider leads to its tenant. Its rules still need their `tenantId` to map to the groups of that tenant.
- **One provider for all.** Set `tenantId` on each rule of the mapping: a user in the IdP group of an organization is attached to its tenant and put in its group.

The **Tenant identifier** of a tenant is in :menu[Settings > Parameters], while you work in it. A `tenant_id` that names no tenant is ignored, with a warning in the log, and the account joins no tenant. Removing a user from a tenant is done in OpenAEV: a login never detaches an account.

:::example[Log the staff of Grand Meridian in with Microsoft Entra ID]
Example CERT hosts a tenant for each member hotel chain. Grand Meridian Hotels signs its staff in with Microsoft Entra ID, and wants its security engineers to build simulations and its CISO, Hélène Marchetti, to read them.

In the **Grand Meridian Hotels** tenant, the administrator creates two groups, `Grand Meridian managers` with the **Manager** role and `Grand Meridian observers` with the **Observer** role. In the Entra admin center, Grand Meridian registers an application with the web redirect URI `https://openaev.example.com/login/oauth2/code/grandmeridian` and creates a client secret. It adds the `email`, `given_name` and `family_name` optional claims to the ID token, and a groups claim that emits group IDs. The platform engineer of Example CERT adds to the `openaev` service:

```yaml
services:
  openaev:
    environment:
      OPENAEV_AUTH-LOCAL-ENABLE: "true"
      OPENAEV_AUTH-OPENID-ENABLE: "true"
      SPRING_SECURITY_OAUTH2_CLIENT_PROVIDER_GRANDMERIDIAN_ISSUER-URI: https://login.microsoftonline.com/3f6c2a1e-8b4d-4e2a-9c71-5d0e8f2b7a14/v2.0
      SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_GRANDMERIDIAN_CLIENT-ID: 0b7e5c9d-2f41-4a8e-b6d3-91c4e7a20f55
      SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_GRANDMERIDIAN_CLIENT-SECRET: ${GRANDMERIDIAN_CLIENT_SECRET}
      SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_GRANDMERIDIAN_SCOPE: openid,profile,email
      SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_GRANDMERIDIAN_REDIRECT-URI: https://openaev.example.com/login/oauth2/code/grandmeridian
      OPENAEV_PROVIDER_GRANDMERIDIAN_LOGIN: Log in with Grand Meridian
      OPENAEV_PROVIDER_GRANDMERIDIAN_TENANT_ID: 7c1e4b2a-5d3f-4e8b-9a61-2f0c8d4e6b13
      OPENAEV_PROVIDER_GRANDMERIDIAN_GROUPS_PATH: groups
      OPENAEV_PROVIDER_GRANDMERIDIAN_GROUPS_MANAGEMENT: '[{"idpGroup":"8d2f4c1a-6b3e-4f7d-9a21-0c5e7b9d3f48","userGroup":"Grand Meridian managers","autoCreate":false,"tenantId":"7c1e4b2a-5d3f-4e8b-9a61-2f0c8d4e6b13"},{"idpGroup":"c41e9a07-3d5b-4b62-8f1e-2a7d6c90b3e5","userGroup":"Grand Meridian observers","autoCreate":false,"tenantId":"7c1e4b2a-5d3f-4e8b-9a61-2f0c8d4e6b13"}]'
```

`GRANDMERIDIAN_CLIENT_SECRET` is in the `.env` file. Both rules carry the **Tenant identifier** of the Grand Meridian Hotels tenant, because their groups belong to that tenant: without it, OpenAEV would look for platform groups of those names. After a restart, the login page shows **Log in with Grand Meridian** under the form. Hélène's first login creates `helene.marchetti@grand-meridian.example`, attaches it to the Grand Meridian Hotels tenant, and puts it in `Grand Meridian observers`, because her Entra account is in the second group. When Entra ID drops her from that group, her next login removes her from it.

When a user belongs to more than 200 groups, Entra ID leaves the groups claim out of the token. Emit only the groups assigned to the application to avoid it.
:::

## Keep an emergency access

When everyone logs in through SSO, an outage, a wrong setting or an expired certificate of the IdP locks everyone out. Keep `openaev.auth-local-enable` at `true`: the bootstrap admin, whose email and password come from `OPENAEV_ADMIN_EMAIL` and `OPENAEV_ADMIN_PASSWORD`, can then still log in with the form. [Sign in the first time and keep an emergency account](/deploy/configure/first-access/#keep-a-break-glass-account) explains how to keep that account safe and how to turn local login back on.

## Check that it works

Open the login page in a private browser window: it shows the button of each provider, with the text of its `login` key. Log in through it, then, as an administrator, open :menu[Settings > Security > Users] in the tenant of the provider: the account is listed, and its groups are those of the mapping. An unknown or misspelled group leaves an error naming it in the log of the platform.

## What's next

- [How access works in OpenAEV](/openaev/concepts/access/), for the roles to give the mapped groups.
- [Give each organization its own tenant](/openaev/administer/multi-tenancy/).
- [Set the login message and the consent box](/openaev/administer/policies/), shown above the buttons.
