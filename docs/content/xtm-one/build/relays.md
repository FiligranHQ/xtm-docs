---
title: Reach on-premise tools through a relay
description: Install a relay inside a network XTM One cannot reach, so that integrations and MCP servers call the services there without any inbound connection or VPN.
doc_type: how-to
products: [xtm-one]
personas: [admin, platform-engineer, ai-builder]
owner: docs-xtm-one
last_reviewed: 2026-11-08
migrated_from:
  - xtm-one:/usage/resources/relays/
---

A relay lets XTM One reach the services of a network it cannot reach directly, such as a SIEM in your datacenter, an internal API or an on-premises OpenCTI, without opening a single inbound connection to that network. You install the relay on a machine or in a container inside the network. It connects out to XTM One over HTTPS and carries the requests of the integrations and MCP servers routed through it. This page creates a relay group, enrolls a relay, routes a resource through it and follows its traffic.

::diagram{id=xtm-one-build-relays-path}

## How relays work

Relays belong to a relay group, and a group stands for one network: a datacenter, a cloud account, a site. A resource is routed through a group, never through one relay: every relay of the group can carry its requests, and XTM One balances them across the relays that are online. Two relays or more in a group keep the traffic flowing when one stops.

Once a resource is routed, all its requests go through the group: its tools, its connection test, its OAuth sign-in and token refresh, its event polling. When no relay of the group is online, those requests fail with a message saying so. A routed resource never falls back to a direct call.

Two lists decide what a relay reaches, and a request must pass both:

- The group's **Allowed destinations**, checked by XTM One before the request reaches a relay. Empty means no limit on the platform side.
- The relay's own list, set when it is enrolled and kept on its machine. It always has the last word: with an empty list, the relay reaches nothing.

## Before you start

- Only administrators create relay groups. Administrators enroll the relays of a company-managed group; a group shared with you to configure it lets you enroll its relays too. A user routes a resource through a group that is company-managed or shared with them.
- Relays are on by default. A platform engineer can remove them with `RELAYS_ENABLED=false`, which hides the **Relays** tab: see the [Relays section of the configuration reference](/reference/xtm-one/configuration/#relays), with the cap on relays per group and the retention of the request log.
- A machine or a container platform inside the network (Linux, Windows, macOS, Docker or Kubernetes) that can reach XTM One over HTTPS, directly or through a proxy.

## Create a relay group

:::steps
1. Open :menu[Tools & Integrations > Connections], then the **Relays** tab, and click **New relay group**.
2. Enter a **Name** for the network, an optional **Zone**, and a **Description** of what the network holds.
3. Optionally list the **Allowed destinations**, as `host:port`, wildcards (`*.corp.local:443`) or networks (`10.20.0.0/16:8089`).
4. Keep **Company-managed** on to let every user route their resources through the group, or turn it off and share the group with groups or people afterwards, in its **Settings** tab.
5. Click **Create and enroll a relay**. The enrollment dialog opens.
:::

## Enroll a relay

:::steps
1. Under **Where will it run?**, choose **Linux**, **Windows**, **macOS**, **Docker** or **Kubernetes**.
2. Optionally enter a **Relay name (optional)**. The machine's hostname is used otherwise.
3. Under **What may it reach?**, list the services the relay may carry requests to.
4. Click **Generate the install command**, and run the command inside the network: as root on Linux and macOS, in an elevated PowerShell on Windows, on a Docker host, or with `kubectl` against a cluster in the network. On a machine, it downloads the relay from your XTM One, checks its signature and installs it as a system service.
5. Wait in the dialog: once the relay connects, it says the relay is connected, and you can route resources through the group.
:::

::screenshot{id=xtm-one-build-relays-enroll alt="The Enroll a relay in Paris datacenter dialog on Docker: the optional relay name, booking-tools.grand-meridian.example:443 under What may it reach?, taken from the group's allowed destinations, and Generate the install command."}

The enrollment key in the command is shown only once. A key for one machine enrolls one relay and expires soon; a key for several Kubernetes replicas stays valid for 7 days, and each replica enrolls as its own relay. Your XTM One serves the relay installers itself, so a deployment without internet access needs nothing else. A relay installed as a service updates itself; a relay in a container is updated by pulling a newer image.

### Change what a relay reaches

A relay reads its settings from `relay.toml`, and reloads the file when it changes: `/etc/xtm-one-relay/relay.toml` on Linux, `/Library/Application Support/XTM One Relay/relay.toml` on macOS, `%ProgramData%\XTM One Relay\relay.toml` on Windows. In a container, each setting is an environment variable instead, such as `XTM_RELAY_ALLOWED_DESTINATIONS`.

| Setting | What it does |
| --- | --- |
| `allowed_destinations` | The hosts, wildcards and networks the relay may reach, each with its port. `*` allows every destination, and the security audit flags it. |
| `ca_bundle` | A file of internal certificate authorities to trust, for services signed by your own CA. |
| `upstream_proxy` | The outbound proxy the relay uses to reach XTM One. |
| `transport` | `auto` (WebSocket, falling back to long-poll), `websocket` or `long_poll`, for a proxy that refuses WebSocket. |
| `max_concurrent_requests` | How many requests the relay carries at once. |
| `health_listen` | An address for a local health endpoint: `GET /healthz` answers 200 while the relay is connected. |

On the relay's machine, `xtm-one-relay status` says whether it is enrolled and connected (exit code 0 when online), `xtm-one-relay check-config` shows its effective allow-list, and `xtm-one-relay test-destination` sends a request from the machine through the relay's own checks.

## Route a resource through the group

On the configuration of an integration or of a remote MCP server, choose the group under **Route through** instead of **Direct (from the platform)**. The field warns you when no relay of the group is online, or when the group's or a relay's list does not include the resource's address. An integration of a service reached only through a relay, such as an LM Studio model server, offers no **Direct (from the platform)**: it cannot be created without a group. A local MCP server (**Subprocess (stdio)**) runs on the platform itself and cannot be routed.

See [Connect an MCP server](/xtm-one/build/mcp-servers/) and [Connect agents to your tools with integrations](/xtm-one/build/integrations/).

## Follow a group

The group's page has six tabs:

- **Overview**: relays online, requests and failures over the last 24 hours, latency, traffic and the destination policy.
- **Relays**: each relay with its status, version, connection (**WebSocket** or **Long-poll**), what its own list allows and when it was last seen. **Revoke** cuts a relay off at once; **Enrollment keys** lists the keys.
- **Routed resources**: what is routed through the group.
- **Activity**: every request a relay carried, newest first, with its method and address, its outcome, its source (an integration, an MCP server, a flow or a test), the relay, the duration and the size. Never a header, a body or a credential. Search by host, narrow the list with the **Outcome** (**Succeeded** or **Failed**) and **Relay** filters, where you can pick several relays at once, and **Clear all filters** to start over. Click **When** to put the oldest first, and **Export CSV** to download the rows of the page. The page size and the columns you hide under **Columns** are kept in your browser for your next visit. Hover a failed outcome to read why it failed and what to check.
- **Diagnostics**: **Test a destination** sends one request from the group's network, and the commands to run on the relay's machine when something does not connect.
- **Settings**: name, zone, allowed destinations, sharing, and **Delete this group**, refused while something is still routed through it.

<!-- shot: xtm-one-build-relays-group -->

The security audit, in :menu[Administration > Settings > Access and security > Security audit], reports relays that allow every destination, relays older than the minimum supported version, groups in use with no relay online, and groups with a single relay.

:::example[Reach the booking platform in the Paris datacenter]
The internal MCP server of the hotel booking platform listens on `booking-tools.grand-meridian.example:443`, inside the Paris datacenter, which accepts no inbound connection from the internet. Hélène Marchetti creates the group in :menu[Tools & Integrations > Connections], tab **Relays**, **New relay group**:

- **Name**: `Paris datacenter`
- **Zone**: `Paris`
- **Description**: `Booking platform and internal tools of the head office`
- **Allowed destinations**: `booking-tools.grand-meridian.example:443`
- **Company-managed**: on

She clicks **Create and enroll a relay**, chooses **Docker**, lists `booking-tools.grand-meridian.example:443` under **What may it reach?**, and hands the generated command to Marc Dupuis, who runs it on a Docker host of the datacenter. The command has this shape, with the one-time key the dialog shows in place of the `RELAY_ENROLLMENT_KEY` variable:

```bash
docker run -d --name xtm-one-relay --restart unless-stopped \
  -v xtm-one-relay:/var/lib/xtm-one-relay \
  -e XTM_RELAY_SERVER_URL='https://xtm-one.grand-meridian.example' \
  -e XTM_RELAY_ENROLLMENT_KEY="$RELAY_ENROLLMENT_KEY" \
  -e XTM_RELAY_ALLOWED_DESTINATIONS='booking-tools.grand-meridian.example:443' \
  -e XTM_RELAY_NAME='paris-relay-1' \
  xtmone/relay:1.261112.0
```

Marc enrolls a second relay on another host, `paris-relay-2`, for redundancy. The group shows **2 of 2 relays online**, and Sophie Laurent routes the **Hotel booking platform** MCP server through **Paris datacenter**: its tools now answer from inside the datacenter.
:::

## What's next

- [Connect an MCP server](/xtm-one/build/mcp-servers/)
- [Connect agents to your tools with integrations](/xtm-one/build/integrations/)
- [Configure XTM One](/deploy/configure/xtm-one/)
