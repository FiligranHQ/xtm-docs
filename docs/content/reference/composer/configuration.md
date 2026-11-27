---
title: XTM Composer configuration keys
description: Every setting XTM Composer reads, with its environment variable, type, default and effect, block by block, generated from the XTM Composer source code.
sidebar_label: Configuration
doc_type: reference
products:
  - composer
personas: [platform-engineer, developer]
generated: true
generator: reference:composer:composer-configuration
---

::diagram{id=reference-composer-configuration}

XTM Composer deploys and runs the integrations of OpenCTI and OpenAEV on a container engine. It reads `config/default.yaml`, shipped in its image, then `config/<COMPOSER_ENV>.yaml` when that file exists (`COMPOSER_ENV` is `production` by default), then its environment: a variable sets the key of the same path, in upper case with `__` between levels, such as `OPENCTI__DAEMON__SELECTOR` for `opencti.daemon.selector`. The `openaev` block takes the same keys as the `opencti` block, with variables that start with `OPENAEV__`. This reference is built from the code of the XTM Composer release it names, the one the documentation describes. [Install XTM Composer](/deploy/integrations/install-composer/) and [Configure XTM Composer](/deploy/integrations/configure-composer/) show the settings in use.
