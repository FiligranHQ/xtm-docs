---
title: MCP servers
description: The Model Context Protocol servers XTM One exposes for OpenCTI, OpenAEV, OpenCRQ and itself, their endpoints, protocol versions, authentication, answers and every tool they offer.
doc_type: reference
products:
  - xtm-one
  - opencti
  - openaev
  - opencrq
personas: [developer, ai-builder]
owner: docs-platform
last_reviewed: 2026-11-09
sidebar_position: 6
---

The modules of the XTM platform reach Model Context Protocol (MCP) clients through XTM One. XTM One serves one MCP server per registered product, OpenCTI, OpenAEV and OpenCRQ, and its own servers for its agents and its configuration. A client such as Claude, Cursor, VS Code or an agent you write calls their tools with the identity of a user, and each product applies that user's rights. This page is the reference of these servers in XTM One 1.261119: endpoints, protocol, authentication, answers and tools. To connect a client step by step, see [Use XTM One's own MCP servers from other clients](/xtm-one/build/platform-mcp-servers/).

::diagram{id=develop-apis-mcp}

## Endpoints

Every endpoint is on the address of XTM One, such as `https://xtm-one.example.com/mcp/opencti`.

| Endpoint | Server name | What its tools reach | Available when | Edition |
| --- | --- | --- | --- | --- |
| `/mcp/opencti` | OpenCTI (via XTM One) | OpenCTI knowledge: entities, observables, indicators, relationships, containers, notes, labels, markings | An OpenCTI is registered with XTM One | Community |
| `/mcp/openaev` | OpenAEV (via XTM One) | OpenAEV scenarios, simulations, injects, payloads, atomic tests, expectations, findings, assets, content | An OpenAEV is registered | Community |
| `/mcp/opencrq` | OpenCRQ (via XTM One) | OpenCRQ frameworks, controls, assessments, risk scenarios, prices, assets, findings, threats, treatment plans | An OpenCRQ is registered | Community |
| `/mcp/user` | | The agents the user selected in their profile, or every agent they can see; one tool per agent | AI is on | Enterprise |
| `/mcp/global` | | The agents an administrator exposes in :menu[Administration > Settings > Integrations > MCP Endpoint]; one tool per agent | AI is on | Enterprise |
| `/mcp/xtm` | | XTM One itself: agents, assignments, runs and their traces, knowledge bases, prompts, skills, custom tools, integrations, MCP servers, variables | An administrator turns it on | Enterprise |

The server name carries the platform name of your branding in place of XTM One. The three product servers, the platform MCP servers, call the APIs of the products and no language model. A product registered under its former identifier, `openbas` for OpenAEV or `opengrc` for OpenCRQ, is served at `/mcp/openaev` and `/mcp/opencrq` all the same.

An administrator turns the platform MCP servers off with **Platform MCP Servers**, and `/mcp/xtm` on with **Platform Tools MCP Server**, in :menu[Administration > Settings > Integrations > MCP Endpoint]. With AI switched off, every endpoint refuses.

::screenshot{id=xtm-one-build-platform-mcp-servers-profile alt="The Platform MCP Servers card of the MCP Endpoint tab of a profile, 3 connected: OpenCTI with 28 tools and OpenAEV with 52, each with its address under /mcp/, a copy button and Config."}

## Protocol

| Item | Value |
| --- | --- |
| Transport | Streamable HTTP: JSON-RPC 2.0 over `POST`, answered in JSON. No session, no change notifications, no stdio: `GET` and `DELETE` answer `405` |
| Protocol versions | `2026-07-28`, without handshake; `2025-11-25`, `2025-06-18` and `2025-03-26`, negotiated by `initialize`. A request without `MCP-Protocol-Version` is read as `2025-03-26` |
| Methods | `tools/list` and `tools/call`; `initialize` and `ping` up to `2025-11-25`; `server/discover` and `subscriptions/listen` in `2026-07-28` |
| Batches | Accepted for a request in `2025-03-26` only |
| Tool annotations | Reads are marked read-only, writes that only add are marked not destructive, writes that change or remove are marked destructive |

A call to a tool the server does not list is a JSON-RPC error `-32602`, `Unknown tool`. A tool that runs and fails answers a result with `isError` set and a message the model can read.

## Authentication

| Credential | Header | Endpoints |
| --- | --- | --- |
| A personal API key, `fcp-...`, created in **My Profile** > **API Keys** with the **MCP endpoints** surface | `Authorization: Bearer fcp-...` | All but `/mcp/global` |
| An OAuth 2.1 access token, issued to a client the user approves in XTM One | `Authorization: Bearer` followed by the token | All, each token bound to one endpoint |
| The platform MCP key, `mcp-...`, set by an administrator | `Authorization: Bearer mcp-...` | `/mcp/global` |

A request without a valid credential answers `401` with a `WWW-Authenticate` header that points to the protected resource metadata (RFC 9728), from which an OAuth client finds the sign-in by itself.

A product server never holds a token of the product. For each call, XTM One signs a short-lived token for the user, matched by email in OpenCTI and OpenAEV. OpenCRQ matches it to the account tied to the user's XTM One account (the token's `sub`), never to an email address. The product applies that account's capabilities, markings and audit trail. Changes made through `/mcp/opencti` go straight to the knowledge, not to a draft.

| API key setting | Effect on the MCP servers |
| --- | --- |
| **Read-only** | `initialize`, `tools/list`, and `tools/call` on the tools marked read below. On `/mcp/user`, every tool runs an agent, so a read-only key calls none |
| **Limit to specific agents** | The key reaches the `/mcp/user` tools of those agents only; the product servers and `/mcp/xtm` refuse it |

A web page is accepted only from the address of XTM One or an origin of `CORS_ORIGINS`. Desktop, command-line and hosted clients send no `Origin` and are not concerned.

## Answers

| Code | Meaning |
| --- | --- |
| `202` | A notification or a client response, with nothing to answer |
| `400` | A malformed request, an unsupported protocol version, or a batch outside `2025-03-26` |
| `401` | No key, or an invalid, expired or revoked one |
| `403` | AI is switched off, the server is turned off, the key does not cover the endpoint, the edition does not include it, or a web origin is not allowed. The `detail` says which |
| `404` | The product of the endpoint is not registered with XTM One |
| `429` | The key or its user reached a rate limit or a quota |

## Tools of the OpenCTI server

The sections below group the tools of each server. The [MCP tools reference](/reference/xtm-one/mcp-tools/) lists each one with what it does.

The 28 tools of `/mcp/opencti`. Platform administration is never exposed: no users, groups, roles, playbooks, dashboards, notifications, ingestion, data sharing or connector settings.

| Group | Tools | Read-only key |
| --- | --- | --- |
| Read and search | `search_opencti`, `get_opencti_entity`, `get_opencti_relationship`, `list_opencti_entities`, `list_opencti_observables`, `list_opencti_relationships`, `get_opencti_container_full`, `get_opencti_entity_stats`, `get_opencti_top_related`, `get_opencti_entity_history`, `validate_nlq_filter` | Yes |
| Create | `create_opencti_entity`, `create_opencti_relationship`, `create_opencti_observable`, `create_opencti_indicator`, `create_opencti_container`, `create_opencti_note` | No |
| Update and delete | `update_opencti_field`, `update_opencti_relationship_field`, `delete_opencti_entity`, `delete_opencti_relationship` | No |
| Labels, markings and containers | `add_opencti_label`, `remove_opencti_label`, `add_opencti_marking`, `remove_opencti_marking`, `add_opencti_to_container`, `remove_opencti_from_container` | No |
| Observables | `promote_opencti_observable` | No |

## Tools of the OpenAEV server

The 50 tools of `/mcp/openaev`. People and group management, taxonomy changes and platform administration stay out. The `manage_*` tools take an action that reads or writes, so a read-only key calls none of them.

| Group | Tools | Read-only key |
| --- | --- | --- |
| Read and search | `search_openaev_assets`, `get_openaev_asset`, `search_openaev_asset_groups`, `get_openaev_asset_group`, `search_openaev_teams`, `get_openaev_team`, `search_openaev_players`, `search_openaev_attack_patterns`, `get_openaev_attack_pattern`, `search_openaev_scenarios`, `get_openaev_scenario`, `search_openaev_exercises`, `get_openaev_exercise`, `search_openaev_payloads`, `get_openaev_payload`, `search_openaev_injector_contracts`, `get_openaev_injector_contract`, `search_openaev_findings`, `get_openaev_vulnerability`, `full_text_search_openaev`, `list_openaev_kill_chain_phases`, `list_openaev_tags`, `list_openaev_domains` | Yes |
| Analytics | `query_openaev_analytics` | Yes |
| Create and change | `create_openaev_scenario`, `update_openaev_scenario`, `delete_openaev_scenario`, `create_openaev_payload`, `add_openaev_inject_to_scenario`, `create_openaev_atomic_testing` | No |
| Manage by domain | `manage_openaev_simulation`, `manage_openaev_scenario`, `manage_openaev_inject`, `manage_openaev_expectation`, `manage_openaev_atomic_testing`, `manage_openaev_payload`, `manage_openaev_threat_arsenal`, `manage_openaev_finding`, `manage_openaev_vulnerability`, `manage_openaev_asset`, `manage_openaev_asset_group`, `manage_openaev_document`, `manage_openaev_channel`, `manage_openaev_article`, `manage_openaev_challenge`, `manage_openaev_lessons`, `manage_openaev_reporting`, `manage_openaev_phishing`, `manage_openaev_dashboard`, `manage_openaev_variable` | No |

`manage_openaev_phishing` covers landing pages and email templates; OpenAEV checks the phishing capabilities of the user on each call.

## Tools of the OpenCRQ server

The 85 tools of `/mcp/opencrq`. Tenant and user administration, service accounts, engine settings and connectors stay out.

| Group | Tools | Read-only key |
| --- | --- | --- |
| Tenancy and frameworks | `list_opencrq_tenants`, `list_opencrq_organizations`, `list_opencrq_frameworks`, `get_opencrq_framework`, `list_opencrq_framework_requirements`, `list_opencrq_drata_controls` | Yes |
| Controls, read | `list_opencrq_control_capabilities`, `get_opencrq_control_capability`, `get_opencrq_control_capability_history` | Yes |
| Controls, write | `update_opencrq_control_capability_assessment`, `manage_opencrq_scope_control_posture`, `manage_opencrq_capability_techniques` | No |
| Assessments, read | `list_opencrq_compliance_assessments`, `get_opencrq_compliance_assessment` | Yes |
| Assessments, write | `create_opencrq_compliance_assessment`, `manage_opencrq_compliance_assessment`, `update_opencrq_requirement_answer` | No |
| Risk, read | `list_opencrq_risk_scenarios`, `get_opencrq_risk_scenario`, `get_opencrq_scenario_exposure`, `list_opencrq_attack_outcomes`, `get_opencrq_attack_outcome` | Yes |
| Risk, write | `create_opencrq_risk_scenario`, `update_opencrq_risk_scenario`, `delete_opencrq_risk_scenarios`, `update_opencrq_risk_tolerance` | No |
| Quantification, read | `get_opencrq_financial_summary`, `get_opencrq_crq_run`, `list_opencrq_crq_run_history`, `get_opencrq_crq_what_if`, `get_opencrq_tef_input`, `list_opencrq_loss_drivers`, `get_opencrq_loss_driver`, `get_opencrq_financial_profile` | Yes |
| Quantification, write | `run_opencrq_crq_simulation`, `update_opencrq_tef_input`, `update_opencrq_loss_estimates`, `update_opencrq_financial_profile` | No |
| Assets and CVEs, read | `list_opencrq_primary_assets`, `list_opencrq_business_value_categories`, `list_opencrq_supporting_assets`, `get_opencrq_supporting_asset`, `list_opencrq_asset_dependencies`, `get_opencrq_evidence`, `list_opencrq_cve_evidences`, `list_opencrq_unique_findings`, `search_opencrq_cves`, `list_opencrq_cves`, `get_opencrq_cve` | Yes |
| Assets and CVEs, write | `create_opencrq_primary_asset`, `update_opencrq_primary_asset`, `delete_opencrq_primary_assets`, `create_opencrq_supporting_asset`, `update_opencrq_supporting_asset`, `delete_opencrq_supporting_assets`, `reset_opencrq_supporting_asset_to_connectors`, `create_opencrq_asset_dependency`, `update_opencrq_asset_dependency_roles`, `delete_opencrq_asset_dependencies`, `create_opencrq_asset_evidence`, `update_opencrq_evidence_state`, `delete_opencrq_asset_evidence`, `reset_opencrq_evidence_state` | No |
| MITRE ATT&CK | `list_opencrq_mitre_tactics`, `search_opencrq_mitre_techniques`, `get_opencrq_mitre_technique`, `list_opencrq_mitre_mitigations` | Yes |
| Jobs | `list_opencrq_jobs`, `get_opencrq_job`, to follow a simulation started by `run_opencrq_crq_simulation` | Yes |
| Findings, threats, treatment and dashboard | `get_opencrq_findings_summary`, `get_opencrq_top_vulnerabilities`, `list_opencrq_findings`, `get_opencrq_cve_finding`, `get_opencrq_asset_posture`, `get_opencrq_technique_test_posture`, `list_opencrq_threats`, `get_opencrq_threat`, `list_opencrq_scenario_threats`, `list_opencrq_cve_threats`, `list_opencrq_treatment_plans`, `get_opencrq_treatment_plan`, `list_opencrq_budget_reviews`, `get_opencrq_budget_review`, `get_opencrq_control_validation`, `get_opencrq_risk_dashboard` | Yes |

## Example: list the tools of the OpenCTI server

A client speaking `2025-06-18` opens with `initialize`, then lists the tools. Each request is one `POST`, with the key in `XTM_ONE_API_KEY`:

```bash
curl -s https://xtm-one.example.com/mcp/opencti \
  -H "Authorization: Bearer $XTM_ONE_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "MCP-Protocol-Version: 2025-06-18" \
  -d '{"jsonrpc": "2.0", "id": 1, "method": "tools/list"}' | jq -r '.result.tools[].name'
```

With a read-only key, the answer lists the 11 read and search tools only. A call runs as the key's user: asking `search_opencti` for Velvet Harbor returns only the knowledge that user's groups and markings let them see.

## What's next

- [Use XTM One's own MCP servers from other clients](/xtm-one/build/platform-mcp-servers/), to connect Claude, Cursor or VS Code with a package or by hand.
- [Set up your profile and API keys](/xtm-one/use/profile/), to create a key with the MCP endpoints surface.
- [Connect an MCP server](/xtm-one/build/mcp-servers/), for the other direction: giving XTM One agents the tools of another server.
