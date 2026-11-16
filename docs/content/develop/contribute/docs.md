---
title: Write a page of the XTM documentation
sidebar_label: Contribute to the docs
description: Set up the documentation repository, write a page from its template with facts taken from the product code, declare its screenshots, check it as the CI does, and see it built.
doc_type: how-to
products:
  - suite
personas: [developer]
owner: docs-platform
last_reviewed: 2026-11-09
sidebar_position: 6
no_screenshot: "This page edits Markdown files and runs commands in a terminal: the diagram shows how a page goes from the repository to the site."
---

The XTM documentation is plain Markdown in one repository, checked against the code of the products and built by a static engine for every documented version. This page sets up the repository, writes a page the way its readers and its checks expect, declares the screens it shows, and takes it through the checks of the continuous integration (CI). It is for anyone who changes a page: a writer, a developer documenting their feature, a reviewer.

::diagram{id=develop-contribute-docs}

## What the repository holds

The repository is the ground truth of the XTM products, taken from their code and from the running products. Four modules share it:

| Module | Folders | What it does for a page |
| --- | --- | --- |
| xtm-docs | `docs/content/`, `docs/static/`, `docs/server/`, `engine/`, `factory/` | The pages, the engine that builds them, the checks, the templates and the style guide, and the server that hosts the site and its drafts |
| xtm-atlas | `atlas/` | Reads each product's code at each documented version: routes, screen labels, configuration keys, capabilities. The checks and the generated reference pages come from it |
| xtm-capture | `capture/` | Starts each product at its version, seeds the demo story, and captures every screenshot in both themes |
| xtm-lens | `lens/` | Compares two captures screen by screen, to see what a release changed |

The files a writer touches most:

| File | What it holds |
| --- | --- |
| `factory/STYLE.md` | The style guide: one reader per page, titles, names, versions, examples, visuals |
| `factory/templates/` | One template per page type: tutorial, how-to, reference, explanation |
| `engine/README.md` | Every directive a page can use, such as `:menu`, `:::steps` or `:::only` |
| `docs/content/personas.yml` | The readers: what each one knows, what a page must explain to them, the situations of their examples |
| `factory/migration-map.yml` | Every page of the site, planned or written, with its title, section and type |
| `docs/content/versions.yml` | The versions the site describes: the latest release of each product and the supported OpenCTI LTS lines |
| `capture/shots.yml` | The specification of every screenshot |
| `capture/stories/hospitality/` | The demo story: Velvet Harbor against hotels, Example CERT, Grand Meridian Hotels |

## Before you start

- Node.js 20 or later, the Rust toolchain (`cargo`, for the engine), Python 3, and [Vale](https://vale.sh/) for the style check.
- GitHub credentials that read the product repositories. The internal ones are skipped without them, and the checks of their pages then do less.

## Set up the repository

:::steps
1. Install the dependencies and fetch the code of the products:

   ```bash
   npm ci
   npm run atlas:sources
   npm run atlas:versions
   ```

   `atlas:sources` takes a sparse checkout of each product into `.factory-cache/`, at the refs `atlas/sources.yml` names. `atlas:versions` generates the reference data of every version of `docs/content/versions.yml` into `factory/generated/`.

2. Serve the site while you write:

   ```bash
   npm run dev
   ```

   The site answers at `http://localhost:3220` and rebuilds a page a few seconds after you save it.
:::

## Write the page

:::steps
1. Find the page in `factory/migration-map.yml`: its title, its section and its type. Copy the template of its type from `factory/templates/`.
2. Fill the frontmatter: `title`, a `description` of 50 to 200 characters, `doc_type`, `products`, `personas` with the main reader first, `owner` and `last_reviewed`. `factory/schemas/frontmatter.schema.json` lists every field.
3. Title a how-to or a tutorial by what it lets the reader do, starting with a verb: **Share knowledge with your security tools**, not "Native feeds".
4. Take every fact from the code in `.factory-cache/`, never from memory: the label of a button from the product's translation file, a default from its configuration, a capability from the role editor. Name a label in **bold** exactly as the screen shows it, and a menu path with `:menu[Data > Processing > Automation]`.
5. Mark what differs between versions with a block, `:::only{opencti=">=7.261030"}`, bounded on a documented version line. Describe a feature behind a feature flag nowhere: the reader gets what the product does by default.
6. Show the idea: a `::screenshot` of the screen the task happens in, or a `::diagram` drawn from a file of `docs/content/diagrams/`.
7. Give an `:::example[...]` from the demo story, with realistic values: hosts under `example.com`, secrets in named variables, never `foo` or `<value>`.
:::

Never write a reference page by hand: the pages under `/reference/` are generated from the code of each version.

When the code shows a product defect, write it in `issues/<product>/`, in the format of `issues/README.md`. A security weakness is never described in a page: its file is marked `visibility: private`, and the page gives the reader the safe step only.

## Declare a screenshot

A screenshot is never an image file you add by hand. It is a specification the factory captures from every documented version, in both themes.

:::steps
1. Add an entry to `capture/shots.yml`: an `id`, the `product`, and a `capture` block with the seeded account, the route, the steps by accessible role and name, and the part of the screen to keep. The header of the file explains every field.
2. Find the accessible names of a screen on a running stack:

   ```bash
   npm run capture:inspect -- opencti /dashboard/data/processing/automation
   ```

3. When the screen needs data the story lacks, add it to `capture/stories/hospitality/<product>/demo.yml`. A screen that shows "No data has been found." teaches nothing.
4. Place it in the page with the `::screenshot` directive, its `id` and its `alt` text. The alternative text is a full sentence on what the reader should notice.
5. Capture what is missing, for one version:

   ```bash
   npm run capture:refresh -- 7.261030.1 --missing
   ```
:::

## Check the page

Run the checks of the CI before you push:

```bash
npm run factory:check
vale --minAlertLevel=error docs/content/develop/contribute/docs.md
```

`factory:check` reads every page and fails on:

| Check | Fails when |
| --- | --- |
| `frontmatter` | A field is missing or out of the schema |
| `images` | A screenshot is not declared, not captured for a version it applies to, or its alternative text is too short |
| `labels` | A segment of a `:menu` path is not a label of the product's interface |
| `links` | A link reads "here", "click here" or "this page" |
| `diagrams` | A diagram file is missing, an arrow joins no card, an icon is unknown, or a page holds a Mermaid block |
| `practice` | A page of a product space lacks its screenshot or its example, or its example holds a placeholder |
| `visuals` | A page shows neither a screenshot nor a diagram |
| `personas` | A persona is unknown, or does not read about the page's products |
| `markup` | A directive block is not closed |
| `flags` | A page names a feature flag |
| `versions` | An `:::only` bound is not a documented version line |
| `titles` | A how-to or a tutorial title does not start with a verb |

`npm run factory:check -- anchors` checks the links to sections on the built site: build it first with `npm run build`.

## Propose the change

Push a branch and open a pull request. The CI fetches the product code, regenerates the reference data, runs `factory:check` and Vale, builds every version and checks accessibility (WCAG 2.2 AA) in both themes. The documentation server builds every branch as a draft for signed-in Filigran people, under `/preview/<branch>/`, with a band that links to its pull request: reviewers read the page as it will be published, from the **Drafts** menu of the site.

:::example[Document a new playbook component]
A developer at Example CERT adds a component to OpenCTI playbooks and documents it on the branch `contrib/ana/playbook-hotel-label`. They open `docs/content/opencti/use/automate/playbooks/choose-a-component.md`, add a row for the component with its label read from `en.json`, and add a `:::only{opencti=">=7.261030"}` block around the sentence that describes it. They then run:

```bash
npm run dev
npm run factory:check
vale --minAlertLevel=error docs/content/opencti/use/automate/playbooks/choose-a-component.md
git push -u origin contrib/ana/playbook-hotel-label
```

The page shows the row at `http://localhost:3220`, the checks pass, and the reviewers read the draft at `/preview/contrib--ana--playbook-hotel-label/` before they merge. The generated [playbook components reference](/reference/opencti/playbook-components/) lists the component on its own, from the code.
:::

## What's next

- The style guide, `factory/STYLE.md`, and the directives, `engine/README.md`, in the repository.
- [Build and run from source](/develop/contribute/build-from-source/), to run a product and see the screen you document.
- [Add or change a translation in OpenCTI or OpenAEV](/develop/contribute/translations/), when your change adds a text to an interface.
