---
title: Keep secrets in your vault
description: Connect HashiCorp Vault, AWS Secrets Manager, Google Cloud Secret Manager, Azure Key Vault or 1Password, grant it, and let secret fields read from it instead of storing a copy.
doc_type: how-to
products:
  - xtm-one
personas: [admin, platform-engineer]
owner: docs-xtm-one
last_reviewed: 2026-11-08
migrated_from:
  - xtm-one:/deployment/secret-vaults/
---

XTM One can read the API keys, tokens and client secrets it needs from your enterprise vault instead of storing them. You connect a vault once and grant it; people then pick a secret from it in any secret field. The field stores a reference, `vault://<reference name>/<path>#<key>`, and XTM One reads the secret each time a feature uses it. Rotating the secret in the vault is enough: no change in XTM One, no restart. Until a vault is connected, secret fields take a value as before.

::diagram{id=xtm-one-administer-secret-vaults-reference}

## Before you start

- Your account is an administrator.
- An identity for XTM One in the vault, with read (and list, for browsing) on the paths people will pick. Prefer the platform's workload identity where the vault offers one: nothing is stored then.
- The vault is reachable from the platform and the worker, over HTTPS, at a private or public address. A vault on the platform's own host needs its origin in `SECRET_VAULT_TRUSTED_ORIGINS`.

## Connect a vault

:::steps

1. Open :menu[Administration > Settings > Access and security > Secret vaults] and click **Connect a vault**.
2. Pick the **Vault type**: **HashiCorp Vault**, **AWS Secrets Manager**, **Google Cloud Secret Manager**, **Azure Key Vault** or **1Password**.
3. Give a **Display name**, what people see in the picker, and a **Reference name**: lowercase letters, digits and `_`, starting with a letter, 32 at most. References name the vault by it, and it cannot change while a secret field uses it.
4. Choose the **Sign-in method** and fill in its settings (below). The form marks the recommended one.
5. Click **Test connection**. It reports each step, with the reason when one fails, and never a value: the settings, the sign-in, listing the secrets, reading one.
6. Set **Who may use it** (below), then click **Connect**.

:::

The list shows each vault's status (**Connected**, **Failing** with the reason, **Not tested yet**, **Disabled**, **Misconfigured**, or declared in the environment), its last test, and how many secret fields read from it: click the count to see which. What opens the vault is stored encrypted, shown as **Set**, and never returned by any API, log or trace. Every connection, change, test and deletion is recorded in :menu[Administration > Audit logs].

::screenshot{id=xtm-one-administer-secret-vaults alt="The Secret vaults section with the Grand Meridian Vault: Connected, its reference vault://corp/, HashiCorp Vault, its last test and who changed it, granted to SOC and browsable, with no secret field reading from it yet."}

| Vault | Sign-in methods | What the identity needs |
| --- | --- | --- |
| **HashiCorp Vault** | **AppRole** (role ID and secret ID; the platform renews its token) or **Token** | `read` and `list` on the paths of a KV engine, version 1 or 2. **Address** in `https://`, **KV mount** required with KV version 1, **CA certificate (PEM)** for a private CA |
| **AWS Secrets Manager** | The platform's AWS identity (IRSA, task role, instance profile), **Assume a role**, or **Access key** | `secretsmanager:GetSecretValue`, `secretsmanager:ListSecrets` for browsing, `kms:Decrypt` with a customer-managed key |
| **Google Cloud Secret Manager** | **Application Default Credentials / Workload Identity**, or **Service account key** | `roles/secretmanager.secretAccessor`, and `roles/secretmanager.viewer` on the project for browsing |
| **Azure Key Vault** | **Managed or workload identity**, or **Client secret** | The **Key Vault Secrets User** role. The **Vault URL** is a key vault of an Azure cloud (`*.vault.azure.net`) |
| **1Password** | **Connect server access token** of your 1Password Connect server | Read access to the 1Password vaults people will pick from. A one-time password field reads as its seed, which a browser profile needs to answer an MFA prompt |

A Kubernetes service account login to HashiCorp Vault is declared in the environment only (below).

## Grant a vault

**Who may use this vault** decides who can pick from it:

- **Administrators only**, the default. Administrators always reach every path of every enabled vault.
- **Everyone**, with **Path prefixes (optional)** to narrow it: type each prefix, such as `secret/shared` then `secret/ci`, and press Enter after each one.
- **Specific groups and users**: **Add a group** or **Add a user**, with the **Path prefixes** each may use, entered the same way; empty, every path. A row with no group or user picked says "Pick a group, or remove the row." and keeps the vault from being saved.

Prefixes match whole segments and are case-sensitive: `secret/soc` admits `secret/soc/jira`, never `secret/soc-staging/jira`. **Browsable** lets people who are not administrators list secret names in the picker; without it, they type the path, still checked against their grant. A path outside a person's grant is refused when they save it, in the picker, and when they check it.

Give a grant to everyone only with path prefixes: someone who can edit an integration could otherwise point it at their own server with any secret the vault identity reads. The **Security audit** reports a vault open to everyone with no path restriction.

To cut people off from a vault at once, click **Disable**: every read of it fails until you enable it again. Then replace the fields that read from it, listed by its count of secret fields.

## Pick a secret

Every secret field shows **From a vault** to a person who may use at least one vault:

| Where | Fields |
| --- | --- |
| Integrations | API key, bot token, client secret |
| MCP servers | Header values, environment variable values, the OAuth client secret |
| Settings, **AI and models** > **Models** | A provider's API key |
| Variables | The value of a secret variable |
| Settings, **Integrations** > **Channels & Bots** | Bot token, client secret |
| Browser profiles | Username, password, TOTP secret |
| Settings, **Access and security** > **Authentication** | The OpenID Connect client secret |
| Settings, **Usage and quotas** > **Observability** | The Langfuse keys |

In the picker, browse or search, pick the secret, then the key to read, or **Whole value**. **Check** reads it on the server and says whether it can be read, never what it is. **Use this secret** puts the reference in the field, shown as a chip with the vault, the path and the key. **Paste a reference instead** takes one written by hand.

The reference is the whole value of the field: `Bearer vault://corp/...` in a header is text, not a reference. Store the whole header value, `Bearer` and the token, in the vault instead. A prompt, a persona or a flow never holds a reference: a secret reaches them through a secret variable. See [Store values agents reuse in variables](/xtm-one/build/variables/).

## Rotation, cache and failures

A secret read from a vault is kept in memory only, for the vault's **Cache duration (seconds)**, or `SECRET_VAULT_CACHE_SECONDS` (300 by default), then read again. A rotated secret is used within that time. `0` reads the vault at every use. When an integration's service refuses its credential, its secrets are read again at once.

When a secret cannot be read, the feature that needs it fails with a generic error, and the vault's answer never reaches the user. The details are in the server log; the entity's page shows which of its fields failed and when; the **Security audit** reports the failures of the last 7 days.

## Declare a vault in the environment

For a deployment managed as code, a platform engineer declares a vault in the environment of the platform and the worker. It is listed in Settings, read-only, and wins over a vault of the same name connected there.

```bash
SECRET_VAULT_<NAME>_TYPE=hashicorp    # or aws, gcp, azure, onepassword
SECRET_VAULT_<NAME>_<SETTING>=...
```

`<NAME>` is the reference name: `SECRET_VAULT_CORP_TYPE` declares `vault://corp/...`. Its settings are those of [Security](/reference/xtm-one/configuration/#security), plus two that decide who may use it:

- `WRITERS`: `admins` (default), or `all` for anyone who may edit a secret field.
- `PATHS`: the path prefixes references may read; empty for every path. Checked again at each read, so narrowing it stops the references outside it.

Prefer a `_FILE` setting (`TOKEN_FILE`, `SECRET_ID_FILE`) or a workload identity to an inline token: whoever inspects the container sees the environment.

:::example[Read the SOC's Jira token from the Grand Meridian vault]
Grand Meridian Hotels keeps its secrets in HashiCorp Vault at `https://vault.grand-meridian.example:8200`, KV version 2. Marc Dupuis creates an AppRole `xtm-one`, with read and list on `secret/xtm-one/*`.

Hélène Marchetti connects it in :menu[Administration > Settings > Access and security > Secret vaults]:

| Field | Value |
| --- | --- |
| **Vault type** | **HashiCorp Vault** |
| **Display name** | `Grand Meridian Vault` |
| **Reference name** | `corp` |
| **Address** | `https://vault.grand-meridian.example:8200` |
| **Sign-in method** | **AppRole**, with the role ID and secret ID Marc gave her |
| **Who may use this vault** | The group **SOC**, path prefix `secret/xtm-one/soc` |
| **Browsable** | On |

Paul Garnier connects the Jira integration of "Night shift triage" and, for its API token, picks **From a vault**, `secret/xtm-one/soc/jira`, key `api_token`. The field holds `vault://corp/secret/xtm-one/soc/jira#api_token`. When the Jira token is rotated in the vault, the agent uses the new one within five minutes.

The same vault, declared in the environment instead:

```bash
SECRET_VAULT_CORP_TYPE=hashicorp
SECRET_VAULT_CORP_ADDRESS=https://vault.grand-meridian.example:8200
SECRET_VAULT_CORP_AUTH_METHOD=approle
SECRET_VAULT_CORP_ROLE_ID=0f3c9d2e-7a41-4b8e-9c55-2d1e6a7b8c90
SECRET_VAULT_CORP_SECRET_ID_FILE=/run/secrets/vault-secret-id
SECRET_VAULT_CORP_PATHS=secret/xtm-one
```
:::

## What's next

- [Set up the platform in Settings](/xtm-one/administer/settings/)
- [Connect agents to your tools with integrations](/xtm-one/build/integrations/)
- [Security](/reference/xtm-one/configuration/#security)
