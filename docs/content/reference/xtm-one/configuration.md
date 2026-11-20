---
title: XTM One configuration
description: Every environment variable XTM One reads at startup, on the platform and on the worker, with
  its default and what it does, generated from the source code.
sidebar_label: Configuration
doc_type: reference
products:
  - xtm-one
personas: [platform-engineer, admin]
generated: true
generator: reference:xtm-one:configuration
---

::diagram{id=reference-xtm-one-configuration}

XTM One reads its infrastructure from environment variables when it starts: its URLs, its database, Redis, file storage, the providers of its models, its limits. Set each one on the platform and on the worker, with the same value, then restart both. Sign-in, the integrations and most security options are set from the **Settings** screens instead and kept in the database. Where a variable and a field of **Settings** cover the same option, the variable wins and the field is shown locked.

The first administrator comes from `ADMIN_EMAIL` and `ADMIN_PASSWORD`, set again at every start. `SECRET_KEY` signs the sessions and encrypts the secrets stored in the database: generate it once and never change it.
