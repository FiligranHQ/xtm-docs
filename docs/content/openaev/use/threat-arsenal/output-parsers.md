---
title: Turn an action's output into findings
description: Add output parsers to an action so OpenAEV extracts hosts, ports, credentials or CVEs from its raw output, turns them into findings, and feeds them to the next steps.
doc_type: how-to
products:
  - openaev
personas: [validation-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/build/threat-arsenals/output-parsers/
---

An output parser reads the raw output of an [action](/openaev/use/threat-arsenal/) and pulls structured values out of it: the open ports a scan reported, the credentials a search returned, the CVEs a check found. Those values become findings you can read and sort, and they feed the next steps of a chained scenario. Without a parser, an action's output is just text. This page adds parsers to an action, on the **Output** tab of its form. Creating the action is in [Build the actions of your threat arsenal](/openaev/use/threat-arsenal/).

## Before you start

- Your role has **Manage threat arsenal** to edit an action.
- You know the raw output of the action, so you can write a regular expression that matches it.
- Output parsers read the standard output (**StdOut**) and match with a regular expression (**REGEX**); these are the modes available today.

## How a parser turns output into findings

::diagram{id=openaev-output-parsers-flow}

An action runs on a target and sends its raw output back to OpenAEV. Each parsing rule applies its regular expression, maps the capture groups to the fields of a type, and keeps the matches. When **Create finding(s)** is on, each match becomes a finding; the values also go into the run's state, where a chained scenario's events and arguments read them.

## Add an output parser

:::steps
1. Open the action from the Threat Arsenal, or create one, and go to the **Output** tab.
2. Click **Add attribute**, then set the rule:

   | Field | What it is | Required |
   | --- | --- | --- |
   | **Name** | The name of the rule | Yes |
   | **Key** | A unique key for the rule | Yes |
   | **Type** | The data type to extract, such as **Text**, **Number**, **Port**, **Ports scan**, **IPv4**, **IPv6**, **Credentials**, **CVE** or **Vulnerability** | Yes |
   | **Tags** | Labels for the rule | No |
   | **Regex group rules** | The regular expression that extracts the data, with its capture groups. It runs with multiline, case-insensitive and Unicode matching | Yes |
   | **Output value** | Maps each capture group to a field of the type | Yes |

3. Under **Output value**, map each field to a capture group, written `$1`, `$2` and so on. To join several groups in one field, write them next to each other, such as `$1$2`.
4. To make the matches findings, turn on **Create finding(s)** for the rule.
5. Save the action.
:::

::screenshot{id=openaev-output-parsers-output-tab alt="The Output tab of the action Velvet Harbor - booking web front discovery: Output mode Stdout and Parsing Regex, and the rule Database account of type Credentials, Create finding(s) checked, with its regular expression and its Hash, Password and Username output values."}

The number of fields depends on the type:

| Type | Fields | Finding value |
| --- | --- | --- |
| **Ports scan** | host, port, service | `host:port (service)` |
| **Credentials** | username, password, hash | `username:password` |
| **CVE** | host, id, severity | `host:id (severity)` |
| **Other** | a single value | the value |

## Check that it works

- Run the action, in an atomic test or a simulation, and open its results.
- When **Create finding(s)** is on, the extracted values show on the **Findings** tab, and the parser's details on the **Action details** tab, of the [atomic testing detail view](/openaev/use/atomic-testing/).
- In a chained scenario, the values become available to the events and to the arguments of later actions. See [Build the logic of a chained scenario](/openaev/use/attack-chaining/logic/).

:::example[Extract the services the booking web front reaches]
The action **Velvet Harbor - booking web front discovery** prints one line per service it reaches from the web front of the booking system, such as `reachable 10.20.8.15:5432 postgresql`. The validation engineer of Grand Meridian Hotels adds a rule on its **Output** tab:

| Field | Value |
| --- | --- |
| **Name** | `Reachable services` |
| **Key** | `reachable_services` |
| **Type** | **Ports scan** |
| **Regex group rules** | `reachable ([\d.]+):(\d+) (\S+)` |
| **Output value** | **Host** `$1`, **Port** `$2`, **Service** `$3` |
| **Create finding(s)** | On |

When the atomic test runs on `booking-web-01`, the line becomes the **Ports scan** finding `10.20.8.15:5432 (postgresql)`. The action has two more rules, **Apache CVE** of type **CVE** and **Database account** of type **Credentials**, whose password the findings mask.
:::

## What's next

- [Build the actions of your threat arsenal](/openaev/use/threat-arsenal/): create and manage the actions parsers apply to.
- [Set the properties of an action](/openaev/use/threat-arsenal/action-properties/): every field of the form.
- [Findings](/openaev/use/findings/): read and sort the values extracted.
- [Build the logic of a chained scenario](/openaev/use/attack-chaining/logic/): branch on the values a parser extracts.
