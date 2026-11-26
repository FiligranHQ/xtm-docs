---
title: Troubleshoot OpenAEV
description: Fix the common problems of an OpenAEV instance, from a refused tenant or login to injects that do not run, and find the page that covers each part.
doc_type: how-to
products:
  - openaev
personas: [platform-engineer, admin]
owner: docs-openaev
last_reviewed: 2026-11-07
no_screenshot: "This page sends the reader to the screen or the log of each problem: the diagram shows where to look first."
migrated_from:
  - openaev:/troubleshooting/
---

This page helps you find why OpenAEV does not do what you expect. It covers a user who cannot open a tenant, a single sign-on login without rights, simulations that do not move, a refused player link and a slow screen. Each problem gives its likely cause and the fix. For the logs of the platform and of the other services of the stack, start with [Troubleshoot the XTM stack](/deploy/operate/troubleshoot/).

## Before you start

- For the access problems, your role has **Access tenant users, groups and roles** in the tenant, or the platform capabilities for the **Platform** scope.
- For the platform problems, you can read the logs of the `openaev` service: `docker compose logs --tail 200 openaev`.
- Note the version of the platform, in the **OpenAEV platform** card of :menu[Settings > Parameters]: support asks for it first. Click it to copy the version and its commit hash.

## Where to look first

::diagram{id=openaev-troubleshoot-where-to-look}

## Access problems

| What the user sees | Likely cause | Fix |
| --- | --- | --- |
| **Tenant access denied**, with **You are not a member of this tenant** | The account is not attached to the tenant of the address. Administrators are no exception | Attach it: in the tenant, :menu[Settings > Security > Users] and **Create** with the same email, or, with the **Platform** scope, add the tenant to the **Tenants** of the user. See [Give each organization its own tenant](/openaev/administer/multi-tenancy/#give-people-access) |
| A message that the account is not attached to any tenant | A single sign-on login whose `tenant_id` names no tenant, or a user removed from all their tenants | Fix `tenant_id`, or attach the user to a tenant |
| The user logs in but sees almost no menu | The user belongs to no group of the tenant, or to groups without roles | Add the user to a group with a role, such as **Observer** |
| After a single sign-on login, the mapped groups are missing | `groups_path` is not set, the IdP sends another value than `idpGroup`, or the group does not exist | Read the error the log gives for each missing group, then fix the mapping. See [Configure single sign-on](/openaev/administer/sso/#map-groups) |
| A single sign-on login fails for everyone of a provider | A mapping names a group whose name is used by two groups, such as **Manager** | Rename the target groups so that their names are unique on the platform |
| A user cannot add a capability to a role, or a user to a group | The administrator does not hold the capability, or a role of the group holds one they do not have | Ask an administrator who holds it. See [How access works in OpenAEV](/openaev/concepts/access/#how-it-works), rule 5 |
| A form says that a capability is missing | The action needs a second capability, such as **Access channels** to write an article | Add the capability to a role of the user |
| **Tenants** or the **Platform** scope do not show | The platform has no Enterprise Edition license, or the user has no platform role | See [Editions and licensing](/start/editions/) |

## Platform problems

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| The banner **Safe mode is active: background processing is disabled.** shows, and nothing runs | The platform runs with `OPENAEV_RUN-MODE: safe` | Return to normal mode: see [Start the platform in safe mode](/openaev/administer/run-modes/#return-to-normal-mode) |
| Injects stay pending on endpoints | No executor reaches the endpoints, or the agent is not running | See [How integrations run](/deploy/integrations/how-they-run/) and [OpenAEV agents](/openaev/use/agents/) |
| An integration deployed from the **Integrations** screen does not start | XTM Composer is not registered or cannot pull the image | See [Troubleshoot XTM Composer](/deploy/integrations/troubleshoot/), [Run integrations behind a proxy](/deploy/integrations/proxy/) and [Pull integrations from a private registry](/deploy/integrations/private-registry/) |
| A collector or injector you run yourself fills the wrong tenant | `OPENAEV_TENANT_ID` is missing: it works in the default tenant | Set it to the **Tenant identifier**. See [Run an integration without XTM Composer](/deploy/integrations/manual/) |
| A new tenant has no attack patterns | The MITRE ATT&CK collector works for another tenant | Deploy it for the new tenant. See [Manage the tags, attack patterns and vulnerabilities](/openaev/administer/taxonomies/#where-the-taxonomies-come-from) |
| A player's link to an article or a challenge is refused | The token expired, 7 days after it was sent while the simulation runs | Send the email again, or lengthen the margin. See [Protect the links in player emails](/openaev/administer/url-access-token/#how-long-a-link-lasts) |
| **SMTP** reads **Disabled** in the **Tools** card | The platform cannot connect to its mail server | Check the [spring.mail block](/reference/openaev/configuration/#spring-mail) and the network to the server |
| The XTM Hub card reads **Connectivity lost** | The platform no longer reaches XTM Hub, or the connection was removed on the XTM Hub side | See [Connect a tenant to XTM Hub](/openaev/administer/xtm-hub/#keep-the-connection-alive) |
| A screen is slow, or a request fails without a clear error | A slow or repeated SQL query, or a load on the JVM | Trace it: see [Turn on the debug mode to find slow requests](/openaev/administer/debug-mode/) |

:::example[Find why Grand Meridian's CISO reaches no tenant]
Hélène Marchetti, CISO of Grand Meridian Hotels, logs in with **Log in with Grand Meridian** and is told that her account is not attached to any tenant. The administrator of Example CERT checks:

1. With the **Platform** scope, in **Users**, her account exists, but its **Tenants** list is empty: her first login did not attach it.
2. The log of the `openaev` service shows `SSO tenant ID '7c1e4b2a-5d3f-4e8b-9a61-2f0c8d4e6b31' configured but not found in database`: two characters of `OPENAEV_PROVIDER_GRANDMERIDIAN_TENANT_ID` are swapped.
3. He copies the **Tenant identifier** from :menu[Settings > Parameters] in the Grand Meridian Hotels tenant, `7c1e4b2a-5d3f-4e8b-9a61-2f0c8d4e6b13`, and the platform engineer fixes the variable and recreates the container.

Hélène's next login attaches her account to the Grand Meridian Hotels tenant, puts it in the groups of that tenant that have **Auto assign** on, and opens the tenant.
:::

## Ask for help

When you open a ticket with Filigran support, give the version and commit hash of the platform, the time of the problem, its tenant and its page, and the log lines around it. For a slow or failing request, add the output of the [debug mode](/openaev/administer/debug-mode/).

## What's next

- [Troubleshoot the XTM stack](/deploy/operate/troubleshoot/), for the logs of every service and their level.
- [Turn on the debug mode to find slow requests](/openaev/administer/debug-mode/).
- [How access works in OpenAEV](/openaev/concepts/access/).
