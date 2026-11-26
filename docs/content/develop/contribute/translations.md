---
title: Add or change a translation in OpenCTI or OpenAEV
sidebar_label: Translations
description: Wrap a new text of the OpenCTI or OpenAEV interface for translation, add its English key, fill the other eight languages with DeepL, and pass the checks of the CI.
doc_type: how-to
products:
  - opencti
  - openaev
personas: [developer]
owner: docs-platform
last_reviewed: 2026-11-09
sidebar_position: 5
no_screenshot: "This page edits translation files and runs scripts in a terminal: there is no module screen to show."
migrated_from:
  - openaev:/development/translations/
---

The interfaces of OpenCTI and OpenAEV are translated into nine languages: English, German, Spanish, French, Italian, Japanese, Korean, Russian and Chinese. A text on screen is a key, the English text itself, looked up in one JSON file per language. This page takes a new text from your component to every language file, and through the checks the continuous integration (CI) runs on your pull request. Set up the repository first: see [Build and run from source](/develop/contribute/build-from-source/).

::diagram{id=develop-contribute-translations}

## How translation works

|  | OpenCTI | OpenAEV |
| --- | --- | --- |
| Translation function | `t_i18n('Add tags')` | `t('Add tags')` |
| Interface texts | `opencti-platform/opencti-front/lang/front/<language>.json` | `openaev-front/src/utils/lang/<language>.json` |
| Other texts | `lang/back/<language>.json`: the labels of attributes and relationships the back end declares, as `label: '...'` in `opencti-graphql/src` | The messages the back end returns with a `400` or `409`, which the interface passes to `t()`: they must match a key of `en.json` exactly |
| Languages | `de`, `en`, `es`, `fr`, `it`, `ja`, `ko`, `ru`, `zh` | `de`, `en`, `es`, `fr`, `it`, `ja`, `ko`, `ru`, `zh` |
| Scripts run from | `opencti-platform/opencti-front` | `openaev-front` |

The key is the English text: `en.json` maps `"Add tags"` to `"Add tags"`, and `fr.json` maps it to its French. A text built by concatenation, such as `'Unknown scenario: ' + id`, cannot be translated: use a placeholder, `t('Unknown scenario: {id}', { id })`.

## Before you start

- A clone of the repository with its dependencies installed (`yarn install`).
- A DeepL API Free key, to translate the new keys into the eight other languages. The scripts read it from `SUBSCRIPTION_KEY` and stop without it.

## Add a text

:::steps
1. Wrap the text in the translation function of the module: `t_i18n('Replay on the front desks')` in OpenCTI, `t('Replay on the front desks')` in OpenAEV.
2. Add the new keys to `en.json`:

   ```bash
   yarn extract-translation
   ```

   The script appends the keys it finds and never removes one. What it finds differs:

   | | OpenCTI | OpenAEV |
   | --- | --- | --- |
   | Files read | `.jsx` and `.tsx` of the front end, and the `label: '...'` of the back end | `.js`, `.jsx` and `.tsx` of the front end |
   | Calls found | `t_i18n('...')` and `t_i18n("...")`, with or without options | `t('...')` with single quotes and no other argument |

   Add the others to `en.json` by hand: in OpenAEV, a call with placeholder values, double quotes, an escaped quote, or in a `.ts` file; in both modules, a text passed as a property that a component translates later, such as `label: 'Seen'`.

3. Translate the new keys into the other languages, with your DeepL key in `SUBSCRIPTION_KEY`:

   ```bash
   SUBSCRIPTION_KEY="$DEEPL_API_KEY" yarn auto-translation:all
   ```

   The script translates the keys missing from each language file, then sorts the files. OpenCTI translates the `front` and `back` files; OpenAEV also brings the first letter of each new translation to the case of the English source.

4. Read the diff of every language file. DeepL can translate a word out of context, translate a field name or add quotes. Fix them by hand, as [Translation rules](#translation-rules) says.
5. Run the check of the CI:

   ```bash
   yarn verify-translation   # OpenCTI
   yarn i18n-checker         # OpenAEV
   ```
:::

To change an existing translation, edit its value in the language file and run the check. To change an English text, change the call in the component and add the new key: the old key stays in the files until someone removes it.

## The checks the CI runs

A pull request fails when its check fails.

| Module | Job and command | Fails when |
| --- | --- | --- |
| OpenCTI | **Frontend verify translation**, `yarn verify-translation` | A `t_i18n('...')` of the front end, or a `label: '...'` of the back end, has no key in the matching `en.json` |
| OpenAEV | **Frontend Quality & Unit Tests**, `yarn i18n-checker` | See the table below |

OpenAEV checks every language file:

| Check | Fails when |
| --- | --- |
| Missing key | A `t()` call uses a key absent from a language file, placeholders, double quotes and escaped quotes included |
| Missing or untranslated value | A key of `en.json` is absent from another language file, or its value is the English one, unless it only holds protected terms or is a listed loanword |
| Placeholders | A translation does not keep the placeholders (`{x}`, `{{x}}`, `${x}`) and tags of the English value, or contains a literal `undefined` |
| Protected terms | A term of the glossary present in the English value is missing from the translation |

Two files of `openaev-front/scripts/` configure it: `i18n-glossary.json` lists in `globalTerms` the brand names, acronyms and standards never translated, such as `ATT&CK`; `i18n-loanwords.json` lists the values that are rightly the same as in English, under `all` or under one language.

## Translation rules

| Rule | Right | Wrong |
| --- | --- | --- |
| The first letter keeps the case of the English source, in every language | `breached` → `compromis` | `Compromis` |
| Protected terms stay as they are | `Kubernetes 클러스터` | `쿠버네티스 클러스터` |
| Field names, enumeration values and code identifiers stay in English | `Au moins un élément parmi exercise ou scenario` | `Au moins un élément parmi exercice ou scénario` |
| No quotes the source does not have | `action_execution_arch ist erforderlich` | `„action_execution_arch“ ist erforderlich` |
| Placeholders and tags unchanged | `{count} injects` | `{nombre} injects` |

When a value is rightly the same as in English, such as `Dashboard` in German, add it to `i18n-loanwords.json` under its language in OpenAEV, rather than changing the translation.

:::example[A new button in the OpenAEV scenario screen]
A developer at Example CERT adds a button to replay a scenario on the front-desk workstations of Grand Meridian Hotels. In the component, the label is `t('Replay on the front desks')`. From `openaev-front`, with the team's DeepL key in the secret store variable `DEEPL_API_KEY`:

```bash
yarn extract-translation
SUBSCRIPTION_KEY="$DEEPL_API_KEY" yarn auto-translation:all
git diff --stat src/utils/lang/
yarn i18n-checker
```

`en.json` gains `"Replay on the front desks"`, and each of the eight other files gains its translation. The French reads `Rejouer sur les postes d'accueil`; the developer checks the German and Japanese values with colleagues, and `yarn i18n-checker` exits without an error before the pull request.
:::

## What's next

- [Build and run from source](/develop/contribute/build-from-source/), to see the change in the interface.
- [Prepare an Ubuntu development environment](/develop/contribute/environment/ubuntu/), [Windows](/develop/contribute/environment/windows/) or [macOS](/develop/contribute/environment/macos/), for the Node.js version each repository needs.
