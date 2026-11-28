---
title: XTM One MCP tools
description: Every tool of the MCP servers XTM One exposes for OpenCTI, OpenAEV and OpenCRQ in this version,
  what it does and whether a read-only key may call it, generated from the source code.
sidebar_label: MCP tools
doc_type: reference
products:
  - xtm-one
  - opencti
  - openaev
  - opencrq
personas: [developer, ai-builder]
generated: true
generator: reference:xtm-one:mcp-tools
---

::diagram{id=reference-xtm-one-mcp-tools}

XTM One serves one Model Context Protocol (MCP) server per module registered with it: `/mcp/opencti`, `/mcp/openaev` and `/mcp/opencrq`. An assistant or an agent of your own lists the tools of a server and calls them with a personal API key that has the **MCP endpoints** surface, or with an OAuth access token. Each call runs in the module as the key's user, matched by email, with the rights of that account. [MCP servers](/develop/apis/mcp/) explains the endpoints, the protocol, the authentication and the answers.

A tool marked **Read** changes nothing, and a read-only API key may call it. The others create, change or delete data in the module.

XTM One has three more servers, `/mcp/xtm`, `/mcp/user` and `/mcp/global`. Their tools depend on who calls them: the agents and the resources that user may see. Ask the server for its list with `tools/list`.
