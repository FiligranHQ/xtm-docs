---
title: Use XTM One's own MCP servers from other clients
description: Connect Claude, Cursor, VS Code or Codex to the OpenCTI, OpenAEV and OpenCRQ registered with XTM One, or to XTM One itself, with your own identity and permissions.
doc_type: how-to
products: [xtm-one, opencti, openaev, opencrq]
personas: [ai-builder, developer, cti-analyst]
owner: docs-xtm-one
last_reviewed: 2026-11-08
migrated_from:
  - xtm-one:/usage/resources/platform-mcp-servers/
  - xtm-one:/usage/resources/platform-tools-mcp-server/
---

XTM One is also a Model Context Protocol (MCP) server. From an MCP client you already use, such as Claude, Cursor, VS Code or Codex, you can search and update the OpenCTI knowledge, the OpenAEV scenarios or the OpenCRQ risk registered with XTM One, and read the traces of your agents and fix their configuration in XTM One itself. Every call runs as you, with your permissions in each product. This page finds your endpoints, connects a client, and says what each server can and cannot do. To give XTM One agents the tools of another MCP server, see [Connect an MCP server](/xtm-one/build/mcp-servers/).

::diagram{id=xtm-one-build-platform-mcp-servers-endpoints}

## The endpoints

| Endpoint | What it reaches | Available |
| --- | --- | --- |
| `/mcp/opencti` | OpenCTI knowledge: search, entities, observables, indicators, relationships, reports and other containers, notes, labels, markings | As soon as an OpenCTI is registered with XTM One |
| `/mcp/openaev` | OpenAEV: scenarios, simulations, injects, payloads, atomic tests, expectations, findings, assets, lessons learned | As soon as an OpenAEV is registered |
| `/mcp/opencrq` | OpenCRQ: frameworks and requirements, control capabilities, assessments, risk scenarios, financial summaries, assets, vulnerabilities, treatment plans | As soon as an OpenCRQ is registered |
| `/mcp/xtm` | XTM One itself: agent traces, agents, assignments, runs, knowledge bases, prompts, skills, custom tools, integrations, MCP servers, variables | Once an administrator enables it; requires the Enterprise Edition |

The three product servers, called the platform MCP servers, are available in the Community Edition: they call the products' APIs and no language model. Each takes the full address of your instance, for example `https://xtm-one.grand-meridian.example/mcp/opencti`, and uses the MCP Streamable HTTP transport.

All of them need AI to be switched on. An administrator can turn the platform MCP servers off with **Platform MCP Servers**, and turn the XTM One server on with **Platform Tools MCP Server**, both in :menu[Administration > Settings > Integrations > MCP Endpoint]. The XTM One server is off by default because it changes the platform's own configuration.

:::xtm-one[Your rights, not XTM One's]
XTM One never stores the products' API tokens for these servers. For each call it signs a short-lived token for your user, matched by email in OpenCTI, OpenAEV or OpenCRQ. Your capabilities, data markings and audit trail apply there exactly as if you did it yourself. On `/mcp/xtm`, you see and change what you could in the XTM One interface: company-managed objects stay read-only unless you are an administrator.
:::

## Find your endpoints

Open your profile from the user menu (**My Profile**), then the **MCP Endpoint** tab.

- The **Platform MCP Servers** card lists one server per product, **Connected** or **Registered** with its address and a copy button, and **Config** to copy a client configuration. A product not registered with XTM One shows **Not connected**.
- The **Platform Tools MCP Server** card, once an administrator has enabled it, shows the address of `/mcp/xtm` and the packages under **Install in your client**.

::screenshot{id=xtm-one-build-platform-mcp-servers-profile alt="The Platform MCP Servers card of the MCP Endpoint tab, 3 connected: OpenCTI with 28 tools and OpenAEV with 52, each Connected with its version, its address under /mcp/, a copy button and Config."}

## Connect a client

A client signs in to these servers in one of two ways:

- **OAuth**: clients that support it find the sign-in by themselves. An XTM One page in your browser names the client, for example Claude Code; approve it only if you are installing it yourself.
- **A personal API key** (`fcp-...`), created in the **API Keys** tab of your profile and sent as a bearer token. Create one key per client, so you can revoke one without cutting off the others.

### Install a package

Under **Install in your client**, pick the format of your client, then the server when several are live. A package installs one server: install a second one when you need it.

| Format | For | How it installs |
| --- | --- | --- |
| **Claude Code** | Claude Code | A plugin with a skill that teaches the model the tools, installed from the terminal |
| **Claude Desktop** | Claude Desktop | An extension you double-click |
| **Agent Plugins** | Any Agent Plugins client, VS Code included | A vendor-neutral package, never with a key |
| **Other** | Codex, the ChatGPT desktop app, Cursor | A fragment to merge into the client's configuration file, shown with its path |

**Include the API key** decides what the download carries:

- **Yes** (the default): XTM One creates a new API key for this package, named after the client, and writes it into the file. Nothing is asked at install, and the file is then a live credential: delete it once installed, never share it, and revoke its key from the **API Keys** tab if it leaks.
- **No**: the package carries no key, and your client asks for one at install. In the **Other** fragment, the key is read from the `XTM_ONE_API_KEY` environment variable.

<!-- shot: xtm-one-build-platform-mcp-servers-install -->

Prefer one server per client to all of them at once: the four servers together carry more than two hundred tools, and a client that loads them all chooses its tools less well.

### Configure a client by hand

A client with no package of its own takes the server's address and an `Authorization` header with your key. The **Config** button of the **Platform MCP Servers** card copies a ready configuration.

:::example[A CTI analyst searches OpenCTI from Cursor]
Camille Roux writes her Velvet Harbor notes in Cursor and wants to look up the intrusion set's indicators without leaving it. She creates an API key named `Cursor` in her profile, exports it as `XTM_ONE_API_KEY` in her shell, and merges this fragment into `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "opencti": {
      "url": "https://xtm-one.grand-meridian.example/mcp/opencti",
      "headers": {
        "Authorization": "Bearer ${env:XTM_ONE_API_KEY}"
      }
    }
  }
}
```

She asks Cursor for the indicators of Velvet Harbor seen since 9 January 2026. Cursor calls the OpenCTI tools as Camille, and sees only the knowledge her OpenCTI groups and markings allow.
:::

## What the servers do and do not cover

The platform MCP servers expose business and knowledge tools only. Platform administration stays out by design, whatever your rights: no users, groups, roles or organizations, no playbooks or dashboards, no notifications, ingestion, data sharing or connector settings. A tool that is not on the server's allow-list is never listed nor run.

Every tool carries the standard MCP annotations: reads are marked read-only, writes that only add something are marked not destructive, and writes that change or remove something are marked destructive, which most clients ask you to confirm. These are hints for your client; your permissions decide what a call may do.

:::warning
A change made through `/mcp/opencti` goes straight into the OpenCTI knowledge, as you. The drafts that XTM One agents write for review do not apply to MCP clients: read what your client proposes before you let it run a write tool.
:::

On `/mcp/xtm`, reading the traces of every agent at once is for administrators; anyone else names an agent they manage. Every trace listing is recorded in the audit log.

## When something does not work

| Answer | Meaning | What to do |
| --- | --- | --- |
| `401` | No key, or an invalid or revoked one | Check the `Authorization` header and that the key is active. A package whose key you revoked must be downloaded again. |
| `403` | AI is switched off, the server is turned off, or `/mcp/xtm` lacks the Enterprise Edition | The message says which. Ask an administrator. |
| `404` | The product is not registered with XTM One | Register the OpenCTI, OpenAEV or OpenCRQ instance first. |
| A tool is missing | It is outside the allow-list on purpose, or you have nothing it applies to | Administration tools are never exposed. On `/mcp/xtm`, the knowledge search appears only when you can read a knowledge base. |

A web-based client is accepted only from XTM One's own address or an address an administrator allows (`CORS_ORIGINS`, see the [configuration reference](/reference/xtm-one/configuration/)). Desktop and command-line clients are not concerned.

## What's next

- [Set up your profile and API keys](/xtm-one/use/profile/), which also exposes your agents as MCP tools.
- [XTM One in OpenCTI](/xtm-one/use/in-opencti/)
- [Review what agents did](/xtm-one/use/work-history/)
