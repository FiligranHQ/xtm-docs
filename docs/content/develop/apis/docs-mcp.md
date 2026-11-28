---
title: Connect your AI assistant to the documentation
description: Add the documentation's MCP server to Claude, ChatGPT, Cursor, VS Code or XTM One, so your assistant answers from the published pages, for your version, with links.
doc_type: how-to
products:
  - suite
personas: [developer, platform-engineer, admin]
owner: docs-platform
last_reviewed: 2026-11-27
sidebar_position: 7
no_screenshot: The task happens in the reader's own assistant, not in a screen of an XTM product; the diagram shows the connection instead.
---

The XTM documentation has a Model Context Protocol (MCP) server, `https://filigran.io/docs/mcp/`. Add it to your AI assistant once, and the assistant answers your questions about OpenCTI, OpenAEV, OpenCRQ, XTM One and XTM Composer from the documentation as it is published, for the version you run, with a link to the section behind each answer. It needs no account and no key, and it only reads.

::diagram{id=develop-apis-docs-mcp}

## What the documentation's server gives your assistant

The documentation's MCP server gives an assistant three tools, which read the pages of the site and nothing else:

| Tool | What it does |
| --- | --- |
| `search_docs` | Finds the sections that best answer a question, for a product and a version: each with its title, its address with its anchor, and its first lines |
| `read_page` | Reads a page, or one of its sections by its address with its anchor, as Markdown: its products and versions, its edition and its readers first |
| `lookup_reference` | Gives exact values from the references generated from the products' code: a configuration key with its environment variable and default, a capability, a filter key, a metric, an error |

The version is what you run: an OpenCTI LTS line (`7.260811-lts`), a version of another product (`3.261116.1`), or the latest when you give none. An earlier version of a product is prepared the first time someone asks for it: the tool then says to ask again in about a minute. When your assistant connects, the server also gives it the rules of a good answer: answer from the tools only, cite each section, answer for your version, and say when the documentation does not say something.

The server is public and read-only: it serves the published pages and keeps nothing of what you ask. One address may call it 120 times a minute; past that, it answers with an error for the rest of the minute.

## Before you start

- An assistant that connects to remote MCP servers over HTTP (Streamable HTTP): Claude, ChatGPT in developer mode, Cursor, VS Code, XTM One, or your own MCP client.
- Nothing on the XTM side: no account, no key, no setting.

## Connect your assistant

::::tabs
:::tab[Claude]
Claude adds the server as a custom connector, in claude.ai and in the Claude desktop app alike: see [Anthropic's guide to custom connectors](https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp).

1. Open **Customize > Connectors**, then **Add custom connector**.
2. Name it `XTM documentation` and enter the server's address, `https://filigran.io/docs/mcp/`.
3. Leave authentication empty (no sign-in, no header), then **Add**.
4. In a conversation, turn the connector on from the tools menu.

On a Team or Enterprise plan, an owner adds the connector for the organization, and members turn it on.
:::
:::tab[Claude Code]
Run this command once, in a terminal:

```bash
claude mcp add --transport http xtm-docs https://filigran.io/docs/mcp/
```

Add `--scope user` to have it in every project, rather than in the current one.
:::
:::tab[ChatGPT]
ChatGPT connects to an MCP server in developer mode: see [OpenAI's guide to developer mode](https://developers.openai.com/api/docs/guides/developer-mode).

1. Turn on developer mode in ChatGPT's settings, under the connectors' advanced settings.
2. Create a connector with the server's address, `https://filigran.io/docs/mcp/`, and no authentication.
3. In a conversation, choose the connector from the tools menu.
:::
:::tab[Cursor]
Open this link on the computer where Cursor runs: [Add the XTM documentation to Cursor](cursor://anysphere.cursor-deeplink/mcp/install?name=xtm-docs&config=eyJ1cmwiOiJodHRwczovL2ZpbGlncmFuLmlvL2RvY3MvbWNwLyJ9). Cursor asks you to confirm.

Or add it to `~/.cursor/mcp.json` yourself:

```json
{
  "mcpServers": {
    "xtm-docs": {"url": "https://filigran.io/docs/mcp/"}
  }
}
```
:::
:::tab[VS Code]
Open this link on the computer where VS Code runs: [Add the XTM documentation to VS Code](vscode:mcp/install?%7B%22name%22%3A%22xtm-docs%22%2C%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Ffiligran.io%2Fdocs%2Fmcp%2F%22%7D). VS Code asks you to confirm.

Or add it to the `mcp.json` of your user profile or of a workspace (`.vscode/mcp.json`):

```json
{
  "servers": {
    "xtm-docs": {"type": "http", "url": "https://filigran.io/docs/mcp/"}
  }
}
```
:::
:::tab[XTM One]
Add it as a remote MCP server of your XTM One, as [Connect an MCP server](/xtm-one/build/mcp-servers/) describes: **Custom Server**, the **Streamable HTTP** transport, the address `https://filigran.io/docs/mcp/`, no header. Then attach its three tools to the agents that should answer from the documentation. They only read, so they can run without asking.
:::
::::

## Ask your first question

Ask with your version in the question, or say it once at the start of the conversation: the answer differs between versions for some features, and the assistant asks when it matters and you did not say.

:::example[A platform engineer checks how to turn email off on an LTS line]
Example CERT runs OpenCTI 7.260811 LTS for its hotel chains, and a test copy of it must send no email. Its platform engineer asks Claude Code, with the documentation's server added:

> We run OpenCTI 7.260811 LTS. Which environment variable turns email sending off, and what happens to password resets?

The assistant looks the setting up in the configuration reference of 7.260811, and answers that `SMTP__ENABLED` set to `false` turns email sending off, with OpenCTI sending no email at all, password resets included, and links the section of the reference and the page on email it read, both at `/lts/7.260811-lts/`.
:::

## Check that it works

- Your assistant lists the three tools of the server: `search_docs`, `read_page` and `lookup_reference`.
- An answer links pages of `https://filigran.io/docs/`, at your version: under `/lts/<line>/` for an OpenCTI LTS line, under `/v/<product>/<version>/` for an earlier version of another product.
- A question the documentation does not answer gets "the documentation does not say", and an offer to report the gap, rather than a guess.

## What's next

- On any page, **Ask Claude about this page** and **Ask ChatGPT** send the page to an assistant, with nothing to add first.
- To work with the products themselves from an assistant, their data rather than their documentation, use [XTM One's MCP servers](/develop/apis/mcp/).
- When an answer shows a gap or an error in the documentation, [tell us](https://github.com/FiligranHQ/xtm-docs/issues/new?template=page-feedback.md).
