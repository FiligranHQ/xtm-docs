---
title: Password encryption
description: OpenAEV 2.1.0 encrypts the secrets of integrations stored in its database and needs two new mandatory settings, an encryption key and a salt.
doc_type: how-to
products:
  - openaev
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - openaev:/deployment/breaking-changes/2.1.0-encrypting-password/
---

From OpenAEV 2.1.0, the integrations of the catalog are managed from the OpenAEV interface, and their configuration is stored in the database. OpenAEV encrypts the passwords and secrets of that configuration, which needs two new mandatory settings. This page gives them and how to set them.

::diagram{id=breaking-openaev-2-1-0-password-encryption}

| | Version |
| --- | --- |
| Introduced in | OpenAEV 2.1.0 |

## The two settings

| Parameter | Environment variable | Default | Description |
| --- | --- | --- | --- |
| `openaev.admin.encryption_key` | `OPENAEV_ADMIN_ENCRYPTION_KEY` | None, mandatory | Key used with the salt to derive the 256-bit key that encrypts sensitive data in the database |
| `openaev.admin.encryption_salt` | `OPENAEV_ADMIN_ENCRYPTION_SALT` | None, mandatory | Salt used with the key. At least 8 bytes long |

## Set them

Set both before you start OpenAEV 2.1.0 or later. With xtm-docker, they are in `.env`.

:::steps
1. Check whether `.env` already gives them a value.

   ```bash
   grep -E '^OPENAEV_ADMIN_ENCRYPTION_(KEY|SALT)=[^ #]' .env
   ```

   If it prints both lines, keep them and stop here: OpenAEV needs the same values to read the secrets it has already encrypted.

2. Otherwise, generate a key and a salt and write them to `.env`.

   ```bash
   sed -i.bak -e "s|^OPENAEV_ADMIN_ENCRYPTION_KEY=.*|OPENAEV_ADMIN_ENCRYPTION_KEY=$(openssl rand -hex 32)|" \
     -e "s|^OPENAEV_ADMIN_ENCRYPTION_SALT=.*|OPENAEV_ADMIN_ENCRYPTION_SALT=$(openssl rand -hex 16)|" .env
   ```

   :::verify
   ```bash
   grep -E '^OPENAEV_ADMIN_ENCRYPTION_(KEY|SALT)=[^ #]' .env
   ```
   It prints both lines, each with a value.
   :::

3. Start OpenAEV again: `docker compose up -d openaev`.
:::

Keep a copy of both values with your backups: without them, OpenAEV cannot read the secrets it has encrypted.

## What's next

- [OpenAEV configuration reference](/reference/openaev/configuration/), every OpenAEV setting.
- [Breaking changes](/deploy/upgrade/breaking-changes/), the other changes per version.
