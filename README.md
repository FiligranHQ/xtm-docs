# XTM documentation

The content of the XTM documentation, published at **[docs.filigran.io](https://docs.filigran.io)**: XTM One,
OpenCTI, OpenAEV, OpenCRQ, XTM Hub and XTM Composer.

| Folder | What it holds |
| --- | --- |
| `docs/content/` | The pages, in Markdown, with the site's structure (`versions.yml`, `suite.yml`, `journeys.yml`) |
| `docs/generated/` | The reference data of each documented version, read from the products' code: configuration keys, capabilities, APIs, data model |
| `docs/static/` | The site's static files |

## A mirror, one way

This repository is written only by Filigran's documentation pipeline: each time the documentation is published, its
`main` branch receives the new content. Its other branches are drafts the documentation team is reviewing. Nothing
written here directly reaches the site, so **pull requests here cannot be merged**: they are closed automatically.

## Tell us what to fix

- **On any page** of [docs.filigran.io](https://docs.filigran.io): **Give us feedback**, or select a passage and
  click **Report an issue**. Either opens an issue here, filled with the page, its version and the passage.
- **Here**: [open an issue](https://github.com/FiligranHQ/xtm-docs/issues/new/choose).

The documentation team reads every issue. When it takes one on, the change is written, reviewed and published, and
the issue is closed with it.

A security vulnerability in a product is never an issue here: see [SECURITY.md](SECURITY.md).
